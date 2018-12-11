title: "Express配置HTTPS服务器"
date: 2015-11-04 15:09:32
tags:
- Express
- Node.js
- Server
- HTTPS
---


项目部署到生产环境下，出现了一个Bug，同样的代码在DEV环境下却是好的，怀疑是HTTPS导致的问题，所以在本地搭了一个HTTPS的服务器用来测试。

整个过程还是很简单的，简单记录一下。

<!-- more -->

### 创建一个Express项目

```sh
express -e https-example
cd https-example
npm install
```

### 使用OpenSSL生成证书文件

```sh
openssl genrsa -out privatekey.pem 1024
openssl req -new -key privatekey.pem -out certrequest.csr
openssl x509 -req -in certrequest.csr -signkey privatekey.pem -out certificate.pem
```


### 修改app.js配置HTTPS

```js
var fs = require('fs'),
    https = require('https'),
    express = require('express'),
    app = express();
    app.use(express.static('public'));
    https.createServer({
      key: fs.readFileSync('privatekey.pem'),
      cert: fs.readFileSync('certificate.pem')
    }, app).listen(8888);
```

### 运行Server

```sh
node app.js

```