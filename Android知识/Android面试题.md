# Android面试题
[toc]
## onSaveInstanceState方法
### 触发条件(以下之一)
- 当用户按下Home键时。
- 长按Home键，选择运行其他程序时。
- 按下电源键(关闭屏幕)时。
- 从Activity A中启动一个新的Activity时。
- 屏幕方向切换时，例如从竖屏切换到横屏时。
- 优先级较低并因为内存不足而被回收。
### 实现
在Activity/Fragment中重写onSaveInstanceState方法。
保存数据：
```java
@Override
public void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    String tempData = "Something you just typed";
    outState.putString("data_key", tempData);
}
```
获取数据
方法一(在onCreate方法中执行)：
```java
if (savedInstanceState != null){
    tempData = savedInstanceState.getString("data_key");
    Log.d(TAG, tempData);
}
```
方法二(重写onRestoreInstanceState方法)：
```java
@Override
protected void onRestoreInstanceState(Bundle savedInstanceState) {
    super.onRestoreInstanceState(savedInstanceState);
    tempData = savedInstanceState.getString("data_key");
    Log.d(TAG, tempData);
}
```
方法一和方法二的区别：
- 方法一需要判断savedInstanceState(bundle对象)是否为空，而方法二则不需要判断，因为方法二是在savedInstanceState不为空的情况下才能执行。
- 两个方法执行时所在的生命周期不同。方法一是在onCreate中实现，方法二是在onStart方法之后调用。

### 用onSaceInstanceState()和用onPause()保存数据的区别
前者适合保存临时性的状态，后者适合在其中实现对数据的持久化保存。

## 启动Activity A，然后从Activity A中启动Activity B，其中经历了哪些生命周期
Activity A的onCreate() -> onStart() -> onResume() -> onPause() -> Activity B的onCreate() -> onStart() -> onResume() -> Activity A的onStop()。
注意：从Activity A中启动Activity B时，先调用Activity A的onPause()，等Activity B启动完毕后，才调用Activity A的onStop()。

### 如果Activity B是透明的或者半透明的，那么会经历哪些生命周期
与上面的类似，但是在Activity B启动完毕后，不会调用Activity A的onStop()。

## 如何避免配置改变时Activity重建
配置Activity的configChanges属性。如果在AndroidManifest.xml中设置对应Activity为android:configChanges="orientation|screenSize"，表示屏幕翻转时，Activity不会被销毁重建，只会调用Activity中的onConfigurationChanged方法。

## Activity的四个启动模式
- standard模式：每启动一次Activity就创建一个新的实例。
- singleTop模式：如果要启动的Activity已经位于任务栈的**栈顶**，那么就不会重新创建，并回调onNewIntent(intent)方法。
- singleTask模式：如果要启动的Activity已经在**任务栈中**了，就会将该Activity之前的Activity全部出栈，使得该Activity放在栈顶。
- singleInstance模式：为该Activity开辟一个专用的任务栈。
### 什么时候会调用onNewIntent()？
- singleTop模式：任务已经位于栈顶。
- singleTask模式：任务已经位于栈中。
### Activity启动模式的标记位
- FLAG_ACTIVITY_SINGLE_TOP：对应singleTop启动模式。
- FLAG_ACTIVITY_NEW_TASK：对应singleTask模式。

## 如何从一个Activity中启动另一个Activity
有两种方法，一种是显式的，另一种式隐式的。
显式的：
```java
Intent intent = new Intent(MainActivity.this, SecondActivity.class);
startActivity(intent);
```
隐式的：
配置Activity的action和category。
```xml
<activity android:name=".SecondActivity">
	<intent-filter>
		<action android:name="com.example.activitytest.ACTION_START"/>
		<category android:name="android.intent.category.DEFAULT"/>
	</intent-filter>
</activity>
```
使用Intent启动：
```java
Intent intent = new Intent("com.example.activitytest.ACTION_START");
startActivity(intent);
```
每个Intent只能有一个action，但是可以有多个category。
```xml
<activity android:name=".SecondActivity">
	<intent-filter>
		<action android:name="com.example.activitytest.ACTION_START"/>
		<category android:name="android.intent.category.DEFAULT"/>
		<category android:name="com.example.activitytest.MY_CATEGORY"/>
	</intent-filter>
</activity>
```
```java
Intent intent = new Intent("com.example.activitytest.ACTION_START");
intent.addCategory("com.example.activitytest.MY_CATEGORY");
startActivity(intent);
```

## 如何在当前Activity启动其他应用的Activity
在有权限访问的情况下，通过隐式Intent进行目标Activity的IntentFilter匹配，原则是：
- 一个Intent只有**同时**匹配某个Activity的intent-filter中的action，category，data才算**完全匹配**，才能启动该Activity。
- 一个Activity可以有**多个**intent-filter，一个intent只要成功匹配任意一组intent-filter，就可以启动该Activity。

## Activity的启动过程
调用startActivity()后经过重重方法会转移到ActivityManagerService的startActivity()，并通过一个IPC回到ActivityThread的内部类ApplicationThread中，并调用其scheduleLaunchActivity()将启动Activity的消息发送并交由Handler H处理。Handler H对消息的处理会调用handlerLaunchActivity -> performLaunchActivity()得以完成Activity对象的创建和启动。

## Fragment的生命周期
从创建到销毁涉及到的方法。
onAttach() -> onCreate() -> onCreateView() -> onActivityCreated() -> onStart() -> onResume() -> onPause() -> onStop() -> onDestroyView() -> onDestroy() -> onDetach()。
与Activity不同的方法有：
- onAttach()：当Fragment和Activity建立关联时调用。
- onCreateView()：当Fragment创建视图时调用。
- onActivityCreated()：当与Fragment相关联的Activity完成onCreate()之后调用。
- onDestroyView()：在Fragment中布局被移除时调用。
- onDetach()：当Fragment和Activity接触关联时调用。