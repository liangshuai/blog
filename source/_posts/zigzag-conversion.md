title: LeetCode算法题 -- Zigzag Conversion
date: 2016-07-06 18:10:09
tags:
- Leetcode
- Algorithm
- Zigzag
---

原题：

The string "PAYPALISHIRING" is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

```
P   A   H   N
A P L S I I G
Y   I   R
```

<!-- more -->

And then read line by line: "PAHNAPLSIIGYIR"
Write the code that will take a string and make this conversion given a number of rows:

string convert(string text, int nRows);
convert("PAYPALISHIRING", 3) should return "PAHNAPLSIIGYIR".

题目的大意是给定一个字符串和行数，然后之字形转换字符串，按行输出结果。 比如"zigzagconversion"转换成4行的之字形如下：

![ZigZag Conversion](http://7u2s4m.com1.z0.glb.clouddn.com/zigzag-conversion.png)

按行输出的结果就是`zcsigoriganeozvn`.

要求按行输出的话就要确定每一行的字符序列， 通过观察发现第一行和最后一行的规律是相邻的相隔 `2 * rowNum - 2` 个，

中间的那些行比较特殊， 如上图中的 `g` 、 `r` 、 `a`、 `e` , 除了这几个之外其余的也是遵循相邻的相隔`2 * rowNum - 2`个， 进一步观察发现第 2 行 中的 `g` 、 `r` 依次是`o`、`i` 减 2， 第 3 行 中的 `a` 、 `e` 依次是`n`、`o` 减 4，  即 第 i 行的元素依次是`i`、 `i + 2 * rowNum - 2 - 2 * i`、 `i + 2 * (2 * rowNum - 2) - 2 * i`、`...`.

最终的JavaScript实现如下：

```js
/**
 * @param {string} s
 * @param {number} numRows
 * @return {string}
 */
var convert = function(s, numRows) {
   var len = s.length;
   var result = [];
   var push = [].push;
   var index = 0;
   var skip = 2 * numRows - 2;
   var i = 0;

   if(numRows === 1) {
   		return s;
   }

   for(; i < numRows; i++) {
   		index = i;
   		while(index < len) {
   			push.call(result, s[index]);
   			(i !== 0 && i !== numRows -1 && (index + skip - 2 * i < len)) && (push.call(result, s[index + skip - 2 * i]));
   			index += skip;
   		}
   }
   return result.join('');
};
```

时间复杂度为O(n).



