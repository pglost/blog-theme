---
layout: post
title:  "博客重构总结"
date:   2015-09-08 13:30:54
categories: design pattern
banner-url: "/public/images/post-banner/learn-by-making.jpg"
---

* content
{:toc}

## 概述

最近花了大概一周的时间，写了一个[博客](http://fullstacks.info/blog-demo/)的demo，使用github托管。除了使用jekyll，没有用到其它任何前端框架，也算是锻炼了裸写html和css的能力。本文对搭建博客中使用的技术要点加以总结，并且后期计划依照demo重构本博客。

## 清除浮动

### 什么是 clearfix ？

clearfix 是一种 CSS 技巧，可以在不添加新的 html 标签的前提下，解决让父元素包含浮动的子元素的问题以及外边距重叠问题。[Micro Clearfix](http://nicolasgallagher.com/micro-clearfix-hack/)给出了一个完整的解决方案,并且兼容IE6/7。这里不考虑IE6/7的兼容性问题，只讨论clearfix的核心内容。

### 代码

	.clearfix:before,
	.clearfix:after
	{
	 	content: "   ";
	 	display: table;
	}
	.clearfix:after {
		clear: both;
	}

### 子元素浮动问题

子元素浮动指的是如果一个父元素的子元素都设置为浮动，那么这个父元素的高度都为零，从而给页面的布局带来很多问题.

导致这个问题的原因是设置为浮动的子元素脱离了正常的文档流，从而导致父元素的高度塌陷。

因此，解决这个问题的一种思路是在子元素的最后，插入一个伪元素，并且为这个元素设置清除左右浮动，从而重新撑起父元素的高度。

代码：

	.clearfix:after
	{
	 	content: "   ";
	 	display: table;
	 	clear: both;
	}

QA：
为什么这里display设置为table，设置也block可不可以？

实际在这种场景下设置为block是可以的，但是为了兼容外边距重叠的场景，这里设置为table而不设置成block。


### 外边距重叠问题

外边距重叠也是布局中很头疼的一个问题。外边距重叠所带来的一个问题是给父元素的第一个子元素设置margin-top，会导致这个margin-top溢出父元素的box，从表现上来看设置这个margin-top值是没有起到任何作用的。同理，对于最后一个子元素设置margin-bottom也同样会溢出。

解决方案类似于子元素浮动问题，即在子元素的头尾分别插入两个伪元素即可。

代码：

	.clearfix:before,
	.clearfix:after
	{
	 	content: "   ";
	 	display: table;
	}

###总结

综合以上两种使用场景，合并后的代码为：

	.clearfix:before,
	.clearfix:after
	{
	 	content: "   ";
	 	display: table;
	}
	.clearfix:after {
		clear: both;
	}

如果想兼容IE6/7，可以继续添加如下代码：

	.clearfix {
	    *zoom: 1;
	}


## 如何设置大背景图

### 为元素添加背景图

	background-url = url("your path");

### 设置背景图完全覆盖背景区域
	background-size : cover;

### 设置背景图的起始位置

当背景区域缩小到比背景图还要小时，可以选择让背景图的哪部分显示出。

	background-positon: 50% 50%;

##CSS优先级计算

CSS的优先级在英文中所对应的单词为[Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)，直译过来应该是精确度，这正好阐述了优先级计算所依据的基本准则，具体来说：

1. 定位DOM元素越精确的CSS优先级越高
2. 优先级相同时，后加载的CSS可以覆盖以前的样式
3. 使用!important标记的CSS元素最高

## 如何为元素添加过度效果

[CSS过渡](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Using_CSS_transitions)(transition)，是 CSS3 规范的一部分，用来控制 CSS 属性的变化速率。可以让属性的变化过程持续一段时间，而不是立即生效。

[blog-demo](http://fullstacks.info/blog-demo)中的博客列表中的图片链接就实现了一个鼠标悬停的过渡的效果。

HTML结构如下

	<div class="post">
	  <a class="thumbnail">
	    <img>
	  </a>
	</div>

CSS样式如下

	.thumbnail {
	  display: block;
	  position: relative;
	}

	.thumbnail:before {
	  display: block;
	  content: " ";
	  position:absolute;
	  left: 0px;
	  top: 0px;
	  width: 100%;
	  height: 100%;
	  background: #000;
	  border-top-left-radius: 5px;
	  border-top-right-radius: 5px;
	  opacity: 0;
	  transition: opacity 1s;
	}

	.thumbnail:hover:before {
	   opacity: 0.6;
	}

主要的实现思路是在图片前添加一个脱离文档流的伪元素覆盖在图片上，鼠标悬停时改变伪元素背景的透明度，并且为这种样式的改变添加了过渡效果。

简写语法：

	div {
	    transition: <property> <duration> <timing function> <delay>;
	}

## 为header添加透明样式

为了给header添加如下的效果，需要涉及到[z-index](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index)属性。

![透明header](/public/images/post-content/2/z-index.png)

实现的思路是把header下面的banner元素的margin-top的值设置为负的header的高度，这样就会导致header中的背景被覆盖。

![透明header](/public/images/post-content/2/z-index2.png)

header元素的CSS如下：

	.header {
	  height: 50px;
	  background: rgba(0,0,0,.3);
	  position: relative;
	  z-index: 1;
	}

## 为footer添加font-awesome图标

[Font Awesome](http://fortawesome.github.io/Font-Awesome/)是一款很流行的字体图标工具。在我的blog-demo中使用这个工具为footer添加好看的图标。

![footer效果](/public/images/post-content/2/font-awesome.png)

使用步骤：

1. 去[官网](http://fortawesome.github.io/Font-Awesome/)下载

2. 进入下载的文件夹，根据个人需要选择使用css/less/scss,将fonts文件夹和你选用的css/less/scss文件夹拷贝到项目的目录下

3. 将font-awesome样式文件链接到hmtl文件，根据如下语法选用不同的图标。可以到官网的[Icons](http://fortawesome.github.io/Font-Awesome/icons/)页面寻找你需要的图标名替换icon-name

	<i class="fa fa-icon-name"></i>

4. 为图标设置样式，主要是设置字体颜色，背景，居中等

我的footer代码

HTML:

	<div class="footer">
	  <div class="container clearfix">
	    <ul class="icon-wrapper">
	      <li class="icon github">
	        <a href="http://github.com/happyrain">
	          <i class="fa fa-github-alt"></i>
	        </a>
	      </li>
	      <li class="icon email">
	        <a href="mailto:xiyuvip@gmail.com">
	          <i class="fa fa-envelope-o"></i>
	        </a>
	      </li>
	      <li class="icon weibo">
	        <a href="http://weibo.com/vipxiyu">
	          <i class="fa fa-weibo"></i>
	        </a>
	      </li>
	    </ul>
	  </div>
	</div>

CSS:

	.footer {
	  height: 200px;
	  background: #292d35;
	  margin-top: 50px;
	}
	.footer .icon-wrapper{
	  margin-top: 30px;
	}
	.footer .icon {
	  width: 40px;
	  height: 40px;
	  border-radius: 50%;
	  line-height: 40px;
	  text-align: center;
	  float: left;
	  margin-right: 10px;
	}
	.icon.github {
	  background: #2196f3;
	}
	.icon.weibo {
	  background: #d32f2f;
	}
	.icon.email {
	  background: #00bcd4;
	}
	.footer .icon a {
	  color: #fff;
	}

##为博客添加favicon

在head中添加如下代码即可实现：

	<link rel="short icon" type="image/x-icon"href="images/favicon.ico">

效果：

![favicon](/public/images/post-content/2/favicon.png)

title前面多了个可爱的小图标^-^

##响应式设计

blog-demo中涉及到的响应式设计主要集中在两方面：

1. 使用[media query](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Media_queries)技术相应不同的屏幕尺寸
2. 使用相对长度单位

当然，在head中设置viewport也是必不可少的：

	<meta name="viewport" content="width=device-width, initial-scale=1">

pc上的显示效果：

![responsive-pc](/public/images/post-content/2/responsive-pc.png)

ipad上的显示效果：

![responsive-ipad](/public/images/post-content/2/responsive-ipad.png)

iphone6plus上的显示效果：

![responsive-ip6plus](/public/images/post-content/2/responsive-ip6plus.png)

iphone6上的显示效果：

![responsive-ip6](/public/images/post-content/2/responsive-ip6.png)

iphone5上的显示效果：

![responsive-ip5](/public/images/post-content/2/responsive-ip5.png)

##下一步的计划

1. 把blog-demo迁移到本博客上。

2. 使用glup+react+webpack重新构建一个新的demo














