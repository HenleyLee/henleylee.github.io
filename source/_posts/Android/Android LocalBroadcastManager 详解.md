---
title: Android LocalBroadcastManager 详解
categories: Android
tags:
  - Android
abbrlink: 5cf56a1
date: 2019-05-10 19:06:30
---

Android 中有两种广播机制，一种是 `BroadcastReceiver`，另一种是 `LocalBroadcastManager`。
 - 应用场景
  - `BroadcastReceiver` 用于应用之间的传递消息；
  - `LocalBroadcastManager` 用于应用内部传递消息，比 `BroadcastReceiver` 更加高效。
 - 安全性
  - `BroadcastReceiver` 使用 Content API，所以本质上它是跨应用的，所以在使用它时必须要考虑到不要被别的应用滥用；
  - `LocalBroadcastManager` 不需要考虑安全问题，因为它只在应用内部有效。

## 简介 ##
`LocalBroadcastManager` 是 Android Support 包提供了一个工具，用于在同一个应用内的不同组件间发送 `Broadcast`。`LocalBroadcastManager` 也称为局部通知管理器，这种通知的好处是安全性高，效率也高，适合局部通信，可以用来代替 `Handler` 更新 `UI`。

`LocalBroadcastManager` 的优点：
 - 因广播数据在本应用范围内传播，你不用担心隐私数据泄露的问题。
 - 不用担心别的应用伪造广播，造成安全隐患。
 - 相比在系统内发送全局广播，它更高效。

> `BroadcastReceiver` 设计的初衷是从全局考虑可以方便应用程序和系统、应用程序之间、应用程序内的通信，所以对单个应用程序而言 `BroadcastReceiver` 是存在安全性问题的(恶意程序脚本不断的去发送你所接收的广播)。为了解决这个问题 `LocalBroadcastManager` 就应运而生了。

## 用法 ##
 - LocalBroadcastManager 对象的创建(需要传入 Context 对象)
```java
LocalBroadcastManager localBroadcastManager = LocalBroadcastManager.getInstance(context);
```

 - 注册广播接收器(需要传入 BroadcastReceiver 和 IntentFilter 对象)
```java
localBroadcastManager.registerReceiver(receiver, filter);
```

 - 发送广播(需要传入 Intent 对象)
```java
localBroadcastManager.sendBroadcast(intent);
```

 - 取消注册广播接收器(需要传入创建的 BroadcastReceiver 对象)
```java
localBroadcastManager.unregisterReceiver(receiver);
```

## 源码分析 ##
1. 构建 LocalBroadcastManager
```java
public final class LocalBroadcastManager {

    // 使用ApplicationContext，有效避免Context内存泄漏问题
    private final Context mAppContext;
    // 使用Handler机制发送和接收消息
    private final Handler mHandler;

    // 广播接收器作为key，接收器记录集合作为value
    private final HashMap<BroadcastReceiver, ArrayList<ReceiverRecord>> mReceivers
            = new HashMap<>();
    // 过滤动作(类型)作为key，接收器记录集合作为value
    private final HashMap<String, ArrayList<ReceiverRecord>> mActions = new HashMap<>();

    private static final Object mLock = new Object();
    private static LocalBroadcastManager mInstance;

    @NonNull
    public static LocalBroadcastManager getInstance(@NonNull Context context) {
        synchronized (mLock) {
            if (mInstance == null) {
                mInstance = new LocalBroadcastManager(context.getApplicationContext());
            }
            return mInstance;
        }
    }

    private LocalBroadcastManager(Context context) {
        mAppContext = context;
        mHandler = new Handler(context.getMainLooper()) {

            @Override
            public void handleMessage(Message msg) {
                switch (msg.what) {
                    case MSG_EXEC_PENDING_BROADCASTS:
                        executePendingBroadcasts();
                        break;
                    default:
                        super.handleMessage(msg);
                }
            }
        };
    }
}
```
从这部分源码可以看出两点：
 - 在获取 `LocalBroadcastManager` 对象实例的时候，这里用了单例模式。并且把外部传进来的 `Context` 转化成了 `ApplicationContext`，有效的避免了当前 `Context` 的内存泄漏的问题。
 - 在 `LocalBroadcastManager` 构造函数中创建了一个 `Handler`。可见 `LocalBroadcastManager` 的本质上是通过 `Handler` 机制发送和接收消息的。
 - 在创建 `Handler` 的时候，用了 `context.getMainLooper()`，说明这个 `Handler` 是在 Android 主线程中创建的，广播接收器的接收消息的时候会在 Android 主线程，所以我们决不能在广播接收器里面做耗时操作，以免阻塞 UI。

2. 内部类
 - ReceiverRecord
```java
    private static final class ReceiverRecord {
         final IntentFilter filter;            // 过滤器
        final BroadcastReceiver receiver;     // 广播接收器
        boolean broadcasting;                 // 广播是否处于激活状态
        boolean dead;                         // 广播是否已经销毁

        ReceiverRecord(IntentFilter _filter, BroadcastReceiver _receiver) {
            filter = _filter;
            receiver = _receiver;
        }

    }
```
`ReceiverRecord` 是 `LocalBroadcastManager` 的一个内部类，主要是存储广播接收器 `BroadcastReceiver` 和过滤器 `IntentFilter`，及广播运行状态。
 - BroadcastRecord
```java
    private static final class BroadcastRecord {
        final Intent intent;
        final ArrayList<ReceiverRecord> receivers;

        BroadcastRecord(Intent _intent, ArrayList<ReceiverRecord> _receivers) {
            intent = _intent;
            receivers = _receivers;
        }
    }
```
`BroadcastRecord` 也是 `LocalBroadcastManager` 的一个内部类，主要是存储发送的意图 `Intent` 和 广播接收器 `BroadcastReceiver`。

3. 注册广播接收器
```java
public void registerReceiver(BroadcastReceiver receiver, IntentFilter filter) {
    synchronized (mReceivers) {
        ReceiverRecord entry = new ReceiverRecord(filter, receiver);
        ArrayList<ReceiverRecord> filters = mReceivers.get(receiver);
        if (filters == null) {
            filters = new ArrayList<>(1);
            // 以BroadcastReceiver为键，ReceiverRecord集合为值存放到mReceivers中
            mReceivers.put(receiver, filters);
        }
        // 将ReceiverRecord存放到ReceiverRecord集合中
        filters.add(entry);
        // 遍历需要注册的action集合
        for (int i = 0; i < filter.countActions(); i++) {
            String action = filter.getAction(i);
            ArrayList<ReceiverRecord> entries = mActions.get(action);
            if (entries == null) {
                entries = new ArrayList<ReceiverRecord>(1);
                // 以action为键，action对应的ReceiverRecord集合为值，存放到mActions中
                mActions.put(action, entries);
            }
            // 将ReceiverRecord存放到action对应的ReceiverRecord集合中
            entries.add(entry);
        }
    }
}
```
`registerReceiver()` 方法只是做了一些数据的分类保存操作。

4. 发送广播
```java
public boolean sendBroadcast(@NonNull Intent intent) {
    // 保证同一时间，只有一个线程在操作
    synchronized (mReceivers) {
        // 取出Intent中的信息
        final String action = intent.getAction();
        final String type = intent.resolveTypeIfNeeded(
                mAppContext.getContentResolver());
        final Uri data = intent.getData();
        final String scheme = intent.getScheme();
        final Set<String> categories = intent.getCategories();

        final boolean debug = DEBUG ||
                ((intent.getFlags() & Intent.FLAG_DEBUG_LOG_RESOLUTION) != 0);
        
        // 获得对象动作(类型)的广播接收器记录集合
        ArrayList<ReceiverRecord> entries = mActions.get(intent.getAction());
        if (entries != null) {
            if (debug) Log.v(TAG, "Action list: " + entries);
            
            // 真正符合条件的接收器记录集
            ArrayList<ReceiverRecord> receivers = null;
            // 遍历集合，找到符合条件的ReceiverRecord
            for (int i = 0; i < entries.size(); i++) {
                ReceiverRecord receiver = entries.get(i);
                if (debug) Log.v(TAG, "Matching against filter " + receiver.filter);
                
                // 已经处于激活状态的ReceiverRecord过滤掉（避免重复加入）
                if (receiver.broadcasting) {
                    if (debug) {
                        Log.v(TAG, "  Filter's target already added");
                    }
                    continue;
                }

                // 判断本次循环取出的ReceiverRecord，在action、data、categories是否匹配
                int match = receiver.filter.match(action, type, scheme, data,
                        categories, "LocalBroadcastManager");
                if (match >= 0) {
                    if (debug) Log.v(TAG, "  Filter matched!  match=0x" +
                            Integer.toHexString(match));
                    if (receivers == null) {
                        receivers = new ArrayList<ReceiverRecord>();
                    }
                    // 符合条件的加入到receivers中，并且标记广播状态为已激活
                    receivers.add(receiver);
                    receiver.broadcasting = true;
                } else {
                    if (debug) {
                        String reason;
                        switch (match) {
                            case IntentFilter.NO_MATCH_ACTION:
                                reason = "action";
                                break;
                            case IntentFilter.NO_MATCH_CATEGORY:
                                reason = "category";
                                break;
                            case IntentFilter.NO_MATCH_DATA:
                                reason = "data";
                                break;
                            case IntentFilter.NO_MATCH_TYPE:
                                reason = "type";
                                break;
                            default:
                                reason = "unknown reason";
                                break;
                        }
                        Log.v(TAG, "  Filter did not match: " + reason);
                    }
                }
            }

            if (receivers != null) {
                // 筛选完后，遍历集合并将状态重置
                for (int i = 0; i < receivers.size(); i++) {
                    receivers.get(i).broadcasting = false;
                }
                // 重新构造一个BroadcastRecord对象，并添加到待发送状态下的广播列表
                mPendingBroadcasts.add(new BroadcastRecord(intent, receivers));
                // 查看handler在排队执行的消息中是否有MSG_EXEC_PENDING_BROADCASTS类型，
                // 没有，添加一个MSG_EXEC_PENDING_BROADCASTS类型的消息
                if (!mHandler.hasMessages(MSG_EXEC_PENDING_BROADCASTS)) {
                    mHandler.sendEmptyMessage(MSG_EXEC_PENDING_BROADCASTS);
                }
                return true;
            }
        }
    }
    return false;
}
```
`sendBroadcast()` 方法中主要进行的是广播接受器的筛选操作，然后通知 `Handler` 准备调用筛选出来的广播接收器。

5. 执行接收器的onReceive()方法
```java
void executePendingBroadcasts() {
    // 注意这里是死循环
    while (true) {
        // 使用数组存放广播接受记录
        final BroadcastRecord[] brs;
        synchronized (mReceivers) {
            final int N = mPendingBroadcasts.size();
            if (N <= 0) {
                return;
            }
            // 转换为数组格式并清空集合
            brs = new BroadcastRecord[N];
            mPendingBroadcasts.toArray(brs);
            mPendingBroadcasts.clear();
        }
        for (int i = 0; i < brs.length; i++) {
            final BroadcastRecord br = brs[i];
            final int nbr = br.receivers.size();
            // 遍历调用广播接收器方法
            for (int j = 0; j < nbr; j++) {
                final ReceiverRecord rec = br.receivers.get(j);
                // 判断广播接收器是否为销毁状态
                if (!rec.dead) {
                    // 执行广播接收器的onReceive()方法
                    rec.receiver.onReceive(mAppContext, br.intent);
                }
            }
        }
    }
}
```
`executePendingBroadcasts()` 方法用于执行接收器的 `onReceive()` 方法。

6. 取消注册广播接收器
```java
public void unregisterReceiver(@NonNull BroadcastReceiver receiver) {
    synchronized (mReceivers) {
        // 从mReceivers中移除，并返回对应的记录集合
        final ArrayList<ReceiverRecord> filters = mReceivers.remove(receiver);
        if (filters == null) {
            return;
        }
	// 遍历接收器记录集合
        for (int i = filters.size() - 1; i >= 0; i--) {
            final ReceiverRecord filter = filters.get(i);
	    // 设置dead = true，避免正在运行的executePendingBroadcasts()执行此接收器的回调
            filter.dead = true;
            for (int j = 0; j < filter.filter.countActions(); j++) {
                final String action = filter.filter.getAction(j);
                final ArrayList<ReceiverRecord> receivers = mActions.get(action);
                if (receivers != null) {
                    for (int k = receivers.size() - 1; k >= 0; k--) {
                        final ReceiverRecord rec = receivers.get(k);
                        if (rec.receiver == receiver) {
                            rec.dead = true;
                            receivers.remove(k);
                        }
                    }
                    if (receivers.size() <= 0) {
                        mActions.remove(action);
                    }
                }
            }
        }
    }
}
```
`unregisterReceiver()` 方法中主要是资源的释放，并将注册的 `BroadcastReceiver` 相关信息，从集合中删除，并设置 `BroadcastReceiver` 对应的 `ReceiverRecord` 的 `dead` 状态标记为 `true`，避免被执行。

## 封装 ##
```java
public final class LocalBroadcastHelper {

    private LocalBroadcastHelper() {
        throw new UnsupportedOperationException("Instantiation operation is not supported.");
    }

    /**
     * 返回LocalBroadcastManager实例
     */
    @NonNull
    public static LocalBroadcastManager getBroadcastManager(@NonNull Context context) {
        return LocalBroadcastManager.getInstance(context);
    }

    /**
     * 返回IntentFilter实例
     */
    @Nullable
    public static IntentFilter getIntentFilter(@NonNull String... actions) {
        IntentFilter filter = null;
        if (actions.length > 0) {
            filter = new IntentFilter();
            for (String action : actions) {
                filter.addAction(action);
            }
        }
        return filter;
    }

    /**
     * 通过Action注册广播接收器
     */
    public static void registerReceiver(@NonNull Context context,
                                        @NonNull BroadcastReceiver receiver, 
                                        @NonNull String... actions) {
        IntentFilter filter = getIntentFilter(actions);
        if (filter != null) {
            registerReceiver(context, receiver, filter);
        }
    }

    /**
     * 通过IntentFilter注册广播接收器
     */
    public static void registerReceiver(@NonNull Context context,
                                        @NonNull BroadcastReceiver receiver, 
                                        @NonNull IntentFilter filter) {
        getBroadcastManager(context).registerReceiver(receiver, filter);
    }

    /**
     * 取消注册广播接收器
     */
    public static void unRegisterReceiver(@NonNull Context context, BroadcastReceiver receiver) {
        getBroadcastManager(context).unregisterReceiver(receiver);
    }

    /**
     * 通过Action发送广播
     */
    public static void sendBroadcast(@NonNull Context context, @NonNull String action) {
        sendBroadcast(context, new Intent(action));
    }

    /**
     * 通过Intent发送广播
     */
    public static void sendBroadcast(@NonNull Context context, @NonNull Intent intent) {
        getBroadcastManager(context).sendBroadcast(intent);
    }

    /**
     * 通过Action同步发送广播
     */
    public static void sendBroadcastSync(@NonNull Context context, @NonNull String action) {
        sendBroadcastSync(context, new Intent(action));
    }

    /**
     * 通过Intent同步发送广播
     */
    public static void sendBroadcastSync(@NonNull Context context, @NonNull Intent intent) {
        getBroadcastManager(context).sendBroadcastSync(intent);
    }

}
```

## 注意事项 ##
虽然 `LocalBroadcastManager` 也通过 `BroadcastReceiver` 来接收消息，但是他们两个之间还是有很多区别的。
 - `LocalBroadcastManager` 注册广播只能通过代码注册的方式，传统的广播可以通过代码和 xml 两种方式注册。
 - `LocalBroadcastManager` 注册广播后，一定要记得取消监听，这一步可以有效的解决内存泄漏的问题。
 - `LocalBroadcastManager` 注册的广播，在发送广播的时候务必使用 `LocalBroadcastManager.sendBroadcast(intent)` 而不是 `context.sendBroadcast(intent)`，否则接收不到广播。

