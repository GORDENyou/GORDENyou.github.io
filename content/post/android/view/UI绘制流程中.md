---
title: "UI绘制流程(中) 布局绘制前奏"
author: "Gordenyou"
date: 2019-02-29T19:04:43+08:00
categories: ["技术之美"]
tags: ["Android", "View"]
lastmod: 
draft: false
summary: "在我们正式绘制自定义布局之前，Activity具体如何操作？"
---

  老规矩，先上图：

![](../piciture/ActivityThread_handlerResumeActivity.png)

界面的更新一般都在UI主线程中操作。具体实现就是在`ActivityThread.java`。让我们来看看它是如何操作的。



## ActivityThread.java

```java
public void handleResumeActivity(IBinder token, boolean finalStateRequest, boolean isForward, String reason) {

        if (r.window == null && !a.mFinished && willBeVisible) {
            r.window = r.activity.getWindow();//得到PhoneWindow
            View decor = r.window.getDecorView();//得到根布局
            decor.setVisibility(View.INVISIBLE);
            ViewManager wm = a.getWindowManager();
            WindowManager.LayoutParams l = r.window.getAttributes();
            a.mDecor = decor;
            l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
            l.softInputMode |= forwardBit;
            if (r.mPreserveWindow) {
                a.mWindowAdded = true;
                r.mPreserveWindow = false;
                // Normally the ViewRoot sets up callbacks with the Activity
                // in addView->ViewRootImpl#setView. If we are instead reusing
                // the decor view we have to notify the view root that the
                // callbacks may have changed.
                ViewRootImpl impl = decor.getViewRootImpl();//得到ViewRoot实现类
                if (impl != null) {
                    impl.notifyChildRebuilt();
                }
            }
            if (a.mVisibleFromClient) {
                if (!a.mWindowAdded) {
                    a.mWindowAdded = true;
                    wm.addView(decor, l);//1
                } 
                ...
        }
}
```

我们具体看1处，这里的`wm`是一个`ViewManager`的实例，具体是`Activity`在`PhoneWindow.class`的父类`Window.class`中获取的，而这个实例的实现是`WindowManagerImpl`，所以我们要查看`WindowManagerImpl.addView()`方法，而这个方法最终调用的是`WindowManagerGlobal.addView()`：



## WindowManagerGlobal.java

```java
public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow) {

    ...
        ViewRootImpl root;
        View panelParentView = null;

        synchronized (mLock) {
            ...
            root = new ViewRootImpl(view.getContext(), display);

            view.setLayoutParams(wparams);

            mViews.add(view);
            mRoots.add(root);
            mParams.add(wparams);

            // do this last because it fires off messages to start doing things
            try {
                root.setView(view, wparams, panelParentView);//2
            } catch (RuntimeException e) {
                // BadTokenException or InvalidDisplayException, clean up.
                if (index >= 0) {
                    removeViewLocked(index, true);
                }
                throw e;
            }
        }
    }
```

`root`是一个`ViewRootImpl.class`的一个实例。在2 处调用了`setView()`并将`DecorView`作为参数传入，这样就将`DecorView`加载到了`Window`中。但是界面仍不会显示画面，因为`View`的工作流程还没有执行完成，还需要执行`measure`, `layout`和`draw`才能将`View`绘制出来。我们看看`ViewRootImpl`中的`setView()`方法：



## ViewRootImpl.java

```java
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
        synchronized (this) {
            ...
                // Schedule the first layout -before- adding to the window
                // manager, to make sure we do the relayout before receiving
                // any other events from the system.
                requestLayout();//3
           ...
        }
    }
```

其中最关键的就是调用了`requestLayout()`,来看看它的实现：

```java
@Override
    public void requestLayout() {
        if (!mHandlingLayoutInLayoutRequest) {
            checkThread(); //在这里检查了是否在主线程
            mLayoutRequested = true;
            scheduleTraversals();//4
        }
    }
```

在这里检查了线程，并调用了`scheduleTraversals()`,最终在`TraversalRunnable`子线程中调用了`doTraversal()`：

```java
void doTraversal() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

            if (mProfile) {
                Debug.startMethodTracing("ViewAncestor");
            }

            performTraversals();//5

            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        }
    }
```

我们来看看`performTraversals()`的实现：

```java
private void performTraversals() {
        final View host = mView;
        ...
            if (!mStopped || mReportNextDraw) {
                ...
                    int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
                    int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
                    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);//6 
        }

        final boolean didLayout = layoutRequested && (!mStopped || mReportNextDraw);
        boolean triggerGlobalLayoutListener = didLayout
                || mAttachInfo.mRecomputeGlobalAttributes;
        if (didLayout) {
            performLayout(lp, mWidth, mHeight);//7
        }

        if (!cancelDraw) {
   			...
            performDraw();//8
        }
        ...
    }
```

我们终于在这里看到了`View`绘制的三大函数。

前期操作的调用探究先到此一步，我们接下来看看这三大函数的具体操作。



