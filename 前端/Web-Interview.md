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

# React

- 为何在 JSX 或者 TSX 的文件首部需要引入 React？

- Hooks 函数式组件与类组件相比各有何优劣？

- 猜想一下，React DevTools 是如何监听 React 的状态变化？

- 调用 setState 之后发生了什么？

- React 中 Element 与 Component 的区别是？

- 在什么情况下你会优先选择使用 Class Component 而不是 Functional Component？

- React 中 keys 的作用是什么？

- 如果你创建了类似于下面的 `Twitter` 元素，那么它相关的类定义是啥样子的？

```js
<Twitter username="tylermcginnis33">
  {user => (user === null ? <Loading /> : <Badge info={user} />)}
</Twitter>
```

- Controlled Component 与 Uncontrolled Component 之间的区别是什么？

- 在生命周期中的哪一步你应该发起 AJAX 请求？

- shouldComponentUpdate 的作用是啥以及为何它这么重要？

- 如何告诉 React 它应该编译生产环境版本？

- 为什么我们需要使用 React 提供的 Children API 而不是 JavaScript 的 map？

- 概述下 React 中的事件处理逻辑？

- createElement 与 cloneElement 的区别是什么？

- 传入 setState 函数的第二个参数的作用是什么？

## 库与框架

- 在 Antd 中如何实现主题切换的特性，如何在单个项目中打包多个不同版本的 Antd？

- 在 Antd 中，如何实现表单的异步校验？如何将某个表单的数据存放到 Redux 中？

# Vue

- Vue 单文件组件是如何解析的？Vue 中是否使用到了 Virtual DOM？

- v-if 指令是如何实现的？

- vue-class-component 是如何实现的？如何在页面跳转前后添加自定义的操作？如何自定义装饰器？

# 链接

- https://zhuanlan.zhihu.com/p/74258351
