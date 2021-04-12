---
title: let 暂存死区
categories: 前端进阶
tags: 
- 基础
- 笔记
wordcount: true
---
# 关于let/const
通过查阅MDN文档得到的信息。
1. 声明一个块级作用域的本地变量。
2. 不会再全局声明时创建window对象的属性。
3. 在同一块级作用域下，不能重复声明。
4. 暂存性死区。（不能在声明前访问）
5. const声明创建一个值的只读引用，不可变，必须在初始化时赋值。（但这并不意味着它所持有的值是不可变的，只是变量标识符不能重新分配）
## 暂存性死区
let 声明的变量在执行时才进行初始化。在变量初始化前访问该变量会导致**ReferenceError**。该变量处在一个自块顶部到初始化处理得**暂存性死区**中。
```javascript
let a = 1
{
  // 暂存性死区 start
  console.log(a)
  // 暂存性死区 end
  let a = 3
}
```
关于上面代码，结果就是会报**ReferenceError**错。我的理解是在块作用域内，存在一个自块开始到let声明的一个暂存性死区，在这个暂存性死区中存在一个a这样一个变量，在变量a的暂存性死区中访问a就回出现**ReferenceError**错误。

关于暂存性死区我的理解是它是一个关于哪个变量的暂存性死区，而不是在一个块作用域中只有一个暂存性死区。
```javascript
{
  // a,b 的暂存性死区 start
  // a 的暂存性死区 end
  let a = 1
  console.log(a)
  console.log(b)
  // b 的的暂存性死区 end
  let b = 2
}
```
上述代码console.log(a)正常执行，console.log(b)因为在b的的暂存性死区中访问b，所以报错**ReferenceError**。