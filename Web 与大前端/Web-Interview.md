# Web Interview

# 语法基础

## HTML

## CSS

## DOM

# 工程实践

## 网络请求

- CORS 是什么？它的跨域请求规则是什么样子的？

- Axios 中如何取消掉某个请求？其在内部是如何执行请求撤销的操作的？

## 客户端可用性

- 如何设计一个客户端错误上报系统？应该如何捕获常见的浏览器错误？

- 如果设计一个客户端实时录屏系统，可能的技术挑战点会在哪里？应该如何去设计前后端架构？

## 状态管理

- Redux 的设计理念，其与 Flux 的区别？为什么 Redux 的 Store 会使用不可变数据类型？如下的使用可能会有哪些风险？

```ts
export default connect(
  state => (...state),
  { ...commonActions }
)(withRouter(App));
```

- Redux 中的 Reducer 为何要设置为纯函数？常见的 Side Effects 应当在哪个模块中处理？combineReducer 函数是如何实现的？

- Redux 的中间件是如何实现的？如何在 Redux 中实现简单的 Promise 中间件？

- Redux 常见的异步处理机制，简单的 Thunk 组件实现？

- Redux Sagas 为何要使用 Generator？put, get 等函数的原理是什么？其是如何处理 Effects 的？

- MobX 是如何实现响应式监听的？MobX 与 Redux 相比其适用场景、工程案例上有何差异？

- Dva 中的典型模型由哪几个部分组成，各自负责些什么？

## 响应式界面

- 如何实现动态主题切换的效果？

# 架构与优化

- 简单概述浏览器工作原理，即从用户输入某个 URL 开始到渲染，经历了哪些主要的步骤？浏览器的核心组件包含哪些？

- 如何设计一个 JavaScript 应用沙箱？

# 跨端开发

- Taro 等小程序跨端框架的实现原理是啥？编译时框架与运行时框架有何异同？
