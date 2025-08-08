> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/tangta789/article/details/126345759)

**更多源码分析请访问：**[LVGL 源码分析大全](https://blog.csdn.net/tangta789/article/details/126309716)  

1、分析原因
------

LVGL 本身是没有对接最终的显示框架的，所以到具体的某个平台上，需要自行 porting。

2、源码分析
------

先来两段 porting 的代码片段：

```c
static void DisplayInitFB(int width,int height)
{
	lv_color_t *buf = malloc(width * height * sizeof(lv_color_t) * 2);
	memset(buf,0,width * height * sizeof(lv_color_t) * 2);
	static lv_disp_draw_buf_t disp_buf;
    lv_disp_draw_buf_init(&disp_buf, buf, &buf[width * height], width * height);
    static lv_disp_drv_t disp_drv;
    lv_disp_drv_init(&disp_drv);
    disp_drv.draw_buf   = &disp_buf;
    disp_drv.flush_cb   = fbdev_flush;
    disp_drv.hor_res    = width;
    disp_drv.ver_res    = height;
    lv_disp_drv_register(&disp_drv);
}
```

```c
static void DisplayInitSDL2(int width,int height)
{
    sdl_init();
    /*A small buffer for LittlevGL to draw the screen's content*/
    lv_color_t *buf = malloc(width * height * sizeof(lv_color_t) * 2); 
    memset(buf,0,width * height * sizeof(lv_color_t) * 2); 
    /*Initialize a descriptor for the buffer*/
    static lv_disp_draw_buf_t disp_buf;
#if USE_SDL_GPU
    sdl_gpu_disp_draw_buf_init(&disp_buf);
#else
    lv_disp_draw_buf_init(&disp_buf, buf, &buf[width * height], width * height);
#endif
    /*Initialize and register a display driver*/
    static lv_disp_drv_t disp_drv;
#if USE_SDL_GPU
    sdl_gpu_disp_drv_init(&disp_drv);
#else
    lv_disp_drv_init(&disp_drv);
#endif
    disp_drv.draw_buf   = &disp_buf;
    disp_drv.flush_cb   = sdl_display_flush;
    disp_drv.hor_res    = width;
    disp_drv.ver_res    = height;
    lv_disp_t * disp = lv_disp_drv_register(&disp_drv);
#if USE_SDL_GPU
    sdl_display_resize(disp,width,height);
#endif
}
```

`DisplayInitFB` 和 `DisplayInitSDL2` 分别 porting 了 framebuffer 和 SDL2 的显示渲染功能。同时，其支持了 LVGL 的双 buffer 功能。源码中两种方案都是通过`lv_disp_drv_register` 接口向框架注册了`lv_disp_drv_t` 对象来完成的。值的注意的是 SDL2 又分为 GPU 绘图和 CPU 绘图，采用了宏 USE_SDL_GPU 来控制。显然，如果要跑的 Linux 上支持 GPU，打开 USE_SDL_GPU 的宏对效率提升不少。  
很显然，这套接口代码是多例模式实现的，故而 LVGL 是支持多 display 渲染的。这点在官方文档里也有说明。

> _节选自：https://docs.lvgl.io/master/intro/index.html#key-features 中的 Key features_  
> Multi-display support, i.e. use multiple TFT, monochrome displays simultaneously

在文档中又有说明，可以通过`lv_scr_act`接口来获取当前活跃的 display 对象，又可以通过`lv_disp_set_default` 接口来设置当前活跃的 display 对象。由此可知，多屏时，具体控制那一个屏来做当前操作是由开发者自行决定的，LVGL 只是提供了相关接口，并未做任何逻辑切换的代码功能。

> _节选自：https://docs.lvgl.io/master/porting/display.html#_CPPv419lv_disp_set_defaultP9lv_disp_t_ 

> ![[6e6f07d9ded5d2dc1ea7387d3dbee61f_MD5.png]]

回到`lv_disp_drv_register`接口，有几个关键的设置需要注意。先给出节选的关键代码如下（代码在`lvgl/src/hal/lv_hal_disp.c`）：

```c
lv_disp_t * lv_disp_drv_register(lv_disp_drv_t * driver)
{
	// 1. 渲染所用的定时器
	disp->refr_timer = lv_timer_create(_lv_disp_refr_timer, LV_DISP_DEF_REFR_PERIOD, disp);
	// 2. 支持主题
	#if LV_USE_THEME_DEFAULT
    if(lv_theme_default_is_inited() == false) {
        disp->theme = lv_theme_default_init(disp, lv_palette_main(LV_PALETTE_BLUE), lv_palette_main(LV_PALETTE_RED),
                                            LV_THEME_DEFAULT_DARK, LV_FONT_DEFAULT);
    }   
    else {
        disp->theme = lv_theme_default_get();
    }   
#endif
	// 3. 渲染的图层分布
	disp->act_scr   = lv_obj_create(NULL); /*Create a default screen on the display*/
    disp->top_layer = lv_obj_create(NULL); /*Create top layer on the display*/
    disp->sys_layer = lv_obj_create(NULL); /*Create sys layer on the display*/
    // 4. 刷新图层
    lv_obj_invalidate(disp->act_scr);
}
```

上面的几个关键功能代码引发以下几个问题，后续我们一个一个来解析。

*   LV_DISP_DEF_REFR_PERIOD 是在`lv_conf.h`里定义的。如果 UI 没有修改，LVGL 是按此时间定时将数据渲染，还是暂停送数据？
*   为啥要设计三个图层，各有什么用途？
*   三个图层如何合并送显的？
*   为什么在注册接口中只刷新了一个`act_scr`图层？

### 2.1、三个图层

关于图层的描述，LVGL 在 文档 [Overview-Objects-Screens-Layers](https://docs.lvgl.io/master/overview/object.html#layers) 中有描述。描述中只提到两个图层 `top layer` 和 `system layer`，实际上创建的屏幕也是一个图层。

> 节选自：  
> ![[d03d19cf79e4ae9f436cfb8fcd54aa9e_MD5.png]]

描述中讲述了三个层的用途，`top layer` 和 `sys layer` 显示在屏幕 (`act_scr`) 之上，`top layer` 用于弹出窗口的显示，而`sys layer` 用于例如鼠标光标等被要求绝对显示在顶层的内容。当然，`act_scr` 就是一般 UI 所在的位置了。它的图层层级关系大致可以如下图所示：  
![[8544c9010935fbbb34acda91ecd7ffcf_MD5.png]]

### 2.2、合并送显

前面代码中有看到在 `lv_disp_drv_register` 接口中会启动 `_lv_disp_refr_timer` 回调的定时器。而`_lv_disp_refr_timer` 接口中就是做这个合并送显工作的，代码在 `lvgl/src/core/lv_refr.c`中，节选关键代码如下:

```c
void _lv_disp_refr_timer(lv_timer_t * tmr)
{
	// 1. 找到显示屏幕
	if(tmr) {
        disp_refr = tmr->user_data;
#if LV_USE_PERF_MONITOR == 0 && LV_USE_MEM_MONITOR == 0
        lv_timer_pause(tmr);
#endif
	} else {
        disp_refr = lv_disp_get_default();
    }
    // 2. 更新送显屏act_scr图层
    lv_obj_update_layout(disp_refr->act_scr);
    // 3. 更新上一个送显屏act_scr图层
    if(disp_refr->prev_scr) lv_obj_update_layout(disp_refr->prev_scr);
    // 4. 更新送显屏top_layer图层
    lv_obj_update_layout(disp_refr->top_layer);
    // 5. 更新送显屏sys_layer图层
    lv_obj_update_layout(disp_refr->sys_layer);
    // 6. 合并需要更新的区域
	lv_refr_join_area();
	lv_refr_areas();
	// 7. 发现刷新区域
	if(disp_refr->inv_p != 0) {
		if(disp_refr->driver->full_refresh) {
			// 8. 将合成图层送显
			draw_buf_flush(disp_refr);
		}
	}
	// 9. 回收此过程中生成的中间资源
	lv_mem_buf_free_all();
    _lv_font_clean_up_fmt_txt();
#if LV_DRAW_COMPLEX
    _lv_draw_mask_cleanup();
#endif
	// 10. 计算送显帧率/CPU等信息
#if LV_USE_PERF_MONITOR && LV_USE_LABEL
......
#endif
}
```

> **小白提问**: _找屏的代码里怎么会有一个 `lv_timer_pause` 接口调用？_  
> **答**：这是 LVGL 的两种送显功能的实现：一种是定时送显，一种是触发式送显。依据`lv_conf.h` 默认的配置，`LV_USE_PERF_MONITOR` 和 `LV_USE_MEM_MONITOR` 都是为`0`的，故默认 LVGL 是走的触发式送显的。当 UI 有更新时，会主动调用 `lv_timer_resume`接口来启动送显（对 UI 开发来讲，调用 `lv_obj_invalidate` 或者 `lv_obj_invalidate_area` 接口就会触发这一动作）。 而定时送显会增加程序非必要开销（没有 UI 更新时，也会往下执行去检测送显的），主要是调试时检测帧率用的，用于测试触发式送能否达到设置的帧率（测试帧率是你设置的帧率，说明定时器运行是有时间空闲的，应用不会有性能问题。测试帧率低于你设置的帧率，说明有其它定时器会影响送显流程，而导致出现卡顿等问题）。

> **小白提问**: _为啥要去更新上一个屏幕的 act_scr 图层？_  
> **答**：因为多屏切换时，有时候需要做屏幕动画的功能，所在在更新切换后的屏幕时，需要同步触发更新前一个屏幕的状态。

合并需要更新的区域有两个函数：`lv_refr_join_area` 和 `lv_refr_areas` 。先来看简单一点的`lv_refr_join_area`，如果区域 (join_from) 在区域 (join_in) 之上，就把它们合并了，可以用以下图形来表示(代码就自行查看吧，这些接口都是具体功能实现的了)：  
![[b72dfa079e0959e5593de207a462bc43_MD5.png]]  
而`lv_refr_areas` 的实现就比较复杂了。用一句话来解析就是把多个区域合并成一个出来。值的注意的是，`lv_refr_join_area` 函数只是计算了位置信息，并未做实际的图像更新，最新所有的更新都是在`lv_refr_areas`里来做的 (具体调用顺序：`lv_refr_areas`->`lv_refr_area`->`lv_refr_area_part`->`lv_draw_rect`)。

`draw_buf_flush`函数是用来送显的，它主要实现了三个功能：一是回调注册进来的`flush_cb`函数实现送显，二是管理双 buffer 切换逻辑，三是支持软旋转功能。节选关键代码如下：

```c
static void draw_buf_flush(lv_disp_t * disp)
{
    if(disp->driver->flush_cb) {
        /*Rotate the buffer to the display's native orientation if necessary*/
        if(disp->driver->rotated != LV_DISP_ROT_NONE && disp->driver->sw_rotate) {
        	// 旋转后送显
            draw_buf_rotate(draw_ctx->buf_area, draw_ctx->buf);
        }    
        else {
        	// 包裹了flush_cb函数的送显接口
            call_flush_cb(disp->driver, draw_ctx->buf_area, draw_ctx->buf);
        }    
    }
        /*If there are 2 buffers swap them. With direct mode swap only on the last area*/
        // 管理双buffer切换逻辑
    if(draw_buf->buf1 && draw_buf->buf2 && (!disp->driver->direct_mode || flushing_last)) {
        if(draw_buf->buf_act == draw_buf->buf1)
            draw_buf->buf_act = draw_buf->buf2;
        else 
            draw_buf->buf_act = draw_buf->buf1;
    } 
}
```

从合并的流程来看，LVGL 的实现并不是按图层的概念来做合并的，而是采用的区域的方式，所有无论有几个图层，最终却只是把需要更新的内容加进去即可。当组件有内容更新时，会将更新的内容信息放在`struct _lv_disp_t` 结构体中的 `inv_areas`字段中来，节选关键性字段如下 ：

```c
typedef struct _lv_disp_t {
    /** Invalidated (marked to redraw) areas*/
    lv_area_t inv_areas[LV_INV_BUF_SIZE]; // 记录需要更新的区域
    uint8_t inv_area_joined[LV_INV_BUF_SIZE];
    uint16_t inv_p;  // 记录需要更新区域的数量
} lv_disp_t;
```

**友情提示**：LV_INV_BUF_SIZE 是宏定义来的，也就是说如果 UI 做的过于复杂，同一帧超过个数量的区域需要更新，将会发生部分内容更新不了的情况。

最后一个问题，只调用了一个 `lv_obj_invalidate(disp->act_scr)`接口呢，一是 top_layer 和 sys_layer 暂未使用，二是有内容更新时，会将需要更新内容统一到`inv_areas`里来管理，也没有必要多次重复调用了。