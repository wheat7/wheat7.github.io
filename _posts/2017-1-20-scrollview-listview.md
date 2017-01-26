---
layout:     post   
title:      "ScrollView 嵌套 ListView 的简单解决方案"     
date:       2017-1-25 15:30:00   
author:     "Wheat7"        
header-img: "img/post-bg-01.jpg"

---

# ScrollView 嵌套 ListView 的简单解决方案

最近在做一个项目的时候，有一个功能需要大量的信息展示，包括字符串信息、图片、附件链接，客户想要在一个界面展示，于是我尝试了在ScrollView 嵌套 ListView，最后还是由于图片过大出现了问题，最后嵌了网页233。。其间研究了一下解决ScrollView 嵌套 ListView冲突的一些方法，选了最简单的一个，在这详细的演示一遍。            
说明：**Google不建议在ScrollView中嵌套ListView**                             
## 通过重写Listview的onMeasure方法实现动态计算ListView的高度
在ScrollView 嵌套 ListView时，如果将Listview的高度写死，保证数据填充后的高度不大于设置的高度，冲突就解决了，但是如果设置的高度大于数据填充后的高度，就会有留白。通过重写Listview的onMeasure方法实现动态计算ListView的高度。  

### 新建一个ScrollListView（类名自拟）类继承Listview 

```
public class ScrollListView extends ListView{
}
```
### 重写onMeasure方法    

```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
    super.onMeasure(widthMeasureSpec, expandSpec);
}
```

具体实现我们在这里不作了解了，毕竟官方也不建议使用。     

### 完整代码如下          

```     
public class ScrollListView extends ListView {
	public ScrollListView(Context context) {
		super(context);
		setVerticalScrollBarEnabled(true);
	}

	public ScrollListView(Context context, AttributeSet attrs, int defStyle) {
		super(context, attrs, defStyle);
		setVerticalScrollBarEnabled(true);
	}

	public ScrollListView(Context context, AttributeSet attrs) {
		super(context, attrs);
		setVerticalScrollBarEnabled(true);
	}

	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
		int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
		super.onMeasure(widthMeasureSpec, expandSpec);
	}
}
```        
      
### 要注意的是，在布局的时候，要写完整的包名进行使用
           
```        
<ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fillViewport="false" >

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_marginBottom="5dp"
            android:layout_marginLeft="5dp"
            android:layout_marginRight="5dp"
            android:layout_marginTop="5dp"
            android:orientation="vertical" >

            <com.wheat7.view.ScrollListView
                android:id="@+id/list_view"
                android:layout_width="match_parent"
                android:layout_height="wrap_content" />

            <com.wheat7.view.ScrollListView
                android:id="@+id/bx_list_view"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />


            <com.wheat7.view.ScrollListView
                android:id="@+id/image_list_view"
                android:layout_width="match_parent"
                android:layout_height="wrap_content" />
                
        </LinearLayout>
        
    </ScrollView>
```

### 在类里添加的时候，也要new ScrollListView 而不是ListView
```
private ScrollListView scrollListView;
scrollListView = (ScrollListView) findViewById (R.id.scrollListView);
```

## 几个问题
* 这个方法可以解决一些简单的问题和冲突，但是建立在损失性能上的，嵌套多个ListView的时候，还是会出现闪屏，计算错误的问题。     
* 还有其他的方法，大家可以自己试一下，如果有更好的解决方案，可以给我留言，我无耻的嵌网页了233.。
* 官方不建议使用




