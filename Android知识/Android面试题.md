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

## Activity和Fragment的异同？
- Activity和Fragment的相似点在于，它们都可包含**布局**，可有**自己的生命周期**，Fragment可看作迷你的Activity。
- 不同点是，由于Fragment是依附在Activity上的，多了些和宿主Activity相关的生命周期方法，如onAttach()，onActivityCreated()，onDetach()；另外Fragment的生命周期方法是由宿主Activity调用而不是操作系统调用，Activity中生命周期方法都是protected，而Fragment都是public，也能印证这一点，因为Activity需要调用Fragment那些方法并管理它。

## Activity和Fragment的关系
- 它的出现是为了解决Android碎片化，它可作为Activity界面的组成部分，可在Activity运行中实现动态地加入，移除和交换。
- 一个Activity中可同时出现多个Fragment，一个Fragment也可在多个Activity中使用。
- 另外Activity的FragmentManager负责调用队列中Fragment的生命周期方法，只要Fragment的状态与Activity的状态保持了同步，宿主Activity的FragmentManager便会继续调用其他生命周期方法以继续保持Fragment与Activity的状态一致。

## 何时会考虑使用Fragment？
用两个Fragment封装两个界面模块，这样只使用一套代码就能适配两种设备，达到两种界面效果；单一场景切换时使用Fragment更轻量化，如ViewPager和Fragment搭配使用。

## Service的生命周期
- onCreate()：在服务第一次被创建时调用。
- onStartCommand()：服务启动时调用。
- onBind()：服务被绑定时调用。
- onUnBind()：服务被解绑时调用。
- onDestroy()：服务停止时调用。

## Service的两种启动方式？区别在哪？
- 其他组件调用Context的startService()方法可以启动一个Service，并回调服务中的onStartCommand()。如果该服务之前还没创建，那么回调的顺序时onCreate() -> onStartCommand()。服务启动了之后会一直保持运行状态，知道stopService()或stopSelf()方法被调用，服务停止并回调onDestroy()。另外，无论调用多少次startService()方法，只需调用一次stopService()或stopSelf()方法，服务就会停止了。
- 其它组件调用Context的bindService()可以绑定一个Service，并回调服务中的onBind()方法。类似的，如果该服务之前还没创建，那么回调的顺序是onCreate() -> onBind()。之后，调用方可以获取到onBind()方法里放回的IBinder对象的实例，从而实现和服务进行通信。只要调用方和服务之间的连接没有断开，服务就会一直保持运行状态，直到调用了unbindService()方法服务会停止，回调顺序onUnBind() -> onDestroy()。

## 一个Activity先start一个Service后，再bind时会回调什么方法？此时如何做才能回调Service的destroy()方法？
startService()启动Service之后，再bindService()绑定，此时只会回调onBind()方法；若想回调Service的destroy()方法，需要同时调用stopService()和unbindService()方法才能让服务销毁掉。

## Service如何和Activity进行通信？
通过bindService()可以实现Activity调用Service中的方法；通过广播实现Service向Activity发送消息。

## 系统的Service
- WINDOW_SERVICE
- LAYOUT_INFLATER_SERVICE
- ACTIVITY_SERVICE
- POWER_SERVICE
- ALARM_SERVICE
- NOTIFICATION_SERVICE
- KEYGUARD_SERVICE

## 能否在Service进行耗时操作？如果非要可以怎么做？
Service默认并不会运行在子线程中，也不会运行在一个独立的进程中，它同样执行在主线程中(UI线程)。换句话说，不要再Service里执行耗时操作，除非手动打开一个子线程，否则有可能出现主线程被阻塞(ANR)的情况。

## AlarmManager能实现定时的原理？
通过调用AlarmManager的set()方法就可以设置一个定时任务，并提供三个参数(工作类型，定时任务触发的事件，PendingIntent对象)。其中第三个PendingIntent对象时关键，一般会调用它的getBroadcast()方法来获取一个能够执行广播的PendingIntent。这样当定时任务被触发的时候，广播接收器的onReceive()方法就可以得到执行。即通过服务和广播的循环触发实现定时服务。

## 前台服务是什么？和普通服务的不同？如何开启一个前台服务？
前台服务是用户可见的。与普通服务的不同是，前台服务一直有一个正在运行的图标再系统的状态栏显示，下拉状态栏后可以看到更加详细的信息，非常类似于通知的效果，且当系统内存不足服务被杀死时，通知会被移除。实现一个前台服务就是再构建好一个Notification之后，不需要NotificationManger将通知显示出来，而实调用了startForeground()方法。

## ActivityManagerService的作用
ActivityManagerService是Android中最核心的服务，主要负责系统中四大组件的启动，切换，调度及应用进程的管理和调度等工作，其职责与操作系统中的进程管理和调度模块类似。

## 如何保证Service不被杀死？
- 在Service的onStartCommand()中设置flages值位START_STICKY，使得Service被杀死后尝试再次启动Service。
- 提升Service优先级，比如设置为一个前台服务。
- 在Activity的onDestroy()通过发送广播，并在广播接收器的onReceive()中启动Service。

## 广播有几种形式
- 普通广播：完全异步执行，所有广播接收器几乎都会在同一时刻接收到这条广播消息，因此它们接收的先后顺序是随机的。
- 有序广播：同步执行，优先级高的广播接收器会先接收广播消息，然后再往下传，如果广播消息被拦截，则后续的广播接收器不会接收到这条广播消息。
- 本地广播：发出的广播消息只能够在应用程序的内部进行传递，并且广播接收器也只能接收本应用程序发出的广播。
- 粘性广播：这种广播会一直滞留，当有匹配该广播的接收器被注册后，该接收器就会收到这条广播。

## 广播的两种注册形式？区别？
广播的注册有两种方法：一种在活动里通过代码动态注册，另一种在配置文件里静态注册。两种方式的相同点是都完成了对接收器以及它能接收的广播值这两个值的定义；不同点是动态注册的接收器必须是在程序启动之后才能接收到广播，而静态注册的接收器即便程序未启动也能接收到广播，比如想接收到手机开机完成后系统发出的广播就只能用静态注册。

## ContentProvider
四大组件之一，主要负责存储和共享数据。与文件存储，SharedPreferences存储，SQLite数据库存储这几种数据存储方法不同的是，后者保存下的数据只能被该应用使用，而前者可以让不同应用程序之间进行数据共享，它还可以选择只对哪一部分数据进行共享，从而保证程序中的隐私数据不会有泄漏风险。

## Android中提供哪些数据持久化存储的方法？
- File文件存储：写入和读取文件的方法和Java中实现I/O的程序一样。
- SharedPreferences存储：一种轻型的数据存储方式，常用来存储一些简单的配置信息，本质是基于XML文件存储key-value键值对数据。
- SQLite数据库存储：一款轻量级的关系型数据库，运算速度快，占用资源少，在存储大量复杂的关系型数据的时可以使用。
- ContentProvider：用于数据的存储和共享，可以让不同应用程序之间进行数据共享，还可以选择只对哪一部分数据进行共享，可保证程序中的隐私数据不会有泄漏风险。

## SharedPreferences使用情形？使用中需要注意什么？
一种轻型的数据存储方式，适用于存储一些简单的配置信息。由于系统对SharedPreferences的读/写有一定的缓存策略，即在内存中有一份该文件的缓存，因此在多进程模式下，其读/写会变得不可靠，甚至丢失数据。

## 如果要删除SQLite中表的一个字段如何做？
SQLite不允许修改和删除表字段，只能创建一个新表保留原表想要的字段，再将原表删除。

## Android中进程和线程的关系？
