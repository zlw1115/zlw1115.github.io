---
layout:     post
title:      JavaScript运行原理
subtitle:   
date:       2020-10-22
author:     zlw
header-img: img/home.jpg
catalog: 	 true
tags:
    - JavaScript
---

## JavaScript运行原理

### 一、V8引擎原理

1. 词法分析器（lexical analyzer，也叫扫描器 scanner）对JavaScript源代码进行词法分析，将字符序列转换成token序列，
2. 再由语法分析器（parser）进行语法分析，转换成AST（abstract syntax tree,抽象语法树）。如果函数没有被调用，那么不会被转换成AST。
3. 再由解释器（lgnition），将AST转成ByteCode（字节码）。同时会收集TurboFan优化所需的信息（比如函数参数的类型信息等）。如果函数只调用一次，lgnition会解释执行ByteCode.
4. 如果一个函数被多次调用，那么会被标记成热点函数，那么会被编译器优化成机器码，提高代码的执行性能。TurboFan是一个编译器，可以将字节码编译为CPU可以直接执行的机器码。但是，机器码实际上也会被还原成ByteCode，当后续执行过程中，类型发生了变化（比如sum函数原来执行的是number类型，后来执行变成了string类型），之前优化的机器码并不能正确的处理运算，就会逆向的转换成字节码。

![](E:\gitrepo\zlw1115.github.io\img\2022-10-24\v8引擎解析图.jpg)

### 二、JavaScript执行过程

**初始化全局对象**

js引擎在执行代码之前，会在堆内存中创建一个全局对象：Global Object（GO）。该对象所有的作用域（scope）都可以访问；里面包含Date、Array、String、Number、setTimeout、setInterval等，其中还有一个window属性指向自己。

**执行上下文（Execution Contexts）**

- js引擎内部有一个执行上下文栈（Execution Context Stack，简称ECS）,它是用于执行代码的调用栈。

- 全局的代码块为了执行会构建一个Global Execution Context（GEC），GEC会被放入到ECS中执行：
  - 第一部分：在代码执行前，在parse转成AST的过程中，会将全局定义的变量、函数等加入到GlobalObject中，但是并不会赋值，这个过程称之为变量的作用域提升（hoisting）。
  - 第二部分：在代码执行中，对变量赋值，或者执行其他函数。

**VO对象（Variable Object）**

每一个执行上下文会关联一个VO(Variable Object，变量对象)，变量和函数声明会被添加到这个VO对象中。

当全局代码被执行的时候，VO就是GO对象了。

**函数执行**

在执行过程中，执行到一个函数时，就会根据函数体创建一个函数执行上下文（Fuction Execution Context，简称FEC），并且押入到EC Stack中。

因为每个执行上下文都会关联一个VO，函数执行上下文关联的VO：

- 当进入一个函数执行上下文时，会创建一个AO（Activattion Object）

- 这个AO对象会使用arguments作为初始化，并且初始值是传入的参数

- 这个AO对象对作为执行上下文的VO来存放变量的初始化。

### 三、全局代码执行

**待执行代码**

![](E:\gitrepo\zlw1115.github.io\img\2022-10-24\全局代码执行示例.png)

**执行前**

![](E:\gitrepo\zlw1115.github.io\img\2022-10-24\01_全局代码的执行过程2.png)



**全局代码执行**

![](E:\gitrepo\zlw1115.github.io\img\2022-10-24\全局代码执行.jpg)

### 四、函数代码执行

**待执行代码**

![](E:\gitrepo\zlw1115.github.io\img\2022-10-24\函数代码执行示例.png)

**函数代码执行前**

![](E:\gitrepo\zlw1115.github.io\img\2022-10-24\函数代码执行前.png)

**函数代码执行**

![](E:\gitrepo\zlw1115.github.io\img\2022-10-24\函数代码执行.png)

**函数代码多次执行**

![](E:\gitrepo\zlw1115.github.io\img\2022-10-24\函数代码多次执行.jpg)

![](E:\gitrepo\zlw1115.github.io\img\2022-10-24\函数代码多次执行.png)

### 五、闭包的访问

**待执行代码**

![](E:\gitrepo\zlw1115.github.io\img\2022-10-24\闭包的访问过程.png)



### 六、作用域和作用域链

**当进入到一个执行上下文时，执行上下文也会关联一个作用域链**

作用域链是一个对象列表，用于变量标识符的求值；

当进入一个执行上下文时，这个作用域链被创建，并且根据代码类型，添加一系列的对象；

### 七、题目

```
var n = 100

function foo1(){
    console.log(n)//100
}

function foo2(){
    var n = 200
    console.log(n)//200
    foo1()
}

foo2()
// 函数有作用域
console.log(n) // 100
```

```
function foo(){
    var a = b = 100
}

foo()
// console.log(a) // test.js:13 Uncaught ReferenceError: a is not defined
console.log(b) // 100
```



