---
layout: post                        ## 文章使用的模板
title: <<React进阶之路>> 第一章 初识React  				## 文章的标题
date: 2018-10-8 11:20:30			## 发布时间
author: "FaceWaller"                ## 作者
categories: React
tags: React
---

开始学习React !!!!!

# React简介 

### React的特点
- 声明式的视图层
	采用javascript语法来声明视图层,因此可以在视图层随意使用各种状态数据
- 简单的更新流程
	只需要定义UI状态,React便会负责把它渲染成最终的UI.从状态到UI这一单向数据流让React组件的更新流程清晰简洁
- 灵活的渲染实现
	React把视图渲染为虚拟DOM(普通的JavaScript对象),再结合其他依赖库把这个对象渲染成不同终端上的UI.
- 高效的DOM操作
	虚拟DOM是普通的JavaScript对象,有了这一隔离层,操作方便了很多.



# ES 6语法简介

## let、const 

>let和const都是块级作用域，const声明一个只读的常量，一旦声明值就不能改变。

## 箭头函数 

> ES6允许使用箭头定义函数

	var f = a => a + 1; 
	等价于
	var f = function(a){
		return a + 1;
	}
	
	
	function foo(){
		this.bar = 1;
		this.f = (a) => a + this.bar;
	}	
	等价于
	function foo(){
		this.bar = 1;
		this.f = (function(a){
			return a + this.bar
		}).bind(this);	
	}
	
如果箭头函数的参数多于1个或者不需要参数,就需要使用一个圆括号代表参数部分
	
## 模板字符串

>模板字符串是增强版的字符串,用反引号标识字符串,除了可以当作普通字符串外,还可以用来定义多行字符串,以及在字符串中嵌入变量.
	
## 解构赋值

>ES6允许按照一定模式从数组和对象中提取值,对变量进行赋值,这被称为解构

	let person = {name: 'Lily', age:4};
	let {name,age} = person;
	name //'Lily'
	age //4
	
	function sum([x,y]){
		return x+y;	
	}
	sum([1,2]); //3
	

## rest参数

>ES6引入rest参数用于获取函数的多余参数

	function languages(lang, ... types){
		console.log(types);
	}
	
	languages('JavaScript','Java','C++');

## 扩展运算符

>扩展运算符是三个点... 他将一个数组转为用逗号分隔的参数序列,类似于rest参数的逆运算

	function sum(a,b,c){
		return a + b + c;
	}
	let numbers = [1,2,3];
	sum(... numbers);     //6

## class

>ES6引入了class(类)这个概念,新的class写法让对象原型的写法更加清晰,也更像传统的面向对象编程语言的写法

	//定义一个类
	class Person{
		
		constructor(name.age){
			this.name = name;
			this.age = age;
		}
		
		getName(){
			return this.name;
		}
		
		getAge(){
			return this.age;
		}
	}
	
	
	//根据类创建对象
	let person = new('Lily',4);
	
	//class之间可以通过extends关键字实现继承
	class Man extends Person{
		
		constructor(name.age){
			super(name,age);
		}
		
		getGender(){
			return 'male';
		}
	}
	
	let man = new Man('Jack',20);
	

## import、export
	
>ES6实现了自己的模块化标准,ES6模块功能主要由两个关键字构成:export和import. export用于规定模块对外暴露的接口,import用于引入其他模块提供的接口


# 开发环境及工具

## Node.js

>Node.js是一个JavaScript运行时,React应用的执行不依赖Node.js环境,但React应用开发编译的过程中用到的很多依赖都需要Node.js环境.

## NPM

>NPM是一个模块管理工具,用来管理模块的发布,下载及依赖关系.


## Create React App


	1. 安装
	npm install -g create-react-app
	通过使用-g参数,将create-react-app安装到了系统的全局环境,这样就可以在任意路径下使用它了.

	2. 创建应用
	create-react-app my-app

	3. 运行应用
	cd my-app
	npm start

# 小结 #

>了解了React的基本理念和主要特性,对React有了一个大概的了解,并创建了基本的app.