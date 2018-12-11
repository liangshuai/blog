title: JavaScript的一些坑
date: 2016-06-07 10:54:52
tags:
- Javascript
---

JavaScript作为一门同时结合了函数式编程和面向对象编程的语言，使用起来非常的宽松灵活。但是由于其诞生之初仓促的设计([Javascript诞生记](http://www.ruanyifeng.com/blog/2011/06/birth_of_javascript.html)) 、设计之初并没有同时结合函数式编程和面向对象编程的先例、过早标准化等问题，导致使用过程中经常会遇到各种问题。本文总结一些问题，部分是可能语言上的缺陷，也有部分属于用法的问题。

<!-- more -->

### 1. 整数上调用Number.prototype的方法

Number.prototype里面有toFixed()、toLocaleString()、toString()、valueOf()、toSource()等方法，直接调用的话会报错，以toString()为例

```js
3.toString()
3..toString()
(3).toString()
3 + '' // 推荐的转换姿势
```

上面的第一行会报
> Uncaught SyntaxError: Unexpected token ILLEGAL(…)

该问题的原因是当在整数后调调用toString()时，JavaScript发现整数后面的点后，会期望遇到的是一个浮点数，结果点后面是toString()，所以出现异常。

3..toString() 能够正常输出是由于`3.`  会被解释成浮点数，相当于对一个浮点数调用toString()


### 2. 2 == [[[2]]]

```js
2 == [[[2]]]; 			// true
var a = [0, 1, 2, 3];	
a[[2]] === a[2];		// true
[[[[[[[2]]]]]]] == 2;	//true

```

对于非严格相等， JavaScript比较规则如下([ECMAScript® 2015 Language Specification](http://www.ecma-international.org/ecma-262/6.0/#sec-abstract-equality-comparison)):

```js
ReturnIfAbrupt(x).
ReturnIfAbrupt(y).
If Type(x) is the same as Type(y), then
Return the result of performing Strict Equality Comparison x === y.
If x is null and y is undefined, return true.
If x is undefined and y is null, return true.
If Type(x) is Number and Type(y) is String,
return the result of the comparison x == ToNumber(y).
If Type(x) is String and Type(y) is Number,
return the result of the comparison ToNumber(x) == y.
If Type(x) is Boolean, return the result of the comparison ToNumber(x) == y.
If Type(y) is Boolean, return the result of the comparison x == ToNumber(y).
If Type(x) is either String, Number, or Symbol and Type(y) is Object, then
return the result of the comparison x == ToPrimitive(y).
If Type(x) is Object and Type(y) is either String, Number, or Symbol, then
return the result of the comparison ToPrimitive(x) == y.
Return false.
```

所以当`2 == [[[2]]]` 比较时， [[[2]]] 会被转换成原始类型，相当于执行了

```js
2 === Number([[[2]]].valueOf().toString()) 
```
所以结果是true

而`a[[2]] === a[2]` 比较时由于a[] 里面需要的是数字或字符串， 所以`[2]` 会被转换成数字或字符串，所以相当于a[2]

### 3. Number上的一些坑

```js
Number.MIN_VALUE > 0; //true
typeof NaN	//number
NaN === NaN	//false
```
上面第一个 `Number.MIN_VALUE` 其中`MIN_VALUE`返回的是大于零的最小数字，这个很容易被误解。
第二个NaN从字面上看应该是"Not a Number", 但是ECMA Spec对 [Number type](http://www.ecma-international.org/ecma-262/5.1/#sec-4.3.20) 定义为`set of all possible Number values including the special “Not-a-Number” (NaN) values, positive infinity, and negative infinity` , 所以NaN、Number.NEGATIVE_INFINITY、Number.POSITIVE_INFINITY都是Number类型的。
第三个NaN === NaN表面上看起来应该是相等的，但是ECMA上对严格相等 [The Strict Equality Comparison Algorithm](http://www.ecma-international.org/ecma-262/5.1/#sec-11.9.6) 定义中有
```
If Type(x) is Number, then
If x is NaN, return false.
If y is NaN, return false.
If x is the same Number value as y, return true.
If x is +0 and y is −0, return true.
If x is −0 and y is +0, return true.
Return false.
```
由于NaN类型是Number，可见不管任何数与NaN相比较，均会返回false。

### 4. String字面量的instanceof

```js
"string" instanceof String;	//false
var string = new String('string');
string instanceof String;	//true
```
JavaScript中字符串分两种，一种是字面量的，一种是Object的，对于字面量的字符串，并不是String的instance。 所以通常判断类型是不是字符串通常使用
```js
typeof(s) === 'string' || s instanceof String;
//或者
'string'.constructor === String;
```

### 5. ==的一些问题

JavaScript推荐使用严格相等`===` , 因为非严格相等经常会出现一些问题

```js 
undefined == null //true
"" == false; // true 
[] == false; // true
[] == ![];	//true
null == false; // false
null == true; // false
```

对于第一行从上面第二条非严格相等的规则中可以知道是true。
对于`"" == false` 由于Type(y) 是Boolean， 所以转换成Number， 即`0` ，此时由于Type(x)是String，所以根据规则，会转换成Number，即 `0` 从而得到相等。

对于`[] == false` 比较复杂， 首先是`Type(y)` 是`Boolean`，转换成数字`0`, 由于`Type(x)`是`Object`，此时的`Type(y)`是`Number`，所以执行`ToPrimitive(x)`

`ToPrimitive`规则如下
> ToPrimitive(input, PreferredType?)

PreferredType 可以是Number和String, 代表一个转换的偏好，结果不一定是该类型的，但是结果一定是一个原始类型的值。 如果PreferredType是Number会按照如下规则

```js
如果输入的值已经是个原始值,则直接返回它.
否则,如果输入的值是一个对象.则调用该对象的valueOf()方法.如果valueOf()方法的返回值是一个原始值,则返回这个原始值.
否则,调用这个对象的toString()方法.如果toString()方法的返回值是一个原始值,则返回这个原始值.
否则,抛出TypeError异常.
```

如果是String，那么二三行会互换。如果没有的话，对于Date类型的对象会被设置成String，其它类型的会被设置成Number。

由于`[].valueOf()` 依旧是`[]` , 所以会执行`toString()`方法转换成空字符串。
所以相当于比较`"" == 0` 从而得到相等。

对于`[] == ![]` 情况和上面一样，相当于比较`[] == false`.

对于`null == true` 和 `null == false` 只能匹配到`Type(y)` 是`Boolean`这条，转换成数字，之后再没有规则能转换了，只能`return false`。










