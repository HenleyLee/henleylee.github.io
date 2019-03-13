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

那么什么是进程，什么是线程，在操作系统中，进程和线程是两个截然不同的概念。线程是 CPU 调度的最小单元，同时线程是一种有限的系统资源。而进程指的一个执行单元，在 PC 和移动设备上指的是一个程序或者一个应用。一个进程可以包含多个线程，因此进程和线程是包含被包含的关系，最简单情况下，一个进程可以只有一个线程，即主线程，在 Android 里面也叫 UI 线程，在 UI 线程里才能操作界面元素。

> IPC 不是 Android 中所独有的，任何一个操作系统都需要相应的 IPC 机制，比如 Windows 上可以通过剪贴板等来进行进程间通信。Android 是一种基于 Linux 内核的移动操作系统，它的进程间通信方式并不能完全继承自 Linux，它有自己的进程间通信方式。

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
 - `android:process` 的值以 `:` 开头(如 android:process=":remote")，将运行在`默认包名:remote` 进程中，属于`私有进程`，不允许其他APP的组件来访问。
 - `android:process` 的值以`小写字母`开头(如 android:process="package:remote")，将运行在 `package:remote` 进程中，属于`全局进程`，其他具有相同 shareUID 与签名的 APP 可以跑在这个进程中。

> `AndroidMantifest.xml` 中的 `activity`、`service`、`receiver` 和 `provider` 元素均支持 `android:process` 属性；`application` 元素也支持 `android:process` 属性，可以修改应用程序的默认进程名(默认值为包名)。

## Android IPC 基础概念 ##
### Serializable 接口 ###
**`Serializable`**是 Java 提供的一个序列化接口，它是一个空接口，为对象标准的序列化和反序列化操作。使用 `Serializable` 来实现序列化相当简单，一句话即可。用法如下：
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
**`Parcelable`**是 Android 提供的序列化方式，`Parcel` 内部包装了可序列化的数据，可以在 `Binder` 中自由传输，在序列化过程中需要实现的功能有序列化、反序列化和内容描述序列化功能有 `writeToParcel()` 方法来完成，最终是通过 `Parcel` 中的一系列 `writeXXX()` 方法来完成的。用法如下：
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

> `Serializable` 是 Java 中的序列化接口，其使用起来简单但是开销很大，在序列化和反序列化过程中需要大量的 I/O 操作。而 `Parcelable` 是 Android 中的序列化方式，因此更适合用在 Android 平台上，它的缺点就是使用起来稍微麻烦点，但是它的效率很高。

### Binder ###
直观来说，`android.os.Binder` 是 Android 中的一个类，它实现了 `android.os.IBinder` 接口。从 IPC 角度来说，`Binder` 是 Android 中的一种跨进程通信方式，`Binder` 还可以理解为一种虚拟的物理设备，它的设备驱动是 `/dev/binder`，该通信方式在 `Linux` 中没有。从 Android Framework 角度来说，`Binder` 是 `ServiceManager` 连接各种 `Manager`(ActivityManager、WindowManager等等)和相应 `ManagerService` 的桥梁。从 Android 应用层来说，`Binder` 是客户端和服务端进行通信的媒介，当 `bindService` 的时候，服务端会返回一个包含了服务端业务调用的 `Binder` 对象，通过 `Binder` 对象，客户端就可以获取服务端提供的服务或者数据，这里的服务包括普通服务和基于 `AIDL` 的服务。

## Android 中的 IPC 方式 ##
Android 实现跨进程通信的方式有很多，比如通过 `Intent` 来传递 `Bundle` 数据、`共享文件`、基于 `Binder` 的 `Messenger` 和 `AIDL` 以及 `Socket` 等。

### 通过 Bundle 传递数据 ###
**`通过 Bundle 传递数据`**是最常用的一种进程间通信方式，Android 四大组件中三大组件(Activity、Service、Receiver)都是支持在 `Intent` 中传递 `Bundle` 数据的，由于 `Bundle` 实现了 `Parcelable` 接口，所以它可以方便地在不同的进程间传输。

`Bundle` 支持传递的数据类型包括`基本数据类型`、`String`、`CharSequence` 以及实现了 `Serializable` 或 `Parcellable` 接口的数据结构。

> Serializable 是 Java 的序列化方法，代码量少(仅一句)，但I/O开销较大，一般用于输出到磁盘或网卡；Parcellable 是 Android 的序列化方法，实现代码多，效率高，一般用户内存间序列化和反序列化传输。

### 文件共享 ###
**`共享文件`**也是一种常用的的进程间通信方式，两个进程间通过读/写同一个文件来交换数据，比如一个进程把数据写入文件，另一个进程通过读取这个文件来获取数据。

> Linux 机制下，可以对文件并发写，所以要注意同步；Windows 下不支持并发读或写。

### Messenger ###
**`Messenger`**可以翻译为信使，顾名思义，通过它可以在不同进程中传递 `Message` 对象，在 `Message` 中放入我们需要传递的数据，就可以轻松地实现数据的进程间传递。`Messenger` 是一种轻量级的 IPC 方案，它是基于 `AIDL` 实现的。

实现 `Messenger` 有以下两个步骤，分为服务端进程和客户端进程。服务端(被动方)提供一个 Service 来处理客户端(主动方)连接，维护一个 `Handler` 来创建 `Messenger`，在 `onBind` 时返回 `Messenger` 的 `Binder`。双方用 `Messenger` 来发送数据，用 `Handler` 来处理数据。`Messenger` 处理数据依靠 `Handler`，所以是串行的，也就是说，`Handler` 接到多个 `Message` 时，就要排队依次处理。

### AIDL ###
**`AIDL`**是远程服务跨进程通信的一种方式，它通过定义服务端暴露的接口，以提供给客户端来调用，`AIDL` 使服务器可以并行处理，而 `Messenger` 封装了 `AIDL` 之后只能串行运行，所以 `Messenger` 一般用作消息传递。

通过编写 `aidl` 文件来设计想要暴露的接口，编译后会自动生成响应的 Java 文件，服务器将接口的具体实现写在 `Stub` 中，用 `IBinder` 对象传递给客户端，客户端 `bindService` 的时候，用 `asInterface` 的形式将 `IBinder` 还原成接口，再调用其中的方法。

### ContentProvider ###
**`ContentProvider`**是 Android 中提供的专门用于不同应用间进行数据共享的方式，它的底层实现同样也是 `Binder`，主要用来为其他应用提供数据，可以说天生就是为进程通信而生的。

> 实现一个 `ContentProvider` 需要实现6个方法，其中 `onCreate()` 方法是主线程中回调的，其他方法是运行在 `Binder` 之中的。自定义的 `ContentProvider` 注册时要提供 `authorities` 属性，应用需要访问的时候将属性包装成 `Uri.parse("content://authorities")`。还可以设置 `permission`、`readPermission`、`writePermission` 来设置权限。 `ContentProvider` 有 `query()`、`delete()`、`insert()`、`update()` 等方法，看起来像是是一个数据库管理类，但其实可以用文件、内存数据等等一切来充当数据源，`query()` 方法返回的是一个 `Cursor` 对象，可以自定义继承 `AbstractCursor` 的类来实现。

### Socket ###
**`Socket`**也称为`套接字`，是网络通信中的概念，它分为流式套接字和用户数据套接字两种，分别应于网络的传输控制层中的 `TCP` 和 `UDP` 协议。

> 需要注意的是：`Android 不允许在主线程中请求网络`，而且请求网络必须要注意声明相应的 `permission`。然后，在服务器中定义 `ServerSocket` 来监听端口，客户端使用 `Socket` 来请求端口，连通后就可以进行通信。

## IPC 方式的优缺点和适用场景 ##

| 名称            | 优点                                                                         | 缺点                                                                                              | 适用场景                                                         |
|-----------------|------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|------------------------------------------------------------------|
| Bundle          | 简单易用                                                                     | 只能传输 Bundle 支持的数据类型                                                                    | 四大组件间的进程通信                                             |
| 文件共享        | 简单易用                                                                     | 不适合高并发的情况，并且无法做到进程间的即时通讯                                                  | 无并发访问情况下，交换简单的数据实时性不高的情况                 |
| AIDL            | 功能强大，支持一对多并发通信，支持实时通讯                                   | 需要处理好线程同步                                                                                | 一对多通信且有 RPC 需求                                          |
| Messager        | 支持一对多串行通信，支持实时通讯                                             | 不能很好处理高并发情况，不支持 RPC，数据通过 Message 进行传输，因此只能传输 Bundle 支持的数据类型 | 低并发的一对多即时通信，无 RPC 需求，或者无需返回结果的 RPC 需求 |
| ContentProvider | 在数据源访问方面功能强大，支持一对多并发数据共享，可通过call方法扩展其他操作 | 主要提供数据源的 Crud                                                                             | 一对多的进程间数据共享                                           |
| Socket          | 功能强大，可以通过网络传输字节流，支持一对多并发实时通讯                     | 实现细节有点繁琐，不支持直接的 RPC                                                                | 网络数据交换                                                     |

