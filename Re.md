[TOC]

# 问题

为什么要把Effects List替换为subtreeFlags

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

### Fiber Tree的构建

#### mount

有两种情况：整个应用的首次渲染、某个组件的首次渲染（没有下述1、2两步）

（1）创建FiberRootNode

（2）创建tag为3的FiberNode，代表hostRoot，称为HostRootFiber（HostRoot代表应用在宿主环境挂载的根节点，FiberRootNode通过current指向HostRootFiber，HostRootFiber通过stateNode指向FiberRootNode）

（3）从HostRootFiber开始，以DFS深度优先搜索的顺序生成FIberNode构造workInProgress FIber Tree，并在遍历过程中，为FIberNode标记代表不同副作用的flags，以便后续再Renderer中使用

 （4）把workInProgress Fiber Tree切换成current Fiber Tree

#### Update

（1）根据current Fiber创建workInProgress Fiber

（2）把workInProgress Fiber Tree切换成current Fiber Tree

# Reconciler（render阶段）

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

# Renderer(commit阶段)

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

## 错误处理

React提供了两个与错误处理相关的API

getDerivedStateFromError：静态方法，当错误发生时，提供一个机会渲染fallback UI

componentDidCatch：组件实例方法，当错误发生时，提供一个机会记录错误信息