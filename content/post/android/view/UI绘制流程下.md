---
title: "UI绘制流程（下） 绘制的三大步骤"
author: "Gordenyou"
date: 2019-03-01T10:15:04+08:00
categories: ["技术之美"]
tags: ["Android", "View"]
lastmod: 
draft: true
summary: "其实我们实际开发中最常接触的是View绘制的三大步骤： omMeasure(), onLayout(), onDraw()。让我们对他们了解更进一步。"
---

 我们接着上次的源码分析，看看`performMeasure()`, `performLayout()`和 `performDraw()`具体做了什么。

## 测量

`ViewRootImpl.performMeasure`:

```java
private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
        if (mView == null) {
            return;
        }
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "measure");
        try {
            mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);//1
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
    }
```

`mView`是我们的根布局实例，继承自`View`，这里的 1 调用了`View.measure()`传入的参数实际上也是根布局的宽高规格。



#### View.java

`View.measure()`:

```java
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        ...
            if (cacheIndex < 0 || sIgnoreMeasureCache) {
            	...
                onMeasure(widthMeasureSpec, heightMeasureSpec);//2
                ...
            } 
		...
            
    }
```

这里检查了是否存在缓存，并继续调用了`onMeasure()`方法，实际上根布局`DecorView`继承自`FrameLayout`，所以最终调用的是`ViewGroup.onMeasure()`:

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int count = getChildCount();

        final boolean measureMatchParentChildren =
                MeasureSpec.getMode(widthMeasureSpec) != MeasureSpec.EXACTLY ||
                MeasureSpec.getMode(heightMeasureSpec) != MeasureSpec.EXACTLY;
        mMatchParentChildren.clear();

        int maxHeight = 0;
        int maxWidth = 0;
        int childState = 0;

        for (int i = 0; i < count; i++) {
            final View child = getChildAt(i);
            if (mMeasureAllChildren || child.getVisibility() != GONE) {
                measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0);//3
                final LayoutParams lp = (LayoutParams) child.getLayoutParams();
                maxWidth = Math.max(maxWidth,
                        child.getMeasuredWidth() + lp.leftMargin + lp.rightMargin);
                maxHeight = Math.max(maxHeight,
                        child.getMeasuredHeight() + lp.topMargin + lp.bottomMargin);
                childState = combineMeasuredStates(childState, child.getMeasuredState());
                if (measureMatchParentChildren) {
                    if (lp.width == LayoutParams.MATCH_PARENT ||
                            lp.height == LayoutParams.MATCH_PARENT) {
                        mMatchParentChildren.add(child);
                    }
                }
            }
        }
    ...
    }
```

这里实际上是对所有的子`View`进行了测量。上图：

![](../picture/rootView_measure.png)

`ViewGroup.measureChildWithMargins()`:

```java
protected void measureChildWithMargins(View child,
        int parentWidthMeasureSpec, int widthUsed,
        int parentHeightMeasureSpec, int heightUsed) {
    final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);//4
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

4处这里传入的参数分别为父布局的规格，`padding`和子`View`自己的尺寸。

`ViewGroup.getChildMeasureSpec`:

```java
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```

`View`的`MeasureSpec`由父容器的`MeasureSpec`和自身的`LayoutParams`来决定的。我们可以使用一个表格说明以上的关系：

![](../picture/view_measureSpec.png)

如果你觉得这里的`MeasureSpec`很神秘，可以具体来了解一下：



#### MeasureSpec

实际上是 `Measure specification` ：测量规格。

 用来保存`View`的测量**模式**和测量**尺寸**，占用32 位， 是一个`int`值。前 2 位用来保存测量模式，后30位用来保存测量尺寸。

具体的模式有三种：

```java
        /**
         * Measure specification mode: The parent has not imposed any constraint
         * on the child. It can be whatever size it wants.
         * 父容器不对View 做任何限制，系统内部使用
         */
        public static final int UNSPECIFIED = 0 << MODE_SHIFT;// MODE_SHIFT 30位

        /**
         * Measure specification mode: The parent has determined an exact size
         * for the child. The child is going to be given those bounds regardless
         * of how big it wants to be.
         * 父容器检测出 View 的大小，View 的大小就是 SpecSize。 对应match_parent, 和固定大小如：36dp
         */
        public static final int EXACTLY     = 1 << MODE_SHIFT;

        /**
         * Measure specification mode: The child can be as large as it wants up
         * to the specified size.
         * 父容器指定一个可用大小，View的大小不得超过这个值。 对应wrap_content。
         */
        public static final int AT_MOST     = 2 << MODE_SHIFT;
```

#### View和ViewGroup的测量对比：

![](../picture/ViewGroup_view_measure.png)

唯一的不同就是在`onMeasure()`方法处。



#### 为什么自定义View需要指定wrap_parent 模式的宽高？

我们来看看`View.onMeasure()`方法：

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
```

其中的`getDefaultSize()`方法做了什么？

```java
    /**
     * Utility to return a default size. Uses the supplied size if the
     * MeasureSpec imposed no constraints. Will get larger if allowed
     * by the MeasureSpec.
     *
     * @param size Default size for this view
     * @param measureSpec Constraints imposed by the parent
     * @return The size this view should be.
     */
    public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST://wrap_parent match_parent 两种模式的尺寸相同。
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }
```



## 布局

我们从`VIewRootImpl.performLayout()`开始：

```java
    private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
            int desiredWindowHeight) {

        final View host = mView;
        if (host == null) {
            return;
        }
        
        try {
            host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());//1
            ...
            }
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
        mInLayout = false;
    }
```

这里的`host`即是`mView`，也就是我们的根布局`DecorView`。在1 处调用了根布局的`layout()`方法，传入的是4个参数分别是`View`从左，上，右，下**相对于父容器的距离**：

```java
    public void layout(int l, int t, int r, int b) {
        ...

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);//2

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);//3
            ...
    }
```

我们接下来看看`setFrame()`做了什么：

```java
    protected boolean setFrame(int left, int top, int right, int bottom) {
        boolean changed = false;

        if (DBG) {
            Log.d(VIEW_LOG_TAG, this + " View.setFrame(" + left + "," + top + ","
                    + right + "," + bottom + ")");
        }

        if (mLeft != left || mRight != right || mTop != top || mBottom != bottom) {
            changed = true;

            // Remember our drawn bit
            int drawn = mPrivateFlags & PFLAG_DRAWN;

            int oldWidth = mRight - mLeft;
            int oldHeight = mBottom - mTop;
            int newWidth = right - left;
            int newHeight = bottom - top;
            boolean sizeChanged = (newWidth != oldWidth) || (newHeight != oldHeight);

            // Invalidate our old position
            invalidate(sizeChanged);

            mLeft = left;
            mTop = top;
            mRight = right;
            mBottom = bottom;
            mRenderNode.setLeftTopRightBottom(mLeft, mTop, mRight, mBottom);

            ...
        }
        return changed;
    }
```

通过传入的4个参数确定`View`在父容器中的位置。在调用完`setFrame()`之后，会调用`onLayout()`方法：

```java
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
}
```

我们发现这个方法需要我们自己实现，**所以当我们自定义一个`ViewGroup`的时候，我们需要重写此方法摆放我们的子控件。**

我们来看看`FrameLayout.onLayout()`的实现：

```java
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        layoutChildren(left, top, right, bottom, false /* no force left gravity */);
    }

    void layoutChildren(int left, int top, int right, int bottom, boolean forceLeftGravity) {
        final int count = getChildCount();

        final int parentLeft = getPaddingLeftWithForeground();
        final int parentRight = right - left - getPaddingRightWithForeground();

        final int parentTop = getPaddingTopWithForeground();
        final int parentBottom = bottom - top - getPaddingBottomWithForeground();

        for (int i = 0; i < count; i++) {
            final View child = getChildAt(i);
            if (child.getVisibility() != GONE) {
                final LayoutParams lp = (LayoutParams) child.getLayoutParams();

                final int width = child.getMeasuredWidth();
                final int height = child.getMeasuredHeight();

                int childLeft;
                int childTop;

                int gravity = lp.gravity;
                if (gravity == -1) {
                    gravity = DEFAULT_CHILD_GRAVITY;
                }

                final int layoutDirection = getLayoutDirection();
                final int absoluteGravity = Gravity.getAbsoluteGravity(gravity, layoutDirection);
                final int verticalGravity = gravity & Gravity.VERTICAL_GRAVITY_MASK;

                switch (absoluteGravity & Gravity.HORIZONTAL_GRAVITY_MASK) {
                    case Gravity.CENTER_HORIZONTAL:
                        childLeft = parentLeft + (parentRight - parentLeft - width) / 2 +
                        lp.leftMargin - lp.rightMargin;
                        break;
                    case Gravity.RIGHT:
                        if (!forceLeftGravity) {
                            childLeft = parentRight - width - lp.rightMargin;
                            break;
                        }
                    case Gravity.LEFT:
                    default:
                        childLeft = parentLeft + lp.leftMargin;
                }

                switch (verticalGravity) {
                    case Gravity.TOP:
                        childTop = parentTop + lp.topMargin;
                        break;
                    case Gravity.CENTER_VERTICAL:
                        childTop = parentTop + (parentBottom - parentTop - height) / 2 +
                        lp.topMargin - lp.bottomMargin;
                        break;
                    case Gravity.BOTTOM:
                        childTop = parentBottom - height - lp.bottomMargin;
                        break;
                    default:
                        childTop = parentTop + lp.topMargin;
                }

                child.layout(childLeft, childTop, childLeft + width, childTop + height);//4
            }
        }
    }
```

可以看出方法遍历了子控件，通过不同的设置对子控件的4个方向的参数赋值，最终调用子控件的`layout()`方法进行摆放。

总结：

对于`ViewGroup `:`layout（来确定自己的位置，4个点的位置）` --》`onLayout(进行子view的绘制)`

对于`View`: 只需要调用`layout()`来确定自身的位置即可。



## 绘制

1. 绘制背景 `drawBackground(canvas)`
2. 绘制自己 `onDraw(canvas)`
3. 绘制子`view` `dispatchDraw(canvas)`
4. 绘制前景，滚动条等装饰`onDrawForeground(canvas)`



主要来看看`ViewGroup.draw()`方法：

```java
public void draw(Canvas canvas) {
        final int privateFlags = mPrivateFlags;
        mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

        /*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         */

        // Step 1, draw the background, if needed
        int saveCount;

        drawBackground(canvas);

        // skip step 2 & 5 if possible (common case)
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            onDraw(canvas);

            // Step 4, draw the children
            dispatchDraw(canvas);

            drawAutofilledHighlight(canvas);

            // Overlay is part of the content and draws beneath Foreground
            if (mOverlay != null && !mOverlay.isEmpty()) {
                mOverlay.getOverlayView().dispatchDraw(canvas);
            }

            // Step 6, draw decorations (foreground, scrollbars)
            onDrawForeground(canvas);

            // Step 7, draw the default focus highlight
            drawDefaultFocusHighlight(canvas);

            if (debugDraw()) {
                debugDrawFocus(canvas);
            }

            // we're done...
            return;
        }
    ...
    }
```

#### 第一步 绘制背景

```java
    private void drawBackground(Canvas canvas) {
        final Drawable background = mBackground;
        if (background == null) {
            return;
        }


        final int scrollX = mScrollX;
        final int scrollY = mScrollY;
        if ((scrollX | scrollY) == 0) { //1
            background.draw(canvas);
        } else {
            canvas.translate(scrollX, scrollY);
            background.draw(canvas);
            canvas.translate(-scrollX, -scrollY);
        }
    }
```

在 1 处，可以看出绘制背景考虑了偏移参数`scrollX`和`scrollY`.如果有偏移，则会在偏移后的`canvas`绘制背景。

一般第二步和第五步都会跳过。

我们来看第三步：

#### 第三步 绘制`View`内容

```java
protected void onDraw(Canvas canvas) {
}
```

这个方法在我们自定义`View`的时候要自己实现。

#### 第四步 绘制子`View`

```
protected void dispatchDraw(Canvas canvas) {
}
```

同样需要我们自己实现，我们来看看`ViewGroup`中的实现：

```java
protected void dispatchDraw(Canvas canvas) {
        ...
    for (int i = 0; i < childrenCount; i++) {
        while (transientIndex >= 0 && mTransientIndices.get(transientIndex) == i) {
                final View transientChild = mTransientViews.get(transientIndex);
                if ((transientChild.mViewFlags & VISIBILITY_MASK) == VISIBLE ||
                        transientChild.getAnimation() != null) {
                    more |= drawChild(canvas, transientChild, drawingTime);//2
                }
                transientIndex++;
                if (transientIndex >= transientCount) {
                    transientIndex = -1;
                }
        }

     ...       
    }
```

主要也是对子`View`进行了遍历，并调用了`drawChild()`方法：

```java
protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
        return child.draw(canvas, this, drawingTime);
}
```

我们最终会发现`drawChild()`方法最终还是调用了`View.draw()`方法。从而实现了对每个子`View`的绘制。



到此我们UI绘制流程的分析就到此为止了，我们接下来看看驱动Activity的消息事件是如何分发和传递的。