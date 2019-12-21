# View的绘制
[toc]
## Android界面结构
整个Android界面的根元素是Activity，在Activity中有一个Windows抽象类的具体实现类PhoneWinddow，PhoneWindow中有一个DecorView，这个DecorView实际上是个FrameView。DecorView中有两个关键的子元素，分别是TitleView和ContentView，其中TitleView负责ActionBar的绘制，它包含了一个ActionBarContainer，该Container下有一个ActionBar。而一般主要内容都显示在ContentView中。

## setContentView方法
在Activity的onCreate方法中会调用setContentView方法并传入布局id(例如：R.layout.XXX)。这个函数实际上是调用了PhoneWindow中的setContentView方法，在PhoneWidow的setContentView中还要调用LayoutInflater的inflate方法，接着在这个方法中再调用inflate(XmlPullParser, ViewGroup, boolean)来真正的实现布局的填充，在这个方法中会使用解析XML文件的方式对布局文件进行解析。

## View的绘制流程
View的绘制流程分为以下三个阶段：
- measure(onMeasure()方法)：判断是否需要重新计算View的大小，需要的话则计算。
- layout(onLayout()方法)：判断是否需要重新计算View的位置，需要的话则计算。
- draw(onDraw()方法)：判断是否需要重新绘制View，需要的话则重新绘制。

View的绘制是由ViewRoot来负责的。每个应用程序窗口(Window)的decorView都有一个与之关联的ViewRoot对象，这种关联关系是由WindowManager来维护的。而且decorView与ViewRoot的关联是在Activity启动的时候就建立了。在关联关系建立后，ViewRoot类的requestLayout()方法就会被调用，以完成应用程序用户界面的**初次布局**，而实际上调用的是ViewRootImpl类的requestLayout()方法(scheduleTraversals()方法中的performTraversals()方法)。

### measure
该阶段是计算控件树中各个控件要显示所需要的尺寸。起点是ViewRootImpl类中的measureHierarchy()方法。
其中MeasureSpec是一个32位的整数，最高的两位表示SpecMode(测量模式)，剩下的表示SpecSize(对应测量模式下的测量尺寸)。View的SpecMode由本View的LayoutParams结合父View的MeasureSpec生成。
SpecMode有以下三种取值：
- EXACTLY：对子View提出一个确切的建议尺寸；
- AT_MOST：子View的大小不得超过SpecSize；
- UNSPECIFIED：对子View的尺寸不作限制，通常用于系统内部。

进行重新测量的判断条件：
```java
final boolean forceLayout = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT; //强制重新布局
  final boolean specChanged = widthMeasureSpec != mOldWidthMeasureSpec
      || heightMeasureSpec != mOldHeightMeasureSpec; // 本次传入的MeasureSpec与上次传入的不同
  final boolean isSpecExactly = MeasureSpec.getMode(widthMeasureSpec) == MeasureSpec.EXACTLY
      && MeasureSpec.getMode(heightMeasureSpec) == MeasureSpec.EXACTLY; // 若父View对子View提出了精确的宽高约束
  final boolean matchesSpecSize = getMeasuredWidth() == MeasureSpec.getSize(widthMeasureSpec)
      && getMeasuredHeight() == MeasureSpec.getSize(heightMeasureSpec); // 表示父View的宽高尺寸要求与上次测量的结果不同
  final boolean needsLayout = specChanged
      && (sAlwaysRemeasureExactly || !isSpecExactly || !matchesSpecSize);
  // 需要重新布局  
  if (forceLayout || needsLayout)
```

### layout
该阶段不同的布局方式有不同的实现方法。主要就是计算各个子控件与父控件在四个方位上的间距(控件内padding，父子控件margin)。

### draw
该阶段和layout阶段类似，也是不同的布局方式有不同的实现方法。
会执行以下几步：
- 绘制背景；
- 通过onDraw()绘制自身内容；
- 通过dispatchDraw()绘制子View；
- 绘制滚动条。