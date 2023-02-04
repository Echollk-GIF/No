[TOC]

# Vue2->Vue3

如果有Vue2的基础，并在此基础上学习Vue3，并不需要把完整的官网看完，我们只需要关注一些新功能和非兼容的变化即可进行开发。



## Vue3变化

- 同一元素上使用的`v-if`和`v-for`优先级已更改，但**不推荐**同时使用 `v-if` 和 `v-for`。
- 组件事件需要在`emits`选项中声明
- `destroyed` 生命周期选项被重命名为 `unmounted`
- `beforeDestroy` 生命周期选项被重命名为 `beforeUnmount`
- 自定义指令的API已更改为与组件生命周期一致
- 新增了三个组件：`Fragment` 支持多个根节点、`Suspense` 可以在组件渲染之前的等待时间显示指定内容、`Teleport` 可以让子组件能够在视觉上跳出父组件(如父组件overflow:hidden)
- 新增指令 `v-memo`，可以缓存 html 模板，比如 v-for 列表不会变化的就缓存，简单说就是用内存换时间。
- 用 `Proxy` 代替 Object.defineProperty 重构了响应式系统，可以监听到数组下标变化，及对象新增属性，因为监听的不是对象属性，而是对象本身，还可拦截 apply、has 等13种方法
- 重构了虚拟 DOM，在编译时会将事件缓存、将 slot 编译为 lazy 函数、保存静态节点直接复用(静态提升)、以及添加静态标记、Diff 算法使用 最长递增子序列 优化了对比流程，使得虚拟 DOM 生成速度提升 `200%`
- 支持在 `<style></style>` 里使用 `v-bind`，给 CSS 绑定 JS 变量(`color: v-bind(str)`)
- 新增 `Composition API` 可以更好的逻辑复用和代码组织，同一功能的代码不至于像以前一样太分散，虽然 Vue2 中可以用 minxin 来实现复用代码，但也存在问题，比如方法或属性名会冲突，代码来源也不清楚等
- 全局函数 `set` 和 `delete` 以及实例方法 `$set` 和 `$delete`移除。基于代理的变化检测已经不再需要它们了
- 毕竟 Vue3 是用 `TS` 写的，所以对 `TS` 的支持度更好
- Vue3 不兼容 `IE11`
- `$on`，`$off` 和 `$once` 实例方法已被移除，组件实例不再实现事件触发接口。



## 组合式API

原有的组件选项(`data`、`computed`、`methods`、`watch`) 的方式来组织组件代码通常是非常有效的，但是也存在一些不好的地方，例如把原有的关联逻辑按照选项划分开来，掩盖了原有潜在的逻辑问题，这个时候我们就必须要不断地上下滚动代码来找到响应的代码块来查找，这样带来了极大的不便，特别是一开始没有编写过这组件的人来说，这导致组件难以阅读和理解。

所以针对上述的情况，Vue3提出了新的组织组件代码的方式---组合式API。组合式API需要一个可以实际使用的地方，那就是`setup`。

`setup`的触发时机是在组件创建之前执行的。需要注意，在`setup`中你应该避免使用`this`，因为这个时候`this`并不代表组件实例。`setup`的调用发生在`data`、`computed`和`methods`被解析之前，所以它们没法在`setup`中被获取。

当然，我们依然可以在Vue3使用选项（Option API）的方式来组织代码，这个和Vue2没有区别，但是不建议这样写。

**Vue3.x组件的选项（Option API）写法(不建议)**

```vue
<script>
	import { defineComponent, ref } from 'vue';
  export default defineComponent({
    setup() {},
    mounted() {
      console.log('生命周期mounted')
    },
    components: {},
    methods: {},
    watch: {},
    comuted: {}
  });
</script>
```

**Vue3.x组合式写法（推荐）**

```vue
<script>
	import { defineComponent, ref, onMounted } from 'vue';
  export default defineComponent({
    setup(){
      let num = ref(0);
      let fn = () => {};
      onMounted(():void => {
				console.log('生命周期mounted');
      });
      return {
        num,
        fn
      }
    }
  })
</script>
```





下面例子代码都是在`Typescript`的环境下进行的，所以需要`Typescript`基础。

### 1. 生命周期

通过在生命周期钩子前面加上 “on” 来访问组件的生命周期钩子。

下表包含如何在 Option API 和 setup() 内部调用生命周期钩子

|  Option API   |      setup      |
| :-----------: | :-------------: |
| beforeCreate  |        -        |
|    created    |        -        |
|  beforeMount  |  OnBeforeMount  |
|    mounted    |    onMounted    |
| beforeUpdate  | onBeforeUpdate  |
|    updated    |    onUpdated    |
| beforeUnmount | onBeforeUnmount |
|   unmounted   |   onUnmounted   |
|   activated   |   onActivated   |
|  deactivated  |  onDeactivated  |

>因为 `setup` 是在 `beforeCreate` 和 `created` 生命周期钩子前运行的，所以不需要显式地定义它们。换句话说，在这些钩子中编写的任何代码都应该直接在 `setup` 函数中编写。

```vue
<template>
	<div id="div">123</div>
</template>

<script lang="ts">
import { defineComponent, onMounted } from "vue";
export default defineComponent({
  setup() {
    console.log("setup");
    onMounted((): void => {
      console.log("onMounted");
      console.log(document.getElementById('div')?.innerHTML);
    });
  },
});
</script>
```



### 2. ref、reactive、toRefs响应式和methods

Vue2.x默认写在data的值，初始化的时候内部会完成值的数据的响应式（`get`、`set`绑定），但是Vue3要手动调用内置方法实现，那么接下来看看常用的实现数据响应式方法都有哪些。

> ref不仅可以用在数据的响应式，还可以绑定`DOM`元素

```vue
<template>	
	<div>
		<div>{{ num }}</div>
    <button @click="add1">num++</button>
    <p>-------------------------------</p>
    <div>{{ state.count }}</div>
    <button @click="add2">state.count++</button>
    <p>-------------------------------</p>
    <div>{{ count }}</div>
    <button @click="add3">count++</button>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, toRefs } from "vue";
export default defineComponent({
  setup() {
    interface ObjItf {
      count: number;
    }

    // ref声明响应式数据，用于声明基本数据类型
    let num = ref<number>(1);

    let obj = {
      count: 1,
    };

    // reactive声明响应式数据，用于声明引用数据类型
    let state = reactive<ObjItf>(obj);

    // toRefs解构响应式数据
    let { count } = toRefs<ObjItf>(state);

    const add1 = (): void => {
      num.value++; // 注意通过ref声明的变量，所以js要修改对应的值是要通过.value访问才可以，template模板不需要通过.value访问
    };

    const add2 = (): void => {
      state.count++; // 通过reactive声明的遍历，不需要通过.value访问值
    };

    const add3 = (): void => {
      count.value++; // 通过toRefs结构的值和ref声明的变量一样，需要通过.value访问其值
    };

    return {
      num,
      state,
      count,
      add1,
      add2,
      add3
    };
  },
});
</script>
```

**注意：**

1. reactive可以传递基础数据类型和引用数据类型，基础数据类型不会被包装成响应式数据
2. reactive返回的响应式数据本质是Proxy对象，对象里面每一层都会被包装成Proxy对象
3. reactive返回的响应式数据和原始的数据会相互影响
4. ref可以传递基础数据类型和引用数据类型。如果是基础数据类型，那么这个基础数据值保存在返回的响应式数据的`.value`上；如果是对象，响应式数据在`.value`上。
5. ref本质是将一个数据变成一个对象，这个对象具有响应式特点

**为什么需要toRefs和toRef？**

和`ref`不一样的是，`toRef`和`toRefs`整两个方法，它们不创造响应式，而是延续响应式。创造响应式一般由ref和reactive来解决，而toRef和toRefs则把对象的数据进行分解和扩散，其这个对象针对的是**响应式对象**(`reactive`)而非普通对象。

### 3. watch

语法：`watch(监听源|[多个], (val, oldVal) => {}, {immediate?: false, deep: false})`

watch写法上支持一个或者多个监听源，这些监听源必须只能是`getter/effect`函数，ref数据，reactive对象，或者是数组类型

```vue
<template>
  <div>
  	<div>{{ count }}</div>
    <button @click="add">count++</button>
  </div>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs, watch } from "vue";
export default defineComponent({
  setup() {
    interface ObjItf {
      count: number;
    }

    let obj = {
      count: 1,
    };

    // reactive声明响应式数据，用于声明引用数据类型
    let state = reactive<ObjItf>(obj);

    // toRefs解构响应式数据
    let { count } = toRefs<ObjItf>(state);

    const add = (): void => {
      count.value++;
    };

    watch(count, (newVal: number, oldVal: number): void => {
      console.log(newVal, oldVal);
    });

    return {
      count,
      add,
    };
  },
});
</script>
```



### 4. watchEffect

它立即执行传入的一个函数，同时**响应式追踪其依赖**，并在其依赖变更时重新运行该函数。

```vue
<script>
	let num = ref(0);
  watchEffect(() => {
    console.log(num.value);
  });
  setTimeout(() => {
    num.value++;
  }, 1000);
</script>
```

**watch 和 watchEffect的区别**

- 两者都可以监听 `data` 属性变化；
- `watch` 需要**明确监听哪个属性**；
- 而 `watchEffect` 会根据其中的属性，**自动监听**其变化。

### 5. computed

```vue
<template>
	<div>{{ count }}</div>
  <button @click="state.count++">state.count++</button>
</template>
<script lang="ts">
import { defineComponent, reactive, computed } from "vue";
export default defineComponent({
  setup() {
    interface ObjItf {
      count: number;
    }

    let obj = {
      count: 1,
      arr: [1, 2, 3]
    };

    const state = reactive<ObjItf>(obj);

    let count = computed((): number => {
      return state.count;
    });

    return {
      count,
      state
    };
  },
});
</script>
```



### 6. 组件

**全局组件：**

```ts
const app = Vue.createApp({...})

app.component('my-component-name', {
  /* ... */
})
```



**局部组件：**

> 子组件

```vue
<template>
  <div>{{ aname }}</div>
</template>


<script lang="ts">
import { defineComponent, toRefs } from "vue";
export default defineComponent({
  props: {
    aname: {
      type: String,
      default: "张三",
    },
  },
  emits: ["uname"],
  setup(props, { emit }) {
    let { aname } = toRefs(props);
    const updateName = () => {
      emit('uname', 'a改变后的名字')
    };
    return {
      aname,
      updateName,
    };
  },
});
</script>
```



> 父组件

```vue
<template>
	<mychild :aname="aname" @uname="updateName"></mychild>
</template>

<script lang="ts">
import { defineComponent, toRefs, ref } from "vue";
import mychild from './mychild.vue';
  export default defineComponent({
    components: {
      mychild
    },
    setup() {
      let aname = ref('李四');
      const updateName = (p: string) => {
        aname.value = p;
      }
      return {
        aname,
        updateName
      }
    }
  })
</script>
```



**setup语法糖组件**

> 子组件

```vue
<!--  -->
<template>
  <div>{{ aname }}</div>
  <button @click="updateName">修改姓名</button>
</template>

<script lang="ts" setup>
// setup语法糖下defineProps,defineEmits 不需要引入
let emit = defineEmits(["updatename"]);
let props = defineProps({
  aname: {
    type: String,
    default: "李四",
  },
});
const updateName = (): void => {
    emit('updatename', '修改后的名字')
};
</script>

<style lang="less" scoped></style>

```

> 父组件

```vue
<!--  -->
<template>
  <div></div>
  <mychild :aname="aname" @updatename="updateName"></mychild>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import mychild from "./b.vue";
let aname = ref("张三");
let updateName = (name: string): void => {
  aname.value = name;
};
</script>

<style lang="less" scoped></style>

```





### 7. v-model

> 子组件

```vue
<!--  -->
<template>
  <div>{{ name }}{{ age }}</div>
  <button @click="updateName">修改姓名</button>
</template>

<script lang="ts" setup>
let props = defineProps({
  age: Number,
  name: String,
});
let emits = defineEmits(["update:name", "update:age"]);
let updateName = () => {
  emits("update:age", 30);
  emits("update:name", "李四");
};
</script>

<style lang="less" scoped></style>

```



> 父组件

```vue
<!--  -->
<template>
  <div></div>
  <mychild v-model:name="state.name" v-model:age="state.age"></mychild>
</template>

<script lang="ts" setup>
import { reactive } from "vue";
import mychild from "./b.vue";
let info = {
  name: '张三',
  age: 20
}
let state = reactive(info);

</script>

<style lang="less" scoped></style>
```



### 8. 插槽

> 子组件

```vue
<!--  -->
<template>
  <slot></slot>
  <slot name="title"></slot>
  <slot name="footer" :user="state.user" :d="state.d"></slot>
</template>

<script lang="ts" setup>
import { reactive } from "vue";
let state = reactive({
  user: { a: 1, b: 2 },
  d: 1,
});
</script>

<style lang="less" scoped></style>

```

> 父组件

```vue
<!--  -->
<template>
  <div></div>
  <mychild>
    <div>匿名插槽</div>
    <template #title>
      <div>具名插槽</div>
    </template>
    <template #footer="scope">
      <div>作用域插槽{{ scope.user }} {{ scope.d }}</div>
    </template>
  </mychild>
</template>

<script lang="ts" setup>
import mychild from "./child.vue";
</script>

<style lang="less" scoped></style>

```

### 9. await支持

不必再配合 async 就可以直接使用 await 了，这种情况下，组件的 setup 会自动变成 async setup 。

```vue
<script setup>
  const post = await fetch('/api').then(() => {})
</script>
```



### 10. style scoped

```vue
<style scoped>
/* 修改第三方组件样式 */
::v-deep(.foo) {}
/* 简写 */
:deep(.foo) {}

/* 修改插槽内容样式 */
::v-slotted(.foo) {}
/* 简写 */
:slotted(.foo) {}

/* 修改全局样式 */
::v-global(.foo) {}
/* 简写 */
:global(.foo) {}
</style>
```



### 11. teleport

这个组件可以把组件进行传送，to属性就是要传送的位置目标，用css选择器

```vue
<template>
  <div class="header"></div>
  <teleport :to="target">
    <p>哈哈哈哈哈</p>
  </teleport>
  <button @click="changeTarget">点击切换</button>
  <p>-------------------------------</p>
  <div class="footer"></div>
</template>

<script lang="ts" setup>
import { ref } from "vue";
let target = ref<string>(".header");
const changeTarget = (): void => {
  target.value = ".footer";
};
</script>

<style lang="less" scoped></style>

```

