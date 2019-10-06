# NestedScrollViewproblem

前言
在开发中我们经常会用到 NestedScrollView 和 RecycleView，一般情况下这两种布局是不需要进行嵌套的，很多情况下 RecycleView 就可以自行解决，但是毕竟是一般情况，因此超出一般情况外的，我们可能就需要进行嵌套了，虽然 Google 大大也不鼓励我们这样使用。
这样使用可能会带来一些问题，一如当年的 ScrollView 和 ListView 的矛盾一样。这里就出现的一些情况进行总结和解决。
开发中碰到的类似的问题，都会放在这里，持续更新中……


出现的问题

问题一：
NestedScrollView 和 RecycleView 嵌套时出现滑动卡顿的情况
解决方法：
    LinearLayoutManager linearLayoutManager = new LinearLayoutManager(this);
    linearLayoutManager.setSmoothScrollbarEnabled(true);
    linearLayoutManager.setAutoMeasureEnabled(true);
    recyclerView.setHasFixedSize(true);
    recyclerView.setNestedScrollingEnabled(false);
    recyclerView.setLayoutManager(linearLayoutManager);

这个是 LinearLayoutManager 实现 GridView 的效果，是一样的设置，只是换成了 GridLayoutManager 即可。
其中 recyclerView.setNestedScrollingEnabled(false); 设置可以在 xml 文件中间进行设置：android:nestedScrollingEnabled="false"
同样我使用这种方法，一般可以解决滑动冲突的问题，也有一些网上重写 LinearLayoutManager 的方式，根据需要进行选择吧，如果以上方法无法满足，则可以选择重写的方式，这个网上的资料很多，这里就不再重复。
问题二：
在 NestedScrollView 嵌套 RecycleView 时，可能会出现首次进入页面，页面的位置不在最顶部的问题。有可能是停在了 RecycleView 的部位。
解决方法：
在 NestedScrollView 唯一子布局中加入
android:descendantFocusability=“blocksDescendants”

android:descendantFocusability 有三个属性
beforeDescendants：优先于子控件获取焦点
afterDescendants：当子控件不需要焦点时，获取焦点
blocksDescendants：覆盖所有子控件获取焦点

问题三：
在使用 NestedScrollView 作为父布局的时候，子布局的嵌套问题
解决方法：
使用 NestedScrollView 和 ScrollView 是一样的，其子布局是唯一的，不然在运行时会报错。因此在使用的时候要设置唯一的子布局进行展示。
问题四：
NestedScrollView 嵌套 ViewPager 导致 ViewPager 显示不出来的问题
解决方法：
在 NestedScrollView 的布局中加入下面代码
android:fillViewport="true"

问题五：
使用 ViewPager 嵌套 Fragment 时默认的我们使用 PagerAdapter／FragmentPagerAdapter／FragmentStatePagerAdapter 来管理 Fragment，这时会默认加载的 Fragment 的个数时系统默认的 DEFAULT_OFFSCREEN_PAGES = 1，即显示一个，预加载一个，如果我们想控制加载的 Fragment 的个数需要如何处理？
解决方法：
可以通过设置 ViewPager 来进行管理
mViewPager.setOffscreenPageLimit(num);

设置这个属性后，系统会加载相应的 Fragment 的个数，但是运行时会 Crash 掉，提示的下面的错误：
java.lang.IllegalStateException: FragmentManager is already executing transactions

解决方法：
mViewPager = new MyViewPager(getFragmentManager());
将 getFragmentManager() 替换为 getChildFragmentManager();

FragmentManager is already executing transactions.
问题六：
场景：当 ViewPager1 嵌套 Fragment0、Fragment0、Fragment1、Fragment0，在其中的 Fragment1 中又有一个 ViewPager2，在 ViewPager2 中嵌套 Fragment2，这时当我们滑倒 Fragment1 的时候，由于是第一次滑动到这个位置，Fragment2 正常显示，但是当我们滑动到其他位置，再返回到 Fragment1 时，就会出现 Fragment1 显示空白的问题，日志显示，正常加载。
解决方法：
1、跟产品提出将第二层的 Fragment 提到与第一层同级上去（如果产品同意的话）。
2、按照下面设置：
mViewPager = new MyViewPager(getFragmentManager());
将 getFragmentManager() 替换为 getChildFragmentManager();

问题七：
getFragmentManager()、getChildFragmentManager()、getSupportFragmentManager() 的区别：
解决方法：
getSupportFragmentManager(): 主要用于支持 3.0以下 android 系统
API 版本，3.0以上系统可以直接调用 getFragmentManager() ，因为
fragment是3.0以后才出现的组件，为了这之前的系统版本也能使用
fragment,借助V4包里面的 getSupportFragmentManager() 方法来间接获取 FragmentManager() 对象，3.0版本之后，有了 Fragment 的 api，就可以直接使用 getFragmentManager() 这个方法来获取对象。
getFragmentManager(): 所得到的是所在 fragment 的父容器的管理器
getChildFragmentManager(): 所得到的是在 fragment 里面子容器的管理器
因此当我们使用 Fragment 嵌套 Fragment 时，应该使用 getChildFragmentManager()
可能会出现的类似的问题又：
1、Fragment低频率点击切换不会发生问题，过快点击马上崩溃
2、错误：Java.lang.IllegalArgumentException：No view found for id for fragment
3、调用fragment的replace方法不走onDestroy()、onDestroyView()方法，无法销毁fragment
4、在fragment中写倒计时，每次切换后倒计时越来越快的问题
参考链接
问题八：
当点击上层布局结果下面的布局同时响应点击事件
解决方法：
在上层根布局中添加下面代码：
android:clickable="true"

这样可以使上层布局拦截点击事件，不会让下面的布局获取到。
问题九：
键盘问题
解决方法：
关于软键盘弹出的各个属性简介：
在manifest文件中可以设置Activity的android:windowSoftInputMode属性，这个属性值常见的设置如下：
android:windowSoftInputMode="stateAlwaysHidden|adjustPan"
那么这里值的含义列表如下：
【A】stateUnspecified：软键盘的状态并没有指定，系统将选择一个合适的状态或依赖于主题的设置
【B】stateUnchanged：当这个activity出现时，软键盘将一直保持在上一个activity里的状态，无论是隐藏还是显示
【C】stateHidden：用户选择activity时，软键盘总是被隐藏
【D】stateAlwaysHidden：当该Activity主窗口获取焦点时，软键盘也总是被隐藏的
【E】stateVisible：软键盘通常是可见的
【F】stateAlwaysVisible：用户选择activity时，软键盘总是显示的状态
【G】adjustUnspecified：默认设置，通常由系统自行决定是隐藏还是显示
【H】adjustResize：该Activity总是调整屏幕的大小以便留出软键盘的空间
【I】adjustPan：当前窗口的内容将自动移动以便当前焦点从不被键盘覆盖和用户能总是看到输入内容的部分

后记
这里只是记录了几种自己开发中碰到的问题，会持续更新这篇帖子，如果有人遇到过其他问题，欢迎提问。
