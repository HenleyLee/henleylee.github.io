---
title: Android 中的颜色
categories: Android
tags:
  - Android
abbrlink: 936e3ab3
date: 2018-12-16 10:30:35
---

Android 中的颜色值通常遵循 `RGB/ARGB` 标准，使用时通常以 `#` 字符开头，以`16进制`表示。 
 - **`RGB`**依次代表红色(Red)、绿色(Green)、蓝色(Blue)。
 - **`ARGB`**依次代表透明度(Alpha)、红色(Red)、绿色(Green)、蓝色(Blue)。

> eg: `#FF00CC99` 其中FF是透明度，00是红色值，CC是绿色值，99是蓝色值。

## 颜色模式 ##
Android 支持的颜色模式有以下几种：

| 颜色模式   | 描述                |
|------------|---------------------|
| `ARGB8888` | 四通道高精度(32位)  |
| `ARGB4444` | 四通道低精度(24位)  |
| `RGB565`   | 屏幕默认模式(16位)  |
| `Alpha8`   | 仅有透明通道(8位)   |

> 其中字母表示通道类型，数值表示该类型用多少位二进制来描述。如 `ARGB8888` 则表示有四个通道(ARGB)，每个对应的通道均用 `8` 位来描述。

> **注意：**我们常用的是 `ARGB8888` 和 `ARGB4444`，而在所有的安卓设备屏幕上默认的模式都是 `RGB565`。

## 颜色定义 ##
以 `ARGB8888` 为例介绍颜色定义：

| 类型       | 解释   | 0(0x00) | 255(0xFF) |
|------------|--------|---------|-----------|
| `A(Alpha)` | 透明度 | 透明    | 不透明    |
| `R(Red)`   | 红色   | 无色    | 红色      |
| `G(Green)` | 绿色   | 无色    | 绿色      |
| `B(Blue)`  | 蓝色   | 无色    | 蓝色      |

其中 `A` `R` `G` `B` 的取值范围均为`0~255`(即16进制的`0x00~0xFF`)：
 - **`A：`**从`00`到`FF`表示从透明到不透明。
 - **`RGB：`**从`00`到`FF`表示颜色从浅到深。

> 当 `RGB`全取最小值(`0`或`0x000000`)时颜色为**黑色**，全取最大值(`255`或`0xFFFFFF`)时颜色为**白色**。

## Android 中的 Color ##
`android.graphics.Color` 类提供了创建、转换和操作颜色的方法。

### 颜色常量 ###
`android.graphics.Color` 类中提供了一些常用的颜色常量。
```java
@ColorInt public static final int BLACK       = 0xFF000000;
@ColorInt public static final int DKGRAY      = 0xFF444444;
@ColorInt public static final int GRAY        = 0xFF888888;
@ColorInt public static final int LTGRAY      = 0xFFCCCCCC;
@ColorInt public static final int WHITE       = 0xFFFFFFFF;
@ColorInt public static final int RED         = 0xFFFF0000;
@ColorInt public static final int GREEN       = 0xFF00FF00;
@ColorInt public static final int BLUE        = 0xFF0000FF;
@ColorInt public static final int YELLOW      = 0xFFFFFF00;
@ColorInt public static final int CYAN        = 0xFF00FFFF;
@ColorInt public static final int MAGENTA     = 0xFFFF00FF;
@ColorInt public static final int TRANSPARENT = 0;
```
### 静态方法 ###
`android.graphics.Color` 类中提供了一些构造颜色值的静态方法。
```java
Color.alpha(color)      // 提取alpha成分
Color.red(color)        // 提取红色成分
Color.green(color)      // 提取绿色成分
Color.blue(color)       // 提取蓝色成分

Color.rgb(red, green, blue)              // 根据red、green、blue成分返回一个颜色值
Color.argb(alpha, red, green, blue)      // 根据alpha、red、green、blue成分返回一个颜色值

Color.parseColor(colorString)            // 解析颜色字符串并返回一个颜色值

Color.luminance(color)                   // 返回颜色的相对亮度

Color.RGBToHSV(red, green, blue, hsv)    // 将RGB转换为HSV
Color.colorToHSV(color, float hsv[])     // 将ARGB颜色转换为HSV
Color.HSVToColor(hsv[])                  // 将HSV转换为ARGB颜色，alpha默认为0xFF
Color.HSVToColor(alpha, hsv[])           // 将HSV转换为ARGB颜色
```

### 构造方法 ###
`android.graphics.Color` 类中提供了一些构造方法。
```java
Color()
Color(float, float, float, float)
Color(float, float, float, float, android.graphics.ColorSpace)
Color(float[], android.graphics.ColorSpace)
```

> Color 类提供的四个构造方法中，只有无参的构造方法可以使用，其他几个构造方法都是私有的。但是可以通过 `Color.valueOf()` 和 `Color.convert()` 方法去构造一个颜色。

## 颜色透明度 ##

### 透明度 ###
 - 透明度分为256阶，取值范围是`0-255`，在计算机中，我们就用16进制(`00-FF`)表示，**`全透明就是00，完全不透明就是FF`**。
 - 透明度和不透明度是两个概念，他们加起来等于1，或者说100%
 - ARGB 中的透明度 alpha，表示的是不透明度。

### 计算方法 ###
在开发过程中，UI/UE 给的标注图上，所有颜色值是 RGB，但是透明度经常都是百分比，使用过程中我们需要进行换算，换算过程如下：
1. 将透明度转换成不透明度。
2. 不透明度乘以255。
3. 将计算结果转换成16进制。
4. 将不透明度和颜色值拼接成ARGB格式。

计算代码如下：
```java
public static void main(String[] args) {
    for (int i = 0; i <= 100; i++) {
        int alpha = Math.round(255 * i * 1.0f / 100f);
        String hex = Integer.toHexString(alpha);
        if (hex.length() < 2) {
            hex = "0" + hex;
        }
        System.out.println(i + "% ==> " + hex.toUpperCase());
    }
}
```

### 干货(懒程序员必备) ###
颜色值(#AARRGGBB)的透明度百分比和十六进制对应关系，如下表所示：

| 透明度 | 十六进制 |
|--------|----------|
| 100%   | FF       |
| 99%    | FC       |
| 98%    | FA       |
| 97%    | F7       |
| 96%    | F5       |
| 95%    | F2       |
| 94%    | F0       |
| 93%    | ED       |
| 92%    | EB       |
| 91%    | E8       |
| 90%    | E6       |
| 89%    | E3       |
| 88%    | E0       |
| 87%    | DE       |
| 86%    | DB       |
| 85%    | D9       |
| 84%    | D6       |
| 83%    | D4       |
| 82%    | D1       |
| 81%    | CF       |
| 80%    | CC       |
| 79%    | C9       |
| 78%    | C7       |
| 77%    | C4       |
| 76%    | C2       |
| 75%    | BF       |
| 74%    | BD       |
| 73%    | BA       |
| 72%    | B8       |
| 71%    | B5       |
| 70%    | B3       |
| 69%    | B0       |
| 68%    | AD       |
| 67%    | AB       |
| 66%    | A8       |
| 65%    | A6       |
| 64%    | A3       |
| 63%    | A1       |
| 62%    | 9E       |
| 61%    | 9C       |
| 60%    | 99       |
| 59%    | 96       |
| 58%    | 94       |
| 57%    | 91       |
| 56%    | 8F       |
| 55%    | 8C       |
| 54%    | 8A       |
| 53%    | 87       |
| 52%    | 85       |
| 51%    | 82       |
| 50%    | 80       |
| 49%    | 7D       |
| 48%    | 7A       |
| 47%    | 78       |
| 46%    | 75       |
| 45%    | 73       |
| 44%    | 70       |
| 43%    | 6E       |
| 42%    | 6B       |
| 41%    | 69       |
| 40%    | 66       |
| 39%    | 63       |
| 38%    | 61       |
| 37%    | 5E       |
| 36%    | 5C       |
| 35%    | 59       |
| 34%    | 57       |
| 33%    | 54       |
| 32%    | 52       |
| 31%    | 4F       |
| 30%    | 4D       |
| 28%    | 4A       |
| 28%    | 47       |
| 27%    | 45       |
| 26%    | 42       |
| 25%    | 40       |
| 24%    | 3D       |
| 23%    | 3B       |
| 22%    | 38       |
| 21%    | 36       |
| 20%    | 33       |
| 19%    | 30       |
| 18%    | 2E       |
| 17%    | 2B       |
| 16%    | 29       |
| 15%    | 26       |
| 14%    | 24       |
| 13%    | 21       |
| 12%    | 1F       |
| 11%    | 1C       |
| 10%    | 1A       |
| 9%     | 17       |
| 8%     | 14       |
| 7%     | 12       |
| 6%     | 0F       |
| 5%     | 0D       |
| 4%     | 0A       |
| 3%     | 08       |
| 2%     | 05       |
| 1%     | 03       |
| 0%     | 00       |

## 常用的颜色值 ##
补充一些常用的颜色值：
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="white">#FFFFFF</color> <!--白色 -->
    <color name="ivory">#FFFFF0</color> <!--象牙色 -->
    <color name="lightyellow">#FFFFE0</color> <!--亮黄色 -->
    <color name="yellow">#FFFF00</color> <!--黄色 -->
    <color name="snow">#FFFAFA</color> <!--雪白色 -->
    <color name="floralwhite">#FFFAF0</color> <!--花白色 -->
    <color name="lemonchiffon">#FFFACD</color> <!--柠檬绸色 -->
    <color name="cornsilk">#FFF8DC</color> <!--米绸色 -->
    <color name="seashell">#FFF5EE</color> <!--海贝色 -->
    <color name="lavenderblush">#FFF0F5</color> <!--淡紫红 -->
    <color name="papayawhip">#FFEFD5</color> <!--番木色 -->
    <color name="blanchedalmond">#FFEBCD</color> <!--白杏色 -->
    <color name="mistyrose">#FFE4E1</color> <!--浅玫瑰色 -->
    <color name="bisque">#FFE4C4</color> <!--桔黄色 -->
    <color name="moccasin">#FFE4B5</color> <!--鹿皮色 -->
    <color name="navajowhite">#FFDEAD</color> <!--纳瓦白 -->
    <color name="peachpuff">#FFDAB9</color> <!--桃色 -->
    <color name="gold">#FFD700</color> <!--金色 -->
    <color name="pink">#FFC0CB</color> <!--粉红色 -->
    <color name="lightpink">#FFB6C1</color> <!--亮粉红色 -->
    <color name="orange">#FFA500</color> <!--橙色 -->
    <color name="lightsalmon">#FFA07A</color> <!--亮肉色 -->
    <color name="darkorange">#FF8C00</color> <!--暗桔黄色 -->
    <color name="coral">#FF7F50</color> <!--珊瑚色 -->
    <color name="hotpink">#FF69B4</color> <!--热粉红色 -->
    <color name="tomato">#FF6347</color> <!--西红柿色 -->
    <color name="orangered">#FF4500</color> <!--红橙色 -->
    <color name="deeppink">#FF1493</color> <!--深粉红色 -->
    <color name="fuchsia">#FF00FF</color> <!--紫红色 -->
    <color name="magenta">#FF00FF</color> <!--红紫色 -->
    <color name="red">#FF0000</color> <!--红色 -->
    <color name="oldlace">#FDF5E6</color> <!--老花色 -->
    <color name="lightgoldenrodyellow">#FAFAD2</color> <!--亮金黄色 -->
    <color name="linen">#FAF0E6</color> <!--亚麻色 -->
    <color name="antiquewhite">#FAEBD7</color> <!--古董白 -->
    <color name="salmon">#FA8072</color> <!--鲜肉色 -->
    <color name="ghostwhite">#F8F8FF</color> <!--幽灵白 -->
    <color name="mintcream">#F5FFFA</color> <!--薄荷色 -->
    <color name="whitesmoke">#F5F5F5</color> <!--烟白色 -->
    <color name="beige">#F5F5DC</color> <!--米色 -->
    <color name="wheat">#F5DEB3</color> <!--浅黄色 -->
    <color name="sandybrown">#F4A460</color> <!--沙褐色 -->
    <color name="azure">#F0FFFF</color> <!--天蓝色 -->
    <color name="honeydew">#F0FFF0</color> <!--蜜色 -->
    <color name="aliceblue">#F0F8FF</color> <!--艾利斯兰 -->
    <color name="khaki">#F0E68C</color> <!--黄褐色 -->
    <color name="lightcoral">#F08080</color> <!--亮珊瑚色 -->
    <color name="palegoldenrod">#EEE8AA</color> <!--苍麒麟色 -->
    <color name="violet">#EE82EE</color> <!--紫罗兰色 -->
    <color name="darksalmon">#E9967A</color> <!--暗肉色 -->
    <color name="lavender">#E6E6FA</color> <!--淡紫色 -->
    <color name="lightcyan">#E0FFFF</color> <!--亮青色 -->
    <color name="burlywood">#DEB887</color> <!--实木色 -->
    <color name="plum">#DDA0DD</color> <!--洋李色 -->
    <color name="gainsboro">#DCDCDC</color> <!--淡灰色 -->
    <color name="crimson">#DC143C</color> <!--暗深红色 -->
    <color name="palevioletred">#DB7093</color> <!--苍紫罗兰色 -->
    <color name="goldenrod">#DAA520</color> <!--金麒麟色 -->
    <color name="orchid">#DA70D6</color> <!--淡紫色 -->
    <color name="thistle">#D8BFD8</color> <!--蓟色 -->
    <color name="lightgray">#D3D3D3</color> <!--亮灰色 -->
    <color name="lightgrey">#D3D3D3</color> <!--亮灰色 -->
    <color name="tan">#D2B48C</color> <!--茶色 -->
    <color name="chocolate">#D2691E</color> <!--巧可力色 -->
    <color name="peru">#CD853F</color> <!--秘鲁色 -->
    <color name="indianred">#CD5C5C</color> <!--印第安红 -->
    <color name="mediumvioletred">#C71585</color> <!--中紫罗兰色 -->
    <color name="silver">#C0C0C0</color> <!--银色 -->
    <color name="darkkhaki">#BDB76B</color> <!--暗黄褐色-->
    <color name="rosybrown">#BC8F8F</color> <!--褐玫瑰红 -->
    <color name="mediumorchid">#BA55D3</color> <!--中粉紫色 -->
    <color name="darkgoldenrod">#B8860B</color> <!--暗金黄色 -->
    <color name="firebrick">#B22222</color> <!--火砖色 -->
    <color name="powderblue">#B0E0E6</color> <!--粉蓝色 -->
    <color name="lightsteelblue">#B0C4DE</color> <!--亮钢兰色-->
    <color name="paleturquoise">#AFEEEE</color> <!--苍宝石绿 -->
    <color name="greenyellow">#ADFF2F</color> <!--黄绿色 -->
    <color name="lightblue">#ADD8E6</color> <!--亮蓝色 -->
    <color name="darkgray">#A9A9A9</color> <!--暗灰色 -->
    <color name="darkgrey">#A9A9A9</color> <!--暗灰色 -->
    <color name="brown">#A52A2A</color> <!--褐色 -->
    <color name="sienna">#A0522D</color> <!--赭色 -->
    <color name="darkorchid">#9932CC</color> <!--暗紫色 -->
    <color name="palegreen">#98FB98</color> <!--苍绿色 -->
    <color name="darkviolet">#9400D3</color> <!--暗紫罗兰色 -->
    <color name="mediumpurple">#9370DB</color> <!--中紫色 -->
    <color name="lightgreen">#90EE90</color> <!--亮绿色 -->
    <color name="darkseagreen">#8FBC8F</color> <!--暗海兰色 -->
    <color name="saddlebrown">#8B4513</color> <!--重褐色 -->
    <color name="darkmagenta">#8B008B</color> <!--暗洋红 -->
    <color name="darkred">#8B0000</color> <!--暗红色 -->
    <color name="blueviolet">#8A2BE2</color> <!--紫罗兰蓝色 -->
    <color name="lightskyblue">#87CEFA</color> <!--亮天蓝色 -->
    <color name="skyblue">#87CEEB</color> <!--天蓝色 -->
    <color name="gray">#808080</color> <!--灰色 -->
    <color name="grey">#808080</color> <!--灰色 -->
    <color name="olive">#808000</color> <!--橄榄色 -->
    <color name="purple">#800080</color> <!--紫色 -->
    <color name="maroon">#800000</color> <!--粟色 -->
    <color name="aquamarine">#7FFFD4</color> <!--碧绿色 -->
    <color name="chartreuse">#7FFF00</color> <!--黄绿色 -->
    <color name="lawngreen">#7CFC00</color> <!--草绿色 -->
    <color name="mediumslateblue">#7B68EE</color> <!--中暗蓝色 -->
    <color name="lightslategray">#778899</color> <!--亮蓝灰 -->
    <color name="lightslategrey">#778899</color> <!--亮蓝灰 -->
    <color name="slategray">#708090</color> <!--灰石色 -->
    <color name="slategrey">#708090</color> <!--灰石色 -->
    <color name="olivedrab">#6B8E23</color> <!--深绿褐色 -->
    <color name="slateblue">#6A5ACD</color> <!--石蓝色 -->
    <color name="dimgray">#696969</color> <!--暗灰色 -->
    <color name="dimgrey">#696969</color> <!--暗灰色 -->
    <color name="mediumaquamarine">#66CDAA</color> <!--中绿色 -->
    <color name="cornflowerblue">#6495ED</color> <!--菊兰色 -->
    <color name="cadetblue">#5F9EA0</color> <!--军兰色 -->
    <color name="darkolivegreen">#556B2F</color> <!--暗橄榄绿-->
    <color name="indigo">#4B0082</color> <!--靛青色 -->
    <color name="mediumturquoise">#48D1CC</color> <!--中绿宝石 -->
    <color name="darkslateblue">#483D8B</color> <!--暗灰蓝色 -->
    <color name="steelblue">#4682B4</color> <!--钢兰色 -->
    <color name="royalblue">#4169E1</color> <!--皇家蓝 -->
    <color name="turquoise">#40E0D0</color> <!--青绿色 -->
    <color name="mediumseagreen">#3CB371</color> <!--中海蓝 -->
    <color name="limegreen">#32CD32</color> <!--橙绿色 -->
    <color name="darkslategray">#2F4F4F</color> <!--暗瓦灰色 -->
    <color name="darkslategrey">#2F4F4F</color> <!--暗瓦灰色 -->
    <color name="seagreen">#2E8B57</color> <!--海绿色 -->
    <color name="forestgreen">#228B22</color> <!--森林绿 -->
    <color name="lightseagreen">#20B2AA</color> <!--亮海蓝色 -->
    <color name="dodgerblue">#1E90FF</color> <!--闪兰色 -->
    <color name="midnightblue">#191970</color> <!--中灰兰色 -->
    <color name="aqua">#00FFFF</color> <!--浅绿色 -->
    <color name="cyan">#00FFFF</color> <!--青色 -->
    <color name="springgreen">#00FF7F</color> <!--春绿色 -->
    <color name="lime">#00FF00</color> <!--酸橙色 -->
    <color name="mediumspringgreen">#00FA9A</color> <!--中春绿色 -->
    <color name="darkturquoise">#00CED1</color> <!--暗宝石绿 -->
    <color name="deepskyblue">#00BFFF</color> <!--深天蓝色 -->
    <color name="darkcyan">#008B8B</color> <!--暗青色 -->
    <color name="teal">#008080</color> <!--水鸭色 -->
    <color name="green">#008000</color> <!--绿色 -->
    <color name="darkgreen">#006400</color> <!--暗绿色 -->
    <color name="blue">#0000FF</color> <!--蓝色 -->
    <color name="mediumblue">#0000CD</color> <!--中兰色 -->
    <color name="darkblue">#00008B</color> <!--暗蓝色 -->
    <color name="navy">#000080</color> <!--海军色 -->
    <color name="black">#000000</color> <!--黑色 -->
</resources>
```

