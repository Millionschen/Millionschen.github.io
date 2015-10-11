---
layout: post
title: drawerLayout以及使用drawerLayout的布局实例
category: 安卓
tags: android
---

## 什么是DrawerLayout

![drawer layout 例1](http://img.blog.csdn.net/20150119164659309)

![drawer layout 例2](http://img.blog.csdn.net/20150119164718778)

先来看看网易新闻客户端，当我们手指在屏幕左侧向右滑动时候，就会有一个抽屉式的菜单从左边弹出，并且是“悬浮”在主界面之上的，合理的利用了设备上有限的空间，同样手指在屏幕右侧向左滑动也会出现一个向左弹出的抽屉式菜单，用户体验效果还是不错的，在DrawerLayout出现之前，我们需要做侧滑菜单时，不得不自己实现一个或者使用Github上的开源的项目SlidingMenu，也许是Google也看到了SlidingMenu的强大之处，于是在Android的后期版本中添加了DrawerLayout来实现SlidingMenu同样功能的组件，而且为了兼容早期版本，将其添加在android,support.v4包下。

[关于DrawerLayout的Training](http://developer.android.com/training/implementing-navigation/nav-drawer.html)

[关于DrawerLayout的API](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.html)

### 效果预览

![效果预览](http://img.blog.csdn.net/20150129161223024)

 下面这个抽屉布局引用的是android.support.v4.DrawerLayout，类似于LineaLayout、RelativeLayout等布局一样定义，在DrawerLayout内部再定义3个布局，分别是管理主界面的FrameLayout，此布局用来展示界面切换的Fragment，下面是ListView，用来展示菜单列表，最后是一个RelativeLayout，用来展示右边的布局，布局代码如下:
 
 ```
 <android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/drawerlayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:id="@+id/content_frame"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <ListView
        android:id="@+id/left_drawer"
        android:layout_width="200dp"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:background="#111"
        android:choiceMode="singleChoice"
        android:divider="@android:color/transparent"
        android:dividerHeight="0dp" />

    <RelativeLayout
        android:id="@+id/right_drawer"
        android:layout_width="220dp"
        android:layout_height="match_parent"
        android:layout_gravity="end"
        android:background="#111"
        android:gravity="center_horizontal">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="这是右边栏"
            android:textColor="@android:color/white"
            android:textSize="24sp" />
    </RelativeLayout>
</android.support.v4.widget.DrawerLayout>
 ```
 
 这个布局文件示范了一些重要的布局特征:
 
* 主要内容的视图(FrameLayout)必须是DrawLayout的第一个子元素, 因为导航抽屉是在主要内容视图的上面.
* 主要内容视图设置为匹配父视图的宽度和高度, 因为它代表了整个界面导航抽屉是隐藏的.
* 抽屉视图(ListView)必须指定其水平重力与android:layout_gravity属性。支持从右到左(RTL)语言,指定值与 "start" 代替 "left"(所以抽屉里出现在布局的右侧当布局是RTL时).这里将ListView设置为左边栏菜单，所以android:layout_gravity属性设置为“start”，将RelativeLayout设置为右边栏，设置android:layout_gravity属性为“end”.
* 抽屉视图指定其宽度用dp单位和高度匹配父视图。抽屉里的宽度不能超过**320dp**, 所以用户总是可以看到主要内容视图的一部分。
 
 
 ## 初始化抽屉列表
 
 首先初始化这个抽屉列表，并且为这个列表适配上数据，数据适配器使用的是最简单的ArrayAdapter，模拟数据被简单的定义在res/values/strings.xml里，如下：
 
 ```
 <string-array name="menu_array">  
       <item>Menu 1</item>  
       <item>Menu 2</item>  
       <item>Menu 3</item>  
       <item>Menu 4</item>  
</string-array>  
 ```
 
 然后初始化左侧的抽屉列表：
 
 ```
mMenuTitles = getResources().getStringArray(R.array.menu_array);
ArrayAdapter<String> adapter = new
        ArrayAdapter<String>(DemoApplication.getInstance(),
                android.R.layout.simple_list_item_1,
                android.R.id.text1,
                mMenuTitles);
mLeftDrawer.setAdapter(adapter);
mLeftDrawer.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(
            AdapterView<?> parent, 
            View view, 
            int position,
            long id) {
                Toast.makeText(DemoApplication.getInstance(),
                        "I'm position " + position,
                        Toast.LENGTH_SHORT).
                        show();
                selectItem(position);
            }
        });
 ```

其中selectItem主要做了两件事:更新FragmentLayout以及设置被点选的item为select然后关闭抽屉列表

```
  private void selectItem(int position) {
        Fragment fragment = new ContentFragment();
        Bundle args = new Bundle();
        switch (position) {
            case 0:
                args.putString("key", mMenuTitles[position]);
                break;
            case 1:
                args.putString("key", mMenuTitles[position]);
                break;
            case 2:
                args.putString("key", mMenuTitles[position]);
                break;
            case 3:
                args.putString("key", mMenuTitles[position]);
                break;
            default:
                break;
        }
        fragment.setArguments(args); // FragmentActivity将点击的菜单列表标题传递给Fragment
        FragmentManager fragmentManager = getFragmentManager();
        fragmentManager.
        beginTransaction().
        replace(R.id.content_frame, fragment).
        commit();

        // 更新选择后的item和title，然后关闭菜单
        mLeftDrawer.setItemChecked(position, true);
        setTitle(mMenuTitles[position]);
        mDrawerLayout.closeDrawer(mLeftDrawer);
    }
```

## 设置actionBar以及抽屉view的打开与关闭的listener

首先，初始化actionBar

```
    getActionBar().setDisplayShowHomeEnabled(false);
    getActionBar().setDisplayHomeAsUpEnabled(true);
    getActionBar().setHomeButtonEnabled(true);
```

然后设置抽屉事件的listener:

```
mDrawerLayout.setDrawerShadow(
R.drawable.drawer_shadow,
 Gravity.LEFT);
mDrawerToggle = new ActionBarDrawerToggle(
                this,                  /* host Activity */
                mDrawerLayout,         /* DrawerLayout object */
                R.drawable.ic_drawer,  /* nav drawer image to replace 'Up' caret */
                R.string.drawer_open,  /* "open drawer" description for accessibility */
                R.string.drawer_close  /* "close drawer" description for accessibility */
        ) {
            public void onDrawerClosed(View view) {
                getActionBar().setTitle(R.string.title);
                invalidateOptionsMenu(); // creates call to onPrepareOptionsMenu()
                if (view == mRightDrawer) {
                    mRightDrawerOpen = false;
                }
            }

            public void onDrawerOpened(View drawerView) {
                if (drawerView.getId() == R.id.left_drawer) {
                    getActionBar().setTitle(R.string.left_draw);
                } else {
                    getActionBar().setTitle(R.string.right_draw);
                    mRightDrawerOpen = true;
                }
                invalidateOptionsMenu(); // creates call to onPrepareOptionsMenu()
            }
        };
        mDrawerLayout.setDrawerListener(mDrawerToggle);
        if (savedInstanceState == null) {
            selectItem(0);
        }
```

几个要注意的地方：
1. 在抽屉view打开和关闭时，为了修改actionBar，需要调用invalidateOptionsMenu，从而调用onPrepareOptionsMenu，在onPrepareOptionsMenu可以进行对actionBar上各种optiion menu的修改

```
    /* Called whenever we call invalidateOptionsMenu() */
    @Override
    public boolean onPrepareOptionsMenu(Menu menu) {
        // do something, like hide or show action items 
        return super.onPrepareOptionsMenu(menu);
    }
```

2. 在使用ActionBarDrawerToggle时，需要重写onPostCreate和onConfigurationChanged，调用ActionBarDrawerToggle的对应方法

```
  //When using the ActionBarDrawerToggle, you must call it during
  //onPostCreate() and onConfigurationChanged()     
    @Override
    protected void onPostCreate(Bundle savedInstanceState) {
        super.onPostCreate(savedInstanceState);
        // Sync the toggle state after onRestoreInstanceState has occurred.
        mDrawerToggle.syncState();
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        // Pass any configuration change to the drawer toggls
        mDrawerToggle.onConfigurationChanged(newConfig);
    }
```

3. 如果使用meterial-menu，则需要重写以下函数：

```
/** 
* 根据onPostCreate回调的状态，还原对应的icon state 
*/  
@Override  
protected void onPostCreate(Bundle savedInstanceState) {  
    super.onPostCreate(savedInstanceState);  
    mMaterialMenuIcon.syncState(savedInstanceState);  
}  
  
/** 
* 根据onSaveInstanceState回调的状态，保存当前icon state 
*/  
@Override  
protected void onSaveInstanceState(Bundle outState) {  
    mMaterialMenuIcon.onSaveInstanceState(outState);  
    super.onSaveInstanceState(outState);  
}  
```

## 自定义optionMenu的onOptionsMenuSelected事件


```
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item android:id="@+id/action_right"
        android:icon="@drawable/ic_drawer"
        android:title="右边栏"
        android:showAsAction="always"  />
</menu>
```

相关的代码：

```
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // The action bar home/up action should open or close the drawer.
        // ActionBarDrawerToggle will take care of this.
        if (mDrawerToggle.onOptionsItemSelected(item)) {

            return true;
        }

        if (item.getItemId() == R.id.action_right) {
            //操作右边栏出入
            if (mRightDrawerOpen) {
                mDrawerLayout.closeDrawer(mRightDrawer);
            } else {
                mDrawerLayout.openDrawer(mRightDrawer);
            }
            return true;
        }
        // Handle action buttons
        return super.onOptionsItemSelected(item);
    }
}  
```
对于左边栏，由mDrawerToggle的onOptionsItemSelected处理，对于右边栏，记录一个状态，根据右边栏是否打开决定点击此menu后的操作


最后，附上不使用ActionBarDrawerToggle而是使用MaterialMenuIcon的源码。当菜单没有打开时，显示“三”这样的三条横线，当菜单打开（无论左右菜单）时，会显示“<-”这样的按钮，不停的变化，这样的效果是不是有点绚丽啊？！了解过Android5.0的朋友，应该会知道这种效果是使用了Android5.0新推出的Material Design设计语言做出来的效果，那么该怎么模仿这个效果呢？不好意思，由于偷懒，我已经在牛牛的Github中找到了这样的效果——material-menu组件，该组件模拟出了Android5.0下的Material Design效果,注意的是该组件中使用了JackWharton的NineOldAndroids动画效果。

[material-menu主页](https://github.com/balysv/material-menu)

[NineOldAndroids主页](https://github.com/JakeWharton/NineOldAndroids)

关于material-menu的使用可以参考其主页上的Demo和说明，集成时需要下载NineOldAndroids导出jar集成到项目中。

```
public class MainActivity extends FragmentActivity {  
  
    /** DrawerLayout */  
    private DrawerLayout mDrawerLayout;  
    /** 左边栏菜单 */  
    private ListView mMenuListView;  
    /** 右边栏 */  
    private RelativeLayout right_drawer;  
    /** 菜单列表 */  
    private String[] mMenuTitles;  
    /** Material Design风格 */  
    private MaterialMenuIcon mMaterialMenuIcon;  
    /** 菜单打开/关闭状态 */  
    private boolean isDirection_left = false;  
    /** 右边栏打开/关闭状态 */  
    private boolean isDirection_right = false;  
    private View showView;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
  
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);  
        mMenuListView = (ListView) findViewById(R.id.left_drawer);  
        right_drawer = (RelativeLayout) findViewById(R.id.right_drawer);  
        this.showView = mMenuListView;  
  
        // 初始化菜单列表  
        mMenuTitles = getResources().getStringArray(R.array.menu_array);  
        mMenuListView.setAdapter(new ArrayAdapter<String>(this,  
                R.layout.drawer_list_item, mMenuTitles));  
        mMenuListView.setOnItemClickListener(new DrawerItemClickListener());  
  
        // 设置抽屉打开时，主要内容区被自定义阴影覆盖  
        mDrawerLayout.setDrawerShadow(R.drawable.drawer_shadow,  
                GravityCompat.START);  
        // 设置ActionBar可见，并且切换菜单和内容视图  
        getActionBar().setDisplayHomeAsUpEnabled(true);  
        getActionBar().setHomeButtonEnabled(true);  
  
        mMaterialMenuIcon = new MaterialMenuIcon(this, Color.WHITE, Stroke.THIN);  
        mDrawerLayout.setDrawerListener(new DrawerLayoutStateListener());  
  
        if (savedInstanceState == null) {  
            selectItem(0);  
        }  
  
    }  
  
    /** 
     * ListView上的Item点击事件 
     *  
     */  
    private class DrawerItemClickListener implements  
            ListView.OnItemClickListener {  
        @Override  
        public void onItemClick(AdapterView<?> parent, View view, int position,  
                long id) {  
            selectItem(position);  
        }  
    }  
  
    /** 
     * DrawerLayout状态变化监听 
     */  
    private class DrawerLayoutStateListener extends  
            DrawerLayout.SimpleDrawerListener {  
        /** 
         * 当导航菜单滑动的时候被执行 
         */  
        @Override  
        public void onDrawerSlide(View drawerView, float slideOffset) {  
            showView = drawerView;  
            if (drawerView == mMenuListView) {// 根据isDirection_left决定执行动画  
                mMaterialMenuIcon.setTransformationOffset(  
                        MaterialMenuDrawable.AnimationState.BURGER_ARROW,  
                        isDirection_left ? 2 - slideOffset : slideOffset);  
            } else if (drawerView == right_drawer) {// 根据isDirection_right决定执行动画  
                mMaterialMenuIcon.setTransformationOffset(  
                        MaterialMenuDrawable.AnimationState.BURGER_ARROW,  
                        isDirection_right ? 2 - slideOffset : slideOffset);  
            }  
        }  
  
        /** 
         * 当导航菜单打开时执行 
         */  
        @Override  
        public void onDrawerOpened(android.view.View drawerView) {  
            if (drawerView == mMenuListView) {  
                isDirection_left = true;  
            } else if (drawerView == right_drawer) {  
                isDirection_right = true;  
            }  
        }  
  
        /** 
         * 当导航菜单关闭时执行 
         */  
        @Override  
        public void onDrawerClosed(android.view.View drawerView) {  
            if (drawerView == mMenuListView) {  
                isDirection_left = false;  
            } else if (drawerView == right_drawer) {  
                isDirection_right = false;  
                showView = mMenuListView;  
            }  
        }  
    }  
  
    /** 
     * 切换主视图区域的Fragment 
     *  
     * @param position 
     */  
    private void selectItem(int position) {  
        Fragment fragment = new ContentFragment();  
        Bundle args = new Bundle();  
        switch (position) {  
        case 0:  
            args.putString("key", mMenuTitles[position]);  
            break;  
        case 1:  
            args.putString("key", mMenuTitles[position]);  
            break;  
        case 2:  
            args.putString("key", mMenuTitles[position]);  
            break;  
        case 3:  
            args.putString("key", mMenuTitles[position]);  
            break;  
        default:  
            break;  
        }  
        fragment.setArguments(args); // FragmentActivity将点击的菜单列表标题传递给Fragment  
        FragmentManager fragmentManager = getSupportFragmentManager();  
        fragmentManager.beginTransaction()  
                .replace(R.id.content_frame, fragment).commit();  
  
        // 更新选择后的item和title，然后关闭菜单  
        mMenuListView.setItemChecked(position, true);  
        setTitle(mMenuTitles[position]);  
        mDrawerLayout.closeDrawer(mMenuListView);  
    }  
  
    /** 
     * 点击ActionBar上菜单 
     */  
    @Override  
    public boolean onOptionsItemSelected(MenuItem item) {  
        int id = item.getItemId();  
        switch (id) {  
        case android.R.id.home:  
            if (showView == mMenuListView) {  
                if (!isDirection_left) { // 左边栏菜单关闭时，打开  
                    mDrawerLayout.openDrawer(mMenuListView);  
                } else {// 左边栏菜单打开时，关闭  
                    mDrawerLayout.closeDrawer(mMenuListView);  
                }  
            } else if (showView == right_drawer) {  
                if (!isDirection_right) {// 右边栏关闭时，打开  
                    mDrawerLayout.openDrawer(right_drawer);  
                } else {// 右边栏打开时，关闭  
                    mDrawerLayout.closeDrawer(right_drawer);  
                }  
            }  
            break;  
        case R.id.action_personal:  
            if (!isDirection_right) {// 右边栏关闭时，打开  
                if (showView == mMenuListView) {  
                    mDrawerLayout.closeDrawer(mMenuListView);  
                }  
                mDrawerLayout.openDrawer(right_drawer);  
            } else {// 右边栏打开时，关闭  
                mDrawerLayout.closeDrawer(right_drawer);  
            }  
            break;  
        default:  
            break;  
        }  
        return super.onOptionsItemSelected(item);  
    }  
  
    /** 
     * 根据onPostCreate回调的状态，还原对应的icon state 
     */  
    @Override  
    protected void onPostCreate(Bundle savedInstanceState) {  
        super.onPostCreate(savedInstanceState);  
        mMaterialMenuIcon.syncState(savedInstanceState);  
    }  
  
    /** 
     * 根据onSaveInstanceState回调的状态，保存当前icon state 
     */  
    @Override  
    protected void onSaveInstanceState(Bundle outState) {  
        mMaterialMenuIcon.onSaveInstanceState(outState);  
        super.onSaveInstanceState(outState);  
    }  
  
    /** 
     * 加载菜单 
     */  
    @Override  
    public boolean onCreateOptionsMenu(Menu menu) {  
        // Inflate the menu; this adds items to the action bar if it is present.  
        getMenuInflater().inflate(R.menu.main, menu);  
        return true;  
    }  
  
}  
```


