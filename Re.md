[TOC]

# 概念

## 副作用

副作用是指在函数执行过程中产生对外部环境的影响，如修改函数外部变量。另外调用DOM API、I/O操作、控制台打印信息等函数调用过程中产生的，外部可观察的变化都属于副作用

## 纯函数

如果一个函数同时满足（1）相同的输入始终获得相同的输出（2）不会修改程序的状态或引起副作用这两个条件，则为纯函数

# React的架构

React的架构可以分为三个部分：

Scheduler（调度器）——调度任务的优先级，高优先级任务优先进入Reconciler协调器

Reconciler（协调器）——VDOM的实现， 负责根据自变量变化计算出UI变化

Renderer（渲染器）——负责将UI变化渲染到宿主环境中

在新架构中，Reconciler中的更新流程从递归变成了可中断的循环任务，每次循环都会掉用shouldYield判断当前time Slice是否有剩余时间， 如果没有剩余时间则暂停更新，将主线程交给渲染流水线，等待下一个任务再继续执行

```js
//workLoopConcurrent、shouleYield方法
function workLoopConcurrent(){
  ///一直循环任务，直到任务执行完成或者中断
  while(workInProgress !== null && !shouleYield()){
    performUnitOfWork(workInProgress)
  }
}

//shouleYield方法
function shouleYield(){
  //当前时间是否大于过期时间
  //其中deadline = getCurrentTime()+yieldInterval
  //yieldInterval为调度器预设的时间间隔，默认为5ms，这也是每个Time Slice宏任务的时间长度是5ms左右的原因
  return getCurrentTime() >= deadline
}
```

当Scheduler将调度后的任务交给Reconciler之后，Reconciler会为VDOM元素标记各种副作用flags

Renderer根据Reconciler为VDOM元素标记的各种flags执行对应操作

# Fiber

## FiberNode

```js
//ReactFiber.old.js
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  //作为静态的数据结构 保存节点的信息 
  this.tag = tag;//对应组件的类型Funcion/Class/Host...
  this.key = key;//key属性
  this.elementType = null;//元素类型，大部分情况同type，某些情况不同，比如FunctionComponent使用React.memo包裹
  this.type = null;//func或者class
  this.stateNode = null;//真实dom节点

  //作为fiber树架构 连接成fiber树
  this.return = null;//指向父FiberNode
  this.child = null;//指向第一个子FiberNode
  this.sibling = null;//指向右边的兄弟FiberNode
  this.index = 0;

  this.ref = null;

  //用作为工作单元 来计算state
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;
    
	//effect相关
  this.effectTag = NoEffect;
  this.nextEffect = null;
  this.firstEffect = null;
  this.lastEffect = null;
    
  //与本次更新将在Renderer中执行的操作相关
  this.flags = NoFlags
  this.subtreeFlags = NoFlags
  this.deletions = null

  //优先级相关的属性
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  //current和workInProgress的指针
  this.alternate = null;
}
```

这里有一个细节，为什么指向父FIberNode的字段叫做return而不是parent或者father？因为作为一个工作单元，return指FiberNode执行完completeWork后返回的下一个FiberNode。子FiberNode及其兄弟FiberNode执行完completeWork后会返回父FiberNode，所以return用来指代父FiberNode

## 双缓存机制

Fiber 架构的工作原理类似于显卡的工作原理。具体来讲，显卡包含前缓冲区和后缓冲区。对于刷新频率为60赫兹的显示器每秒会从前缓冲区读取60次图像将其显示到显示器上，显卡的职责是合成图像并写入后缓冲区，一旦后缓冲区被写入图像，前后缓冲区就会互换。这种将数据保存在缓冲区再替换的技术，被称为双缓存。

Fiber架构中同时存在两棵Fiber树，一颗是真实UI对应的Fiber树，可以理解为前缓冲区，叫做current Fiber Tree。另一棵是正在内存中构建的Fiber Tree，可以理解为后缓冲区，叫做workInProgress Fiber Tree。Fiber上的alternate属性指向另一个缓冲区对应的FIberNode

## Fiber Tree

Fiber Tree的构建：

**mount**

有两种情况：整个应用的首次渲染、某个组件的首次渲染（没有下述1、2两步）

（1）创建FiberRootNode

（2）创建tag为3的FiberNode，代表hostRoot，称为HostRootFiber（HostRoot代表应用在宿主环境挂载的根节点，FiberRootNode通过current指向HostRootFiber，HostRootFiber通过stateNode指向FiberRootNode）

（3）从HostRootFiber开始，以DFS深度优先搜索的顺序生成FIberNode构造workInProgress FIber Tree，并在遍历过程中，为FIberNode标记代表不同副作用的flags，以便后续再Renderer中使用

 （4）把workInProgress Fiber Tree切换成current Fiber Tree

**Update**

（1）根据current Fiber创建workInProgress Fiber

（2）把workInProgress Fiber Tree切换成current Fiber Tree

# Schedule调度器

React实现了一套基于lane模型的优先级算法，并基于这套算法实现了Batched Updates（批量更新）、任务打断/恢复机制等特性。

Scheduler对外导出的scheduleCallback（优先级,fn）方法接收优先级与回调函数fn去调度fn的执行，这个方法会返回task这一数据结构代表一个被调度的任务work，task.calback = fn,task.expirationTime:startTime+timeout，startTime一般为执行scheduleCallBack时的当前时间，如果传递了delay参数还会在此基础上增加延迟时间，不同优先级对应不同的timeout。

根据是否传递delay参数，执行scheduleCallBack方法后生成的task会进入timerQueue或taskQueue，其中timerQueue中的task以currentTime+delay为排序依据，taskQueue中的task以expirationTime为排序依据

当timerQueue中的第一个task延迟的时间到期后，执行advanceTimers将到期的task从timerQueue移动到taskQueue中

接下来执行requestHostCallback方法，它会在新的宏任务中执行workLoop方法。workLoop方法会循环消费taskQueue中的task，直至taskQueue中不存在task或者Time Slice时间用完且当前task未过期则中断循环。

循环中断后如果taskQueue不为空，则继续执行requestHostCallback方法，如果timerQueue不为空则继续执行advanceTimers方法

执行是通过perform方法执行。

work的工作流程随时可能中断（shouldYield），比如React应用很复杂需要遍历很多组件的情况或者React单个组件render逻辑复杂的情况。但是当满足work的priority是ImmediatePriority时即同步优先级或者priority是didTimeout即本次调度已过期时，work对应的工作不会中断而是同步执行直到work结束。

didTimeout这个参数的意义是为了避免饥饿问题，当一个work长时间未执行完，随着时间推移当前时间离work.expirationTime越近，work的优先级越高。当work.expirationTime小于当前时间即该work已经过期，表现为didTimeout为true，过期work会被同步执行。

Schedule存在一大一小两种循环，大循环式调度优先级最高的任务的执行，小循环是调度一个相同优先级任务的反复执行。

## 优先级队列的实现

用小顶堆的数据结构实现优先级队列，小顶堆的特点是：是一棵完全二叉树，除最后一层外，其他层的节点个数都是满的，且最后一层节点是靠左排列，并且堆中每一个节点的值都小于等于其子树的每一个值。完全二叉树很适合用数组保存，用数组下标代表指向左右子节点的指针，比如数组下标为i的节点的左右子节点下标分别为2i+1、2i+2。

push向堆中推入数据和pop从堆顶取出数据时间复杂度与二叉树的高度正相关。为O(log n)

peek获取排序依据最小的值对应节点时间复杂度为O(1)

## 宏任务的选择

workLoop方法会在新的宏任务中执行，浏览器会在宏任务执行间隔执行Layout、Paint。

（1）requestIdleCallback（简称rIC），它会在每帧的空闲时期执行，但有如下缺点：浏览器兼容性、执行频率不稳定比如切换浏览器Tab之后之前Tab注册的rIC执行的频率会被大幅降低。再有就是rIC的设计初衷是能够在事件循环中执行低优先级的工作从而减少对动画、用户输入等高优先级事件的影响，这意味着rIC的应用场景被局限在低优先级工作中，这与Schedule中多种优先级的调度策略不符。

（2）requestAnimationFrame（建成rAF），这个API定义的回调函数会在浏览器下次Paint前执行，一般用于更新动画。因为rAF的执行时机取决于每一帧Paint前的时机，即它的执行与帧相关，执行频率不高，满足条件的话应该一帧时间内可以执行多次并且执行时机越早越好。

**Schedule最终选择**

（1）在支持setImmediate的环境（Nodejs）中，Schedule使用setImmediate调度宏任务。原因是不同于MessageChannel，它不会阻止Nodejs进程退出，并且相比MessageChannel执行时机更早。

（2）在支持MessageChannel的环境（浏览器、worker）中，使用MessageChannel调度宏任务。这个API会创建一个新的消息通道，并通过它的两个MeaagePort属性发送数据，接收消息的回调函数onmessage会在新的宏任务重执行。

（3）其他情况使用setTimeOut调度宏任务

## Lane模型

Lane的和Scheduler是两套优先级机制，相比来说Lane的优先级粒度更细，Lane的意思是车道，类似赛车一样，在task获取优先级时，总是会优先抢内圈的赛道

Scheduler预置了五种优先级，优先级依次降低：

- ImmediatePriority（最高优先级，同步执行） 
- UserBlockingPriority 
- NormalPriority 
- LowPriority 
- IdlePriority（最低优先级）

作为独立的包，考虑到通用性，Scheduler和React并不共用一套优先级体系，React有四种优先级：

```js
export const DiscreteEventPriority = SyncLane
//DiscreteEventPriority对应离散事件的优先级，例如click、input、focus等事件
export const ContinuousEventPriority = InputContinuousLane
//ContinuousEventPriority对应连续事件的优先级，例如drag、mounsemove、scroll等事件
export const DefaultEventPriority = DefaultLane
//DefaultEventPriority对应默认的优先级，例如通过计时器周期性触发更新
export const IdleEventPriority = IdleLane
//IdleEventPriority对应空闲状态的优先级
```

从React到Schedule优先级要经历两次变换（1）将lanes转换为EventPriority（2）将EventPriority转换为Schedule优先级

lane模型需要解决两个基本问题：（1）以优先级为依据，对update进行排序（2）表达“批”的概念

对于第一个问题，一个lane就是一个32bit Int。最高位是符号位所以最多可以有31位参与运算。不同的优先级对应不同lane，越低的位代表越高的优先级。

对于第二个问题，React为TransitionLane预留了16个位，通过位运算可以判断某一优先级（某一lane）是否属于同一批（某个lanes）。expirationTime模型经过改进最多只能将范围内的update划分为批，而lane模型可以将多个不相邻的优先级划分为批。

**Lane模型中task是怎么获取优先级的（赛车的初始赛道）**

 任务获取赛道的方式是从高优先级的lanes开始的，这个过程发生在findUpdateLane函数中，如果高优先级没有可用的lane了就下降到优先级低的lanes中寻找，其中pickArbitraryLane会调用getHighestPriorityLane获取一批lanes中优先级最高的那一位，也就是通过`lanes & -lanes`获取最右边的一位

```js
export function findUpdateLane(
  lanePriority: LanePriority,
  wipLanes: Lanes,
): Lane {
  switch (lanePriority) {
    //...
    case DefaultLanePriority: {
      let lane = pickArbitraryLane(DefaultLanes & ~wipLanes);//找到下一个优先级最高的lane
      if (lane === NoLane) {//上一个level的lane都占满了下降到TransitionLanes继续寻找可用的赛道
        lane = pickArbitraryLane(TransitionLanes & ~wipLanes);
        if (lane === NoLane) {//TransitionLanes也满了
          lane = pickArbitraryLane(DefaultLanes);//从DefaultLanes开始找
        }
      }
      return lane;
    }
  }
}
```

**Lane模型中高优先级是怎么插队的（赛车抢赛道）**

 在Lane模型中如果一个低优先级的任务执行，并且还在调度的时候触发了一个高优先级的任务，则高优先级的任务打断低优先级任务，此时应该先取消低优先级的任务，因为此时低优先级的任务可能已经进行了一段时间，Fiber树已经构建了一部分，所以需要将Fiber树还原，这个过程发生在函数prepareFreshStack中，在这个函数中会初始化已经构建的Fiber树

```js
function ensureRootIsScheduled(root: FiberRoot, currentTime: number) {
  const existingCallbackNode = root.callbackNode;//之前已经调用过的setState的回调
  //...
	if (existingCallbackNode !== null) {
    const existingCallbackPriority = root.callbackPriority;
    //新的setState的回调和之前setState的回调优先级相等 则进入batchedUpdate的逻辑
    if (existingCallbackPriority === newCallbackPriority) {
      return;
    }
    //两个回调优先级不一致，则被高优先级任务打断，先取消当前低优先级的任务
    cancelCallback(existingCallbackNode);
  }
	//调度render阶段的起点
	newCallbackNode = scheduleCallback(
    schedulerPriorityLevel,
    performConcurrentWorkOnRoot.bind(null, root),
  );
	//...
}
```

**Lane模型中怎么解决饥饿问题**

 在调度优先级的过程中，会调用markStarvedLanesAsExpired遍历pendingLanes（未执行的任务包含的lane），如果没过期时间就计算一个过期时间，如果过期了就加入root.expiredLanes中，然后在下次调用getNextLane函数的时候会优先返回expiredLanes

# Reconciler协调器（render阶段）

Reconciler工作的阶段在React内部被称为render阶段，根据Scheduler调度的结果不同，render阶段可能开始于performSyncWorkOnRoot（同步更新流程）或performConcurrentWorkOnRoot（并发更新流程）方法。

```js
//performSyncWorkOnRoot会执行该方法
function workLoopSync(){
  while(workInProgress !== null){
    performUnitOfWork(workInProgress)
  }
}
//performConcurrentWorkOnRoot会执行该方法
function workLoopConcurrent(){
  while(workInProgress !== null && !shouldYield()){
    performUnitOfWork(workInProgress)
  }
}
//workInProgress变量代表“生成Fiber Tree工作已经进行到的workInProgress fiberNode”，performUnitOfWork方法会创建下一个fiberNode并赋值workInProgress，并将workInProgress与已创建的fiberNode连接起来构成FiberTree
```

performUnitOfWork的工作可以分为两个部分：“递”和“归”

“递”阶段会从HostRootFiber开始向下以DFS的方式遍历，为遍历到的每个fiberNode执行beginWork方法，该方法会根据传入的fiberNode创建下一级fiberNode，有两种情况：

（1）下一级只有一个元素，这时beginWork会创建子fiberNode，并与wip连接

（2）下一级有多个元素，这时beginWork方法会依次创建所有子fiberNode并且子FiberNode依次连接在一起，为首的子FiberNode会与wip（父FIberNode）连接

当遍历到叶子元素（不包含子FiberNode时），performUnitOfWork就会进入“归”阶段

“归”阶段会调用completeWork方法处理fiberNode。当某个fiberNode执行完completeWork方法后，如果其存在兄弟fiberNode，会进入其兄弟fiberNode的“递”阶段，如果不存在兄弟fiberNode，则进入父fiberNode的“归”阶段。

## beginWork

首先判断当前流程属于mount还是update阶段（判断依据是current !== null），如果是update阶段，则如果本次更新不影响fiberNode.child，则可以复用对应的current FiberNode，这是一条render阶段的优化路径。如果无法复用，则mount和update的流程大体一致，包括：（1）根据wip.tag进入不同类型元素的处理分支（2）使用reconcile算法生成下一级fiberNode

mount和update的区别在于最终是否会为生成的子FiberNode标记副作用flags

```js
//beginWork方法
//beginWork主要的工作是创建或复用子fiber节点
function beginWork(
  current: Fiber | null,//当前存在于dom树中对应的Fiber树
  workInProgress: Fiber,//正在构建的Fiber树
  renderLanes: Lanes,//优先级
): Fiber | null {
 // 1.update时满足条件即可复用current fiber进入bailoutOnAlreadyFinishedWork函数
  if (current !== null) {
    const oldProps = current.memoizedProps;
    const newProps = workInProgress.pendingProps;
    if (
      oldProps !== newProps ||
      hasLegacyContextChanged() ||
      (__DEV__ ? workInProgress.type !== current.type : false)
    ) {
      didReceiveUpdate = true;
    } else if (!includesSomeLane(renderLanes, updateLanes)) {
      didReceiveUpdate = false;
      switch (workInProgress.tag) {
        // ...
      }
      return bailoutOnAlreadyFinishedWork(
        current,
        workInProgress,
        renderLanes,
      );
    } else {
      didReceiveUpdate = false;
    }
  } else {
    didReceiveUpdate = false;
  }

  //2.根据tag来进入不同处理逻辑来创建不同的fiber 最后进入reconcileChildren函数
  switch (workInProgress.tag) {
    case IndeterminateComponent: //FC mount时进入的分支
      // ...
    case LazyComponent: 
      // ...
    case FunctionComponent: //FC update时进入的分支
      // ...
    case ClassComponent: 
      // ...
    case HostRoot:
      // ...
    case HostComponent://HostComponent代表原生Element类型比如div、span
      // ...
    case HostText: //文本元素类型
      // ...
  }
}
```

创建子fiber的过程会进入reconcileChildren，该函数的作用是为workInProgress fiber节点生成它的child fiber即 workInProgress.child。然后继续深度优先遍历它的子节点执行相同的操作。

```js
//ReactFiberBeginWork.old.js
export function reconcileChildren(
  current: Fiber | null,
  workInProgress: Fiber,
  nextChildren: any,
  renderLanes: Lanes
) {
  if (current === null) {
    //mount时
    workInProgress.child = mountChildFibers(
      workInProgress,
      null,
      nextChildren,
      renderLanes,
    );
  } else {
    //update时
    workInProgress.child = reconcileChildFibers(
      workInProgress,
      current.child,
      nextChildren,
      renderLanes,
    );
  }
}
```

reconcileChildren会区分mount和update两种情况，进入mountChildFibers或reconcileChildFibers。mountChildFibers和reconcileChildFibers都是ChildReconciler方法传递不同的参数返回的函数，这个shouldTrackSideEffects参数用来表示是否为对应的节点标记flags追踪副作用

```js
//两者的区别只是传参不同
var reconcileChildFibers = ChildReconciler(true);
var mountChildFibers = ChildReconciler(false);

function childReconciler(shouldTrackSideEffects){
  //省略代码实现
}
```

## flags位运算

所有flags都在packages/react-reconciler/src/ReactFiberFlags.js中定义，以Int32（32位有符号整数）的形式参与运算

标记flags的本质是二进制数的位运算，位运算（如与或非）可以用来表示增删改查

## completeWork

completeWork主要工作是处理fiber的props、创建dom、创建effectList

```js
//ReactFiberCompleteWork.old.js
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  const newProps = workInProgress.pendingProps;
    
//根据workInProgress.tag进入不同逻辑，这里我们关注HostComponent，HostComponent，其他类型之后在讲
  switch (workInProgress.tag) {
    case IndeterminateComponent:
    case LazyComponent:
    case SimpleMemoComponent:
    case HostRoot:
   	//...
      
    case HostComponent: {
      popHostContext(workInProgress);
      const rootContainerInstance = getRootHostContainer();
      const type = workInProgress.type;

      if (current !== null && workInProgress.stateNode != null) {
        // update时
       updateHostComponent(
          current,
          workInProgress,
          type,
          newProps,
          rootContainerInstance,
        );
      } else {
        // mount时
        const currentHostContext = getHostContext();
        // 创建fiber对应的dom节点
        const instance = createInstance(
            type,
            newProps,
            rootContainerInstance,
            currentHostContext,
            workInProgress,
          );
        // 将后代dom节点插入刚创建的dom里
        appendAllChildren(instance, workInProgress, false, false);
        // dom节点赋值给fiber.stateNode
        workInProgress.stateNode = instance;

        // 处理props和updateHostComponent类似
        if (
          finalizeInitialChildren(
            instance,
            type,
            newProps,
            rootContainerInstance,
            currentHostContext,
          )
        ) {
          markUpdate(workInProgress);
        }
     }
      return null;
    }
```

completeWork做了两件事:

（1）创建或者标记元素更新（beginWork的reconcileChildFibers方法用来标记fiberNode的插入、删除、移动，completeWork会标记更新操作）

（2）flags冒泡

当更新流程经过reconciler后会得到一颗wip Fiber Tree，其实部分FIberNode被标记flags，之后Renderer需要对被标记的FiberNode对应的DOM元素执行flags对应的DOM操作，但是问题是如果高效地找到这些散落在wip FIber Tree各处的被标记的FIberNode，就要用到flags冒泡

我们知道comoleteWork处于“归"的阶段，从叶子节点开始整体流程是自下而上的。FiberNode.subtreeFlags记录了该fiberNode的所有子孙fiberNode上被标记的flags。

```js
let subtreeFlags = NoFlags;

//收集子fiberNode的子孙fiberNode中标记的flags
subtreeFlags |= child.subtreeFlags
//收集子fiberNode标记的flags
subtreeFlags |= child.flags
//附加在当前fiberNode的subtreeFlags上
completeWork.subtreeFlags |= subtreeFlags
```

这样当HostRootFiber完成completeWork时，整棵wip Fiber Tree中所有被标记的flags都在HostRootFiber.subtreeFlags中定义

其实FIber架构的早期版本采用被称为Effects List的链表结构标记包含副作用的fiberNode，但是被subtreeFlags替代了



completeWork  mount具体流程

（1）根据wip.tag进入不同处理分支（下面以HostComponent为例）

（2）与beginWork相同，会根据current !== null 判断现在是mount还是update阶段

（3）先通过createInstance方法创建fiberNode对应的DOM元素

```js
function createInstance(type,props,rootContainerInstance,hostContext,internalInstanceHandle){
  //省略代码
  if(typeof props.children === 'string' || typeof props.children === 'number'){
    //省略children是string、number时的特殊处理
  }
  //创建DOM Element
  const domELement = createElement(type,props,rootContainerInstance,parentNamespace)
  //...省略
  return domELement
}
```

（4）执行appendAllChildren方法，将下一层DOM元素插入createInstance方法创建的DOM元素中，具体逻辑为：1.从当前fiberNode向下遍历，将遍历到的第一层DOM元素类型通过appendChild方法插入parent末尾 2.对兄弟fiberNode执行步骤1 3.如果没有兄弟fiberNode，则对父fiberNode的兄弟执行操作1 4.当遍历流程回到最初执行步骤1所在层或者parent所在层时终止

（5）执行finalizeInitialChildren方法完成属性的初始化

（6）执行bubbleProperties方法将flags冒泡



completeWork  update具体流程

（1）根据wip.tag进入不同处理分支（下面以HostComponent为例）

（2）与beginWork相同，会根据current !== null 判断现在是mount还是update阶段

（3）执行updateHostComponent，这个方法的主要逻辑在diffProperties方法

（4）执行diffProperties方法，这个方法包括两次遍历。第一次遍历标记删除“更新前有，更新后没有的属性”，第二次遍历标记更新“update流程前后发生改变”的属性。所有变化的属性会通过key、value的形式保存在updatePayload属性中，diffProperties方法最后会返回updatePayload并作为数组的相邻两项依次保存在fiberNode.updateQueue数组中，同时该fiberNode的flags会标记Update

# Renderer渲染器(commit阶段)

在render阶段的末尾会调用commitRoot(root)进入commit阶段，这里的root指的就是fiberRootNode。

Renderer工作的阶段被称为commit阶段，在这个阶段会将各种副作用（flags表示）commit（提交）到宿主环境UI中

render阶段可能被打断，而commit阶段一旦开始就会同步执行直到完成，整个过程可以分为三个子阶段：

BeforeMutation阶段、Mutation阶段、Layout阶段

在三个子阶段执行之前，需要判断本次更新是否涉及与三个子阶段相关的副作用。如果Wip HostRootFiber或其子孙存在副作用flags时，会进入三个子阶段，否则会跳过三个子阶段

commit阶段的三个子阶段会完成自下而上的subtreeFlags消费过程，具体来说每个子阶段的执行过程都遵循三段式

（1）commitXXXEffects

入口函数，finishedWork（就是Wip HostRootFiber）会作为firstChild参数传入，把firstChild赋值给全局变量nextEffect，执行commitXXXEffects_begin

（2）commitXXXEffects_begin

向下遍历直到第一个满足如下条件的fiberNode：

- 当前fiberNode的子fiberNode不包含该子阶段对应的flags，即当前fiberNode是包含该子阶段对应flags的层级最低的fiberNode
- 当前fiberNode不存在子fiberNode，即当前fiberNode是叶子元素

接下来，对目标fiberNode执行commitXXEffects_complete

（3）commitXXEffects_complete

执行flags对应操作的函数，包含三个步骤

- 对当前fiberNode执行flags对应的操作，即执行commitXXXEffectsOnFiber
- 如果当前fiberNode存在兄弟fiberNode，即对兄弟fiberNode执行commitXXXEffects_begin
- 如果不存在fiberNode，则对父fiberNode执行commitXXEffects_complete

综上所述，子阶段的遍历会以DFS的顺序，从HostRootFiber开始向下遍历到第一个满足条件的fiberNode，再从该fiberNode向上遍历直到HostRootFIber为止，在遍历的过程中会执行flags对应操作

## Effects List

FIber架构早期版本没有用subtreeFlags而是Effects List的链表结构保存被标记副作用的fiberNode

在completeWork的时候，如果fiberNode存在副作用，就会被插入到Effects List中，commit阶段的三个子阶段只需要遍历Effect List并对fiberNode执行flags对应操作

如果把FIber Tree比喻为圣诞树，那么Effect List就像圣诞树上的彩灯链

那么为什么React 18把Effect List替换为subtreeFlags？

这是因为虽然subtreeFlags遍历子树的操作要比Effect List遍历更多的节点，但是在React18中Suspense(React16就提供的功能，开启并发更新后行为有区别)的行为恰恰需要遍历子树

再具体一点来说，我们把未开启并发更新时的Suspense成为Legacy Suspense，开启并发更新时的Suspense成为Concurrent Suspense

首先Suspense的作用是等待子孙组件中异步的部分加载（比如被React.lazy包裹的组件）完毕后统一渲染，并在加载期间渲染fallback

但是如果子孙中除了异步加载的组件还有非异步加载的组件，我们假设叫Sibling。在开启并发更新前，虽然因为异步加载组件的存在导致Suspense渲染fallback，但是并不会阻止Sibling渲染，也不会组织Sibling中useEffect回调的执行，同时为了在UI上显示Sibling没有渲染，Sibling对应的DOM元素会被设置display:none，这其实是一种取巧的做法，和Suspense的理念并不完全一致。

所以Concurretn Suspensr针对Suspense内不显示的子树进行了单独的处理，几不会渲染设置display:none的内容，也不会执行useEffect回调。要实现这部分处理，需要改变commit阶段的遍历方式，即将Effect List重构为subtreeFlags

## BeforeMutation阶段

BeforeMutation阶段的工作主要发生在commitBeforeMutationEffects_complete中和commitBeforeMutationEffectsOnFiber方法中，整个过程主要处理两种类型的FIberNode

ClassComponent，执行getSnapshotBeforeUpdate方法

HostRoot，清空HostRoot挂载的内容，方便Mutation阶段渲染

## Mutation阶段

Mutation阶段的工作主要是进行DOM元素的增删改

**删除DOM元素**

在commitMutationEffects_begin向下遍历过程中会执行删除操作

整个删除操作是以DFS的顺序，遍历子树的每个fiberNode执行对应操作

会for循环遍历fiber.deletions数组一次执行commitDeletion方法执行删除操作，数组中的项是在render阶段的beginWork执行reconcile操作时，发现需要删除子fiberNode对应的DOM元素时，执行deleteChild方法添加的

**插入、移动DOM元素（placement）**

进入commitMutationEffects_complete方法后，其会对遍历到的每个fiberNode执行commitMutationEffectsOnFiber，该方法会执行具体的DOM操作，其实就是switch操作情况如插入、移动等分别进入不同逻辑比如placement flag对应commitPlacement方法

commitPlacement方法步骤：

（1）从当前fiberNode向上遍历，获取第一个类型为HostComponent、HostRoot、HostPortal三者之一的祖先fiberNode，其对应的DOM元素时执行DOM操作的目标元素的父级DOM元素

（2）获取用于执行parentNode.insertBefore(child,before)方法的before对应DOM元素（该DOM元素必须是稳定的即before对应的fiberNode不能被标记为placementflag，如果最终没有被找到，只能选择插入到父DOM元素的末尾）

（3）执行parentNode.insertBefore方法（存在before）或者parentNode.appendChild方法(不存在before)

**更新DOM元素**

执行DOM元素更新操作的方法是commitWork，因为所有变化属性的key、value会保存在fiberNode.updateQueue中，当finishedWork.updateQueue存在时，其最终会在updateDOMProperties方法中遍历并改变对应属性，处理如style属性变化、innerHTML、直接文本节点变化等变化

**Fiber Tree切换**

当Mutation阶段的主要工作完成进入Layout阶段前，会执行root.current = finishedWork进行Fiber Tree的切换

## Layout阶段

Layout阶段会在commitLayoutEffects_begin向下遍历过程中会执行OffscreenComponent的显/隐逻辑

进入commitLayoutEffects_complete方法后，其会对遍历到的每个fiberNode执行commitLayoutEffectOnFiber根据fiberNode.tag不同执行不同的操作，比如对于ClassComponent，在该阶段执行componentDidMount/Update方法。对于FC，在该阶段执行useLayoutEffect callback

同时对于ClassComponent，执行this.setState(newState,callback)和对于HostRoot执行ReactDOM.render(element,container,callback)中的callback，都会在这个commitLayoutEffectOnFiber中执行

# 状态更新

## 事件系统

**合成事件**

合成事件是对浏览器原生事件对象的封装，存在的目的是消除不同浏览器在事件对象间的差异，但是对于不支持某一事件的浏览器，合成事件并不会提供polyfill（因为这会显著增加ReactDOM的体积）

**实现传播机制**

利用事件委托的原理，React基于Fiber Tree实现了事件的“捕获、目标、冒泡”流程，并且加入了新特性比如不同事件对应不同优先级、定制事件名、定制事件行为。

事件传播机制的实现步骤：

（1）在根元素绑定“事件类型对应的事件回调”，所有子孙元素触发这类事件最终都会委托给“根元素的事件回调”处理

（2）寻找触发事件的DOM元素，找到其对应的fiberNode

（3）收集从当前fiberNode到HostRootFiber之间“所有注册的该事件的回调函数”，实现思路是从当前fiberNode向上遍历，直到HostRootFiber，收集遍历过程中fiberNode.memoizedProps属性内保存的对应事件回调，最后返回的是一个数组

（4）反向遍历并执行一遍收集的所有回调函数（模拟捕获阶段的实现）

（5）正向遍历并执行一遍收集的所有回调函数（模拟冒泡阶段的实现，如果不允许冒泡则停止遍历就可以）

## Update

可以用git来比喻Update，比如我们在逐步迭代需求ABC，突然遇到紧急线上bug为D。我们就暂存当前分支的修改，在master分支修复bug并紧急上线。bug修复上线后通过gir rebase与开发分支连接，开发分支基于修复bug的版本继续开发。

同理，高优先级update会中断正在进行中的低优先级update，待完成后低优先级update基于高优先级update计算出的state重新完成更新流程。

在react中触发状态更新的几种方式：

- ReactDOM.render(ReactDOM.createRoot)对应HostRoot
- this.setState对应ClassComponent
- this.forceUpdate对应ClassComponent
- useState对应FunctionComponent
- useReducer对应FunctionComponent

以上方法在fiberNode.tag中对应三种tag:HostRoot、ClassComponent、FunctionComponent。

存在两种不同数据结构的Update，ClassComponent和HostRoot共用一种，FunctionComponent单独用一种。

ClassComponent和HostRoot用的Update数据结构：

```js
function createUpdate(eventTime,lane){
  let update = {
    eventTime,
    lane,
    //区别触发更新的场景，UpdateState表示使用ReactDOM.createRoot或者this.setState触发更新
    //ReplaceUpdate表示在ClassComponent生命周期函数中改变this.state
    //CaptureUpdate表示发生错误的情况相爱在ClassComponent或者HostRoot中触发更新，比如getDerivedStateFromError
    //ForceUpdate表示通过this.forceUpdate触发更新
    tag:UpdateState,
    payload:null,
    //UI渲染后触发的回调函数
    callback:null,
    next:null
  }
}
```

FC用的Update数据结构：

```js
const update = {
  lane,
  action,
  //优化策略相关字段
  hasEagerState:false,
  eagerState:null,
  next:null
}
```



# 错误处理

React提供了两个与错误处理相关的API

- getDerivedStateFromError：静态方法，当错误发生时，提供一个机会渲染fallback UI
- componentDidCatch：组件实例方法，当错误发生时，提供一个机会记录错误信息

使用这两个API的ClassComponent通常被称为Error Boundaries（错误边界），在Error Boundaries的子孙组件中发生的所有React工作流程内(render阶段和commit阶段)的错误都会被Error Boundaries捕获

## 不会被Error Boundaries捕获的错误

根据官方文档，有四类错误不会被Error Boundaries捕获

（1）事件回调中的错误（如点击触发handleClick抛出的错误）

原因：事件回调不属于React工作流程(render阶段和commit阶段)

（2）异步代码（如setTimeOut，requestAnimationFrame回调）

原因：异步代码不属于React工作流程(render阶段和commit阶段)

（3）Server side rendering

原因：SSR不属于React工作流程(render阶段和commit阶段)

（4）在Error Boundaries所属的Component内发生的错误

原因：Error Boundaries只会捕获子孙组件发生的React工作流程内的错误

## Error Boundaries捕获React工作流程内错误流程

整体分为三个阶段：捕获错误、构造callback、执行callback

（1）捕获错误：

render阶段的错误会被捕获交给handleError(root,thrownValue)处理

commit阶段的错误会被捕获交给captureCommitPhaseError(fiber,fiber.return,error)处理

（2）构造callback

无论是handleError还是captureCommitPhaseError，都会从发生错误的fiberNode的父fiberNode开始，逐层向上遍历，寻找最近的Error Boundaries。一旦找到就会执行createClassErrorUpdate方法，构造两个callback，一个是用于执行Error Boundaries API的callback，一个是用于抛出React提示信息的callback。

如果没有找到Error Boundaries，则继续向上遍历直到HostRootFiber，并执行createRootErrorUpdate方法构造callback，在callbakc内抛出未捕获的错误

（3）执行callback

对于Error Boundaries，类似于触发了如下的更新

```js
this.setState(()=>{
  //用于执行getDerivedStateFromError的callback
},()=>{
  //用于执行componentDidCatch的callback
  //以及用于抛出React提示信息的callback
})
```

对于HostRoot，类似于执行了如下代码

```js
ReactDOM.render(element,container,()=>{
  //用于抛出未捕获的错误及React提示信息的callback
})
```

