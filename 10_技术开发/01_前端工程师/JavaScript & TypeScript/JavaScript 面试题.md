> 参考答案请查阅《[JavaScript-Notes](https://github.com/wx-chevalier/JavaScript-Notes?q=)》

# JavaScript Interview

- 如何设计一个 JS SDK，你会从哪几个维度进行考虑？

# 语法基础

## 类与对象

- 以下代码的输出是什么样子的？

```js
class Base {
  constructor(data = {}) {
    Object.assign(this, data);
  }
}

class A extends Base {
  a = null;

  constructor(data = {}) {
    super(data);
  }
}

new A({ a: 1 }); // { a: null }
```

# 异步并发

- Event Loop 的基础概念是否了解？浏览器中的 Event Loop 与 Node.js 中的 Event Loop 有何区别？

- JavaScript 常见的异步写法，Promise.finally 的实现？

# TypeScript

- 如何在 TypeScript 中设计简单的 IoC & DI 框架？如何实现简单的接口注解以自动生成类 Swagger 文档？

- 在 TypeScript 中如何定义递归类型，譬如 `[1, [2, 3, 4], 5]`？

- TypeScript 的 Partial 在底层是如何定义的？如果要实现 DeepPartial，应该如何定义？

- TypeScript 中如何进行类型断言与类型捕获的？

# V8

- V8 引擎中的对象与数组是如何存储的？为何 JS 的数组每个数据类型都可以不一样？为何 JS 的数组无需提前设置长度，是可变数组？为何数组可以像 Object 一样挂载任意属性？如果浏览器需要进行大规模数组运算，应该选择怎样的存储结构？

# TypeScript

- How to create a Partial-like that requires a single property to be set

```ts
type AtLeast<T, K extends keyof T> = Partial<T> & Pick<T, K>;
```

- TypeScript 中如何实现类的多继承？
