---
layout: post                        ## 文章使用的模板
title: React进阶之路 第二章 React基础  				## 文章的标题
date: 2018-10-9 11:5:00				## 发布时间
author: "FaceWaller"                ## 作者
categories: React
tags: React
---

基础知识了解一下 !!!!!

# JSX


## JSX简介

>JSX是一种用于描述UI的JavaScript扩展语法,React使用这种语法描述组件的UI.  
>React认为一个组件应该具备UI描述和UI数据的完整体,不应该将它们分开处理,于是发明了JSX作为UI描述和UI数据之间的桥梁.



## JSX语法

### 基本语法

>JSX的基本语法和XML语法相同,使用成对的标签构成一个树状结构的数据.
	
	const element = (
		<div>
			<h1>hello,world!</h1>
		</div>
	)
	
	
### 标签类型

>JSX语法中,使用的标签类型有两种,DOM类型的标签和React组件类型的标签.
>当使用DOM类型的标签时,标签的首字母必须小写;当使用React组件类型的标签时,组件名称的首字母必须大写.React通过首字母的大小写来判断渲染的是一个DOM类型的标签还是React组件类型的标签.

	//DOM类型标签
	const element = <h1>Hello,world!</h1>
	
	//React组件类型标签
	const element = <HelloWorld />
	
	//二者可以互相嵌套使用
	const element = (
		<div>
			<HelloWorld />
		</div>
	)

### JavaScript表达式

>JSX可以使用JavaScript表达式,因为JSX本质上仍然是JavaScript.
>表达式在JSX中使用场景主要有两个:通过表达式给标签属性赋值和通过表达式定义义子组件.


### 标签属性

>当JSX标签是DOM类型的标签时,对应DOM标签支持的属性JSX也支持(class要写成className).React对DOM标签支持的事件重新做了封装,采用了更常用的驼峰命名法命名事件.

	<div id='content' className='foo' onClick={ () => {console.log('Hello,World')} } />
	
>当JSX标签是React组件类型时,可以任意自定义标签的属性名
	
	<User name='React' age='4' address='America' />

### 注释
	
>JSX中的注释需要用大括号将/**/包裹起来
	
	const element= (
		<div>
			{/* 这里是注释 */}
			<span>React</span>
		</div>
	)
	


## JSX不是必需的

>JSX不是必须的,它只是React.createElement(component,props,...children)的语法糖,所有的JSX语法最终都会被转换成这个方法调用.

	//JSX语法
	const element = <div className='foo'>Hello,React</div>
	
	//转换后
	const element = React.createElement('div',{className:'foo'}, 'Hello,React');
	

# 组件


## 组件定义

>组件是React的核心概念,是React应用程序的基石,组件将应用的UI拆分成独立的可复用的模块,React应用正是由一个一个组件搭建而成的.  
>定义一个组件有两种方式,使用ES6 class(类组件)和使用函数(函数组件).
	
>使用class定义组件需要满足两个条件:    
>1.class继承自React.Component  
>2.class内部必须定义render方法,render方法返回代表该组件UI的React元素


## 组件的props

>组件的props属性用于把父组件中的数据或方法传递给子组件,供子组件使用.

## 组件的state

>组件的state是组件内部的状态,state的变化最终将反映在组件UI的变化上.  
>this.state定义组件的初始状态,并通过this.setState方法改变组件状态,进而组件UI也会随之重新渲染.  
>React组件正式由props和state两种类型的数据驱动渲染组件UI.props是组件对外的接口,state是组件对内的接口


## 有状态组件和无状态组件

>如果一个组件的内部状态是不变的就用不到state,这样的组件称为无状态组件.反之需要使用state来保存变化的就称之为有状态组件.  
>定义无状态组件除了使用ES6 class的方式外,还可以用函数定义.

	function Welcom(props) {
		return <h1>Hello,{props.name}</h1>
	}
	
>开发React应用时,要认真思考哪些组件设计成有状态哪些设计成无状态,并且尽可能多地使用无状态组件,无状态组件更容易被复用.

## 属性校验和默认属性

>React提供了PropTypes这个对象,用于校验组件属性的类型.

	//为属性指定类型

	import PropTypes from 'prop-types';
	
	class TestItem extends React.Component{
		//......
	}
	
	TestItem.propTypes = {
		post: PropTypes.object,
		onVote: PropTypes.func
	};
	
	

## 组件样式

>为组件添加样式的方法主要有两种:外部CSS样式表和内联样式.

### 外部CSS样式表

>CSS样式表中根据HTML标签类型,ID,class等选择器定义元素的样式,使用className座位选择器,然后在CSS样式表中定义组件的样式.

	function Welcome(props){
		return  <h1 className='foo'>Hello,{props.name}</h1>;
	}
	
	//style.css
	.foo {
		width:100,
		height:50
	}


### 内联样式

>用js对象表示CSS样式,通过DOM类型节点的style属性引用相应样式对象.

	function Welcome(props){
		return  <h1 style = {
			width:100,
			height:50
		}>
		Hello,{props.name}
		</h1>;
	}
>注意:使用内联样式时,样式的属性名必须使用驼峰格式的命名.


## 组件和元素

>React组件和元素这两个概念非常容易混淆.  
>React元素是一个普通的JavaScript对象,这个对象通过DOM节点或React组件描述界面是什么样子.


# 组件的生命周期

>组件从被创建到被销毁的过程称为组件的生命周期,React为组件在不同的生命周期阶段提供不同的生命周期方法.  
>组件的生命周期可以被分为三个阶段:挂载阶段,更新阶段,卸载阶段.

## 挂载阶段

>这个阶段组件被创建,执行初始化,并被挂载到DOM中,完成组件的第一次渲染,依次调用的生命周期方法有:  
>constructor  
>componentWillMount  
>render  
>componentDidMount  

	constructor
	这是ES6 class的构造方法,组件被创建时,会首先调用它.
	
	componentWillMount
	这个方法在组件被挂载到DOM前调用,切只会被调用一次.
	
	render
	这是定义组建时唯一必要的方法(其他生命周期方法都可以省略),这个方法返回一个React元素,用于描述UI.
	
	componentDidMount
	在组件被挂载到DOM后调用,且只会被调用一次.
	

## 更新阶段

>组件被挂载到DOM后,组件的props或state与组件更新有关.  
>父组件的render会引起组件的更新,props可能会变化.  
>state通过this.setState修改组件state来触发组件更新.

>组件更新,依次调用的生命周期方法有:  

	componentWillReceiveProps(nextProps)    该方法只有父组件引起更新时才会调用  
	shouldComponentUpdate(nextProps,nextState)   返回bool,用来决策是否继续更新组件的过程,用这个方法减少不必要的渲染,优化组件性能
	componentWillUpdate(nextProps,nextState)    很少用到  
	render    
	componentDidUpdate  组件更新后调用

## 卸载阶段

>componentWillUnmount方法在组件被卸载前调用,可以在这里执行一些清理工作


# 列表和Keys

>略

# 事件处理


><button onClick={clickButton}> //clickButton是一个函数  
>Click  
></button>

>使用箭头函数  
	<button onClick={(event) => {  
		alert(123);  
	}}>  
	Click  
	</button>  



# 表单
>略



# 小结
>本章介绍了React的主要特性及用法.
>组件是React的核心,根据组件的外部接口props和内部接口state完成自身UI的渲染.
>需要理解组件的生命周期