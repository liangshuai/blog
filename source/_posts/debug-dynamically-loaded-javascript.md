title: Chrome调试动态创建的JS
date: 2016-08-06 09:50:42
tags:
- Chrome
- JavaScript
- Debug
---

Chrome开发者工具的Debug在开发JS时非常有用， 可是现在越来越多的JS框架都会在运行时动态添加或加载JS脚本， 比如很多JS模板引擎， 就可能添加生成的JS文件，这个时候调试起来并不像普通JS那样直接在Chrome Developer Tools 的`Sources` Tab下面直接看到，造成调试脚本的不便。

<!-- more -->

不过Chrome提供了一种简单的方式，可以直接看到添加的脚本，并且可以打BreakPoints。

```html
<!doctype html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>调试动态创建的JS</title>
	<link rel="canonical" href="http://liangshuai.me/" >
</head>
<body>
	
<script type="text/javascript">
(function(){
	var script = document.createElement('script');
	var head = document.getElementsByTagName('head')[0];
    script.setAttribute('type', 'text/javascript');
    script.text = 'var test = 123; \nconsole.log(test);\n//# sourceURL=debugDynamicScripts.js';
    head.appendChild(script);
})();
	
</script>
</body>
</html>
```

上面的例子中`script.text` 就相当于动态生成的JS， 如果想要在Sources里面看到的话 只需要在结尾添加上`\n//# sourceURL=` 后面跟JS的文件名就可以了。这样动态创建JS文件就能在Sources下面的no domain下面看到。

![Debug Dynamic Script](https://ooo.0o0.ooo/2017/03/04/58ba0a459b919.png)

不过这个时候页面DOM中也会被添加了script标签以及脚本， 如果要创建的脚本过多可能造成DOM臃肿。 实际上在`appendChild` 之后就可以直接`removeChild` 掉了。脚本依旧能够看到并且可以执行。

```js
head.appendChild(script);
head.removeChild(script);
```
