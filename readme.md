# public-sdk

> 通用sdk库封装, 抹平平台的差异

## Basic

#### 线下demo

[点击体验](http://10.179.19.121/public-biz/@didi/public-sdk/index.html#/)

#### 开发规范

1. clone并新建分支 

```bash
# install dependencies
git clone && git checkout -b feature-xx
```

2. 在feature-xx分支`开发测试`完成后，rebase master 并在主干合并
   
```bash
git rebase master

git checkout master

git merge feature-xx
```

3. 修改package.json的version, 然后发布

```
  npm包：npm publish
  js link: npm run lib
```

## Usage

### 引入资源

1. 外链 JS 引入

```html
<script src="http://webapp.didistatic.com/static/webapp/shield/z/publicSdk/1.0.0/public-sdk/public-sdk.min.js"></script>
<script>
  window.publicSdk.geoLocation(...) // 获取定位
  window.publicSdk.setPageTitle({
    title: 'test'
  })
  window.publicSdk.setShare(...)
  window.publicSdk.pay(...)
</script>
```

2. npm 包引入

   执行 `npm install @didi/public-sdk` 进行安装

   搭配后编译使用，效果更佳，后编译webpack 配置

   ```
     * webpack.base.conf.js
     const BabelRulePlugin = require('@didi/babel-rule-plugin') (版本使用1.0.1)// 后编译
     const TransformModulesPlugin = require('webpack-transform-modules-plugin') // 按需加载
     plugins: [
         new BabelRulePlugin(),
         new TransformModulesPlugin()
       ]

     * .postcssrc.js (处理css)
     module.exports = {
       "plugins": {
         "postcss-import": {},
         "postcss-url": {},
         // to edit target browsers: use "browserslist" field in package.json
         "autoprefixer": {}
       }
     }
   ```

   开始使用

   ```javascript
   import { setPageTitle } from '@didi/public-sdk'

   setPageTitle({
         title: 'test'
   })
   ```

### 所有方法

  所有的 callback 函数传参中，形式为 callback(err, result)

- err 类型为 Error 可以通过 err.message 获取到报错信息
- result 结果。当存在错误时，该值为 undefined

#### 分享 setShare(options, callback)

> options

| 值               |   类型    | 是否必须 |   默认值   | 描述                                       |
| --------------- | :-----: | :--: | :-----: | ---------------------------------------- |
| disabled        | Boolean |  否   |  false  | 是否禁止分享                                   |
| title           | String  |  是   |  空字符串   | 标题                                       |
| lang            | String  |  否   | 'zh-CN' | 可选值"zh-CN"、"en-US"; 报错时返回的message语言为中文/英文 |
| desc            | String  |  是   |  空字符串   | 描述                                       |
| shareUrl        | String  |  是   |    无    | 分享链接                                     |
| imgUrl          | String  |  是   |    无    | 分享图标链接                                   |
| alipayShareAble | String  |  否   |  false  | 是否开启支付宝分享。如果要设置支付宝的分享，需要设置值为 true        |

```javascript
  sdk.setShare({
    title: '欢迎注册滴滴司机',
    desc: '闲时接几单，轻松赚收入！时间自由，收入稳定，欢迎加入！',
    imgUrl: 'https://img-hxy021.didistatic.com/static/gsh5actimg/dpub2_static/register_share_icon.jpg',
    shareUrl: 'https://page.udache.com/driver-register/index.html?channel=30&channelId=83262&reg_channel=30'
  }, (err, result) => {
    console.log(err) // null
    console.log(result)
    /*
    {
    from: 'wechat', // 微信分享:wechat, QQ分享：mqq 支付宝分享：alipay
    to: '', // 微信朋友圈：wechat timeline; 微信好友： wechat contact; QQ空间: mqq timeline QQ好友: mqq contact 支付宝朋友圈：alipay timeline；支付宝好友：alipay contact
    shared: true,  // 分享成功 true; 分享失败或取消分享 false
    msg: 'share success',
    ret: {         // 只有支付宝分享有此字段
      channelName // 分享渠道名称
      shareResult: // 分享结果 boolean
    }
    */
  })
```



#### 获取定位 geoLocation([options,] callback)

> options

| 值        |   类型    | 是否必须 | 默认值  | 描述                           |
| -------- | :-----: | :--: | :--: | ---------------------------- |
| fallback | Boolean |  否   | true | 通过bridge获取定位失败时，是否采用h5获取定位方法 |

> example

```javascript
  sdk.geoLocation({
    fallback: true
  }, (err, result) => {
    console.log(err)  // null
    console.log(result)
    /* 
    {
      lng:
      lat:
      maptype: 'soso' // 坐标系类型,最终转换为 soso 坐标系
      way: // 位置获取来源, 'wechat|alipay|didi|mqq|h5'
      accuracy: // 精度
    }
    */
  })

```



#### 设置标题 setPageTitle(options, callback)

> options

| 值        |   类型   | 是否必须 |   默认值   | 描述            |
| -------- | :----: | :--: | :-----: | ------------- |
| title    | String |  是   |  空字符传   | 标题            |
| lang     | String |  否   | 'zh-CN' | 可选值"zh-CN"    |
| subTitle | String |  否   |  空字符传   | 副标题，注意：仅支付宝支持 |

> example

  由于端内设置标题为同步方式，并无成功失败回调。所以设置标题成功调用端内 bridge 后就会返回成功的结果

```javascript
  sdk.setPageTitle({
    title: 'xxx'
  }, (err, result) => {
    console.log(result) // true
  })
// 设置标题完成
  callback(null, true)
```



#### 拉起支付 pay(options, callback)

> options

| 值                           |   类型   | 是否必须 |   默认值   | 描述         |
| --------------------------- | :----: | :--: | :-----: | ---------- |
| lang                        | String |  否   | 'zh-CN' | 可选值"zh-CN" |
| 其他参数：参考微信、支付宝、mqq支付拉起时需要的参数 |        |      |         |            |

> example

```javascript
  sdk.pay({
    lang: 'zh-CN'
  }, (err, result)=>{
    console.log(err) // 失败时err为一个error类型的原因,成功时为null 
    console.log(result)
    /*
    {
      msg: 'pay success'
   }
    */  
  })
```

#### 查询用户登录态 isLogin(options, callback)

> options

| 值                                        | 类型     | 是否必须 | 默认值        | 描述                                       |
| ---------------------------------------- | ------ | ---- | ---------- | ---------------------------------------- |
| env                                      | String | 否    | 'mainland' | 引入的 Web 登录 SDK 环境，可选值 'dev'、'mainland'、'global' |
| 其余参数请参考 [Global Login SDK](https://git.xiaojukeji.com/webapp/trinity-login/blob/global/README.md) 中的 API -> setConfig 部分 |        |      |            |                                          |

此处的 options 参数仅提供给 Web 登录使用，因为乘客端、司机端查询用户登录态不需要配置参数

> example

```js
sdk.isLogin({
  env: 'mainland',
  appid: 30004,
  role: 1,
  ...extraParams // 其余参数请参考 Global Login SDK 中的 API -> setConfig 部分
}, (err, result)=> {
  console.log(err) // 用户已登录err为null，其他为类型为error的错误原因
  console.log(result)
  /*
  {
    phone: '11000000112',
    token: '123213qwqweq$2454213qwe'
  }
  */
})   
```



#### 拉起登录框 forceLogin(options, callback)

> options

| 值                                        | 类型     | 是否必须 | 默认值        | 描述                                       |
| ---------------------------------------- | ------ | ---- | ---------- | ---------------------------------------- |
| env                                      | String | 否    | 'mainland' | 引入的 Web 登录 SDK 环境，可选值 'qatest'、'mainland'、'global' |
| 其余参数请参考 [Global Login SDK](https://git.xiaojukeji.com/webapp/trinity-login/blob/global/README.md) 中的 API -> setConfig 部分 |        |      |            |                                          |

此处的 options 参数仅提供给 Web 登录使用，因为乘客端、司机端查询用户登录态不需要配置参数

> example

```js
  sdk.forceLogin({
    env: 'mainland',
    appid: 30004,
    role: 1,
    ...extraParams // 其余参数请参考 Global Login SDK 中的 API -> setConfig 部分
  },(err, result)=>{
    console.log(err) // 登录成功err为null，其他为类型为error的错误原因
    console.log(result)
    /*
    {
      phone: '11000000112',
      token: '123213qwqweq$2454213qwe'
    }
    */
  })
```

