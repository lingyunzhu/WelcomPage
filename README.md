>上篇文章 ：[RxJava实践之打造酷炫启动页](http://www.jianshu.com/p/d36ebec6edab)中，我们尝试了用RxJava实现酷炫的启动页，今天我们再此基础上加入首次使用APP时的引导页功能。效果如下图：

![welcome7.gif](http://upload-images.jianshu.io/upload_images/1372557-108a456b2b60943f.gif?imageMogr2/auto-orient/strip)


思路：思路其实很简单，就是在WelcomeActivity 中setContentView（）之前判断是否是首次打开APP，若是，则去启动引导页（WelcomeGuideActivity）并return；若不是，则直接setContentView（），然后启动动画再启动MainActivity。

##一、WelcomeActivity中判断是否是第一次启动
~~~
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 判断是否是第一次开启应用
        boolean isFirstOpen = SharedPreferencesUtil.getBoolean(this, SharedPreferencesUtil.FIRST_OPEN, true);
        // 如果是第一次启动，则先进入功能引导页
        if (isFirstOpen) {
            Intent intent = new Intent(this, WelcomeGuideActivity.class);
            startActivity(intent);
            finish();
            return;
        }

        // 如果不是第一次启动app，则正常显示启动屏
        setContentView(R.layout.activity_welcome);
        ButterKnife.bind(this);
        startMainActivity();
    }
~~~
我们判断是否是第一次打开APP是用了SharedPreferences，我们这里对他进行了一下简单封装，代码如下：
~~~
/**
 * Created by xialo on 2016/7/25.
 */
public class SharedPreferencesUtil {

    private static final String spFileName = "welcomePage";
    public static final String FIRST_OPEN = "first_open";

    public static Boolean getBoolean(Context context, String strKey,
                                     Boolean strDefault) {//strDefault	boolean: Value to return if this preference does not exist.
        SharedPreferences setPreferences = context.getSharedPreferences(
                spFileName, Context.MODE_PRIVATE);
        Boolean result = setPreferences.getBoolean(strKey, strDefault);
        return result;
    }

    public static void putBoolean(Context context, String strKey,
                                  Boolean strData) {
        SharedPreferences activityPreferences = context.getSharedPreferences(
                spFileName, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = activityPreferences.edit();
        editor.putBoolean(strKey, strData);
        editor.commit();
    }

}

~~~
##二、WelcomeGuideActivity中，我们使用ViewPager以加载多个引导页面使其可以左右滑动
不多说，请看WelcomeGuideActivity.java代码：
~~~
/**
 * Created by xialo on 2016/7/25.
 */
public class WelcomeGuideActivity extends Activity implements View.OnClickListener {
    private ViewPager vp;
    private GuideViewPagerAdapter adapter;
    private List<View> views;
    private Button startBtn;

    // 引导页图片资源
    private static final int[] pics = { R.layout.guide_view1,
            R.layout.guide_view2, R.layout.guide_view3};

    // 底部小点图片
    private ImageView[] dots;

    // 记录当前选中位置
    private int currentIndex;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_guide);

        views = new ArrayList<View>();

        // 初始化引导页视图列表
        for (int i = 0; i < pics.length; i++) {
            View view = LayoutInflater.from(this).inflate(pics[i], null);

            if (i == pics.length - 1) {
                startBtn = (Button) view.findViewById(R.id.btn_enter);
                startBtn.setTag("enter");
                startBtn.setOnClickListener(this);
            }

            views.add(view);

        }

        vp = (ViewPager) findViewById(R.id.vp_guide);
        adapter = new GuideViewPagerAdapter(views);
        vp.setAdapter(adapter);
        vp.addOnPageChangeListener(new PageChangeListener());

        initDots();

    }

    @Override
    protected void onResume() {
        super.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        // 如果切换到后台，就设置下次不进入功能引导页
        SharedPreferencesUtil.putBoolean(WelcomeGuideActivity.this, SharedPreferencesUtil.FIRST_OPEN, false);
        finish();
    }

    @Override
    protected void onStop() {
        super.onStop();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
    }

    private void initDots() {
        LinearLayout ll = (LinearLayout) findViewById(R.id.ll);
        dots = new ImageView[pics.length];

        // 循环取得小点图片
        for (int i = 0; i < pics.length; i++) {
            // 得到一个LinearLayout下面的每一个子元素
            dots[i] = (ImageView) ll.getChildAt(i);
            dots[i].setEnabled(false);// 都设为灰色
            dots[i].setOnClickListener(this);
            dots[i].setTag(i);// 设置位置tag，方便取出与当前位置对应
        }

        currentIndex = 0;
        dots[currentIndex].setEnabled(true); // 设置为白色，即选中状态

    }

    /**
     * 设置当前view
     *
     * @param position
     */
    private void setCurView(int position) {
        if (position < 0 || position >= pics.length) {
            return;
        }
        vp.setCurrentItem(position);
    }

    /**
     * 设置当前指示点
     *
     * @param position
     */
    private void setCurDot(int position) {
        if (position < 0 || position > pics.length || currentIndex == position) {
            return;
        }
        dots[position].setEnabled(true);
        dots[currentIndex].setEnabled(false);
        currentIndex = position;
    }

    @Override
    public void onClick(View v) {
        if (v.getTag().equals("enter")) {
            enterMainActivity();
            return;
        }

        int position = (Integer) v.getTag();
        setCurView(position);
        setCurDot(position);
    }


    private void enterMainActivity() {
        Intent intent = new Intent(WelcomeGuideActivity.this,
                WelcomeActivity.class);
        startActivity(intent);
        SharedPreferencesUtil.putBoolean(WelcomeGuideActivity.this, SharedPreferencesUtil.FIRST_OPEN, false);
        finish();
    }

    private class PageChangeListener implements ViewPager.OnPageChangeListener {
        @Override
        public void onPageScrollStateChanged(int position) {

        }

        @Override
        public void onPageScrolled(int position, float arg1, int arg2) {

        }

        @Override
        public void onPageSelected(int position) {
            // 设置底部小点选中状态
            setCurDot(position);
        }

    }
}
~~~
我们用了三个页面guide_view1、guide_view2、guide_view3作为引导页面，布局类似，只是guide_view3多了个点击进入的Button。以下是guide_view3.xml
~~~
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scaleType="centerCrop"
        android:src="@mipmap/guide_img3" />
    <Button
        android:id="@+id/btn_enter"
        android:layout_width="100dp"
        android:layout_height="26dp"
        android:layout_gravity="bottom|center_horizontal"
        android:layout_marginBottom="75dp"
        android:background="@drawable/button_shape"
        android:text="@string/entry"
        android:textColor="@color/white"
        android:textSize="18sp"
        android:visibility="visible" />

</FrameLayout>
~~~
WelcomeGuideActivity中值得注意的是该Button点击事件的处理，在点击Button后我们并没有直接进入MainActivity，而是先把SharedPreferences中标记是否第一次进入的布尔值设为false，而后再次进入WelcomeActivity，此时WelcomeActivity会直接setContentView（）然后启动动画，进入MainActivity。

以上，我们华丽丽的引导页就完成了。
