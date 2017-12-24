## 前言
在写这篇源码解析到一半时发现EventBus有很多高级用法是不能直接忽略的，于是回过头来写了EventBus高级用法这一篇，传送门：[](http://note.youdao.com/)
这篇主要通过源码分析来了解EventBus的实现原理，EventBus的版本为3.1.1，主要分为三部分，也就是实现的三个步骤
1. 注册订阅者
2. 事件发布
3. 反注册订阅者

话不多说，马上开始

## 源码分析

### 注册订阅者
首先我们从EventBus注册订阅者的这一句入手
```
EventBus.getDefault().register(this);
```
getDefault()表面意思大概是获取一个EventBus的实例，看一下源码
```
/** Convenience singleton for apps using a process-wide EventBus instance. */
public static EventBus getDefault() {
    if (defaultInstance == null) {
        synchronized (EventBus.class) {
            if (defaultInstance == null) {
                defaultInstance = new EventBus();
            }
        }
    }
    return defaultInstance;
}
```
一个DoubleCheck的单例模式获取到实例，能高效的面对并发获取实例的情况  
再看一下EventBus的构造方法

```
public EventBus() {
    this(DEFAULT_BUILDER);
}

EventBus(EventBusBuilder builder) {
    subscriptionsByEventType = new HashMap<>();
    typesBySubscriber = new HashMap<>();
    stickyEvents = new ConcurrentHashMap<>();
    mainThreadPoster = new HandlerPoster(this, Looper.getMainLooper(), 10);
    backgroundPoster = new BackgroundPoster(this);
    asyncPoster = new AsyncPoster(this);
    indexCount = builder.subscriberInfoIndexes != null ? builder.subscriberInfoIndexes.size() : 0;
    subscriberMethodFinder = new SubscriberMethodFinder(builder.subscriberInfoIndexes,
            builder.strictMethodVerification, builder.ignoreGeneratedIndex);
    logSubscriberExceptions = builder.logSubscriberExceptions;
    logNoSubscriberMessages = builder.logNoSubscriberMessages;
    sendSubscriberExceptionEvent = builder.sendSubscriberExceptionEvent;
    sendNoSubscriberEvent = builder.sendNoSubscriberEvent;
    throwSubscriberException = builder.throwSubscriberException;
    eventInheritance = builder.eventInheritance;
    executorService = builder.executorService;
}
```
默认的构造方法又调用参数为EventBusBuilder的构造方法，构造出EventBus的实例，这里使用的是默认的一个EventBusBuilder，从名字就可以看出EventBusBuilder采用的是Builder模式，个性化配置EventBus的各项参数保存在Builder中，build时构造EventBus并将各项配置应用于EventBus

```
/** Builds an EventBus based on the current configuration. */
public EventBus build() {
    return new EventBus(this);
}
```

接下来看register()
```
public void register(Object subscriber) {
    Class<?> subscriberClass = subscriber.getClass();
    // 获取订阅者的订阅方法并用List封装
    List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
    synchronized (this) {
        // 逐个订阅
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod);
        }
```
register()接收参数为Object类型的订阅者，通常也就是代码中Activity和Fragment的实例this。subscriberMethodFinder是EventBus的一个成员，可以看作是一个订阅方法查找器，是在刚刚的EventBus构造方法通过EventBusBuilder的一些参数构造出来的。调用findSubscriberMethods方法，传入订阅者的Class对象，字面意思是找出订阅者中所有的订阅方法，用一个List集合来接收，看看是不是

```
List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
    List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
    if (subscriberMethods != null) {
        return subscriberMethods;
    }
    if (ignoreGeneratedIndex) {
        // 使用反射方式获取
        subscriberMethods = findUsingReflection(subscriberClass);
    } else {
        // 使用SubscriberIndex方式获取
        subscriberMethods = findUsingInfo(subscriberClass);
    }
    // 若订阅者中没有订阅方法，则抛异常
    if (subscriberMethods.isEmpty()) {
        throw new EventBusException("Subscriber " + subscriberClass
                + " and its super classes have no public methods with the @Subscribe annotation");
    } else {
        // 缓存订阅者的订阅方法List
        METHOD_CACHE.put(subscriberClass, subscriberMethods);
        return subscriberMethods;
    }
```
METHOD_CACHE是一个Map，以订阅者的Class对象为key，订阅者中的订阅方法List为value，缓存了注册过的订阅方法，如果有缓存则返回返回缓存，如果没有则继续往下执行。这里看到ignoreGeneratedIndex这个属性，意思为是否忽略生成index，是在构造SubscriberMethodFinder通过EventBusBuilder的同名属性赋值的，默认为false，当为true时，表示以反射的方式获取订阅者中的订阅方法，当为false时，则以Subscriber Index的方式获取。接下来分别分析这两种方式。  
首先是ignoreGeneratedIndex为true时，追踪源码

```
private List<SubscriberMethod> findUsingReflection(Class<?> subscriberClass) {
    // 创建并初始化FindState对象
    FindState findState = prepareFindState();
    // findState与subscriberClass关联
    findState.initForSubscriber(subscriberClass);
    while (findState.clazz != null) {
        // 使用反射的方式获取单个类的订阅方法
        findUsingReflectionInSingleClass(findState);
        // 使findState.clazz指向父类的Class，继续获取
        findState.moveToSuperclass();
    }
    // 返回订阅者极其父类的订阅方法List，并释放资源
    return getMethodsAndRelease(findState);
}
```
创建一个FindState对象来辅助获取订阅方法，这里只看反射获取的关键代码：findUsingReflectionInSingleClass

```
private void findUsingReflectionInSingleClass(FindState findState) {
    Method[] methods;
    try {
        // This is faster than getMethods, especially when subscribers are fat classes like Activities
        methods = findState.clazz.getDeclaredMethods();
    } catch (Throwable th) {
        // Workaround for java.lang.NoClassDefFoundError, see https://github.com/greenrobot/EventBus/issues/149
        methods = findState.clazz.getMethods();
        findState.skipSuperClasses = true;
    }
    for (Method method : methods) {
        int modifiers = method.getModifiers();
        // 忽略非public的方法和static等修饰的方法
        if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
            // 获取订阅方法的所有参数
            Class<?>[] parameterTypes = method.getParameterTypes();
            // 订阅方法只能有一个参数，否则忽略
            if (parameterTypes.length == 1) {
                // 获取注解
                Subscribe subscribeAnnotation = method.getAnnotation(Subscribe.class);
                if (subscribeAnnotation != null) {
                    // 获取第一个参数
                    Class<?> eventType = parameterTypes[0];
                    // 检查eventType决定是否订阅，通常订阅者不能有多个eventType相同的订阅方法
                    if (findState.checkAdd(method, eventType)) {
                        // 获取线程模式
                        ThreadMode threadMode = subscribeAnnotation.threadMode();
                        // 添加订阅方法进List
                        findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode,
                                subscribeAnnotation.priority(), subscribeAnnotation.sticky()));
                    }
                }
            } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
                String methodName = method.getDeclaringClass().getName() + "." + method.getName();
                throw new EventBusException("@Subscribe method " + methodName +
                        "must have exactly 1 parameter but has " + parameterTypes.length);
            }
        } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
            String methodName = method.getDeclaringClass().getName() + "." + method.getName();
            throw new EventBusException(methodName +
                    " is a illegal @Subscribe method: must be public, non-static, and non-abstract");
        }
    }
}
```
代码也不难懂，经过修饰符、参数个数、是否有注解、和订阅者是否有eventType相同的方法几层条件的筛选，最终将订阅方法添加进findState的subscriberMethods这个List中。来到这里，不得不说EventBus真是非常强大，它不仅仅获取当前类的订阅方法，还会获取它所有父类的订阅方法

```
// 使findState.clazz指向父类的Class，继续获取
findState.moveToSuperclass();
```
通过改变寻找状态对象findState的clazz属性，使之指向父类的Class，来遍历出当前类整个家族的订阅方法，最终获取到所有的订阅方法后返回List并释放资源。  
接下来就看看ignoreGeneratedIndex为false的情况

```
private List<SubscriberMethod> findUsingInfo(Class<?> subscriberClass) {
    FindState findState = prepareFindState();
    findState.initForSubscriber(subscriberClass);
    while (findState.clazz != null) {
        // 获取当前clazz对应的SubscriberInfo
        findState.subscriberInfo = getSubscriberInfo(findState);
        if (findState.subscriberInfo != null) {
            // 通过SubscriberInfo获取阅方法数组
            SubscriberMethod[] array = findState.subscriberInfo.getSubscriberMethods();
            // 逐个添加进findState.subscriberMethods
            for (SubscriberMethod subscriberMethod : array) {
                if (findState.checkAdd(subscriberMethod.method, subscriberMethod.eventType)) {
                    findState.subscriberMethods.add(subscriberMethod);
                }
            }
        } else {
            // 若SubscriberInfo为空，则采用反射方式获取
            findUsingReflectionInSingleClass(findState);
        }
        findState.moveToSuperclass();
    }
```
跟反射方式的findUsingReflection的首尾有点类似，不同的是它是通过SubscriberInfo这个类来获取订阅方法的，那么SubscriberInfo对象是怎么获取的呢，那么同样只看关键代码：getSubscriberInfo

```
private SubscriberInfo getSubscriberInfo(FindState findState) {
    if (findState.subscriberInfo != null && findState.subscriberInfo.getSuperSubscriberInfo() != null) {
        SubscriberInfo superclassInfo = findState.subscriberInfo.getSuperSubscriberInfo();
        if (findState.clazz == superclassInfo.getSubscriberClass()) {
            return superclassInfo;
        }
    }
    if (subscriberInfoIndexes != null) {
        for (SubscriberInfoIndex index : subscriberInfoIndexes) {
            // 通过SubscriberIndex获取findState.clazz对应的SubscriberInfo
            SubscriberInfo info = index.getSubscriberInfo(findState.clazz);
            if (info != null) {
                return info;
            }
        }
    }
    return null;
}
```
这时候主角出现了，我们看subscriberInfoIndexes，它是一个List，类型为Subscriber Index，意思订阅者索引，也是这种方式的重要角色。追踪它的出处

```
SubscriberMethodFinder(List<SubscriberInfoIndex> subscriberInfoIndexes, boolean strictMethodVerification,
                       boolean ignoreGeneratedIndex) {
    this.subscriberInfoIndexes = subscriberInfoIndexes;
    this.strictMethodVerification = strictMethodVerification;
    this.ignoreGeneratedIndex = ignoreGeneratedIndex;
}
```
是在构造SubscriberMethodFinder时赋值的，而SubscriberMethodFinder是在EventBus的构造方法中通过EventBusBuilder的参数构造的

```
subscriberMethodFinder = new SubscriberMethodFinder(builder.subscriberInfoIndexes,
        builder.strictMethodVerification, builder.ignoreGeneratedIndex);
```
继续追踪EventBusBuilder中的subscriberInfoIndexes，发现如下代码

```
/** Adds an index generated by EventBus' annotation preprocessor. */
public EventBusBuilder addIndex(SubscriberInfoIndex index) {
    if(subscriberInfoIndexes == null) {
        subscriberInfoIndexes = new ArrayList<>();
    }
    subscriberInfoIndexes.add(index);
    return this;
}
```
通过注释可知，Subscriber Index是由EventBus注解处理器生成的，那么EventBus注解处理器是什么呢？如果你想用索引的方式来获取订阅方法，这时候就很有必要看一下官方文档，它教你如何使用Subscriber Index，非常重要  
#### [EventBus Subscriber Index 使用教程](http://greenrobot.org/eventbus/documentation/subscriber-index/)

看完文档，我们知道Subscriber Index是EventBus 3上的新技术，所以这里也建议还没学习过EventBus的可以跳过2.X之前的版本直接学习最新版本。  
关于EventBus的Subscriber Index技术的特点，翻译一下官方解释：
>  It is an optional optimization to speed up initial subscriber registration.

Subscriber Index是一个可选的优化技术，用来加速初始化订阅者注册。
> The subscriber index can be created during build time using the EventBus annotation processor. While it is not required to use an index, it is recommended on Android for best performance.

Subscriber Index在编译时使用EventBus注解处理器创建，虽然没有规定必须使用它，但是官方推荐使用这种方式，因为它在Android上有着最佳的性能。  

回到代码中来，findState.subscriberInfo始终指向当前正在获取订阅方法的订阅者的subscriberInfo，看关键代码

```
SubscriberMethod[] array = findState.subscriberInfo.getSubscriberMethods();
```
通过getSubscriberMethods()获取到订阅方法数组，然后与反射方式类似地，逐个将订阅方法添加进findState的subscriberMethods这个List中，返回该List并释放资源。

```
// 逐个添加进findState.subscriberMethods
for (SubscriberMethod subscriberMethod : array) {
    if (findState.checkAdd(subscriberMethod.method, subscriberMethod.eventType)) {
        findState.subscriberMethods.add(subscriberMethod);
    }
}
```

到此，两种方式讲解完毕。无论通过哪种方式获取，获取到订阅方法List之后，接下来是真正订阅的过程，回到register()中看代码

```
synchronized (this) {
    for (SubscriberMethod subscriberMethod : subscriberMethods) {
        subscribe(subscriber, subscriberMethod);
    }
}
```
遍历逐个订阅，看subscribe方法

```
// Must be called in synchronized block
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
    Class<?> eventType = subscriberMethod.eventType;
    // 创建Subscription封装订阅者和订阅方法信息
    Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
    // 根据事件类型从subscriptionsByEventType这个Map中获取Subscription集合
    CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    // 若Subscription集合为空，创建并put进Map中
    if (subscriptions == null) {
        subscriptions = new CopyOnWriteArrayList<>();
        subscriptionsByEventType.put(eventType, subscriptions);
    } else {
        // 若集合中已包含该Subscription则抛异常
        if (subscriptions.contains(newSubscription)) {
            throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                    + eventType);
        }
    }
    int size = subscriptions.size();
    for (int i = 0; i <= size; i++) {
        // 按照优先级插入Subscription
        if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
            subscriptions.add(i, newSubscription);
            break;
        }
    }
    // typesBySubscriber与subscriptionsByEventType类似
    // 用来存放订阅者中的事件类型
    List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
    if (subscribedEvents == null) {
        subscribedEvents = new ArrayList<>();
        typesBySubscriber.put(subscriber, subscribedEvents);
    }
    subscribedEvents.add(eventType);
    // 订阅方法是否设置黏性模式
    if (subscriberMethod.sticky) {
        // 是否设置了事件继承
        if (eventInheritance) {
            // Existing sticky events of all subclasses of eventType have to be considered.
            // Note: Iterating over all events may be inefficient with lots of sticky events,
            // thus data structure should be changed to allow a more efficient lookup
            // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
            Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
            for (Map.Entry<Class<?>, Object> entry : entries) {
                Class<?> candidateEventType = entry.getKey();
                // 判断当前事件类型是否为黏性事件或者其子类
                if (eventType.isAssignableFrom(candidateEventType)) {
                    Object stickyEvent = entry.getValue();
                    // 执行设置了sticky模式的订阅方法
                    checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                }
            }
        } else {
            Object stickyEvent = stickyEvents.get(eventType);
            checkPostStickyEventToSubscription(newSubscription, stickyEvent);
        }
    }
}
```
方便起见，关键代码都附上了注释，简单讲解一下关键角色： 
- subscriptionsByEventType：以事件类型为key，拥有相同事件类型的订阅方法List为value，存放所有的订阅方法。
- typesBySubscriber：以订阅者为key，订阅者订阅的所有事件类型List为value，存放所有的事件类型。

在这里出现了stickyEvent（黏性事件），接下来会在事件分布中讲解它的作用，以上便是注册订阅者的过程。


### 事件发布
一般的事件发布方式
```
EventBus.getDefault().post(new UpdateUIEvent());
```
这里我们看post()

```
/** Posts the given event to the event bus. */
public void post(Object event) {
    // 获取当前线程的posting状态
    PostingThreadState postingState = currentPostingThreadState.get();
    List<Object> eventQueue = postingState.eventQueue;
    // 将事件添加进当前线程的事件队列
    eventQueue.add(event);
    // 判断当前线程是否正在发布事件
    if (!postingState.isPosting) {
        postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper();
        postingState.isPosting = true;
        // 取消发布状态没有重置，抛异常
        if (postingState.canceled) {
            throw new EventBusException("Internal error. Abort state was not reset");
        }
        try {
            while (!eventQueue.isEmpty()) {
                PostingThreadState(eventQueue.remove(0), postingState);
            }
        } finally {
            postingState.isPosting = false;
            postingState.isMainThread = false;
        }
    }
}
```
EventBus用ThreadLocal存储每个线程的PostingThreadState，一个存储了事件发布状态的类，当post一个事件时，添加到事件队列末尾，等待前面的事件发布完毕后再拿出来发布，这里看事件发布的关键代码PostingThreadState()

```
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
    Class<?> eventClass = event.getClass();
    boolean subscriptionFound = false;
    if (eventInheritance) {
        List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
        int countTypes = eventTypes.size();
        for (int h = 0; h < countTypes; h++) {
            Class<?> clazz = eventTypes.get(h);
            // 发布事件
            subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
        }
    } else {
        // 发布事件
        subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
    }
    if (!subscriptionFound) {
        if (logNoSubscriberMessages) {
            Log.d(TAG, "No subscribers registered for event " + eventClass);
        }
        if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                eventClass != SubscriberExceptionEvent.class) {
            post(new NoSubscriberEvent(this, event));
        }
    }
}
```
代码也非常简单，首先看eventInheritance这个属性，是否开启事件继承，若是，找出发布事件的所有父类，也就是lookupAllEventTypes()，然后遍历每个事件类型进行发布，若不是，则直接发布该事件。如果需要发布的事件没有找到任何匹配的订阅信息，则发布一个NoSubscriberEvent事件。这里只看发布事件的关键代码postSingleEventForEventType()

```
private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
    CopyOnWriteArrayList<Subscription> subscriptions;
    synchronized (this) {
        // 根据事件类型找出相关的订阅信息
        subscriptions = subscriptionsByEventType.get(eventClass);
    }
    if (subscriptions != null && !subscriptions.isEmpty()) {
        for (Subscription subscription : subscriptions) {
            postingState.event = event;
            postingState.subscription = subscription;
            boolean aborted = false;
            try {
                // 发布事件到具体的订阅者
                postToSubscription(subscription, event, postingState.isMainThread);
                aborted = postingState.canceled;
            } finally {
                postingState.event = null;
                postingState.subscription = null;
                postingState.canceled = false;
            }
            if (aborted) {
                break;
            }
        }
        return true;
    }
    return false;
}
```
来到这里，开始根据事件类型匹配出订阅信息，如果该事件有订阅信息，则执行postToSubscription()，发布事件到每个订阅者，返回true，若没有，则发回false。继续追踪发布事件到具体订阅者的代码

```
private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
    switch (subscription.subscriberMethod.threadMode) {
        // 订阅线程跟随发布线程
        case POSTING:
            // 订阅线程和发布线程相同，直接订阅
            invokeSubscriber(subscription, event);
            break;
        // 订阅线程为主线程
        case MAIN:
            if (isMainThread) {
                // 发布线程和订阅线程都是主线程，直接订阅
                invokeSubscriber(subscription, event);
            } else {
                // 发布线程不是主线程，订阅线程切换到主线程订阅
                mainThreadPoster.enqueue(subscription, event);
            }
            break;
        // 订阅线程为后台线程
        case BACKGROUND:
            if (isMainThread) {
                // 发布线程为主线程，切换到后台线程订阅
                backgroundPoster.enqueue(subscription, event);
            } else {
                // 发布线程不为主线程，直接订阅
                invokeSubscriber(subscription, event);
            }
            break;
        // 订阅线程为异步线程
        case ASYNC:
            // 使用线程池线程订阅
            asyncPoster.enqueue(subscription, event);
            break;
        default:
            throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
    }
}
```
看到四种线程模式，我们回顾一下上一篇的内容
- POSTING：事件发布在什么线程，就在什么线程订阅。
- MAIN：无论事件在什么线程发布，都在主线程订阅。
- BACKGROUND：如果发布的线程不是主线程，则在该线程订阅，如果是主线程，则使用一个单独的后台线程订阅。
- ASYNC：在非主线程和发布线程中订阅。

订阅者四种线程模式的特点对应的就是以上代码，简单来讲就是订阅者指定了在哪个线程订阅事件，无论发布者在哪个线程，它都会将事件发不到订阅者指定的线程。这里我们继续看实现订阅者的方法

```
void invokeSubscriber(Subscription subscription, Object event) {
    try {
        subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
    } catch (InvocationTargetException e) {
        handleSubscriberException(subscription, event, e.getCause());
    } catch (IllegalAccessException e) {
        throw new IllegalStateException("Unexpected exception", e);
    }
}
```
到这里我们终于看到了Java的API，有种拨开云雾见月明的感觉

```
subscription.subscriberMethod.method.invoke(subscription.subscriber, event)
```
订阅者接收到了事件，调用订阅方法，传入发布的事件作为参数，至此，事件发布过程就结束了。

### 反注册订阅者
先看反注册的代码

```
EventBus.getDefault().unregister(this);
```
跟踪unregister()
```
/** Unregisters the given subscriber from all event classes. */
public synchronized void unregister(Object subscriber) {
    List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
    if (subscribedTypes != null) {
        for (Class<?> eventType : subscribedTypes) {
            unsubscribeByEventType(subscriber, eventType);
        }
        typesBySubscriber.remove(subscriber);
    } else {
        Log.w(TAG, "Subscriber to unregister was not registered before: " + subscriber.getClass());
    }
}
```
注册过程我们就知道typesBySubscriber是保存订阅者的所有订阅事件类型的一个Map，这里根据订阅者拿到订阅事件类型List，然后逐个取消订阅，最后typesBySubscriber移除该订阅者,。这里只需要关注它是如果取消订阅的，跟踪unsubscribeByEventType()

```
/** Only updates subscriptionsByEventType, not typesBySubscriber! Caller must update typesBySubscriber. */
private void unsubscribeByEventType(Object subscriber, Class<?> eventType) {
    List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    if (subscriptions != null) {
        int size = subscriptions.size();
        for (int i = 0; i < size; i++) {
            Subscription subscription = subscriptions.get(i);
            if (subscription.subscriber == subscriber) {
                subscription.active = false;
                subscriptions.remove(i);
                i--;
                size--;
            }
        }
    }
}
```
subscriptionsByEventType是存储事件类型对应订阅信息的Map，代码逻辑非常清晰，找出某事件类型的订阅信息List，遍历订阅信息，将要取消订阅的订阅者和订阅信息封装的订阅者比对，如果是同一个，则说明该订阅信息是将要失效的，于是将该订阅信息移除。  

到这里，反注册订阅者的过程就讲解完毕啦。

## 总结

回顾一下EventBus的三个步骤
1. 注册订阅者
2. 事件发布
3. 反注册订阅者

通过分析EventBus源码，发现注册的过程最为复杂，发布次之，反注册最简单，但这也是相对而言，总的来讲，EventBus这个库的源码没有太多晦涩难懂的地方，算是非常好理解的，这里不得不佩服EvenBus的作者们，整个库的架构、逻辑设计，接口设计，小到方法和变量的命名，都显得非常优雅易懂，相比之下，曾阅读过的Glide的源码就显得......哈哈，不过这两个库的代码量也不是同一个级别的，不能简单地直接比较。  
好了，EventBus的源码解析到这就结束了，想进一步了解EventBus的朋友可以亲自去阅读源码，毕竟自己亲身去探索，比听别人的理解更深。这里借用Linus的一句话，与大家共勉
> Read the fucking source code

### **感谢阅读！**