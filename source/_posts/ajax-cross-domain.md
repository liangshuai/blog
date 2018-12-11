title: Ajax跨域问题解决方案
date: 2015-02-11 09:50:43
tags:
- Ajax
- cross domain
- jsonp
- Access-Control-Allow-Origin
---
JavaScript出于同源策略[Same-origin policy](http://en.wikipedia.org/wiki/Same-origin_policy)，对跨域问题作了限制，不同域的客户端脚本是不能发送请求其它域。同源策略是浏览器最核心也是最基本的安全功能，所有的浏览器(注意这里说的是浏览器，移动开发中的WebView并不是浏览器，所以是可以跨域的!)都已经实现了同源策略。但是项目中经常因为各种原因，可能会遇到跨域问题。

<!-- more -->

###怎么样算是跨域?

* 协议不同(比如HTTP协议和HTTPS协议)
* 域名不同(比如 a.com和 b.com，也包含子域的情况，例如 a.com和 www.a.com)
* 端口不同(例如 http://a.com/ 和 http://a.com:8080/)

满足以上任意一种情况均属于跨域

###怎么解决跨域问题

* 通过CORS方式
* 通过JSONP方式
* 通过Apache/Nginx反向代理方式
* 通过动态构建Script

#### 通过CORS方式

[CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing)是新的[W3C](http://www.w3.org/TR/cors/)策略,它定义了在跨域访问资源时浏览器和服务器之间如何通信。
CORS的基本思想是使用自自定义的HTTP头部允许浏览器和服务器之间相互了解对象，从而
决定请求或响应成功与否。浏览器在发送POST请求的时候会在Request Headers里面包含Origin属性
来向服务器端标识请求是从哪里发起的。服务器端对CORS支持是通过Access-Control-Allow-Origin
来进行的，如果Access-Control-Allow-Origin里面允许来自Origin的请求，那么就可以实现跨域。
下面用Java 的[Spark](http://sparkjava.com/)微框架+Velocity举例说明

源码在 https://github.com/liangshuai/cors_demo


``` java

public class App
{
    public static void main( String[] args )
    {
      port(80);
      staticFileLocation("/public");
      get("/", (req, res) -> {
        Map<String,Object> attributes = new HashMap<>();
        attributes.put("message", "Spark World");
        attributes.put("templateName", "index.vm");
        return new ModelAndView(attributes, "layout.vm");
      },new VelocityTemplateEngine());
    }
}

```

上面的代码会在80端口上启动一个Server，仅仅包含一个页面，页面上包含一个按钮，点击该按钮会向http://localhost:8080/cors 发送Ajax Post请求，很明显端口号不同，所以是跨域的，默认情况下

Ajax Request代码如下


``` javascript

$(".btn-primary").click(function(){
     $.post("http://localhost:8080/cors",function(result){
      $("p").text(result);
    });
  });

```

![cors demo ](https://ooo.0o0.ooo/2017/03/04/58ba094dd12a2.jpg)

监听8080端口的Server代码如下

``` java

public class CorsServer
{
    public static void main( String[] args )
    {
        port(8080);
        post("/cors", (req, res) -> {
          /*res.header("Access-Control-Allow-Origin", "*");*/
          return "Response from cross domain";
        });

    }
}

```

上面的*代表通配符，匹配所有来源的Request



此时点击"Send CORS Request"按钮，会发现Chrome Console中提示

![Access-Control-Allow-Origin](https://ooo.0o0.ooo/2017/03/04/58ba097d187aa.jpg)

取消CorsServer 中的注释，并且再次运行，重新点击按钮会发现

![ajax cors](https://ooo.0o0.ooo/2017/03/04/58ba09abc9e26.jpg)

可以看到已经可以成功POST请求。

这个例子简单演示可CORS的原理，实际项目中这样用是非常危险的，关于CORS的安全防范可以参考[HTML5安全风险详析之一：CORS攻击](http://blog.csdn.net/hfahe/article/details/7961566)

#### 通过JSONP方式

[JSONP](http://en.wikipedia.org/wiki/JSONP) 是JSON的一种使用模式，可以用来解决主流浏览器的跨域数据访问问题，也就是上面所说的同源策略。JSONP实质上并不是使用Ajax请求，而是想引入正常的JS文件(引用普通的JS文件、图片等都不存在跨域问题)一样，
只不过这个JS文件是动态生成，关于JSONP的详细介绍了一看[说说JSON和JSONP，也许你会豁然开朗，含jQuery用例](http://www.cnblogs.com/dowinning/archive/2012/04/19/json-jsonp-jquery.html) ,这篇博文中详细介绍了JSONP的原理和用法，写的非常详细。

#### 通过Apache/Nginx反向代理方式

反向代理的方式原理是让Ajax请求同域下的资源，该资源再被反向代理拦截，转发至异域，这样就绕开了浏览器的限制。

使用反向代理的方式不仅仅可以用来解决跨域问题，还可以用来解决一下问题:
* 请求的统一控制，包括设置权限、过滤规则等
* 隐藏内部服务真实地址，暴露在外的只是反向代理服务器地址
* 实现负载均衡，内部可以采用多台服务器来组成服务器集群，外部还是可以采用一个地址访问
* 作为真实服务器的缓冲，解决瞬间负载量大的问题

仍以前面的例子为例，修改App类的让其监听8000端口

```java

port(8000);
staticFileLocation("/public");

```

同时注释掉CorsServer类Access-Control-Allow-Origin 所在行。

以Nginx为例，设置反向代理，修改conf/nginx.conf里面配置

``` conf

server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass    http://localhost:8000/;
        }
        location /cors{
            proxy_pass http://localhost:8080/cors;
        }

        ...

}

```

同时修改Ajax Request 的URL

``` javascript

$(".btn-primary").click(function(){
     $.post("http://localhost/cors",function(result){
      $("p").text(result);
    });
  });

```

启动Nginx之后打开 http://localhost/ 测试,可以看到能够正确显示8080的Response内容
