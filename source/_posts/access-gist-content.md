title: "墙内查看Github Gist内容"
date: 2017-03-22 11:00:32
tags:
- Gist
- Github
---

## 命令行查看
公司电脑不方便翻墙， 但是遇到个问题，正好Gist上有人演示怎么解决这个问题， 可是没有办法直接看到，想了想，之前用过C9提供的在线IDE， 环境位于墙外， 这个IDE自带terminal正好可以拿来用一用,
先需要去https://c9.io/ 登录，直接使用Github账号就可以， 登录了创建Workspace， 进入workspace后就能看到下方的Terminal
<!-- more -->

### 认证获取Token
使用自己的Github账号和密码，获取token
```sh
curl -v -H "Content-Type: application/json" -u myUsername:myPassword -d '{"scopes":["gist"],"note":"gist"}' https://api.github.com/authorizations
```

### 获取指定Gist的内容
```sh
curl -v -H "Authorization: token 上一步返回信息中的TOKEN" https://api.github.com/gists/GistID
```
也可以做一些Gist的操作
### 创建Gist
```sh
curl -v -H "Authorization: token 之前获取的token" -d '{"description": "a gist for a user with token api call","public": true,"files": {"file1.txt": {"content": "String file contents"}}}' https://api.github.com/gists
```
### 更新Gist
```sh
curl -v --request PATCH -H "Authorization: token 之前获取的token" -d '{"description": "updated gist","public": true,"files": {"file1.txt": {"content": "String file contents are now updated"}}}' https://api.github.com/gists/GistID
```
更多的操作可以参考以下链接
https://developer.github.com/v3/gists/#list-a-users-gists

### *更新*
一些在线服务也可以查看

roughdraft.io/gist-id 可以查看任意一个知道Gist ID的Gist

http://www.gistboxapp.com/ 可以管理和创建自己Github账号下面的Gist
