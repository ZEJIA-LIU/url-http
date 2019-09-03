# 浅析url+HTTP
## 1. url的组成
先举个具体的url的例子：
```
https://www.baidu.com/s?wd=hello&rsv_spt=1#5
```
url的组成包括以下几点：

* 协议 https://
* 域名或IP加上端口 www.baidu.com:433 端口为433这里省略了
* 路径 /s
* 查询字符串 ?wd=hello&rsv_spt=1
* 锚点 #5

## 2. HTTP基本概念
HTTP 全称为 HyperText Transfer Protocol 
### 2.1 请求
强求行：

请求动词[(get/post/put/patch/delete) 其中get用来获取内容，post用来上传内容。] + 路径和查询参数(所有路径必须以/开头) + 协议名/版本

请求头：

* Host:域名或ip+端口号
* Accept:表示要接受什么，大部分为text/html
* Content-Type：请求体格式

回车（必须写！！！）

请求体：

也就是上传内容。请求体在get请求中一般为空。


### 2.2 响应
状态行：

协议名/版本 + 状态码(常见的状态码有：200表示ok，404表示加载失败) + 状态字符串

响应头：

Content-type：响应体的格式

回车 （必须写！！！）

响应体：
也就是下载内容。

### 2.3 用curl构造请求
先来一个最简单的请求：
```
curl -v http://127.0.0.1:8888
```
* 设置请求动词 如把请求动词改为POST，只需要加上 -X -POST，具体为：
``` 
curl -v -X -POST http://127.0.0.1:8888
```
* 设置路径和查询参数 直接在curl后面加 具体为:
```
curl -v -X -POST http://127.0.0.1:8888/xx?wd=hi
```
* 设置请求头 -H 'Name:value' 或者 -header 'Name:value' 具体为：
```
curl -v -X -POST -H 'Accept:text/html' -H 'Content-Type:text/plain;charset=utf-8' http://127.0.0.1:8888/xx?wd=hi
```
* 设置请求体 -D '内容' 或者 -data '内容' 具体为：
```
curl -v -D '请求内容' http://127.0.0.1:8888
```
### 2.4 用node.js读取请求
* 读取请求动词：
```
request.method
```
* 读取路径：
1. 路径带查询参数：request.url
2. 纯路径：path
3. 只有查询参数：query

* 读取请求头：
```
request.headers['Accept']
```
### 2.5 用node.js设置响应
* 设置状态行的状态码：
```
response.statusCode=200
```
* 设置响应头：
```
response.setHeader('Content-Type','text/html;charset:utf-8')
```
* 设置响应体：注意点是包住内容的符号不是引号，是反引号。
```
response.write(`内容`)
```

### 2.6 举例
附上一个server.js 可以更好的理解以上知识：

 [我的一个github上的server.js](https://github.com/Wurui-suck/node-demo-1)


## 3. 域名、IP、和端口
### 3.1 IP
IP 全称为Internet Protocol 它主要约定了两件事情：
1. 如何ing为一台设备
2. 如何封装数据报文，与其他设备交流
   
只要在互联网中，你就至少有一个IP。IP又分为内网和外网，由路由器沟通内外网。

如何获得外网IP：
1. 从电信租用宽带
2. 买个路由器，然后用电脑或手机连接路由器广播出来的无线wifi
3. 只要路由器连接上电信的服务器，那么路由器就会有一个外网IP，这就是你在互联网中的地址，如 14.17.32.211
4. 但是如果重启路由器，那么你很可能被重新分配到一个外网IP，也就是说你爹路由器没有固定的外网IP
5. 如果你的路由器的外网IP是 14.17.32.222 那么你的手机和电脑的IP又是什么呢？答案是内网IP。
6. 查看外网IP可以登录 http://ip138.com 查看

如何获得内网IP：
1. 路由器会在你家创建一个内网，内网中的设备用内网IP，一般来说这个格式是 192.168.xxx.xxx
2. 一般路由器会给自己分配一个好记的内网IP，如：192.168.1.1
3. 然后路由器会给每一个内网中的设备分配一个不用的内网IP，如你的电脑是192.168.1.2 你的手机是 192.168.1.3

路由器的功能：

* 路由器有两个IP，一个内网IP，一个外网IP
* 内网中的设备可以和互相访问，但不能直接反访外网
* 内网设备想要访问外网，就必须通过路由器中转
* 外网中的设备可以互相访问，但是不能访问你的内网
* 外网中的设备想要把内容送到内网，也必须通过路由器
* 也就是说内网和外网就像两个隔绝的空间，无法互通，路由器就是唯一的联通点
* 所以路由器有时候也叫网关

几个特殊的IP
* 127.0.0.1 表示自己
* localhost同过host制定自己 host文件的路径为 C:\Windows\System32\drivers\etc 可以通过在里面加入一行127.0.0.1 xxx ，让xxx称为127.0.0.1 的别称
* 0.0.0.0 不表示任何设备

### 3.2 端口 port
一台机器可以提供不同的服务，其中：
* 要提供HTTP服务最好用80端口
* 要提供HTTPS服务最好用443端口
* 要提供FTP服务用21端口
* 一共有65535个端口（基本够用）

端口的使用规则：
* 0到1023端口是留给系统使用的，你有了管理员权限后，才有资格使用这些端口
* 其他端口可以给普通用户使用，如http-server默认使用8080端口
* 一个端口被占用，你就只能换一个

### 3.3 域名
* 域名就是对IP的别称。
* 一个域名可以对应多个IP，这个叫做均衡负载，防止一台机器扛不住
* 一个IP可以有多个域名，这个叫做共享主机，穷开发者才会这么做

ping 命令的使用：

简单来说，「ping」是用来探测本机与网络中另一主机之间是否可达的命令，如果两台主机之间ping不通，则表明这两台主机不能建立起连接。ping是定位网络通不通的一个重要手段。

就是我们玩游戏会有个ping，表示延迟，如果ping值越低，说明你和服务器的连接越好。

DNS： 连接域名和IP
* 但你输入一个域名后，你的chrome浏览器会向电信或者联通的DNS服务器询问改域名对应了什么IP
* 电信或联通会回答一个IP，具体过程很复杂，不研究
* 然后才会向对应IP的443或者80端口发送请求
* 请求内容是查看xxx（你输入的域名）的首页

nslookup: 你可以通过在命令行输入：
```
nslookup baidu.com
```
来查看百度的IP。

域名的分级：
* com/cn/org/io等都是顶级域名。com的全称是company，org是指非营利性组织。
* github.io是二级域名（俗称一级域名）
* username.github.io是三级域名（俗称二级域名）
* 只是github.io把它的子域名们免费提供给用户使用
* 二级域名与三级域名之间是父子关系，所以说 www.xiedamala.com 与  xiedaimala.com 可以是一家公司，也可以不是。

## 4.路径
通过路径，可以做到访问相同域名，但是请求不同页面，如：

https://developer.mozilla.org/zh-Cn/docs/Web/HTML

https://developer.mozilla.org/zh-Cn/docs/Web/CSS

## 5. 查询参数
通过查询参数，可以做到请同一个页面，访问不同内容，如：

http://www.baidu.com/s?wd=hi

http://www.baidu.com/s?wd=hello

## 6. 锚点
通过锚点可以做到，访问相同内容，出现在不同位置，如：

https://developer.mozilla.org/zh-Cn/docs/Web/CSS#参考书

https://developer.mozilla.org/zh-Cn/docs/Web/CSS#教程

注意：
* 锚点看起来有中文，实际不支持中文
* 锚点无法在Network版面看到，因为锚点不会传给服务器，他只跟浏览器有关。