[TOC]

# TypeScript



## JavaScript的缺陷

* 类型缺失: 对于标识符是没有任何的类型校验, 有安全隐患.

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