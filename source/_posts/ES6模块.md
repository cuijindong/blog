---
title: ES6模块
categories: 前端进阶
tags: 
- 模块化
- es6
wordcount: true
---
# 理解模块
什么是模块呢，模块模式的中心思想很简单，就是把逻辑分块，个自封装，相互独立，每个模块自行决定向外暴露什么，同时自行决定引入执行那些外部代码。可以把每个模块想象成一个键/值实体，键就是模块文件的路径，值就是模块所暴露的内容。
# ES6之前的模块加载器
在 ES6 原生支持模块之前，使用模块的 JavaScript 代码本质上是希望使用默认没有的语言特性。因
此，必须按照符合某种规范的模块语法来编写代码，另外还需要单独的模块工具把这些模块语法与
JavaScript 运行时连接起来。这里的模块语法和连接方式有不同的表现形式，通常需要在浏览器中额外
加载库或者在构建时完成预处理。
## CommonJS同步模块定义
CommonJS规范了同步依赖的模块定义。主要用于服务端实现模块化，但可以通过工具编译成普通js后再浏览器中运行。CommonJS语法不能直接再浏览器中运行。
### require/module.exports
CommonJS通过require指定依赖，通过module.exports暴露公共api。
```javascript
var moduleB = require('./moduleB');
module.exports = {
 stuff: moduleB.doStuff();
}; 
```
### 赋值
请求模块会加载模块，把模块赋值给变量是非常常见的，但不是必要的，调用**require()**就意味着模块会原封不动的加载进来。
```javascript
// moduleA
console.log('moduleA');
// moduleB
require('./moduleA');   // moduleA
```
### 模块永远是单例
一个模块不论被require多少次，它永远只被加载一次。模块第一次加载会被缓存，后续加载会从缓冲中取该模块。下面的列子'moduleA'只会被打印一次。
```javascript
// moduleA
console.log('moduleA');
// moduleB
const test1 = require('./moduleA');   // moduleA
const test2 = require('./moduleA');
test1 === test2;    // true
```
## AMD异步模块定义
AMD的策略是让模块声明自己的依赖，而再浏览器中的模块系统会按需获取依赖，并再依赖加载完成后立即执行依赖他们的模块。
AMD模块实现核心使用函数包装模块定义。这样可以防止全局命名污染，并允许加载器库控制何时加载模块。
```javascript
// ID 为'moduleA'的模块定义。moduleA 依赖 moduleB，
// moduleB 会异步加载
define('moduleA', ['moduleB'], function(moduleB) {
 return {
 stuff: moduleB.doStuff();
 };
}); 
```
# ES6模块
有原生浏览器的支持，集**CommonJS**和**AMD**之大成者！
## 模块标签以及定义
ES6模块作为一整块js代码而存在，通过带有 **type="module"** 属性的 **&lt;script&gt;** 的标签来告诉浏览器要把相关代码当作模块来执行，而不是普通的js脚本文件。模块代码可以是内嵌式，也可是外部文件引入。
```html
<!-- 内嵌式 -->
<script type="module">
 // 模块代码
</script> 
<!-- 外部文件引入 -->
<script type="module" src="path/to/myModule.js"></script> 
```
浏览器会在解析到 **&lt;script type="module" &gt;&lt;/script&gt;** 标签后立即进行该模块的下载，但不会立即执行，会在文档解析完成后执行，执行顺序就是在页面中出现的顺序。
```html
<!-- 第二个执行 -->
<script type="module" src="module.js"></script>
<!-- 第三个执行 -->
<script type="module" src="module.js"></script>
<!-- 第一个执行 -->
<script><script> 
```
也可以给模块标签添加 **async** 属性。这样影响就是双重的：不仅模块执行顺序不再与 **&lt;script&gt;** 标签在页面中的顺序绑定，模块也不会等待文档完成解析才执行。不过，入口模块仍必须等待其依赖加载完成。
## ES模块特性
* 模块代码只在加载后执行。
* 模块只被加载一次。
* 模块是单例。
* 模块可以定义公共接口，其他模块可以基于这个公共接口观察和交互。
* 模块可以请求加载其他模块。
* 支持循环依赖。
* 模块是异步加载和执行的。
* 模块不污染全局命名空间。
* 模块默认在严格模式下执行。
## improt/export
ES6模块通过**improt**进行导入，通过**export**进行导出。**export**进行导出有两种方式：命名导出和默认导出。
## improt
### 命名导出
1. 变量声明和导出在一起。
```javascript
// 导出 module.js
export const test = 'test';
// 导入
import {test} from './module.js';
```
2. 变量声明和导出不在一起。

```javascript
// 导出 module.js
const test = "test";
export {test};
// 导入
import {test} from './module.js';
```
3. 可以混合使用。
```javascript
// 导出 module.js
const test1 = "test1";
const test2 = "test2";

export const test3 = "test3";
export const test4 = "test4";
export {test1, test2};
// 导入
import {test1, test2, test3, test4} from './module.js';
```
4. as关键字，提供别名。
```javascript
// 导出 module.js
const test1 = "test1";
const test2 = "test2";

export {test1 as myTest, test2};
// 导入
import {myTest, test2} from './module.js';
```
### 默认导出
通过使用**default**关键字将一个值声明为默认导出，每个模块只能有一个默认导出。
1. 默认导出，导入默认导出时可以自定义变量名进行接收。
```javascript
// 导出 module.js
const test = 'test';
export default test;
// 导入
import testDemo from './module.js'  // 可自定义变量名
```
2. 和命名导出混用。
```javascript
// 导出 module.js
const test1 = 'test1';
const test2 = 'test2';

export {test2};
export default test1;
// 导入
import {test2} from './module.js'   // 导入命名导出test2
import testDemo from './module.js'  // 导入默认导出，可自定义变量名
```
3. as关键字，提供别名。
```javascript
const foo = 'foo';
// 等同于 export default foo;
export { foo as default }; 

// 这两个 export 语句可以组合为一行
const foo = 'foo';
const bar = 'bar';
export { foo as default, bar };
```
## export
不是必须通过导出的成员才能导入模块。如果不需要模块的特定导出，但仍想加载和执行模块以利
用其副作用，可以只通过路径加载它
```javascript
import './test.js';
```
1. 指名导入（可用as关键字）。
```javascript
import { foo, bar, baz as myBaz } from './foo.js';
console.log(foo); // foo
console.log(bar); // bar
console.log(myBaz); // baz 
```
2. 默认导入（可用as关键字）。
```javascript
// 等效
import { default as foo } from './foo.js';
import foo from './foo.js';
```
3. 命名导出和默认导出的区别也反映在它们的导入上。命名导出可以使用*批量获取并赋值给保存导
出集合的别名，而无须列出每个标识符。
```javascript
const foo = 'foo', bar = 'bar', baz = 'baz';
export { foo, bar, baz }
import * as Foo from './foo.js';
console.log(Foo.foo); // foo
console.log(Foo.bar); // bar
console.log(Foo.baz); // baz 
```
4. 默认导入和知名导入混用。
```javascript
// 等效
import { default as foo, bar, baz } from './foo.js';
import foo, { bar, baz } from './foo.js';
import foo, * as Foo from './foo.js';
```