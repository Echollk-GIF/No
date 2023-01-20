[TOC]

# TypeScript

## JavaScript的缺陷

* 类型缺失: 对于标识符是没有任何的类型校验, 有安全隐患.

## 定义变量的方式

* 类型注解

```tsx
let message:string = 'hello world';
```

* 类型推导

声明一个标识符时，如果有直接进行赋值，会根据赋值的类型推导出标识符的类型注释，这个过程称之为类型推导

使用let进行类型推导，推导出来的是通用类型

使用const进行类型推导，推导出来的是字面量类型

## JavaScript中的数据类型

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

## TypeScript特有的类型

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
* tuple类型
  * 介于数组和对象之间类型
  * useState封装

## 踩坑

### ERROR in node_modules/@types/node/globals.global.d.ts(1,44): error TS2304: Cannot find name ‘globalT

这个错误一般是项目ts和系统全局安装的ts版本不一致，重新安装ys即可