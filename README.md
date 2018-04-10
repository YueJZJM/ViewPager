# ViewPager 学习

学习了 ViewPager，在此做个总结，主要包括一下几个方面：
- ViewPager 简介
- ViewPager 的使用
- FragmentStatePagerAdapter 和 FragmentPagerAdapter
- ViewPager 的工作原理
- 恢复 CrimeFragment 的边距
- 添加 Jump to First 按钮和 Jump to Last 按钮
## ViewPager 简介
1.  ViewPager 是 android 扩展包 v4 包中的类，这个类可以让用户左右切换当前的 view
2. ViewPager 直接继承了 ViewGroup，所有它是一个容器类，可以在其中添加其他的 view 类。
3.  ViewPager 需要一个 PagerAdapter 适配器类给它提供数据。
4. ViewPager 经常和 Fragment 一起使用，并且提供了专门的 FragmentPagerAdapter 和 FragmentStatePagerAdapter 类供 Fragment 中的 ViewPager 使用。

## ViewPager 的使用
因为之前的封装，CrimeFragment 类可以不做修改直接使用。接下来看我们需要完成的任务。
1. 创建 CrimePagerActivity 类
2. 在 CrimePagerActivity 类中关联使用 ViewPager 及其 Adapter
3. 修改 CrimeHolder.onClick(...) 方法，转而启动 CrimePagerActivity

#### 创建 CrimePagerActivity 类

**CrimePagerActivity.java**
```
public class CrimePagerActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_crime_pager);
    }
```
布局文件

**activity_crime_pager.xml**
```
<android.support.v4.view.ViewPager xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/crime_view_pager"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".CrimePagerActivity">
</android.support.v4.view.ViewPager>
```
####  在 CrimePagerActivity 类中关联使用 ViewPager 及其 Adapter
ViewPager  某种程度上与 RecyclerView 类似，都需要借助 Adapter 提供视图。因为 ViewPager 与 PagerAdapter 间的配合要复杂很多，这里先用 FragmentPagerAdapter，它能处理很多细节的问题

FragmentPagerAdapter 提供了两个有用的方法：getCount() 和 getItem(int)。调用 getItem(int) 方法，获取并显示 crime 数组中指定位置的 Crime 时，它会返回配置过的 CrimeFragment 来显示指定的 Crime。
**CrimePagerActivity.java**
```
public class CrimePagerActivity extends AppCompatActivity {

    private ViewPager mViewPager;
    private List<Crime> mCrimes;
    
     @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_crime_pager);

        mViewPager = (ViewPager) findViewById(R.id.crime_view_pager);

        mCrimes  =CrimeLab.get(this).getCrimes();
        FragmentManager fragmentManager = getSupportFragmentManager();
        mViewPager.setAdapter(new FragmentStatePagerAdapter(fragmentManager) {
            @Override
            public Fragment getItem(int position) {
                Crime crime = mCrimes.get(position);
                return CrimeFragment.newInstance(crime.getId());
            }
            @Override
            public int getCount() {
                return mCrimes.size();
            }
        });
```
解释一下上面的代码，在 activity 视图中找到 ViewPager 后，我们从 CrimeLab 获取数据集，然后获取 FragmentManager 的实例。接下来，设置 adapter 为 FragmentStatePagerAdapter 的一个匿名实例。创建 FragmentStatePagerAdapter 需要 FragmentManager。如前所述，qiansuoshuFragmentStatePagerAdapter 是我们的代理，代理首先将 getItem(int) 方法返回的 fragment 添加给 activity，然后才能使用 fragment 完成自己的工作。

代理究竟做了哪些工作呢？简单来说就是将返回的 fragment 添加给托管 activity，并帮助 ViewPager 找到 fragment 的视图一一对应。getItem(int) 方法首先获取指定位置的 Crime 实例，然后利用该 Crime 实例的 ID 创建并返回一个经过有效配置发 CrimeFragment。 
#### 修改 CrimeHolder.onClick(...) 方法，转而启动 CrimePagerActivity
**CrimePagerActivity.java**
```
 private static final String EXTRA_CRIME_ID = "com.bignerdranch.android.criminalintent.crime_id";

    private ViewPager mViewPager;
    private List<Crime> mCrimes;

    public static Intent newIntent(Context packageContext, UUID crimeId){
        Intent intent = new Intent(packageContext,CrimePagerActivity.class);
        intent.putExtra(EXTRA_CRIME_ID,crimeId);
        return intent;
    }
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_crime_pager);
        
        UUID crimeId = (UUID)getIntent().getSerializableExtra(EXTRA_CRIME_ID) ;
        ...
    }

```
#### 修改 CrimeHolder.onClick(...) 方法，转而启动 CrimePagerActivity
**CrimePagerActivity.java**
```
    @Override
    public void onClick(View view) {
       // Intent intent = CrimeActivity.newIntent(getActivity(),mCrime.getId());
        Intent intent = CrimePagerActivity.newIntent(getActivity(),mCrime.getId());
        mPosition = this.getAdapterPosition();
        startActivity(intent);
    }
```
最后在 AndroidManifest.xml中删除 CrimeActivity 的代码。

注意，目前 ViewPager 还不完美，ViewPager 默认只显示 PagerAdapter 中的第一个列表项。要显示选中的列表项需要在 CrimePagerActivity.onCreate(Bundle) 末尾添加以下代码。

**CrimePagerActivity.java**
```

for (int i = 0; i < mCrimes.size(); i++){
    if (mCrimes.get(i).getId().equals(crimeId)){
        mViewPager.setCurrentItem(i);
        break;
    }
}
```
## FragmentStatePagerAdapter 和 FragmentPagerAdapter
FragmentPagerAdapter 是另一种可用的 PagerAdapter，其用法和 FragmentStatePagerAdapter 基本一致，只是在卸载不需要的 fragment 时，各自采用的处理方法不同。

FragmentStatePagerAdapter 会销毁不需要的 fragment，而 FragmentPagerAdapter 是调用 detach(Fragment) 方法来处理它，只是销毁了 fragment 的视图，而 fragment 的实例由 FragmentManager 维护，因此，FragmentPagerAdapter 创建的 fragment 永远不会被销毁。

所以当数据量大时，可以选择 FragmentStatePagerAdapter，用户界面只有少量固定的 fragment 时，可以选择 FragmentPagerAdapter。

## ViewPager 的工作原理

首先明确一点，要实现自己的 PagerAdapter 接口时，就需要了解它的工作原理，那么什么时候需要实现 PagerAdapter 接口呢？当需要托管非 fragment 视图时（如图片），就需要实现原生的 PagerAdapter。

为什么使用 ViewPager而不是 RecyclerView？

Adapter 需要我们及时提供 View。然而，觉得 fragment 创建的是 FragmentManager。因此，当 RecyclerView 要求 Adapter 提供 fragment 视图时。我们无法立即创建 fragment 并提供其视图。这就是 PagerView 存在的理由。

下面看它的内部实现。

PagerAdapter 不使用可返回视图的 onBindViewHolder(...) 方法，而是使用以下方法：
```
public Object instantiateItem(ViewGroup container,int position)
public void destroyItem(ViewGroup container,int position,Object object) 
public abstract boolean isViewFromObject(View view,Object object)
```
instantiateItem(ViewGroup,int) 方法是告诉 pager adapter 创建指定位置的列表项视图，但并不要求立即创建视图，pager adpter 可以自己决定何时创建视图。然后将其添加给 ViewGroup，而 destroyItem(ViewGroup,int,Object) 方法则是告诉 pager adapter 视图已经销毁（FragmentStatePagerAdapter 和 FragmentPagerAdapter 的不同主要是在这里）。

详情可以参考此链接：
[ViewPager 全面剖析及使用详解](https://www.jianshu.com/p/e5abbda4a71c)

视图创建后，ViewPager 会在某个时间点看到它，为了确定该视图所属的对象，ViewPager 会调用 isViewFromObject(View,Object)，这里的 object 是 instantiateItem(ViewGroup,int) 方法返回的对象。
## 恢复 CrimeFragment 的边距
把 fragment_crime.xml 下的 LinearLayout 的 android:layout_margin="16dp" 改成 android:padding="16dp"。

[GitHub地址](https://github.com/YueJZJM/ViewPager)
