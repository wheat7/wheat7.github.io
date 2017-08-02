---
layout:     post   
title:      "通过点击事件监听setOnClickListener彻底理解回调"     
date:       2017-8-2 12:40:00   
author:     "Wheat7"        
header-img: "img/post-bg-01.jpg"
--- 

### [个人小站](http://wheat7.com/)
### [Github](https://github.com/wheat7)
### [简书](http://www.jianshu.com/users/6005415e3069/)
## 前言
老司机们对于回调肯定熟悉得不能再熟悉了,但是新司机可能还是一脸懵逼的，我比较笨，当年懵逼了好久，看夏安明的这一篇博客[地址](http://blog.csdn.net/xiaanming/article/details/8703708)，虽然下边的留言都是，写得好！懂了懂了！但是我当时看了三遍还是不懂好吗 - -，现在我站在我的角度，用我理解的方式给大家讲解回调，我这么笨都理解了，聪明的新司机们肯定也是可以的

## setOnClickListener分析
setOnCLickLinstener,只要写过Android的同学应该都见过，大家都知道是点击事件监听，但是是怎么实现的呢？对，你没有猜错，就是回调                                    
你在onClick(View view)中写的方法，就是一个回调方法，你仔细想一想，这个方法是在你传的参数new View.OnClickListener()中的方法，你再仔细的想一想，为什么你传入了new View.OnClickListener()这个参数，Android Studio就会自动补全，让你去实现onClick(View view)这个方法呢？
一切都在你想象之中，OnClickListener就是一个接口，new出一个接口，你就得实现他里边的抽象方法，在Android中，大多数回调都是靠接口来进行的                                  
并且，你实现了onClick(View view)方法后，这个方法并没有在我们的Activity或者Fragment中调用，那为什么他生效了呢？这就是回调，你实现了他，而他却是在另一个地方调用的                
那是在什么地方调用的呢？

我们点进setOnClickListener方法中一探虚实    

于是我们跳到了View.java，原来这个方法是写在View中的，这时你想到，第一行代码中说了，我们的控件都继承于View，原来如此

```
    public void setOnClickListener(@Nullable OnClickListener l) {
        if (!isClickable()) {
            setClickable(true);
        }
        getListenerInfo().mOnClickListener = l;
    }

```
setOnClickListener方法就如同我们调用时的那样，传入一个OnClickListener对象作为参数，那我们来看一看OnClickListener是个啥子

```
    public interface OnClickListener {
        /**
         * Called when a view has been clicked.
         *
         * @param v The view that was clicked.
         */
        void onClick(View v);
    }
```

果然不出你所料，就是个interface

然后注意这一行

```
getListenerInfo().mOnClickListener = l;
```

把我们传入的OnClickListener对象赋值给了getListenerInfo().mOnClickListener，记住我们传入的OnClickListener对象就相当于携带了我们实现的onClick(View view)方法，进到View里边来了

记好了哦！

我们来看看getListenerInfo()方法

```
 ListenerInfo getListenerInfo() {
        if (mListenerInfo != null) {
            return mListenerInfo;
        }
        mListenerInfo = new ListenerInfo();
        return mListenerInfo;
    }

```
getListenerInfo()返回一个ListenerInfo，如果mListenerInfo已经存在，就返回，如果不存在，就new一个返回，也许你已经知道，或许不久后你就知道，这叫单例模式，保证只有一个ListenerInfo对象

然后我们来看看ListenerInfo又是个啥子

```
static class ListenerInfo {
        protected OnFocusChangeListener mOnFocusChangeListener;
        private ArrayList<OnLayoutChangeListener> mOnLayoutChangeListeners;
        protected OnScrollChangeListener mOnScrollChangeListener;
        private CopyOnWriteArrayList<OnAttachStateChangeListener> mOnAttachStateChangeListeners;
        public OnClickListener mOnClickListener;
         protected OnLongClickListener mOnLongClickListener;
        protected OnContextClickListener mOnContextClickListener;
        protected OnCreateContextMenuListener mOnCreateContextMenuListener;
        private OnKeyListener mOnKeyListener;
        private OnTouchListener mOnTouchListener;
        private OnHoverListener mOnHoverListener;
        private OnGenericMotionListener mOnGenericMotionListener;
        private OnDragListener mOnDragListener;
        private OnSystemUiVisibilityChangeListener mOnSystemUiVisibilityChangeListener;
        OnApplyWindowInsetsListener mOnApplyWindowInsetsListener;
    }
```

原来是一个内部静态类，成员包括各种事件的监听接口,其中包括

```
public OnClickListener mOnClickListener;

```

诶哟，和我们传入的一样的一个OnClickListener接口引用，于是绕了这么一大圈（我们先不管为啥绕），我们传入的持有我们实现的onClick(View view)方法的OnClickListener接口对象(还记得吗？)，被赋值到了View中的mListenerInfo中的mOnClickListener对象，也就是，我们实现的onCLick(View view) 方法，被mListenerInfo.mOnClickListener持有了

这时，你应该想到了，我们实现的onClick(View view)应该就是在 View中被调用了，bingo！

```
    public boolean performClick() {
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }

        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
        return result;
    }
```

从字面意思理解，这个方法就是执行Click的方法，
他将mListenerInfo对象传给了一个静态的ListenerInfo对象，
li，后边的故事大家都知道了

```
li.mOnClickListener.onClick(this);
```

这个方法执行了点击事件，并调用了我们实现的onClick(View view) 方法

让我们来梳理一遍流程，我们在Activity或者Fragment中调用

View.setOnClickListener方法，传入一个OnCLickListener对象，实现了onCLick(View view)方法，然后在View中的某个地方，我们实现的onCLick(View view)被调用，实现了回调，这就是回调的流程

## 异步
回调有什么用呢，就是异步，想象一下，系统一直在监听着屏幕的点击事件，在我们触摸到屏幕的时候进行响应，这是一个线程操作，因为如果这个放在主线程，那在事件被响应之前，我们的线程都是阻塞的，因为屏幕的资源被占用了，无法进行其他操作，而在子线程中，系统监听着屏幕的活动，然后在我们触摸时，调用performClick()方法实现了点击，并且调用了onClick(View view)方法实现了点击事件的回调，我们就可以恰恰刚好在点击时间触发的时候，进行我们想要的操作，也就是我们实现的on CLick(View view)方法

## 半伪代码实现一个回调给你看

A.class
```
//先定义一个接口
public interface Listener {
    //回调方法
    void 回调方法();
}

//申明一个接口
private Listener mLinstener;

//一个set接口的方法
public void setListener(Listener listener) {
    //把传入的listener赋值给mLinstener
    mLinstener = listener
}

... 
//在某个地方，进行某个操作的时候

private void 某个操作() {
    //回调方法执行
    mLinstener.回调方法();
}

```
另一个类 B.class

```
private A a = new A();

a.setListener(new Linstener() {
    public void 回调方法() {
        //我要在A中某个操作()执行的时候要搞的事情
        搞事情阿搞事情();
    }
});

```
然后在某个操作()调用的时候，我们的回调方法()也就被调用开始搞事情了

你如果看不懂的话，自己写一遍，这就是Android中回调的一般写法，你可以在各种自定义View中用来了，用着用着就理解了

## 为啥要绕那一圈

那一圈保证了View中只有一个mOnClickListener对象，保证了我们一次只执行一次onClick() 方法

## 最后
新司机们如果觉得有帮助，麻烦请给我的github项目点一个star
[地址点这里](https://github.com/wheat7)




