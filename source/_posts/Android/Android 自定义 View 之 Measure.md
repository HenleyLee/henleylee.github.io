---
title: Android 自定义 View 之 Measure
categories: Android
tags:
  - Android
abbrlink: 48d8e111
date: 2019-01-06 18:39:26
---

## 作用 ##
在自定义 View 过程中，Measure 的主要作用就是**`测量 View 的宽/高`**。

> 在某些情况下，需要多次测量(`measure`)才能确定 View 最终的宽/高；该情况下，`measure` 过程后得到的宽/高可能不准确；此处建议：在 `layout` 过程中 `onLayout()` 去获取最终的宽/高。

## LayoutParams ##
**`android.view.ViewGroup.LayoutParams`** 类是 `android.view.ViewGroup` 的一个内部类，表示布局参数。

### 作用 ###
`LayoutParams` 的主要作用是用来指定 View 的高度(height)和 宽度(width)等布局参数。

### 具体使用 ###
`LayoutParams` 主要通过以下参数指定：

| 参数         | 解释                                                         |
|--------------|--------------------------------------------------------------|
| 具体值       | 指定具体的宽/高(dp / px)                                     |
| fill_parent  | 强制性使子视图的大小扩展至与父视图大小相等(不含 padding)     |
| match_parent | 与 fill_parent 相同，用于 Android 2.3 及以上版本             |
| wrap_content | 自适应大小，强制性地使视图扩展以便显示其全部内容(含 padding) |

### 构造方法 ###
```java
public LayoutParams(android.content.Context context, android.util.AttributeSet attrs)

public LayoutParams(int width, int height)

public LayoutParams(android.view.ViewGroup.LayoutParams source)
```

## MeasureSpec ##
**`android.view.View.MeasureSpec`** 类是 `android.view.View` 的一个内部类，表示测量规格。

### 作用 ###
`MeasureSpec` 的主要作用是用来作为测量 View 的大小(宽/高)的依据。

### 组成 ###
`MeasureSpec` 由测量模式(`mode`)和测量大小(`size`)。MeasureSpec 是 View 中的内部类，基本都是二进制运算。由于 int 是 32 位的，用**高两位表示 mode，低 30 位表示 size**，MODE_SHIFT = 30 的作用是移位。

### 测量模式 ###
测量模式(`mode`)的类型有3种：UNSPECIFIED、EXACTLY 和 AT_MOST。具体如下：

| 模式        | 描述                                                                           | 应用场景                |
|-------------|--------------------------------------------------------------------------------|-------------------------|
| UNSPECIFIED | 父控件没有给子视图任何限制，子视图可以设置为任意大小(不常用)                   | ListView，ScrollView    |
| EXACTLY     | 父控件为子视图指定了确切的尺寸，子视图大小必须在该指定尺寸内                   | 具体数值或 march_parent |
| AT_MOST     | 父控件为子视图指定一个最大尺寸，子视图必须确保自身和所有子视图可适应在该尺寸内 | wrap_content            |

### 具体使用 ###
`MeasureSpec` 类用1个变量封装了2个数据(size  和 mode)：通过使用二进制，将测量模式(mode)和测量大小(size)打包成一个 int 值来，并提供了打包和解包的方法。
```java
// 获取测量模式(Mode)
int specMode = MeasureSpec.getMode(measureSpec)

// 获取测量大小(Size)
int specSize = MeasureSpec.getSize(measureSpec)

// 通过 Mode 和 Size 生成新的 SpecMode
int measureSpec=MeasureSpec.makeMeasureSpec(size, mode)
```

### 源码分析 ###
```java
/**
 * MeasureSpec 类的源码分析
 */
public class MeasureSpec {

    // 进位大小为2的30次方(int的大小为32位，所以进位30位就是要使用int的最高位和倒数第二位也就是32和31位做标志位)
    private static final int MODE_SHIFT = 30;

    // 运算遮罩，0x3为16进制，10进制为3，二进制为11。3向左进位30，就是11 00000000000(11后跟30个0)
    // 遮罩的作用：用1标注需要的值，0标注不要的值。因为1与任何数做与运算都得任何数，0与任何数做与运算都得0
    private static final int MODE_MASK = 0x3 << MODE_SHIFT;

    // UNSPECIFIED的模式设置：0向左进位30 = 00后跟30个0，即00 00000000000
    public static final int UNSPECIFIED = 0 << MODE_SHIFT;

    // EXACTLY的模式设置：1向左进位30 = 01后跟30个0 ，即01 00000000000
    public static final int EXACTLY = 1 << MODE_SHIFT;

    // AT_MOST的模式设置：2向左进位30 = 10后跟30个0，即10 00000000000
    public static final int AT_MOST = 2 << MODE_SHIFT;

    /**
     * 根据提供的size和mode得到一个详细的测量结果吗，即 measureSpec
     * <p>
     * 设计目的：使用一个32位的二进制数，其中：32和31位代表测量模式(mode)、后30位代表测量大小(size)
     */
    public static int makeMeasureSpec(int size, int mode) {
        // measureSpec = size + mode；此为二进制的加法，而不是十进制！
        return size + mode;
    }

    /**
     * 通过 measureSpec 获得测量模式(mode)
     * <p>
     * 原理：保留measureSpec的高2位（即测量模式）、使用0替换后30位
     */
    public static int getMode(int measureSpec) {
        // 测量模式(mode) = measureSpec & MODE_MASK;
        // MODE_MASK = 运算遮罩 = 11 00000000000(11后跟30个0)
        return (measureSpec & MODE_MASK);
    }

    /**
     * 作用：通过 measureSpec 获得测量大小(size)
     * <p>
     * 原理：将 MODE_MASK 取反，也就是变成了00 111111(00后跟30个1)，将32,31替换成0也就是去掉 mode，保留后30位的 size
     *
     * @param measureSpec the measure specification to extract the size from
     * @return the size in pixels defined in the supplied measure specification
     */
    public static int getSize(int measureSpec) {
        // 测量大小(size) = measureSpec & ~MODE_MASK;
        return (measureSpec & ~MODE_MASK);
    }

    /**
     * 打印 mode 和 size 的信息
     */
    public static String toString(int measureSpec) {
        int mode = getMode(measureSpec);
        int size = getSize(measureSpec);
        StringBuilder sb = new StringBuilder("MeasureSpec: ");
        if (mode == UNSPECIFIED)
            sb.append("UNSPECIFIED ");
        else if (mode == EXACTLY)
            sb.append("EXACTLY ");
        else if (mode == AT_MOST)
            sb.append("AT_MOST ");
        else
            sb.append(mode).append(" ");
        sb.append(size);
        return sb.toString();
    }

}
```

## measure 过程详解 ##
**`measure`** 过程根据 View 的类型分为以下两种情况：
 - **`单一 View：`**仅测量 `View` 自身的大小。
 - **`ViewGroup：`**对 `ViewGroup` 视图中的所有子 `View` 都进行测量(遍历调用所有子元素的 `measure()` 和各子元素再递归去执行该流程)。

### 单一 View 的 measure 过程 ###
#### 应用场景 ####
在没有现成的控件 View 满足需求、需自己实现时，则使用自定义单一 View。

#### 使用方法 ####
继承自 `View`、`SurfaceView` 或 其他 `View`；`不包含子 View`。

#### 测量流程 ####
![单一View的measure过程](https://lyl873825813.github.io/medias/view/view_measure_single.png)

下面将 `measure` 过程中的方法进行详细分析：`measure` 过程入口为 `measure()`
```java
    /**
     * 源码分析：measure()
     * <p>
     * 定义：Measure 过程的入口；属于 View 类 & final 类型，即子类不能重写此方法
     * <p>
     * 作用：基本测量逻辑的判断
     *
     * @param widthMeasureSpec  View 的宽度测量规格
     * @param heightMeasureSpec View 的高度测量规格
     */
    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        ...

        if (forceLayout || needsLayout) {
            ...
            int cacheIndex = forceLayout ? -1 : mMeasureCache.indexOfKey(key);
            if (cacheIndex < 0 || sIgnoreMeasureCache) {
                // 调用 onMeasure() 计算视图大小 ->>分析1
                onMeasure(widthMeasureSpec, heightMeasureSpec);
		mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            } else {
                ...
            }
        }

        ...
    }

    /**
     * <p>
     * 分析1：onMeasure()
     * <p>
     * 作用：a. 根据 View 宽/高的测量规格计算 View 的宽/高值：getDefaultSize()
     *       b. 存储测量后的 View 宽/高：setMeasuredDimension()
     *
     * @param widthMeasureSpec  View 的宽度测量规格
     * @param heightMeasureSpec View 的高度测量规格
     */
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        // setMeasuredDimension()：获得 View 宽/高的测量值 ->>分析2
        // 传入的参数通过 getDefaultSize() 获得 ->>分析3
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }

    /**
     * 分析2：setMeasuredDimension()
     * <p>
     * 作用：存储测量后的 View 宽/高
     * <p>
     * 注：该方法即为重写 onMeasure() 所要实现的最终目的
     * <p>
     * 此方法必须由 onMeasure(int, int) 调用来存储测量的宽度和高度
     *
     * @param measuredWidth  测量后 View 的宽度值
     * @param measuredHeight 测量后 View 的高度值
     */
    protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {
        boolean optical = isLayoutModeOptical(this);
        if (optical != isLayoutModeOptical(mParent)) {
            Insets insets = getOpticalInsets();
            int opticalWidth = insets.left + insets.right;
            int opticalHeight = insets.top + insets.bottom;

            measuredWidth += optical ? opticalWidth : -opticalWidth;
            measuredHeight += optical ? opticalHeight : -opticalHeight;
        }
	// setMeasuredDimensionRaw()：设置 View 宽/高的测量值 ->>分析4
        setMeasuredDimensionRaw(measuredWidth, measuredHeight);
    }

    /**
     * 分析4：setMeasuredDimensionRaw()
     * <p>
     * 作用：设置测量的尺寸
     *
     * @param measuredWidth  测量后 View 的宽度值
     * @param measuredHeight 测量后 View 的高度值
     */
    private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
        // 将测量后子View的宽 / 高值进行传递
        mMeasuredWidth = measuredWidth;
        mMeasuredHeight = measuredHeight;

        mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
    }

     /**
     * 分析3：getDefaultSize()
     * <p>
     * 作用：根据 View 宽/高的测量规格计算 View 的宽/高值
     *
     * @param size        View 的默认大小
     * @param measureSpec 宽/高的测量规格(含模式和测量大小)
     */
    public static int getDefaultSize(int size, int measureSpec) {
        // 设置默认大小
        int result = size;

	// 获取宽/高测量规格的模式 & 测量大小
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
            // 模式为 UNSPECIFIED 时，使用提供的默认大小 = 参数 Size
            case MeasureSpec.UNSPECIFIED:
                result = size;
                break;
            // 模式为 AT_MOST、EXACTLY 时，使用 View 测量后的宽/高值 = measureSpec 中的 Size
            case MeasureSpec.AT_MOST:
            case MeasureSpec.EXACTLY:
                result = specSize;
                break;
        }
        // 返回 View 的宽/高值
        return result;
    }
```

#### 总结 ####
![单一View的measure过程总结](https://lyl873825813.github.io/medias/view/view_measure_single_all.png)

### ViewGroup 的 measure 过程 ###
#### 应用场景 ####
利用现有的组件根据特定的布局方式来组成新的组件。

#### 使用方法 ####
继承自 `ViewGroup` 或各种 `Layout`；`包含子 View`。

#### 测量原理 ####
自上而下、一层层地传递下去，直到完成整个 View 树的 `measure()` 过程：
1. 遍历测量所有子 `View` 的尺寸。
2. 将所有子 `View` 的尺寸进行合并，最终得到 `ViewGroup` 父视图的测量值。

![ViewGroup自上而下遍历](https://lyl873825813.github.io/medias/view/view_group_tree.png)

#### 测量流程 ####
![ViewGroup的measure过程](https://lyl873825813.github.io/medias/view/view_measure_group.png)

下面将 `measure` 过程中的方法进行详细分析：`measure` 过程入口为 `measure()`
```java
    /**
     * 源码分析：measure()
     * <p>
     * 定义：Measure 过程的入口；属于 View 类 & final 类型，即子类不能重写此方法
     * <p>
     * 作用：基本测量逻辑的判断
     *
     * @param widthMeasureSpec  View 的宽度测量规格
     * @param heightMeasureSpec View 的高度测量规格
     */
    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        ...

        if (forceLayout || needsLayout) {
            ...
            int cacheIndex = forceLayout ? -1 : mMeasureCache.indexOfKey(key);
            if (cacheIndex < 0 || sIgnoreMeasureCache) {
                // 调用 onMeasure() 计算视图大小 ->>分析1
                onMeasure(widthMeasureSpec, heightMeasureSpec);
		mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            } else {
                ...
            }
        }

        ...
    }

    /**
     * <p>
     * 分析1：onMeasure()
     * <p>
     * 作用：遍历所有子 View 并进行测量
     * <p>
     * 注：ViewGroup 是一个抽象类，没有重写 View 的 onMeasure() 方法，需自身重写
     *
     * @param widthMeasureSpec  View 的宽度测量规格
     * @param heightMeasureSpec View 的高度测量规格
     */
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        ...
    }
```

由于不同的 `ViewGroup` 子类(`LinearLayout`、`RelativeLayout`、`自定义 ViewGroup 子类`等)具备不同的布局特性，这导致它们的子 `View` 的测量方法各有不同，而 `onMeasure()` 方法的作用就是测量 View 的宽/高值。因此 ViewGroup 的 measure 过程无法像单一 View 的 measure 过程那样可以对 onMeasure() 做统一的实现。这个也是单一 View 的 measure 过程与 ViewGroup 过程最大的不同。

> 注：其实，在单一 View 的 measure 过程中，getDefaultSize() 只是简单的测量了宽高值，在实际使用时有时需更精细的测量，所以有时候也需重写 onMeasure() 方法。

在自定义 `ViewGroup` 中，关键在于：根据需求复写 `onMeasure()` 从而实现子 View 测量逻辑。复写 `onMeasure()` 的实现如下：
```java
    /**
     * 根据自身的测量逻辑复写 onMeasure()，分为3步
     * 1. 遍历所有子 View 并测量：measureChildren()
     * 2. 合并所有子 View 的尺寸大小,最终得到V iewGroup 父视图的测量值(自身实现)
     * 3. 存储测量后 View 宽/高的值：调用 setMeasuredDimension()
     *
     * @param widthMeasureSpec
     * @param heightMeasureSpec
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        // 定义存放测量后的View宽/高的变量
        int widthMeasure;
        int heightMeasure;

        // 1. 遍历所有子 View 并测量：measureChildren()
        // ->> 分析1
        measureChildren(widthMeasureSpec, heightMeasureSpec);

        // 2. 合并所有子View的尺寸大小，最终得到ViewGroup父视图的测量值(自身实现)
        ...

        // 3. 存储测量后View宽/高的值：调用setMeasuredDimension()
        // 类似单一View的过程，此处不作过多描述
        setMeasuredDimension(widthMeasure, heightMeasure);
    }

    /**
     * 分析1：measureChildren()
     * <p>
     * 作用：遍历子 View 并调用 measureChild() 方法进行下一步测量
     *
     * @param widthMeasureSpec  父视图的宽度测量规格
     * @param heightMeasureSpec 父视图的高度测量规格
     */
    protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        // 遍历所有子View
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            // 调用measureChild()进行下一步的测量 ->>分析1
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }

    /**
     * 分析2：measureChild()
     * <p>
     * 作用：a. 计算单个子 View 的 MeasureSpec
     *      b. 测量每个子 View 最后的宽 / 高：调用子 View 的 measure()
     *
     * @param child 要测量的子 View
     * @param parentWidthMeasureSpec 父视图的宽度测量规格
     * @param parentHeightMeasureSpec 父视图的高度测量规格
     */
    protected void measureChild(View child, int parentWidthMeasureSpec, int parentHeightMeasureSpec) {
        // 1. 获取子视图的布局参数
        final LayoutParams lp = child.getLayoutParams();

        // 2. 根据父视图的 MeasureSpec 和布局参数LayoutParams，计算单个子View的MeasureSpec
        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom, lp.height);

        // 3. 将计算好的子View的MeasureSpec值传入measure()，进行最后的测量
        // 下面的流程即类似单一View的过程，此处不作过多描述
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }

     /**
     * 分析3：getChildMeasureSpec()
     * <p>
     * 作用：根据父 View 的 MeasureSpec 与子 View 的 LayoutParams 计算子 View 的 MeasureSpec
     *
     * @param spec           父 View 的宽/高测量规格
     * @param padding        View 当前尺寸的的内边距和外边距(padding、margin)
     * @param childDimension 子 View 在当前尺寸下的布局参数宽/高值
     */
    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        // 父View的模式和大小
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        // 通过父View计算出的子View
        int size = Math.max(0, specSize - padding);

        // 子View想要的实际大小和模式(需要计算)
        int resultSize = 0;
        int resultMode = 0;

        // 通过父View的MeasureSpec和子View的LayoutParams属性来确定子View的大小
        switch (specMode) {
            // 当父View的模式为EXACITY时，父View为子View指定了确切的尺寸
            // 一般是父View设置为match_parent或者固定值的ViewGroup
            case MeasureSpec.EXACTLY:
                if (childDimension >= 0) {// 当子View的LayoutParams>0也就是有确切的值
                    // 子View大小为子自身所赋的值，模式大小为EXACTLY
                    resultSize = childDimension;
                    resultMode = MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.MATCH_PARENT) {
                    // 子View大小为父View大小，模式为EXACTLY
                    resultSize = size;
                    resultMode = MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                    // 子View决定自己的大小，但最大不能超过父View，模式为AT_MOST
                    resultSize = size;
                    resultMode = MeasureSpec.AT_MOST;
                }
                break;

            // 当父View的模式为AT_MOST时，父View为子View指定一个最大尺寸
            // 一般是父View设置为wrap_content
            case MeasureSpec.AT_MOST:
                if (childDimension >= 0) {
                    // 子View大小为子自身所赋的值，模式大小为EXACTLY
                    resultSize = childDimension;
                    resultMode = MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.MATCH_PARENT) {
                    // 子View大小为父View大小，模式为AT_MOST
                    resultSize = size;
                    resultMode = MeasureSpec.AT_MOST;
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                    // 子View决定自己的大小，但最大不能超过父View，模式为AT_MOST
                    resultSize = size;
                    resultMode = MeasureSpec.AT_MOST;
                }
                break;

            // 当父View的模式为UNSPECIFIED时，父View没有给子View任何限制，子View可以设置为任意大小
            // 多见于ListView、GridView
            case MeasureSpec.UNSPECIFIED:
                if (childDimension >= 0) {
                    // 子View大小为子自身所赋的值，模式大小为EXACTLY
                    resultSize = childDimension;
                    resultMode = MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.MATCH_PARENT) {
                    // 因为父View为UNSPECIFIED，所以MATCH_PARENT的话子类大小为0
                    resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                    resultMode = MeasureSpec.UNSPECIFIED;
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                    // 因为父View为UNSPECIFIED，所以WRAP_CONTENT的话子类大小为0
                    resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                    resultMode = MeasureSpec.UNSPECIFIED;
                }
                break;
        }
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```

#### 总结 ####
![ViewGroup的measure过程总结](https://lyl873825813.github.io/medias/view/view_measure_group_all.png)

## 总结 ##
![View的measure过程总结](https://lyl873825813.github.io/medias/view/view_measure_all.png)

