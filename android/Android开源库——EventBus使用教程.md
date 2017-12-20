## 前言

EventBus这个开源库相信很多Android开发者都用过，它是greenrobot出产的，greenrobot的厉害就不用我多说了吧，它还出产过比较出名的数据库操作开源库greenDAO。  
Android传统的消息传递方式一般是通过Intent、Handler、BroadcastReceiver等等来进行的，这些方式也算是简单易用，但也有许多缺点，比如说在进行消息传递时需要写大量的模板代码，代码耦合度高，而EventBus的诞生就完美解决了这些问题，它的原理是发送事件到事件总线，然后根据事件类型来匹配相应订阅者的订阅方法，用一张图简单说明
![EventBus原理图](http://img.blog.csdn.net/20171220153452729?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ2hhcm1pbmdXb25n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

简单列举一下它的优点
- 简单有效：简单两句代码就可以实现事件传递功能，并且订阅者不知道发送者是谁，降低耦合
- 高性能：EvnetBus针对Android平台进行了优化
- 注解：通过@Subscribe注解订阅方法，同时避免了非常耗时的运行时注解反射
- 线程：主线程和后台线程都可以传递事件

这一篇就主要讲解EventBus的基本使用方式，比较简单，下一篇主要分析EventBus的源码

## 使用方式

### 依赖
项目中依赖EventBus库，这里通过gradle依赖的方式，使用的是最新版本
```
compile 'org.greenrobot:eventbus:3.1.1'
```
想要了解详细的使用方式和官方文档，请看EventBus的Github，传送门：https://github.com/greenrobot/EventBus  

#### 实现步骤
使用方式非常简单，可分为三点
1. 需要接受事件的页面注册EventBus
2. 通过EventBus发送事件，相关页面接收事件并处理
3. 页面销毁，反注册EventBus

## 代码实现

先来看一下实现效果
![实现效果](http://img.blog.csdn.net/20171220153506967?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ2hhcm1pbmdXb25n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 注册
```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    mTextView = findViewById(R.id.tv);
    findViewById(R.id.btn_jump).setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View v) {
            Intent intent = new Intent(MainActivity.this, SecondActivity.class);
            startActivity(intent);
        }
    });

    // 注册该页面为订阅者
    EventBus.getDefault().register(this);
}
```

设置按钮监听，点击跳转，最后一句注册该页面为订阅者，看一下订阅方法

```
@Subscribe(threadMode = ThreadMode.MAIN)
public void onUpdateUIEvent(UpdateUIEvent updateUIEvent) {
    mTextView.setText("陈奕迅只有一个");
}
```
在MainActivity定义一个方法，通过@Subscribe注解订阅方法，接收UpdateUIEvent作为参数，意思是，在该页面注册成为订阅者之后，当别处发送UupdateUIEvent事件时，这个订阅方法就会接收到事件并处理，此处的做法是更新TextView显示的文本。UpdateUIEvent是自定义的一个事件类，名字可以随便起，threadMode表示订阅事件运行在哪个线程，这里指定主线程。  

### 发送事件
点击跳转之后，来到SecondActivity，看代码

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_second);

    findViewById(R.id.btn_send).setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View v) {
            Toast.makeText(SecondActivity.this, "滋醒你~~~", Toast.LENGTH_SHORT).show();
            // 发送事件
            EventBus.getDefault().post(new UpdateUIEvent());
        }
    });
}
```
设置按钮监听，onClick方法里post一个事件，该事件正是UpdateUIEvent，这也意味着MainActivity的订阅方法`onUpdateUIEvent(UpdateUIEvent)`会接收到该事件，结合图来看确实是如此。

### 反注册
当销毁页面时，也就意味着该页面不需要再接收事件，可以在onDestroy中反注册EventBus

```
    @Override
    protected void onDestroy() {
        EventBus.getDefault().unregister(this);
        super.onDestroy();
    }
```

到这里可能很多人会问：实际的开发中有哪些场景是适合用EventBus来实现的呢，我举个简单地例子：比如说一开始用户未登录，而当用户登录时，需要更新非前台页面的一些页面信息，以便在用户在返回这些页面时立马看到更新的效果，这时候可以使用EventBus来进行post更新事件达到静默更新的效果。

最后附上完整源码，xml就不贴了  
MainActivity：
```
public class MainActivity extends AppCompatActivity {

    private TextView mTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mTextView = findViewById(R.id.tv);
        findViewById(R.id.btn_jump).setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(MainActivity.this, SecondActivity.class);
                startActivity(intent);
            }
        });

        // 注册该页面为订阅者
        EventBus.getDefault().register(this);
    }

    @Override
    protected void onDestroy() {
        EventBus.getDefault().unregister(this);
        super.onDestroy();
    }

    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onUpdateUIEvent(UpdateUIEvent updateUIEvent) {
        mTextView.setText("陈奕迅只有一个");
    }
}
```

SecondActivity：
```
public class SecondActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);

        findViewById(R.id.btn_send).setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(SecondActivity.this, "滋醒你~~~", Toast.LENGTH_SHORT).show();
                // 发送事件
                EventBus.getDefault().post(new UpdateUIEvent());
            }
        });
    }
}
```

## 总结
好了，基本使用大致如上，非常简单，相信大部分同学都用过了，没用过的同学看了之后应该也有个大致的了解了。下一篇来点有意思的，通过分析EventBus的源码来了解实现原理。

## 总结
好了，基本使用大致如上，非常简单，相信大部分同学都用过了，没用过的同学看了之后应该也有个大致的了解了。下一篇来点有意思的，通过分析EventBus的源码来了解实现原理。
