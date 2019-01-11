---
title: Android 自定义 View 之 Draw
categories: Android
tags:
  - Android
abbrlink: 98a748f6
date: 2019-01-10 18:55:27
---

## 作用 ##
在自定义 View 过程中，Draw 的主要作用就是**`绘制 View 视图`**。

> 绘制 View 视图就是绘制 `View` 自身和装饰：背景、内容、滚动指示器、滚动条、和前景等。

## draw 过程详解 ##
**`draw`** 过程根据 View 的类型分为以下两种情况：
 - **`单一 View：`**仅绘制 `View` 自身。
 - **`ViewGroup：`**除了绘制 `View` 自身外，还需要绘制父容器中的其它所有子 `View`(遍历调用所有子元素的 `draw()` 和各子元素再递归去执行该流程)。

### 单一 View 的 draw 过程 ###
#### 应用场景 ####
在没有现成的控件 View 满足需求、需自己实现时，则使用自定义单一 View。

#### 使用方法 ####
继承自 `View`、`SurfaceView` 或 其他 `View`；`不包含子 View`。

#### 绘制原理 ####
绘制 View 视图一般分为2个步骤：
1. View 绘制自身(含背景、内容)；
2. 绘制装饰(滚动指示器、滚动条、和前景)。

#### 绘制流程 ####
![单一View的draw过程](https://lyl873825813.github.io//medias/view/view_draw_single.png)

下面将 `draw` 过程中的方法进行详细分析：`draw` 过程入口为 `draw()`
```java
    /**
     * 源码分析：draw()
     * <p>
     * 作用：根据给定的 Canvas 自动渲染 View(包括其所有子 View)
     * <p>
     * 绘制过程：
     * 1. 绘制 View 背景
     * 2. 绘制 View 内容
     * 3. 绘制子 View
     * 4. 绘制装饰(渐变框，滑动条等等)
     * <p>
     * 注：
     * a. 在调用该方法之前必须要完成 layout 过程
     * b. 所有的视图最终都是调用 View 的 draw() 绘制视图(ViewGroup 没有复写此方法)
     * c. 在自定义 View 时，不应该复写该方法，而是复写 onDraw(Canvas) 方法进行绘制
     * d. 若自定义的视图确实要复写该方法，那么需先调用 super.draw(canvas)完成系统的绘制，然后再进行自定义的绘制
     *
     * @param canvas 渲染视图的画布
     */
    @CallSuper
    public void draw(Canvas canvas) {
        ...

        /**
         * 绘制遍历执行几个绘制步骤，必须以适当的顺序执行：
         * 1.绘制背景
         * 2.如有必要，保存画布的图层以备复原图层
         * 3.绘制视图的内容
         * 4.绘制子视图
         * 5.如有必要，绘制淡化边缘并恢复图层
         * 6.绘制装饰(例如滚动条)
         */

        int saveCount;

        // 步骤1： 绘制View本身的背景
        if (!dirtyOpaque) {
            drawBackground(canvas);
        }

        // 若有必要，则保存图层(还有一个复原图层)
        // 优化技巧：当不需绘制 Layer 时，可以跳过“保存图层”和“复原图层”这两步
        // 因此在绘制时，节省 layer 可以提高绘制效率
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // 步骤3：绘制View本身的内容(View和ViewGroup中默认为空实现，需复写)
            if (!dirtyOpaque) {
                onDraw(canvas);
            }

            // 步骤4：绘制子View
            // 由于单一View无子View，故View 中：默认为空实现
            // ViewGroup中：系统已经复写好对其子视图进行绘制我们不需要复写
            dispatchDraw(canvas);

            drawAutofilledHighlight(canvas);

            // 叠加层是视图内容的一部分，需要在前景下方绘制
            if (mOverlay != null && !mOverlay.isEmpty()) {
                mOverlay.getOverlayView().dispatchDraw(canvas);
            }

            // 步骤6：绘制装饰(前景、滚动条等)
            onDrawForeground(canvas);

            // 步骤7：绘制默认焦点高亮显示
            drawDefaultFocusHighlight(canvas);

            if (debugDraw()) {
                debugDrawFocus(canvas);
            }

            return;
        }

        ...
    }
```

下面，继续分析在 `draw()` 中4个步骤调用的 `drawBackground()`、`onDraw()`、`dispatchDraw()`、`onDrawForeground()` 等方法：
```java
    /**
     * 步骤1：drawBackground(canvas)
     * <p>
     * 作用：绘制 View 本身的背景
     *
     * @param canvas 画布
     */
    private void drawBackground(Canvas canvas) {
        // 获取背景Drawable
        final Drawable background = mBackground;
        if (background == null) {
            return;
        }

        // 根据在 layout 过程中获取的 View 的位置参数，来设置背景的边界
        setBackgroundBounds();

        ...

        // 获取 mScrollX 和 mScrollY值
        final int scrollX = mScrollX;
        final int scrollY = mScrollY;
        if ((scrollX | scrollY) == 0) {
            background.draw(canvas);
        } else {
            // 若 mScrollX 和 mScrollY 有值，则对 canvas 的坐标进行偏移
            canvas.translate(scrollX, scrollY);

            // 调用 Drawable 的 draw 方法绘制背景
            background.draw(canvas);
            canvas.translate(-scrollX, -scrollY);
        }
    }

    /**
     * 步骤3：onDraw(canvas)
     * <p>
     * 作用：绘制 View 本身的内容
     * <p>
     * 注：
     * a. 由于 View 的内容各不相同，所以该方法是一个空实现
     * b. 在自定义绘制过程中，需由子类去实现复写该方法，从而绘制自身的内容
     * c. 谨记：自定义 View 中必须且只需复写 onDraw() 方法
     *
     * @param canvas 画布
     */
    protected void onDraw(Canvas canvas) {
        // 复写该方法从而实现绘制逻辑
    }

    /**
     * 步骤4：dispatchDraw(canvas)
     * <p>
     * 作用：绘制子 View
     * <p>
     * 注：由于单一View中无子View，故为空实现
     *
     * @param canvas 画布
     */
    protected void dispatchDraw(Canvas canvas) {
        // 空实现
    }

    /**
     * 步骤6： onDrawForeground(canvas)
     * <p>
     * 作用：绘制装饰(滚动指示器、滚动条、前景等)
     *
     * @param canvas 画布
     */
    public void onDrawForeground(Canvas canvas) {
        // 绘制滚动指示器
        onDrawScrollIndicators(canvas);
        // 绘制滚动条
        onDrawScrollBars(canvas);
        // 获取前景Drawable
        final Drawable foreground = mForegroundInfo != null ? mForegroundInfo.mDrawable : null;
        // 绘制前景
        if (foreground != null) {
            if (mForegroundInfo.mBoundsChanged) {
                mForegroundInfo.mBoundsChanged = false;
                final Rect selfBounds = mForegroundInfo.mSelfBounds;
                final Rect overlayBounds = mForegroundInfo.mOverlayBounds;

                if (mForegroundInfo.mInsidePadding) {
                    selfBounds.set(0, 0, getWidth(), getHeight());
                } else {
                    selfBounds.set(getPaddingLeft(), getPaddingTop(),
                            getWidth() - getPaddingRight(), getHeight() - getPaddingBottom());
                }

                final int ld = getLayoutDirection();
                Gravity.apply(mForegroundInfo.mGravity, foreground.getIntrinsicWidth(),
                        foreground.getIntrinsicHeight(), selfBounds, overlayBounds, ld);
                foreground.setBounds(overlayBounds);
            }

            foreground.draw(canvas);
        }
    }
```

#### 总结 ####
![单一View的draw过程总结](https://lyl873825813.github.io//medias/view/view_draw_single_all.png)


### ViewGroup 的 draw 过程 ###
#### 应用场景 ####
利用现有的组件根据特定的布局方式来组成新的组件。

#### 使用方法 ####
继承自 `ViewGroup` 或各种 `Layout`；`包含子 View`。

#### 绘制原理 ####
绘制 ViewGroup 视图一般分为2个步骤：
1. ViewGroup 绘制自身(含背景、内容)；
2. ViewGroup 遍历其所有子 View 并绘制其所有子 View。

![ViewGroup自上而下遍历](https://lyl873825813.github.io//medias/view/view_group_tree.png)

#### 绘制流程 ####
![ViewGroup的draw过程](https://lyl873825813.github.io//medias/view/view_draw_group.png)
`ViewGroup` 和 `View` 同样拥有 `draw()` 和 `ondraw()`，但二者不同的：

下面将 `draw` 过程中的方法进行详细分析：`draw` 过程入口为 `draw()`
```java
    /**
     * 源码分析：draw()
     * <p>
     * 作用：根据给定的 Canvas 自动渲染 View(包括其所有子 View)
     * <p>
     * 绘制过程：
     * 1. 绘制 View 背景
     * 2. 绘制 View 内容
     * 3. 绘制子 View
     * 4. 绘制装饰(渐变框，滑动条等等)
     * <p>
     * 注：
     * a. 在调用该方法之前必须要完成 layout 过程
     * b. 所有的视图最终都是调用 View 的 draw() 绘制视图(ViewGroup 没有复写此方法)
     * c. 在自定义 View 时，不应该复写该方法，而是复写 onDraw(Canvas) 方法进行绘制
     * d. 若自定义的视图确实要复写该方法，那么需先调用 super.draw(canvas)完成系统的绘制，然后再进行自定义的绘制
     *
     * @param canvas 渲染视图的画布
     */
    @CallSuper
    public void draw(Canvas canvas) {
        ...

        /**
         * 绘制遍历执行几个绘制步骤，必须以适当的顺序执行：
         * 1.绘制背景
         * 2.如有必要，保存画布的图层以备复原图层
         * 3.绘制视图的内容
         * 4.绘制子视图
         * 5.如有必要，绘制淡化边缘并恢复图层
         * 6.绘制装饰(例如滚动条)
         */

        int saveCount;

        // 步骤1： 绘制View本身的背景
        if (!dirtyOpaque) {
            drawBackground(canvas);
        }

        // 若有必要，则保存图层(还有一个复原图层)
        // 优化技巧：当不需绘制 Layer 时，可以跳过“保存图层”和“复原图层”这两步
        // 因此在绘制时，节省 layer 可以提高绘制效率
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // 步骤3：绘制View本身的内容(View和ViewGroup中默认为空实现，需复写)
            if (!dirtyOpaque) {
                onDraw(canvas);
            }

            // 步骤4：绘制子View
            // 由于单一View无子View，故View 中：默认为空实现
            // ViewGroup中：系统已经复写好对其子视图进行绘制我们不需要复写
            dispatchDraw(canvas);

            drawAutofilledHighlight(canvas);

            // 叠加层是视图内容的一部分，需要在前景下方绘制
            if (mOverlay != null && !mOverlay.isEmpty()) {
                mOverlay.getOverlayView().dispatchDraw(canvas);
            }

            // 步骤6：绘制装饰(前景、滚动条等)
            onDrawForeground(canvas);

            // 步骤7：绘制默认焦点高亮显示
            drawDefaultFocusHighlight(canvas);

            if (debugDraw()) {
                debugDrawFocus(canvas);
            }

            return;
        }

        ...
    }
```

由于在 `draw()` 中绘制步骤调用的 `drawBackground()`、`onDraw()`、`onDrawForeground()` 方法，与单一 View 的 draw 过程类似，下面只详细分析一下与单一 View 的 draw 过程最大不同的步骤：`dispatchDraw()`
```java
    /**
     * 源码分析：dispatchDraw()
     * <p>
     * 作用：遍历所有子 View 并绘制子 View
     * <p>
     * 注：
     * a. ViewGroup 中：由于系统为我们实现了该方法，故不需重写该方法
     * b. View 中默认为空实现(因为没有子View可以去绘制)
     *
     * @param canvas 画布
     */
    @Override
    protected void dispatchDraw(Canvas canvas) {

        final int childrenCount = mChildrenCount;
        ...

        // 1. 遍历子View
        for (int i = 0; i < childrenCount; i++) {
            while (transientIndex >= 0 && mTransientIndices.get(transientIndex) == i) {
                final View transientChild = mTransientViews.get(transientIndex);
                if ((transientChild.mViewFlags & VISIBILITY_MASK) == VISIBLE ||
                        transientChild.getAnimation() != null) {
                    // 2. 绘制子View视图 ->>分析1
                    more |= drawChild(canvas, transientChild, drawingTime);
                }
                ...
            }
        }
        ...
    }

    /**
     * 分析1：drawChild()
     * <p>
     * 作用：绘制子 View
     *
     * @param canvas      绘制子视图的画布
     * @param child       要绘制的子视图
     * @param drawingTime 绘制开始的时间
     */
    protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
        // 最终还是调用了子 View 的 draw() 方法进行子 View 的绘制
        return child.draw(canvas, this, drawingTime);
    }
```

#### 总结 ####
![ViewGroup的draw过程总结](https://lyl873825813.github.io//medias/view/view_draw_group_all.png)


## 总结 ##
![View的draw过程总结](https://lyl873825813.github.io//medias/view/view_draw_all.png)

