---
title: "pd首页重构实践"
date: 2023-11-30
draft: false
tags: ["性能优化", "安卓"]
categories: ["技术"]
---

> 使用ConcatAdapter、AsyncListDifferDelegationAdapter、MediatorLiveData、Flow、静态代码检查工具PMD&detekt等对安卓pd首页重构记录。


## 一、背景简介
### 1.1 业务背景
pd 首页按顺序包括“ *封面, 断网提示, 加入pd , 访客首页, 帖子互动消息, pd 主新手任务, pd 主实名, 禁言, pd 导航栏, 精选房间卡片, 全局公告, 帖子广场, 最近浏览子pd , 子pd 列表* ”等。

这里页面元素本身已然庞杂，加上是不同业务侧的同学迭代开发的，代码风格不统一、积重难返，有很大的维护成本、亟待梳理与优化。

本文主要记录了使用`ConcatAdapter`、`AsyncListDifferDelegationAdapter`、`MediatorLiveData`、`Flow`、静态代码检查工具`PMD`&`detekt`等对安卓手Qpd 首页重构过程。

### 1.2 效果简介
笔者最终借助`ConcatAdapter`等将pd 首页各内容，按照业务逻辑解耦开来，进行了代码结构清晰、质量良好的重构实现。
![图片](/images/5/e69b0edd-7513-478d-b745-b8c2567cb45c.png)



## 二、现网实现

此章节主要是笔者在维护此模块过程中，梳理了一些维护成本高的原因。

先简单介绍下现网整体实现。这里自上而下地，借助`RecyclerViewWithHeaderFooter`，将“pd 封面标题～置顶区域”，全部作为一个大的`View`塞给了`RecyclerView`的`header`部分。
![图片](/images/5/49c2ff2b-7046-4559-923b-c457e3034e51.png)

据了解，这里的代码结构原因主要是**前期置顶区域只有简单的“加入pd 按钮”等，可以将其与“蜂封面标题”看作一个整体**，这自然无可厚非。

**但随着业务迭代，置顶区域逐渐拓展到了十来个**。拓展后，此区域包含复杂的数据来源、层层嵌套的子列表、复杂的跳转交互处理、音视频组件等。此时，再将其看作一个整体，就有待商榷了。


### 2.1 错综复杂的代码风格

xxx

可以看到，各个置顶项组织方式、包位置等比较混乱。有诸多问题点。包括：
- **各置顶项形式不一致，既有纯view成员、又有controller等**。
- 各置顶项**代码位置散乱**，分布在不同包下。
- 各置顶项**职责不一致**，既有纯view，把数据源交给上级；又有内部自我控制数据源的。
- view不推荐持有controller。controller是处理与界面相关的事件和数据的。
- controller不推荐放到widget包下。widget本质是负责渲染和交互的组件。



### 2.2 难以为继的间距处理

**不同的pd 首页item，相互的间距不同**。

简单举例：A和B的间距是48px、B和C的间距是36px，但B不显示时，A和C的间距是24px。而现网代码散乱在各处，非常难以控制。

### 2.3 晦涩难懂的动画实现

“pd 封面标题+置顶区域”全部写在了一个固定`View`的`ViewHolder`，数据变化时不经过`RecyclerViewAdapter`的`notifyItem`。因此无法使用`RecyclerViewItemAnimator`直接实现全部内容。拓展性较差。

### 2.4 每况愈下的代码质量

在功能的不断迭代中，涉及不同业务侧的开发参与，**代码质量在不断下跌**。体现在：

2.4.1 ** 类文件职责与行数不断膨胀**

以`xxController`(1530行)为例，在处理`viewModel`、`view`的基础上，陆续引入了：
* 直接监听sdk数据变化(建议放到ViewModel)。
* 下拉刷新直接进行pd 信息拉取(建议放到ViewModel)。
* 下拉刷新身份组相关接口拉取(建议放到ViewModel)。
* 处理部分置顶项的强业务相关逻辑(建议放到业务类本身处理)。
* 直接处理&构建一些跳转信息(建议放到子handler)。

2.4.2 **代码可读性、质量不断下降**，以PMD静态代码检查报告为例，可以看到不少代码问题。

xx


## 三、重构方案
这里的重构，主要是拆分了“封面标题、置顶区域”这一庞杂的类，通过`ConcatAdapter`等，将其UI、数据分别分散解耦开。

![图片](/images/5/eaf8058c-6b2a-4509-b0a7-9e853700d8e0.png)


### 3.1 UI 层

这里我们期望对“封面标题、置顶区域”这一整体进行解耦，每一个小项，都作为`RecyclerView.ViewHolder`独立存在。

#### 3.1.1 ConcatAdapter

现网已有子pd 列表区域的`xxListAdapter`，且子pd 列表的业务逻辑已经比较庞杂。因此我们不期望将“封面标题、置顶区域、子pd 列表”三大块的`Adapter`、`ViewModel`、数据列表混合到一起；希望将这三块内容独立开来。
我们最终选用了`ConcatAdapter`，先看一下官网的介绍：

>ConcatAdapter
```java
java.lang.Object
   ↳   androidx.recyclerview.widget.RecyclerView.Adapter
       ↳  androidx.recyclerview.widget.ConcatAdapter
```
An  `Adapter`  implementation that presents the contents of multiple adapters in sequence.
```java
MyAdapter adapter1 = ...;
AnotherAdapter adapter2 = ...;
ConcatAdapter concatenated = new ConcatAdapter(adapter1, adapter2);
recyclerView.setAdapter(concatenated);
```
By default,  `ConcatAdapter`  isolates view types of nested adapters from each other such that it will change the view type before reporting it back to the  `RecyclerView`  to avoid any conflicts between the view types of added adapters. This also means each added adapter will have its own isolated pool of  `ViewHolder` s, with no re-use in between added adapters.

可以看到，`ConcatAdapter`允许我们针对“封面标题、置顶区域、子pd 列表”三大块构建不同的子`Adapter`，再聚合到一起。比较符合我们的预期。针对pd 首页的技术实现上，最终方案：


- `CoverViewAdapter` 包含: 封面标题；有一个单独的`CoverViewModel`控制其数据刷新。
- `HeaderBarsAdapter` 包含: pd 首页置顶区域，加入pd ～帖子广场；有一个单独的`HeaderBarsViewModel`控制其数据刷新。
- `ChannelListAdapter` 包含: 最近浏览子pd ～子pd 列表；有一个单独的`ChannelListViewModel`控制其数据刷新。


为此，我们完成了pd 首页内容架构的调整，可以下线`GuildFacadeTopFrameController`、`GuildChannelListHeadView`相关代码，并使存量各个独立的置顶项`View`层代码改写为`RecyclerViewHolder`即可。
#### 3.1.2 AsyncListDifferDelegationAdapter

`AsyncListDifferDelegationAdapter`  是一个高效的适配器。该适配器支持对列表项的修改`notifyItemChanged`、删除`notifyItemRemoved`以及插入`notifyItemInserted`等操作，并且可以在不影响 UI 的情况下，异步处理数据差异，从而提高了适配器的效率；并且可以通过将数据差异委托给其他适配器来保持 UI 响应。
简单使用示例如下：

```
class MyAdapter() : AsyncListDifferDelegationAdapter<MyUIData>(createDiffCallback()) {

    init {
        // 注册 delegate
        delegatesManager.addDelegate(MyAdapterDelegate1())
            .addDelegate(MyAdapterDelegate2)
    }

    companion object {
        // 构建 Diff 比较方式、用于局部刷新
        private fun createDiffCallback(): DiffUtil.ItemCallback<MyUIData> =
            object : DiffUtil.ItemCallback<MyUIData>() {
                override fun areItemsTheSame(oldItem: MyUIData, newItem: MyUIData): Boolean {
                    TODO("Not yet implemented")
                }

                override fun areContentsTheSame(oldItem: MyUIData, newItem: MyUIData): Boolean {
                    TODO("Not yet implemented")
                }

                override fun getChangePayload(oldItem: MyUIData, newItem: MyUIData): Any? {
                    return super.getChangePayload(oldItem, newItem)
                }
            }
    }
}
```
其中需要注意的是，使用 `DiffUtil` 比较时 ，如果一个元素发生了变化，新、旧的集合中该元素不能是同一个实例。不然在 `DiffUtil.ItemCallback` 回调中比较时，他们是同一个实例对象，会误判为数据没发生变化，也就不会通知 `notifyItemChanged()` 。

```
/**
 * differ 比较需要不同的对象，不然会一直比较数据没变化
 */
@JvmStatic
fun <T : GuildHeaderBarsUIData> MutableList<T>.replaceData(item: T): T? {
    val oldItem = this.find { it.javaClass == item.javaClass }
    if (oldItem == null) {
        Logger.w(TAG) { "replaceData but not found old data!" }
        this.add(item)
        return null
    }
    this[this.indexOf(oldItem)] = item
    return oldItem
}
```
#### 3.1.3 RecyclerView.ItemAnimator

通过3.1.1和3.1.2，我们已然能够将pd 首页的元素，全部`ViewHolder`化。我们也就能够直接使用`RecyclerView.ItemAnimator`了，可以看一下简介：

> RecyclerView.ItemAnimator
> This class defines the animations that take place on items as changes are made to the adapter. Subclasses of ItemAnimator can be used to implement custom animations for actions on ViewHolder items. The RecyclerView will manage retaining these items while they are being animated, but implementors must call  `dispatchAnimationFinished`  when a ViewHolder's animation is finished. In other words, there must be a matching  `dispatchAnimationFinished`  call for each  `animateAppearance()` ,  `animateChange()`, `animatePersistence()` , and  `animateDisappearance()`  call.


具体地，针对2.3-复杂的动画实现，在`ItemAnimator`内部完整实现精选房间卡片的动画：
1. `DiffUtil.ItemCallback`在`getChangePayload`时传递`payloads`，用于后续告知`ItemAnimator`该次刷新需要做动画。
2. `ItemAnimator` `recordPreLayoutInformation`读取`payloads`，并构建`ItemHolderInfo`。
3. `ItemAnimator` `animateChange`时，读取`ItemHolderInfo`，决定是否做动画，并将做动画`ViewHolder`存起来。
4. `ItemAnimator` `runPendingAnimations`，读取存起来的要做动画的`ViewHolder`，并执行消失、出现、位移等动画。


### 3.2 数据层

#### 3.2.1 MediatorLiveData

既然使用了`RecyclerView`，那我们必然需要在数据层把需要展示的置顶项收归到一个列表。而重构前，各个置顶项子业务，已经写了一些存量的业务`ViewModel`。出于工作量的考虑，我们没有必要全部重写置顶区域数据源。因此需要一种方式，把存量`ViewModel`中的各个`LiveData`数据聚合起来、并进一步组合成列表、最终提交给`RecyclerViewAdapter`。
我们最终使用的是`MediatorLiveData`，先看一下官网的介绍。

>MediatorLiveData
```
java.lang.Object
  ↳ android.arch.lifecycle.LiveData<T>
      ↳ android.arch.lifecycle.MutableLiveData<T>
          ↳ android.arch.lifecycle.MediatorLiveData<T>
```
`LiveData`  subclass which may observe other  `LiveData`  objects and react on  `OnChanged`  events from them.
This class correctly propagates its active/inactive states down to source  `LiveData`  objects.
Consider the following scenario: we have 2 instances of  `LiveData` , let's name them  `liveData1`  and  `liveData2` , and we want to merge their emissions in one object:  `liveDataMerger` . Then,  `liveData1`  and  `liveData2`  will become sources for the  `MediatorLiveData liveDataMerger`  and every time  `onChanged`  callback is called for either of them, we set a new value in  `liveDataMerger` .
```
 LiveData liveData1 = ...;
 LiveData liveData2 = ...;
​
 MediatorLiveData liveDataMerger = new MediatorLiveData<>();
 liveDataMerger.addSource(liveData1, value -> liveDataMerger.setValue(value));
 liveDataMerger.addSource(liveData2, value -> liveDataMerger.setValue(value));
```

具体到我们的业务，以pd 导航栏为例，其存量`ViewModel`有“可见性 isHideObservable”、“pd ID guildIdObservable”、“导航栏列表guildNavigationInfoObservable”三个LiveData，我们需要将三个LiveData的数据聚合成一个UIData，提交到`RecyclerViewAdapter`给`ViewHolder`展示使用。
为此，这里给LiveData写了一个针对pd 置顶区域的拓展方法，专门用于聚合其他额外的1~3个LiveData。

```
  /**
  * 可见性 LiveData 计算体，当自己不可见；或者自己可见且相关 1-3 个 data 都有数据时，再emit更新
  * @param hideStatus 自己不可见时 this.value 的值
  */
  fun <T, A, B, C, D> LiveData<A>.visibleCombine(
      hideStatus: A,
      other1: LiveData<B>,
      option2: LiveData<C>?,
      option3: LiveData<D>?,
      onChange: (A?, B?, C?, D?) -> T
  ): MediatorLiveData<T> {
      var source0Emitted = false
      var source1Emitted = false
      var source2NeedEmit = option2 == null
      var source3NeedEmit = option3 == null

      val result = MediatorLiveData<T>()
      val mergeFunc = {
          if (source0Emitted && this.value == hideStatus ||
              source0Emitted && source1Emitted && source2NeedEmit && source3NeedEmit
          ) {
              result.value = onChange.invoke(this.value, other1.value, option2?.value, option3?.value)
          }
      }
      result.addSource(this) { source0Emitted = true; mergeFunc.invoke() }
      result.addSource(other1) { source1Emitted = true; mergeFunc.invoke() }
      option2?.let { result.addSource(it) { source2NeedEmit = true; mergeFunc.invoke() } }
      option3?.let { result.addSource(it) { source3NeedEmit = true; mergeFunc.invoke() } }
      return result
  }
```
使用举例：

```
// 不需要聚合LiveData变化的子ViewModel,正常监听即可
globalMsgVM.globalMsgLiveData.observe(lifecycleOwner) { uiData ->
    if (this@GuildHeaderBarsViewModel.guildId != uiData.guildId) {
        return@observe
    }
    replaceDataAndSubmit(NoticeGlobalMsgV2(isActive = uiData.hasValidInfo(), uiData = uiData))
}

// 需要聚合多个LiveData变化的子ViewModel
navigationBarVM.isHideObservable.visibleCombine(
    true, navigationBarVM.guildIdObservable, navigationBarVM.guildNavigationInfoObservable, nullLivedata
) { isHide, guildId, infoList, _ ->
    NoticeNavigationBar(isHide == false, guildId, infoList)
}.observe(lifecycleOwner) { noticeNavigationBar ->
    if (this@GuildHeaderBarsViewModel.guildId != noticeNavigationBar.guildId) {
        return@observe
    }
    replaceDataAndSubmit(noticeNavigationBar)
}
```
以此，我们完成了数据层存量`ViewModel`的数据聚合。

#### 3.2.2 数据聚合接口

3.2.1的方案，解决了数据的聚合问题。但是数个版本后，这里又来了**新的挑战：冷启动时由于数据源分散，谁有数据就直接展示，会导致短时间内页面的频繁跳变**。为了良好的用户体验，我们**期望尽可能以较少的次数，将数据全部刷新出来**。

为了跨端的表现统一，我们决定**摒弃在UI层的置顶项可见性数据源聚合，用跨平台`sdk`层提供数据收归的聚合接口，统一返回置顶区域数据**。对于首屏单个置顶项的可见性，核心聚合接口采用：异步拉取接口`getXXMainFrameHeaderInfo` + 监听接口 `onXXMainFrameHeaderUpdated`的方式。**具体地，调用拉取接口时，优先拿到本地已有的数据，并用于UI层第一次展示；对于本地暂无数据的情况，由`sdk`层统一拉取后，一并返回，并用于UI层第二次展示**。


#### 3.2.3 页面刷新频率控制

3.1.2的`DiffUtil`较，已经能优化页面的刷新范围。但在数据层，我们可以进一步优化数据刷新的频率。具体地，我们将数据变化以flow的方式抛出，并做统一聚合。

以下述代码为例，我们可以控制一些关注度较小的置顶项内容更新，控制在800ms聚合一次，统一抛通知去刷新。

```
private fun collectProcessorEventSource() {
   collectJob?.cancel()
   collectJob = processors.eventSource
       .filter { it.guildId == this.guildId }
       .onEach { event ->
           handleProcessorEvent(event)
       }.throttle(800L) {  // 频率（时间）控制
           notifyDataChanged(false, "collectProcessorEventSource")
       }.catch {
           Logger.e(TAG) { "collectProcessorEventSource error $it \n${it.stackTraceToString()}" }
       }.launchIn(viewModelScope)
   collectJob?.invokeOnCompletion {
       Logger.d(TAG) { "collectProcessorEventSource completed $it" }
   }
}
```
#### 3.2.4 统一的间距处理

由于在UI层滞后地处理间距问题比较复杂，重构后**将间距数据前置地放到了数据层统一处理**。具体地，重构后新构建一种空白内容的 `ViewHolder` 作为间距 `item` ，在数据层构建其 `UIData` 。


## 四、效果对比
### 4.1 方案设计对比
在整体方案上，我们拆分pd 首页“封面标题”、“置顶区域”，使此整个模块一一`ViewHolder`化，并通过`ConcatAdapter`将整个列表重新聚合到一起。可以看到这种抽象层次更加符合业务功能直觉，维护起来也方便细分边界、划分模块等。
![图片](/images/5/7510b80d-0a08-44e4-8a8e-b18aad9b50bb.png)


### 4.1 目录结构对比

以下图置顶区域为例，可以看到，旧版代码各个包位置混乱不堪，维护起来给问题定位带来了极大的干扰。重构后，整个置顶区域的代码集中到了一块，降低学习成本、方便定位代码等。
xxx


### 4.2 静态代码扫描报告对比
这里主要使用PMD扫描Java代码，使用detekt扫描Kotlin代码。
可以看到，旧版代码随着业务迭代，单个文件能用PMD工具扫描出上百的建议，代码质量令人堪忧。重构后，再用detekt工具结合司内的kotlin代码规范进行扫描，代码建议均控制在个位数（未清零原因：单个类函数数量较多）。
![图片](/images/5/15493835-5bf9-4232-87c7-1e723d3ee5c9.png)

---
参考资料:


> [ConcatAdapter] https://developer.android.com/reference/androidx/recyclerview/widget/ConcatAdapter 	
> [MediatorLiveData]    https://developer.android.com/reference/android/arch/lifecycle/MediatorLiveData 	
> [pmd 插件]    https://plugins.jetbrains.com/plugin/1137-pmd
> [detekt 插件]    https://plugins.jetbrains.com/plugin/10761-detekt