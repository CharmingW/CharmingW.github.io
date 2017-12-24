## 前言
上一篇写了EventBus的基本用法，传送门：
[Android开源库——EventBus使用教程](http://note.youdao.com/)  
在上一篇中只是简单展示了EventBus的基本用法，其实还有很多好玩和强大的功能，那么在本篇将一步一步地去探索EventBus那些好玩又高级的用法。主要有
- Thread Mode（线程模式）
- Configuration（配置）
- Sticky Events（粘性事件）
- Priorities and Event Cancellation（优先级和取消事件传递）
- Subscriber Index（订阅者索引）
- AsyncExecutor（异步执行器）

话不多说，马上开始

## 高级用法
### Thread Mode
还记得上一篇中，订阅方法的@Subscribe中包含的信息
```
@Subscribe(threadMode = ThreadMode.MAIN)
public void onUpdateUIEvent(UpdateUIEvent updateUIEvent) {
    mTextView.setText("陈奕迅只有一个");
```
通过指定threadMode来确定订阅方法在哪个线程执行，ThreadMode.MAIN只是其中一种，看一下其他的

```
public enum ThreadMode {
    /**
     * Subscriber will be called in the same thread, which is posting the event. This is the default. Event delivery
     * implies the least overhead because it avoids thread switching completely. Thus this is the recommended mode for
     * simple tasks that are known to complete is a very short time without requiring the main thread. Event handlers
     * using this mode must return quickly to avoid blocking the posting thread, which may be the main thread.
     */
    POSTING,

    /**
     * Subscriber will be called in Android's main thread (sometimes referred to as UI thread). If the posting thread is
     * the main thread, event handler methods will be called directly. Event handlers using this mode must return
     * quickly to avoid blocking the main thread.
     */
    MAIN,

    /**
     * Subscriber will be called in a background thread. If posting thread is not the main thread, event handler methods
     * will be called directly in the posting thread. If the posting thread is the main thread, EventBus uses a single
     * background thread, that will deliver all its events sequentially. Event handlers using this mode should try to
     * return quickly to avoid blocking the background thread.
     */
    BACKGROUND,

    /**
     * Event handler methods are called in a separate thread. This is always independent from the posting thread and the
     * main thread. Posting events never wait for event handler methods using this mode. Event handler methods should
     * use this mode if their execution might take some time, e.g. for network access. Avoid triggering a large number
     * of long running asynchronous handler methods at the same time to limit the number of concurrent threads. EventBus
     * uses a thread pool to efficiently reuse threads from completed asynchronous event handler notifications.
     */
    ASYNC
}
```
注释一目了然，总结一下四种模式的应用场景
- POSTING：事件发布在什么线程，就在什么线程订阅。
- MAIN：无论事件在什么线程发布，都在主线程订阅。
- BACKGROUND：如果发布的线程不是主线程，则在该线程订阅，如果是主线程，则使用一个单独的后台线程订阅。
- ASYNC：用线程池线程订阅。


比如
```
@Subscribe(threadMode = ThreadMode.BACKGROUND)
public void onUpdateUIEvent(UpdateUIEvent updateUIEvent) {
    mTextView.setText("陈奕迅只有一个");
}
```
这样我们就是在后台中订阅事件，但由于更新UI只能在主线程中，所以会出现以下异常

```
Could not dispatch event: class com.charmingwong.androidtest.UpdateUIEvent to subscribing class class com.charmingwong.androidtest.MainActivity
android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views.
```


### Configuration
我们获取EventBus实例的默认方式是

```
EventBus.getDefault().register(this);
```
这种方式的获取到的EventBus的属性都是默认的，有时候并不能满足我们的要求，这时候我们可以通过EventBusBuilder来个性化配置EventBus的属性

```
EventBus eventBus = EventBus.builder().eventInheritance(true)
    .ignoreGeneratedIndex(false)
    .logNoSubscriberMessages(true)
    .logSubscriberExceptions(false)
    .sendNoSubscriberEvent(true)
    .sendSubscriberExceptionEvent(true)
    .throwSubscriberException(true)
    .strictMethodVerification(true)
    .build();
eventBus.register(this);
```
使用类似于以上这种Builder模式，来达到个性化配置的效果，builder()返回的是EventBusBuilder，来看一下EventBusBuilder各项配置的含义

```
// 创建默认的EventBus对象，相当于EventBus.getDefault()。
EventBus installDefaultEventBus()：
// 添加由EventBus“注释预处理器生成的索引
EventBuilder addIndex(SubscriberInfoIndex index)：
// 默认情况下，EventBus认为事件类有层次结构（订户超类将被通知）
EventBuilder eventInheritance(boolean eventInheritance)：
// 定义一个线程池用于处理后台线程和异步线程分发事件
EventBuilder executorService(java.util.concurrent.ExecutorService executorService)：
// 设置忽略订阅索引，即使事件已被设置索引，默认为false
EventBuilder ignoreGeneratedIndex(boolean ignoreGeneratedIndex)：
// 打印没有订阅消息，默认为true
EventBuilder logNoSubscriberMessages(boolean logNoSubscriberMessages)：
// 打印订阅异常，默认true
EventBuilder logSubscriberExceptions(boolean logSubscriberExceptions)：
// 设置发送的的事件在没有订阅者的情况时，EventBus是否保持静默，默认true
EventBuilder sendNoSubscriberEvent(boolean sendNoSubscriberEvent)：
// 发送分发事件的异常，默认true
EventBuilder sendSubscriberExceptionEvent(boolean sendSubscriberExceptionEvent)：
// 在3.0以前，接收处理事件的方法名以onEvent开头，方法名称验证避免不是以此开头，启用严格的方法验证（默认：false）
EventBuilder strictMethodVerification(java.lang.Class<?> clazz)
// 如果onEvent***方法出现异常，是否将此异常分发给订阅者（默认：false）
EventBuilder throwSubscriberException(boolean throwSubscriberException) 

```
通过这种方式，我们就可以灵活地使用EventBus。


### Sticky Events
一般的事件发布之后，后来注册的订阅者无法再收到这些事件，而sticky事件发布之后，EventBus会将它保存起来，直到有新的同事件类型的sticky事件发布，才将旧的覆盖，因此后续的订阅者只要使用sticky模式，依然可以获得这个sticky事件。使用方式如下：  

在SecondActivity中订阅sticky事件并注册
```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    // 注册该页面为订阅者
    EventBus.getDefault().register(this);
}

@Subscribe(sticky = true)
public void onMyStickyEvent(MyStickyEvent myStickyEvent) {
    mTextView.setText("有人问你他是不是陈奕迅");
}

```
在MainActivity中发布事件

```
@Override
public void onClick(View v) {
    // 发布黏性事件
    EventBus.getDefault().postSticky(new MyStickyEvent());
    Intent intent = new Intent(MainActivity.this, SecondActivity.class);
    startActivity(intent);
}
```
看一下运行效果：
![运行效果](C:/Users/56223/Pictures/EventBus/device-2017-12-21-164214.gif)

MainActivity启动在前，发布sticky事件，SecondActivity再启动，注册成为订阅者之后依然能够收到sticky事件，成功说明sticky的作用，也验证了以上说法。发布sticky事件之后，也可以将它移除

```
MessageEvent stickyEvent = EventBus.getDefault().removeStickyEvent(MessageEvent.class);
// Better check that an event was actually posted before
if(stickyEvent != null) {
    // Now do something with it
}
```

### Priorities and Event Cancellation
这里用一个例子验证优先级关系

```
// MainActivity
@Subscribe(priority = 2)
public void onUpdateUIEvent(UpdateUIEvent updateUIEvent) {
    Log.d(TAG, "onUpdateUIEvent: priority = 2");
}

// SecondActivity
@Subscribe(priority = 1)
public void onUpdateUIEvent(UpdateUIEvent updateUIEvent) {
    Log.d(TAG, "onUpdateUIEvent: priority = 1");
}
```
两个Activity中都订阅相同的事件类型，但优先级不同，在SecondActivity发布事件

```
// 发布事件
EventBus.getDefault().post(new UpdateUIEvent());
```
看运行效果，日志：

```
D/MainActivity: onUpdateUIEvent: priority = 2
D/SecondActivity: onUpdateUIEvent: priority = 1
```
再将priority调换一下
```
// MainActivity
@Subscribe(priority = 1)
public void onUpdateUIEvent(UpdateUIEvent updateUIEvent) {
    Log.d(TAG, "onUpdateUIEvent: priority = 1");
}

// SecondActivity
@Subscribe(priority = 2)
public void onUpdateUIEvent(UpdateUIEvent updateUIEvent) {
    Log.d(TAG, "onUpdateUIEvent: priority = 2");
}
```
运行，看日志：

```
D/SecondActivity: onUpdateUIEvent: priority = 2
D/MainActivity: onUpdateUIEvent: priority = 1
```
可见priority决定收到事件的先后顺序，priority越大，越先收到事件。先收到的订阅者还可以拦截该事件，并取消继续传递，后续优先级低订阅者就不会再收到该事件。我们可以这样设置取消事件继续传递：

```
@Subscribe(priority = 2)
public void onUpdateUIEvent(UpdateUIEvent updateUIEvent) {
    Log.d(TAG, "onUpdateUIEvent: priority = 2");
    EventBus.getDefault().cancelEventDelivery(updateUIEvent);
}
```
运行，看日志：

```
D/SecondActivity: onUpdateUIEvent: priority = 2
```
果然，priority=1的订阅者就没有收到该事件，这就是EventBus的取消事件继续传递功能。

### Subscriber Index
看完官方文档，我们知道Subscriber Index是EventBus 3上的新技术，所以这里也建议还没学习过EventBus的可以跳过2.X之前的版本直接学习最新版本。  
关于EventBus的Subscriber Index技术的特点，翻译一下官方解释：
>  It is an optional optimization to speed up initial subscriber registration.

Subscriber Index是一个可选的优化技术，用来加速初始化订阅者注册。
> The subscriber index can be created during build time using the EventBus annotation processor. While it is not required to use an index, it is recommended on Android for best performance.

Subscriber Index在编译时使用EventBus注解处理器创建，虽然没有规定必须使用它，但是官方推荐使用这种方式，因为它在Android上有着最佳的性能。

使用方式：在gradle做如下配置

```
android {
    defaultConfig {
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [ eventBusIndex : 'com.example.myapp.MyEventBusIndex' ]
            }
        }
    }
}
 
dependencies {
    compile 'org.greenrobot:eventbus:3.1.1'
    annotationProcessor 'org.greenrobot:eventbus-annotation-processor:3.1.1'
}
```
如果以上方法不生效，还可以使用android-apt Gradle plugin

```
buildscript {
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
 
apply plugin: 'com.neenbedankt.android-apt'
 
dependencies {
    compile 'org.greenrobot:eventbus:3.1.1'
    apt 'org.greenrobot:eventbus-annotation-processor:3.1.1'
}
 
apt {
    arguments {
        eventBusIndex "com.example.myapp.MyEventBusIndex"
    }
}
```
配置完了之后，我们就可以在代码中添加索引了

```
EventBus.builder().addIndex(new MyEventBusIndex()).installDefaultEventBus();
// Now the default instance uses the given index. Use it like this:
EventBus eventBus = EventBus.getDefault();
```
除了application，还可以在library中使用索引

```
EventBus eventBus = EventBus.builder()
    .addIndex(new MyEventBusAppIndex())
    .addIndex(new MyEventBusLibIndex()).build();
```
好了，这就是索引的使用方式。

### AsyncExecutor
先看个例子

```
AsyncExecutor.create().execute(
    new AsyncExecutor.RunnableEx() {
        @Override
        public void run() throws Exception {
            // No need to catch any Exception (here: LoginException)
            prepare();
            EventBus.getDefault().postSticky(new MyStickyEvent());
        }
    }
);

private void prepare() throws Exception {
    throw new Exception("prepare failed");
}

@Subscribe(threadMode = ThreadMode.MAIN)
public void onUpdateUIEvent(UpdateUIEvent updateUIEvent) {
    Log.d(TAG, "onUpdateUIEvent: ");
}

@Subscribe(threadMode = ThreadMode.MAIN)
public void handleFailureEvent(ThrowableFailureEvent event) {
    Log.d(TAG, "handleFailureEvent: " + event.getThrowable().getMessage());
```
通过这样的方式，如果相关代码出现了异常，则会将异常封装成ThrowableFailureEvent，自动发布出去，只要订阅者定义了接收ThrowableFailureEvent的方法，就可以拿到异常信息，后续的UpdateUIEvent也不会再发布，如果没有出现异常，则正常发布事件。
运行，看日志：

```
D/SecondActivity: handleFailureEvent: prepare failed
```
果然，只有handleFailureEvent(ThrowableFailureEvent event)收到了异常事件。这种写法的好处是能把异常信息交给订阅者，让订阅者根据情况处理。

## 总结
好了，以上就是EventBus相对高级一点的用法，除了以上提到的，还有一些别的特性，具体请看官方文档，如果有不明白的地方，也一定要看官方文档，这个非常重要，附上传送门：
#### [戳我，看EventBus官方文档！](http://greenrobot.org/eventbus/documentation/)

下一篇着重于分析EventBus的源码，了解其实现原理。
### **感谢阅读！**