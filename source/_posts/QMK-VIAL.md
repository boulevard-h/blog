---
title: QMK编译+Vial在线改键功能
date: 2022-10-24 08:14:00
categories: 客制化键盘
tags: [客制化键盘, DIY]
---

## 环境

QMK MSYS 0.18.0

环境配置不再赘述，msys集成度较高，git clone不出问题基本上就不会出问题。

## 下载源码

[Keyboard Firmware Builder](https://kbfirmware.com/)，参考丈二、苏达等人教程，配置好PIN、MAP等，到COMPILE中下载第二项：Download.zip，即可下载源码。

**但是要注意的是，在线固件网站的QMK版本很老，在0.18.0版本中编译不会通过，所以我们只在需要时复制其中的一些代码过去，整体模板并不适用。**

## 编写基本源码

### QMK基本代码结构

QMK MSYS的工程默认放置于`C:/用户/用户名/qmk_firmware`/中，为了方便，记为`~`

`~/.build/`中存放的是编译输出的固件，`~/keyboards/`中存储的是各种键盘的源码，主要在这个目录编写代码

在`~/keyboards/键盘名/`文件夹中，是该键盘对应的源码，如果你的键盘有多版本的话，可以在文件夹下创建各个版本的文件夹`~/keyboards/键盘名/rev_x`

进入到qmk工程目录以后，使用`qmk compile -kb 键盘名[/rev_x]`，即可在keyboards中自动找到并编译

在一个键盘基本的工程中，存在如下结构：

> │  键盘名.c
> │  键盘名.h
> │  config.h
> │  rules.mk
> │
> └─keymaps
>     ├─default       
>             keymap.c

## 基础文件内容编写

### rules.mk

规定编译选项：

``` makefile
# MCU name
MCU = atmega32u4

# Bootloader selection
BOOTLOADER = atmel-dfu

# Build Options
#   change yes to no to disable
#
LTO_ENABLE = yes
BOOTMAGIC_ENABLE = yes      # Enable Bootmagic Lite
EXTRAKEY_ENABLE = yes       # Audio control and System control
NKRO_ENABLE = yes           # Enable N-Key Rollover

MOUSEKEY_ENABLE = no        # Mouse keys
CONSOLE_ENABLE = no         # Console for debug
COMMAND_ENABLE = no         # Commands for debug and configuration
BACKLIGHT_ENABLE = no     # Enable keyboard backlight functionality
RGBLIGHT_ENABLE = no      # Enable keyboard RGB underglow
AUDIO_ENABLE = no           # Audio output
```

最开始两个MCU和BL选项，本次使用的是最简单的32u4。

LTO_ENABLE：压缩固件内容，编译时间稍微长一点，编译出来的固件内容小一点，一般开启。

后面的具体每一个对应什么参见QMK官方文档。

### config.h

规定键盘基本设置：

``` c
#pragma once

#include "config_common.h"

/* USB Device descriptor parameter */
#define VENDOR_ID       0xFEED
#define PRODUCT_ID      0x6060
#define DEVICE_VER      0x0001
#define MANUFACTURER    boulevard
#define PRODUCT         amx55

/* key matrix size */
#define MATRIX_ROWS 5
#define MATRIX_COLS 12

/* key matrix pins */
#define MATRIX_ROW_PINS { F0, F1, F4, F5, F6 }
#define MATRIX_COL_PINS { C7, C6, D7, D6, D5, D4, D3, D2, D1, D0, B7, B6 }
// 

/* COL2ROW or ROW2COL */
#define DIODE_DIRECTION ROW2COL

/* Set 0 if debouncing isn't needed */
#define DEBOUNCE 5
```

第一二行，照抄

**下面的几行就可以从下载的源码复制过来**，具体定义如下：

USB Device descriptor parameter：下面的五个Define是USB的ID和名字，自己设置就行

key matrix size：矩阵的行列数

key matrix pins：矩阵行、列引脚编号

DIODE_DIRECTION：二极管方向，写反了会按键没反应。有时候键盘连接显示名称但是按键没反应可以看看这个有没有反。

DEBOUNCE：消抖次数，一般设置为5就行

### 键盘名.h

定义LAYOUT：

``` c
#pragma once

#include "quantum.h"  

#define LAYOUT( \
	K000, K001, K002, K003, K004, K005, K006, K007, K008, K009, K010, K011, \
	K100, K101, K102, K103, K104, K105, K106, K107, K108, K109, K110, K111, \
	K200, K201, K202, K203, K204, K205, K206, K207, K208, K209,       K211, \
	K300,       K302, K303, K304, K305, K306, K307, K308, K309, K310, K311, \
	K400, K401, K402,       K404,       K406,       K408, K409, K410, K411  \
) { \
	{ K000,  K001,  K002,  K003,  K004,  K005,  K006,  K007,  K008,  K009,  K010,  K011 }, \
	{ K100,  K101,  K102,  K103,  K104,  K105,  K106,  K107,  K108,  K109,  K110,  K111 }, \
	{ K200,  K201,  K202,  K203,  K204,  K205,  K206,  K207,  K208,  K209,  KC_NO, K211 }, \
	{ K300,  KC_NO, K302,  K303,  K304,  K305,  K306,  K307,  K308,  K309,  K310,  K311 }, \
	{ K400,  K401,  K402,  KC_NO, K404,  KC_NO, K406,  KC_NO, K408,  K409,  K410,  K411 }  \
}
```

上面两行任然照抄，下面的`#define LAYOUT()`内容也是从下载的源码复制。

### 键盘名.c

只有一行，就是include一下上面的.h文件

``` c
#include "amx55.h"
```

### keymaps文件夹

存放的是默认按键配列，下面可以放很多个文件夹，名称是配列名。编译的时候可以通过`-km 配列名`选择编译的配列。最简单的版本只需要有一个default文件夹，文件夹下面只需要一个：`keymap.c`文件，内容如下：

``` c
#include QMK_KEYBOARD_H

const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {

	[0] = LAYOUT(
		KC_GRV, KC_1, KC_2, KC_3, KC_4, KC_5, KC_6, KC_7, KC_8, KC_9, KC_0, KC_SPC, 
		KC_TAB, KC_Q, KC_W, KC_E, KC_R, KC_T, KC_Y, KC_U, KC_I, KC_O, KC_P, KC_BSLS, 
		KC_CAPS, KC_A, KC_S, KC_D, KC_F, KC_G, KC_H, KC_J, KC_K, KC_L, KC_ENT, 
		KC_LSFT, KC_Z, KC_X, KC_C, KC_V, KC_B, KC_N, KC_M, KC_LSFT, KC_UP, KC_DEL, 
		KC_LCTL, KC_LGUI, KC_LALT, KC_SPC, KC_SPC, MO(1), KC_LEFT, KC_DOWN, KC_RGHT),

	[1] = LAYOUT(
		KC_F1, KC_F2, KC_F3, KC_F4, KC_F5, KC_F6, KC_F7, KC_F8, KC_F9, KC_F10, KC_F11, KC_F12, 
		KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_LBRC, KC_RBRC, KC_TRNS, 
		KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_SCLN, KC_QUOT, KC_TRNS, 
		MO(2), KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_COMM, KC_DOT, KC_SLSH, MO(2), KC_TRNS, KC_TRNS, 
		KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS),

        
	[2] = LAYOUT(
		KC_ESC, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, RESET, 
		KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, 
		KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, 
		KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, 
		KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS, KC_TRNS),

};
```

第一行的#include照抄，这个数组注意不能直接复制源码，和源码有所不同。

首先写好

``` c
const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {
    [0] = LAYOUT(),
    ...
    [n] = LAYOUT(),
}
```

默认配置有几层写几个LAYOUT就行，然后去下载的源码里边把LAYOUT复制过来。

## 编译固件

`qmk compile -kb 键盘名\[rev_x] -km keymap名`

编译成功以后刷入测试即可。

注意有的时候会有一个什么backslash报错，需要在报错的文件最后加一个空行即可...

## Vial配置

### via与vial介绍

via是客制化常用的改键方案了，老版的via需要下载软件、导入配置好的json文件。新版可以使用网页且不用导入json，但是暂时还不知道怎么写固件...

vial有网页和软件两个方式，都不需要导入json，写好固件以后连接即可识别，且不需要保存，即改即存，功能似乎也比via完善。和不需要json的via版本一致，最大的缺点是把json需要的信息量存入到了键盘mcu的flash中，导致固件大很多，flash不是很大的mcu（比如又贵又拉的32u4）可能有点吃不消。

### 支持vial的qmk编译环境搭建

在qmk msys中输入命令：

`qmk setup -H 分支目录 -b vial vial-kb/vial-qmk`

分支目录是你vial版本qmk工程文件放置的目录，然后会开始git clone和setup，和最开始的qmk环境搭建基本一致

然后cd到分支目录中，这个目录和qmk默认的工程目录几乎是一样的，把之前的keyboards中的键盘文件夹复制到这边，继续修改即可

### 支持vial的固件编写

在keymap中新建一个文件夹vial，把之前default中的keymap.c复制过去，然后还需要三个文件：

config.h、rules.mk、vial.json

下面介绍三个文件的配置

#### rules.mk

只有两行

``` makefile
VIA_ENABLE = yes
VIAL_ENABLE = yes
```

#### vial.json

``` json
{
    "lighting": "none",
    "matrix": {
        "rows": 5,
        "cols": 12
    },
    "layouts": {
        "keymap":
    }
}
```

lighting是灯光模式，方便在vial中设置灯光，我没有用灯，none

matrix中写入行列数

layouts中的keymap比较有趣，需要在常用的配列网站[Keyboard Layout Editor](http://www.keyboard-layout-editor.com/#/)中，使用自己的配列，把每个键上面的字改为:`row,col`，表示这个键所在的行列：

![image-20221024090736063](/images/QMK/image-20221024090736063.png)

然后再</> Raw data中到处json，这个json的内容复制到keymap后面去就行。

#### config.h

内容如下：

``` c
/* SPDX-License-Identifier: GPL-2.0-or-later */

#pragma once

#define VIAL_KEYBOARD_UID {0xDA, 0xDB, 0x70, 0x9F, 0xE7, 0x18, 0x63, 0x55}

#define VIAL_UNLOCK_COMBO_ROWS { 0, 2 }
#define VIAL_UNLOCK_COMBO_COLS { 0, 11 }
```

第一行，按惯例，照抄

第二行，UID的配置，这个不能自己乱配，需要在工程目录下执行python脚本

``` shell
python util/vial_generate_keyboard_uid.py
```

执行完成后会显示一个：

``` c
#define VIAL_KEYBOARD_UID {0xXX, 0xXX, 0xXX, 0xXX, 0xXX, 0xXX, 0xXX, 0xXX}
```

把生成的这个内容复制过去就行。

第三四行，定义了两个键的组合：

由于vial的权限比via高很多，可以执行一些危险操作，所以有的功能是被上锁的，执行这些功能之前，必须要按下两个固定的键解锁，这两行就是要设置这两个键。

这两行的坐标是竖着看的，上面这个代码的意思就是按下(0,0)、(2,11)两个键可以解锁vial的危险功能。

### 支持vial的固件编译

仍然在vial分支的qmk工程目录下面执行：

``` shell
qmk compile -kb 键盘名/[rev_x] -km vial
```

生成以后，刷入固件，访问网站[Vial Web](https://vial.rocks/)就可以在线改键。

### 进阶：Vial旋钮固件自定义

#### 开启QMK编码器映射功能

QMK旋钮固件有两种编写方式：编码器映射（Mapping）、回调（Callback）。Vial的目前版本只支持映射方式。

首先需要在`rules.mk`中打开编码器映射：

``` makefile
ENCODER_ENABLE = yes
ENCODER_MAP_ENABLE = yes
```

然后在`config.h`中配置旋钮引脚和解析度：

``` c
/* Encoder Setting */
#define ENCODERS_PAD_A { A10 }
#define ENCODERS_PAD_B { A8 }
#define ENCODER_RESOLUTION 4
```

再到`keymap.c`中支持开启编码器映射：

``` c
#if defined(ENCODER_MAP_ENABLE)
const uint16_t PROGMEM encoder_map[][NUM_ENCODERS][2] = {
    [_HOME] =   { ENCODER_CCW_CW(KC_1, KC_2) },
    [_FN2] =  { ENCODER_CCW_CW(KC_VOLD, KC_VOLU) },
};
#endif
```

映射支持多层多旋钮，这里以两层（HOME、FN2）一个旋钮为例。

到这里，就通过编码器映射方式启用了旋钮，功能应该是可以正常使用的。

#### 开启Vial对于旋钮改键支持

到配列编辑网站加入两个按键，代表旋钮的两个方向，按键内容如下：

![image-20221112130821772](/images/QMK/image-20221112130821772.png)

然后按照前面教程所说的，把这个网站生成的json复制到vial.json->layouts->keymap即可

再次编译刷写，进入vial就可以看到了。

## 参考教程

### 手把手视频教学

**32u4主控PCB绘制，在线固件生成：**B站/zf用户：丈二先生

 [【零基础】客制化键盘DIY系列教程（二）Keyboard Editor 编辑自己的配列~_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ch411x7J2/?spm_id_from=333.788)

**qmk本地代码修改、编译：**B站用户：HiryKun

[【QMK教程】从配置编译环境到实现RBG矩阵灯效 OLED屏幕动画 旋钮编码器功能 VIA改键 ARM移植的详细大教学，看完就会写QMK固件_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1pt4y1a7dG/?spm_id_from=333.880.my_history.page.click)

**vial固件编写：**B站用户：HiryKun

[【QMK教程】Vial的支持与配置，让你的键盘方便改键/改编码器/改灯效，无需选择JSON，比VIA更方便_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ft4y1t7vx/?spm_id_from=333.999.0.0)

### 官方文档

**qmk官方文档：**[QMK Firmware](https://docs.qmk.fm/#/)

**vial官方文档：**[Home - Vial](https://get.vial.today/)
