---
layout:  post
title:  "发布/订阅者模式"
date:   2015-07-20 14:30:54
categories:  design pattern
banner-url:  "/public/images/post-banner/new-way-out.jpg"
---

* content
{:toc}


## 什么是publish/subscribe模式

publish/subscribe模式其实就是观察者模式，这种模式提供了一种事件管理的机制。在这种机制下，某些对象可以向“事件管理内核”发送订阅(subscribe)“信号”，表示该对象注册了某个类型的事件；另外一些对象可以向“事件管理中心”发送发布（publish）信号，表示要触发某个类型的事件。

## publish/subscribe模式中的事件管理内核

### 代码

	//事件管理对象的构造函数
	function EventManger(){
	    this.handlers = {};
	}
	//为事件管理对象的构造函数添加原型
	EventManger.prototype = {
	    constructor: EventManger,
	//订阅函数，即注册事件
	    subscribe: function(type, handler){
	        if (typeof this.handlers[type] == "undefined"){
	            this.handlers[type] = [];
	        }

	        this.handlers[type].push(handler);
	    },
	//发布函数，即触发事件
	    publish: function(event){
	        if (!event.target){
	            event.target = this;
	        }
	        if (this.handlers[event.type] instanceof Array){
	            var handlers = this.handlers[event.type];
	            for (var i=0, len=handlers.length; i < len; i++){
	                handlers[i](event);
	            }
	        }
	    },
	//移除事件注册
	    remove: function(type, handler){
	        if (this.handlers[type] instanceof Array){
	            var handlers = this.handlers[type];
	            for (var i=0, len=handlers.length; i < len; i++){
	                if (handlers[i] === handler){
	                    break;
	                }
	            }

	            handlers.splice(i, 1);
	        }
	    }
	};

## publish/subscribe模式的应用

下面举一个简单的图书馆书目管理的例子，来说明publish/subscribe模式的作用

### 说明

本例中有三类对象，Book类型的对象代表一条图书记录，BookCollection类型的对象代表所有馆藏图书，里面存有所有图书的记录以及添加删除书籍记录的方法，BookListView类型的对象表示待显示的所有图书的记录。在这里，BookCollection为发布者，BookListView为订阅者。我们期望，在对BookCollection类型的对象进行添加或删除操作时，BookListView类型的对象也能有相应的响应。

### 代码


	//创建事件管理器
	var eventManger = new EventManger();
	//Book类型对象构造函数
	function Book(name, isbn) {
	    this.name = name;
	    this.isbn = isbn;
	}
	//BookCollection类型对象构造函数
	function BookCollection(books) {
	    this.books = books;
	}

	BookCollection.prototype = {
		constructor: BookCollection,

		addBook: function (book) {
	    	this.books.push(book);
	    	eventManger.publish({type:'book-added',info:book});
	   		return book;
			}
		removeBook: function (book) {
	   		var removed;
	   		if (typeof book === 'number') {
	       		removed = this.books.splice(book, 1);
	   			}
	  		for (var i = 0; i < this.books.length; i += 1) {
	      		if (this.books[i] === book) {
	         	 removed = this.books.splice(i, 1);
	      		}
	   		}
	    	eventManger.publish({type:'book-removed',info:book});
	   		return removed;
		}
	}
	//订阅者
	var BookListView = (function () {

	   function removeBook(event) {
	      console.log("removeBook " + " isbn:" + event.info.isbn + "  name:" + event.info.name);
	   }

	   function addBook(event) {
	     console.log("addBook " + " isbn:" + event.info.isbn + "  name:" + event.info.name);
	   }

	   return {
	      init: function () {
	         eventManger.subscribe('book-removed', removeBook);
	         eventManger.subscribe('book-added', addBook);
	      }
	   }
	}());


	var book1 = new Book("JavaScript高级程序设计","9787115000415"),
		book2 = new Book("锋利的jQuery","9787115281609"),
		books = [],
		bookCol = new BookCollection(books);
	//为BookListView订阅'book-removed'和'book-added'事件
		BookListView.init();
	//bookCol添加book1，并发布'book-added'事件
		bookCol.addBook(book1);
		bookCol.addBook(book2);
	//bookCol移除book1，并发布'book-removed'事件
		bookCol.removeBook(book2);

		bookCol.removeBook(book1);

### 结果

![结果](/public/images/post-content/publisher-subscriber-pattern/result.png)


