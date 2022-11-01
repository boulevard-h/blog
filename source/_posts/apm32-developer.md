---
title: APM32键盘开发板の开发记录
date: 2022-10-29 15:28:00
catagories: 客制化键盘
tags: [客制化键盘, 简易教程]
---

## 基本说明

本开发板采用`apm32f103`系列的`c8t6`或`cbt6`主控，目的是作为一个QMK固件学习的廉价开发板，其基本情况：

开发板外观如下

![8f71e3499f27e378391ac2a1cbdc555](\images\apm32dvlp\8f71e3499f27e378391ac2a1cbdc555.jpg)

### 关于主控

主控方面，有四个选择：`stm32/apm32 f103 c8t6/cbt6`

apm32是stm32的国产版，软硬件都兼容，也就是说在设计电路和写固件的时候直接按stm32弄就行。

c8t6和cbt6的区别主要是flash大小，也就是最大支持的固件大小，其中c8t6为`64k`、cbt6为`128k`，目前而言两者价格差别很小，所以建议选择后者。

文章使用的是`apm32 f103 cbt6`

### 关于电路设计

电路分为两个部分，左边是apm32开发板，包含1个apm32键盘最小系统、9个按键、1个旋钮、2个插线母座。

其中apm32开发板的硬件设计参考苏达酱的apm32最小系统板：

b站教程：[【苏达】还在用32u4？该换apm啦！apm主控键盘教程第一期（前期准备）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1FV4y1K7mm/?spm_id_from=333.788&vd_source=a42d90bcf7fcd7bedcf7188f6b5545c7)

立创开源工程：[apm32键盘最小系统 - 嘉立创EDA开源硬件平台 (oshwhub.com)](https://oshwhub.com/nimrodlord/apm32-jian-pan-zui-xiao-ji-tong)

在此基础上，添加了按键和旋钮，将一部分无用的引脚引出连接插线段子，便于后续功能开发。

右边的usb hub部分基本与该开源工程一致[【已验证】USB拓展坞 USB集线器 USBHub - 嘉立创EDA开源硬件平台 (oshwhub.com)](https://oshwhub.com/apana/usbhub)

该项目只用了SL2.1A芯片，支持四个USB 2.1输入，本项目只是在其基础上，去掉了一个USB输入，将其换成了一个FFP连接器，然后在开发板的Type-C输出也接入到另一个FFP连接器上，可以通过一根FFP软排线将左侧的开发板接入到右侧的HUB。

### 关于固件

基础固件参考了上述苏达的教程以及[【QMK教程】从配置编译环境到实现RBG矩阵灯效 OLED屏幕动画 旋钮编码器功能 VIA改键 ARM移植的详细大教学，看完就会写QMK固件_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1pt4y1a7dG/?spm_id_from=333.880.my_history.page.click)。

基础固件与之前的博客[QMK编译+Vial在线改键功能 | Boulevard's Blog (blog-boulevard.top)](https://blog-boulevard.top/2022/10/24/qmk-vial/)基本类似，只是更改了主控有关。

## Bootloader烧录

apm32出厂是不带dfu的，也就是说不能和Atmega32u4一样，焊接好板子以后连上USB即可使用qmk工具箱烧录。apm32和stm32都需要刷写好bootloader以后才能够用usb连接qmk工具箱烧录固件。

### 准备

1. 烧录器：支持对Arm Cortex M3设备进行SWD接口烧录的烧录器，ST-Link、CMSIS-DAP即可。我使用的是猛龙电子的CMSIS-DAP v2。
2. Keil5：软件方面，使用Keil进行烧录。Keil是常用的单片机开发软件，下载和安装都非常傻瓜。

### Bootloader准备

使用苏达视频中提供的BL，不过需要注意的是，keil只支持hex文件，苏达群里的是bin文件，需要进行转换。

### Keil配置

#### 下载Apm32设备包

在Keil官网的包下载页面[MDK5 Software Packs (keil.com)](https://www.keil.com/dd2/pack/#!#eula-container) > APEXMIC > APEX Microelectronics APM32F1软件包下载即可：

![image-20221029163444442](\images\apm32dvlp\image-20221029163444442.png)

#### 导入包到Keil

打开Keil，打开Pack Installer

![1667032525749](\images\apm32dvlp\1667032525749.png)

在Pack Installer菜单中找到File>Import，找到上面下载的包导入即可

#### 配置apm32烧录工程

新建一个工程，将Bootloader的hex文件放置到工程文件夹中：

![image-20221029164114604](\images\apm32dvlp\image-20221029164114604.png)

在Keil中，工程会有一个Target1：

![image-20221029164202675](\images\apm32dvlp\image-20221029164202675.png)

右键第一个，设置：

在Device中选择到对应的芯片

![image-20221029164235505](\images\apm32dvlp\image-20221029164235505.png)

在Output中，把Name of Executable改为BL的hex文件：

![image-20221029164313095](\images\apm32dvlp\image-20221029164313095.png)

再Device中选择你的烧录器，我这里选择CMSIS-DAP Debugger：

![image-20221029164424713](\images\apm32dvlp\image-20221029164424713.png)

然后在烧录器右边的Settings中继续设置烧录器：

在Debug中，选择你的CMSIS-DAP版本，这里我选择CMSIS-DAP v2；然后在Flash Download中设置如下，注意Programming Algorithm需要选择你芯片对应的，如果没有需要点Add加进来：

![image-20221029164602341](\images\apm32dvlp\image-20221029164602341.png)

### 烧录

在上述的Keil工程配置完毕以后，将烧录器连接好开发板的SWD接口，点击LOAD即可：

![image-20221029164735953](\images\apm32dvlp\image-20221029164735953.png)

### 固件烧录

在Bootloader烧录好以后，插上开发板的type-c，在`设备管理器>通用串行总线设备`中会看到一个`Maple 003`的设备，就说明上面的BL烧录没有问题了。

打开QMK Toolbox，可以看到设备已经被识别了，后续就可以和32u4一样，用QMK工具箱烧录QMK键盘固件了：

![image-20221029165051241](\images\apm32dvlp\image-20221029165051241.png)

## 固件开发过程

### rev0

#### 版本说明

rev0是一个**最基础的验证MCU电路的版本，只验证了键盘功能**

最基础的版本参考我的博客：[QMK编译+Vial在线改键功能 | Boulevard's Blog (blog-boulevard.top)](https://blog-boulevard.top/2022/10/24/qmk-vial/)

下面只描写apm32版本与atmega32u4版本不同的地方

#### 工程结构

![image-20221029165533531](\images\apm32dvlp\image-20221029165533531.png)

工程结构如下，对比32u4版本，只多了`chconf.h`、`halconf.h`、`mcuconf.h`三个文件，下面一一说明每个文件修改之处。

#### keymap、键盘名.c、键盘名.h、config.h

这三个文件不变，和32u4完全一样

#### chconf.h

照抄就行，不知道干啥的

``` c
#pragma once

#define CH_CFG_ST_TIMEDELTA 0

#define CH_CFG_USE_CONDVARS_TIMEOUT FALSE

#include_next <chconf.h>
```

#### halconf.h

第一行和最后一行照抄，中间两行是启用SPI和PWM总线，普通的键盘功能可以不需要。后续可能需要修改，比如OLED需要使用I2C总线。

``` c
#pragma once

// #define HAL_USE_PWM TRUE

// #define HAL_USE_SPI TRUE

#include_next <halconf.h>
```

#### mcuconf.h

暂时也只需要照抄，需要和上面规定的总线对应。

``` c
#pragma once

#include_next <mcuconf.h>

// #undef STM32_PWM_USE_TIM1
// #define STM32_PWM_USE_TIM1 TRUE
```

#### rules.mk

MCU和BL类型改为STM32对应的，下面这些设置在上次的博客已经说明了，一样。

``` makefile
# MCU name
MCU = STM32F103

# Bootloader selection
BOOTLOADER = stm32duino

LTO_ENABLE = yes
BOOTMAGIC_ENABLE ?= yes	# Virtual DIP switch configuration
MOUSEKEY_ENABLE ?= no	# Mouse keys
EXTRAKEY_ENABLE ?= yes	# Audio control and System control
CONSOLE_ENABLE ?= no	# Console for debug
COMMAND_ENABLE ?= no    # Commands for debug and configuration
SLEEP_LED_ENABLE ?= no  # Breathing sleep LED during USB suspend
NKRO_ENABLE ?= yes	    # USB Nkey Rollover
BACKLIGHT_ENABLE ?= no
RGBLIGHT_ENABLE ?= no
```

#### 问题备注

功能比较简单，没有其他问题，但是reset键不是很灵，不知道是啥原因。所以**在固件中为键盘设置一个reset的组合键**，方便进入BL模式刷机。

### rev0.5

#### 版本说明

rev0.5是一个添加了旋钮的版本，旋钮还有一些问题没有提及

#### 代码内容

添加旋钮需要修改如下内容

1. 在rules.mk中开启旋钮：

``` makefile
ENCODER_ENABLE = yes
```

2. 在config.h中设置旋钮：

第一二行，显而易见是设置旋钮的A、B脚连接的MCU引脚。

第三行设置的是ENCODER的分辨率（灵敏度），也就是编码器在每个止动之间记录的脉冲数。

``` c
/* Encoder Setting */
#define ENCODERS_PAD_A { A10 }
#define ENCODERS_PAD_B { A8 }
#define ENCODER_RESOLUTION 4
```

3. 在keymap.c中定义`encoder_update_user`：

分别定义了顺时针，逆时针的键码。注意，如果键码是音量加减等功能，需要把`tap_code`改为`tap_code(KC_COLU, 10)`这样。同时需要在`rules.mk`里面开启`EXTRAKEY_ENABLE = yes`

``` c
bool encoder_update_user(uint8_t index, bool clockwise) {
    if (clockwise) {
        tap_code(KC_1);
    } else {
        tap_code(KC_2);
    }
    return false;
}
```

#### 问题备注

有一个很奇怪的BUG，在我的电路设计中，我将旋钮的上面两个脚当作轴体接到了ROW0, COL3，将下面的A、B脚接到了主控的B0, B1脚。然而在排除旋钮、固件本身问题的情况以后，无论是按下还是旋转都没有反应，将旋钮上面两个脚飞线到ROW0,COL0以后就可以触发这个键。

在DEBUG很久以后，还是没有效果，于是我将另一个旋钮的A、B脚接到主控的A8、A9，C接地，其他不接。这个旋钮就可以正常工作了。

目前还未找到原因，旋钮本身是可以用的。不知道是MCU这两个脚的原因还是电路设计的原因，有空再去排除。

### rev1

#### 版本说明

rev1主要是写了OLED屏幕的demo，屏幕为128*32分辨率的SSD1306驱动OLED。

#### I2C基础

OLED屏幕需要**开启I2C总线**，下面简介一下I2C的一些基础知识。

使用I2C的OLED屏幕有四个脚：VCC、GND、SCL(有的叫SCK，都是SClock的意思)、SDA。

其中VCC理论上接入3.3V和5V都可以，但是为了避免烧掉，还是建议接3.3V。GND不必多说，接地。

SCL和SPA是I2C的两个脚，直接连接到MCU，但是需要注意的是这两个脚不能够随便接，需要**查阅主控说明书**，查看其**支持I2C的脚**。

一般而言，一个MCU可以有多对I2C的脚记为I2C1、I2C2...。**对于APM32F103而言，我们使用其I2C1，其SCL为B6、SDA为B7**。

#### 代码内容

##### 开启I2C

需要在与芯片相关的文件中开启I2C：

`chconf.h`不需要修改。

`halconf.h`需要开启I2C：

``` c
#pragma once

#include_next <halconf.h>

#undef HAL_USE_I2C
#define HAL_USE_I2C TRUE
```

`mcuconf.h`则需要选择I2C使用I2C1这根：

``` c
#pragma once

#include_next <mcuconf.h>

#undef STM32_I2C_USE_I2C1
#define STM32_I2C_USE_I2C1 TRUE
```

##### 设置I2C参数，OLED参数

在`config.h`中加入如下：

第一行，开启I2CD1。对应的还有I2CD2，这是I2C的两个版本v1、v2。其选择与主控型号有关，在qmk官方文档中给出了，常用的STM32 F1X F3X系列都是I2Cv1。

后面四行，设置I2C1的时钟速率、第二个我也不知道、后面两个是I2C1的两个Pin，前面说了是B6/B7（注意别写反）

最后五行，设置OLED的亮度，超时时间，滚动时间等显示参数。

``` c
/*OLED*/
#define I2C_DRIVER I2CD1
#define I2C1_CLOCK_SPEED 400000
#define I2C1_DUTY_CYCLE FAST_DUTY_CYCLE_2
#define I2C1_SCL_PIN B6
#define I2C1_SDA_PIN B7
#define OLED_BRIGHTNESS 255
#define OLED_TIMEOUT 80000
#define OLED_FADE_OUT
#define OLED_FADE_OUT_INTERVAL 5
#define OLED_SCROLL_TIMEOUT 40000
```

##### 内容代码

内容代码主要在keymap.c的函数`oled_task_user()`中：

下面这个实例是官方给的显示文字的实例，内容比较简单，显示不出汉字。

``` c
#ifdef OLED_ENABLE
bool oled_task_user(void) {
    // Host Keyboard Layer Status
    oled_write_P(PSTR("Layer: "), false);

    switch (get_highest_layer(layer_state)) {
        case _HOME:
            oled_write_P(PSTR("HOME\n"), false);
            break;
        case _FN2:
            oled_write_P(PSTR("FN\n"), false);
            break;
        default:
            // Or use the write_ln shortcut over adding '\n' to the end of your string
            oled_write_ln_P(PSTR("Undefined"), false);
    }

    // Host Keyboard LED Status
    led_t led_state = host_keyboard_led_state();
    oled_write_P(led_state.num_lock ? PSTR("NUM ") : PSTR("    "), false);
    oled_write_P(led_state.caps_lock ? PSTR("CAP ") : PSTR("    "), false);

    oled_write_P(PSTR("\n\nCCRzzz(- -)"), false);
    
    return false;
}
#endif
```

下面这个是显示LOGO的示例：

``` c
bool oled_task_user(void) {
        static const char PROGMEM qmk_logo[] = {
            0x80, 0x81, 0x82, 0x83, 0x84, 0x85, 0x86, 0x87, 0x88, 0x89, 0x8A, 				0x8B, 0x8C, 0x8D, 0x8E, 0x8F, 0x90, 0x91, 0x92, 0x93, 0x94,
            0xA0, 0xA1, 0xA2, 0xA3, 0xA4, 0xA5, 0xA6, 0xA7, 0xA8, 0xA9, 0xAA, 				0xAB, 0xAC, 0xAD, 0xAE, 0xAF, 0xB0, 0xB1, 0xB2, 0xB3, 0xB4,
            0xC0, 0xC1, 0xC2, 0xC3, 0xC4, 0xC5, 0xC6, 0xC7, 0xC8, 0xC9, 0xCA, 				0xCB, 0xCC, 0xCD, 0xCE, 0xCF, 0xD0, 0xD1, 0xD2, 0xD3, 0xD4, 0x00
        };


    oled_write_P(qmk_logo, false);
}
```

同时，我们可以在keymap.c同级文件夹中定义一个.h文件，在其中定义多个render_xxx函数显示不同内容，然后在keymap.c中include这个文件并进行逻辑调用，如在默认层和FN层显示不同内容，调用不同函数等。

#### 问题备注

播放动画时，需要把每一帧都存到Flash中，再加上可能的Vial等功能，很容易导致固件大小超出B8T6的Flash大小（64k），但是却小于CBT6的Flash（128k）。

但是我们在rules.mk定义MCU的时候，只填了`MCU = STM32F103`，并未标注MCU为什么版本，所以默认认为其是C8T6，导致固件超过64K的时候会报错。

所以需要把C8T6改为CBT6：

在qmk_firmware根目录中，有`builddefs`这个文件夹，点进去有很多mk文件，需要修改其中的两个：

1. bootloader.mk

找到这个文件的191行，把`x8`改为`xB`

![image-20221101153707975](\images\apm32dvlp\image-20221101153707975.png)

2. mcu_selection.mk

找到这个文件的第278行，同样`x8`改为`xB`

![image-20221101153800616](\images\apm32dvlp\image-20221101153800616.png)

这样的话，固件超过64k而小于128k时就还是可以编译了。当然，由于C8和CB在其他地方时没有区别的，所以不会导致编译出来的小于64k的固件用在C8T6上有问题。

### to be continued...

旋钮的debug、屏幕更高级的功能、电磁阀等...

