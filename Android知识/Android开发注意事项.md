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


