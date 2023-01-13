当前待优化：
请求按钮的loading

请求与账户暂时未关联

创建项目

```
create-react-app react-2b
```

安装scss

```
yarn add scss --save
```

安装axios

```
yarn add axios --save
```

安装http-proxy-middleware

```
yarn add http-proxy-middleware
```

```js
//setupProxy.js
const { createProxyMiddleware } = require('http-proxy-middleware')

module.exports = function (app) {
  app.use(
    '/api',
    createProxyMiddleware({
      target: 'http://127.0.0.1:4523/m1/2184036-0-default',
      changeOrigin: true,
      pathRewrite: {
        "^/api": "/"
      }
    })
  )
}
```

安装路由

```
yarn add react-router-dom
```

引入Antd

```
yarn add antd
```

Layout布局

```css
//解决高度问题
#root,
.ant-layout {
  height: 100vh;
}
```

