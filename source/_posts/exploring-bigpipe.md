title: "探索Bigpipe"
date: 2016-12-01 12:09:32
tags:
- Bigpipe
- Node.js
- Koa
- Async

---

网站加载速度的快慢对用户体验是否良好起着至关重要的作用，据研究
> 页面加载时间每增加1s，就会导致浏览量降低11%，客户满意度下降16%，转换率损失7%
> 如果3秒内，网页还未加载完毕，57%的用户会放弃 
> 每延长1秒，亚马逊一年就会减少16亿美元销售额
> 74%的用户登录某网站时间超过5秒后就不会再登录这个网站
> 60%的用户希望手机上的页面加载时间不要超过3秒

由此可见 ，网页加载速度是多么的重要。页面加载速度的优化有很多种途径，本文探讨一下FaceBook提出的`Bigpipe技术`，该技术对于比较复杂的页面具有非常明显的作用。
<!-- more -->
### 基本思路
Bigpipe的基本思路是将页面划分为多个不同的块， 一般来说这些块是相互独立的，如下图所示，每一块称作一个`pagelet`  当用户请求页面时，服务器端分块处理内容，分块输出响应内容，客户端分块显示，从而使得各个块的处理和渲染得以并发进行。
![enter image description here](http://7u2s4m.com1.z0.glb.clouddn.com/image.jpg) 

先来看一下网页加载流程
![page load flow](http://7u2s4m.com1.z0.glb.clouddn.com/%E6%B5%8F%E8%A7%88%E5%99%A8.png)
这个流程从请求到页面完整呈现出来存在大量的资源闲置， 比如浏览器请求结束到浏览器接收到服务器端响应这段时间是完全空闲的， 服务器端也有大量的闲置时间， 使用pagelet之后服务器持续分块输出内容到浏览器，浏览器持续不断的渲染各个pagelet，从而使得加载速度得到提升。
![enter image description here](http://7u2s4m.com1.z0.glb.clouddn.com/4.png)

![enter image description here](http://7u2s4m.com1.z0.glb.clouddn.com/5.png)

### Bigpipe实现原理
要实现BigPipe需要服务器先返回页面的框架结构（没有闭合body和html标签），然后需要能够持续的输出各个pagelet到页面，这就需要使用HTTP1.1 支持的[分块传输编码](https://en.wikipedia.org/wiki/Chunked_transfer_encoding)  ,  如果一个HTTP消息（请求消息或应答消息）的`Transfer-Encoding`消息头的值为`chunked`，那么，消息体由数量未定的块组成，每一块有自己的大小指示器，并以最后一个大小为`0`的块为结束, 这种编码允许发送端能动态生成内容，并能携带让接收端判断消息是否接收完整的有用信息。Big Pipe 完成这样的传输只需要使用一个连接, 不需要额外的请求， 相对于Ajax取数据的方式节省了多次HTTP连接的代价。
对于浏览器端，由于页面的框架结构是`没有闭合body和html标签`的文档, 此时浏览器会把已经接收到的dom渲染出来(如果还有css, 也渲染上).  此时由于TCP连接还没有断开, body和html还没有闭合, 服务器可以继续推送更多的dom或者script到浏览器。
![HTTP Chunked](http://7u2s4m.com1.z0.glb.clouddn.com/BEiVV.gif)

上图是HTTP Chunked的一个例子， 每一个消息体都有一个16进制的大小指示器，并且以大小为0 的结束。

### Node.js 简单实现
Node.js http模块Response的write使用的就是分块传输编码， 这里使用Node.js简单实现一个BigPipe示例
```js
var http = require('http'),
	fs = require('fs'),
	port = 8888,
	server,
	handle;

handle = function(req, res){
	var html = '',
		containers = ['header', 'content', 'footer'];

	res.writeHead(200, { 'Content-Type': 'text/html' });
	html = fs.readFileSync(__dirname + "/layout.html").toString();
	res.write(html);
	containers.forEach(function(item){
		res.write('<script>bigpipe.render("#' + item + '","module ' + item + '");</script>');
	});
	res.write('</body></html>');
	res.end();
}

server = http.createServer(handle);
server.listen(port, function(){
    console.log("Server listening on: http://localhost:%s", port);
});
```

```html
<!doctype html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>BigPipe Testing</title>
	<script type="text/javascript">
		var BigPipe = function() {
			var _render;

			_render = function(selector, html) {
				document.querySelector(selector).innerHTML = html;
			};
			return {
				render: _render
			};
		};
		var bigpipe = new BigPipe();
	</script>

</head>
<body>
	<div id="header"></div>
	<div id="content"></div>
	<div id="footer"></div>
```
