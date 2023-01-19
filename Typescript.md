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
  * { name: string, age: number }

* symbol
* null/undefined



## 踩坑

### ERROR in node_modules/@types/node/globals.global.d.ts(1,44): error TS2304: Cannot find name ‘globalT

这个错误一般是项目ts和系统全局安装的ts版本不一致，重新安装ys即可