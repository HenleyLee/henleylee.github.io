---
title: Android 事件分发机制详解
categories: Android
tags:
  - Android
abbrlink: f360d385
date: 2019-01-19 11:15:26
---

`Android` 上的 `View` 是树形结构的，`View` 可能会重叠在一起，当我们点击的地方有多个 `View` 都可以响应的时候，这个点击事件应该给谁呢？事件分发机制就是为了处理这个问题的。

## 事件分发基础认知 ##
### 事件分发的对象是谁？ ###
**答：点击事件(Touch 事件)**

 - 定义：当用户触摸屏幕时(`View` 或 `ViewGroup` 派生的控件)，将产生点击事件(`Touch` 事件)。
  > `Touch` 事件相关细节（发生触摸的位置、时间、历史记录、手势动作等）被封装成 `MotionEvent` 对象

 - 主要发生的 `Touch` 事件的事件类型有如下四种：

| 事件类型                    | 具体动作                    |
|-----------------------------|-----------------------------|
| `MotionEvent.ACTION_DOWN`   | 按下 `View`(所有事件的开始) |
| `MotionEvent.ACTION_MOVE`   | 滑动 `View`                 |
| `MotionEvent.ACTION_UP`     | 抬起 `View`(与 `DOWN` 对应) |
| `MotionEvent.ACTION_CANCEL` | 非人为原因结束事件          |

 - 特别说明：事件列
   从手指接触屏幕 至 手指离开屏幕，这个过程产生的一系列事件，就叫事件列。
  > 注意：一般情况下，事件列都是以 `DOWN` 事件开始、`UP` 事件结束，中间有无数的 `MOVE` 事件，如下图：
  > ![事件列](https://henleylee.github.io/medias/touch/event_queue.png)
  即当一个 `MotionEvent` 产生后，系统需要把这个事件传递给一个具体的 `View` 去处理。

### 事件分发的本质是什么？ ###
**答：将点击事件(`MotionEvent`)传递到某个具体的 `View` 并处理的整个过程**
> 即当一个点击事件发生后，系统需要将这个事件传递给一个具体的 `View` 去处理。这个事件传递的过程就是分发过程。

### 事件在哪些对象之间进行传递？ ###
**答：`Activity`、`ViewGroup`、`View`**
> 一个事件产生后，传递顺序是：`Activity(Window) -> ViewGroup -> View`，事件分发的顺序就是事件传递的顺序。

| 对象        | 简介                                         | 备注                                                                           |
|-------------|----------------------------------------------|--------------------------------------------------------------------------------|
| `Activity`  | 控制生命周期并处理事件                       | 统筹视图的加载和显示，通过其它回调方法与 Window、View 交互                     |
| `ViewGroup` | 一组 `View` 的集合(包含多个子 `View`)        | ViewGroup 继承自 View，区别于单一 View，多了可包含子 View 和定义布局参数的功能 |
| `View`      | 所有 `UI` 组件的基类(不包含子 `View` 的组件) | 不包含子 View 的单一 View，如 TextView、ImageView、Button 等                   |

`Android` 的 `UI` 界面是由 `Activity`、`ViewGroup`、`View` 及其派生类组合而成的。`View` 是所有 `UI` 组件的基类，`ViewGroup` 是容纳 `UI` 组件的容器，即一组 `View` 的集合(包含很多子 `View` 和子 `VewGroup`)。

### 事件分发过程由哪些方法协作完成？ ###
**答：`dispatchTouchEvent()`、`onInterceptTouchEvent()` 和 `onTouchEvent()`**
![事件分发过程的方法](https://henleylee.github.io/medias/touch/event_method.png)

### 总结 ###
![事件分发基础认知总结](https://henleylee.github.io/medias/touch/event_base_summary.png)

## 事件分发机制核心方法 ##
事件分发过程由 `dispatchTouchEvent()`、`onInterceptTouchEvent()` 和 `onTouchEvent()` 三个核心方法协助完成，如下图所示：
![事件分发机制核心方法](https://henleylee.github.io/medias/touch/event_core_method.png)

### dispatchTouchEvent() ###
#### 简介 ####
![dispatchTouchEvent()方法简介](https://henleylee.github.io/medias/touch/event_dispatch_intro.png)

![dispatchTouchEvent()方法业务流程说明图](https://henleylee.github.io/medias/touch/event_dispatch_process.png)

#### 返回情况：默认 ####
![dispatchTouchEvent()方法返回情况(默认)](https://henleylee.github.io/medias/touch/event_dispatch_return_default.png)

![dispatchTouchEvent()方法返回情况(默认)业务流程说明图](https://henleylee.github.io/medias/touch/event_dispatch_process_default.png)

#### 返回情况：返回true ####
![dispatchTouchEvent()方法返回情况(返回true)](https://henleylee.github.io/medias/touch/event_dispatch_return_true.png)

![dispatchTouchEvent()方法返回情况(返回true)业务流程说明图](https://henleylee.github.io/medias/touch/event_dispatch_process_true.png)

> 事件停止分发，逐层往上返回`(若无上层返回，则结束)`；后续事件会继续分发到该 `View`。

#### 返回情况：返回false ####
![dispatchTouchEvent()方法返回情况(返回false)](https://henleylee.github.io/medias/touch/event_dispatch_return_false.png)

![dispatchTouchEvent()方法返回情况(返回false)业务流程说明图](https://henleylee.github.io/medias/touch/event_dispatch_process_false.png)

> 将事件回传给上层的 `onTouchEvent()` 处理`(若无上层返回，则结束)`；当前 `View` 仍然接受此事件的其他事件`(与 onTouchEvent() 区别)`。

### onInterceptTouchEvent() ###
#### 简介 ####
![onInterceptTouchEvent()方法简介](https://henleylee.github.io/medias/touch/event_intercept_intro.png)
> **注意：**`Activity`、`View` 都无该方法。

![onInterceptTouchEvent()方法业务流程说明图](https://henleylee.github.io/medias/touch/event_intercept_process.png)

#### 返回情况：返回true ####
![onInterceptTouchEvent()方法返回情况(返回true)](https://henleylee.github.io/medias/touch/event_intercept_return_true.png)

![onInterceptTouchEvent()方法返回情况(返回true)业务流程说明图](https://henleylee.github.io/medias/touch/event_intercept_process_true.png)

> 拦截事件，事件停止往下传递，`ViewGroup` 自己处理事件，调用父类 `super.dispatchTouchEvent()`，最终执行自己的 `onTouchEvent()`；同一个事件的其他事件列都交由该 `View` 处理；在同一个事件列中该方法不会再次被调用。

#### 返回情况：返回false(默认) ####
![onInterceptTouchEvent()方法返回情况(返回false)](https://henleylee.github.io/medias/touch/event_intercept_return_false.png)

![onInterceptTouchEvent()方法返回情况(返回false)业务流程说明图](https://henleylee.github.io/medias/touch/event_intercept_process_false.png)

> 不拦截事件，事件继续往下传递，事件传递到子 `View`，调用父类 `View.dispatchTouchEvent()` 方法中去处理；当前 `View` 仍然接受此事件的其他事件`(与 onTouchEvent() 区别)`。

### onTouchEvent() ###
#### 简介 ####
![onTouchEvent()方法简介](https://henleylee.github.io/medias/touch/event_touch_intro.png)
> **注意：**`Activity`、`View` 都无该方法。

![onTouchEvent()方法业务流程说明图](https://henleylee.github.io/medias/touch/event_touch_process.png)

#### 返回情况：返回true ####
![onTouchEvent()方法返回情况(返回true)](https://henleylee.github.io/medias/touch/event_touch_return_true.png)

![onTouchEvent()方法返回情况(返回true)业务流程说明图](https://henleylee.github.io/medias/touch/event_touch_process_true.png)

> 事件停止分发，逐层往上返回`(若无上层返回，则结束)`；后续事件序列让其处理。

#### 返回情况：返回false(默认) ####
![onTouchEvent()方法返回情况(返回false)](https://henleylee.github.io/medias/touch/event_touch_return_false.png)

![onTouchEvent()方法返回情况(返回false)业务流程说明图](https://henleylee.github.io/medias/touch/event_touch_process_false.png)

> 将事件向上传递给给上层的 `onTouchEvent()` 处理`(若无上层返回，则结束)`；当前 `View` 不再接受此事件的其他事件`(与 dispatchTouchEvent() onInterceptTouchEvent() 区别)`。

### 三者关系 ###
下面将用一段伪代码来阐述上述三个方法的关系和点击事件传递规则：
```java
/**
* 点击事件产生后
*/
// 步骤1：调用dispatchTouchEvent()
public boolean dispatchTouchEvent(MotionEvent ev) {
    // 代表是否会消费事件
    boolean consume = false;
    // 步骤2：判断是否拦截事件
    if (onInterceptTouchEvent(ev)) {
        // a. 若拦截，则将该事件交给当前View进行处理
        // 即调用onTouchEvent()方法去处理点击事件
        consume = onTouchEvent(ev);
    } else {
        // b. 若不拦截，则将该事件传递到下层
        // 即 下层元素的dispatchTouchEvent()就会被调用，重复上述过程
        // 直到点击事件被最终处理为止
        consume = child.dispatchTouchEvent(ev);
    }
    // 步骤3：最终返回通知 该事件是否被消费（接收 & 处理）
    return consume;
}
```

## 事件分发机制源码分析 ##
Android 中事件分发顺序：**`Activity(Window) -> ViewGroup -> View`**。
> 即：一个点击事件发生后，事件先传递到 `Activity`、再传递到 `ViewGroup`、最终再传递到 `View`。

因此，要想充分理解 Android 的事件分发机制，本质上是要理解：
1. `Activity` 对点击事件的分发机制
2. `ViewGroup` 对点击事件的分发机制
3. `View` 对点击事件的分发机制

### Activity 的事件分发机制 ###
当一个点击事件发生时，事件最先传到 `Activity` 的 `dispatchTouchEvent()` 方法进行事件分发。

> 具体是由 `Activity` 的 `Window` 来完成。

#### 源码分析 ####
```java
/**
* 源码分析：Activity.dispatchTouchEvent()
*/
public boolean dispatchTouchEvent(MotionEvent ev) {
    // 一般事件列开始都是DOWN事件(按下事件)，故此处基本是true
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        // ->>分析1
        onUserInteraction();
    }

    // ->>分析2
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    // ->>分析4
    return onTouchEvent(ev);
}

/**
 * 分析1：onUserInteraction()
 * 作用：实现屏保功能
 * 注意：
 * a. 该方法为空方法
 * b. 当此Activity在栈顶时，触屏点击按home、back、menu键等都会触发此方法
 */
public void onUserInteraction() {

}

/**
 * 分析2：getWindow().superDispatchTouchEvent(ev)
 * 说明：
 * a. getWindow()就是获取Window类的对象
 * b. Window类是抽象类，且PhoneWindow是Window类的唯一实现类；即此处的Window类对象就是PhoneWindow类对象
 * c. Window类的superDispatchTouchEvent()是一个抽象方法，由子类PhoneWindow类实现
 */
@Override
public boolean superDispatchTouchEvent(MotionEvent event) {
    // ->> 分析3
    // mDecor是顶层View(DecorView)的实例对象
    return mDecor.superDispatchTouchEvent(event);
}

/**
 * 分析3：mDecor.superDispatchTouchEvent(event)
 * 定义：属于顶层View(DecorView)
 * 说明：
 * a. DecorView类是PhoneWindow类的一个内部类
 * b. DecorView继承自FrameLayout，是所有界面的父类
 * c. FrameLayout是ViewGroup的子类，故DecorView的间接父类就是ViewGroup
 */
public boolean superDispatchTouchEvent(MotionEvent event) {
    // 调用父类的方法实际上就是调用ViewGroup的dispatchTouchEvent()方法
    // 即：将事件传递到ViewGroup去处理，详细请看ViewGroup的事件分发机制
    return super.dispatchTouchEvent(event);
}

/**
 * 分析4：Activity.onTouchEvent()
 * 定义：属于顶层View(DecorView)
 * 说明：
 * a. DecorView类是PhoneWindow类的一个内部类
 * b. DecorView继承自FrameLayout，是所有界面的父类
 * c. FrameLayout是ViewGroup的子类，故DecorView的间接父类就是ViewGroup
 */
public boolean onTouchEvent(MotionEvent event) {
    // 当一个点击事件未被Activity下任何一个View接收/处理时
    // 应用场景：处理发生在Window边界外的触摸事件
    // ->> 分析5
    if (mWindow.shouldCloseOnTouch(this, event)) {
        finish();
        return true;
    }

    // 即：只有在点击事件在Window边界外才会返回true，一般情况都返回false，分析完毕
    return false;
}

/**
 * 分析5：mWindow.shouldCloseOnTouch(this, event)
 */
public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
    // 主要是对于处理边界外点击事件的判断：是否是DOWN事件，event的坐标是否在边界内等
    if (mCloseOnTouchOutside && event.getAction() == MotionEvent.ACTION_DOWN && isOutOfBounds(context, event) && peekDecorView() != null) {
        return true;
    }
    // 返回true：说明事件在边界外，即 消费事件
    // 返回false：未消费(默认)
    return false;
}
```

#### 总结 ####
 - 过程：当一个点击事件发生时，从 `Activity` 的事件分发开始(`Activity.dispatchTouchEvent()`)
![Activity事件分发的过程](https://henleylee.github.io/medias/touch/event_activity_summary_process.png)

 - 核心方法总结：
![Activity事件分发的方法总结](https://henleylee.github.io/medias/touch/event_activity_summary_mathod.png)

### ViewGroup 的事件分发机制 ###
从 `Activity` 事件分发机制可知，`ViewGroup` 的事件分发机制从 `dispatchTouchEvent()` 开始。

#### 源码分析 ####
> Android 5.0 后，`ViewGroup.dispatchTouchEvent()` 的源码发生了变化(更加复杂)，但原理相同；为了更容易理解，故采用 Android 5.0 前的版本。

```java
/**
 * 源码分析：ViewGroup.dispatchTouchEvent（）
 */
public boolean dispatchTouchEvent(MotionEvent ev) {

     ...// 仅贴出关键代码

    // 重点分析1：ViewGroup每次事件分发时，都需调用onInterceptTouchEvent()询问是否拦截事件
    if (disallowIntercept || !onInterceptTouchEvent(ev)) {

        // 判断值1：disallowIntercept = 是否禁用事件拦截的功能(默认是false)，可通过调用requestDisallowInterceptTouchEvent()修改
        // 判断值2： !onInterceptTouchEvent(ev) = 对onInterceptTouchEvent()返回值取反

        // a. 若在onInterceptTouchEvent()中返回false（即不拦截事件），就会让第二个值为true，从而进入到条件判断的内部
        // b. 若在onInterceptTouchEvent()中返回true（即拦截事件），就会让第二个值为false，从而跳出了这个条件判断
        // c. 关于onInterceptTouchEvent() ->>分析1

        ev.setAction(MotionEvent.ACTION_DOWN);
        final int scrolledXInt = (int) scrolledXFloat;
        final int scrolledYInt = (int) scrolledYFloat;
        final View[] children = mChildren;
        final int count = mChildrenCount;

        // 重点分析2：通过for循环，遍历了当前ViewGroup下的所有子View
        for (int i = count - 1; i >= 0; i--) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE || child.getAnimation() != null) {
                child.getHitRect(frame);

                // 判断当前遍历的View是不是正在点击的View，从而找到当前被点击的View
                // 若是，则进入条件判断内部
                if (frame.contains(scrolledXInt, scrolledYInt)) {
                    final float xc = scrolledXFloat - child.mLeft;
                    final float yc = scrolledYFloat - child.mTop;
                    ev.setLocation(xc, yc);
                    child.mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;

                    // 条件判断的内部调用了该View的dispatchTouchEvent()
                    // 即：实现了点击事件从ViewGroup到子View的传递（具体请看下面的View事件分发机制）
                    if (child.dispatchTouchEvent(ev)) {
                        mMotionTarget = child;
                        // 调用子View的dispatchTouchEvent后是有返回值的
                        // 若该控件可点击，那么点击时，dispatchTouchEvent的返回值必定是true，因此会导致条件判断成立
                        // 于是给ViewGroup的dispatchTouchEvent()直接返回了true，即直接跳出，即把ViewGroup的点击事件拦截掉
                        return true;
                    }
                }
            }
        }
    }

    boolean isUpOrCancel = (action == MotionEvent.ACTION_UP) || (action == MotionEvent.ACTION_CANCEL);
    if (isUpOrCancel) {
        mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
    }

    final View target = mMotionTarget;
    // 重点分析3：若点击的是空白处(即无任何View接收事件)/拦截事件(手动复写onInterceptTouchEvent()，从而让其返回true)
    if (target == null) {
        ev.setLocation(xf, yf);
        if ((mPrivateFlags & CANCEL_NEXT_UP_EVENT) != 0) {
            ev.setAction(MotionEvent.ACTION_CANCEL);
            mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;
        }
        // 调用ViewGroup父类的dispatchTouchEvent()，即View.dispatchTouchEvent()
        // 因此会执行ViewGroup的onTouch() ->> onTouchEvent() ->> performClick() ->> onClick()，即自己处理该事件，事件不会往下传递
        // 具体请参考View事件的分发机制中的View.dispatchTouchEvent()
        // 此处需与上面区别：子View的dispatchTouchEvent()
        return super.dispatchTouchEvent(ev);
    }
    ...
}

/**
 * 分析1：ViewGroup.onInterceptTouchEvent()
 * 作用：是否拦截事件
 * 说明：
 * a. 返回true表示拦截，即事件停止往下传递(需手动设置，即复写onInterceptTouchEvent()，从而让其返回true)
 * b. 返回false表示不拦截(默认)
 */
public boolean onInterceptTouchEvent(MotionEvent ev) {
    if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
            && ev.getAction() == MotionEvent.ACTION_DOWN
            && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
            && isOnScrollbarThumb(ev.getX(), ev.getY())) {
        return true;
    }
    return false;
}
```

#### 总结 ####
 - 结论：Android 事件分发总是先传递到 `ViewGroup`、再传递到 `View`。
 - 过程：当点击了某个控件时
![ViewGroup事件分发的过程](https://henleylee.github.io/medias/touch/event_viewgroup_summary_process.png)

 - 核心方法总结：
![ViewGroup事件分发的方法总结](https://henleylee.github.io/medias/touch/event_viewgroup_summary_mathod.png)

### View 的事件分发机制 ###
从 `ViewGroup` 事件分发机制知道，`View` 事件分发机制从 `dispatchTouchEvent()` 开始。

#### 源码分析 ####
```java
/**
 * 源码分析：View.dispatchTouchEvent()
 */
public boolean dispatchTouchEvent(MotionEvent event) {
    // 说明：只有以下3个条件都为真，dispatchTouchEvent()才返回true；否则执行onTouchEvent()
    // 条件1. mOnTouchListener != null
    // 条件2. (mViewFlags & ENABLED_MASK) == ENABLED
    // 条件3. mOnTouchListener.onTouch(this, event)
    ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnTouchListener != null
            && (mViewFlags & ENABLED_MASK) == ENABLED
            && li.mOnTouchListener.onTouch(this, event)) {
        return true;
    }
    return onTouchEvent(event);
}

/**
 * 条件1：mOnTouchListener != null
 * 说明：mOnTouchListener变量在View.setOnTouchListener()方法里赋值
 */
public void setOnTouchListener(OnTouchListener l) {
    // 即只要我们给控件注册了Touch事件，mOnTouchListener就一定被赋值(不为空)
    getListenerInfo().mOnTouchListener = l;
}

/**
 * 条件2：(mViewFlags & ENABLED_MASK) == ENABLED
 * 说明：
 * a. 该条件是判断当前点击的控件是否enable
 * b. 由于很多View默认enable，故该条件恒定为true
 */

 /**
 * 条件3：mOnTouchListener.onTouch(this, event)
 * 说明：即 回调控件注册Touch事件时的onTouch（）；需手动复写设置，具体如下（以按钮Button为例）
 */
button.setOnTouchListener(new OnTouchListener() {
    @Override
    public boolean onTouch (View v, MotionEvent event){
        return false;
    }
});

// 若在onTouc()返回true，就会让上述三个条件全部成立，从而使得View.dispatchTouchEvent()直接返回true，事件分发结束
// 若在onTouch()返回false，就会使得上述三个条件不全部成立，从而使得View.dispatchTouchEvent()中跳出if，执行onTouchEvent(event)
```

接下来，继续进行 onTouchEvent(event) 的源码分析：
> Android 5.0 后 `View.onTouchEvent()` 源码发生了变化(更加复杂)，但原理相同；为了更容易理解，故采用 Android 5.0 前的版本。

```java
/**
 * 源码分析：View.onTouchEvent()
 */
public boolean onTouchEvent(MotionEvent event) {
    final int viewFlags = mViewFlags;
    if ((viewFlags & ENABLED_MASK) == DISABLED) {
        return (((viewFlags & CLICKABLE) == CLICKABLE || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
    }
    if (mTouchDelegate != null) {
        if (mTouchDelegate.onTouchEvent(event)) {
            return true;
        }
    }
    // 若该控件可点击，则进入switch判断中
    if (((viewFlags & CLICKABLE) == CLICKABLE || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {
        switch (event.getAction()) {
            // a. 若当前的事件 = 抬起View(主要分析)
            case MotionEvent.ACTION_UP:
                boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;
                    ...// 经过种种判断，此处省略
                // 执行performClick() ->>分析1
                performClick();
                break;
            // b. 若当前的事件 = 按下View
            case MotionEvent.ACTION_DOWN:
                if (mPendingCheckForTap == null) {
                    mPendingCheckForTap = new CheckForTap();
                }
                mPrivateFlags |= PREPRESSED;
                mHasPerformedLongPress = false;
                postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                break;
            // c. 若当前的事件 = 结束事件(非人为原因)
            case MotionEvent.ACTION_CANCEL:
                mPrivateFlags &= ~PRESSED;
                refreshDrawableState();
                removeTapCallback();
                break;
            // d. 若当前的事件 = 滑动View
            case MotionEvent.ACTION_MOVE:
                final int x = (int) event.getX();
                final int y = (int) event.getY();
                int slop = mTouchSlop;
                if ((x < 0 - slop) || (x >= getWidth() + slop) || (y < 0 - slop) || (y >= getHeight() + slop)) {
                    // Outside button
                    removeTapCallback();
                    if ((mPrivateFlags & PRESSED) != 0) {
                        // Remove any future long press/tap checks
                        removeLongPressCallback();
                        // Need to switch from pressed to not pressed
                        mPrivateFlags &= ~PRESSED;
                        refreshDrawableState();
                    }
                }
                break;
        }
        // 若该控件可点击，就一定返回true
        return true;
    } // 若该控件不可点击，就一定返回false
    return false;
}

/**
 * 分析1：performClick()
 */
public boolean performClick() {
    if (mOnClickListener != null) {
        playSoundEffect(SoundEffectConstants.CLICK);
        mOnClickListener.onClick(this);
        // 只要我们通过setOnClickListener()为控件View注册1个点击事件
        // 那么就会给mOnClickListener变量赋值（即不为空）
        // 则会往下回调onClick()并且performClick()返回true
        return true;
    }
    return false;
}
```

#### 总结 ####
 - 过程：每当控件被点击时
![View事件分发的过程](https://henleylee.github.io/medias/touch/event_view_summary_process.png)

 - 核心方法总结：
![View事件分发的方法总结](https://henleylee.github.io/medias/touch/event_view_summary_mathod.png)

## 事件分发工作流程总结 ##
![事件分发工作流程总结](https://henleylee.github.io/medias/touch/event_process_summary.png)
> 左侧虚线：具备相关性 & 逐层返回

### 以角色为核心的图解说明 ###
![以角色为核心的事件分发工作流程图解说明](https://henleylee.github.io/medias/touch/event_process_summary_role.png)

### 以方法为核心的图解说明 ###
![以方法为核心的事件分发工作流程图解说明](https://henleylee.github.io/medias/touch/event_process_summary_method.png)

## 事件分发额外知识 ##
### onTouch() 和 onTouchEvent() 的区别 ###
 - 这两个方法都是在 `View.dispatchTouchEvent()` 中调用；
 - 但 `onTouch()` 优先于 `onTouchEvent()` 执行；若手动复写在 `onTouch()` 中返回 `true`(即：将事件消费掉)，将不会再执行 `onTouchEvent()`。

> 注意：若一个控件不可点击(即非enable)，那么给它注册 `onTouch()` 事件将永远得不到执行，对于该类控件，若需监听它的 touch 事件，就必须通过在该控件中重写 `onTouchEvent()` 方法来实现，具体原因看如下代码：

```java
// &&为短路与，即如果前面条件为false，将不再往下执行，故：onTouch() 能够得到执行需2个前提条件：
// 1. mOnTouchListener的值不能为空
// 2. 当前点击的控件必须是enable的
mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED && mOnTouchListener.onTouch(this, event)
```

### Touch 事件的后续事件(MOVE、UP)层级传递 ###
 - 如果给控件注册了 `Touch` 事件，每次点击都会触发一系列 `action` 事件(`ACTION_DOWN`、`ACTION_MOVE`、`ACTION_UP` 等)；
 - 当 `dispatchTouchEvent()` 在进行事件分发的时候，只有前一个事件(如 `ACTION_DOWN`)返回 `true`，才会收到后一个事件(`ACTION_MOVE` 和 `ACTION_UP`)；
> 即如果在执行 `ACTION_DOWN` 时返回 false，后面一系列的 `ACTION_MOVE` 和 `ACTION_UP` 事件都不会执行。

从上面对事件分发机制分析知：
 - `dispatchTouchEvent()` 和 `onTouchEvent()` 消费事件、终结事件传递(返回 `true`)；
 - 而 `onInterceptTouchEvent()` 并不能消费事件，它相当于是一个分叉口起到分流导流的作用，对后续的 `ACTION_MOVE` 和 `ACTION_UP` 事件接收起到非常大的作用。
> 注意：接收了 `ACTION_DOWN` 事件的函数不一定能收到后续事件(`ACTION_MOVE`、`ACTION_UP`)。

`ACTION_MOVE` 和 `ACTION_UP` 事件的传递结论：
 - 结论1：若对象(`Activity`、`ViewGroup`、`View`)的 `dispatchTouchEvent()` 方法分发事件后消费了事件(返回 `true`)，那么收到 `ACTION_DOWN` 的函数也能收到 `ACTION_MOVE` 和 `ACTION_UP`；
> ![ACTION_MOVE和ACTION_UP事件的传递结论1](https://henleylee.github.io/medias/touch/event_action_conclusion_1.png)
> 黑线：ACTION_DOWN 事件传递方向
> 红线：ACTION_MOVE、ACTION_UP 事件传递方向

 - 结论2：若对象(`Activity`、`ViewGroup`、`View`)的 `onTouchEvent()` 方法处理了事件(返回 `true`)，那么 `ACTION_MOVE`、`ACTION_UP` 的事件从上往下传到该 `View` 后就不再往下传递，而是直接传给自己的 `onTouchEvent()` 方法并结束本次事件传递过程。
> ![ACTION_MOVE和ACTION_UP事件的传递结论2](https://henleylee.github.io/medias/touch/event_action_conclusion_2.png)
> 黑线：ACTION_DOWN 事件传递方向
> 红线：ACTION_MOVE、ACTION_UP 事件传递方向

