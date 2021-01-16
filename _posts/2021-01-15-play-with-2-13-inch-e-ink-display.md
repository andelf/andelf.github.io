---
layout: post
title: Play with 2.13 inch E-Ink display
date: 2021-01-15 01:24 +0800
categories: blog
image: https://user-images.githubusercontent.com/72891/104635824-6825c900-56dd-11eb-8caa-d5eb00d11f70.jpg
author: andelf
tags:
  - embedded
  - epd
usemathjax: true
toc: true
---

故事从买屏幕说起。

无聊逛咸鱼，发现有便宜的电子墨水屏，这玩意正常价格大几十，而咸鱼一片 2.13 寸模块只需要 15 块钱人民币。

是的，还等啥，先来几片凑个包邮。

了解到之所以这么便宜，是因为实际上是拆机屏，拆的是电子价签。来自一茬又一茬倒闭的超市和便利店。

## 背景

电子墨水屏，也叫 E-ink, 墨水屏（瓶），电子纸，也简写为 EPD(Electronic Paper Display)。

和你用来压泡面的 Kindle 的屏幕是一个东西。只不过你的 Kindle 屏幕更大素质更高，而超市价签要小很多，相对低成本。

![E-Ink technology](https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Electronic_paper_%28Side_view_of_Electrophoretic_display%29_in_svg.svg/2560px-Electronic_paper_%28Side_view_of_Electrophoretic_display%29_in_svg.svg.png)

如上来自维基，原理一目了然。

不需要扯什么 gate 什么的，每个像素是一个小胶囊，顶部是公共 $$V_{com}$$ 电压，透明。
底面是驱动芯片在屏幕每一行每一列像素的输出电压，可正可负（相对于 $$V_{com}$$），胶囊内部是对电场方向有反应的带电颜色微粒。
不同电压不同时长作用下，胶囊顶部上的微粒分布情况不同，肉眼看到的像素深浅就不同。

屏幕的驱动芯片，和我们常见的 IC 芯片那种黑色块带引脚是不同的，屏幕驱动芯片一般和屏幕一起封装，
对应屏幕的行列有输出。对外暴露接口，物理上一般是排线。

## 准备开搞

屏幕模块到手。显示“微雪电子”，那当然是不可能的，这只是因为默认用了微雪的示例代码出厂测试。
墨水屏的特点就是断电画面驻留，也可以说保持显示状态不需要供电。

![Boards](https://user-images.githubusercontent.com/72891/104638297-f5b6e800-56e0-11eb-93d6-0bc0fde0d475.jpg)

合影是一只凑单的 STM32F4 板子（本例未用到），一只 ESP8266（就准备用它来驱动屏幕了），和屏幕模块（主角）。

一共需要八根线驱动。熟悉的 SPI + CS + DC 式，和多数 SPI 接口的 LCD / TFT-LCD 屏幕接口类似。

不同的是多了一个 BUSY pin，这是墨水屏特有的输出信号，表示屏幕正在刷新，其他操作需要 MCU 延后。

告知该模块兼容微雪 2.13 寸黑白屏幕 v1 版。参数如下:

- 尺寸： 2.13 inch
- 外形尺寸（裸屏）：59.2mm × 29.2mm × 1.05mm
- 显示尺寸：48.55mm × 23.71mm
- 工作电压：3.3V/5V
- 通信接口：SPI
- 点距：0.194\*0.194
- 分辨率：250\*122
- 显示颜色：黑、白
- 灰度等级：2
- 局部刷新 ：0.3s
- 全局刷新 ：2s
- 刷新功耗 ： 26.4mW(typ.)
- 待机功耗 ：<=0.017mW

查询模块手册，得知该显示屏驱动芯片是 IL3895, 来自 Good Display(大连佳显)。
屏幕排线上有型号 HINK-E0213A04-G01(HINK-E0213-G01).

### ESP8266

ESP8266 算是较推荐的墨水屏之友，IO 接口不多但够用，带 WiFi 功能, 适合做这类显示屏小制作。
可以轻易找到各类天气时钟等代码。

推荐编程环境 Arduino. 省事。需要安装 [ESP8266 Board Support 库](https://github.com/esp8266/Arduino).

我这里的 ESP8266 板子是一只 NodeMCU DevKit 兼容板。最普通不过，但比较麻烦的是它的管脚标签和标准的 ESP8266 GPIO
之间有一个映射关系。

![NodeMCU GPIOs](https://user-images.githubusercontent.com/72891/104639665-a2de3000-56e2-11eb-9396-04cf946880bd.png)
(ref: <https://www.electronicwings.com/nodemcu/nodemcu-gpio-with-arduino-ide>)

搞清楚映射关系，写代码就不会错了。

官方有出售专门的 ESP8266 驱动板，集成了 ESP8266, 只需要按照相同的接线，即可使用官方例程。

接线方式:

| EPD board | NodeMCU pin | ESP8266 pin |       description |
| --------- | ----------: | ----------: | ----------------: |
| BUSY      |          D1 |       GPIO5 |        屏幕刷新忙 |
| RES/RST   |          D4 |           2 |              复位 |
| DC        |          D2 |           4 | Data/Command 信号 |
| CS        |          D8 |          15 |              片选 |
| CLK/SCK   |          D5 |          14 |          SPI 时钟 |
| DIN/SDA   |          D7 |          13 |          SPI MOSI |

常识 VCC = 3.3V, GND 接地。

### 屏幕官方例程

虽然不是官方正版，只是个拆机屏模块，但好在屏幕型号一致，全兼容微雪官方例程。

- [官方 ESP8266 驱动板及例程](https://www.waveshare.net/wiki/E-Paper_ESP8266_Driver_Board)

官方例程解压后子目录复制到 Arduino libraries 目录。

然后就可以直接从 Arduino 的 File->Examples 菜单打开例程。

例程压缩包中的 `src/`, `extras/` 目录，其实就是官方驱动，所有的不同型号屏幕的例程，都依赖驱动库。

该款屏幕的例程是 `waveshare-e-Paper/epd2in13-demo`.

设备选择 NodeMCU 或 Generic EPS8266, 编译上传例程。

屏幕噌噌闪动几下，清屏后，开始执行例程。

![Official Demo](https://user-images.githubusercontent.com/72891/104642461-e4bca580-56e5-11eb-9a8f-ea2c7a38da22.jpg)

屏幕右下角的时间显示，秒位在不停变动。

例程中可以看到基础绘图，中英文数字显示，时间显示（局部刷新功能）。照着改改可以整出不少好玩的。
再加上 ESP8266 的 WiFi 功能，想象力足够。

### End of get start

至此，屏幕跑通。画画图，改改文字，皆大欢喜。🤪

---

以下是干货部分。需要知识预备:

- 二进制位运算知识
- 数字电子基础
- 电路基础
- 计算机图形学基础概念

## 中文显示

看到例程中直接有中文显示语句，暗爽不是？

```c
Paint_DrawString_CN(140, 60, "你好abc", &Font12CN, BLACK, WHITE);
Paint_DrawString_CN(5, 65, "**电子", &Font24CN, WHITE, BLACK);
```

于是改成“你好世界”，然而只见“你好”不见“世界”。

是的，官方例程（驱动）里没有完整字库，只有测试时候屏幕上出现的那几个汉字。所以需要加字库！

我们能接触到的绝大多数屏幕，都是点阵屏。
所谓字库，就是字符编码到图形象素点的映射。所以这里要增加缺失的字型，怎么整？

先看看官方怎么实现的。

当然，字库还有其他意思，在手机维修界，字库也指 ROM 芯片。
这是历史遗留问题了，当年字库都存在专门的芯片里。
字库和字库芯片在很多场合不区分。这里叫字库，其实是字模，即汉字的模型，对应的二进制数据。

### 官方驱动字库格式

找到驱动目录 `~/Documents/Arduino/libraries/esp8266-waveshare-epd`.

在 `src/` 目录下，找到若干 `font*.cpp`, `font*.h` 文件就是字库了。

例如 `font12CN` 字体，微软雅黑 12:

```c
const CH_CN Font12CN_Table[] =
{
    /*--  文字:  你  --*/
    /*--  微软雅黑12;  此字体下对应的点阵为：宽x高=16x21   --*/
    {"你",
    0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x1D,0xC0,0x1D,0x80,0x3B,0xFF,0x3B,0x07,
    0x3F,0x77,0x7E,0x76,0xF8,0x70,0xFB,0xFE,0xFB,0xFE,0x3F,0x77,0x3F,0x77,0x3E,0x73,
    0x38,0x70,0x38,0x70,0x3B,0xE0,0x00,0x00,0x00,0x00},
    // ...
    {"A",
    0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x0E,0x00,0x1F,0x00,0x1F,0x00,
    0x1F,0x00,0x3B,0x80,0x3B,0x80,0x71,0x80,0x7F,0xC0,0x71,0xC0,0xE0,0xE0,0xE0,0xE0,
    0xE0,0xE0,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00},
};

cFONT Font12CN = {
  Font12CN_Table,
  sizeof(Font12CN_Table)/sizeof(CH_CN),  /*size of table*/
  11, /* ASCII Width */
  16, /* Width */
  21, /* Height */
};
```

明显看到中文字库分两部分，一部分是字型表 `Font12CN_Table`, 一部分是配置结构体 `Font12CN`.
字型表即字的象素点二进制表示。因为中文字符较多，为节省空间字库只有例程所需汉字，
所以每条记录第一个元素是中文汉字，用于索引。
而对于英文字库来说，字符是连续的，且总体占用空间较小，一般使用连续字节块表示。

从配置结构我们可以得知，该字型宽 16 位，高 21 位，即两个字节表示一行象素，一共 21 行。
我们可以写个脚本展示下:

```py
char = [0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x1D,0xC0,0x1D,0x80,0x3B,0xFF,0x3B,0x07,
    0x3F,0x77,0x7E,0x76,0xF8,0x70,0xFB,0xFE,0xFB,0xFE,0x3F,0x77,0x3F,0x77,0x3E,0x73,
    0x38,0x70,0x38,0x70,0x3B,0xE0,0x00,0x00,0x00,0x00]
# group by 2 -> to 0-1 -> padding with 0
lines = ["%08d%08d" % (int(bin(l)[2:]), int(bin(r)[2:])) for (l,r) in zip(char[::2], char[1::2])]

print('\n'.join(lines).replace('0', '.').replace('1', '*'))
```

得到命令行下输出:

```text
................
................
................
................
...***.***......
...***.**.......
..***.**********
..***.**.....***
..******.***.***
.******..***.**.
*****....***....
*****.*********.
*****.*********.
..******.***.***
..******.***.***
..*****..***..**
..***....***....
..***....***....
..***.*****.....
................
................
```

一个汉字被解析了出来，以点阵的方式展示。上方和下方的 0, 用于行间距。
在实际应用中可以省去，节约空间。

而配置中的 ASCII Width 11 表示该中文字体中的 ASCII 字符宽度。因为半角全角的关系，
中文字体中的半角英文字符(ASCII)相对来说宽度都不足一字，为了显示效果，不留过多字间距，
丢弃多余位不用，所以这里单独有一个配置项。

```py
char = [0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x0E,0x00,0x1F,0x00,0x1F,0x00,
     ...:     0x1F,0x00,0x3B,0x80,0x3B,0x80,0x71,0x80,0x7F,0xC0,0x71,0xC0,0xE0,0xE0,0xE0,0xE0,
     ...:     0xE0,0xE0,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00]
lines = ["%08d%08d" % (int(bin(l)[2:]), int(bin(r)[2:])) for (l,r) in zip(char[::2], char[1::2])]

print('\n'.join(lines).replace('0', '.').replace('1', '*'))

# outputs:
"""
................
................
................
................
................
....***.........
...*****........
...*****........
...*****........
..***.***.......
..***.***.......
.***...**.......
.*********......
.***...***......
***.....***.....
***.....***.....
***.....***.....
................
................
................
................
"""
```

### 生成字库

搞明白了原理和格式，接下来就是生成所需的字库了。网上有非常多的 Windows 下小工具可以做。
但我这么肝，就自己写了。代码来自之前给 TFT-LCD 写的抠字模小脚本儿。

原理很简单，用 `Pillow/PIL` 即可。先把需要的字画在图片上，然后读取象素点，移位生成对于字节表示。
最后最好直接生成 C 代码，就万事大吉。

而字体选择，一般使用点阵字体而非矢量字体，矢量字体在渲染的时候边缘都是带灰阶的，
如果忽略灰阶直接二值化，会导致最终字型锯齿严重，丑。

为了方便处理，这里选用开源的等宽字体[文泉驿点阵字体 Unibit](http://wenq.org/wqy2/index.cgi?Unibit).
以中文点阵最常见的 16x16 输出。正好两个字节宽，16 行高，一个汉字 32 字节。
英文字符也正好是中文字符宽度的一半。

当然你可以随便从系统找个中文字体。需要注意的是，为了在黑白（无灰度）屏幕上达到最好效果，需要位图字体(bitmap font, raster font, pixel font),
否则矢量字体在渲染的过程中会有灰阶边缘，最终屏幕效果锯齿明显。

```sh
# 安装依赖

pip3 install Pillow

# 用于字体缩放的依赖库
brew install libraqm
```

```py
from PIL import Image, ImageDraw, ImageFont
import PIL.features

# brew install libraqm
assert PIL.features.check('raqm'), "libraqm required"

# 画布大小
size = (320, 16)

# 黑白格式
FORMAT = '1'
BG = 0
FG = 1

# Y offset, 多数字体有自带行间距，可以用此参数消除行间距
YOFF = 0  # or -1


CHARS = "晴天卧槽可以了！你好世界最怕你一生碌碌无为，还安慰自己平凡可贵。雾霾"
CHARS = ''.join(list(set(CHARS)))

im = Image.new(FORMAT, size, BG)

font = ImageFont.truetype("Unibit.ttf", size=16, index=0)


draw = ImageDraw.Draw(im)

# 代码段用于检查字体渲染
# draw.text((0, YOFF), CHARS, font=font, fill=FG, language='zh-CN')
# im.save('font.png')
# im.show()

draw.rectangle([(0, 0), size], fill=BG)

for i, c in enumerate(CHARS):
    charmap = []
    draw.text((0, YOFF), c, font=font, fill=FG)

    for y in range(16):
        v = 0
        for x in range(0, 16):
            b = im.getpixel((x, y))
            v = (v << 1) + b

        charmap.append(v >> 8)
        charmap.append(v & 0xFF)

    draw.rectangle([(0, 0), size], fill=BG)
    print("{", end='')
    print('"{}", {}'.format(c, ', '.join(map(lambda c: "0x%02x" % c, charmap))), end="")
    print("},")
```

直接输出 C 代码片段:

```c
{"一", 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xff, 0xfe, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00},
{"生", 0x01, 0x00, 0x11, 0x00, 0x11, 0x00, 0x11, 0x00, 0x3f, 0xfc, 0x21, 0x00, 0x41, 0x00, 0x81, 0x00, 0x01, 0x00, 0x3f, 0xf8, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00, 0xff, 0xfe, 0x00, 0x00},
{"碌", 0x00, 0x00, 0x01, 0xf8, 0xf8, 0x08, 0x20, 0x08, 0x21, 0xf8, 0x40, 0x08, 0x78, 0x08, 0x4b, 0xfe, 0xc8, 0x20, 0x4a, 0x22, 0x49, 0x74, 0x48, 0xa8, 0x79, 0x24, 0x4a, 0x22, 0x00, 0xa0, 0x00, 0x40},
{"碌", 0x00, 0x00, 0x01, 0xf8, 0xf8, 0x08, 0x20, 0x08, 0x21, 0xf8, 0x40, 0x08, 0x78, 0x08, 0x4b, 0xfe, 0xc8, 0x20, 0x4a, 0x22, 0x49, 0x74, 0x48, 0xa8, 0x79, 0x24, 0x4a, 0x22, 0x00, 0xa0, 0x00, 0x40},
{"无", 0x00, 0x00, 0x3f, 0xf0, 0x02, 0x00, 0x02, 0x00, 0x02, 0x00, 0x02, 0x00, 0x7f, 0xfc, 0x04, 0x80, 0x04, 0x80, 0x04, 0x80, 0x08, 0x80, 0x08, 0x80, 0x10, 0x84, 0x20, 0x84, 0x40, 0x7c, 0x80, 0x00},
{"为", 0x01, 0x00, 0x21, 0x00, 0x11, 0x00, 0x11, 0x00, 0x01, 0x00, 0x7f, 0xf8, 0x02, 0x08, 0x02, 0x08, 0x02, 0x88, 0x04, 0x48, 0x04, 0x48, 0x08, 0x08, 0x10, 0x08, 0x20, 0x08, 0x40, 0x50, 0x80, 0x20},
```

### 使用字库

要想使用字体，我们需要新建一个字体 `.h` 文件，就叫 `ch_font.h`，在项目目录即可，不需要修改驱动库:

```c

#include "fonts.h"

const CH_CN Font16CN_Table[] =
{
    {"一", 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xff, 0xfe, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00},
    {"生", 0x01, 0x00, 0x11, 0x00, 0x11, 0x00, 0x11, 0x00, 0x3f, 0xfc, 0x21, 0x00, 0x41, 0x00, 0x81, 0x00, 0x01, 0x00, 0x3f, 0xf8, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00, 0x01, 0x00, 0xff, 0xfe, 0x00, 0x00},
    // ... 这里写入剩余字型
};

cFONT Font16CN = {
  Font16CN_Table,
  sizeof(Font16CN_Table)/sizeof(CH_CN),  /*size of table*/
  8, /* ASCII Width */
  16, /* Width */
  16, /* Height */
};
```

使用:

```c
// ...
#include "cn_font.h"


void setup() {
    // ...
    Paint_DrawString_CN(140, 70, "卧槽可以了！", &Font16CN, BLACK, WHITE);

    EPD_2IN13_Display(BlackImage);
    // ...
}

// ...
```

效果大概是(忘记拍照了，这里是另外一个开源字体 Sarasa):

![Chicken soup](https://user-images.githubusercontent.com/72891/104816620-5b2ce500-5857-11eb-92fe-2dbd3add8eda.jpg)

是的，毒鸡汤。至此中文字体搞定，其他 CJK 字体同理。

而英文字体就更简单了，参考其他驱动中的 `font*` 文件即可。字型生成代码略改即可使用。

具体制作中可以混合中英文大小字体，还可以选择例如数码管字体等方案，完成布局。

![Mixed Font](https://user-images.githubusercontent.com/72891/104816434-21a7aa00-5856-11eb-9dd3-dc05255c8cdb.jpg)

## 其他显示需求

### 图片显示

墨水屏最烂大街的应用，大概就是天气时钟了，各式各样，各种尺寸。

显示文字信息的事情，上面我们已经通过自定义字库搞定了，
发挥想象力可以搞出诸如数码管字体，手写字体，等各种适合在墨水屏上实现的显示效果。

但问题来了，我想搞个天气图标符号显示，比如，[中国天气网](http://www.weather.com.cn/) 那样的。

![weather](https://user-images.githubusercontent.com/72891/104684474-165c5d80-5734-11eb-8850-9216e07a9dca.png)

官方例程里其实有全屏图片显示例子，可以看到相关的调用函数 `Paint_DrawBitMap`。
同时驱动库里也提供了 `Paint_DrawImage(buf, x_start, y_start, img_width, img_height)`
函数用于在任意位置显示任意大小图片。

看一眼驱动库里 `Paint_DrawImage` 代码，好像哪里不对，只支持宽度象素是 8 倍数的图片。
改也简单，用 `Paint_SetPixel` 函数替换即可，简单的位运算。

现在问题是怎么生成一张用于显示的图片。其实和上面的字库非常类似，就是用二进制位去映射象素点。
然后生成 C 数组。甚至核心代码逻辑也差不多。

需要注意的是图片只能是黑白二值图。为方便库函数识别，也提前将图片宽度处理成 8 的整数倍。

图源，就取天气网那堆图标，原图是用 CSS offset 方式显示的，也就是所有图标在一张图片上，需要切下。

完整代码见 [gist: crop-blue30.py](https://gist.github.com/andelf/acb0d317eef5d0cc8b7d53d4133c09c2).
这里贴要点

```py
# RGBA 到 RGB 的转换
pix = r, g, b, a = im1.getpixel((x, y))
BG = 255 # 背景是白色
r = (BG * (255 - a) + r * a) // 255
g = (BG * (255 - a) + g * a) // 255
b = (BG * (255 - a) + b * a) // 255
a = 255 # alpha 通道置空，不透明度

im1.putpixel((x, y), (r, g, b, a))
```

```py
# 图像二值化,
im1 = im1.resize((64, 64), Image.LANCZOS)
im1 = im1.filter(ImageFilter.SHARPEN)
im1 = im1.convert('1', dither=Image.NONE)
```

当然，你要是用 ImageMagick 或者 Photoshop 也一样可以。

然后把生成的二进制装入 C `uint8_t[]` 即可。

显示一个太阳:

![image](https://user-images.githubusercontent.com/72891/104686101-84565400-5737-11eb-9bf5-4e0347e1eb28.png)

因为二值化和缩放的关系，象素周围略有点腐蚀的感觉。
效果还行，考虑考虑界面元素布局，做个天气时钟足够了。

### 局部刷新

正常情况下在显示文字和图案的时候屏幕会连续从最黑到最白来回闪动几下，参考显示原理，
这里是为了避免残留墨水粒子影响显示效果，即消除残影的影响。
有使用过 Kindle 的同学对这点会比较清楚，一般是翻页若干次全部刷新一次。

官方例程中右下角有个时间显示，用到了局部刷新技术:

```c
EPD_2IN13_Init(EPD_2IN13_PART);
Paint_SelectImage(BlackImage);
// ...
Paint_ClearWindows(140, 90, 140 + Font20.Width * 7, 90 + Font20.Height, WHITE);
Paint_DrawTime(140, 90, &sPaint_time, &Font20, WHITE, BLACK);

EPD_2IN13_Display(BlackImage);
```

首先以 `EPD_2IN13_PART` 方式重新初始化屏幕（不用清屏），然后就通过 `Paint_ClearWindows` 清除需要局部更新的矩形区域，
之后就可以使用各种绘图绘字符函数填写屏幕的这部分。最后调用 Display 上屏。

局部刷新的效果因屏幕体质不同各异，经常会遇到残影特别严重的时候。做小制作的时候也可以学习电纸书，局部刷新多次后全局刷新一次。

## 再看显示原理

以上，基本介绍完了墨水屏的常见功能。已满足绝大部分需求。

回头再看显示原理，有了一开始介绍的小胶囊阵列结构，驱动怎么通过搞定显示的呢？这里以 2.13 寸显示屏的驱动 IC IL3895 为例。

### LUT

LUT, 即 Waveform Look Up Table(LUT), 是很多介绍电子墨水驱动文章的离不开的话题。

所谓的 LUT 功能，其实是驱动芯片的“可编程驱动电压波形”功能。即通过若干寄存器字节，
设置像素在不同状态转换情况下使用的底板电压高低的时序。该设置全局有效，针对全屏幕的任何一个像素的变动。
整个波形通过更新屏幕指令(MasterActivation = 0x20, Activate Display Update Sequence)触发，
此过程中 BUSY 信号有效，更新逻辑完成后, BUSY 信号结束。

例如数据手册中:

![image](https://user-images.githubusercontent.com/72891/104814031-a63efc00-5847-11eb-8fd4-8595a69ad1eb.png)

IL3895 芯片支持 10 个 phase 的波形，分别是 phase0A phase0B phase1A phase1B phase2A phase2B phase3A phase3B phase4A phase4B,
其中每个 phase 可以指定维持状态的时间周期数 TP.
每两个波形可以设置一个重复次数 RP.
在每个 phase, 都可以指定像素底板电压 VS 高低。不同电压级别驱动颜色微粒向不同方向运动，维持不同的时间，最终实现像素的黑白变化。

图表右侧的 XY=LL, XY=LH, ... 表示不同的像素值变动情形。例如 HL 表示像素从白色变动到黑色, LL 表示像素在此次刷新中值没有变化。

以上的所有配置项按照固定格式，最终形成了 LUT 表，可以通过命令设置:

![image](https://user-images.githubusercontent.com/72891/104814577-b1475b80-584a-11eb-9cf5-3dcc9b559bf5.png)

而例程中的 `EPD_2IN13_Init(EPD_2IN13_FULL/PART)`, 其中最重要也是唯一的区别就是 FULL 和 PART 初始化时使用的 LUT 表不同。

```c
const unsigned char EPD_2IN13_lut_full_update[] = {
    0x22, 0x55, 0xAA, 0x55, 0xAA, 0x55, 0xAA, 0x11,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x1E, 0x1E, 0x1E, 0x1E, 0x1E, 0x1E, 0x1E, 0x1E,
    0x01, 0x00, 0x00, 0x00, 0x00, 0x00
};

const unsigned char EPD_2IN13_lut_partial_update[] = {
    0x18, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x0F, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00
};
```

初始化后，每次显示更新，都会执行对应的 LUT 表时序。也就是说，全局刷新情况下的屏幕从全黑到全白好几次清空屏幕的动作，
就定义在这个 FULL 对应的 LUT 中。有兴趣的小伙伴可以解析下 `EPD_2IN13_lut_full_update`.

正因为有了 LUT 定义，屏幕以 `EPD_2IN13_PART` 方式初始化时，新显示内容不再需要全屏刷新后再显示，直接在原始状态进行绘制。

这里以 `EPD_2IN13_lut_partial_update` 为例介绍下实际 LUT 执行时候发生的动作。简单得多，以至于整个表只有 3 个非空字节，
对应 phase0A, phase0B, 其余 phase 设置因对应的 TP(period) 为 0, 不生效。所以只有两个 phase 的波形。

```rust
/// LUT for partial update.
#[rustfmt::skip]
pub const LUT_PARTIAL_UPDATE: [u8; 30] = [
    // VS, voltage in phase n
    // <<VS[0A-HH]:2/binary, VS[0A-HL]:2/binary, VS[0A-LH]:2/binary, VS[0A-LL]:2/binary>>
    // <<VS[0B-HH]:2/binary, VS[0B-HL]:2/binary, VS[0B-LH]:2/binary, VS[0B-LL]:2/binary>>
    // HL: white to black
    // LH: black to white
    // e.g. 0x18 = 0b00_01_10_00
    0x18, 0x00, // phase 0
    0x00, 0x00, // phase 1
    0x00, 0x00,
    0x00, 0x00,
    0x00, 0x00, // phase 4
    // padding
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    // RP, repeat counter, 0 to 63, 0 means run time = 1
    // TP, phase period, 0 to 31
    // <<RP[0]_L:3/binary, TP[0A]:5/binary>>
    // <<RP[0]_H:3/binary, TP[0B]:5/binary>>
    0x0F, 0x01, // phase 0
    0x00, 0x00,
    0x00, 0x00,
    0x00, 0x00,
    0x00, 0x00, // phase 4
    // padding
    0x00, 0x00, 0x00, 0x00
];
```

如上，改写为 Rust 代码，加入详细注释，注释混搭假 Erlang 语法。

先看 VS 部分，即驱动电压部分。`0x18 = 0b00_01_10_00`, 按照格式拆出:

```text
VS[0A-HL] = 01
VS[0A-LH] = 10

# 其他情况为 00
```

手册中有介绍 VS 格式:

```text
00–VSS
01–VSH
10–VSL

分别是三种电压输出级别，其中 VSS 与顶版 VCOM 相等。相当于无电场存在。
VSH, VSL 分别会导致是两种不同的电场方向。
```

所以解读 phase0A 的配置即:

- 当像素点从白转黑时(HL), $$V_{pixel}$$ 电压为 VSH
- 当象素点从黑转白时(LH), $$V_{pixel}$$ 电压为 VSL
- 其他情况下，$$V_{pixel}$$ 电压为 VSS, 由芯片手册可知, VSS = VCOM, 相当于无变化

再来看 RP, TP 部分。需要将字节的高地位组合:

```text
0x0F = 0b000_01111
0x01 = 0b000_00001

所以得到
RP[0] = 0b000_000 = 0
TP[0A] = 0b01111 = 15
TP[0B] = 0b00001 = 1
```

如上提到, TP 为 0 时表示该 phrase 无效，所以上面说只有 phase0A, phase0B 两个 phase 有效。
RP = 0 是表示重复 1 次。

总结以上那么该 LUT 的逻辑是:

- 当像素点从白转黑时(HL), $$V_{pixel}$$ 电压为 VSH, 持续 15 个周期，随后转 VSS, 持续一个周期
- 当象素点从黑转白时(LH), $$V_{pixel}$$ 电压为 VSL, 持续 15 个周期，随后转 VSS, 持续一个周期
- 其他情况下，恒为 VSS
- 以上逻辑执行 1 次
- 单个周期文档中介绍时长为 $$T_{FRAME}$$

所以似乎明了，所谓局部刷新，就是在像素点不变的时候，不做任何操作。
在像素翻转的情况下，执行一次给电压操作，随后静置少量时间。

而全刷新，可以很明显看到，前几个周期都是全置黑全置白的统一操作，为的是清屏。
之后才是处理不同像素状态变更的波形。

#### Idea

持续电压 15 个周期，那么少一点会怎么样？

测试发现这样出图效果没那么黑，甚至是灰色。

所以似乎就有了在黑白墨水屏上实现灰度显示的方案。这里的黑白墨水屏，特指驱动手册中单个像素为 1 位的屏幕。即 1bpp(bit per pixel).

本文题头图，即为测试效果。

灰度显示的内容，另开坑讲。

### framebuffer

> A framebuffer (frame buffer, or sometimes framestore) is a portion of random-access memory (RAM) containing a bitmap that drives a video display.
> It is a memory buffer containing data representing all the pixels in a complete video frame. -- Wikipedia

回到例程，可以看到屏幕初始化调用大概是:

```c
EPD_2IN13_Init(EPD_2IN13_FULL/PART);
EPD_2IN13_Clear();

UBYTE *BlackImage = malloc(...);
Paint_NewImage(BlackImage, EPD_2IN13_WIDTH, EPD_2IN13_HEIGHT, 270, WHITE);
Paint_SelectImage(BlackImage);

// ...

EPD_2IN13_Display(BlackImage);
```

阅读对应函数得知，这里创建了一个 `BlackImage` 字节数组做墨水屏的 framebuffer. 所有的绘图操作都只对这个 framebuffer 进行，
不和设备进行交互，直到调用 Display 才会进行实际的设备交互。

framebuffer 在 Display 被调用时，按照驱动芯片所设定的方向，比如按行，按列的方式，
一个个字节被从 MCU 传输到驱动芯片的内部的“显存”中，随后执行 LUT 更新逻辑。
驱动芯片会对比像素点的当前状态和目标状态，执行对应的 LH, HL, LL, HH 波形。
最终像素出现在屏幕上，完成显示。

### Next

之后抽时间写写灰度显示相关的折腾过程。

和 Rust embedded-grahics 驱动的情况。

## 参考

- Waveshare 官方 store, [wiki](https://www.waveshare.com/wiki/2.13inch_e-Paper_HAT)
- [IL3895 数据手册](https://www.e-paper-display.com/download_detail/downloadsId=538.html)
- [Wikipedia: E-Ink](https://en.wikipedia.org/wiki/E_Ink)
