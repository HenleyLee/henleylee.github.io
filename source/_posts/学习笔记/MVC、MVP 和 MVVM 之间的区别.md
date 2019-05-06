---
title: MVC、MVP 和 MVVM 之间的区别
categories: 学习笔记
tags:
  - 学习笔记
abbrlink: de9d67a4
date: 2019-04-03 10:26:35
---

`MVC`、`MVP` 和 `MVVM` 都是常见的软件架构设计模式(Architectural Pattern)，它通过分离关注点来改进代码的组织方式。不同于设计模式(Design Pattern)，只是为了解决一类问题而总结出的抽象方法，一种架构模式往往使用了多种设计模式。

要了解 `MVC`、`MVP` 和 `MVVM`，就要知道它们的相同点和不同点。不同部分是 `C(Controller)`、`P(Presenter)`、`VM(View-Model)`，而相同的部分则是 `MV(Model-View)`。

## MVC ##
**`MVC`**(Model-View-Controller 的缩写)模式是一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。

![MVC模式](https://henleylee.github.io/medias/study/architectural_pattern_mvc.png)

 - **`Model(模型)：`**负责处理数据、状态和业务逻辑；
 - **`View(视图)：`**负责界面的布局和显示以及与用户的交互；
 - **`Controller(控制器)：`**作为 View 与 Model 交互的桥梁，用于控制应用程序的流程和页面的业务逻辑，以此来达到分离视图显示和业务逻辑层。

### 特点 ###
`MVC` 模式的特点在于实现关注点分离，即应用程序中的数据模型与业务和展示逻辑解耦。在客户端 Web 开发中，就是将模型(M-数据、操作数据)、视图(V-显示数据的 HTML 元素)之间实现代码分离，松散耦合，使之成为一个更容易开发、维护和测试的客户端应用程序。
1. View 传送指令到 Controller；
2. Controller 完成业务逻辑后，要求 Model 改变状态；
3. Model 将新的数据发送到 View，用户得到反馈。

### 流程 ###
`MVC` 模式的流程一共有两种，在日常开发中都会使用到：
 - 一种是通过 View 接受指令，传递给 Controller，然后对模型进行修改或者查找底层数据，最后把改动渲染在视图上。 
 - 另一种是通过 Controller 接受指令，然后对模型进行修改或者查找底层数据，最后把改动渲染在视图上。 

> 不管哪种模式，MVC 的通信都是单向的。View 层会从 Model 层拿数据，因此 MVC 中的 View 层和 Model 层还是存在耦合的。

### 优点 ###
`MVC` 模式具有以下优点：
 - 重用性高，生命周期成本低，部署快；
 - 使开发和维护用户接口的技术含量降低；
 - 耦合性低，视图层和业务层分离，允许更改视图层代码而不用重新编译模型和控制器代码；
 - 可维护性高，分离视图层和业务逻辑层也使得 Web 应用更易于维护和修改。

### 缺点 ###
`MVC` 模式具有以下缺点：
 - 不适合小型，中等规模的应用程序，花费大量时间将 MVC 应用到规模并不是很大的应用程序通常会得不偿失；
 - 视图与控制器间过于紧密连接，视图与控制器是相互分离，但却是联系紧密的部件，视图没有控制器的存在，其应用是很有限的，反之亦然，这样就妨碍了他们的独立重用；
 - 视图对模型数据的低效率访问，依据模型操作接口的不同，视图可能需要多次调用才能获得足够的显示数据。对未变化数据的不必要的频繁访问，也将损害操作性能；

## MVP ##
**`MVP`**(Model-View-Presenter 的缩写)模式是 MVC 的改良模式，由 IBM 的子公司 Taligent 提出。和 MVC 的相同之处在于：Controller/Presenter 负责业务逻辑，Model 管理数据，View 负责显示，只不过是将 Controller 改为了 Presenter，View 通过接口与 Presenter 进行交互，降低耦合，方便进行单元测试，同时改变了通信方向。

![MVP模式](https://henleylee.github.io/medias/study/architectural_pattern_mvp.png)

 - **`View(视图)：`**负责绘制 UI 元素、与用户进行交互(在 Android 中 Activity、Fragment、View都可以做为 View 层)
 - **`Model(模型)：`**负责处理数据、状态和业务逻辑(主要职责是存储、检索、操纵数据，也可以实现一个 Model interface 用来降低耦合)；
 - **`Presenter(控制器)：`**作为 View 与 Model 交互的中间纽带，负责完成 View 与 Model 间的交互。可以把 Presenter 理解为一个中间层的角色，它接受 Model 层的数据，并且处理之后传递给 View 层，还需要处理 View 层的用户交互等操作。

> 开发中往往把 Android 中界面部分的实现也理解为采用了 MVC 框架，常常把 Activity 理解为 MVC 模式中的 Controller。

### 特点 ###
`MVP` 模式的特点在于 Presenter 完全把 Model 和 View 进行了分离，并且 Model、View 和 Presenter 之间双向通信。
 - View 与 Model 不通信，都通过 Presenter 传递。Presenter 完全把 Model 和 View 进行了分离，主要的程序逻辑在 Presenter 里实现。
 - View 非常薄，不部署任何业务逻辑，称为“被动视图(Passive View)”，即没有任何主动性，而 Presenter 非常厚，所有逻辑都部署在那里。
 - Presenter 与具体的 View 是没有直接关联的，而是通过定义好的接口进行交互，从而使得在变更 View 时候可以保持 Presenter 的不变，这样就可以重用。
 - 还可以编写测试用的 View，模拟用户的各种操作，从而实现对 Presenter 的测试而不需要使用自动化的测试工具。 

> 在 MVP 中，View 并不直接使用 Model，它们之间的通信是通过 Presenter 来进行的，所有的交互都发生在 Presenter 内部。在 MVC 中，View 会直接从 Model 中读取数据而不是通过 Controller。

### 优点 ###
`MVP` 模式具有以下优点：
 - 模型与视图完全分离，可以修改视图而不影响模型；
 - 可以更高效地使用模型，因为所有的交互都发生在一个地方 —— Presenter 内部；
 - 可以将一个 Presenter 用于多个视图，而不需要改变 Presenter 的逻辑。这个特性非常的有用，因为视图的变化总是比模型的变化频繁；
 - 如果把逻辑放在 Presenter 中，那么就可以脱离用户接口来测试这些逻辑(单元测试)。

### 缺点 ###
`MVP` 模式具有以下缺点：
 - View 和 Presenter 的交互会过于频繁，使得他们的联系过于紧密。也就是说，一旦视图变更了，Presenter 也要变更。

## MVVM ##
**`MVVM`**(Model-View-ViewModel 的缩写)模式和 MVP 模式相比，MVVM 模式用 ViewModel 替换了 Presenter ，其他层基本上与 MVP 模式一致，ViewModel 可以理解成是 View 的数据模型和 Presenter 的合体。

![MVVM模式](https://henleylee.github.io/medias/study/architectural_pattern_mvvm.png)

 - **`View(视图)：`**负责绘制 UI 元素、与用户进行交互(在 Android 中 Activity、Fragment、View都可以做为 View 层)
 - **`Model(模型)：`**负责处理数据、状态和业务逻辑(主要职责是存储、检索、操纵数据)；
 - **`ViewModel(视图模型)：`**主要包括界面逻辑和模型数据封装，Behavior/Command 事件响应，绑定的属性定义等。

### 特点 ###
`MVVM` 模式采用双向绑定(data-binding)：View 的变动，自动反映在 ViewModel，反之亦然。这种模式实际上是框架替应用开发者做了一些工作(相当于 ViewModel 类是由库帮开发者生成的)，开发者只需要较少的代码就能实现比较复杂的交互。

### 优点 ###
`MVVM` 模式具有以下优点：
 - 双向绑定技术，当 Model 变化时，ViewModel 会自动更新，View 也会自动变化，很好的做到数据的一致性；
 - 低耦合，视图(View)可以独立于 Model 变化和修改，一个 ViewModel 可以绑定到不同的 View 上，当 View 变化的时候 Model 可以不变，当 Model 变化的时候 View 也可以不变；
 - 独立开发，开发人员可以专注于业务逻辑和数据的开发(ViewModel)，设计人员可以专注于页面设计，使用 Expression Blend 可以很容易设计界面并生成 xml 代码；
 - 可测试，界面向来是比较难于测试的，而现在测试可以针对 ViewModel 来写。

### 缺点 ###
`MVVM` 模式具有以下缺点：
 - 数据绑定也使得 Bug 很难被调试，数据绑定使得一个位置的 Bug 被快速传递到别的位置，要定位原始出问题的地方就变得不那么容易了；
 - 数据双向绑定不利于代码重用。客户端开发最常用的是 View，但是数据双向绑定技术，让每一个 View 都绑定了一个 Model，不同的模块 Model 都不同；
 - 一个大的模块中 Model 也会很大，虽然使用方便了也很容易保证数据的一致性，但是长期持有，不释放内存就造成话费更多的内存。

