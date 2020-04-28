---
title: "UI绘制流程（上）布局是如何加载的？"
author: "Gordenyou"
date: 2019-02-29T16:13:53+08:00
categories: ["技术之美"]
tags: ["Android", "View"]
lastmod: 
draft: false
summary: "从setContentView(R.layout.activity_main);入手了解UI的绘制起始过程"
---

一行我们再也熟悉不过的代码，背后究竟是什么暗箱操作？

先上图：

![](../picture/Activity_setContentView.png)

![](../picture/Activity_viewStruct.png)

这个是我们本篇的主要框架图，反映了我们Activity布局加载的框架。

现在我们从源码入手，进行探究。

## Activity.java

```java
public void setContentView(@LayoutRes int layoutResID) {
        getWindow().setContentView(layoutResID);
    }
```

这是我们的源头所在，我们已经很熟悉了。这里的`getWindow()`得到的是抽象类`Window.class`的唯一实现类`PhoneWindow`的实例，并且最终调用了它的`setContentView()`方法。



## PhoneWindow.class

```java
@Override
    public void setContentView(int layoutResID) {
        if (mContentParent == null) {
            installDecor(); //1.我们需要进去看看做了什么
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            mLayoutInflater.inflate(layoutResID, mContentParent);//2.虽然不知道installDecor()处做了什么，但我们可以在这里推断出是为了给加载布局做出准备。
        }
        ...
    }
```

我们接下来仔细看看`installDecor();`做了什么。

```java
private void installDecor() {
    mForceDecorInstall = false;
    if (mDecor == null) {
        mDecor = generateDecor(-1);//3
        
            ...
    if (mContentParent == null) {
        mContentParent = generateLayout(mDecor);//4

           ...
}
```

在`installDecor()`函数中，最重要的两件事就`generateDecor(-1)`和`generateLayout(mDecor)`。

`generateDecor(-1)`返回了一个`DecorView`实例。`DecorView`继承自`FrameLayout`。

`generateLayout(mDecor)`则将我们刚才初始化的`DecorView`进行默认的布局（`R.layout.screen_simple;`）加载(`mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);`),并且创建了我们自定义布局的容器：

```java
ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
```

这里的`contentParent`便对应我们2处的`mContentParent`。

接下来我们的自定义布局便被加载到`mContentParent`容器中。

到了这里便告一段落，接下来我们会看看我们的自定义布局在主线程中的处理过程。