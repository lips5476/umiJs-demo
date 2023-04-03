## 创建和运行
npm  create @umijs/umi-app
npm i 
npm start


## 文件目录
.editorconfig  对编译器做高亮支持的
.umirc.ts    相当于文件目录下的config/config.js   用来配置构建时的环境
文件目录下的dist  打完包后输出的文件夹（生产目录）
文件目录下的public  放一些变通的数据资源和无需打包的资源
.env  配置环境变量的文件
src/layouts    全局的一些页面布局的文件夹   container组件之类的
src/models     状态管理相关的
src/wrappers    权限管理相关的
src/global.css   配置全局的样式文件 
src/global.js   配置全局的js文件 
src/app.js       用来配置运行时的环境
pages/document.ejs    相当于webpack打包时候public里的html入口文件模板



## 构建时候的配置  config/config.js   或者  .umirc.js（配置不复杂的时候使用）  二者有一个就好了
nodeModulesTransform：{type:'none'}  配置关于node_module编译的问题 type 为 'none'为不编译  'all'为编译全部会影响速度
routes:[{path:'/',component:''}]  用来配置路由 也可以写成一个独立的路由文件
fastRefresh:{},  快速编译 同时保持组件状态  类似于keep-alive
devServer:{
  port:8081   //.env里也可以配置 且权限更高
  https:true  //启用https安全访问
},
title:'UMI'  相当于最开始的document.title  之后在别的运行时也可以手动写代码修改
favicon:'线上地址',
aynamicImport:{  //按需加载
  loading:'@/components/loading.tsx'  //按需加载的时候loading的组件
}，
mountElementId:'app'，  //配合pages/document.ejs 入口文件里的根元素  <div id="app"></div>
theme, 可以和routes一样外部引入,
proxy, 可以外部引入  反向代理配置


## 路由权限
[
  {
    path:'/home',
    wrapper:['@/pages/auth'],    //当进入home的时候会先进入wrapper组件进行权限判断
    component:'@/pages/home',
  },
  {
    path:'/login',
    component:'@/page/login'
  }
]

所以warpper里的auth.tsx可以这么写

import {Redirect} from 'umi'

export default (props)=>{
  if(没有权限){
    return (
      <div>{props.children}</div>
    )
  }
  return <Redirect to="/login"/>
}



## umi的反向代理原理
umi —》 发送请求 —》umi服务器 —》代理转发 —》目标服务器

## umi反向代理配置
proxy.js

export  default {
  '/api':{
    target:'后端服务器地址',
    https:true,
    changeOrigin:true,  //依赖origin的功能需要这个 如cookie
    pathRewrite:{'^/api',''}   //路径替换
  }
}


## useReuest在config.js里的配置
由于useRequest返回值里必须携带一个data字段
所以可以在config.js里设置一下
request:{
  dataField:''
} 


## 用于权限校验的业务可以放在src/app.js  的render函数里
import {request,history} from 'umi'
export const render =async (oldRender) =>{
  const {isLogin}  = await request('xxxxxx')
  if(!isLogin){
    history.push('./login')
  }

  //oldRender需要至少被调用一次
  oldRender()
}

  

## 动态添加路由添加可以放在src/app.js的    patchRoutes函数里

export const patchRoutes =({routes})=>{
   //动态添加路由必须是这种形式的
   routes.push({exact:true,componeent:require(@/pages/404).default})

}

 注意点 
 component是require引入而非import  所以要.default
 动态路由跳转不显示 需要关闭mfsu
 主要子路由构建时需要带上exact:true否则不显示
 
  
## 路由监听可以在  src/app.js 的 onRouteChange的函数里
export const onRouteChange({matchRoutes,location,routes,action}){
  console.log('routes',routes) //路由对象合集
  console.log('matchRoutes',matchRoutes) //当前匹配的路由和子路由
  console.log('location',location) //location对象
  console.log('action',action) //跳转到当前的路由所执行的操作  POP PUSH ..
}


## 拦截器  在发送和接收数据的时候做的通用配置(如loading) 可以在 src/app.js的  request对象里
export const request = {
  timeout: 1000,
  errConfig:{},  //错误处理
  middlewares: [],  //使用中间件
  requestInterceptors: [   //用来做请求拦截的
    (url,option)=>{
      options.headers = {token:'dajkasdb'}  //如在发送请求的过程中携带上token
      return {url,option}  //主要操作完成之后是需要返回的
    }

  ], 
  responseInterceptors: [  //用来做响应拦截的
     (res,option)=>{
      //res 响应体   //出错的话显示错误组件或者出错操作
      return res 
    }

  ], 
}
