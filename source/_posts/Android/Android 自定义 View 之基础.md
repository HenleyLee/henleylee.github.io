---
title: Android 自定义 View 之基础
categories: Android
tags:
  - Android
abbrlink: 55df9ec1
date: 2019-01-02 18:46:35
---

## View 的分类 ##
视图 View 主要分为两类：

| 类别     | 解释                                          | 特点          |
| :------: | :-------------------------------------------: | :------------ |
| 单一视图 | 即一个View，如 TextView                       | 不包含子 View |
| 视图容器 | 即多个 View 组成的 ViewGroup，如 LinearLayout | 包含子 View   |

## View 类简介 ##
 - View 类是 Android 中各种组件的基类，如 View 是 ViewGroup 的基类。
 - View 表现为显示在屏幕上的各种视图，在屏幕上占据一个矩形区域，负责绘图和事件处理。
    > Android 中的 UI 组件都由 View、ViewGroup 组成。
 - View 的构造函数：共有4个，具体如下：
    ```java
    public class CustomView extends View {
    
        // 如果View是在Java代码里面new的，则调用第一个构造函数
        public CustomView(Context context) {
            super(context);
        }
    
        // 如果View是在xml里声明的，则调用第二个构造函数，自定义属性是从AttributeSet参数传进来的
        public CustomView(Context context, @Nullable AttributeSet attrs) {
            super(context, attrs);
        }
    
        // 不会自动调用，一般是在第二个构造函数里主动调用，如View有style属性时
        public CustomView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
            super(context, attrs, defStyleAttr);
        }
    
        // 不会自动调用，一般是在第二个构造函数里主动调用，如View有style属性时(API21之后才使用)
        public CustomView(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes) {
            super(context, attrs, defStyleAttr, defStyleRes);
        }
    
    }
     ```
    > 自定义 View 必须重写至少一个构造函数。


## View 视图结构 ##
对于多 View 的视图，结构是**`树形结构`**：最顶层是 ViewGroup，ViewGroup 下可能有多个 ViewGroup 或 View，如下图：
![View 视图结构](https://lyl873825813.github.io/medias/view/view_structure.png)

> 无论是 measure 过程、layout 过程还是 draw 过程，**永远都是从 View 树的根节点开始测量或计算（即从树的顶端开始），一层一层、一个分支一个分支地进行（即树形递归）**，最终计算整个 View 树中各个 View，最终确定整个 View 树的相关属性。

## Android 坐标系 ##
Android 的坐标系定义为：
 - 屏幕的左上角为坐标原点；
 - 向右为 X 轴增大方向；
 - 向下为 Y 轴增大方向。

具体如下图： 
![Android 坐标系](https://lyl873825813.github.io/medias/view/android_coordinate_system.png)

区别于一般的数学坐标系：
![其他坐标系](https://lyl873825813.github.io/medias/view/other_coordinate_system.png)

### View 位置描述 ###
View 的位置由4个顶点决定的（如下A、B、C、D）：
![View 位置](https://lyl873825813.github.io/medias/view/view_location.png)

4个顶点的位置描述分别由4个值决定：
 - Top：子 View 上边界到父 View 上边界的距离；
 - Left：子 View 左边界到父 View 左边界的距离；
 - Bottom：子 View 下边距到父 View 上边界的距离；
 - Right：子 View 右边界到父 View 左边界的距离。

如下图：
![View 位置](https://lyl873825813.github.io/medias/view/view_location_get.png)
可以按顶点位置来记忆：
 - Top：子 View 左上角距父 View 顶部的距离；
 - Left：子 View 左上角距父 View 左侧的距离；
 - Bottom：子 View 右下角距父 View 顶部的距离；
 - Right：子 View 右下角距父 View 左侧的距离。

> View 的位置是相对于父控件而言的。

### View 位置获取方式 ###
 - View的位置是通过 `view.getxxx()` 函数进行获取：
    ```java
    getTop();        // 获取子 View 左上角距父 View 顶部的距离
    getLeft();       // 获取子 View 左上角距父 View 左侧的距离
    getBottom();     // 获取子 View 右下角距父 View 顶部的距离
    getRight();      // 获取子 View 右下角距父 View 左侧的距离
    ```

 - 与 MotionEvent 中 `get()` 和 `getRaw()` 的区别：
    ```java
    // get() ：触摸点相对于其所在组件坐标系的坐标
    event.getX();       
    event.getY();

    // getRaw() ：触摸点相对于屏幕默认坐标系的坐标
    event.getRawX();    
    event.getRawY();
    ```
具体如下图所示：
![MotionEvent 坐标](https://lyl873825813.github.io/medias/view/motion_event_location_get.png)

## Android 的角度与弧度 ##
 - 自定义 View 实际上是将一些简单的形状通过计算，从而组合到一起形成的效果。
    > 这会涉及到画布的相关操作(旋转)、正余弦函数计算等，即会涉及到角度(angle)与弧度(radian)的相关知识。
 - 角度和弧度都是描述角的一种度量单位，区别如下图：
![角度与弧度的区别](https://lyl873825813.github.io/medias/view/differ_angle_radian.png)

在默认的屏幕坐标系中角度增大方向为顺时针。
![角度增大方向](https://lyl873825813.github.io/medias/view/angle_increase_direction.png)
> 注：在常见的数学坐标系中角度增大方向为逆时针。

## Android 中的颜色 ##
Android 中的颜色值通常遵循 `RGB/ARGB` 标准，使用时通常以 `#` 字符开头，以`16进制`表示。 
 - **`RGB`**依次代表红色(Red)、绿色(Green)、蓝色(Blue)。
 - **`ARGB`**依次代表透明度(Alpha)、红色(Red)、绿色(Green)、蓝色(Blue)。

> eg: `#FF00CC99` 其中FF是透明度，00是红色值，CC是绿色值，99是蓝色值。

### 颜色模式 ###
Android 支持的颜色模式有以下几种：

| 颜色模式   | 描述                |
|------------|---------------------|
| `ARGB8888` | 四通道高精度(32位)  |
| `ARGB4444` | 四通道低精度(24位)  |
| `RGB565`   | 屏幕默认模式(16位)  |
| `Alpha8`   | 仅有透明通道(8位)   |

> 其中字母表示通道类型，数值表示该类型用多少位二进制来描述。如 `ARGB8888` 则表示有四个通道(ARGB)，每个对应的通道均用 `8` 位来描述。

> **注意：**我们常用的是 `ARGB8888` 和 `ARGB4444`，而在所有的安卓设备屏幕上默认的模式都是 `RGB565`。

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

### 颜色定义 ###
#### 在 Java 中定义颜色 ####
```java
// 使用Color类定义颜色
int color = Color.GRAY;                   // 灰色
// 使用ARGB值进行表示
int color = Color.argb(127, 255, 0, 0);   // 半透明红色
int color = 0xaaff0000;                   // 带有透明度的红色
```

#### 在 xml 中定义颜色 ####
```xml
<resources>
    <!-- 定义了红色（没有alpha（透明）通道）-->
    <color name="red">#ff0000</color>
    <!-- 定义了蓝色（没有alpha（透明）通道）-->
    <color name="green">#00ff00</color>
</resources>
```

在 xml 文件中以 `#` 开头定义颜色，后面跟十六进制的值，有如下几种定义方式：
```
#f00            // 低精度 - 不带透明通道红色
#af00           // 低精度 - 带透明通道红色

#ff0000         // 高精度 - 不带透明通道红色
#aaff0000       // 高精度 - 带透明通道红色
```

### 颜色引用 ###
#### 在 Java 中引用 xml 中定义的颜色 ####
```java
//方法1
int color = getResources().getColor(R.color.red);

//方法2（API 23及以上）
int color = getColor(R.color.red); 
```

#### 在 xml 文件(layout或style)中引用或定义颜色 ####
```xml
<!--在style文件中引用-->
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <item name="colorPrimary">@color/red</item>
</style>

<!--在layout文件中引用在/res/values/color.xml中定义的颜色-->
android:background="@color/red"

<!--在layout文件中创建并使用颜色-->
android:background="#ff0000"
```

