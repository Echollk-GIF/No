[TOC]

# TypeScript

## 邂逅TypeScript

### JavaScript的缺陷

* 类型缺失: 对于标识符是没有任何的类型校验, 有安全隐患.

### 定义变量的方式

* 类型注解

```tsx
let message:string = 'hello world';
```

* 类型推导

声明一个标识符时，如果有直接进行赋值，会根据赋值的类型推导出标识符的类型注释，这个过程称之为类型推导

使用let进行类型推导，推导出来的是通用类型

使用const进行类型推导，推导出来的是字面量类型

### JavaScript中的数据类型

* number/Number

* string/String

* boolean

* 数组类型

  * any[]
  * Array<any>

* 对象类型

  * object

  ```tsx
  //不要这么做，此时我们既不能获取数据也不能设置数据
  const myInfo: object = {
    name: 'name',
  };
  console.log(myInfo.name);
  //类型object上不存在属性name
  export {};
  ```

  * { name: string, age: number,grade?:number }

* symbol

* null/undefined

- function

在定义一个Ts中的函数时，都要明确的指定参数的类型。返回值类型可以明确地指定也可以通过类型推导自行指定

大多数作为参数传递给另一个函数的匿名函数最好不要加类型注解

### TypeScript特有的类型

* any
* unknown

unknown和any类型有点类似，但是unknown类型的值上做任何事情都是不合法的

如果想对unknown类型进行操作，必须先进行类型校验（比如if typeof判断进行类型缩小，再根据类型进行相应的操作）

* void
  * 定义函数的类型时, 会使用

我们可以将undefined赋值给void类型，也就是函数可以返回undefined

当基于上下文的类型推导推导出返回类型为void的时候，并不会强制函数一定不能返回内容

* never类型
  * 自动推导
  * 封装框架/工具: 校验
  * 类型工具: never(了解)

如果函数陷入死循环或者抛出异常，就可以使用never。实际开发中很少使用

* tuple元组类型
  * 介于数组和对象之间类型
  * useState封装

##  TypeScript语法细节

### 类型别名 type和接口类型 interface

* 定义非对象类型时, 肯定使用type
* 定义对象类型的时候, 都可以
  * interface更加强大, 扩展性更强（比如可以多次声明同一个接口名称、支持extends继承、interface可以被类实现）
  * 推荐使用interface

### 联合类型 |

* string | number

### 交叉类型 &

```tsx
interface IKun {
  name:string
}
interface ICoder {
  code:()=>void
}
const info: Ikun & ICoder = {
  name:'tom',
  code:()=>{}
}
```

### 类型断言 as

- Element as HTMLImageElement

### 非空类型断言 

- friend!.name = ""

### 字面量类型

字面量类型经常和联合类型一起使用

```tsx
type Direction = "left"|"right"
```

字面量推理as const

### 类型缩小narrowing

*  typeof
*  平等 ===/!==
*  instanceof
*  in
*  等等

## TypeScript函数类型

### 函数类型表示方式

* 函数类型表达式: () => void、(num:number)=>number
* 函数调用签名: interface { 其他属性; (): void } /{(num:number):number}
* 函数构造签名: interface { 其他属性; new (): void }

### 函数的参数细节

* 可选参数: 类型 | undefined
* 参数默认值: x = 100
* 剩余参数: ...args: number[]

### 函数的重载使用（了解）

* 重载签名
* 重载实现(通用函数)
  * 不能被调用
* 和联合类型的选择:
  * 能使用联合类型尽量使用联合类型

### this的绑定问题

* 默认this是any类型
* 开发*noImplicitThis设置为true*, this在上下文不能正确推导的情况下, 必须明确的指定
  * 作为第一个参数, 并且名字必须加this
  * 后续传入的参数是从第二个开始, 编译出来的代码, this类型会被抹除

### this相关内置工具

* ThisParameterType
* OmitThisParameter
* ThisType

## TypeScript面向对象

### 类中成员的修饰符

* public
* private
* protected

### 枚举类型和值的类型

```tsx
enum Direction{
  LEFT,
  RIGHT
}
function turn(direction:Direction){
  switch(direction){
    case Direction.LEFT:
      console.log('LEFT')
 }
}
```

## 泛型

### 泛型约束

* extends

```tsx
interface ILength {
  length:number
}
function getInfo<Type extends ILength>(args:Type):Type{
  return args
}
//这里表示是传入的类型必须有这个属性，也可以有其他属性，但是必须至少有这个成员
```

* keyof 类型 => key的联合类型

```tsx
//我们希望获取一个对象给定属性名的值,我们需要确保我们不会获取 obj 上不存在的属性
function getObjectProperty<Type,Key extends keyof Type>(obj:Type,key:Key){
  return obj[key]
}
```

### 映射类型

有的时候，一个类型需要基于另外一个类型，但是你又不想拷贝一份，这个时候可以考虑使用映射类型

```tsx
interface IPerson {
  name:string
  age:number
}
type MapType<Type>{
	[property in keyof Type]:Type[property]
}

type NewPerson = MapType<IPerson>
```

**在使用映射类型时，有两个额外的修饰符可能会用到：**

- 一个是 readonly，用于设置属性只读；
- 一个是 ? ，用于设置属性可选；

可以通过前缀 - 或者 + 删除或者添加这些修饰符，如果没有写前缀，相当于使用了 + 前缀

```tsx
interface IPerson {
  name:string
  age:number
}
type MapType<Type>{
	-readonly [property in keyof Type]-?:Type[property]
}

type NewPerson = MapType<IPerson>
```

## 类型工具

### 条件类型

日常开发中我们需要基于输入的值来决定输出的值，同样我们也需要基于输入的值的类型来决定输出的值的类型

```tsx
function sum<T extends number | string>(arg1:T,arg2:T):T extends string ? string : number
function sum(arg1:any,arg2:any)[
  return arg1+arg2
]
```

### 在条件类型中推断（infer）

条件类型提供了 infer 关键词，可以从正在比较的类型中推断类型，然后在 true 分支里引用该推断结果

```tsx
//比如我们现在有一个数组类型，想要获取到一个函数的参数类型和返回值类型
type CalcFnType = (num1:number,num2:number)=>number

type MyReturnType<T extends(...args:any[])=>any> = T extends (...args:any[]) => infer R ? R :never
```

### 分发条件类型

 当在泛型中使用条件类型的时候，如果传入一个联合类型，就会变成分发的

```tsx
type toArray<Type> = Type extends any ? Type[] :never

//string[] | number[]
type newType = toArray<number|string>
```

**如果我们在 ToArray 传入一个联合类型，这个条件类型会被应用到联合类型的每个成员：**

当传入string | number时，会遍历联合类型中的每一个成员；

相当于ToArray<string> | ToArray<number>； 

所以最后的结果是：string[] | number[]；

## 内置工具

### Partial<Type>

用于构造一个Type下面的所有属性都设置为可选的类型

```tsx
type MyPartial<T> = {
  [P in keyof T]?:T[P]
}
```

### Required<Type>

用于构造一个Type下面的所有属性全都设置为必填的类型，这个工具类型跟 Partial 相反。

```tsx
type MyRequired<T> = {
  [P in keyof T]-? :T[P]
}
```

### Readonly<Type>

用于构造一个Type下面的所有属性全都设置为只读的类型，意味着这个类型的所有的属性全都不可以重新赋值。

```tsx
type MyPartial<T> = {
  readonly [P in keyof T]:T[P]
}
```

### Record<Keys, Type>

用于构造一个对象类型，它所有的key(键)都是Keys类型，它所有的value(值)都是Type类型。

```tsx
type MyRecord<K extends key of any,T> = {
  [P in K] : T
}
```

### Pick<Type, Keys>

用于构造一个类型，它是从Type类型里面挑了一些属性Keys

```tsx
type MyPick<T,K extends keyof T> = {
  [p in K]: T[P]
}
interface IPerson{
  name:string
  age:number
  height:number
}
type IKun = Pick<IPerson,"name"| "age">
```

### Omit<Type, Keys>

用于构造一个类型，它是从Type类型里面过滤掉了一些属性Keys

```tsx
type MyOmit<T,K> = {
  [P in keyof T as P extends K ? never : P]:T[P]
}
interface IPerson{
  name:string
  age:number
  height:number
}
type IKun = Omit<IPerson,"name"| "age">
```

### Exclude<UnionType, ExcludedMembers>

用于构造一个类型，它是从UnionType联合类型里面排除了所有可以赋给ExcludedMembers的类型。

```tsx
type MyExclude<T,U> = T extends U ? never : T

type PropertyTypes = "name"|"age"|"height"
type PropertyTypes2 = MyExclude<PropertyTypes,"height">
```

### Extract<Type, Union>

用于构造一个类型，它是从Type类型里面提取了所有可以赋给Union的类型。

```tsx
type MyExtract<T,U> = T extends U ? T :never

type PropertyTypes = "name"|"age"|"height"
type PropertyTypes2 = MyExtract<PropertyTypes,"height">
```

### NonNullable<Type>

用于构造一个类型，这个类型从Type中排除了所有的null、undefined的类型。

```tsx
type IKun = "sing" | "dance"
type MyNonNullable<T> = T extends null | undefined ? never:T
type Ikuns = MyNonNullable<IKun>
```

### ReturnType<Type>

```tsx
type MyReturnType<T extends(...args:any[])=>any> = T extends (...args:any[]) => infer R ? R :never
```

### InstanceType<Type>

用于构造一个由所有Type的构造函数的实例类型组成的类型。

## 踩坑

### ERROR in node_modules/@types/node/globals.global.d.ts(1,44): error TS2304: Cannot find name ‘globalT

这个错误一般是项目ts和系统全局安装的ts版本不一致，重新安装ts即可