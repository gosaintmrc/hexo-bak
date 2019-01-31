---
title: nginx的示例操作
date: 2019-01-31 15:09:00
categories: 
- Nginx
tags:  nginx
---

## 1  正则匹配

    使用正则匹配的方式：location的配置如下：

 

```xml
location ~ \.(gif|jpg|png|js|css)$ {  
     root   /usr/local/static/image;
 } 
```

​    图片的位置在/usr/local/static/image目录下；访问的地址为http://10.22.12.229/20160905134018381.jpg

## 2 通用匹配

```
 location /{  
   root   /usr/local/static/image;
 }  
```

​    图片的位置在/usr/local/static/image目录下；访问的地址为http://10.22.12.229/20160905134018381.jpg

## 3 以某个url开头的正则匹配

```
location ^~ /image/ {  
   root   /usr/local/static;  
}  
```


这个访问的路径是：http://10.22.12.229/image/aaa.jpg  。其实际图片的位置放在/usr/local/static/image下面。就是root+/image/为图片的实际路径；如果配置成下面的方式，则会报404。

## 4  错误配置实例

```
location ^~ /image/ {  
   root   /usr/local/static/image;  
}
```

比如访问的css路径为：http://10.22.12.229/plugin-module/questionnaire/css/question.css
可以这样配置：

```
location ^~ /plugin-module/questionnaire/css/ {  
           root   /usr/local/html/;
 }  
```


