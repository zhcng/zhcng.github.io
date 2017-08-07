---
title: Material Design 实战
date: 2017-05-12 10:17:42
categories: Android
tags: 控件
---
Material Design是由Google推出的全新的设计语言。谷歌表示，这种设计语言旨在为手机、平板电脑、台式机和“其他平台”提供更一致、更广泛的“外观和感觉”。

<!-- more -->

## 简述

一直以来，大多数人都认为，Android系统的UI没有IOS的好看。所以为了保持一致，很多Android应用在设计时会直接参照IOS。但同一个系统中各个应用之间保持界面统一才能给使用者更好的用户体验。所以，Google在2014年Google I/O大会上推出了Material Design。

> Material Design中文译为材料设计语言，它更像是一套界面设计标准。它是由Google的工程师基于传统优秀的设计原则，结合丰富的创意和科学技术所发明的，包含视觉、运动、互动效果等特性。

为了推广这种统一的设计规范，Google在2015年Google I/O大会上推出了**Design Support**库，将Material Design中最具代表性的控件和效果进行了封装，使得开发者可以轻松响应这种设计规范。下面就对这些常用控件进行开发实战。

## Toolbar实现标题栏

Toolbar是为了替代ActionBar的。它不仅继承了ActionBar的所有功能，还摆脱了ActionBar只能位于活动顶部，灵活性不高的缺点，从而可以完成一些Material Design的效果。比如滚动RecyclerView时自动隐藏、可折叠式标题栏等，这些都会在后面一一介绍。

### 添加布局

新建的项目，默认自带的theme会使用DarkActionBar，我们需要在styles文件里将其修改为NoActionBar来去除掉ActionBar。

接着，在layout文件里加入Toolbar。这里用到了新的命名空间app，用来兼容5.0之前的系统，记得要在开头加上。

``` xml
xmlns:app="http://schemas.android.com/apk/res-auto"
```

``` xml
<android.support.v7.widget.Toolbar
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    android:background="?attr/colorPrimary"
    android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
    app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
```

Toolbar上的文字默认是application中指定的label，我们可以在AndroidManifest中对不同的活动指定不同的label。

``` xml
<activity
    android:name=".MainActivity"
    android:label="Picture">
```

### 添加按钮

首先在res目录下添加menu文件夹，并新建一个Menu resource file,在里面添加按钮布局。这里要注意下**showAsAction**这个属性，它一共有三种可选值。
**always**表示永远显示在Toolbar当中，**ifRoom**表示如果空间够就显示在Toolbar中，如果空间不够就显示在菜单中，**never**表示永远显示在菜单中。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/add"
        android:icon="@drawable/add"
        android:title="add"
        app:showAsAction="always"/>
    <item
        android:id="@+id/delete"
        android:icon="@drawable/delete"
        android:title="delete"
        app:showAsAction="ifRoom"/>
    <item
        android:id="@+id/favorite"
        android:icon="@drawable/favorite"
        android:title="favorite"
        app:showAsAction="never"/>
</menu>
```

接着在activity中重写**onCreateOptionsMenu**和**onOptionsItemSelected**方法，加载菜单文件并处理点击事件。

``` java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.toolbar,menu);
    return true;
}
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()){
        case R.id.add:
            Toast.makeText(this,"add click",Toast.LENGTH_SHORT).show();
            break;
        case R.id.delete:
            Toast.makeText(this,"delete click",Toast.LENGTH_SHORT).show();
            break;
        case R.id.favorite:
            Toast.makeText(this,"favorite click",Toast.LENGTH_SHORT).show();
            break;
        default:
            break;
    }
    return true;
}
```

最后运行程序，点击其中的按钮，效果如下图。

![Toolbar示例](http://opzh3oodr.bkt.clouddn.com/material-design/toolbar.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

## DawerLayout实现侧滑

DawerLayout是为了实现滑动菜单。所谓滑动菜单，所谓滑动菜单，就是将一些菜单选项隐藏起来，然后通过侧滑的方式显示。这样既节省了屏幕空间，又实现了很好的动画效果，是Material Design推荐的做法。

### 添加布局

首先在layout文件中加入DawerLayout控件，并使它包裹原有的布局，这样原有的布局就变成了主屏幕显示的内容。再在原有布局后面加一个TextView，这个TextView就是菜单的内容，我们这里只显示一个“menu”。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <RelativeLayout
        android:id="@+id/activity_main"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
    </RelativeLayout>
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:background="#fff"
        android:text="menu"
        android:textSize="30sp" />
</android.support.v4.widget.DrawerLayout>
```

DrawerLayout的第二个控件需要指定**layout_gravity**这个属性。如果是left就表示菜单在左边，如果是right就表示菜单在右边，如果是start就表示会根据系统语言做判断。比如汉语英语这种从左往右的语言环境下就在左边，阿拉伯语这种从右往左的语言环境下就在右边。end则与此相反。运行效果如下图所示。

![DrawerLayout示例一](http://opzh3oodr.bkt.clouddn.com/material-design/drawerlayout_1.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

### 添加导航按钮

上面的效果需要用户在边缘进行侧滑实现，但有时候用户并不能知道这个功能，该怎么引导他们发现这个功能呢？Material Design的建议是在Toolbar最左边加一个导航按钮，如果点击导航按钮，也可以打开侧滑菜单。这个功能直接修改activity中的代码就可以实现。将以下代码加入onCreate()方法中。

``` java
mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
ActionBar actionBar = getSupportActionBar();
if(actionBar!=null){
    actionBar.setDisplayHomeAsUpEnabled(true);
    actionBar.setHomeAsUpIndicator(R.drawable.ic_menu);
}
```

其中mDrawerLayout是一个DrawerLayout类型的全局变量，在这里得到实例，在后面会用到。再通过**getSupportActionBar()**方法得到ActionBar实例，具体实现是上面我们的Toolbar。接着调用**setDisplayHomeAsUpEnabled()**方法让导航按钮显示出来，默认是一个返回箭头。我们可以再通过**setHomeAsUpIndicator()**更改按钮的图标。

接着我们在之前重写的**onOptionsItemSelected()**方法里的switch语句下添加如下代码，使得点击导航按钮可以打开侧滑菜单。


``` java
case android.R.id.home:
    mDrawerLayout.openDrawer(GravityCompat.START);
    break;
```

运行效果如下。

![DrawerLayout示例二](http://opzh3oodr.bkt.clouddn.com/material-design/drawerlayout_2.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

## NavigationView实现菜单

之前已经实现了滑动菜单效果，但是菜单只显示一行文字。我们可以在菜单页面定义任意布局，谷歌为了方便我们，提供了NavigationView帮助我们实现Meterial Design 风格的菜单。

### 添加布局

首先，Navigation是Design Support库中所提供的控件。我们要在app/build.gradle文件的dependencies闭包中加入如下代码，来引入该库到项目中。这里还顺便引入了一个圆形图片的开源框架，供后面使用。

```
compile 'com.android.support:design:24.2.1'
compile 'de.hdodenhof:circleimageview:2.1.0' 
```

接着，在menu文件夹中，新建一个nav_menu.xml的菜单文件。加入如下代码，用来显示具体的菜单项。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">
        <item
            android:id="@+id/nav_call"
            android:icon="@drawable/nav_call"
            android:title="Call" />
        <item
            android:id="@+id/nav_friends"
            android:icon="@drawable/nav_friends"
            android:title="Friends" />
        <item
            android:id="@+id/nav_location"
            android:icon="@drawable/nav_location"
            android:title="Location" />
        <item
            android:id="@+id/nav_mail"
            android:icon="@drawable/nav_mail"
            android:title="Mail" />
        <item
            android:id="@+id/nav_task"
            android:icon="@drawable/nav_task"
            android:title="Task" />
    </group>
</menu>
```

然后，在layout文件夹中新建一个nav_header.xml文件，加入如下代码，用来显示菜单头部。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="180dp"
    android:padding="10dp"
    android:background="?attr/colorPrimary">
    <de.hdodenhof.circleimageview.CircleImageView
        android:layout_width="70dp"
        android:layout_height="70dp"
        android:src="@drawable/nav_icon"
        android:layout_centerInParent="true"/>
    <TextView
        android:id="@+id/mail"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:text="artistzheng@163.com"
        android:textColor="#fff"
        android:textSize="14sp"/>
    <TextView
        android:id="@+id/username"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/mail"
        android:text="zhcng"
        android:textColor="#fff"
        android:textSize="14sp"/>
</RelativeLayout>
```

最后，在activity_main布局文件中修改代码，删掉之前的TextView，改用NavigationView代替。通过**app:menu**和**app:headerLayout**分别加入上面的菜单和头部布局。

``` xml
<android.support.design.widget.NavigationView
    android:id="@+id/nav_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_gravity="start"
    app:menu="@menu/nav_menu"
    app:headerLayout="@layout/nav_header" />
```

### 处理点击事件

在MainActivity中的onCreate方法中加入以下代码，设置默认选中项，并处理点击事件。在这里，点击菜单则会直接关闭了侧滑菜单。

``` java
NavigationView navView = (NavigationView) findViewById(R.id.nav_view);
navView.setCheckedItem(R.id.nav_mail);
navView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
    @Override
    public boolean onNavigationItemSelected(@NonNull MenuItem item) {
        mDrawerLayout.closeDrawers();
        return true;
    }
});
```

最后的运行效果如下图所示。

![NavigationView示例](http://opzh3oodr.bkt.clouddn.com/material-design/navigationview.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

## FloatingActionButton实现悬浮按钮

立面设计是Material Design中一条非常重要的设计思想，这里面最具代表性的立面设计就是悬浮按钮了。

### 添加布局

直接修改前面的activity_main布局文件，在toolbar下面加入以下代码。

``` xml
<android.support.design.widget.FloatingActionButton
    android:id="@+id/fab"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_alignParentBottom="true"
    android:layout_alignParentRight="true"
    android:layout_margin="16dp"
    android:src="@drawable/ic_done"
    app:elevation="8dp"/>
```

这个悬浮按钮跟普通按钮没有什么其他区别，唯一不同的是在按钮下面会有一点阴影，达到悬浮的效果。一般我们用默认的就可以了，当然也可以通过**app:elevation**进行设定。这个高度值越大，投影范围越大，投影效果越淡。

### 处理点击事件

前面说过，这个按钮用起来跟普通按钮没什么区别，所以直接在activity中加入以下代码即可处理点击。

``` java
FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
fab.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Toast.makeText(MainActivity.this,"fab click",Toast.LENGTH_SHORT).show();
    }
});
```

最终效果如下图所示。

![FloatingActionButton示例](http://opzh3oodr.bkt.clouddn.com/meterial-design/floatingactionbutton.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

## Snackbar实现提示

Snackbar是Design Support库中提供的新控件，作用介于Toast和Dialog之间。Dialog的作用是给用户一个提示信息，并让用户根据提示做出判断。而Dialog的特征就是，它会阻止你原本正在进行的操作，必须停止下来对Dialog进行处理。Toast的作用是告诉用户现在发生了什么事情，不会阻挡用户的操作，但同时用户只能被动接收这个事情，因为没有什么办法来让用户是选择同意还是拒绝。Snackbar是对上面这两种状况的一个补充，使用一个动画效果从屏幕的底部弹出来，过一段时间后会自动消失，但是它上面可以加入和用户交互的按钮，提升用户体验。

### 简单使用

将之前FloatingActionButton的点击事件做如下修改。

``` java
FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
fab.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Snackbar.make(v, "data deleted", Snackbar.LENGTH_SHORT)
            .setAction("Undo", new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    Toast.makeText(MainActivity.this, "data restored", Toast.LENGTH_SHORT).show();
                }
            }).show();
    }
});
```

这里通过**make()**方法来创建一个Snackbar对象，其中第一个参数是一个View，只要是当前布局的任意View即可，会通过这个View找到最外层的布局，展示Snackbar。又通过**setAction()**方法来设置与用户交互的动作。运行效果如下图。

![Snackbar示例](http://opzh3oodr.bkt.clouddn.com/material-design/snackbar.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

这个动画过一会会自动消失，也可以通过点击Undo让其消失，点击后会弹出"data restored"的Toast。

### 结合CoordinatorLayout

上面的示例有点美中不足，Snackbar弹出时会遮挡悬浮按钮。这时候可以通过使用CoordinatorLayout来避免。

CoordinatorLayout是由Design Support库提供的，是一个加强版的FrameLayout。它可以监听其所有子控件的各种事件，然后自动帮助做出最合理的反应。所以我们修改activity_main布局文件的代码，将FloatingActionButton外面的相对布局换为**android.support.design.widget.CoordinatorLayout**即可。这是如果Snackbar再弹出来，悬浮按钮会自动上移，就不会被遮挡了，效果如下图。

![CoordinatorLayout示例](http://opzh3oodr.bkt.clouddn.com/material-design/coordinatorlayout.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

## CardView实现卡片布局

卡片式布局是Material Design中提出的一个新概念，它可以让页面中的元素看起来在卡片中一样，并且拥有圆角和投影。

### 添加布局

首先，我们要在app/build.gradle文件的dependencies闭包中加入如下代码，来引入CardView到项目中。这里还顺便引入了RecyclerView和处理图片的Glide框架，供后面使用。

```
compile 'com.android.support:cardview-v7:24.2.1'
compile 'com.android.support:recyclerview-v7:24.2.1'
compile 'com.github.bumptech.glide:glide:3.7.0'
```

CardView也是一个FrameLayout，只是额外的提供了圆角和投影的功能，使得看上去有立体感。在layout目录下创建一个picture_item的布局文件，在里面加入一个卡片式布局。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_margin="5dp"
    app:cardCornerRadius="4dp"
    app:elevation="8dp">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <ImageView
            android:id="@+id/picture"
            android:layout_width="match_parent"
            android:layout_height="100dp"
            android:scaleType="centerCrop" />
        <TextView
            android:id="@+id/pic_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_margin="5dp"
            android:textSize="16sp" />
    </LinearLayout>
</android.support.v7.widget.CardView>
```

之前说过，CardView使用起来跟Framelayout没什么区别，只是多了两个属性。**app:cardCornerRadius**属性指定卡片圆角的弧度，数值越大，弧度越大。**app:elevation**这个属性在悬浮按钮时用过，这里也差不多，用来指定卡片的高度，数值越大，高度越高，投影越大且越淡。

### 结合RecyclerView展示

首先，修改activity_main布局文件，在Toolbar下面加入RecyclerView。

``` xml
<android.support.v7.widget.RecyclerView
    android:id="@+id/recycle_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

接着，定义一个Picture实体类。

``` java
public class Picture {
    private String name;
    private int imageId;
    public Picture(String name, int imageId) {
        this.name = name;
        this.imageId = imageId;
    }
    public String getName() {
        return name;
    }
    public int getImageId() {
        return imageId;
    }
}
```

然后，新建一个PictureAdapt，作为适配器。

``` java
public class PictureAdapter extends RecyclerView.Adapter<PictureAdapter.ViewHolder> {
    private Context mContext;
    private List<Picture> mList;
    public class ViewHolder extends RecyclerView.ViewHolder {
        CardView cardView;
        ImageView pic;
        TextView picName;
        public ViewHolder(View view) {
            super(view);
            cardView = (CardView) view;
            pic = (ImageView) view.findViewById(R.id.picture);
            picName = (TextView) view.findViewById(R.id.pic_name);
        }
    }
    public PictureAdapter(List<Picture> list) {
        mList = list;
    }
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        if (mContext == null) {
            mContext = parent.getContext();
        }
        View view = LayoutInflater.from(mContext).inflate(
                R.layout.picture_item, parent, false);
        return new ViewHolder(view);
    }
    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        Picture picture = mList.get(position);
        holder.picName.setText(picture.getName());
        Glide.with(mContext).load(picture.getImageId()).into(holder.pic);
    }
    @Override
    public int getItemCount() {
        return mList.size();
    }
}
```

最后，修改MainActivity中的代码。

在类全局中加入以下代码。

``` java
private Picture[] pictures = {new Picture("autumn", R.drawable.autumn),
        new Picture("beauty", R.drawable.beauty), new Picture("cat", R.drawable.cat),
        new Picture("dog", R.drawable.dog), new Picture("night", R.drawable.night),
        new Picture("onepiece", R.drawable.onepiece), new Picture("rainbow", R.drawable.rainbow),
        new Picture("universe", R.drawable.universe)};
private List<Picture> pictureList = new ArrayList<>();
private PictureAdapter adapter;
```

再在onCreate()方法里加入以下代码。

``` java
pictureList.clear();
for (int i = 0; i < 50; i++) {
    pictureList.add(pictures[i % pictures.length]);
}
RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recycle_view);
GridLayoutManager layoutManager = new GridLayoutManager(this,2);
recyclerView.setLayoutManager(layoutManager);
adapter = new PictureAdapter(pictureList);
recyclerView.setAdapter(adapter);
```

上述代码都很简单，且与主题关联不大，只是为了让CardView更好的展示出来，所以就不展开讲了。运行效果如下图所示。

![CardView示例](http://opzh3oodr.bkt.clouddn.com/material-design/cardview.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

## AppBarLayout实现滚动封装

仔细观察上面CardView的示例，你会发现一个问题，RecyclerView把Toolbar挡住了。因为他们是放在CoordinatorLayout布局里的，这个布局只是一个特殊的FrameLayout，所以会产生遮挡。我们可以通过偏移解决上述问题，也可以在CoordinatorLayout布局中通过更巧妙的方式，实现这一效果。

AppBarLayout就是这个更巧妙的方式。它实际上是一个垂直方向的LinerLayout，在内部做了很多滚动事件的封装，并应用了一些Material Design的概念。

### 添加布局

第一步，将Toolbar嵌套到AppBarLayout中。

``` xml
<android.support.design.widget.AppBarLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
</android.support.design.widget.AppBarLayout>
```

第二步，给RecyclerView指定一个布局行为，在布局文件里的RecyclerView控件添加**app:layout_behavior="@string/appbar_scrolling_view_behavior"**属性。其中，appbar_scrolling_view_behavior这个字符串也是由Design Support库提供的。

这样，AppBarLayout就加好了，运行后发现Toolbar已经不再被遮挡了。

![AppBarLayout示例一](http://opzh3oodr.bkt.clouddn.com/material-design/appbarlayout_1.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

### Meterial Design特效

前面说到了，AppBarLayout是一个巧妙的方式，并且应用了一些Material Design的概念。下面就来体验一下。

当AppBarLayout接收到滚动事件时，它内部的子控件是可以指定如何去影响这些事件的。我们直接在Toolbar中加入**app:layout_scrollFlags="scroll|enterAlways|snap"**属性。其中scroll表示RecyclerView向上滚动时，Toolbar会一起向上滚动并隐藏；enterAlways表示RecyclerView向下滚动时，Toolbar会一起向下滚动并重新显示；snap表示当Toolbar没有完全隐藏或显示时，会根据当前滚动的距离，自动选择隐藏还是显示。

现在，向上滚动RecyclerView时，效果如下图所示。

![AppBarLayout示例二](http://opzh3oodr.bkt.clouddn.com/material-design/appbarlayout_2.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

## SwipeRefreshLayout实现下拉刷新

下拉刷新功能很多应用里都会用，样式五花八门。谷歌为了让下拉刷新有一个统一的标准，于是在Material Design中制定了一个官方的设计规范。SwipeRefreshLayout就是用于实现下拉刷新功能的核心类，是由support-v4库提供的。

### 添加布局

修改activity_main中的代码，将RecycleView的外面嵌套一层SwipeRefreshLayout，并把之前的**app:layout_behavior**属性放到外层布局，这样才能不遮挡Toolbar。

``` xml
<android.support.v4.widget.SwipeRefreshLayout
    android:id="@+id/swipe_refresh"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior">
    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycle_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</android.support.v4.widget.SwipeRefreshLayout>
```

这时，这个RecyclerView就可以支持刷新了，下拉后会出现刷新的圆圈，一直在转动。

### 处理刷新逻辑

接下来，我们再去处理具体的刷新逻辑，刷新后能够更新数据，更新完成后隐藏转动的圆圈。在activity的oncreate()方法里添加如下代码。


``` java
final SwipeRefreshLayout swipeRefreshLayout = (SwipeRefreshLayout) findViewById(R.id.swipe_refresh);
swipeRefreshLayout.setColorSchemeResources(R.color.colorPrimary);
swipeRefreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
    @Override
    public void onRefresh() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        pictureList.clear();
                        for (int i = 0; i < 50; i++) {
                            Random random = new Random();
                            int index = random.nextInt(pictures.length);
                            pictureList.add(pictures[index]);
                        }
                        adapter.notifyDataSetChanged();
                        swipeRefreshLayout.setRefreshing(false);
                    }
                });
            }
        }).start();
    }
});
```

这里调用**setColorSchemeResources()**方法，设置转动的圆圈的颜色，再调用**setOnRefreshListener()**方法设置下拉刷新的监听器，当触发下拉刷新时会回调该监听器的**onRefresh()**方法。接着，我们开启子线程，模拟网上请求数据。请求完后更新界面，并通过**setRefreshing(false)**隐藏转动的圆圈。运行效果如下图所示。

![SwipeRefreshLayout示例](http://opzh3oodr.bkt.clouddn.com/material-design/swiperefreshlayout.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

## CollapsingToolbarLayout实现折叠式标题栏

Design Support库为我们提供了CollapsingToolbarLayout，借助它我们可以实现可折叠式标题栏的效果。CollapsingToolbarLayout是一个作用于Toolbar基础之上的布局，它不能单独存在，需要作为AppBarLayout的直接子布局使用。而AppBarLayout我们前面介绍过，必须要是CoordinatorLayout的子布局。

### 添加布局

现在我们就新建一个activity_detail.xml布局文件，作为图片点击的详情页，加入以下代码。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/activity_detail"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <android.support.design.widget.AppBarLayout
        android:id="@+id/appBar"
        android:layout_width="match_parent"
        android:layout_height="250dp">
        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:contentScrim="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">
            <ImageView
                android:id="@+id/image_view"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                app:layout_collapseMode="parallax" />
            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin" />
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>
    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">
            <android.support.v7.widget.CardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="15dp"
                android:layout_marginLeft="15dp"
                android:layout_marginRight="15dp"
                android:layout_marginTop="35dp"
                app:cardCornerRadius="4dp">
                <TextView
                    android:id="@+id/content_text"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_margin="10dp"
                    android:lineSpacingExtra="8dp"
                    android:textSize="16sp" />
            </android.support.v7.widget.CardView>
        </LinearLayout>
    </android.support.v4.widget.NestedScrollView>
    <android.support.design.widget.FloatingActionButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:src="@drawable/ic_comment"
        app:layout_anchor="@id/appBar"
        app:layout_anchorGravity="bottom|end" />
</android.support.design.widget.CoordinatorLayout>
```
这里简要对上面的代码做一下说明。最外层布局是一个CoordinatorLayout布局，它有三个子布局，分别是AppBarLayout、NestedScrollView和FloatingActionButton。

其中AppBarLayout布局里面就是我们的主角--CollapsingToolbarLayout。首先给它加了一个主题，这个跟之前加在Toolbar上的主题是一样的。接着给它设定了**app:contentScrim**属性，用来指定折叠后的标题栏的颜色。最后还加了**layout_scrollFlags**属性，这个属性在之前讲AppBarLayout隐藏Toolbar时，为Toolbar加过，scroll表示CollapsingToolbarLayout会跟着内容详情一起滚动，exitUntilCollapsed表示标题栏折叠后会保留在界面上，不再移出屏幕。CollapsingToolbarLayout里面的子布局包括一个ImageView和一个Toolbar，意味着这个高级的标题栏是由图片和普通的标题栏组合而成。它们都有一个**app:layout_collapseMode**属性，parallax表示ImageView会在折叠过程中产生错位偏移，pin表示Toolbar在折叠过程中位置始终不变。

第二个子布局是NestedScrollView，它是一个加强版的ScrollView，增加了嵌套响应滚动事件的功能。由于CollapsingToolbarLayout本身可以响应滚动事件，所以内部需要使用NestedScrollView或者RecyclerView。这里同样需要指定**app:layout_behavior**属性指定布局行为。这里面用了一个CardView，来显示图片的一些详细信息。

第三个子布局是一个悬浮按钮FloatingActionButton，它不是必需的，完全可以不加，这里只是演示悬浮按钮在折叠时的动画。这里使用了**app:layout_anchor="@id/appBar"**设定了一个锚点，这样悬浮按钮就会出现在标题栏的区域里。然后再使用**app:layout_anchorGravity**属性设定gravity。

### 添加跳转及处理逻辑

接下来，我们需要修改PictureAdapter的代码，在onCreateViewHolder方法里为holder.cardView增加点击事件，打开DetailActivity并传递item的图片信息。

最后在DetailActivity中，拿到图片信息，在ImageView上显示图片，这些相关代码就不再赘述。然后将Toolbar作为ActionBar显示，启用HomeAsUp按钮，并在onOptionsItemSelected里处理了点击事件。最后调用CollapsingToolbarLayout的setTitle()方法，设置当前页面的标题。具体代码如下。

``` java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_detail);
    Intent intent = getIntent();
    String name = intent.getStringExtra("name");
    int id = intent.getIntExtra("id", 0);
    Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
    CollapsingToolbarLayout collapsingToolbarLayout =
            (CollapsingToolbarLayout) findViewById(R.id.collapsing_toolbar);
    ImageView imageView = (ImageView) findViewById(R.id.image_view);
    TextView contextText = (TextView) findViewById(R.id.content_text);
    setSupportActionBar(toolbar);
    ActionBar actionBar = getSupportActionBar();
    if (actionBar != null) {
        actionBar.setDisplayHomeAsUpEnabled(true);
    }
    collapsingToolbarLayout.setTitle(name);
    Glide.with(this).load(id).into(imageView);
    contextText.setText(name + "详情简介：\n这个界面分三个部分，标题栏、详情和悬浮按钮。\n" +
            "Toolbar和图片完美的融合到了一起，既保证了图片的展示空间，又不影响Toolbar的任何功能，比如点击箭头返回。\n" +
            "如果往上拖动内容详情，图片上的标题就会慢慢变小，并且图片会产生一些错位偏移的效果。\n" +
            "继续往上拖动，标题栏就会完全折叠,此时背景图片和悬浮按钮都会隐藏，标题栏变为最普通的Toolbar。\n" +
            "这是因为用户正在阅读具体的内容，需要为他提供最充足的阅读空间。整个动画过程浑然一体，非常美观。\n" +
            "如果这时向下拖动详情，就会执行一个完全相反的过程，最终恢复成原状。");
}
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        case android.R.id.home:
            finish();
            return true;
    }
    return super.onOptionsItemSelected(item);
}
```

运行后点击图片进入详情页，初始效果如下图所示。

![CollapsingToolbarLayout示例一](http://opzh3oodr.bkt.clouddn.com/material-design/collapsingtoolbarlayout_1.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

这时慢慢往上拖动内容详情，图片会产生错位偏移，并且标题会慢慢缩小。效果如下图所示。

![CollapsingToolbarLayout示例二](http://opzh3oodr.bkt.clouddn.com/material-design/collapsingtoolbarlayout_2.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

接着往上拖动，标题栏会完全折叠,此时背景图片和悬浮按钮都会隐藏，标题栏变为最普通的Toolbar。效果如下图所示。

![CollapsingToolbarLayout示例三](http://opzh3oodr.bkt.clouddn.com/material-design/collapsingtoolbarlayout_3.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

### 融合系统状态栏

上面的图片与系统状态栏显得有些不搭，我们可以做些优化，把他们融合到一起。

首先，需要通过**android:fitsSystemWindows="true"**，将控件显示在系统状态栏里。这里要注意，必须将要显示的控件及其所有父布局都要加上这个属性。这里从里到外，分别是ImageView、CollapsingToolbarLayout、AppBarLayout和CoordinatorLayout。

接着，要将状态栏设置为透明色。这个功能只有Android 5.0及以上手机才具备，所以需要差异型的实现。在res目录下新建一个values-v21目录，这个目录只有API 21,也就是Android 5.0以上才会读取。在该目录下新建styles.xml，创建一个style，在里面指定状态栏为透明色。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="DetailActivityTheme" parent="AppTheme">
        <item name="android:statusBarColor">@android:color/transparent</item>
    </style>
</resources>
```

在values/styles.xml里也加入这个style，不做任何处理，供Android 5.0以下手机使用。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="DetailActivityTheme" parent="AppTheme">
    </style>
</resources>
```

最后，在Manifest里面为DetailActivty指定主题为刚刚创建的style。

在Android5.0及以上手机运行效果如下图。

![CollapsingToolbarLayout示例四](http://opzh3oodr.bkt.clouddn.com/material-design/collapsingtoolbarlayout_4.png?imageMogr2/thumbnail/360x/blur/1x0/quality/75|watermark/2/text/Q2hlbidzIEJsb2c=/font/5b6u6L2v6ZuF6buR/fontsize/240/fill/IzhGMDIxNg==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)