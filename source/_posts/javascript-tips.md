title: JavaScript一些实用的技巧
date: 2016-08-10 09:50:42
tags:
- JavaScript
---
最近看了很多开源项目的源码，感觉学到了不少东西，也学到了很多比较碎的技巧。JavaScript小技巧实在是太多，把这些小技巧和之前知道的一起总结一下，并且持续更新，权当是备忘。

<!-- more -->

### this.length >>> 0
无符号右移运算，该操作常见于数组遍历，比如玉伯曾经发布的es5-safe模块里面有大量的该操作。作用是确保.length 是一个0到2^32的正整数
* 所有非数字的被转换成0
* -1 转换成 0
* 负数转换成正整数
* 浮点数相当于执行Math.floor

```js
1>>>0            === 1
-1>>>0           === 0xFFFFFFFF          -1>>0    === -1
1.7>>>0          === 1
0x100000002>>>0  === 2
1e21>>>0         === 0xDEA00000          1e21>>0  === -0x21600000
Infinity>>>0     === 0
NaN>>>0          === 0
null>>>0         === 0
'1'>>>0          === 1
'x'>>>0          === 0
Object>>>0       === 0
```
### ~index
在查找某字符串或者数组元素在字符串或者数组里面的索引时，如果不存在index都为-1， 这个时候用~index 如果index为 -1 会被转换成0, 相当于false
```js
var idx = str.indexOf('a');
if(idx > -1){
	doSomething();
}
(idx > -1) && doSomething();
~idx && doSomething()
```
### void 0
void 0 是用来替换undefined的， 由于undefined在JavaScript中并非关键字， 我们可以直接给undefined赋值从而修改了undefined，由此来看undefined并不安全。 void 0 实际上总是返回undefined值，所以更加安全，而且相对来说也更加短，节省字符数。
```js
undefined = 123; //undefined 可以被赋值
```
### (0,eval)('this')

先上代码
```js
var _global = (function(){ 
	return this || (0,eval)("this"); 
}());
```
用于获取全局对象, 可以兼容浏览器和Node.js， 严格模式和非严格模式。 
this 在非严格模式下在浏览器端指向window，在Node.js里面指向global或者GLOBAL (Node.js源码里面这样写的 `global = this;global.GLOBAL = global;` ),
可是在严格模式下this却是undefined。
而(0,eval)("this") 总是能够返回全局对象， 那么为什么不直接调用eval('this')呢？
```
var x = 'outer';
(function() {
  var x = 'inner';
  eval('console.log("直接调用 Eval:" + x)'); 
  (1,eval)('console.log("间接调用Eval: " + x)'); 
})();
```

> 直接调用 Eval:inner
> 间接调用Eval: outer

所以如果直接调用的话可能会返回当前作用域this。

### Date格式化
经常会遇到需要格式化日期时间为 yyyy-MM-dd HH:mm:ss格式，这里有种比较简单的方法
```js
var date = new Date();
var result = date.toLocaleString('zh-CN', { hour12: false })
  .replace(/\//g, '-').replace(/\b\d\b/g, '0$&');
```

### 取月份天数

```js
new Date(year, month + 1, 0).getDate();
```
new Date()第三个参数为0 ，并且month 为 month + 1 是返回的是当前month的最后一天，即是月份的天数。
