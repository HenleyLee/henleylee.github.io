---
title: Android 样式和主题背景
categories: Android
tags:
  - Android
abbrlink: eade425f
date: 2020-07-20 18:55:36
---

Android 提供了功能强大的样式系统 ([Android styling system](https://developer.android.com/guide/topics/ui/look-and-feel/themes)) 来实现应用的视觉设计，但它也容易被误用。正确地使用样式系统会让您在开发应用的时候更容易维护主题与样式，在开发新功能的时候少一些抓狂，而且还可以支持深色模式。

## 主题背景 != 样式 ##
主题背景与样式都使用相同的 `<style>` 语法，但是它们所服务的目的截然不同，可以把它们理解为使用键值对 (`Key-Value`) 来存储数据，其中键 (`Key`) 代表属性，值 (`Values`) 代表资源。

> 样式和主题背景在 `res/values/` 中的[样式资源文件](https://developer.android.com/guide/topics/resources/style-resource)中声明，通常命名为 `styles.xml`。

## 样式(Style) ##
样式是 `View` 属性 (`View Attributes`) 值的集合，可以把它们理解为 `Map<view attribute, resource>` 的结构。其中，一组键 (`Key`) 代表了所有的 `View` 属性，这里的 `View` 属性指的是可以在布局文件使用的 `Widget` 定义的属性。一个样式对应一种类型的 `Widget`，这是因为不同的部件支持不同的属性集合：
```xml
<style name="Widget.Plaid.Button.InlineAction" parent="…">
    <item name="android:gravity">center_horizontal</item>
    <item name="android:textAppearance">@style/TextAppearance.CommentAuthor</item>
    <item name="android:drawablePadding">@dimen/spacing_micro</item>
</style>
```

正如您所见，样式中的每一个键 (`Key`) 其实就是您可以在布局中设置的内容：
```xml
<Button …
    android:gravity="center_horizontal"
    android:textAppearance="@style/TextAppearance.CommentAuthor"
    android:drawablePadding="@dimen/spacing_micro" />
```

把这些提炼成样式，可以让您方便地在多个 View 中复用同一个样式，而且还容易维护。

> 样式是 `View` 属性 (`View Attributes`) 值的集合；一个样式对应一种类型的 `Widget`。

### 使用方法 ###
布局文件中的每一个独立的 View 都可以使用样式:
```xml
<Button …
    style="@style/Widget.Plaid.Button.InlineAction" />
```

一个 `View` 只能使用一个样式，可以将其与 `Web` 技术中使用到的 `CSS` 样式系统相比较，`CSS` 样式系统可以允许一个组件使用多个 `CSS` 类。

### 范围 ###
样式只有在使用它的 `View` 上才起作用，如果该 `View` 包含子 `View`，那么在这些子 `View` 上样式是无效的。举个例子，如果 `ViewGroup` 有三个按钮，设置 `InlineAction` 样式到此 `ViewGroup` 时，只针对这个 `ViewGroup` 有效，而对它的三个按钮来说是无效的。样式中定义的值与布局文件中设置的值会融合在一起 (解决方法见这篇文章：[使用样式优先级顺序](https://medium.com/androiddevelopers/whats-your-text-s-appearance-f3a1729192d))。

## 主题背景(Theme) ##
主题背景是一组命名的资源的集合，这些资源可以被样式或者布局文件等引用。它们提供了一种对 `Android` 资源的语义名称 (`Sematic name`)，能够让开发者在其他地方引用这些资源。例如 `colorPrimary` 就是对一个给定颜色的语义名称。
```xml
<style name="Theme.Plaid" parent="…">
    <item name="colorPrimary">@color/teal_500</item>
    <item name="colorSecondary">@color/pink_200</item>
    <item name="android:windowBackground">@color/white</item>
</style>
```

主题背景是由 `Map<theme attribute, resource>` 结构组成，这些标有名字的资源被称为主题背景属性。主题背景属性跟 `View` 属性不一样，这是因为它们不是特定 `View` 类型的属性而是对一个值的命名，其在应用中有更广泛的用途。主题背景属性为这些标有名字的资源提供了具体的值，在上面的例子中 `colorPrimary` 属性为这个主题背景设置了具体的值，也就是青绿色 (teal)。通过把主题背景中的资源抽象化，我们可以为不同的主题背景提供不同的值，比如: `colorPrimary=orange`。

> 主题背景是一种应用于整个应用、`Activity` 或视图层次结构的样式，而不仅仅应用于单个视图。当您将样式作为主题背景来应用时，应用或 `Activity` 中的每个视图都会应用其支持的每个样式属性。主题背景还可以将样式应用于非视图元素，例如状态栏和窗口背景。

主题背景类似于接口 (`Interface`)，在接口的编程中它允许您为公共接口提供不同的实现方法。主题扮演了一个类似的角色，针对主题属性编写布局和样式，我们可以在不同的主题下使用它们，从而提供不同的具体资源。简化的伪代码如下:
```java
interface ColorPalette {
    @ColorInt val colorPrimary
    @ColorInt val colorSecondary
}

class MyView(colors: ColorPalette) {
    fab.backgroundTint = colors.colorPrimary
}
```

这会让您使用同一套代码可渲染出不同的 `MyView` 效果，而无需新建构建变体。
```java
val lightPalette = object : ColorPalette { … }
val darkPalette = object : ColorPalette { … }
val view = MyView(if (isDarkTheme) darkPalette else lightPalette)
```

### 使用方法 ###
可以把一个主题背景设置给一个组件，这个组件可以包含 `Context` 或者它本身就是 `Context`，比如: `Activity` 或者是 `View/ViewGroups`。
```xml
<!-- AndroidManifest.xml -->
<application …
    android:theme="@style/Theme.Plaid">
<activity …
    android:theme="@style/Theme.Plaid.About"/>

<!-- layout/foo.xml -->
<ConstraintLayout …
    android:theme="@style/Theme.Plaid.Foo">
```

还可以使用 [ContextThemeWrapper](https://developer.android.com/reference/android/view/ContextThemeWrapper.html#ContextThemeWrapper(android.content.Context,%20int)) 类把一个主题背景设置到已经存在的 Context 上，这时候您可使用 [inflate](https://developer.android.com/reference/android/view/LayoutInflater.html#from(android.content.Context)) 方法创建布局。

主题背景的使用效果取决于您的使用方式，您可以通过引用主题背景属性来创建灵活的 Widget。不同的主题背景可以在未来再提供具体的值，比如为 View 层级结构中的某个部分设置背景颜色。
```xml
<ViewGroup …
    android:background="?attr/colorSurface">
```

除了用常量值设置一个颜色 (`#ffffff` 或者 `@color` 资源)，还可以通过 `?attr/themeAttributeName` 语法委托给主题背景来完成。这个语法表示通过指定的属性名称，从主题背景中获取相应的值。这种级别的解耦方式可以让我们提供不同的程序行为 (比如: 在深色模式与浅色模式下提供不同的背景颜色)，而不用创建多个相似但仅有一小部分不一样的布局或者样式，它将主题中的可变元素分离了出来。

> 通过使用 `?attr/themeAttributeName` 语法获得此主题背景中的语义属性代表的值。

### 范围 ###
任何一个带有 [Context](https://developer.android.com/reference/android/content/Context) (如 `Activity`, `View` or `ViewGroup`) 的对象都可以通过访问 `Context` 的属性来访问[主题背景](https://developer.android.com/reference/android/content/res/Resources.Theme.html)。这些对象以树的形式组织而成，比如 `Activity` 包含 `ViewGroup`，而 `ViewGroup` 又包含 `View`。把主题背景设置到一个树状结构的任意一层，此层及下一层都会受到影响。比如把主题背景设置给一个 `ViewGroup`，此 `ViewGroup` 包含的所有子 `View` 都会受到这个主题背景的影响，而样式恰好相反，它只对被设置的 `View` 起作用。
```xml
<ViewGroup …
    android:theme="@style/Theme.App.SomeTheme">
    <! - SomeTheme also applies to all child views. -->
</ViewGroup>
```

如果想在浅色屏幕中获取一个由深色主题背景构成的区域，那这个功能会非常有用。

请注意，这种功能仅在初始化布局的时候生效。在初始化布局之前需要调用 Context 提供的 [setTheme()](https://developer.android.com/reference/android/content/Context#setTheme(int)) 方法或者是主题背景提供的 [applyStyle()](https://developer.android.com/reference/android/content/res/Resources.Theme?hl=en#applyStyle(int,%20boolean)) 方法。布局初始化完毕之后再调用 setTheme 或者 applyStyle 方法，此时对已有的 View 不会造成任何改变。

## 常见的主题背景属性 ##
建议使用主题背景属性来间接引用资源，可以在不同的模式下 (比如在[深色主题背景](https://developer.android.com/guide/topics/ui/look-and-feel/darktheme)) 实现灵活地切换。如果您发现在布局或样式代码中直接引用了资源或者是硬编码了具体的值，请考虑使用主题背景属性来替代之前用法。
```xml
<ConstraintLayout ...
-  android:foreground="@drawable/some_ripple"
-  android:background="@color/blue" />
+  android:foreground="?attr/selectableItemBackground"
+  android:background="?attr/colorPrimarySurface" />
```

但是我们还可以使用哪些主题背景属性的功能呢？下面列举了常用的的关于主题背景属性的通用功能，它们广泛应用在 `Material`、`AppCompact`，或者是平台 (`Platform`) 中。

### 颜色 ###
这些颜色大部分来自于 [Material 颜色系统](https://material.io/design/color/the-color-system.html#color-usage-and-palettes) (`Material color system`) ，它们给每个颜色取了语义化的名称可以让您在应用中使用它们 ([体现为主题背景属性](https://material.io/develop/android/theming/color/)) 。
![](http://localhost:4000/medias/android/material_color_system.png)

 - **`?attr/colorPrimary：`**应用的主要颜色；
 - **`?attr/colorSecondary：`**应用的次要颜色，通常作为主要颜色补充；
 - **`?attr/colorOn[Primary, Secondary, Surface etc]：`**对应颜色的相反色；
 - **`?attr/color[Primary, Secondary]Variant：`**给定颜色的另一种阴影；
 - **`?attr/colorSurface：`**部件的表面颜色，如: 卡片、表格、菜单；
 - **`?android:attr/colorBackground：`**屏幕的背景颜色；
 - **`?attr/colorPrimarySurface：`**在浅色主题中的 `colorPrimary` 与深色主题背景中的 `colorSurface` 中做切换；
 - **`?attr/colorError：`**显示错误时的颜色。

其他常用的颜色:
 - **`?attr/colorControlNormal：`**正常状态下设置给 `icon/controls` 的颜色；
 - **`?attr/colorControlActivated：`**激活模式下设置给 `icons/controls` 的颜色 (如: 单选框被勾选)；
 - **`?attr/colorControlHighlight：`**设置给高亮控制界面的颜色 (如: `ripples`，列表选择器)；
 - **`?android:attr/textColorPrimary：`**设置给文本的主要颜色；
 - **`?android:attr/textColorSecondary：`**设置给文本的次要颜色。

### 大小 ###
 - **`?attr/listPreferredItemHeight：`**列表项的标准高度 (最小值)；
 - **`?attr/actionBarSize：`**工具栏的高度。

### Drawables ###
 - **`?attr/selectableItemBackground：`**可交互条目在 `ripple` 或者是高亮时的背景颜色 (针对外观)；
 - **`?attr/selectableItemBackgroundBorderless：`**无边界的 `ripple`；
 - **`?attr/dividerVertical：`**用于垂直分割可视化元素的 `drawable`；
 - **`?attr/dividerHorizontal：`**用于水平分割可视化元素的 `drawable`。

### TextAppearance ###
`Material` 定义了缩放类型，它是在整个应用中使用的一组由文本样式组成的离散集合，集合中的每个值都是一个主题背景属性，可以被设置为 `textApperance`。请点击 [Material type scale generator](https://material.io/design/typography/the-type-system.html#type-scale) 获得更多关于生成不同字体缩放的帮助。
![](http://localhost:4000/medias/android/material_type_system.png)

 - **`?attr/textAppearanceHeadline1：`**默认为 96sp light 文本；
 - **`?attr/textAppearanceHeadline2：`**默认为 60sp light 文本；
 - **`?attr/textAppearanceHeadline3：`**默认为 48sp regular 文本；
 - **`?attr/textAppearanceHeadline4：`**默认为 34sp regular 文本；
 - **`?attr/textAppearanceHeadline5：`**默认为 24sp regular 文本；
 - **`?attr/textAppearanceHeadline6：`**默认为 20sp medium 文本；
 - **`?attr/textAppearanceSubtitle1：`**默认为 16sp regular 文本；
 - **`?attr/textAppearanceSubtitle2：`**默认为 14sp medium 文本；
 - **`?attr/textAppearanceBody1：`**默认为 16sp regular 文本；
 - **`?attr/textAppearanceBody2：`**默认为 14sp regular 文本；
 - **`?attr/textAppearanceCaption：`**默认为 12sp regular 文本；
 - **`?attr/textAppearanceButton：`**默认为 14sp 全大写 medium 文本；
 - **`?attr/textAppearanceOverline：`**默认为 10sp 全大写 regular 文本。

### 形状 ###
`Material` 采用了形状系统 ([Shape system](https://material.io/design/shape/))，它是由主题背景属性[实现](https://material.io/develop/android/theming/shape)了 `small`、`medium`、`large` 等不同的部件。请注意，如果您想给自定义的部件设置形状外观，您应该使用 [MaterialShapeDrawable](https://developer.android.com/reference/com/google/android/material/shape/MaterialShapeDrawable) 作为它的背景，因为它能够理解并能实现具体形状。
![](http://localhost:4000/medias/android/material_shape_system.png)

 - **`?attr/shapeAppearanceSmallComponent：`**默认圆角为 4dp，用于 Buttons、Chips、TextFields 等；
 - **`?attr/shapeAppearanceMediumComponent：`**默认圆角为 4dp，用于 Cards、Dialogs、Date Pickers 等；
 - **`?attr/shapeAppearanceLargeComponent：`**默认圆角为 0dp (其实是方形)，用于 Bottom Sheets 等。

### 按钮风格 ###
![](http://localhost:4000/medias/android/material_button_style.webp)

`Material` 提供了三种不同类型的按钮: [Contained](https://material.io/components/buttons/#contained-button)、[Text](https://material.io/components/buttons/#text-button) 以及 [Outlined](https://material.io/components/buttons/#outlined-button)。MDC 提供了主题背景属性，您可以使用它们给 [MaterialButton](https://developer.android.com/reference/com/google/android/material/button/MaterialButton) 设置样式:
 - **`?attr/materialButtonStyle：`**(defaults)默认是 Contained 类型 (或者直接省略样式)；
 - **`?attr/borderlessButtonStyle：`**设置为 Text 样式的按钮；
 - **`?attr/materialButtonOutlinedStyle：`**设置为 Outlined 样式的按钮。

### Floats ###
 - **`?android:attr/disabledAlpha：`**默认关闭 Widget 的 alpha；
 - **`?android:attr/primaryContentAlpha：`**设置给 foreground 元素的 alpha 值；
 - **`?android:attr/secondaryContentAlpha：`**设置给 secondary 元素的 alpha 值。

### 命名空间 ###
命名空间分为`应用命名空间`和 `Android 命名空间`。
您可能注意到有些属性的引用是通过 `?android:attr/foo` 而有些只是通过 `?attr/bar`。这是因为一些属性是由 `Android` 平台定义的，所以需要使用 `android` 命名空间来引用由它们自己定义的属性 (类似于布局中使用 `View` 属性 `android:id`) 。编译到应用但不是来自于静态库的属性 (`AppCompact` 或者 `MDC`) ，使用它们时不需要命名空间 (类似于布局中使用 `app:baz`) 。平台跟库有时候定义了相同的属性，如 `colorPrimary`。这时候系统优先使用非平台版本的属性，它们可以被所有级别的 `API` 使用。为了向后兼容，它们会被完整的复制到库中。上面列举的都是非平台版本的案例。

> 优先使用非平台版本的属性，它们可以被所有级别的 `API` 使用

### 更多资源 ###
为了获取可以使用的全部主题背景属性，请查阅以下信息:
 - [Android platform](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/res/res/values/attrs.xml)
 - [AppCompat](https://android.googlesource.com/platform/frameworks/support/+/androidx-master-dev/appcompat/appcompat/src/main/res/values/attrs.xml)

Material 设计的部件:
 - [Color](https://github.com/material-components/material-components-android/blob/master/lib/java/com/google/android/material/color/res/values/attrs.xml)
 - [Shape](https://github.com/material-components/material-components-android/blob/master/lib/java/com/google/android/material/shape/res/values/attrs.xml)
 - [Type](https://github.com/material-components/material-components-android/blob/master/lib/java/com/google/android/material/typography/res/values/attrs.xml)

### 自己动手 ###
当想使用主题背景功能抽象某个东西的时候，发现没有现成的主题背景可用时，可以自定义一个。可以参考 [Google I/O 应用](https://github.com/google/iosched)，它实现了在两个界面中显示主题演讲的列表:
![](http://localhost:4000/medias/android/material_shape_system.png)

这两个界面大部分看起来比较相似，除了左边界面有个显示时间的功能而右边是没有的。

将 item 的对齐部分抽象成一个主题背景属性，给不同界面使用的同一个布局中使用主题背景来区分它们的差异:
1. 在 [attrs.xml](https://github.com/google/iosched/blob/89df01ebc19d9a46495baac4690c2ebfa74946dc/mobile/src/main/res/values/attrs.xml#L41) 中定义主题背景属性:
```xml
<attr name="sessionListKeyline" format="dimension" />
```

2. 在不同的主题背景中使用不同的值([值 1](https://github.com/google/iosched/blob/89df01ebc19d9a46495baac4690c2ebfa74946dc/mobile/src/main/res/values/themes.xml#L51)和[值 2](https://github.com/google/iosched/blob/89df01ebc19d9a46495baac4690c2ebfa74946dc/mobile/src/main/res/values/themes.xml#L78)):
```xml
<style name="Theme.IOSched.Schedule">
    …
    <item name="sessionListKeyline">72dp</item>
</style>

<style name="Theme.IOSched.Speaker">
    …
    <item name="sessionListKeyline">16dp</item>
</style>
```

3. 给两个界面使用的布局文件中[使用主题背景属性](https://github.com/google/iosched/blob/89df01ebc19d9a46495baac4690c2ebfa74946dc/mobile/src/main/res/layout/item_session.xml#L61):
```xml
<Guideline …
    app:layout_constraintGuide_begin="?attr/sessionListKeyline" />
```

### 保持探索 ###
了解了能够使用的主题背景属性功能后，可以在编写布局、样式、drawables 时使用它们。

使用主题背景属性功能更容易实现主题功能 (如深色主题背景)，而且可以编写出更灵活，更易于维护的代码。

## 参考 ##
 - [Styles and Themes](https://developer.android.com/guide/topics/ui/look-and-feel/themes)
 - [Dark theme](https://developer.android.com/guide/topics/ui/look-and-feel/darktheme)

