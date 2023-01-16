```
json-server --watch db.json --port 8888
```

[TOC]



## json-server简介

使用 `json-server` 需要遵守一定的规范。

- 数据查询要使用 `GET`。
- 新增数据要使用 `POST`。
- 删除数据要使用 `DELETE`。
- 修改数据使用 `PUT` 和 `PATCH`。

## 30秒起步

**1. node 环境安装**

`node` 常见的安装方法有2种。第一种是官方下载，第二种是使用 `nvm` 下载。

**2. 安装 json-server**

全局安装方式：

```vbscript
npm install -g json-server
```

**3. 创建数据库**

在本机创建一个文件夹，然后新建一个 `json` 文件，再填入数据即可。建议文件名不要出现中文。

例如：

创建 `json-server` 文件夹，在 `json-server 里创建 `db.json 文件

`db.json` 文件录入以下数据

```json
{
  "posts": [
    {
      "id": 1,
      "title": "json-server",
      "author": "typicode"
    }
  ],
  "comments": [
    {
      "id": 1,
      "body": "some comment",
      "postId": 1
    }
  ],
  "profile": {
    "name": "typicode"
  }
}
```

**4. 启动服务**

进入 `json-server` 目录，打开终端输入以下命令即可

```css
json-server --watch db.json
```

## 查（get）

### 普通查询

> 查询 `/posts` 所有数据

```arduino
http://localhost:3000/posts
```

### id查询

```
http://localhost:3000/posts/{id}
```

### 条件查询

#### 单一条件查询

> 查询 `author` 是 `张三` 的数据

```bash
http://localhost:3000/posts?author=张三
```

在 `http://localhost:3000/posts` 后面加一个 `?` ，然后写上筛选条件即可。

#### 多条件查询（且）

> 这次要筛选的是 `author = 张三` 且 `title = 文章333` 的数据

```bash
http://localhost:3000/posts?author=张三&title=文章333
```

#### 多条件查询（或）

> 这次要筛选的是 `title = 222` 和 `title = 333` 这两条数据出来。

```bash
http://localhost:3000/posts?title=文章222&title=文章333
```

重复使用 `title` ，会把符合条件的都筛查出来。

#### 深度属性查询

数据内容如下

```json
{
  "posts": [
    {
      "id": 1,
      "title": "文章111",
      "authorInfo": {
        "name": "张三",
        "age": 20
      }
    },
    {
      "id": 2,
      "title": "文章222",
      "authorInfo": {
        "name": "李四",
        "age": 24
      }
    }
  ]
}
```

可以看到 `authorInfo` 里面还有子属性。

> 查询 `authorInfo.name = 张三` 的数据。

```bash
http://localhost:3000/posts?authorInfo.name=张三
```

### 分页查询

使用 `_page` 和 `_limit`(可选) 对数据进行分页。需要注意，`_page` 和 `_limit` 前面都要有**下划线**。

- `_page`：页码
- `_limit`：每页的数据量

修改 `db.json` 里的数据方便测试分页功能，如下所示

```json
{
  "comments": [
    { "id": 1, "body": "some comment 1", "postId": 1 },
    { "id": 2, "body": "some comment 2", "postId": 1 },
    { "id": 3, "body": "some comment 3", "postId": 2 },
    { "id": 4, "body": "some comment 4", "postId": 3 },
    { "id": 5, "body": "some comment 5", "postId": 1 },
    { "id": 6, "body": "some comment 6", "postId": 3 },
    { "id": 7, "body": "some comment 7", "postId": 3 },
    { "id": 8, "body": "some comment 8", "postId": 1 },
    { "id": 9, "body": "some comment 9", "postId": 2 },
    { "id": 10, "body": "some comment 10", "postId": 2 },
    { "id": 11, "body": "some comment 11", "postId": 3 },
    { "id": 12, "body": "some comment 11", "postId": 1 }
  ]
}
```

准备了12条数据。

> 需要获取第2页的数据，每页3条：

```bash
http://localhost:3000/comments?_page=2&_limit=3
```

### 排序查询

需要添加 **排序的标记 ( `_sort` )**，然后设置 **排序规则 ( `_order` )**。

其中，排序规则有 **升序 ( `asc` )** 和 **降序 ( `desc` )** 。

```ini
http://localhost:3000/{接口名}?_sort=要排序的字段名&_order=排序规则
```

以这份数据为例：

```json
{
  "comments": [
    { "id": 11, "body": "some comment 1", "postId": 1 },
    { "id": 2, "body": "some comment 2", "postId": 1 },
    { "id": 3, "body": "some comment 3", "postId": 2 },
    { "id": 10, "body": "some comment 4", "postId": 3 },
    { "id": 7, "body": "some comment 5", "postId": 1 },
    { "id": 6, "body": "some comment 6", "postId": 3 },
    { "id": 5, "body": "some comment 7", "postId": 3 },
    { "id": 8, "body": "some comment 8", "postId": 1 },
    { "id": 9, "body": "some comment 9", "postId": 2 },
    { "id": 4, "body": "some comment 10", "postId": 2 },
    { "id": 1, "body": "some comment 11", "postId": 3 },
    { "id": 12, "body": "some comment 11", "postId": 1 }
  ]
}
```

`id` 的排序是乱的，如果使用普通的方式请求回来的数据是原模原样返回的。

#### 升序

> 以 `id` 为参考字段进行升序排列返回给客户端。

```bash
http://localhost:3000/comments?_sort=id

或
http://localhost:3000/comments?_sort=id&_order=asc
```

返回的结果就会以 `id` 为参考字段升序排好。

普通升序排列的话，`_order=asc` 可以不传。只需指定 **参考字段 ( `_sort` )** 即可。

#### 降序

> 降序必须填好 `_order=desc` 。

```bash
http://localhost:3000/comments?_sort=id&_order=desc
```

#### 多字段排序

这次的需求是：

1. 首先按 `postId` 升序排列
2. 在 `1` 的基础上再对 `id` 进行倒序排列

> 多个字段用 `,` 分格。

```bash
http://localhost:3000/comments?_sort=postId,id&_order=asc,desc
```

### 切片查询

切片的意思是指定 **头** 和 **尾** ；也可以指定 **头** 和 **片段长度** 。

用到的关键字有：

- `_start`：开始位置（下标，从0开始）
- `_end`：结束位置
- `_limit`：片段长度

**总数** 会放在 `headers` 里。

以这份数据为例

```json
{
  "comments": [
    { "id": 1, "body": "some comment 1", "postId": 1 },
    { "id": 2, "body": "some comment 2", "postId": 1 },
    { "id": 3, "body": "some comment 3", "postId": 2 },
    { "id": 4, "body": "some comment 4", "postId": 3 },
    { "id": 5, "body": "some comment 5", "postId": 1 },
    { "id": 6, "body": "some comment 6", "postId": 3 },
    { "id": 7, "body": "some comment 7", "postId": 3 },
    { "id": 8, "body": "some comment 8", "postId": 1 },
    { "id": 9, "body": "some comment 9", "postId": 2 },
    { "id": 10, "body": "some comment 10", "postId": 2 },
    { "id": 11, "body": "some comment 11", "postId": 3 },
    { "id": 12, "body": "some comment 11", "postId": 1 }
  ]
}
```

> 需求：返回下标从 `2-6` 的数据

使用 `_start` 和 `_end` 的方式

```bash
http://localhost:3000/comments?_start=2&_end=6
```

使用 `_start` 和 `_limit` 的方式

```bash
http://localhost:3000/comments?_start=2&_limit=4
```

### 范围查询

范围查询包括 **大于等于**、**小于等于**、**不等于** 三种情况。



#### 大于等于 _gte

**大于等于** 使用的关键字是 `_gte` 。注意，前面有个下划线的。

> 需求：查询 `comments` 接口 `id` 大于等于 `4` 的数据

```bash
http://localhost:3000/comments?id_gte=4
```

#### 小于等于 _lte

> 需求：查询 `comments` 接口 `id` 小于等于 `4` 的数据

```bash
http://localhost:3000/comments?id_lte=4
```

#### 联合一起使用

> 需求：查询 `comments` 接口 `id` 大于等于 `4` 且 小于等于 `6` 的数据

```bash
http://localhost:3000/comments?id_gte=4&id_lte=6
```

#### 不等于 _ne

> 需求：查询 `comments` 接口 `id` 不等于 `2` 的数据

```bash
http://localhost:3000/comments?id_ne=2
```

### 模糊查询

模糊查询的关键字是 `_like`。

> 需求：查询 `comments` 接口 `body` 包含 `1` 的数据

### 全文查询

全文查询的关键字是 `q`

准备以下数据比较好演示

```json
{
  "authors": [
    { "id": 1, "name": "zhangsan", "age": 18},
    { "id": 2, "name": "lisi", "age": 21},
    { "id": 3, "name": "wangwu", "age": 24}
  ]
}
```



> 查询所有字段中包含 `2` 的数据出来

```bash
http://localhost:3000/authors?q=2
```

### 外键关联查询

外键查询需要 **2个接口** 关联查询。

准备以下数据方便演示

```json
{
  "posts": [
    { "id": 1, "title": "文章111", "author": "张三" },
    { "id": 2, "title": "文章222", "author": "李四" }
  ],
  "comments": [
    { "id": 1, "body": "some comment 1", "postId": 1 },
    { "id": 2, "body": "some comment 2", "postId": 1 },
    { "id": 3, "body": "some comment 3", "postId": 2 }
  ]
}
```

`posts` 里有2条数据。

`comments` 里有3条数据，其中每条数据都有一个 `postId`，是对应 `posts` 每条数据的 `id`。



> 需求：查询 `posts` 里 `id` 为 `1` 的所有 `comments` 内容

```bash
http://localhost:3000/posts/1/comments
```

### 关系拼装

关系拼装可以把关联的2个接口的数据拼接起来并返回。

其中有2种查询关系：

- 包含子资源 `_embed`
- 包含父资源 `_expand`



准备以下数据方便演示

```json
{
  "posts": [
    { "id": 1, "title": "文章111", "author": "张三" },
    { "id": 2, "title": "文章222", "author": "李四" }
  ],
  "comments": [
    { "id": 1, "body": "some comment 1", "postId": 1 },
    { "id": 2, "body": "some comment 2", "postId": 1 },
    { "id": 3, "body": "some comment 3", "postId": 2 }
  ]
}
```



#### 包含子资源 _embed

```bash
http://localhost:3000/posts?_embed=comments
```

还可以拼接多个条件。

> 需求：在 `comments` 里，把 `posts` 里 `id` 为 `2` 的数据找出来并拼接起来

```bash
http://localhost:3000/posts/2?_embed=comments
```



#### 包含父资源 _expand

```bash
http://localhost:3000/comments?_expand=post
```

## 增（post）

`json-server` **新增数据**需要使用 `POST` 方法。

```
http://localhost:3000/posts
```

POST->Body->raw->JSON

这里使用 `Post` 方法向 `/posts` 接口传输数据，`/posts` 原本的数据结构是包含 `id、title、author` 三个字段，`id` 默认是自增主键，不传的话会默认增加。

## 删（delete）

`json-server` **删除数据**需要使用 `DELETE` 方法。

删除的公式是：

```bash
http://localhost:3000/{接口名}/{id}
```

例如：

```
http://localhost:3000/posts/2
```

## 改（put 和 patch）

修改数据分为2个方法：

- `put` ：覆盖
- `patch` ：更新

公式如下所示：

```bash
http://localhost:3000/posts/{id}
```

**覆盖（put）**

> 例：把 `id` 为 `1` 的数据改成 `{ "title": "leihou", "author": "雷猴" }`
>
> ```json
> http://localhost:3000/posts/1
> {
> "title": "leihou",
> "author": "雷猴"
> }
> ```
>
> PUT->Body->raw->JSON



注意：原本的数据包含 `title` 和 `author` ，使用 `put` 时必须把这两个字段都写上，不然会删掉没传的字段。这就是 **“覆盖”** 的意思。

**更新（patch）**

> 例：使用 `patch` 方法把 `id` 为 `1` 的数据 `title` 字段的值更改成 `hello` 。
>
> PATCH->Body->raw->JSON

```json
{
  "title":"hello"
}
//这里只写title就可以
```



### 配置路由

有时候我们的 `api地址` 可能不像上面所有案例中那么简单，此时就可以使用 **自定义路由** 的方法来模拟。

比如模拟下面这个接口：

```bash
http://localhost:3000/api/users/1
```



实现的步骤如下所示：

1. 创建 `routes.json` 文件（也可以不叫这个名字）
2. 启动服务时使用 `--routes` 参数



1、创建 `routes.json` ，并输入以下内容。

```json
{
  "/api/*": "/$1"
}
```

2、启动服务

```css
json-server db.json --routes routes.json
```

3、访问

```bash
http://localhost:3000/api/posts
```



