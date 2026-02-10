---
title: "一文搞懂Kotlin跨平台概念"
date: 2025-03-31
draft: false
tags: ["KMP", "安卓"]
categories: ["技术"]
---

> Kotlin跨平台技术已经较为成熟。随着其发展陆续涌现了MPP、Jetpack Compose、KMP、Compose for Desktop、KMM、KMD、CMP等相关术语，给开发者带来了困惑。本文总结了Kotlin跨平台历程，阐述了各术语历史渊源、概念原理、发展历史、代码示例、相互关联。



[[toc]]

## 一、背景

随着现代软件开发的不断进步，开发人员需要在各种平台上创建一致且高性能的用户体验。为了实现这一目标，谷歌、JetBrains推出了Kotlin跨平台相关技术和工具。这些技术不仅简化了开发流程，还提高了代码的可维护性和可复用性。

不过随着Kotlin跨平台相关技术的发展，涌现了包括Jetpack **Compose**、Compose for Desktop、Multiplatform Projects(**MPP**)、Kotlin Multiplatform(**KMP**)、Kotlin Multiplatform Mobile(**KMM**)、Kotlin Multiplatform Desktop(**KMD**)、Compose Multiplatform(**CMP**)等术语，会给开发人员带来困扰。

<div align="center">
<img src="/images/7/83cb595f-a1c7-4727-8bf5-1f1d31d7ed99.png" width=50% />
</div>

看到这么多术语，可能读者已经晕了。不过没关系，本文接下来将介绍这些术语的概念原理、发展历史、代码示例及其相互间的关联。

下面是笔者疏离出来的各术语关键时间线，读者们可以先浏览一下，对**Kotlin语言跨平台技术**、**Compose声明式UI框架**的发展历程有个大致的了解。

| 时期       | 厂商             | 内容                                                         | 资料                                                         |
| ---------- | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 2017年05月 | Google           | 宣布安卓添加对Kotlin的支持。                                 | https://android-developers.googleblog.com/2017/05/android-announces-support-for-kotlin.html |
| 2017年09月 | JetBrains        | 发布Kotlin 1.2，宣布其试验性质的**Multiplatform Projects(MPP)**跨平台特性。 | https://kotlinlang.org/docs/whatsnew12.html https://blog.jetbrains.com/kotlin/2017/11/kotlinconf-keynote-recap/ |
| 2018年05月 | Google           | 推出Android **Jetpack**。提高安卓开发效率。                  | https://android-developers.googleblog.com/2018/05/google-io-2018-whats-new-in-android.html |
| 2019年05月 | Google           | 推出 **Jetpack Compose**  (for Android)。后续安卓开发可以使用Compose，类似Swift UI那样声明式编程来开发安卓应用。 | https://android-developers.googleblog.com/2019/05/whats-new-with-android-jetpack.html |
| 2020年08月 | JetBrains        | 推出Kotlin1.4，宣布了alpha版本的**Kotlin Multiplatform(KMP)**。后续Kotlin语言有了逻辑层多平台复用的可能。 | https://kotlinlang.org/docs/whatsnew14.html                  |
| 2020年08月 | JetBrains        | 推出**Kotlin Multiplatform Mobile(KMM)**的alpha版本，但是只提供了Android Studio插件、未提供IDEA插件 。后续Kotlin语言能支持安卓、iOS的跨平台复用。 | https://blog.jetbrains.com/kotlin/2020/08/kotlin-multiplatform-mobile-goes-alpha/ |
| 2020年11月 | JetBrains        | **Compose for Desktop** 发布第一个里程碑版本。Compose UI从此支持 MacOS、Windows、Linux 等桌面端的开发。 | https://blog.jetbrains.com/cross-post/jetpack-compose-for-desktop-milestone-1-released （失效了） |
| 2021年05月 | JetBrains        | 发布 **Compose for Web**技术预览版，Compose 可基于 Kotlin/JS 实现前端 UI 的开发。 | https://blog.jetbrains.com/kotlin/2021/05/technology-preview-jetpack-compose-for-web/ |
| 2021年12月 | jetbrains+Google | **Compose Multiplatform(CMP) 1.0** 即将上线。后续Kotlin语言有了UI层多平台复用的可能。 | https://blog.jetbrains.com/zh-hans/kotlin/2021/12/compose-multiplatform-1-0-is-going-live/ |
| 2023年8月  | JetBrains        | 宣布**弃用MPP、KMM名称**，后续**统一用KMP**术语。            | https://blog.jetbrains.com/zh-hans/kotlin/2023/08/update-on-the-name-of-kotlin-multiplatform/ |
| 2023年11月 | JetBrains        | 宣布**KMP已经稳定**（stable version）、可以投入在生产环境。  | https://blog.jetbrains.com/zh-hans/kotlin/2023/11/kotlin-multiplatform-stable/ |
| 2024年05月 | Google           | 宣布**将在 Android 上支持 KMP**。                                | https://android-developers.googleblog.com/2024/05/android-support-for-kotlin-multiplatform-to-share-business-logic-across-mobile-web-server-desktop.html |
| 2024年11月 | JetBrains        | 对于iOS：预计2025年**CMP for iOS**升级到稳定版、发布 Kotlin-to-Swift 导出和一体化 KMP IDE(2024年06月CMP对iOS的支持进入Beta极端)。 | https://blog.jetbrains.com/zh-hans/kotlin/2024/11/kotlin-multiplatform-development-roadmap-for-2025/ |

## 二、Kotlin简介

[Kotlin](https://kotlinlang.org/)是一种现代的、静态类型的编程语言，由JetBrains 2011年推出。Kotlin在2016年推出官方稳定版本Kotlin v1.0，在2017年得到了 Google 官方支持，并用于开发 Android 应用程序。

Kotlin设计时就旨在解决Java编程语言的一些不足，同时提供更高效、更简洁的语法。Kotlin兼容Java，并且在运行时可以与Java代码无缝互操作。这提高了安卓开发的效率。因此，很多**安卓开发团队的新代码基本都是用Kotlin写的**。




<div align="center">
<img src="/images/7/cd5ee5d6-ff43-415e-ab6d-a2652907bc19.png" width=50% />
</div>

Kotlin除了可以被编译成Java字节码外，也是一种强大的跨平台编程语言：

* 它可以编译成JavaScript，方便在没有JVM的设备上运行。
* 它可以编译成二进制代码直接运行在机器上（例如嵌入式设备或 iOS）。

下面我们将介绍其UI、逻辑层的跨平台性的发展历程。

## 三、Jetpack Compose

### 1 开始之前-Android Jetpack

时间回到2018年，谷歌在开发者大会上宣布了Android Jetpack。[Android Jetpack](https://developer.android.com/jetpack)是一个由多个库组成的套件，可帮助开发者遵循最佳做法、减少样板代码并编写可在各种 Android 版本和设备中一致运行的代码，让开发者可将精力集中于真正重要的编码工作。

也许读者对这个概念本身没那么明确，但实际上安卓开发基本都用过其中的诸如ViewModel、LiveData、Lifecycles等组件。如下图所示，如果读者做过安卓开发，相信能看到不少老熟人：

![图片](/images/7/acdb442d-7bac-4eb9-826a-c48b49fe2176.png)


注：图中的`new`是2018年推出Android Jetpack时间节点，详见 [Use Android Jetpack to Accelerate Your App Development](http://android-developers.googleblog.com/2018/05/use-android-jetpack-to-accelerate-your.html)

Android Jetpack能够大大地提高我们的开发效率(尽管可能读者未发觉到)。以腾讯频道一些页面的首帧上报为例，如果只是使用kotlin，我们可能会这么写：

```kotlin
view.viewTreeObserver.addOnPreDrawListener(
  object : ViewTreeObserver.OnPreDrawListener {
    override fun onPreDraw(): Boolean {
      viewTreeObserver.removeOnPreDrawListener(this)
      reportActionToBeTriggered() /** 触发上报 */
      return true
    }
});
```
在使用**Android KTX** 中更简洁的`kotlin`代码后，就成了：

```kotlin
view.doOnPreDraw { reportActionToBeTriggered() /** 触发上报 */ }
```
### 2 Jetpack Compose 简介

在推出Android Jetpack一年后，谷歌在2019年开发者大会上宣布了Jetpack Compose的早期预览版。它基于Kotlin编程语言，旨在简化和加速UI开发流程，同时提供更灵活和响应式的用户界面。

Jetpack Compose的核心思想是通过函数式编程来简化UI开发，使开发者能够更轻松地创建复杂而动态的用户界面。Jetpack Compose基于Compose DSL（[Domain-specific language](https://en.wikipedia.org/wiki/Domain-specific_language) 领域特定语言），这使得UI组件可以通过简单、直观的代码进行定义。主要特点包括：

1. **声明式编程**：开发者可以定义UI的状态，而Compose根据状态自动渲染UI。
2. **重组机制**：Compose可以高效地更新UI而不需要重新创建整个界面。
3. **可组合性**：UI组件可以嵌套并组合，形成复杂的界面。

### 3 Jetpack Compose 代码示例

这是Jetpack Compose声明式UI开发的一个简单示例：

![图片](/images/7/ae8eaa99-c8a9-4bc6-8f59-4e5601dda9ef.png)


如果读者进行过iOS Swift UI开发，会发现其**代码跟Swift UI开发很相似**（将Column简单理解为VStack）：
![图片](/images/7/9b87bd2c-882e-463e-bc3a-60f0739313cc.png)


### 4 Jetpack Compose 打包

Jetpack Compose的打包过程与传统Android应用相似，使用Gradle进行构建。Compose代码会被编译成字节码并打包成APK或AAB文件。Compose依赖的库和资源会被包含在这些文件中，以供应用在设备上运行时使用。

### 5 Jetpack Compose 绘制

Jetpack Compose使用的是Android的Skia图形引擎进行渲染。每个Compose组件会生成相应的UI树，这棵树会被转换成Skia的绘制指令。当状态发生变化时，Compose会重新计算需要更新的UI树部分，并仅重新渲染这些部分，以提高性能和响应速度。

如下图所示，Jetpack Compose的绘制过程主要有三个阶段：Composition组合、layout布局、Drawing绘制。


![图片](/images/7/c64b5aec-b288-45de-8f1d-73fc31f0bf8f.png)


#### 5.1 Composition组合

在组合阶段，Compose 运行时会执行可组合函数（@Composable注解），并输出表示界面的树结构。

`@Composable` 函数或 lambda 代码块中的状态读取会影响组合阶段，并且可能会影响后续阶段。当状态值发生更改时，Recomposer 会安排重新运行所有要读取相应状态值的可组合函数。

```kotlin
var padding by remember { mutableStateOf(8.dp) }
Text(
    text = "Hello",
    //在组合阶段构建修饰符时，会读取`padding`状态。`padding`的变化将触发重组
    modifier = Modifier.padding(padding)
)
```

#### 5.2 layout布局

在布局阶段，Compose 会使用组合阶段生成的界面树作为输入，对每个节点进行**测量**和**放置**。

在布局阶段，系统会使用以下三步算法遍历树：

* **测量子项**：节点会测量其子项（如果有）。

* **确定自己的尺寸**：节点根据这些测量结果确定自己的尺寸。

* **放置子项**：每个子节点都相对于节点自身的位置进行放置。

在此阶段结束时，每个布局节点都具有：

- 分配的**宽度**和**高度**
- 应绘制该图形的 **x、y 坐标**

```kotlin
var offsetX by remember { mutableStateOf(8.dp) }
Text(
    text = "Hello",
    modifier = Modifier.offset {
        // 在布局阶段的放置步骤中，计算偏移量时会读取`offsetX`状态。 `offsetX`的变化会重新启动布局。
        IntOffset(offsetX.roundToPx(), 0)
    }
)
```

#### 5.3 Drawing绘制

在绘制阶段，系统会再次从上到下遍历树，每个节点都会依次在屏幕上绘制自身。

```kotlin
var color by remember { mutableStateOf(Color.Red) }
Canvas(modifier = modifier) {
    // 在绘制阶段渲染画布时会读取`color`状态。`color`的变化会重新开始绘制。
    drawRect(color)
}
```

总之，Jetpack Compose的绘制过程大概如下所示，不同的代码会触发不同阶段的重新执行（其中还涉及到一些性能优化，这里不再赘述）。

![图片](/images/7/2d37b5a9-b062-4610-95ff-c1783dae4fa9.png)


### 6 Jetpack Compose 小结

Jetpack Compose的开发始于2019年，并且在2020年Google I/O大会上首次公开展示。经过数次预览版和Beta版的发布，Compose在2021年正式发布。Jetpack Compose通过声明式UI编程模型简化了Android应用开发，提供了更灵活和高效的开发体验。其现代化的设计理念和强大的渲染机制，使得开发者可以更专注于业务逻辑和用户体验，而不必纠结于复杂的UI更新问题。

Jetpack Compose的推出标志着Android UI开发进入了一个新的时代，开发者能够更高效地创建现代化的用户界面。

## 四、Jetpack Compose for Desktop

我们知道了谷歌推出的Jetpack Compose是for Android 应用声明式 UI 框架的。JetBrains 看到其中的潜力，并决定将其扩展到桌面应用程序。

2020 年，JetBrains 推出了 Compose for Desktop 的 alpha 版本，并宣布达到[首个里程碑（遗憾的是这篇通告打不开了）](https://blog.jetbrains.com/cross-post/jetpack-compose-for-desktop-milestone-1-released)，之后不断迭代和改进。本篇我们着重介绍一下Compose for Desktop。

### 1 Compose for Desktop介绍

Compose for Desktop是Jetpack Compose的一个扩展，用于构建桌面应用程序。它使开发者能够使用相同的Compose DSL在桌面环境中创建用户界面。Compose for Desktop支持Windows、macOS和Linux平台，提供了与Jetpack Compose类似的声明式编程模型和可组合性。

除了Jetpack Compose本身的声明式编程、重组机制、可组合性等特点外， Compose for Desktop还有以下特性：

* **跨平台一致性**：使用相同的代码库和UI组件，可以在Windows、macOS 和 Linux 桌面操作系统上运行应用程序。
* **原生集成**：Compose for Desktop 能够与现有的 Java 和 Kotlin 框架及库无缝集成。我们可以利用 JVM 生态系统中的各种工具和库，如 JDBC、JavaFX、Swing 等，扩展应用的功能。
* **代码复用**：由于 Compose for Desktop 基于 Jetpack Compose 的设计理念，开发者可以更容易地在不同平台（如 Android 和 Web）之间共享代码。这样可以最大化代码复用，减少开发和维护成本。

到了2021年，Compose for Desktop进入稳定阶段，并开始广泛应用于跨平台桌面应用的开发。

这里介绍一下，[SweetPotato日志工具](https://iwiki.woa.com/p/4009599272)就是2022年初基于Jetpack Compose for Desktop 开发的，支持Mac、Windows双平台。当时的compose版本还是1.0.1-rc2，现在[最新稳定版已经是1.7.0](https://blog.jetbrains.com/kotlin/2024/10/compose-multiplatform-1-7-0-released/)了。


![图片](/images/7/19754cda-17c4-4454-b369-b4a1baa3b782.png)



### 2 Compose for Desktop代码示例

Demo示例代码如下，可以看到还是写`@Composable`。

![图片](/images/7/197c17eb-2eba-46da-bcb8-59d2fab14e75.png)

值得一提的是，我们可以[在Compose for Desktop中嵌入Swing与JavaFX](https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-desktop-swing-interoperability.html#use-swing-in-a-compose-multiplatform-application)，进而兼容更多Java桌面端开发的现有库与能力。比如笔者早期曾[在Compose中内嵌JCEF、JXBrowser等浏览器](https://km.woa.com/articles/show/565507)。

``` kotlin
@Composable
fun WebLinkCenterView() {
  Box(modifier = Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
    SwingPanel(
      factory = {
        // 这里在Composable中内嵌Swing JPanel
        JPanel().apply {
          layout = BoxLayout(this, BoxLayout.Y_AXIS)
          // 这里内嵌JCEF浏览器
          add(webLinkViewerViewModel.mCefBrowser.uiComponent)
//... }})}}
```

### 3 Compose for Desktop绘制

同Jetpack Compose，这里不再赘述。

![图片](/images/7/be7e311e-82c1-4379-b40d-6911bcf8c1f3.png)

### 4 Compose for Desktop原理简介

Compose for Desktop 能够在桌面端运行的原理主要依赖于 Kotlin Multiplatform 项目以及 JetBrains 的 SKIA 图形库。以下是一些关键点（笔者也没详细了解过，这里只是简单介绍一下）：

1. **Kotlin Multiplatform**： Compose for Desktop 利用 Kotlin Multiplatform 的能力，实现了跨平台的代码共享。通过 Kotlin Multiplatform，开发者可以编写通用的业务逻辑代码，同时针对不同平台（如 Android、桌面）编写特定的 UI 代码。
2. **SKIA 图形库**： SKIA 是一个开源的二维图形库，被广泛应用于 Google Chrome、Mozilla Firefox、Android 等项目中。Compose for Desktop 使用 SKIA 进行图形渲染，这使得它能够高效地绘制复杂的 UI 元素，并支持各种图形操作。
3. **JetBrains 的优化**： JetBrains 在 Compose for Desktop 中进行了大量的优化，使其能够在桌面环境中高效运行。这包括对窗口管理、事件处理、输入输出等。

### 5 Compose for Desktop小结

Jetpack Compose与Compose for Desktop两者共享相同的Compose DSL，使得开发者可以使用一致的代码创建移动和桌面应用。
|术语|平台|DSL|功能特性差异/特点|
|:-:|:-:|:-:|:-:|
|Jetpack Compose|Android|Compose|集成安卓系统的相关组件、库|
|Compose for Desktop|Windows, macOS, Linux|Compose|集成了 JDBC、JavaFX、Swing 等JVM下丰富的生态系统资源|

## 五、Kotlin跨平台（MPP、KMP、KMM、KMD）

### 1 MPP (Multiplatform Projects)

#### 1.1 介绍

Kotlin跨平台在[Kotlin1.2版本介绍](https://kotlinlang.org/docs/whatsnew12.html#multiplatform-projects-experimental)、 [KotlinConf 2017](https://blog.jetbrains.com/kotlin/2017/11/kotlinconf-keynote-recap/) 上以“Multiplatform Projects”名称发布，当时支持 JVM  JS、Native 目标。

> **Multiplatform projects** are a new experimental feature in Kotlin 1.2, allowing you to reuse code between target platforms supported by Kotlin – JVM, JavaScript, and (in the future) Native。

当时，有些人称之为“MPP”，但这么称呼的人不多。如果读者在Kotlin跨平台上下文中看到“MPP”字样，知道其是“Multiplatform Projects”的缩写即可。

#### 1.2 代码示例

这里的核心是两个声明关键词**expected**、**actual**。

* expected在common模块中，用于指定一个API。common模块不包含特定平台相关的代码、不依赖于特定平台的 API 声明。API声明比如：

```kotlin
expect fun hello(world: String): String

expect class URL(spec: String) {
    open fun getHost(): String
    open fun getPath(): String
}
```

* actual在特定platform模块中，实现对应API。platform模块主要是特定平台相关的代码，以及实现common模块中的API声明。actual实现比如：

```kotlin
actual fun hello(world: String): String =
    "Hello, $world, on the JVM platform!"

// 直接使用特定平台下已有的实现。java.net.URL是JVM平台下特定的代码，在js、native中没有这个java类。
actual typealias URL = java.net.URL
```

### 2 KMP (Kotlin Multiplatform)

#### 2.1 概念原理

JetBrains应该是[在发布kotlin1.4.0时直接使用Kotlin Multiplatform(简称KMP)](https://kotlinlang.org/docs/whatsnew14.html#kotlin-multiplatform)，而不再直接使用Multiplatform Projects这一MPP术语的。

KMP是JetBrains推出的一项技术，旨在使开发者能够用Kotlin编写共享代码，并在不同的平台（如Android、iOS、Web、桌面）上运行。

KMP允许开发者将业务逻辑、数据模型和其他非UI代码集中在一个共享模块（Common Module）中，而平台特定的代码（**Platform-specific Modules**）则可以在各自的模块中编写。KMP通过模块化的方式来组织代码，其中包含以下模块：

* **Common Module**：包含业务逻辑和数据模型（在所有平台上共享的通用代码），可以被多个平台使用。
* **Platform-specific Modules**：包含与平台相关的代码，比如UI组件或平台特定的API调用。这些模块可以依赖Common Module，并实现特定平台的功能。

KMP使用Kotlin的编译器插件来实现跨平台的编译。对于每个目标平台，Kotlin编译器会生成相应的二进制文件，这些文件可以直接运行在目标平台上。

![图片](/images/7/dfb46cf3-f03c-4a6f-8833-52aa78bf08f3.png)

#### 2.2 代码示例

读者可以参考[Create your Kotlin Multiplatform app](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-create-first-app.html)这里的tutorial创建一个demo项目。以下是一个简单的Kotlin Multiplatform项目示例：

##### 2.2.1 Common模块逻辑层代码

在comminMain模块中，定义以下expect API。外部UI层则调用 `Greeting().greet()`获取最终的字符串。

```kotlin
// commonMain/kotlin/.../Platform.kt
interface Platform {
    val name: String
}
expect fun getPlatform(): Platform

// commonMain/kotlin/.../Greeting.kt
class Greeting {
    private val platform = getPlatform()

    fun greet(): String {
        return "Hello, ${platform.name}!"
    }
}
```

##### 2.2.1 各平台模块逻辑层代码

* 安卓项目

可以看到`androidMain`，这里使用安卓平台特有的`android.os.Build.VERSION.SDK_INT`，来获取安卓平台系统信息。

```kotlin
import android.os.Build

class AndroidPlatform : Platform {
    override val name: String = "Android ${Build.VERSION.SDK_INT}"
}

actual fun getPlatform(): Platform = AndroidPlatform()
```

* iOS项目

可以看到`iosMain`这里，使用iOS平台特有的`platform.UIKit.UIDevice`，来获取iOS平台系统名称、版本。

```kotlin
import platform.UIKit.UIDevice

class IOSPlatform: Platform {
    override val name: String = UIDevice.currentDevice.systemName() + " " + UIDevice.currentDevice.systemVersion
}

actual fun getPlatform(): Platform = IOSPlatform()
```

* 桌面端项目

`jvmMain`这里，直接使用参数`java.version`获取Java版本。

```kotlin
class JVMPlatform: Platform {
    override val name: String = "Java ${System.getProperty("java.version")}"
}

actual fun getPlatform(): Platform = JVMPlatform()
```

* Web端项目

`wasmJsMain` 这里，直接写死了字符串`Web with Kotlin/Wasm`。

```kotlin
class WasmPlatform: Platform {
    override val name: String = "Web with Kotlin/Wasm"
}

actual fun getPlatform(): Platform = WasmPlatform()
```

##### 2.2.3 最终效果

在这个示例中，我们定义了一个通用模块 `commonMain`，其中包含一个期望函数 `platformName()` 和一个使用这个函数的 `greet()` 函数。在具体平台模块 `androidMain` , `iosMain`, `jvmMain`, `wasmJsMain` 中，我们分别实现了 `platformName()` 函数，从而提供平台特定的行为。

这样，在安卓、iOS、桌面端、Web端的UI层分别去调用commonMain中的 `greet()`方法时，会得到不同的字符串。
![图片](/images/7/8c02d620-ff43-4ce7-b70f-979723a53595.png)


#### 2.3 打包原理

这里只是泛泛地介绍一下基本概念，后续笔者再细细探究。Kotlin Multiplatform项目的打包过程涉及以下几个步骤：

1. **编译通用模块**：编译 `commonMain` 中的代码，生成通用库。
2. **编译平台特定模块**：编译每个平台特定的代码，生成相应平台的二进制文件或库。
3. **链接和打包**：根据目标平台的要求，将编译生成的二进制文件或库进行链接和打包，生成最终的应用程序包。

对于Android平台，最终会生成一个APK文件。对于iOS平台，则会生成一个IPA文件。对于JavaScript平台，会生成一个JavaScript包，可以直接在浏览器或Node.js环境中运行。

### 3 KMM (Kotlin Multiplatform Mobile)

#### 3.1 介绍

时间回到2020年08月，当时JetBrains 在推出Kotlin 1.4前后，宣布[推出 Kotlin Multiplatform Mobile(KMM) Alpha版本](https://blog.jetbrains.com/kotlin/2020/08/kotlin-multiplatform-mobile-goes-alpha/)。JetBrains在[2020年度报告](https://www.jetbrains.com/zh-cn/lp/annualreport-2020/)中专门回顾了这一点。

KMM是Kotlin跨平台的一部分，专注于移动应用开发。它使开发者能够在Android和iOS之间共享代码，从而减少重复劳动，提高开发效率。**KMM支持直接调用平台特定的API，并且与Jetpack Compose和SwiftUI等现代UI框架兼容**。

![图片](/images/7/90044620-0c1e-4f1a-aa4c-3ddd03a0fe45.png)


2020年08月31日，JetBrains准们出了一款IDE插件：**[Kotlin Multiplatform Mobile plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)**。借助这个插件，开发者可以在Android Studio IDE 中创建KMM项目，并编写、运行、测试和调试共享代码。注意插件最开始的名字是： *Kotlin Multiplatform Mobile* ，现在的名字是： *Kotlin Multiplatform*。

| Release info                     | Release highlights                                           | Compatible Kotlin version                                    |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 0.8.4Released: 06 December, 2024 | Support for Kotlin’s [K2 mode](https://kotlinlang.org/docs/k2-compiler-migration-guide.html#support-in-ides) for improved stability and code analysis. | [Any of Kotlin plugin versions](https://kotlinlang.org/docs/releases.html#release-details) |
| ...                              | ...                                                          | ...                                                          |
| 0.1.0Released: August 31, 2020   | The first version of the Kotlin Multiplatform Mobile plugin. Learn more in the [blog post](https://blog.jetbrains.com/kotlin/2020/08/kotlin-multiplatform-mobile-goes-alpha/). | [Kotlin 1.4.0](https://kotlinlang.org/docs/releases.html#release-details)[Kotlin 1.4.10](https://kotlinlang.org/docs/releases.html#release-details) |

#### 3.2 代码示例

由于KMM这一概念已被弃用，这里不再直接介绍其代码示例。笔者只需要知道其大致与前面的KMP代码类似就行，只是需要去掉web端、桌面端，仅保留安卓端、iOS端（下图是大致示意）。

![图片](/images/7/de49ff71-b8c9-4356-8978-4ff79354da73.png)


#### 3.3 KMM VS KMP

其实在JetBrains 2020年推出KMM之后，在业界内时引起很多概念混淆的。比如[《KMM vs. KMP — What’s the Deal?》](https://medium.com/vmware-end-user-computing/kmm-vs-kmp-whats-the-deal-dd64bc5d61f2)这篇文章尝试讨论了KMM、KMP之间的关联与区别。

后来在2024年8月14日，Kotlin官方宣布弃用了KMM的称呼。

> 原文：[update-on-the-name-of-kotlin-multiplatform](https://blog.jetbrains.com/zh-hans/kotlin/2023/08/update-on-the-name-of-kotlin-multiplatform/)
>
> 为了解决过去两年长期困扰众多 Kotlin 开发者的命名不一致和缩写混乱问题，**我们将弃用“Kotlin Multiplatform Mobile”(KMM) 产品名称。 从现在开始，无论目标组合如何，“Kotlin Multiplatform”(KMP) 都是跨平台共享代码的 Kotlin 技术的首选术语**。

<div align="center">
<img src="/images/7/bff242b9-0f69-465c-9e46-8c50e48b5624.png" width=50% />
</div>



关于KMM、KMP这两者，值得读者了解的大概有：

* KMM 提供了与 iOS 开发过程的紧密集成。但 [Kotlin Multiplatform Mobile plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform) 插件只能在Android Studio上使用，在IntelliJ Idea上是用不了的。
* KMM插件已经改名为 [Kotlin Multiplatform](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)。但仍然只能在Android Studio上使用。如果大家有调试iOS设备的需求，最好使用Android Studio这个IDE。

![图片](/images/7/07255727-d15e-4df7-93a9-e751b34d2f5c.png)

* 如果大家要使用Kotlin跨平台开发，但不太涉及iOS端，那么使用IntelliJ Idea即可。
* 需要新建Kotlin跨平台时，大家不必非要使用Android Studio上的KMP/KMM插件。可以在[Kotlin Multiplatform Wizard](https://kmp.jetbrains.com/)这里创建。


![图片](/images/7/12af36b5-e75a-4c25-984d-ba7aa0cafc87.png)


### 4 KMD (Kotlin Multiplatform Desktop)

有时有一些KMD（Kotlin Multiplatform Desktop）的说法，比如Jetbrains上的反馈[Kotlin Multiplatform﻿ Desktop Issue](https://youtrack.jetbrains.com/issue/KT-71782/Kotlin-Multiplatform-Desktop-Issue)、stack overflow上的提问[How to play audio in Kotlin Multiplatform (Desktop)](https://stackoverflow.com/questions/75064763/how-to-play-audio-in-kotlin-multiplatform-desktop)。这是Kotlin跨平台针对桌面应用开发的跨平台解决方案，使得开发者能够在Windows、macOS和Linux之间共享代码。

现在统一Kotlin Multiplatform(KMP)，读者只需知道KMD指的是什么即可。

### 5 Kotlin跨平台技术现状

这是当前Kotlin Multiplatform technology的各平台支持情况：

| Platform          | Stability level | Platform                 | Stability level |
| ----------------- | --------------- | ------------------------ | --------------- |
| Android           | Stable          | Web based on Kotlin/Wasm | Alpha           |
| iOS               | Stable          | Web based on Kotlin/JS   | Stable          |
| Desktop (JVM)     | Stable          | watchOS                  | Best effort     |
| Server-side (JVM) | Stable          | tvOS                     | Best effort     |

### 6 Kotlin跨平台各术语小结

* KMP与KMM、KMD：**KMP是基础，KMM和KMD是其具体应用，分别针对移动和桌面平台**，使得业务逻辑和数据模型可以在多个平台之间共享。
* Kotlin跨平台现在统一称为KMP(Kotlin Multiplatform)。**MPP、KMM、KMD等其他术语，Kotlin官方已经宣布弃用**。

## 六、Compose Multiplatform UI Framework

### 1 CMP简介

Compose Multiplatform（CMP）是JetBrains和谷歌合作推出的一项技术，旨在将Jetpack Compose扩展到更多平台。前面介绍过，Jetpack Compose 是 Google 为 Android 开发的声明式 UI 框架。而CMP使开发者能够使用Compose DSL在多个平台上创建一致的用户界面。目前，CMP支持包括Android、iOS、Web以及桌面平台等。

CMP目前最新版本是1.7.0，支持Kotlin2.0.20。2024年06月JetBrains宣布 [CMP对iOS的支持进入Beta极端、对web的支持进入Alpha阶段](https://blog.jetbrains.com/zh-hans/kotlin/2024/06/compose-multiplatform-1-6-10-ios-beta) 。并[计划2025年CMP对iOS的支持进入稳定阶段](https://blog.jetbrains.com/zh-hans/kotlin/2024/11/kotlin-multiplatform-development-roadmap-for-2025/)。这是当前Compose Multiplatform UI framework的各平台支持情况：

| Platform | Stability level | Platform                 | Stability level |
| -------- | --------------- | ------------------------ | --------------- |
| Android  | Stable          | Desktop (JVM)            | Stable          |
| iOS      | Beta            | Web based on Kotlin/Wasm | Alpha           |

CMP使得开发者使用相同的Compose DSL，可以在不同的平台上创建一致的用户界面。

### 2 CMP代码示例

其中KMP逻辑层代码在前面已经有过介绍，这里不再赘述。

其实前面笔者已经介绍了Google的Jetpack Compose (for Android)、JetBrains的Jetpack Compose for Desktop。CMP扩展了Jetpack Compose，使其能够支持更多的平台，如iOS和Web，从而实现真正的跨平台UI开发。读者可以参考[这里](https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-multiplatform-create-first-app.html#create-a-project-using-the-wizard)的官方教程。下面是其示例代码：

#### 2.1 Common模块UI层代码

这里使用@Composable编写声明式UI代码。

其中触发按钮`Click me!`的点击时，showContent的state发生改变，进而触发Compose Logo与打招呼文案的显示、隐藏。

```kotlin
@Composable
fun App() {
    MaterialTheme {
        var showContent by remember { mutableStateOf(false) }
        Column(Modifier.fillMaxWidth(), horizontalAlignment = Alignment.CenterHorizontally) {
            Button(onClick = { showContent = !showContent }) {
                Text("Click me!")
            }
            AnimatedVisibility(showContent) {
                val greeting = remember { Greeting().greet() }
                Column(Modifier.fillMaxWidth(), horizontalAlignment = Alignment.CenterHorizontally) {
                    Image(painterResource(Res.drawable.compose_multiplatform), null)
                    Text("Compose: $greeting")
                }
            }
        }
    }
}
```

#### 2.2 各平台UI层代码与打包渲染

Compose Multiplatform 通过 Kotlin Multiplatform 技术实现跨平台编译。Kotlin Multiplatform 允许开发者编写共享代码，并针对不同平台生成相应的二进制文件。Compose Multiplatform 利用这一技术，生成适用于不同平台的 UI 组件和应用程序包。

* Web端代码

    * 打包：针对 Web 平台，Compose Multiplatform 使用 Kotlin/JS 生成 JavaScript 代码，并通过 Webpack 或其他工具进行打包。

    * 渲染：在 Web 平台上，Compose Multiplatform 使用 Canvas API 或 DOM API 进行渲染，生成 HTML 和 JavaScript 代码。

```kotlin
fun main() {
    ComposeViewport(document.body!!) {
        App()
    }
}
```

* 桌面端代码

    * 打包：针对桌面平台，Compose Multiplatform 使用 Kotlin/Native 或 Kotlin/JVM 生成适用于 Windows、macOS 和 Linux 的代码，并通过 Gradle 打包。

    * 渲染：在桌面平台上，Compose Multiplatform 使用 Skia 图形库进行渲染，并通过 JVM 或 Native 平台的窗口管理系统进行交互。

```kotlin
fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        title = "CMPTry",
    ) {
        App()
    }
}
```

* iOS代码

    * 打包：针对 iOS 平台，Compose Multiplatform 生成适用于 iOS 的代码，并通过 Xcode 项目进行打包。

    * 渲染：在 iOS 平台上，使用 Kotlin/Native 生成的代码与 UIKit 或 SwiftUI 交互，实现 UI 渲染。

```kotlin
fun MainViewController() = ComposeUIViewController { App() }
```

* 安卓代码

    * 打包：针对 Android 平台，Compose Multiplatform 生成与 Jetpack Compose 兼容的代码，并使用 Android Gradle 插件进行打包。

    * 渲染：在 Android 平台上，Compose Multiplatform 使用 Jetpack Compose 的渲染引擎，通过 Skia 库进行图形渲染。

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            App()
        }
    }
}
```

这是Web、桌面端、iOS、安卓各端的展示效果。
![图片](/images/7/2619b304-6e73-4b88-bf23-030f2219d9c9.png)


### 4 CMP绘制过程

跟前面的Jetpack Compose绘制过程类似，这里不再赘述。


![图片](/images/7/679a2fc7-f1ac-4332-892a-1542158beb4e.png)


### 5 CMP VS KMP

* KMP使得我们在开发应用时，使用各端自己的native UI，**KMP共享逻辑层代码**。
* CMP使得我们在开发应用时，除了共享逻辑层代码，**CMP还UI层代码**。
* **KMP是CMP的基石，KMP的跨平台性为CMP的跨平台提供了可能**。
* 如果读者要开发包括iOS的跨平台应用，注意KMP已经稳定支持iOS、但CMP处于Beta阶段（还没有100%完成，means "you can use it, and we'll do our best to minimize migration issues for you"）。

## 七、发展历史与相关术语总结

Kotlin跨平台的概念最早出现在2017年，并且随着Kotlin语言的普及而逐渐成熟。在社区的广泛反馈和支持下，Kotlin跨平台已逐渐发展成为一个强大的跨平台解决方案。

在此也希望Kotlin跨平台能越来越完善。
<div align="center">
<img src="/images/7/bd2f25a1-91b7-46f1-922e-17aa61c41680.png" width=40% />
</div>


我们再总结一下相关术语：
* 缩写：
    - MPP: Multiplatform Projects 已弃用该说法，更改为KMP。
    - KMP: Kotlin Multiplatform 是kotlin语言技术层面的跨平台。跨平台语境下复用代码逻辑层。
    - KMM: Kotlin Multiplatform Mobile 已弃用该说法，统一到了KMP。
    - KMD: Kotlin Multiplatform Desktop 已弃用该说法，统一到了KMP。
    - CMP: Compose Multiplatform 是Compose UI Framework层面的跨平台。跨平台语境下复用UI层代码。
* 逻辑层面：
    - MPP是KMP的早期称呼。
    - KMM和KMD是KMP在移动和桌面平台的具体应用，分别在2020年和2021年进入稳定阶段。
    - 如果开发iOS APP，需要用Android Studio。因为KMM/KMP插件不能在IntelliJ IDEA上使用。
    - Kotlin跨平台相关术语在2023年已被统一收归为KMP。

* UI层面：
    - Google先推出的Jetpack Compose。仅for Android平台。
    - JetBrains基于Compose DSL和KMP推出了Compose for Desktop。仅for桌面端 Mac+Windows+Linux。
    - CMP是两家厂商在Jetpack Compose、KMP成熟之后，于2021年推出。
    - CMP扩展了Jetpack Compose，CMP使得Compose DSL能够支持更多的平台，从而实现完整的Compose跨平台UI开发。

最后，再看下这些关键词，你了解各自的含义了吗？
<div align="center">
<img src="/images/7/dec1d727-ac8c-4b32-b11a-2b1828638647.png" width=50% />
</div>




欢迎在评论区交流～

---

引用：

* 第一章中列举的链接不再重复阐述。
* Kotlin语言官网教程 https://kotlinlang.org/
* Kotlin multiplatform官网教程 https://www.jetbrains.com/kotlin-multiplatform/
* Compose multiplatform官网教程 https://www.jetbrains.com/compose-multiplatform/
* Jetpack Compose官网教程 https://developer.android.com/compose
* Swift UI官网教程 https://developer.apple.com/tutorials/develop-in-swift/hello-swiftui
* 谷歌的Android Jetpack介绍文章 https://android-developers.googleblog.com/2018/05/use-android-jetpack-to-accelerate-your.html
* KMM vs KMP https://medium.com/vmware-end-user-computing/kmm-vs-kmp-whats-the-deal-dd64bc5d61f2
* Kotlin官方的版本更新说明，以1.2为例 https://kotlinlang.org/docs/whatsnew12.html
* KMM/KMP插件 https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform

后续预告：
* 什么是 swift-export？目前支持到了什么程度？
* kotlin是怎么跟swift交互的，目前实现了哪种层次的交互、都能做什么交互？
* kotlin-native的详细原理是什么？