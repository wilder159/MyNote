# 1 LV_MEM_POOL_EXPAND_SIZE
## 1.1 **LV_MEM_POOL_EXPAND_SIZE 的作用**
1. **动态扩展内存池**  
    当LVGL内置的内存池（通过 `LV_MEM_SIZE` 初始分配）不足以满足应用需求时，内存池会按 `LV_MEM_POOL_EXPAND_SIZE` 定义的大小逐步扩展。此参数决定了每次扩展的增量大小。
2. **平衡内存效率与分配性能**  
    较大的扩展步长（如1KB）可减少频繁扩展的次数，提升内存分配效率，但可能浪费内存；较小的步长（如256B）更节省内存，但可能增加扩展频率。
3. **避免内存碎片化**  
    合理的扩展步长有助于减少内存碎片，确保后续内存分配的连续性。
---
## 1.2 **配置建议**
1. **典型值范围**
    - **嵌入式系统**：建议设置为 `512B~2KB`，根据内存总大小和应用复杂度调整。
    - **资源受限场景**：可设为 `256B`，但需注意频繁扩展可能影响实时性。
    - **复杂UI应用**：若控件和动画较多，建议设为 `1KB` 以上。
2. **示例配置（`lv_conf.h`）**
```c
 #define LV_MEM_SIZE         (32 * 1024)   // 初始内存池大小（32KB） 
 #define LV_MEM_POOL_EXPAND_SIZE 1024      // 每次扩展1KB`
```
# 2 LVGL v9 lv_conf.h 配置项速查表_lvgl 表格查询

| 配置项                                    | 含义                                                                |
| -------------------------------------- | ----------------------------------------------------------------- |
| `LV_COLOR_DEPTH`                       | 色彩深度：8 (A8)、16 (RGB565)、24 (RGB888)、32 (XRGB8888)                 |
| `LV_USE_STDLIB_MALLOC`                 | 使用的内存分配实现方式：LVGL 内置、标准 C 函数、MicroPython 实现、RT-Thread 实现、自定义实现     |
| `LV_USE_STDLIB_STRING`                 | 使用的字符串处理实现方式：LVGL 内置、标准 C 函数、MicroPython 实现、RT-Thread 实现、自定义实现    |
| `LV_USE_STDLIB_SPRINTF`                | 使用的格式化字符串处理实现方式：LVGL 内置、标准 C 函数、MicroPython 实现、RT-Thread 实现、自定义实现 |
| `LV_MEM_SIZE`                          | `lv_malloc` 可用的内存大小（单位：字节）                                        |
| `LV_MEM_POOL_EXPAND_SIZE`              | `lv_malloc` 的内存扩展大小（单位：字节）                                        |
| `LV_MEM_ADR`                           | 内存池的地址，0 表示未使用                                                    |
| `LV_DEF_REFR_PERIOD`                   | 默认显示刷新周期（单位：毫秒）                                                   |
| `LV_DPI_DEF`                           | 默认每英寸像素数，用于初始化默认大小                                                |
| `LV_USE_OS`                            | 选择使用的操作系统：无、PTHREAD、FreeRTOS、CMSIS-RTOS2、RT-Thread、Windows、自定义    |
| `LV_DRAW_BUF_STRIDE_ALIGN`             | 绘图缓冲区的步幅对齐字节数                                                     |
| `LV_DRAW_BUF_ALIGN`                    | 绘图缓冲区起始地址对齐字节数                                                    |
| `LV_USE_DRAW_SW`                       | 使用软件绘图                                                            |
| `LV_DRAW_SW_DRAW_UNIT_CNT`             | 软件绘图单元的数量                                                         |
| `LV_DRAW_SW_LAYER_SIMPLE_BUF_SIZE`     | 简单图层块的目标缓冲区大小                                                     |
| `LV_DRAW_SW_COMPLEX`                   | 使用复杂绘图                                                            |
| `LV_DRAW_SW_SHADOW_CACHE_SIZE`         | 阴影计算缓存大小                                                          |
| `LV_DRAW_SW_CIRCLE_CACHE_SIZE`         | 最大缓存圆形数据的数量                                                       |
| `LV_USE_DRAW_ARM2D`                    | 使用 ARM-2D 绘图（仅适用于 Cortex-M 设备）                                    |
| `LV_USE_DRAW_VGLITE`                   | 使用 NXP 的 VG-Lite GPU                                              |
| `LV_USE_VGLITE_BLIT_SPLIT`             | 启用分裂质量退化解决方案                                                      |
| `LV_USE_VGLITE_DRAW_ASYNC`             | 启用 VG-Lite 异步绘图                                                   |
| `LV_USE_VGLITE_ASSERT`                 | 启用 VG-Lite 断言                                                     |
| `LV_USE_DRAW_PXP`                      | 使用 NXP 的 PXP 绘图                                                   |
| `LV_USE_PXP_ASSERT`                    | 启用 PXP 断言                                                         |
| `LV_USE_DRAW_DAVE2D`                   | 使用 Renesas 的 Dave2D 绘图                                            |
| `LV_USE_DRAW_SDL`                      | 使用 SDL 绘图                                                         |
| `LV_USE_DRAW_VG_LITE`                  | 使用 VG-Lite 绘图                                                     |
| `LV_USE_LOG`                           | 启用日志模块                                                            |
| `LV_LOG_LEVEL`                         | 日志记录的级别：跟踪、信息、警告、错误、用户、无                                          |
| `LV_LOG_PRINTF`                        | 使用 `printf` 打印日志                                                  |
| `LV_LOG_USE_TIMESTAMP`                 | 打印时间戳                                                             |
| `LV_LOG_USE_FILE_LINE`                 | 打印文件和行号                                                           |
| `LV_USE_ASSERT_NULL`                   | 检查参数是否为 NULL                                                      |
| `LV_USE_ASSERT_MALLOC`                 | 检查内存是否成功分配                                                        |
| `LV_USE_ASSERT_STYLE`                  | 检查样式是否正确初始化                                                       |
| `LV_USE_ASSERT_MEM_INTEGRITY`          | 检查内存完整性                                                           |
| `LV_USE_ASSERT_OBJ`                    | 检查对象的类型和存在性                                                       |
| `LV_ASSERT_HANDLER_INCLUDE`            | 自定义断言处理程序头文件                                                      |
| `LV_ASSERT_HANDLER`                    | 自定义断言处理程序                                                         |
| `LV_USE_REFR_DEBUG`                    | 在重绘区域绘制随机颜色矩形                                                     |
| `LV_USE_LAYER_DEBUG`                   | 在 ARGB 层绘制红色覆盖，在 RGB 层绘制绿色覆盖                                      |
| `LV_USE_PARALLEL_DRAW_DEBUG`           | 绘制不同颜色的覆盖层以调试并行绘图任务                                               |
| `LV_ENABLE_GLOBAL_CUSTOM`              | 启用全局自定义                                                           |
| `LV_CACHE_DEF_SIZE`                    | 默认缓存大小                                                            |
| `LV_GRADIENT_MAX_STOPS`                | 渐变的最大停止数                                                          |
| `LV_COLOR_MIX_ROUND_OFS`               | 调整颜色混合函数的舍入                                                       |
| `LV_OBJ_STYLE_CACHE`                   | 添加 2 个 32 位变量以加快获取样式属性                                            |
| `LV_USE_OBJ_ID`                        | 添加 `id` 字段到 `lv_obj_t`                                            |
| `LV_USE_OBJ_ID_BUILTIN`                | 使用内置方法获取对象 ID                                                     |
| `LV_USE_OBJ_PROPERTY`                  | 使用对象属性设置 / 获取 API                                                 |
| `LV_USE_VG_LITE_THORVG`                | 启用 VG-Lite ThorVG                                                 |
| `LV_VG_LITE_THORVG_LVGL_BLEND_SUPPORT` | 启用 LVGL 的混合模式支持                                                   |
| `LV_VG_LITE_THORVG_YUV_SUPPORT`        | 启用 YUV 颜色格式支持                                                     |
| `LV_VG_LITE_THORVG_16PIXELS_ALIGN`     | 启用 16 像素对齐                                                        |
| `LV_VG_LITE_THORVG_THREAD_RENDER`      | 启用多线程渲染                                                           |
| `LV_BIG_ENDIAN_SYSTEM`                 | 大端系统设置为 1                                                         |
| `LV_ATTRIBUTE_TICK_INC`                | 自定义 `lv_tick_inc` 函数的属性                                           |
| `LV_ATTRIBUTE_TIMER_HANDLER`           | 自定义 `lv_timer_handler` 函数的属性                                      |
| `LV_ATTRIBUTE_FLUSH_READY`             | 自定义 `lv_display_flush_ready` 函数的属性                                |
| `LV_ATTRIBUTE_MEM_ALIGN_SIZE`          | 缓冲区所需的对齐大小                                                        |
| `LV_ATTRIBUTE_MEM_ALIGN`               | 内存对齐属性                                                            |
| `LV_ATTRIBUTE_LARGE_CONST`             | 标记大常量数组的属性                                                        |
| `LV_ATTRIBUTE_LARGE_RAM_ARRAY`         | 声明大数组在 RAM 中的编译器前缀                                                |
| `LV_ATTRIBUTE_FAST_MEM`                | 将性能关键函数放入更快的内存中                                                   |
| `LV_EXPORT_CONST_INT`                  | 将整数常量导出到绑定                                                        |
| `LV_ATTRIBUTE_EXTERN_DATA`             | 为所有全局外部数据添加前缀                                                     |
| `LV_USE_FLOAT`                         | 使用 `float` 作为 `lv_value_precise_t`                                |
| `LV_FONT_MONTSERRAT_8`                 | 使用 Montserrat 8 字体                                                |
| `LV_FONT_MONTSERRAT_10`                | 使用 Montserrat 10 字体                                               |
| `LV_FONT_MONTSERRAT_12`                | 使用 Montserrat 12 字体                                               |
| `LV_FONT_MONTSERRAT_14`                | 使用 Montserrat 14 字体                                               |
| `LV_FONT_MONTSERRAT_16`                | 使用 Montserrat 16 字体                                               |
| `LV_FONT_MONTSERRAT_18`                | 使用 Montserrat 18 字体                                               |
| `LV_FONT_MONTSERRAT_20`                | 使用 Montserrat 20 字体                                               |
| `LV_FONT_MONTSERRAT_22`                | 使用 Montserrat 22 字体                                               |
| `LV_FONT_MONTSERRAT_24`                | 使用 Montserrat 24 字体                                               |
| `LV_FONT_MONTSERRAT_26`                | 使用 Montserrat 26 字体                                               |
| `LV_FONT_MONTSERRAT_28`                | 使用 Montserrat 28 字体                                               |
| `LV_FONT_MONTSERRAT_30`                | 使用 Montserrat 30 字体                                               |
| `LV_FONT_MONTSERRAT_32`                | 使用 Montserrat 32 字体                                               |
| `LV_FONT_MONTSERRAT_34`                | 使用 Montserrat 34 字体                                               |
| `LV_FONT_MONTSERRAT_36`                | 使用 Montserrat 36 字体                                               |
| `LV_FONT_MONTSERRAT_38`                | 使用 Montserrat 38 字体                                               |
| `LV_FONT_MONTSERRAT_40`                | 使用 Montserrat 40 字体                                               |
| `LV_FONT_MONTSERRAT_42`                | 使用 Montserrat 42 字体                                               |
| `LV_FONT_MONTSERRAT_44`                | 使用 Montserrat 44 字体                                               |
| `LV_FONT_MONTSERRAT_46`                | 使用 Montserrat 46 字体                                               |
| `LV_FONT_MONTSERRAT_48`                | 使用 Montserrat 48 字体                                               |
| `LV_FONT_MONTSERRAT_28_COMPRESSED`     | 使用 Montserrat 28 压缩字体                                             |
| `LV_FONT_DEJAVU_16_PERSIAN_HEBREW`     | 使用 DejaVu 16 字体，支持波斯语和希伯来语                                        |
| `LV_FONT_SIMSUN_16_CJK`                | 使用 SimSun 16 字体，支持 CJK 字符                                         |
| `LV_FONT_UNSCII_8`                     | 使用 UnscII 8 像素字体                                                  |
| `LV_FONTUNSCII_16`                     | 使用 UnscII 16 像素字体                                                 |
| `LV_FONT_CUSTOM_DECLARE`               | 声明自定义字体                                                           |
| `LV_FONT_DEFAULT`                      | 设置默认字体                                                            |
| `LV_FONT_FMT_TXT_LARGE`                | 启用大字体支持                                                           |
| `LV_USE_FONT_COMPRESSED`               | 启用压缩字体支持                                                          |
| `LV_USE_FONT_PLACEHOLDER`              | 启用字体占位符绘制                                                         |
| `LV_TXT_ENC`                           | 字符编码：UTF-8 或 ASCII                                                |
| `LV_TXT_BREAK_CHARS`                   | 可以换行的字符                                                           |
| `LV_TXT_LINE_BREAK_LONG_LEN`           | 长单词换行长度                                                           |
| `LV_TXT_LINE_BREAK_LONG_PRE_MIN_LEN`   | 长单词换行前的最小字符数                                                      |
| `LV_TXT_LINE_BREAK_LONG_POST_MIN_LEN`  | 长单词换行后的最小字符数                                                      |
| `LV_USE_BIDI`                          | 启用双向文本支持                                                          |
| `LV_BIDI_BASE_DIR_DEF`                 | 双向文本的默认方向                                                         |
| `LV_USE_ARABIC_PERSIAN_CHARS`          | 启用阿拉伯语 / 波斯语字符处理                                                  |
| `LV_WIDGETS_HAS_DEFAULT_VALUE`         | 小部件是否有默认值                                                         |
| `LV_USE_ANIMIMG`                       | 启用动画图像小部件                                                         |
| `LV_USE_ARC`                           | 启用弧形小部件                                                           |
| `LV_USE_BAR`                           | 启用条形小部件                                                           |
| `LV_USE_BUTTON`                        | 启用按钮小部件                                                           |
| `LV_USE_BUTTONMATRIX`                  | 启用按钮矩阵小部件                                                         |
| `LV_USE_CALENDAR`                      | 启用日历小部件                                                           |
| `LV_CALENDAR_WEEK_STARTS_MONDAY`       | 日历周从周一开始                                                          |
| `LV_USE_CALENDAR_HEADER_ARROW`         | 启用日历头部箭头                                                          |
| `LV_USE_CALENDAR_HEADER_DROPDOWN`      | 启用日历头部下拉菜单                                                        |
| `LV_USE_CANVAS`                        | 启用画布小部件                                                           |
| `LV_USE_CHART`                         | 启用图表小部件                                                           |
| `LV_USE_CHECKBOX`                      | 启用复选框小部件                                                          |
| `LV_USE_DROPDOWN`                      | 启用下拉菜单小部件                                                         |
| `LV_USE_IMAGE`                         | 启用图像小部件                                                           |
| `LV_USE_IMAGEBUTTON`                   | 启用图像按钮小部件                                                         |
| `LV_USE_KEYBOARD`                      | 启用键盘小部件                                                           |
| `LV_USE_LABEL`                         | 启用标签小部件                                                           |
| `LV_LABEL_TEXT_SELECTION`              | 启用标签文本选择                                                          |
| `LV_LABEL_LONG_TXT_HINT`               | 启用长文本提示                                                           |
| `LV_LABEL_WAIT_CHAR_COUNT`             | 标签等待字符计数                                                          |
| `LV_USE_LED`                           | 启用 LED 小部件                                                        |
| `LV_USE_LINE`                          | 启用线条小部件                                                           |
| `LV_USE_LIST`                          | 启用列表小部件                                                           |
| `LV_USE_MENU`                          | 启用菜单小部件                                                           |
| `LV_USE_MSGBOX`                        | 启用消息框小部件                                                          |
| `LV_USE_ROLLER`                        | 启用滚动选择器小部件                                                        |
| `LV_USE_SCALE`                         | 启用刻度小部件                                                           |
| `LV_USE_SLIDER`                        | 启用滑块小部件                                                           |
| `LV_USE_SPAN`                          | 启用跨度小部件                                                           |
| `LV_SPAN_SNIPPET_STACK_SIZE`           | 跨度描述符堆栈大小                                                         |
| `LV_USE_SPINBOX`                       | 启用旋转框小部件                                                          |
| `LV_USE_SPINNER`                       | 启用旋转器小部件                                                          |
| `LV_USE_SWITCH`                        | 启用开关小部件                                                           |
| `LV_USE_TEXTAREA`                      | 启用文本区域小部件                                                         |
| `LV_TEXTAREA_DEF_PWD_SHOW_TIME`        | 默认密码显示时间                                                          |
| `LV_USE_TABLE`                         | 启用表格小部件                                                           |
| `LV_USE_TABVIEW`                       | 启用标签视图小部件                                                         |
| `LV_USE_TILEVIEW`                      | 启用平铺视图小部件                                                         |
| `LV_USE_WIN`                           | 启用窗口小部件                                                           |
| `LV_USE_THEME_DEFAULT`                 | 启用默认主题                                                            |
| `LV_THEME_DEFAULT_DARK`                | 默认主题的暗模式                                                          |
| `LV_THEME_DEFAULT_GROW`                | 启用按压时增长效果                                                         |
| `LV_THEME_DEFAULT_TRANSITION_TIME`     | 默认过渡时间                                                            |
| `LV_USE_THEME_SIMPLE`                  | 启用简单主题                                                            |
| `LV_USE_THEME_MONO`                    | 启用单色主题                                                            |
| `LV_USE_FLEX`                          | 启用类似 CSS 的 Flexbox 布局                                             |
| `LV_USE_GRID`                          | 启用类似 CSS 的 Grid 布局                                                |
| `LV_USE_FS_STDIO`                      | 启用标准文件系统接口                                                        |
| `LV_FS_STDIO_LETTER`                   | 文件系统驱动字母                                                          |
| `LV_FS_STDIO_PATH`                     | 工作目录路径                                                            |
| `LV_FS_STDIO_CACHE_SIZE`               | 缓存大小                                                              |
| `LV_USE_FS_POSIX`                      | 启用 POSIX 文件系统接口                                                   |
| `LV_FS_POSIX_LETTER`                   | POSIX 文件系统驱动字母                                                    |
| `LV_FS_POSIX_PATH`                     | POSIX 工作目录路径                                                      |
| `LV_FS_POSIX_CACHE_SIZE`               | POSIX 缓存大小                                                        |
| `LV_USE_FS_WIN32`                      | 启用 Win32 文件系统接口                                                   |
| `LV_FS_WIN32_LETTER`                   | Win32 文件系统驱动字母                                                    |
| `LV_FS_WIN32_PATH`                     | Win32 工作目录路径                                                      |
| `LV_FS_WIN32_CACHE_SIZE`               | Win32 缓存大小                                                        |
| `LV_USE_FS_FATFS`                      | 启用 FATFS 文件系统接口                                                   |
| `LV_FS_FATFS_LETTER`                   | FATFS 文件系统驱动字母                                                    |
| `LV_FS_FATFS_CACHE_SIZE`               | FATFS 缓存大小                                                        |
| `LV_USE_FS_MEMFS`                      | 启用内存文件系统                                                          |
| `LV_FS_MEMFS_LETTER`                   | 内存文件系统驱动字母                                                        |
| `LV_USE_LODEPNG`                       | 启用 LODEPNG 解码库                                                    |
| `LV_USE_LIBPNG`                        | 启用 libpng 解码库                                                     |
| `LV_USE_BMP`                           | 启用 BMP 解码库                                                        |
| `LV_USE_TJPGD`                         | 启用 JPG 和分割 JPG 解码库                                                |
| `LV_USE_LIBJPEG_TURBO`                 | 启用 libjpeg-turbo 解码库                                              |
| `LV_USE_GIF`                           | 启用 GIF 解码库                                                        |
| `LV_GIF_CACHE_DECODE_DATA`             | 启用 GIF 解码缓存                                                       |
| `LV_BIN_DECODER_RAM_LOAD`              | 解码二进制图像到 RAM                                                      |
| `LV_USE_RLE`                           | 启用 RLE 解码库                                                        |
| `LV_USE_QRCODE`                        | 启用二维码库                                                            |
| `LV_USE_BARCODE`                       | 启用条形码库                                                            |
| `LV_USE_FREETYPE`                      | 启用 FreeType 字体库                                                   |
| `LV_FREETYPE_CACHE_SIZE`               | FreeType 缓存大小                                                     |
| `LV_FREETYPE_USE_LVGL_PORT`            | 让 FreeType 使用 LVGL 内存和文件端口                                        |
| `LV_FREETYPE_CACHE_FT_FACES`           | 最大缓存的 FT_Face 对象数量                                                |
| `LV_FREETYPE_CACHE_FT_SIZES`           | 最大缓存的 FT_Size 对象数量                                                |
| `LV_FREETYPE_CACHE_FT_GLYPH_CNT`       | 最大缓存的 FT_Glyph 对象数量                                               |
| `LV_USE_TINY_TTF`                      | 启用 Tiny TTF 解码器                                                   |
| `LV_TINY_TTF_FILE_SUPPORT`             | 启用 TTF 文件加载支持                                                     |
| `LV_USE_RLOTTIE`                       | 启用 Rlottie 动画库                                                    |
| `LV_USE_VECTOR_GRAPHIC`                | 启用矢量图形 API                                                        |
| `LV_USE_THORVG_INTERNAL`               | 启用内部 ThorVG 矢量图形库                                                 |
| `LV_USE_THORVG_EXTERNAL`               | 启用外部 ThorVG 矢量图形库                                                 |
| `LV_USE_LZ4`                           | 启用 LZ4 压缩 / 解压库                                                   |
| `LV_USE_LZ4_INTERNAL`                  | 使用 LVGL 内置的 LZ4 库                                                 |
| `LV_USE_LZ4_EXTERNAL`                  | 使用外部 LZ4 库                                                        |
| `LV_USE_FFMPEG`                        | 启用 FFmpeg 解码和视频播放                                                 |
| `LV_FFMPEG_DUMP_FORMAT`                | 将输入信息转储到 stderr                                                   |
| `LV_USE_SNAPSHOT`                      | 启用对象快照 API                                                        |
| `LV_USE_SYSMON`                        | 启用系统监控组件                                                          |
| `LV_USE_PERF_MONITOR`                  | 显示 CPU 使用率和 FPS 计数                                                |
| `LV_USE_PERF_MONITOR_POS`              | 性能监视器位置                                                           |
| `LV_USE_PERF_MONITOR_LOG_MODE`         | 性能监视器日志模式                                                         |
| `LV_USE_MEM_MONITOR`                   | 显示已用内存和内存碎片                                                       |
| `LV_USE_MEM_MONITOR_POS`               | 内存监视器位置                                                           |
| `LV_USE_PROFILER`                      | 启用运行时性能分析器                                                        |
| `LV_USE_PROFILER_BUILTIN`              | 启用内置分析器                                                           |
| `LV_PROFILER_BUILTIN_BUF_SIZE`         | 内置分析器跟踪缓冲区大小                                                      |
| `LV_PROFILER_INCLUDE`                  | 分析器头文件                                                            |
| `LV_PROFILER_BEGIN`                    | 分析器起始点函数                                                          |
| `LV_PROFILER_END`                      | 分析器结束点函数                                                          |
| `LV_PROFILER_BEGIN_TAG`                | 带有自定义标签的分析器起始点函数                                                  |
| `LV_PROFILER_END_TAG`                  | 带有自定义标签的分析器结束点函数                                                  |
| `LV_USE_MONKEY`                        | 启用猴子测试                                                            |
| `LV_USE_GRIDNAV`                       | 启用网格导航                                                            |
| `LV_USE_FRAGMENT`                      | 启用对象片段                                                            |
| `LV_USE_IMGFONT`                       | 支持将图像作为字体                                                         |
| `LV_IMGFONT_PATH_MAX_LEN`              | 图像字体路径的最大长度                                                       |
| `LV_IMGFONT_USE_IMAGE_CACHE_HEADER`    | 使用图像缓存缓冲区                                                         |
| `LV_USE_OBSERVER`                      | 启用观察者模式                                                           |
| `LV_USE_IME_PINYIN`                    | 启用拼音输入法                                                           |
| `LV_IME_PINYIN_USE_DEFAULT_DICT`       | 使用默认字典                                                            |
| `LV_IME_PINYIN_CAND_TEXT_NUM`          | 候选面板显示的最大数量                                                       |
| `LV_IME_PINYIN_USE_K9_MODE`            | 启用 9 键输入模式                                                        |
| `LV_IME_PINYIN_K9_CAND_TEXT_NUM`       | 9 键输入模式下候选面板显示的最大数量                                               |
| `LV_USE_FILE_EXPLORER`                 | 启用文件浏览器                                                           |
| `LV_FILE_EXPLORER_PATH_MAX_LEN`        | 文件路径的最大长度                                                         |
| `LV_FILE_EXPLORER_QUICK_ACCESS`        | 启用快速访问栏                                                           |
| `LV_USE_SDL`                           | 使用 SDL 打开窗口并处理鼠标和键盘                                               |
| `LV_SDL_INCLUDE_PATH`                  | SDL 包含路径                                                          |
| `LV_SDL_RENDER_MODE`                   | SDL 渲染模式                                                          |
| `LV_SDL_BUF_COUNT`                     | SDL 缓冲区数量                                                         |
| `LV_SDL_FULLSCREEN`                    | SDL 全屏模式                                                          |
| `LV_SDL_DIRECT_EXIT`                   | 关闭所有 SDL 窗口时退出应用程序                                                |
| `LV_USE_X11`                           | 使用 X11 打开窗口并处理鼠标和键盘                                               |
| `LV_X11_DIRECT_EXIT`                   | 关闭所有 X11 窗口时退出应用程序                                                |
| `LV_X11_DOUBLE_BUFFER`                 | 使用双缓冲进行渲染                                                         |
| `LV_X11_RENDER_MODE_PARTIAL`           | 部分渲染模式                                                            |
| `LV_X11_RENDER_MODE_DIRECT`            | 直接渲染模式                                                            |
| `LV_X11_RENDER_MODE_FULL`              | 完全渲染模式                                                            |
| `LV_USE_LINUX_FBDEV`                   | 使用 Linux Framebuffer 驱动                                           |
| `LV_LINUX_FBDEV_BSD`                   | BSD 系统的 Framebuffer 驱动                                            |
| `LV_LINUX_FBDEV_RENDER_MODE`           | Framebuffer 渲染模式                                                  |
| `LV_LINUX_FBDEV_BUFFER_COUNT`          | Framebuffer 缓冲区数量                                                 |
| `LV_LINUX_FBDEV_BUFFER_SIZE`           | Framebuffer 缓冲区大小                                                 |
| `LV_USE_NUTTX`                         | 使用 Nuttx 打开窗口并处理触摸屏                                               |
| `LV_USE_NUTTX_LIBUV`                   | 使用 Nuttx 的 libuv                                                  |
| `LV_USE_NUTTX_CUSTOM_INIT`             | 使用 Nuttx 自定义初始化 API                                               |
| `LV_USE_NUTTX_LCD`                     | 使用 Nuttx 的 LCD 驱动                                                 |
| `LV_NUTTX_LCD_BUFFER_COUNT`            | LCD 缓冲区数量                                                         |
| `LV_NUTTX_LCD_BUFFER_SIZE`             | LCD 缓冲区大小                                                         |
| `LV_USE_NUTTX_TOUCHSCREEN`             | 使用 Nuttx 的触摸屏驱动                                                   |
| `LV_USE_LINUX_DRM`                     | 使用 Linux DRM 驱动                                                   |
| `LV_USE_TFT_ESPI`                      | 使用 TFT_eSPI 接口                                                    |
| `LV_USE_EVDEV`                         | 使用 evdev 输入设备驱动                                                   |
| `LV_USE_ST7735`                        | 使用 ST7735 LCD 驱动                                                  |
| `LV_USE_ST7789`                        | 使用 ST7789 LCD 驱动                                                  |
| `LV_USE_ST7796`                        | 使用 ST7796 LCD 驱动                                                  |
| `LV_USE_ILI9341`                       | 使用 ILI9341 LCD 驱动                                                 |
| `LV_USE_GENERIC_MIPI`                  | 使用通用 MIPI 驱动                                                      |
| `LV_USE_WINDOWS`                       | 使用 LVGL Windows 后端                                                |
| `LV_BUILD_EXAMPLES`                    | 启用示例代码                                                            |
| `LV_USE_DEMO_WIDGETS`                  | 启用小部件演示                                                           |
| `LV_DEMO_WIDGETS_SLIDESHOW`            | 小部件演示的幻灯片模式                                                       |
| `LV_USE_DEMO_KEYPAD_AND_ENCODER`       | 启用键盘和编码器演示                                                        |
| `LV_USE_DEMO_BENCHMARK`                | 启用基准测试演示                                                          |
| `LV_USE_DEMO_RENDER`                   | 启用绘图测试演示                                                          |
| `LV_USE_DEMO_STRESS`                   | 启用 LVGL 压力测试演示                                                    |
| `LV_USE_DEMO_MUSIC`                    | 启用音乐播放器演示                                                         |
| `LV_DEMO_MUSIC_SQUARE`                 | 方形布局的音乐播放器演示                                                      |
| `LV_DEMO_MUSIC_LANDSCAPE`              | 横向布局的音乐播放器演示                                                      |
| `LV_DEMO_MUSIC_ROUND`                  | 圆形布局的音乐播放器演示                                                      |
| `LV_DEMO_MUSIC_LARGE`                  | 大屏幕音乐播放器演示                                                        |
| `LV_DEMO_MUSIC_AUTO_PLAY`              | 自动播放音乐                                                            |
| `LV_USE_DEMO_FLEX_LAYOUT`              | 启用 Flex 布局演示                                                      |
| `LV_USE_DEMO_MULTILANG`                | 启用多语言演示                                                           |
| `LV_USE_DEMO_TRANSFORM`                | 启用小部件变换演示                                                         |
| `LV_USE_DEMO_SCROLL`                   | 启用滚动设置演示                                                          |
| `LV_USE_DEMO_VECTOR_GRAPHIC`           | 启用矢量图形演示                                                          |
