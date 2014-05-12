---
layout: post

title: 改造WordPress
subtitle: 一个庞大而复杂的CMS

excerpt: 让WordPress更好用，更方便, 更爽

author:
  name: 褚福玺
---

## WordPress

WordPress庞大而繁杂的系统注定它是个不好使用的系统，DashBoard的界面真心让我适应了很长一段时间.鉴于其开源可配置
的特性，这也算勉强可以接受.
为了公开型，所见即所得，只能委屈自己来使用这个庞大而复杂的系统

* * * 

## Markdown集成
一旦使用了上了Markdown，就会上瘾.
这种写作的快感，真是无法阻挡...

所以第一个需要的是在WordPress里面支持Markdown.
[WP-Markdown](http://wordpress.org/plugins/wp-markdown/)
虽然有点丑，但总算是功能齐全了.安装完成以后记得在Setting中修改下设置，一般用来写写文章，评论什么的，当然搞Page也没有问.

* * *

## Personal Image
使用[Gravatar](http://www.gravatar.com/site/signup)系统
Gravatar会关联你注册的电子邮件和头像，这样在网络上，wordpress会直接使用你的Gravatar头像。当然，你必须确保你注册Wordpress
的电子邮件和Gravatar的一致

1. 首先进入Gravatar官网注册
2. 填写一个你常使用的Email地址（注意: 此邮箱地址和你绑定wordpress博客的邮箱,必须一致!）,用于接受gravatar激活信息邮件，点击按钮发送激活邮件。
3. 进入接受激活的邮箱里，点击激活链接进入Gravatar网站进行注册个人用户登录身份。
4. 验证用户进入GravatarManage页面，点击“add a new image”
5. 进入头像上传页面选择第一个”My Computer’s Hard Drive” (从计算机硬盘上传)按钮,也可以选择其他按钮选择从网络上，摄像头，或以前的照片。
6. 上传成功后利用站内工具可裁剪修改头像属性然后提交。
7. 选择图片的分级类型：G(适用于额定gravatar G对显示在所有的网站都有任何观众类型)，PG(额定gravatars可能包含粗鲁的手势,挑逗穿着个人,较小的一个员工说粗话,或轻微的暴力)，R(额定gravatars可能包含诸如严厉的脏话,激烈的暴力、裸体、或硬用药)，X(额定gravatars X可能包含前卫性意象或非常令人不安的暴力)，一般适用于G类型。

