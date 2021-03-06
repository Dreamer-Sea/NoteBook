# Android开发注意事项
## 指定App的主界面
```xml
<activity android:name=".HelloWorldActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

## Activity
### Activity的创建
所有创建的Activity都是Activity这个类的子类，为了满足向下兼容的要求，在AS中创建的Activity默认继承于AppCompatActivity这个类。最低向下兼容到Android 2.1。
### 界面布局的导入
用setContentView方法。
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.hello_world_layout);
}
```

## res目录下各文件的作用
- drawable：放图片；
- mipmap：应用图标；
- values：字符串，样式，颜色；
- layout：布局文件。
### 调用xml文件中的内容
以res/values/string.xml为例
```xml
<resources>
    <string name="app_name">HelloWorld</string>
</resources>
```
- 代码中：R.string.hello_world；
- XML中：@string/hello_world。

## Activity的生命周期
### 原理
- 运行状态：位于栈顶的Activity(**用户能够看见的Activity？**)，一般不会被回收。
- 暂停状态：不再位于栈顶，但是仍然可见（在弹窗之下的Activity），除非内存不够，不然不会被回收。
- 停止状态：Activity不位于栈顶且不可见，Activity的状态和成员变量会被保存，但是其它地方需要内存的时候，处于停止状态的Activity可能会被回收。
- 销毁状态：不在返回栈中的Activity处于这个状态下，也是最先回收内存的状态。

### 具体方法
- onCreate()：Activity在第一次创建的时候就调用这个方法，仅调用一次。比如加载布局、绑定事件等。
- onStart()：Activity从不可见变为可见的时候调用。
- onResume()：Activity准备好和用户进行交互的时候调用。此时Activity一定位于返回栈的栈顶，并且处于运行状态。
- onPuse()：在系统准备去启动或恢复另一个Activity的时候调用。通常会在这个方法中将一些消耗CPU的资源释放掉，并保存一些关键数据，但是这个方法中的操作不能有耗时的操作，不然会影响栈顶Activity的使用。
- onStop()：在Activity完全不可见的时候调用。该方法与onPuse()的区别，该方法在可见但不可执行的时候调用。
- onDestroy()：该方法在Activity被销毁之前调用，之后Activity的状态将变为销毁状态。
- onRestart()：该方法在Activity由停止状态变为运行状态之前调用，也就是Activity被重新启动了。

### 启动方式
在AndroidManifest.xml通过 <activity> 标签指定 android: launchMode 属性来选择启动模式。
- standard：默认的启动方式。每当启动一个新的活动，它就会在返回栈中入栈。对于使用standard模式的activity，系统不会在乎这个activity是否已经在返回栈中存在，每次启动都会创建该Activity的一个新的实例。
- singleTop：发现当前Activity在栈顶的话，就不会创建新的Activity。
- singleTask：如果返回栈中存在目标Activity，那么目标Activity前的Activity都会被出栈，然后目标Activity就到了栈顶。
-  singleInstance：声明为该启动方式的Activity会启用一个新的返回栈(任务栈)，该栈只会存放该Activity。如果同一个App中有多个Activity使用该运行方式，那么就会有一一对应的多个返回栈。

## Fragment的生命周期
### 原理
- 运行状态：Fragment可见，且它所关联的Activity正处于运行状态。
- 暂停状态：与Fragment关联的Activity处于暂停状态。
- 停止状态：与Fragment关联的Activity处于停止状态，或Fragment不可见的时候。
- 销毁状态：与Fragment关联的Activity处于销毁状态，或通过调用FragmentTransaction的remove方法和replace方法将碎片从Activity中移除，但在事务提交前未调用addToBackStack方法。

### 具体方法
- onAttach()：当Fragment和Activity建立关系的时候调用。
- onCreateView()：当Fragment加载布局的时候调用。
- onActivityCreated()：当Fragment和与之关联的Activity完全创建完毕的时候调用。
- onDestroyView()：当Fragment中的布局被移除的时候调用。
- onDetach()：当Fragment和Activity解除关联的时候调用。

## 广播机制
### 分类
- 标准广播：完全异步执行，所有的广播接收器几乎都会在同一时刻接收到这条广播信息。效率高，但是无法截停。
- 有序广播：同步执行。在广播发出后，同一时刻只会有一个广播接收器能够接收这条广播消息。优先级高的广播接收器能够先收到广播消息。

### 注册方式
- 静态注册：在AndroidManifest.xml文件中注册。这种方式App不启动也能接收到广播。
- 动态注册：在代码中注册。这种方法需要App启动才能够接收到广播。

### 注意
Android 8.0 之后对广播做出了改变。建议使用动态注册的方式注册广播。
```java
Intent intent=new Intent("com.example.cc.broadcasttest.MY_BROADCAST");
        //ComponentName("当前类的包名","包名.这条广播的接收器的类名")
intent.setComponent(new ComponentName("com.example.cc.broadcasttest",
                "com.example.cc.broadcasttest.MyBroadcastReceiver"));
//发送标准广播
sendBroadcast(intent);
```

## 持久化
### 类型：
- 文件：将数据原封不动的保存在文件中。使用Context类中的openFileOutput方法，传入两个参数，第一个参数表示文件名；第二个参数表示文件的操作模式，可选的有：MODE_APPEND(如果文件存在则直接附加)，MODE_PRIVATE(默认的，文件存在则覆盖)。
- SharedPreference：
- 数据库：

## 通知
### 在Android 8 下的改动
#### 需要构建通道(Channel)
在Android 8及之后需要给通知设置通道，这样才能在通知栏中显示。
```java
Intent intent = new Intent(this, NotificationActivity.class);
                PendingIntent pi = PendingIntent.getActivity(this, 0, intent, 0);
                NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
                Notification notification;
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
                   String id = "myChannel";
                   String name = "我的渠道";
                   NotificationChannel channel = new NotificationChannel(id, name,NotificationManager.IMPORTANCE_DEFAULT);
                   manager.createNotificationChannel(channel);
                   notification = new NotificationCompat.Builder(this, id)
                           .setChannelId(id)
                           .setContentTitle("This is content title")
                           .setContentText("This is content text")
                           .setWhen(System.currentTimeMillis())
                           .setSmallIcon(R.mipmap.ic_launcher)
                           .setLargeIcon(BitmapFactory.decodeResource(getResources(),
                                   R.mipmap.ic_launcher))
                           .setContentIntent(pi)
                           .setAutoCancel(true)
                           .setSound(Uri.fromFile(new File("/system/media/audio/ringtones/Luna.ogg")))
                           .setVibrate(new long[]{0, 1000, 1000, 1000})
                           .setLights(Color.WHITE, 1000, 1000)
                           .setStyle(new NotificationCompat.BigPictureStyle().bigPicture(BitmapFactory.decodeResource(
                                   getResources(), R.drawable.big_image)))
                           .build();
                   manager.notify(1, notification);
               }else{
                   notification = new NotificationCompat.Builder(this, "default")
                           .setContentTitle("This is content title")
                           .setContentText("This is content text")
                           .setWhen(System.currentTimeMillis())
                           .setSmallIcon(R.mipmap.ic_launcher)
                           .setLargeIcon(BitmapFactory.decodeResource(getResources(),
                                   R.mipmap.ic_launcher))
                           .setContentIntent(pi)
                           .setAutoCancel(true)
                           .setSound(Uri.fromFile(new File("/system/media/audio/ringtones/Luna.ogg")))
                           .setVibrate(new long[]{0, 1000, 1000, 1000})
                           .setLights(Color.WHITE, 1000, 1000)
                           .build();
                   manager.notify(1, notification);
               }
```
#### 设置父Activity
想通过Notification，从同一个App的Activity A跳转到Activity B，需要在AndroidManifest.xml中设置Activity B的父Activity 为Activity A。该方法能通过返回键从Activity B返回到Activity A。
如果不想用户通过返回键返回上一个Activity，那么就要在AndroidManifest.xml中这样设置。
```xml
<activity
    android:name=".ResultActivity"
    android:launchMode="singleTask"
    android:taskAffinity=""
    android:excludeFromRecents="true">
</activity>
```
## HttpURLConnection
该方法在访问‘http://www.baidu.com’的时候，返回空。访问https开头的时候正常。OkHttp3能直接访问http。

## 异步消息处理机制
### 四个部分
- Message：是信息在线程之间传递的主体。
- Handler：用来发送和处理信息。
- MessageQueue：消息队列。用来存放所有通过Handler发送的消息。
- Looper：每个线程中的MessageQueue的管家。调用Looper中的loop方法就会使得Looper进入一个无限循环当中，每当发现MessageQueue中存在一条消息就会将它取出，并传递到Handler的handleMessage方法中，每个线程有且仅有一个Looper对象。

## 使用Intent传递复杂数据(对象)
### 实现Serializable接口
将使想要传递的对象的类实现Serializable接口，然后调用Intent的putExtra方法就能够像传递普通数据一样传递复杂数据。接收这个Intent的Activity就要调用Intent对象的getSerializableExtra方法来提取复杂对象。
### 实现Parcelable接口
与实现Serializable接口类似，但是接收的Activity需要调用getParcelableExtra方法来获取对象。
### 两种方式的对比
前者是对整个对象进行序列化，后者是将对象的字段给分解出来。后者效率更高。