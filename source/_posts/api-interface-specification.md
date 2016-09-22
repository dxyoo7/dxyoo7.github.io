---
title: api接口规范
date: 2016-07-07 16:27:10
tags:
---
# 转

# API接口规范完整版本

## 整体规范建议采用RESTful 方式来实施。

## 协议

API与用户的通信协议，总是使用HTTPs协议，确保交互数据的传输安全。

## 域名

应该尽量将API部署在专用域名之下。
https://api.example.com

如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下。
https://example.org/api/

## api版本控制

应该将API的版本号放入URL。
https://api.example.com/v{n}/
另一种做法是，将版本号放在HTTP头信息中，但不如放入URL方便和直观。Github采用这种做法。

> * 采用多版本并存，增量发布的方式
v{n} n代表版本号,分为整形和浮点型
整形的版本号: 大功能版本发布形式；具有当前版本状态下的所有API接口 ,例如：v1,v2
浮点型：为小版本号，只具备补充api的功能，其他api都默认调用对应大版本号的api 例如：v1.1 v2.2

## API 路径规则

路径又称"终点"（endpoint），表示API的具体网址。
在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数。
举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样。

https://api.example.com/v1/products
https://api.example.com/v1/users
https://api.example.com/v1/employees

## HTTP请求方式

对于资源的具体操作类型，由HTTP动词表示。
常用的HTTP动词有下面四个（括号里是对应的SQL命令）。
GET（SELECT）：从服务器取出资源（一项或多项）。
POST（CREATE）：在服务器新建一个资源。
PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
DELETE（DELETE）：从服务器删除资源。

下面是一些例子。

GET /product：列出所有商品
POST /product：新建一个商品
GET /product/ID：获取某个指定商品的信息
PUT /product/ID：更新某个指定商品的信息
DELETE /product/ID：删除某个商品
GET /product/ID/purchase ：列出某个指定商品的所有投资者
get /product/ID/purchase/ID：获取某个指定商品的指定投资者信息
过滤信息

如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。

## 下面是一些常见的参数。


?limit=10：指定返回记录的数量
?offset=10：指定返回记录的开始位置。
?page=2&per_page=100：指定第几页，以及每页的记录数。
?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
?producy_type=1：指定筛选条件


## API 传入参数

参入参数分为4种类型：


地址栏参数
* restful 地址栏参数 /api/v1/product/122 122为产品编号，获取产品为122的信息
* get方式的查询字串 见过滤信息小节
请求body数据
cookie
request header
cookie和header 一般都是用于OAuth认证的2种途径

## 返回数据

只要api接口成功接到请求，就不能返回200以外的HTTP状态。
为了保障前后端的数据交互的顺畅，建议规范数据的返回，并采用固定的数据格式封装。

接口返回模板：

```JSON
{
status:0,
data:{}||[],
msg:’’
}
status: //接口的执行的状态
```


```Json
=0表示成功
<0 表示有异常 
》0 表示接口有部分执行失败
```

Data 接口的主数据

> * 可以根据实际返回数组或JSON对象

Msg
```Json
当status!=0 都应该有错误信息
```
# 非Restful Api的需求


由于实际业务开展过程中，可能会出现各种的api不是简单的restful 规范能实现的，因此，需要有一些api突破restful规范原则。特别是移动互联网的api设计，更需要有一些特定的api来优化数据请求的交互。

## 页面级的api

把当前页面中需要用到的所有数据通过一个接口一次性返回全部数据

## 举例


api/v1/get-home-data 返回首页用到的所有数据

这类API有一个非常不好的地址，只要业务需求变动，这个api就需要跟着变更。

## 自定义组合api


把当前用户需要在第一时间内容加载的多个接口合并成一个请求发送到服务端，服务端根据请求内容，一次性把所有数据合并返回,相比于页面级api，具备更高的灵活性，同时又能很容易的实现页面级的api功能。
规范

地址：api/v1/batApi

传入参数：

```Json
data:[
{url:'api1',type:'get',data:{...}},
{url:'api2',type:'get',data:{...}},
{url:'api3',type:'get',data:{...}},
{url:'api4',type:'get',data:{...}}
]
```

## 返回数据

```Json
{status:0,msg:'',
data:[
{status:0,msg:'',data:[]},
{status:-1,msg:'',data:{}},
{status:1,msg:'',data:{}},
{status:0,msg:'',data:[]},
]
}
```
