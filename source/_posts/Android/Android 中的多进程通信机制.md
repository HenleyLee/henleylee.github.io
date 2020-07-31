---
title: Android 中的多进程通信机制
categories: Android
tags:
  - Android
abbrlink: 20c92935
date: 2019-02-28 18:56:36
---

## Android IPC 简介 ##
**`IPC`**是 `Inter-Process Communication` 的缩写，含义就是**`进程间通信`**或者**`跨进程通信`**，是指两个进程之间进行数据交换的过程。

那么什么是进程，什么是线程，在操作系统中，进程和线程是两个截然不同的概念。
线程：CPU 调度的最小单元，同时线程是一种有限的系统资源。
进程：是资源分配的最小单位，一般指一个执行单元，在 PC 和移动设备上指一个程序或应用。

一个进程可以包含多个线程，因此进程和线程是包含被包含的关系，最简单情况下，一个进程可以只有一个线程，即主线程，在 Android 里面也叫 UI 线程，在 UI 线程里才能操作界面元素。

> `IPC` 不是 `Android` 中所独有的，任何一个操作系统都需要相应的 `IPC` 机制，比如 `Windows` 上可以通过剪贴板等来进行进程间通信。`Android` 是一种基于 `Linux` 内核的移动操作系统，它的进程间通信方式并不是完全继承自 `Linux`，它有自己最有特色的进程间通信方式 `Binder`。

在 Android 中，一个应用程序就是一个独立的进程(应用运行在一个独立的环境中，可以避免其他应用程序/进程的干扰)。一般来说，当我们启动一个应用程序时，系统会创建一个进程(在 Android 系统中，所有的应用程序进程以及系统服务进程 SystemServer 都是由 Zygote 进程孕育 fork 出来的，并且每个进程都会有独立的 ID)，并为这个进程创建一个主线程(UI 线程)，然后就可以运行应用程序了，应用程序的组件默认都是运行在它的进程中。

## Android 多进程的优缺点 ##
### 多进程的优点 ###
 - 减少主进程所占用的内存，降低应用被系统杀死的概率；
 - 不会影响到主业务的代码的稳定运行，降低应用程序的崩溃率；
 - 有独立的生命周期，可以完全不依赖用户对应用的使用，可以独立启动、退出。

### 多进程的缺点 ###
 - Application 会多次创建，多进程模式中，不同进程的组件拥有相互独立的虚拟机、内存空间、Application；
 - 单例模式、静态变量完全失效，不同进程的内存空间相互独立，故不同的进程中，会有不同的实例，在一个进程中修改静态变量，在另外一个进程中失效；
 - 线程同步机制完全失效，因为空间独立，不管是锁对象还是锁全局类都无法保证线程同步，因为进程不同，锁住的对象不是同一个；
 - SharePreference 的可靠性下降，SharePreference 底层是对 XML 文件操作，不支持两个进程同时去执行写操作，因为它没有实现并发修改的机制，不能保证数据的安全性、准确性。

## Android 多进程的实现 ##
正常情况下，一个应用程序启动后只会运行在一个进程中，其进程名为应用程序的包名，所有的基本组件都会在这个进程中运行。但是如果需要将某些组件(如 Service、Activity 等)运行在单独的进程中，可以通过在 `AndroidManifest.xml` 中声明组件时，用 `android:process` 属性来指定其所运行的进程。

Android 中 `android:process` 的使用分为以下两种情况：
 - 以 `:` 命名开头的进程(如 android:process=":remote")：将运行在`默认包名:remote` 进程中，属于当前应用的`私有进程`，不允许其他APP的组件来访问。
 - 以`小写字母`开头完整命名的进程(如 android:process="package:remote")：将运行在 `package:remote` 进程中，属于`全局进程`，其他具有相同 shareUID 与签名的 APP 可以跑在这个进程中。

> `AndroidMantifest.xml` 中的 `activity`、`service`、`receiver` 和 `provider` 元素均支持 `android:process` 属性；`application` 元素也支持 `android:process` 属性，可以修改应用程序的默认进程名(默认值为包名)。

## Android IPC 基础概念 ##
### 序列化 ###
**`序列化`**表示将一个对象转换成可存储或可传输的状态。序列化后的对象可以在网络上进行传输，也可以存储到本地。
使用场景：需要通过 `Intent` 和 `Binder` 等传输**类对象**就必须完成对象的序列化过程。
两种方式：实现 `Serializable`/`Parcelable` 接口。

### Serializable 接口 ###
**`Serializable`**是 `Java` 提供的一个序列化接口，它是一个空接口，为对象提供标准的序列化和反序列化操作。使用 `Serializable` 来实现序列化相当简单，手动设置/系统自动生成 `serialVersionUID`。用法如下：
```java
public class User implements Serializable {

	private static final long serialVersionUID = 6429127138009130173L;

	private String name;
	private int age;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

}
```

### Parcelable 接口 ###
**`Parcelable`**是 `Android` 提供的序列化方式，`Parcel` 内部包装了可序列化的数据，可以在 `Binder` 中自由传输，在序列化过程中需要实现的功能有序列化、反序列化和内容描述序列化功能有 `writeToParcel()` 方法来完成，最终是通过 `Parcel` 中的一系列 `writeXXX()` 方法来完成的。用法如下：
```java
public class User implements Parcelable {

    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(this.name);
        dest.writeInt(this.age);
    }

    public User() {
    }

    private User(Parcel in) {
        this.name = in.readString();
        this.age = in.readInt();
    }

    public static final Parcelable.Creator<User> CREATOR = new Parcelable.Creator<User>() {
        @Override
        public User createFromParcel(Parcel source) {
            return new User(source);
        }

        @Override
        public User[] newArray(int size) {
            return new User[size];
        }
    };

}
```

`Serializable` 和 `Parcelable` 接口的比较：

|          | Serializable                             | Parcelable                                                 |
| :------: | :--------------------------------------: | :--------------------------------------------------------: |
| 平台     | Java	                              | Andorid                                                    |
| 原理     | 将一个对象转换成可存储或者可传输的状态   | 将对象进行分解，且分解后的每一部分都是传递可支持的数据类型 |
| 优点     | 使用简单 	                              | 高效                                                       |
| 缺点     | 缺点：开销大（需要进行大量的 I/O 操作）  | 使用麻烦                                                   |
| 使用场景 | 将对象序列化到存储设备或者通过网络传输   | 主要用在内存序列化上                                       |

### Binder ###
`Binder` 是什么？
 - 从 API 角度，`android.os.Binder` 是 Android 中的一个类，它实现了 `android.os.IBinder` 接口。
 - 从 IPC 角度来说，`Binder` 是 Android 中的一种跨进程通信方式，`Binder` 还可以理解为一种虚拟的物理设备，它的设备驱动是 `/dev/binder`，该通信方式在 `Linux` 中没有。
 - 从 Framework 角度来说，`Binder` 是 `ServiceManager` 连接各种 `Manager`(ActivityManager、WindowManager等等)和相应 `ManagerService` 的桥梁。
 - 从应用层来说，`Binder` 是客户端和服务端进行通信的媒介，当 `bindService` 的时候，服务端会返回一个包含了服务端业务调用的 `Binder` 对象，通过 `Binder` 对象，客户端就可以获取服务端提供的服务或者数据，这里的服务包括普通服务和基于 `AIDL` 的服务。

`Android` 是基于 `Linux` 内核基础上设计的，却没有把`管道`/`消息队列`/`共享内存`/`信号量`/`Socket` 等一些 `IPC` 通信手段作为 `Android` 的主要 `IPC` 方式，而是新增了 `Binder` 机制，其优点有以下几点:
 - 传输效率高、可操作性强：传输效率主要影响因素是内存拷贝的次数，拷贝次数越少，传输速率越高。
 - 实现 C/S 架构方便：`Linux` 的 `IPC` 方式除了 `Socket` 以外都不是基于 `C/S` 架构，而 `Socket` 主要用于网络间的通信且传输效率较低。`Binder` 基于 `C/S` 架构，`Server` 端与 `Client` 端相对独立，稳定性较好。
 - 安全性高：传统 `Linux` 的 `IPC` 的接收方无法获得对方进程可靠的 `UID/PID`，从而无法鉴别对方身份；而 `Binder` 机制为每个进程分配了 `UID/PID` 且在 `Binder` 通信时会根据 `UID/PID` 进行有效性检测。

`Binder` 驱动：
 - 与硬件设备没有关系，其工作方式与设备驱动程序是一样的，工作于内核态。
 - 提供 `open()`、`mmap()`、`poll()`、`ioctl()`等标准文件操作。
 - 以字符驱动设备中的 `misc` 设备注册在设备目录 `/dev` 下，用户通过 `/dev/binder` 访问该它。
 - 负责进程之间 `Binder` 通信的建立，传递，计数管理以及数据的传递交互等底层支持。
 - 驱动和应用程序之间定义了一套接口协议，主要功能由 `ioctl()` 接口实现，由于 `ioctl()` 灵活、方便且能够一次调用实现先写后读以满足同步交互，因此不必分别调用 `write()` 和 `read()` 接口。
 - 其代码位于 `linux` 目录的 `drivers/misc/binder.c` 中。

`Binder` 工作原理是什么？
 - 服务器端：在服务端创建好了一个 `Binder` 对象后，内部就会开启一个线程用于接收 `Binder` 驱动发送的消息，收到消息后会执行 `onTranscat()`，并按照参数执行不同的服务端代码。
 - Binder驱动：在服务端成功创建 `Binder` 对象后，`Binder` 驱动也会创建一个 `mRemote` 对象（也是 `Binder` 类），客户端可借助它调用 `transcat()` 即可向服务端发送消息。
 - 客户端：客户端要想访问 `Binder` 的远程服务，就必须获取远程服务的 `Binder` 对象在 `Binder` 驱动层对应的 `mRemote` 引用。当获取到 `mRemote` 对象的引用后，就可以调用相应 `Binder` 对象的暴露给客户端的方法。

## Android 中的 IPC 方式 ##
Android 实现跨进程通信的方式有很多，比如通过 `Intent` 来传递 `Bundle` 数据、`共享文件`、基于 `Binder` 的 `Messenger` 和 `AIDL` 以及 `Socket` 等。

### 通过 Bundle 传递数据 ###
**`通过 Bundle 传递数据`**是最常用的一种进程间通信方式，Android 四大组件中三大组件(Activity、Service、Receiver)都是支持在 `Intent` 中传递 `Bundle` 数据的，由于 `Bundle` 实现了 `Parcelable` 接口，所以它可以方便地在不同的进程间传输。

`Bundle` 支持传递的数据类型包括`基本数据类型`、`String`、`CharSequence` 以及实现了 `Serializable` 或 `Parcellable` 接口的数据结构，`Bundle` 不支持的数据类型无法在进程中被传递。

`Intent` 和 `Bundle` 的区别与联系：
 - `Intent` 底层其实是通过 `Bundle` 进行传递数据的；
 - `Intent` 使用比较简单，`Bundle` 使用比较复杂；
 - `Intent` 旨在数据传递，`Bundle` 旨在存取数据。

### 文件共享 ###
**`共享文件`**也是一种常用的的进程间通信方式，两个进程间通过读/写同一个文件来交换数据，比如一个进程把数据写入文件，另一个进程通过读取这个文件来获取数据。

`共享文件`适用于对数据同步要求不高的进程之间进行通信，并且要妥善处理并发读/写的问题。

`SharedPreferences` 也是文件存储的一种，但不建议用作进程间通信。因为系统对 `SharedPreferences` 的读/写有一定的缓存策略，即在内存中有一份该文件的缓存，因此在多进程模式下，其读/写会变得不可靠，甚至丢失数据。

> Linux 机制下，可以对文件并发写，所以要注意同步；Windows 下不支持并发读或写。

### AIDL ###
**`AIDL`**(Android Interface Definition Language，Android接口定义语言)是远程服务跨进程通信的一种方式。如果在一个进程中要调用另一个进程中对象的方法，可使用 `AIDL` 生成可序列化的参数，`AIDL` 会生成一个服务端对象的代理类，通过它客户端实现间接调用服务端对象的方法。

`AIDL` 支持的数据类型如下：
 - 基本数据类型
 - `String` 和 `CharSequence`
 - `ArrayList`、`HashMap` 且里面的每个元素都能被 `AIDL` 支持
 - 实现 `Parcelable` 接口的对象
 - 所有 `AIDL` 接口本身

> 注意：除了基本数据类型，其它类型的参数必须标上方向：`in`、`out`或 `inout`，用于表示在跨进程通信中数据的流向。

`AIDL` 本质是系统提供了一套可快速实现 `Binder` 的工具，关键的类和方法是：
 - **`AIDL 接口：`**继承 `android.os.IInterface`。
 - **`Stub 类：`**`Binder` 的实现类，服务端通过这个类来提供服务。
 - **`Proxy 类：`**服务器的本地代理，客户端通过这个类调用服务器的方法。
 - **`asInterface() 方法：`**客户端调用，将服务端的返回的 `Binder` 对象(若客户端和服务端位于同一进程，则直接返回 `Stub` 对象本身；否则，返回的是系统封装后的 `Stub.proxy` 对象)，转换成客户端所需要的 `AIDL` 接口类型对象。
 - **`asBinder() 方法：`**返回代理 `Proxy` 的 `Binder` 对象。
 - **`onTransact() 方法：`**运行服务端的 `Binder` 线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法来处理。
 - **`transact() 方法：`**运行在客户端，当客户端发起远程请求的同时将当前线程挂起。之后调用服务端的 `onTransact()` 方法直到远程请求返回，当前线程才继续执行。

`AIDL` 的实现方法如下：
 - 服务端：
  - 创建一个 `aidl` 文件；
  - 创建一个 `Service`，实现 `AIDL` 的接口函数并暴露 `AIDL` 接口。
 - 客户端：
  - 通过 `bindService()` 绑定服务端的 `Service`；
  - 绑定成功后，将服务端返回的 `Binder` 对象转化成 `AIDL` 接口所属的类型，进而调用相应的 `AIDL` 中的方法。

> 服务端里的某个 `Service` 给和它绑定的特定客户端进程提供 `Binder` 对象，客户端通过 `AIDL` 接口的静态方法 `asInterface()` 将 `Binder` 对象转化成 `AIDL` 接口的代理对象，通过这个代理对象就可以发起远程调用请求。

### Messenger ###
**`Messenger`**可以翻译为信使，顾名思义，通过它可以在不同进程中传递 `Message` 对象，在 `Message` 中放入我们需要传递的数据，就可以轻松地实现数据的进程间传递。`Messenger` 是一种轻量级的 IPC 方案，它是基于 `AIDL` 实现的。

`Messenger` 的特点如下：
 - 底层实现是 `AIDL`，即对 `AIDL` 进行了封装，更便于进行进程间通信。
 - 其服务端以串行的方式来处理客户端的请求，不存在并发执行的情形，故无需考虑线程同步的问题。
 - 可在不同进程中传递 `Message` 对象，`Messager` 可支持的数据类型即 `Messenge` 可支持的数据类型。

`Messenger` 的实现方法如下：
 - 服务端：
  - 创建一个 `Service` 用于提供服务；
  - 其中创建一个 `Handler` 用于接收客户端进程发来的数据；
  - 利用 `Handler` 创建一个 `Messenger` 对象；
  - 在 `Service` 的 `onBind()` 中返回 `Messenger` 对应的 `Binder` 对象。
 - 客户端：
  - 通过 `bindService()` 绑定服务端的 `Service`；
  - 通过绑定后返回的 `IBinder` 对象创建一个 `Messenger`，进而可向服务器端进程发送 `Message`数据(至此只完成单向通信)。
  - 在客户端创建一个 `Handler` 并由此创建一个 `Messenger`，并通过 `Message` 的 `replyTo` 字段传递给服务器端进程。服务端通过读取 `Message` 得到 `Messenger` 对象，进而向客户端进程传递数据(完成双向通信)。

`Messenger` 的缺点如下：
 - 主要作用是传递 `Message`，难以实现远程方法调用。
 - 以串行的方式处理客户端发来的消息的，不适合高并发的场景。

### ContentProvider ###
**`ContentProvider`**是 `Android` 中提供的专门用于不同应用间进行数据共享的方式，它的底层实现同样也是 `Binder`，主要用来为其他应用提供数据，可以说天生就是为进程间通信而生的。

> 实现一个 `ContentProvider` 需要实现6个方法，其中 `onCreate()` 方法是主线程中回调的，其他的 `query()`、`update()`、`insert()`、`delete()` 和 `getType()` 都运行在 `Binder` 线程池中。
> 自定义的 `ContentProvider` 注册时要提供 `authorities` 属性，应用需要访问的时候将属性包装成 `Uri.parse("content://authorities")`。
> 还可以设置 `permission`、`readPermission`、`writePermission` 来设置权限。
> `ContentProvider` 有 `query()`、`delete()`、`insert()`、`update()` 等方法，看起来像是是一个数据库管理类，但其实可以用文件、内存数据等等一切来充当数据源，`query()` 方法返回的是一个 `Cursor` 对象，可以自定义继承 `AbstractCursor` 的类来实现。

### Socket ###
**`Socket`**不仅可以跨进程，还可以跨设备通信。

`Socket` 也称为`套接字`，是网络通信中的概念，它分为流套接字和数据流套接字两种：
 - 流套接字：基于 `TCP` 协议，采用流的方式提供可靠的字节流服务。
 - 数据流套接字：基于 `UDP` 协议，采用数据报文提供数据打包发送的服务。

`Socket` 的实现方法如下：
 - 服务端：
  - 创建一个 `Service`，在线程中建立 `TCP` 服务、监听相应的端口等待客户端连接请求；
  - 与客户端连接时，会生成新的 `Socket` 对象，利用它可与客户端进行数据传输；
  - 与客户端断开连接时，关闭相应的 `Socket` 并结束线程。
 - 客户端：
  - 开启一个线程、通过 `Socket` 发出连接请求；
  - 连接成功后，读取服务端消息；
  - 断开连接，关闭 `Socket`。

> 需要注意的是：`Android 不允许在主线程中请求网络`，而且请求网络必须要注意声明相应的 `permission`。然后，在服务器中定义 `ServerSocket` 来监听端口，客户端使用 `Socket` 来请求端口，连通后就可以进行通信。

## IPC 方式的优缺点和适用场景 ##

| 名称            | 优点                                                     | 缺点                                                                                              | 适用场景                                                         |
|-----------------|----------------------------------------------------------|---------------------------------------------------------------------------------------------------|------------------------------------------------------------------|
| Bundle          | 简单易用                                                 | 只能传输 Bundle 支持的数据类型                                                                    | 四大组件间的进程通信                                             |
| 文件共享        | 简单易用                                                 | 不适合高并发的情况，并且无法做到进程间的即时通讯                                                  | 无并发访问情况下，交换简单的数据实时性不高的情况                 |
| AIDL            | 功能强大，支持一对多并发通信，支持实时通讯               | 需要处理好线程同步                                                                                | 一对多通信且有 RPC 需求                                          |
| Messager        | 支持一对多串行通信，支持实时通讯                         | 不能很好处理高并发情况，不支持 RPC，数据通过 Message 进行传输，因此只能传输 Bundle 支持的数据类型 | 低并发的一对多即时通信，无 RPC 需求，或者无需返回结果的 RPC 需求 |
| ContentProvider | 在数据源访问方面功能强大，支持一对多并发数据共享         | 主要提供数据源的 Crud                                                                             | 一对多的进程间数据共享                                           |
| Socket          | 功能强大，可以通过网络传输字节流，支持一对多并发实时通讯 | 实现细节有点繁琐，不支持直接的 RPC                                                                | 网络数据交换                                                     |

