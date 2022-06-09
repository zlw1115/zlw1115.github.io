---
layout:     post   				
title:      JavaScript函数和变量的声明与执行顺序 & 回调函数
subtitle:   
date:       2022-06-09 				
author:     zlw				
header-img: img/home.jpg 	
catalog: 	true 						
tags:								
    - JavaScript
---

# JavaScript知识点总结

### JavaScript函数和变量的声明和执行顺序

JavaScript的解析过程分为两个阶段：预编译期（预处理）与执行期

预编译期：JavaScript会被本代码块中的所有var声明的变量和函数进行处理（类似C语言的编译），但是需要注意的是此时处理的函数只是声明式函数，而且变量也只是进行了声明但未进行初始化以及赋值。

执行期：会按照代码块的顺序逐行执行。

**函数的执行顺序**

```
// 声明式函数
function foo(){

}
// 函数表达式(赋值式函数)
var bar = function(){

}
// 声明式函数与赋值式函数的区别在于：JavaScript的预编译期，声明式函数将会被提取出来，然后才按顺序执行js代码。
```

```
// 含参函数倒序
f();        //alert 2
function f() {
  alert(2);
}

// 声明式函数和赋值式函数
f();                        // 2
function f() {              // 声明式函数
  console.log(2);
}
f1();                       // Uncaught TypeError: f1 is not a function
var f1 = function () {      // 赋值式函数
  console.log(2);
}  
```

**变量的执行顺序**

在变量未定义之前，会返回undefined

```
alert(a); // alter undefined
var a = 2;
```

局部变量的执行

注意：JavaScript中全局var声明的为全局变量，函数体内var声明的为局部变量（函数外部访问不到）。但是，函数体内未用var声明的为全局变量（函数外部可以使用）。

```
// alert(a) // a is not defined
function f() {
  // alert(a) // error: a is not defined
  a = 3
}
// alert(a) // a is not defined
f() //error: a is not defined
alert(a) // 3
```



```
function f(){
	alert(a);
	var a = 3;
}
f();        //undefined  
```

函数内部再次全局声明，会修改全局的a

```
var a = 1;
function f() {
  alert(a);
  a = 2;
  alert(a);
}
f();        //1 和 2
```

函数内全局复制一次，var声明一次，函数f()内还是会优先使用自己的变量a

```
var a = 1;                 //           函数f()内变量a的执行顺序
function f() {
  alert(a);           
  a = 2;                 //                    #2   
  alert(a);
  var a = 3;             //等价于 var a ;       #1
  //      a = 3 ;       #3
  alert(a);
}

f();                //undefined 2 和 3
alert(a);       //1  
```

例子：

```
var a, b;
(function () {
  alert(a);          //undefined
  alert(b);          //undefined
  var a = b = 3;             //等价于 var a = 3 ; b = 3; b是全局的
  alert(a);          //3
  alert(b);          //3
})();
alert(a);              //undefined
alert(b);              //3
```



### 回调函数

js在执行程序时，所有代码都在执行栈中，此时都是同步执行，但在运行代码时如果遇见事件绑定、计时器会将抛到异步任务队列里面等待js引擎执行，然后继续执行在执行栈中的代码，与此同时，被抛到异步任务队列里面的计时器会执行等待的时间，如果等待时间结束，则会将此事件抛到执行队列里面。执行栈里面的代码自行完成后，程序回去检查执行队列里面时候有事件，如果有，则拿出来执行，如果没有，则会一直循环检查，指导异步事件队列和执行队列里面都没有才结束程序。

![js执行队列](./img/2022-06-09js执行队列.png)



可以像使用变量一样使用函数，作为另一个函数的参数，在另一个函数中作为返回结果，在另一个函数中调用它。当作为参数传递一个回调函数给另一个函数时，只传递了这个函数的定义，并没有在参数中执行它。当包含（调用）函数拥有了在参数中定义的回调函数后，它可以在任何时候调用（也就是回调）它。















