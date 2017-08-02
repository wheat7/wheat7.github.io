---
layout:     post   
title:      "从零开始打造一个VR视频播放器-VRPlayer源码分析"     
date:       2017-8-1 15:30:00   
author:     "Wheat7"        
header-img: "img/post-bg-01.jpg"
--- 

### [个人小站](http://wheat7.com/)
### [Github](https://github.com/wheat7)
### [简书](http://www.jianshu.com/users/6005415e3069/)

# [项目地址](https://github.com/wheat7/VRPlayer)



# 简介
VRPlayer是一个本地VR视频播放器，整体使用了  DataBinding，MVVM架构，播放部分基于IJKPlayer，VR渲染部分基于MD360Player4Android,UI上部分使用了Carbon，沉浸式状态栏使用了 StatusBarUtil这个项目，图片加载使用 Glide      
VRPlayer会扫描你手机中的视频文件，然后你可以找到你要播放的VR视频文件，点击即可播放
# 效果

![](http://ogzwf5uv0.bkt.clouddn.com/1vr.gif)           
![](http://ogzwf5uv0.bkt.clouddn.com/2vr.gif)    

# 分析
项目主要分三部分,一是重写的MediaController，二就是播放器的包装类，也相当于我们的原生的VideoView,源码中为PlayerView,最后一部分将PlayerView和MD360Player库中的VRLibrary整合包装，也就是源码中的VRPlayerView，实现VR模式控制的接口，在使用的时候就只需要添加这一个View
本文主要分析拓展的部分，因为并不是VideoView,MediaController,或是IJk Demo的分析，所以重合部分就不作分析了，如果同学们对这一部分不熟悉，可以自行学习，
PlayerView的包装可以参考原生VideoView以及IJkPlayer的Demo，
MediaController可参考原生代码，
VRPlayerView可以参考MD360Player的Demo

播放部分根据IJkPlayer的Demo进行修改，重写MediaController，IJKPlayer的Demo也是根据原生的VideoView进行修改，但并没有自定义MediaController，原生的VideoView+MediaController想必做过视频播放的同学都比较熟悉了

```
mVideoView.setMediaController(mMediaController);
mMediaController.setMediaPlayer(mVideoView);
```

优点在于播放和控制解耦，但是原生的MediaController类可定制性很低，创建View的方法都是私有，并且用到了PhoneWindow这种系统内部才开放的类          
创建Window时使用了PhoneWindow

```
mWindow = new PhoneWindow(mContext);
```

初始化Controller的方法私有

```
private void initControllerView(View v) {
}

```

所以通过继承来自定义是不可能的，我看到的好多开源项目都是将播放以及控制写到一个View中，但是我并不认为这是一种优雅的方式，所以我们得自己来重写MediaController



## 重写MediaController
先把MediaController复制粘贴一份，所以重合部分请参考原生MediaController以及源码中的VRMediaController     
下边根据几个关键点给大家讲解分析
### 将Window改为PopWindow
* 定义

```
private PopupWindow mWindow;
```

* 初始化

```
private void initFloatingWindow() {
    mWindow = new PopupWindow(mContext);
    mWindow.setFocusable(false);
    mWindow.setBackgroundDrawable(null);
    mWindow.setOutsideTouchable(true);
    mAnimStyle = android.R.style.Animation;
    requestFocus();
    }
```

* 然后在setAnchorView(View view)方法中，把Controller的View放到PopWindow中,makeControllerView()返回的是Controller的View，下边会进行讲解


```
    public void setAnchorView(View view) {
        mAnchor = view;
        if (!mFromXml) {
            removeAllViews();
            mRoot = makeControllerView();
            mWindow.setContentView(mRoot);
            mWindow.setWidth(LayoutParams.MATCH_PARENT);
            mWindow.setHeight(LayoutParams.WRAP_CONTENT);
        }
        initControllerView(mRoot);
    }
```

下边讲解Controller的创建

## Controller的创建
* 在makeControllerView()方法中生成Controller的View

```
    protected View makeControllerView() {
        return ((LayoutInflater) mContext.getSystemService(Context.LAYOUT_INFLATER_SERVICE)).inflate(getResources().getIdentifier("media_controller_vr", "layout", mContext.getPackageName()), this);
    }
```

这里我们取得View的方式参考原生，使用getSystemService的方式获取，如果同学觉得不好，可以使用常规方式获取，原生采用这样的方式获取，我猜是为了防止包名变化，获取不到View
* View 
在源码中对应media_controller_vr.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="50dp"
              android:background="@color/mediacontroller_bg"
              android:gravity="center"
              android:orientation="vertical" >

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal" >

        <ImageView
            android:id="@+id/mediacontroller_play_pause"
            android:layout_width="30dp"
            android:layout_height="30dp"
            android:layout_centerVertical="true"
            android:layout_marginLeft="10dp"
            android:contentDescription="Play/Pause"
            android:src="@drawable/mediacontroller_pause" />

        <TextView
            android:id="@+id/mediacontroller_time_current"
            style="@style/MediaController_Text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerVertical="true"
            android:layout_marginLeft="5dp"
            android:layout_toRightOf="@id/mediacontroller_play_pause" />

        <TextView
            android:id="@+id/mediacontroller_time_total"
            style="@style/MediaController_Text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerVertical="true"
            android:layout_toLeftOf="@+id/mediacontroller_handlerLayout"
            android:layout_marginRight="5dp" />

        <android.support.v7.widget.AppCompatSeekBar
            android:id="@+id/mediacontroller_seekbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_centerVertical="true"
            android:layout_toLeftOf="@id/mediacontroller_time_total"
            android:layout_toRightOf="@id/mediacontroller_time_current"
            android:focusable="true"
            android:paddingLeft="5dp"
            android:paddingRight="5dp"
            android:thumbTint="#33cc99"
            android:progressTint="#FFFFFF"
            android:max="1000" />

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:id="@+id/mediacontroller_handlerLayout"
            android:gravity="center"
            android:layout_margin="5dp"
            android:orientation="horizontal">
            <ImageView
                android:id="@+id/mediacontroller_interactive"
                android:layout_width="30dp"
                android:layout_height="30dp"
                android:src="@drawable/ic_touch_mode"
                />

            <ImageView
                android:id="@+id/mediacontroller_display"
                android:layout_width="30dp"
                android:layout_height="30dp"
                android:layout_marginLeft="10dp"
                android:src="@drawable/ic_eye_mode"
                />

        </LinearLayout>
    </RelativeLayout>
</LinearLayout>
```

效果是这样的     
![](http://ogzwf5uv0.bkt.clouddn.com/3.png)

最后两个ImageView是我们VR控制的部分，后边要编写对应的接口

* 初始化Controller
初始化Controller和原生的并并无二致，重点是要设置相应的监听

```
private void initControllerView(View v) {
        mPauseButton = (ImageView) v.findViewById(getResources().getIdentifier("mediacontroller_play_pause", "id", mContext.getPackageName()));
        if (mPauseButton != null) {
            mPauseButton.requestFocus();
            mPauseButton.setOnClickListener(mPauseListener);
        }

        mVRInteractiveModeButton = (ImageView) v.findViewById(getResources().getIdentifier("mediacontroller_interactive", "id", mContext.getPackageName()));
        if (mVRInteractiveModeButton != null) {
            mVRInteractiveModeButton.requestFocus();
            mVRInteractiveModeButton.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {
                    mVRControl.onInteractiveClick(interactiveMode);
                    updateInteractive();
                }
            });
        }
        mVRDisplayModeButton = (ImageView) v.findViewById(getResources().getIdentifier("mediacontroller_display", "id", mContext.getPackageName()));
        if (mVRDisplayModeButton != null) {
            mVRDisplayModeButton.requestFocus();
            mVRDisplayModeButton.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {
                    mVRControl.onDisplayClick(displayMode);
                    updateDisplay();
                }
            });
        }

        mProgress = (SeekBar) v.findViewById(getResources().getIdentifier("mediacontroller_seekbar", "id", mContext.getPackageName()));
        if (mProgress != null) {
            if (mProgress instanceof SeekBar) {
                SeekBar seeker = (SeekBar) mProgress;
                seeker.setOnSeekBarChangeListener(mSeekListener);
            }
            mProgress.setMax(1000);
        }

        mEndTime = (TextView) v.findViewById(getResources().getIdentifier("mediacontroller_time_total", "id", mContext.getPackageName()));
        mCurrentTime = (TextView) v.findViewById(getResources().getIdentifier("mediacontroller_time_current", "id", mContext.getPackageName()));

        mFormatBuilder = new StringBuilder();
        mFormatter = new Formatter(mFormatBuilder, Locale.getDefault());
    }
```

这样，Controller 的View的创建过程就完成了

### Progress、show、hide
* 自定义Handler更新Progress，show、hide View

```
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            long pos;
            switch (msg.what) {
                case FADE_OUT:
                    hide();
                    break;
                case SHOW_PROGRESS:
                    pos = setProgress();
                    if (!mDragging && mShowing) {
                        msg = obtainMessage(SHOW_PROGRESS);
                        sendMessageDelayed(msg, 1000 - (pos % 1000));
                        updatePausePlay();
                    }
                    break;
            }
        }
    };
```

因为刷新View的方式改为了自定义的Handler，所以相应部分的代码要进行修改，具体情参见源码                   
并且因为Window改为了PopupWindow，在show、hide时候，要将

```    
mWindowManager.addView(mDecor, mDecorLayoutParams);
mWindowManager.removeView(mDecor);
```

改为直接操作View,即

```
setVisibility(View.VISIBLE);
setVisibility(View.GONE);
```

## 扩展
VRMediaController主要拓展了VR播放模式的控制，以及在视频上方添加一个可定制的Title的功能

* VR播放模式控制
播放模式主要是结合MD360Player库使用，提供回调接口，这里我使用了interactiveMode的INTERACTIVE_MODE_CARDBORAD_MOTION、INTERACTIVE_MODE_TOUCH，主要是切换播放时的画面控制，陀螺仪控制以及触摸控制模式；DisplayMode的MDVRLibrary.DISPLAY_MODE_GLASS模式以及MDVRLibrary.DISPLAY_MODE_NORMAL模式，GLASS模式播放双目的视频，我们可以将手机放到VR盒子里进行观看，NORMAL播放单目视频                 
我们已经在Controller布局文件里定义了两个按钮来切换这两种模式，现在在Controller中编写接口

```
    public interface VRControl {
        void onInteractiveClick(int currentMode);
        void onDisplayClick(int currentMode);
    }
```

并且在点击时，切换按钮的图标

```
    private void updateInteractive() {
        if (mRoot == null || mVRInteractiveModeButton == null)
            return;
        if (interactiveMode == VR_INTERACTIVE_MODE_GYROSCOPE) {
            interactiveMode = VR_INTERACTIVE_MODE_TOUCH;
            mVRInteractiveModeButton.setImageResource(getResources().getIdentifier("ic_gyroscope", "drawable", mContext.getPackageName()));
        }
        else {
            interactiveMode = VR_INTERACTIVE_MODE_GYROSCOPE;
            mVRInteractiveModeButton.setImageResource(getResources().getIdentifier("ic_touch_mode", "drawable", mContext.getPackageName()));
        }
    }

    private void updateDisplay() {
        if (mRoot == null || mVRDisplayModeButton == null)
            return;
        if (displayMode == VR_DISPLAY_MODE_GLASS) {
            displayMode = VR_DISPLAY_MODE_NORMAL;
            mVRDisplayModeButton.setImageResource(getResources().getIdentifier("ic_vr_mode", "drawable", mContext.getPackageName()));
        }
        else {
            displayMode = VR_DISPLAY_MODE_GLASS;
            mVRDisplayModeButton.setImageResource(getResources().getIdentifier("ic_eye_mode", "drawable", mContext.getPackageName()));
        }
    }
```

    
* Title                  
Title主要是实现在视频的上方添加一个自定义的Title，实现返回等功能的实现，并且和Controller一起show、hide,
TitleView就是一个普通的View,在使用VRPlayerView时候定义

```
 <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    <com.wheat7.vrplayer.vr.VRPlayerView
        android:id="@+id/player"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>


        <RelativeLayout
            android:id="@+id/mediacontroller_title"
            android:layout_width="match_parent"
            android:layout_height="33dp"
            android:background="@color/mediacontroller_bg"
            android:visibility="gone">

            <carbon.widget.ImageView
                android:layout_marginTop="3dp"
                android:layout_width="25dp"
                android:layout_height="25dp"
                android:clickable="true"
                android:onClick="@{()-> activity.onBackClick()}"
                android:src="@drawable/ic_back" />
        </RelativeLayout>
    </FrameLayout>
    
```


然后通过setTitleView(View v)方法传入Controller，并在show()、hide()方法中和Controller一同show、hide就可以了


## PlayerView包装
PlayerView就类似于原生的VideoView，准确的说，就是从ViedoView修改过来的，其就是一个MediaPlayer的包装类，与Mediaplayer结合，实现各种控制的回调，不了解的同学可以参考VideoView源码和 IJKPlayer的 Demo，不同的是，VideoView直接继承了SurfaceView作为播放显示的View，我们这里做了修改，继承了FrameLayout，因为播放VR视频使用的是MD360Player的OpenGl库中提供的GLSurfaceView，并通过Media的setSurfacefan方法进行设置，但是在PlayerVie中还是以addView的方式添加SurfaceView，可以通过setSurfaceView(SurfaceView surfaceView)方法传入，以便扩展，下边主要讲解扩展部分
* 扩展
扩展部分主要是添加了IJKPlayer的硬解码功能，这也是MD360Player的Demo中添加的，如果加入硬解，播放会很卡顿，并且发热量很大

```
    private void enableHardwareDecoding(){
        if (mMediaPlayer instanceof IjkMediaPlayer){
            IjkMediaPlayer player = (IjkMediaPlayer) mMediaPlayer;
            player.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "mediacodec", 1);
            player.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "mediacodec-auto-rotate", 1);
            player.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "overlay-format", IjkMediaPlayer.SDL_FCC_RV32);
            player.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "framedrop", 60);
            player.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "max-fps", 0);
            player.setOption(IjkMediaPlayer.OPT_CATEGORY_CODEC, "skip_loop_filter", 48);
        }
    }
    
```

然后在mMediaPlayer创建以后调用enableHardwareDecoding()即可



##  VRPlayerView-PlayerView与VRLibrary整合
VRPlayerView主要是将PlayerViewGLSurfaceView进行包装，并与MDVRLibraryj进行整合，实现VRMediaController.VRControl接口，控制VR播放模式，在使用时，在布局中添加一个VRPlayerView即可        
* VRPlayerView继承于FrameLayout，初始化时候，依次add GLSurfaceView、PlayerView，并且设置MediaController

```
 private void init() {
        setKeepScreenOn(true);
        mGLSurfaceView=new GLSurfaceView(getContext());
        addView(mGLSurfaceView);
        mPlayerView = new PlayerView(getContext());
        addView(mPlayerView);
        mMediaController = new VRMediaController(getContext());
        mPlayerView.setMediaController(mMediaController);
        mMediaController.setMediaPlayer(mPlayerView);
        mMediaController.setOnVRControlListener(this);
        initVRLibrary();
    }
```

* 初始化VRLibrary        
MD360Player有很多使用方法，是个强大的库，具体使用可以参见该项目,下边初始化我们用到的

```
private void initVRLibrary() {
        // new instance
        mVRLibrary = MDVRLibrary.with(getContext())
                .displayMode(MDVRLibrary.DISPLAY_MODE_GLASS)
                .interactiveMode(MDVRLibrary.INTERACTIVE_MODE_CARDBORAD_MOTION)
                .projectionMode(MDVRLibrary.PROJECTION_MODE_SPHERE)
                .pinchConfig(new MDPinchConfig().setDefaultValue(0.7f).setMin(0.5f))
                .pinchEnabled(true)
                .directorFactory(new MD360DirectorFactory() {
                    @Override
                    public MD360Director createDirector(int index) {
                        return MD360Director.builder().setPitch(90).build();
                    }
                })
                .asVideo(new MDVRLibrary.IOnSurfaceReadyCallback() {
                    @Override
                    public void onSurfaceReady(Surface surface) {
                        // IjkMediaPlayer or MediaPlayer
                        mPlayerView.getPlayer().setSurface(surface);
                    }
                })
                .build(mGLSurfaceView);
        mVRLibrary.setAntiDistortionEnabled(true);
    }

```


* 实现VRMediaController.VRControl接口

```
    @Override
    public void onInteractiveClick(int currentMode) {
        if (currentMode == MDVRLibrary.INTERACTIVE_MODE_CARDBORAD_MOTION) {
            mVRLibrary.switchInteractiveMode(getContext(), MDVRLibrary.INTERACTIVE_MODE_TOUCH);
        } else {
            mVRLibrary.switchInteractiveMode(getContext(), MDVRLibrary.INTERACTIVE_MODE_CARDBORAD_MOTION);
        }
    }

    @Override
    public void onDisplayClick(int currentMode) {
        if (currentMode == MDVRLibrary.DISPLAY_MODE_GLASS) {
            mVRLibrary.switchDisplayMode(getContext(), MDVRLibrary.DISPLAY_MODE_NORMAL);
            mVRLibrary.setAntiDistortionEnabled(false);
        } else {
            mVRLibrary.switchDisplayMode(getContext(), MDVRLibrary.DISPLAY_MODE_GLASS);
            mVRLibrary.setAntiDistortionEnabled(true);
        }
    }
```


* 包装一些方法方便调用


```
  public AbstractMediaPlayer getMediaPlayer() {
        return mPlayerView.getPlayer();
    }

    public PlayerView getPlayerView() {
        return mPlayerView;
    }

    public void setVideoPath(String path) {
        mPlayerView.setVideoPath(path);
    }

    public void setVideoUri(Uri uri) {
        mPlayerView.setVideoURI(uri);
    }

    public void setMediaControllerTitle(View v) {
        mMediaController.setTitleView(v);
    }
```


* 生命周期控制
生命周期控制是必须的，具体可以参考MD360Player

```
   public void onPause(){
        if (mVRLibrary!=null)mVRLibrary.onPause(getContext());
        if (mPlayerView!=null)mPlayerView.pause();
    }
    public void onResume(){
        if (mVRLibrary!=null)
            mVRLibrary.onResume(getContext());
        if (mPlayerView!=null)
            mPlayerView.resume();
    }
    public void onDestroy(){
        if (mVRLibrary!=null) mVRLibrary.onDestroy();
        if (mPlayerView!=null) mPlayerView.stopPlayback();
    }
```


# 使用
* 在使用的时候，在布局文件中添加VRPlayerView以及TitleView

```
 <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    <com.wheat7.vrplayer.vr.VRPlayerView
        android:id="@+id/player"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>


        <RelativeLayout
            android:id="@+id/mediacontroller_title"
            android:layout_width="match_parent"
            android:layout_height="33dp"
            android:background="@color/mediacontroller_bg"
            android:visibility="gone">

            <carbon.widget.ImageView
                android:layout_marginTop="3dp"
                android:layout_width="25dp"
                android:layout_height="25dp"
                android:clickable="true"
                android:onClick="@{()-> activity.onBackClick()}"
                android:src="@drawable/ic_back" />
        </RelativeLayout>

    </FrameLayout>
```

* 初始化 Databinding方式

```
getBinding().player.setVideoPath(urlStr);
        getBinding().player.setMediaControllerTitle(getBinding().mediacontrollerTitle);
```

* 生命周期控制

```
    @Override
    protected void onPause() {
        super.onPause();
        getBinding().player.onPause();

    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        getBinding().player.onDestroy();
    }

    @Override
    protected void onResume() {
        super.onResume();
        getBinding().player.onResume();
    }
```


# 题外话

## Databinding BaseActivity封装
对于Databinding的使用，相信同学们已经非常熟悉了，现在分享一种Databinding的BaseActivity的封装方式

```
public abstract class BaseActivity<T extends ViewDataBinding> extends AppCompatActivity {

    private View mainView;
    private ViewDataBinding binding;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        int layoutId = getLayoutId();
        super.onCreate(savedInstanceState);
        try {
            binding = DataBindingUtil.setContentView(this, layoutId);
            if (binding != null) {
                mainView = binding.getRoot();
            } else {
                mainView = LayoutInflater.from(this).inflate(layoutId, null);
                setContentView(mainView);
            }

        } catch (NoClassDefFoundError e) {
            mainView = LayoutInflater.from(this).inflate(layoutId, null);
            setContentView(mainView);
        }
        initView(savedInstanceState);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
    }


    public T getBinding() {
        return (T) binding;
    }

    public abstract int getLayoutId();

    public abstract void initView(Bundle savedInstanceState);

}

```

通过泛型参数将相应Binding类传入，然后就可以通过getBinding()方法获取对应的Binding类，通过getLayoutId()方法传入布局，在使用时在initView()中初始化Activity

## 其他
项目还包括一些其他的东西，包括Databinding的ViewHolder、欢迎界面的闪动TextView、沉浸式状态栏工具类StatusBarUtil的使用等，就不作赘述了，详见源码，如果有要和我讨论的同学，可以联系我哦

## 最后
最后恳请同学们不吝惜的给一个Star，这对于我很重要！谢谢！








