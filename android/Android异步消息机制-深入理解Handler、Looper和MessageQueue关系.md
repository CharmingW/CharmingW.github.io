# Android异步消息机制-深入理解Handler、Looper和MessageQueue之间的关系

相信做安卓的很多人都遇到过这方面的问题，什么是异步消息机制，什么又是`Handler`、`Looper`和`MessageQueue`，它们之间又有什么关系？它们是如何协作来保证安卓app的正常运行？它们在开发中具体的使用场景是怎样的？今天，就让我们来揭开这几个Android异步消息机制中重要角色的神秘面纱。


## 一、写在前面

为什么要学习Android异步消息机制？和AMS、WMS、View体系一样，异步消息机制是Android framework层非常重要的知识点，掌握了对于日常开发、问题定位和解决都是非常有帮助的，会使的我们开发事半功倍。而要想成为一个合格的Android开发人员，光是懂得调用Android提供的那些个api是不够的，还要学会分析这些api背后的原理，知道它们是如果工作的，做到知其然亦知其所以然，如果不去学习技术背后的原理，只流于表面，这样永远都不会有进步，永远都只是一个Android菜鸟。


## 二、源码分析

### 1、主线程创建Looper

Android中主线程也就是我们所说的UI线程，可以简单理解为所有的界面呈现，能看得到的操作，所有的触摸、点击屏幕、更新界面UI事件的处理，都是在主线程中完成的。一个线程只有一条执行路径，如果主线程同时有多个事件要处理，那么是怎么做到有条不紊地处理的呢？接下来，以上提到的几个角色就要登场了，就是`Handler+Looper+MessageQueue`这三个角色在起作用。

`Looper`是线程的消息轮询器，是整个消息机制的核心，来看看主线程的`Looper`是如何创建的。

主线程开启于 `ActivityThread` 的 `main` 方法中，来看一下 `main` 方法的源码。
```
public static void main(String[] args) {

        、、、

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        、、、
    }
```
`Looper.prepareMainLooper()` 这句代码似乎为主线程创建 `Looper`，进入方法内部一探究竟。
```
public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```
果然在这个方法内部又调用 `Looper.prepare(boolean)` 方法为主线程创建 `Looper` 对象，存储在 `ThreadLocal` 中，我们都知道，`ThreadLocal` 为每个线程创建一个副本，所以不同线程 `set` 的值不会被覆盖，再次取出值时对应的是该线程 `set` 进去的值。接下来通过 `Looper.myLooper()` 拿到主线程的 `Looper` 让`Looper` 的静态变量`sMainLooper`持有，之后再想取主线程 `Looper` 通过 `Looper.getMainLooper()` 拿到
```
public static Looper getMainLooper() {
    synchronized (Looper.class) {
        return sMainLooper;
    }
}
```
这样，主线程的`Looper`就创建成功了，需要注意的是，无论是主线程还是子线程，`Looper`只能被创建一次，否则会抛异常，以上源码可以很好地解释。

### 2、子线程创建Looper
与主线程稍稍有点不一样，子线程的`Looper`需要手动去创建，并且有些地方是需要注意的，下面让我们一起来探究一下。

子线程创建`Looper`标准写法是这样的
```
new Thread(new Runnable() {
            @Override
            public void run() {
                //创建子线程的Looper
                Looper.prepare();
                //开启消息轮询
                Looper.loop();
            }
        }).start();
```
需要先创建子线程的`Looper`再开启消息轮询，否则`Looper.loop()`中会抛`RuntimeException`
```
public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        、、、
    }
```
这样，主线程和子线程的`Looper`创建过程我们都知道了，有了`Looper`，我们就能开启消息轮询了吗？不能，因为`Looper`只是消息轮询器，就好比大厨，还需要食材才能烹饪，因此要想开启消息轮询，还需要消息的仓库，消息队列`MessageQueue`。

### 3、MessageQueue的创建
我们看看`Looper`的私有构造方法
```
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```
可见在每个线程创建`Looper`的时候也创建了一个`MessageQueue`，并将`MessageQueue`对象作为该线程`Looper`的成员变量，这就是`MessageQueue`的创建过程。

### 4、开启消息轮询
有了`Looper`和`MessageQueue`之后就能开启消息轮询了，非常简单，通过`Looper.loop()`
```
/**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            final long end;
            try {
                msg.target.dispatchMessage(msg);
                end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (slowDispatchThresholdMs > 0) {
                final long time = end - start;
                if (time > slowDispatchThresholdMs) {
                    Slog.w(TAG, "Dispatch took " + time + "ms on "
                            + Thread.currentThread().getName() + ", h=" +
                            msg.target + " cb=" + msg.callback + " msg=" + msg.what);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```
 在方法中可以看到有一个`for(;;)`死循环，该循环中又调用了`MessageQueue`的`next()`方法 ，进入方法一探究竟。
```
Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }
```
该方法里面同样有一个`for(;;)`死循环，当没有可以处理该消息的`Handler`时，就会一直阻塞
```
if (pendingIdleHandlerCount <= 0) {
    // No idle handlers to run.  Loop and wait some more.
    mBlocked = true;
    continue;
}
```
如果从`MessageQueue`中拿到消息，返回`Looper.loop()`中，`loop()`有以下片段
```
try {
    msg.target.dispatchMessage(msg);
    end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
} finally {
    if (traceTag != 0) {
        Trace.traceEnd(traceTag);
    }
```
可以很清楚看到`Message`是用它所绑定的`Handler`来处理的，调用`dispatchMessage(Message)`，这个`Handler`其实就是发送`Message`到`MessageQueue`时所用的`Handler`，在发送时绑定了。

`Handler`拿到消息之后会怎么处理呢，我们暂且搁一边，先来看看`Handler`是怎么创建并发送消息的

### 5、创建Handler
可以继承于`Handler`并重写`handleMessage()`，实现自己处理消息的逻辑
```
private static class MyHandler extends Handler {

        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
        }
    }
```
简单地，可以在程序中这样创建
```
MyHandler handler = new MyHandler();
```
需要注意的是，线程创建`Handler`实例之前必须先创建`Looper`实例，否则会抛`RuntimeException`
```
ublic Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```
`Handler`的消息处理逻辑同样可以通过实现`Handler`的内部接口`Callback`来完成
```
public interface Callback {
    public boolean handleMessage(Message msg);
}
```
```
Handler handler = new Handler(new Callback() {
            @Override
            public boolean handleMessage(Message msg) {
                //处理消息
                return true;
            }
        });
```
关于这两种处理消息的方式哪个优先级更高，接下来会讲到

### 6、Handler发送消息
首先可以通过类似以下的代码来创建`Message`
```
Message message = Message.obtain();
message.arg1 = 1;
message.arg2 = 2;
message.obj = new Object();
```
`Handler`发送消息的方式多种多样，常见有这几种
```
sendEmptyMessage();           //发送空消息
sendEmptyMessageAtTime();     //发送按照指定时间处理的空消息
sendEmptyMessageDelayed();    //发送延迟指定时间处理的空消息
sendMessage();                //发送一条消息
sendMessageAtTime();          //发送按照指定时间处理的消息
sendMessageDelayed();         //发送延迟指定时间处理的消息
sendMessageAtFrontOfQueue();  //将消息发送到消息队头
```
也可以在设置`Handler`之后，通过`message`自身发送消息，不过最终都是调用`Handler`发送消息的方法
```
message.setTarget(handler);
message.sendToTarget();

public void sendToTarget() {
    target.sendMessage(this);
}
```
除此之外，还有一种另类的发送方式
```
post();
postDelayed();
postAtTime();
postAtFrontOfQueue();
```
以`post(Runnable r)`为例，此种方式是通过`post`一个`Runnable`回调，构造成一个`Message`并发送
```
public final boolean post(Runnable r) {
   return  sendMessageDelayed(getPostMessage(r), 0);
}

private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}
```
`Runnable`回调存储在`Message`的成员变量`callback`中，`callback`的作用，接下来会讲到

以上是消息的发送方式，那么消息是如何发送到`MessageQueue`的呢，再来看
```
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}

private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```
所有的消息发送方式最终都是调用 `Handler` 的 `sendMessageAtTime()`，并且会检查消息队列是否为空，若空则抛 `RuntimeException`，之后调用 `Handler` 的 `enqueueMessage()` ，最后调用`MessageQueue` 的 `enqueueMessage()` 将消息入队。
```
boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```
该方法根据消息的处理时间来对消息进行排序，最终确定哪个消息先被处理

至此，我们已经很清楚消息的创建和发送以及消息轮询过程了，最后来看看消息是怎么被处理的


### 7、消息的处理
回到`Looper.loop()`中的这一句代码
```
msg.target.dispatchMessage(msg);
```
消息被它所绑定的`Handler`的`dispatchMessage()`处理了
```
public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```
由此可见，消息处理到底采用哪种方式，是有优先级区分的

首先是`post`方法发送的消息，会调用`Message`中的`callback`，也就是`Runnable`的`run()`来处理
```
private static void handleCallback(Message message) {
    message.callback.run();
}
```
其次则是看`Handler`在创建时有没有实现`Callback`回调接口，若有，则调用
```
mCallback.handleMessage(msg)
```
如果该方法没能力处理，则返回`false`，让给接下来处理

最后才是调用`Handler`的`handleMessage()`

## 三、总结
1. 熟悉消息机制几个角色的创建过程，先有`Looper`，再有`MessageQueue`，最后才是`Handler`。
2. 熟悉线程中使用消息机制的正确写法，以及消息的创建和发送。
3. 一个线程可以有多个`Handler`，这些`Handler`无论在哪里发送消息，最终都会在创建其的线程中处理消息，
这也是能够异步通信的原因。
4. Android 提供的 `AsyncTask`、`HandlerThread`等等都用到了异步消息机制。

最后借用一张图说明Android异步消息机制
![Android异步消息机制](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1507989458477&di=388a1483024d6ce5cace5a131be4d391&imgtype=0&src=http%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F678225%2F201605%2F678225-20160515184153930-1748602173.png)

## 四、写在最后
至此，Android 异步消息机制就讲解完毕了，有木有一种醍醐灌顶的感觉，哈哈~~~~，这篇文章涉及到的源码不难，非常好理解，关键还是要自己去阅读源码，理解其原理，做到知其然亦知其所以然，这个道理对于大部分领域的学习都适用吧，要知道，Android发展到现在，技术越来越成熟，早已不是那个写几个界面就能拿高薪的时代了，市场对于Android 工程师的要求越来越高，这也提醒着我们要跟上技术发展的步伐，时刻学习，避免被淘汰。

由于水平有限，文章可能会有不少纰漏，还请读者能够指正，Android SDK 源码的广度和深度也不是小小篇幅能够概括的，未能尽述之处，还请多多包涵。

欢迎关注个人微信公众号：**Charming写字的地方**
CSDN：http://blog.csdn.net/charmingwong
简书：http://www.jianshu.com/u/05686c7c92af
掘金：https://juejin.im/user/59924ed5f265da3e161aa74e
Github主页：https://charmingw.github.io/

欢迎转载~~~
