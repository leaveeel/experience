<kbd> 2021年6月10日 </kbd>  
  
# 前言  
  
> 重新工作也有一年，写vue也快一年，有些重复使用的特性/属性/代码/逻辑却不能立刻想好，顾记录工作时自认为有用的段落/结构/方法等。  
  
## 创建  
  
一般来说使用初始/配置package.json和vue.config.js文件，在目录内执行`npm install`。ui框架方面移动端使用 <kbd>vant</kbd> ，pc端使用 <kbd>elementui</kbd> 。  
  
### 结构  
  
__项目结构不尽相同，整体上应该差不多。__  
  
```
├── node_modules             //依赖文件夹，由npm下载。如果没有修改只要存在本地，不用上传服务器  
├── public  
│   ├── index.html          //入口文件，可以在这里引用js或初始化css  
│   └── server.js           //用来配置服务器域名，方便修改生产或开发环境地址  
├── src  
│   ├── api  
│   │   └── index.js       //全局配置接口地址  
│   ├── assets              //资源文件夹  
│   ├── components  
│   │   ├── request.js     //封装axios
│   │   ├── index.js       //公共js
│   │   └── template       //存放各组件文件（夹）
│   │        └── ...  
│   ├── router
│   ├── store
│   ├── views               //项目文件目录  
│   ├── App.vue             //vue入口文件，在这里处理路由缓存、储存全局参数等  
│   └── main.js             //初始化文件，全局依赖引用、配置  
├── package.json             //包管理和配置  
├── package-lock.json  
└── vue.config.js            //vue配置文件  
```
  
### 解析
- public
    - index.html
        - 基本的html文件，在这引用 <kbd>server.js</kbd> 给api使用，`<%= BASE_URL %>`为资源目录，由vue.config.js配置，默认为`/`。
        - 引用css或者在`<style />`里初始化样式
        - 移动端设置rem单位，可引用js或在`<script />`标签设置
    - server.js
        - 配置window全局变量，用来修改服务器地址或者公共配置
