---
title: "【花雕学编程】Arduino LVGL 之事件驱动模型_arduino 事件驱动-CSDN博客"
source: "https://blog.csdn.net/weixin_41659040/article/details/143811270"
author:
published:
created: 2025-02-21
description: "文章浏览阅读1k次，点赞28次，收藏15次。LVGL提供了丰富的控件库，开发者可以通过lv_btn_create、lv_slider_create和lv_dropdown_create等函数创建控件，并通过lv_obj_set_size、lv_obj_align等函数进行配置，确保界面布局符合设计需求。5、创新：Arduino可以让你用电子的方式来表达你的创意和想象，你可以用Arduino来制作各种有趣和有用的项目，如机器人、智能家居、艺术装置等。例如，不同的触摸屏可能需要不同的驱动程序，开发者需要确保所用的硬件能够正常工作。_arduino 事件驱动"
tags:
  - "clippings"
---
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8defa84699397c20e0ea8c1ca698fa6e.gif#pic_center)

Arduino是一个开放源码的电子原型平台，它可以让你用简单的硬件和软件来创建各种互动的项目。Arduino的核心是一个[微控制器](https://so.csdn.net/so/search?q=%E5%BE%AE%E6%8E%A7%E5%88%B6%E5%99%A8&spm=1001.2101.3001.7020)板，它可以通过一系列的引脚来连接各种传感器、执行器、显示器等外部设备。Arduino的编程是基于C/C++语言的，你可以使用Arduino IDE（集成开发环境）来编写、编译和上传代码到Arduino板上。Arduino还有一个丰富的库和社区，你可以利用它们来扩展Arduino的功能和学习Arduino的知识。

Arduino的特点是：  
1、开放源码：Arduino的硬件和软件都是开放源码的，你可以自由地修改、复制和分享它们。  
2、易用：Arduino的硬件和软件都是为初学者和非专业人士设计的，你可以轻松地上手和使用它们。  
3、便宜：Arduino的硬件和软件都是非常经济的，你可以用很低的成本来实现你的想法。  
4、多样：Arduino有多种型号和版本，你可以根据你的需要和喜好来选择合适的Arduino板。  
5、创新：Arduino可以让你用电子的方式来表达你的创意和想象，你可以用Arduino来制作各种有趣和有用的项目，如机器人、智能家居、艺术装置等。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/925fd3f27ae443c55d204b6012c2c122.jpeg#pic_center)  
Arduino LVGL（Light and Versatile Graphics Library）是一个功能强大、轻量级的图形库，专为嵌入式系统设计，尤其适用于Arduino等微控制器平台。以下是对其主要特点、应用场景以及需要注意的事项的详细解释。

1、主要特点  
1）轻量级：  
LVGL是一个轻量级的图形库，适合资源有限的嵌入式系统。它在内存使用和处理速度上进行了优化，使其能够在低功耗设备上流畅运行。  
2）丰富的组件：  
LVGL提供了多种用户界面组件，比如按钮、滑块、图表、列表、图像等，开发者可以利用这些组件快速构建复杂的用户界面。  
3）高效的渲染能力：  
LVGL支持多种显示驱动程序，能够高效地渲染图形，支持抗锯齿、alpha混合等图形效果，从而提升界面的视觉质量。  
4）灵活的主题和样式：  
LVGL支持主题和样式的自定义，使得开发者可以根据需求轻松调整界面的外观，增加了界面的美观性和用户体验。  
5）多种输入设备支持：  
支持触摸屏、鼠标、键盘等多种输入设备，能够实现丰富的用户交互。  
6）多平台支持：  
LVGL不仅可以在Arduino上运行，还可以在多个嵌入式平台和操作系统上使用，包括ESP32、STM32、Linux等，具有良好的跨平台性。

2、应用场景  
1）嵌入式设备的用户界面：  
适用于需要图形用户界面的嵌入式设备，如智能家居控制面板、工业控制系统、医疗设备等。  
2）物联网（IoT）设备：  
在物联网应用中，LVGL可以用来构建设备的控制界面，例如智能传感器、网关设备的设置界面。  
3）便携式设备：  
适合用于便携式电子设备，如手持设备、智能手表等，因其轻量特性和高效性能能够满足电池供电的需求。  
4）教育和原型开发：  
适用于教育和原型开发环境，开发者可以快速实现用户界面，帮助学生和初学者理解图形界面的设计与实现。  
5）高级应用：  
可用于需要复杂图形和动画的应用，如游戏开发或多媒体应用，利用其渲染能力实现丰富的用户体验。

3、需要注意的事项  
1）内存管理：  
尽管LVGL是轻量级的，但在内存使用方面仍需谨慎。开发者需要合理管理内存，避免内存泄漏或溢出，特别是在资源受限的环境中。  
2）性能优化：  
在实现复杂界面时，需要对性能进行优化。例如，减少不必要的重绘，使用图像缓存等方法，提高界面的响应速度和流畅度。  
3）输入设备的兼容性：  
在选择输入设备时，要确保与LVGL的兼容性。例如，不同的触摸屏可能需要不同的驱动程序，开发者需要确保所用的硬件能够正常工作。  
4）学习曲线：  
LVGL虽然功能强大，但其使用和配置可能有一定的学习曲线。开发者需要花时间熟悉LVGL的API和工作流程，才能有效利用其功能。  
5）文档与社区支持：  
LVGL有较好的文档和社区支持，但在遇到问题时，开发者应积极查阅官方文档，参与社区讨论，以获得帮助和解决方案。

总结  
Arduino LVGL是一个强大的图形库，适合在资源有限的嵌入式系统上构建高效的用户界面。通过其丰富的组件和灵活的特性，开发者可以实现多种应用场景中的图形界面。然而，在使用过程中，合理管理内存和优化性能是成功的关键。通过充分利用LVGL的特性，开发者能够创建出美观且功能强大的用户界面。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/579623c84dea4ff8a0786b31668ca46c.jpeg#pic_center)  
Arduino LVGL之事件驱动模型  
LVGL（Light and Versatile Graphics Library）是一款开源的图形库，广泛应用于嵌入式系统和微控制器上，尤其适合Arduino等平台。事件驱动模型是LVGL的核心特性之一，它允许用户通过事件机制实现对用户输入和界面更新的响应。以下是对Arduino LVGL中事件驱动模型的详细解析，包括主要特点、应用场景及注意事项。

主要特点  
事件驱动机制：  
LVGL采用事件驱动模型，所有的用户输入（如触摸、按键等）和系统状态变化（如定时器触发）都会生成相应的事件，系统通过事件分发机制响应这些事件。  
解耦合设计：  
事件驱动模型使得用户界面逻辑与图形渲染逻辑解耦，开发者可以通过处理事件来更新界面，而不需要直接操作界面元素，提高了代码的可维护性和可扩展性。  
灵活的事件管理：  
LVGL支持多种类型的事件（如按下、释放、长按等），开发者可以为不同的界面元素注册相应的事件处理函数，以实现复杂的用户交互。  
高效的资源管理：  
通过事件驱动模型，LVGL能够在需要时动态更新屏幕内容，从而优化内存和处理器的使用，适合资源有限的嵌入式系统。  
良好的用户体验：  
事件驱动模型允许快速响应用户操作，提升了用户界面的交互性和流畅性，使得最终用户体验更佳。

应用场景  
嵌入式用户界面：  
在嵌入式设备的用户界面中，LVGL可以用于实现各种控件（按钮、滑块、列表等）的交互，适用于智能家居、工业控制等领域。  
物联网设备：  
在物联网设备中，通过LVGL的事件驱动模型，可以实现与用户的实时交互，如通过触摸屏控制设备状态、查看[传感器数据](https://so.csdn.net/so/search?q=%E4%BC%A0%E6%84%9F%E5%99%A8%E6%95%B0%E6%8D%AE&spm=1001.2101.3001.7020)等。  
智能仪表盘：  
在汽车和工业设备的智能仪表盘中，LVGL可用于显示实时数据和状态，并响应用户输入，增强用户体验。  
可穿戴设备：  
在智能手表和健康监测设备中，LVGL可以实现简洁而直观的用户界面，支持触摸操作和通知事件。  
教育和科研项目：  
LVGL适合在教育和科研项目中用作图形界面的实现工具，帮助学生和研究人员快速构建和测试交互式应用。

注意事项  
事件处理效率：  
在设计事件处理函数时，应注意代码的效率，避免复杂的计算和阻塞操作，以确保界面的流畅性。  
事件优先级管理：  
对于多个事件的处理，需合理设置事件的优先级，确保关键事件能够及时得到响应，避免延迟。  
内存管理：  
LVGL在资源管理方面非常高效，但在使用过程中仍需注意内存的分配和释放，避免内存泄漏和溢出。  
硬件兼容性：  
在不同的硬件平台上，LVGL的性能和效果可能会有所不同，需进行适当的测试和调整，以确保兼容性和稳定性。  
用户输入的防抖处理：  
在处理用户输入时，需考虑输入信号的抖动问题，适当的防抖处理能够提高用户交互的准确性。  
测试和调试：  
由于事件驱动模型的复杂性，开发过程中应进行充分的测试和调试，确保不同事件的处理逻辑正确无误，提升系统的可靠性。  
通过以上分析，可以看出，Arduino LVGL中的事件驱动模型为开发者提供了强大的图形用户界面构建能力，适合于各种嵌入式应用。但在实际应用中需注意多种因素，以确保系统的可靠性和高效性。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0af7e02d0b2d455b8b19fb0823a1c09f.jpeg#pic_center)  
1、按钮点击事件

```cpp
#include <lvgl.h>
#include <TFT_eSPI.h>

TFT_eSPI tft = TFT_eSPI(); // 创建TFT对象

void btn_event_callback(lv_event_t * e) {
    lv_obj_t * btn = lv_event_get_target(e);
    lv_label_set_text(lv_obj_get_child(btn, NULL), "Clicked!"); // 修改按钮文本
}

void setup() {
    lv_init();
    tft.begin();
    tft.setRotation(1);
    lvgl_port_init(); // 初始化LVGL端口

    lv_obj_t * btn = lv_btn_create(lv_scr_act()); // 创建按钮
    lv_obj_set_size(btn, 120, 50);
    lv_obj_align(btn, LV_ALIGN_CENTER, 0, 0);
    lv_obj_add_event_cb(btn, btn_event_callback, LV_EVENT_CLICKED, NULL); // 添加事件回调

    lv_obj_t * label = lv_label_create(btn); // 创建标签
    lv_label_set_text(label, "Click Me");
    lv_obj_center(label); // 标签居中
}

void loop() {
    lv_task_handler(); // 处理LVGL任务
    delay(5);
}

123456789101112131415161718192021222324252627282930
```

2、滑块值变化事件

```cpp
#include <lvgl.h>
#include <TFT_eSPI.h>

TFT_eSPI tft = TFT_eSPI(); // 创建TFT对象

void slider_event_callback(lv_event_t * e) {
    lv_obj_t * slider = lv_event_get_target(e);
    int value = lv_slider_get_value(slider);
    Serial.print("Slider Value: ");
    Serial.println(value);
}

void setup() {
    Serial.begin(115200);
    lv_init();
    tft.begin();
    tft.setRotation(1);
    lvgl_port_init(); // 初始化LVGL端口

    lv_obj_t * slider = lv_slider_create(lv_scr_act()); // 创建滑块
    lv_obj_set_width(slider, 200);
    lv_obj_align(slider, LV_ALIGN_CENTER, 0, 0);
    lv_obj_add_event_cb(slider, slider_event_callback, LV_EVENT_VALUE_CHANGED, NULL); // 添加事件回调
}

void loop() {
    lv_task_handler(); // 处理LVGL任务
    delay(5);
}

1234567891011121314151617181920212223242526272829
```

3、下拉列表选择事件

```cpp
#include <lvgl.h>
#include <TFT_eSPI.h>

TFT_eSPI tft = TFT_eSPI(); // 创建TFT对象

void dropdown_event_callback(lv_event_t * e) {
    lv_obj_t * dropdown = lv_event_get_target(e);
    const char * selected_item = lv_list_get_selected(btn);
    Serial.print("Selected: ");
    Serial.println(selected_item);
}

void setup() {
    Serial.begin(115200);
    lv_init();
    tft.begin();
    tft.setRotation(1);
    lvgl_port_init(); // 初始化LVGL端口

    lv_obj_t * dropdown = lv_dropdown_create(lv_scr_act()); // 创建下拉列表
    lv_dropdown_set_options(dropdown, "Option 1\nOption 2\nOption 3");
    lv_obj_align(dropdown, LV_ALIGN_CENTER, 0, 0);
    lv_obj_add_event_cb(dropdown, dropdown_event_callback, LV_EVENT_VALUE_CHANGED, NULL); // 添加事件回调
}

void loop() {
    lv_task_handler(); // 处理LVGL任务
    delay(5);
}

1234567891011121314151617181920212223242526272829
```

要点解读  
事件驱动模型：  
在LVGL中，用户界面的交互是通过事件驱动的。每个控件（如按钮、滑块、下拉列表）都可以注册事件回调，当用户执行某个操作时（如点击或滑动），相应的回调函数被触发。  
回调函数的使用：  
示例中使用了不同的回调函数（如btn\_event\_callback、slider\_event\_callback和dropdown\_event\_callback）来处理特定事件。这些回调函数可以根据具体需求实现自定义逻辑，如更新显示内容或记录状态。  
对象创建与配置：  
LVGL提供了丰富的控件库，开发者可以通过lv\_btn\_create、lv\_slider\_create和lv\_dropdown\_create等函数创建控件，并通过lv\_obj\_set\_size、lv\_obj\_align等函数进行配置，确保界面布局符合设计需求。  
与硬件的集成：  
所有示例都使用TFT\_eSPI库与显示屏进行交互。通过lvgl\_port\_init函数初始化LVGL与硬件的连接，确保图形库能够正确绘制图形和响应用户输入。  
实时任务处理：  
lv\_task\_handler()函数在主循环中被调用，以处理LVGL的任务和更新界面。这是实现流畅用户体验的关键步骤，确保界面能及时响应用户操作。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0e8f567d989c4663abe6d02dc525111d.jpeg#pic_center)  
4、按钮点击

```cpp
#include <lvgl.h>
#include <TFT_eSPI.h>

void setup() {
  lv_init();
  tft_init();
  lv_port_init();

  // 创建一个按钮
  lv_obj_t * btn = lv_btn_create(lv_scr_act()); // 在当前活动屏幕上创建按钮
  lv_obj_set_pos(btn, 50, 50); // 设置按钮位置
  lv_obj_set_size(btn, 100, 50); // 设置按钮大小

  // 创建一个标签，然后将其添加到按钮
  lv_obj_t * label = lv_label_create(btn);
  lv_label_set_text(label, "Click Me");

  // 为按钮设置点击事件回调函数
  lv_obj_add_event_cb(btn, btn_event_cb, LV_EVENT_CLICKED, NULL);
}

void loop() {
  lv_task_handler();
  delay(5);
}

// 按钮点击事件回调函数
void btn_event_cb(lv_event_t * e) {
  lv_event_code_t code = lv_event_get_code(e);
  if (code == LV_EVENT_CLICKED) {
    lv_obj_t * btn = lv_event_get_target(e);
    lv_label_set_text动态(btn, "Clicked!");
  }
}

12345678910111213141516171819202122232425262728293031323334
```

5、滑块值改变事件

```cpp
#include <lvgl.h>
#include <TFT_eSPI.h>

void setup() {
  lv_init();
  tft_init();
  lv_port_init();

  // 创建一个滑块
  lv_obj_t * slider = lv_slider_create(lv_scr_act());
  lv_obj_set_pos(slider, 20, 100);
  lv_obj_set_size(slider, 180, 50);

  // 为滑块设置值改变事件回调函数
  lv_obj_add_event_cb(slider, slider_event_cb, LV_EVENT_VALUE_CHANGED, NULL);
}

void loop() {
  lv_task_handler();
  delay(5);
}

// 滑块值改变事件回调函数
void slider_event_cb(lv_event_t * e) {
  lv_event_code_t code = lv_event_get_code(e);
  if (code == LV_EVENT_VALUE_CHANGED) {
    lv_obj_t * slider = lv_event_get_target(e);
    int16_t value = lv_slider_get_value(slider);
    Serial.print("Slider value changed to: ");
    Serial.println(value);
  }
}

1234567891011121314151617181920212223242526272829303132
```

6、列表项选择事件

```cpp
#include <lvgl.h>
#include <TFT_eSPI.h>

void setup() {
  lv_init();
  tft_init();
  lv_port_init();

  // 创建一个列表
  lv_obj_t * list = lv_list_create(lv_scr_act());
  lv_obj_set_size(list, 230, 150);
  lv_obj_set_pos(list, 10, 50);

  // 向列表中添加几个项
  lv_obj_t * btn;
  for (int i = 0; i < 5; i++) {
    btn = lv_list_add_btn(list, LV_SYMBOL_AUDIO, "Item" + String(i));
    lv_obj_add_event_cb(btn, list_event_cb, LV_EVENT_CLICKED, NULL);
  }
}

void loop() {
  lv_task_handler();
  delay(5);
}

// 列表项选择事件回调函数
void list_event_cb(lv_event_t * e) {
  lv_event_code_t code = lv_event_get_code(e);
  if (code == LV_EVENT_CLICKED) {
    lv_obj_t * item = lv_event_get_target(e);
    Serial.println("List item clicked");
  }
}

12345678910111213141516171819202122232425262728293031323334
```

要点解读  
事件驱动模型：  
LVGL支持事件驱动模型，允许用户通过回调函数响应各种事件，如点击、值改变等。  
事件回调函数：  
通过lv\_obj\_add\_event\_cb函数为控件添加事件回调，当事件发生时，指定的回调函数会被调用。  
事件类型：  
LVGL定义了多种事件类型，如LV\_EVENT\_CLICKED、LV\_EVENT\_VALUE\_CHANGED等，用户可以根据需要处理不同的事件。  
事件参数：  
事件回调函数可以通过lv\_event\_get\_xxx系列函数获取事件相关的参数，如事件源对象、事件代码等。  
实际应用：  
事件驱动模型使得LVGL应用能够响应用户的交互操作，实现动态的用户界面。通过以上案例，可以看到如何将事件处理逻辑集成到GUI控件中，实现交互式操作。

注意，以上案例只是为了拓展思路，仅供参考。它们可能有错误、不适用或者无法编译。您的硬件平台、使用场景和Arduino版本可能影响使用方法的选择。实际编程时，您要根据自己的硬件配置、使用场景和具体需求进行调整，并多次实际测试。您还要正确连接硬件，了解所用传感器和设备的规范和特性。涉及硬件操作的代码，您要在使用前确认引脚和电平等参数的正确性和安全性。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1be086f7ce4149d0ade62eb19f0a6090.jpeg#pic_center)