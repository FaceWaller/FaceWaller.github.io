---
layout: post                        ## 文章使用的模板
title: <<React进阶之路>> 第五章 虚拟DOM和性能优化 		## 文章的标题
date: 2018-11-2 17:10:00			## 发布时间
author: "FaceWaller"                ## 作者
categories: React
tags: React
---

需要好好理解的内容!!!!!

# 虚拟DOM
>虚拟DOM是和真实DOM相对应的,真实DOM也就是平时我们所说的DOM,它是对结构化文本的抽象表达.  
>在web环境中,其实就是对HTML文本的一种抽象描述,HTML元素的层级关系也会体现在DOM节点的层级上,所有的这些DOM节点构成一棵DOM树.  
>传统的前端开发,通过调用浏览器提供的一组API直接对DOM执行增删改查的操作.这些操作看似只执行了一条JavaScript语句,但它们的执行效率要比执行一条普通的JavaScript语句慢的多.  
>操作DOM效率低下的问题,可以通过增加一层抽象去解决,虚拟DOM就是这层抽象,简历在真实DOM之上,对真实DOM的抽象.  
>虚拟DOM使用普通的JavaScript对象来描述DOM元素,例如
	
	//一个DOM结构
	<div className='foo'>
		<h1>Hello React</h1>
	</div>
	
	//可以用一个JavaScript对象来表达
	{
		type:'div',
		props:{
			className:'foo',
			children:{
				type:'h1',
				props:{
					children:'Hello React'
				}
			}
		}
	}
	
	

# Diff算法
>每次组件的状态或属性更新,组件的render方法都会返回一个新的虚拟DOM对象,用来表述新的UI结构.  
>每次render,React会通过比较两次虚拟DOM结构的变化找出差异部分,更新到真实DOM上,从而减少最终要在真实DOM上执行的操作,提高程序的执行效率.这一过程就是React的调和过程,其中关键是比较两个树形结构的Diff算法.  


>正常情况下,比较两个树形结构差异的算法的时间复杂度是O(N^3),这个效率是无法接受的.  
>React通过总结DOM的实际应用场景提出了两个在绝大多数场景都成立的假设,基于这两个假设实现了O(N)时间复杂度内完成两棵虚拟DOM树的比较.  
>两个假设:
	
	1.如果两个元素的类型不同,那么它们将生成两棵不同的树.  
	2.为列表中的元素设置key属性,用key标识对应的元素在多次render过程中是否发生变化.
	  
>根节点的类型不同,比较的方法也不同:  

>1.当根节点是不同类型时  
从div变成p,ComponentA变成componentB等等,这些都是节点类型发生变化的情况,是很大的变化,React会认为新的树和旧的树完全不同,不会继续再比较其他属性和子节点,而是把整棵树拆掉重建(包括虚拟DOM树和真实DOM树).虚拟DOM的节点类型分为两类:一类是DOM元素类型,一类是React组件类型.DOM树被拆除的过程中,旧的DOM元素类型的节点会被销毁,旧的React组件实例的componentWillUnmount会被调用.在重建的过程中,新的DOM元素会被插入DOM树中,新的组件实例的componentWillMount和componentDidMount方法会被调用.重建后新的虚拟DOM树又会被整体更新到真实DOM树中,这种情况下,需要大量DOM操作,更新效率最低.
	
>2.当根节点是相同的DOM元素类型时  
如果两个根节点是相同类型的DOM元素,React会保留根节点,而比较根节点的属性,然后只更新那些变化了的属性.  
React比较这两个元素,发现只有className属性发生了变化,然后只更新虚拟DOM树和真实DOM树中对应节点的这一属性.

>3.当根节点是相同的组件类型时  
如果两个根节点是相同类型的组件,对应的组件实例不会被销毁,只是会执行更新操作,同步变化的属性到虚拟DOM树上,这一过程组件实例的componentWillReceiveProps和componentWillUpdate会被调用.对于组件类型的节点,React是无法直接指导如何更新真实DOM树的,需要在组件更新并且render方法执行完成后,根据render返回的虚拟DOM结构决定如何更新真实DOM树.接着递归比较子节点.  
但如果在子节点的开始位置新增一个节点  
	<ul>  
	&#160; &#160;<li>first</li>  
    &#160; &#160;<li>second</li>  
	<ul>  
	与  
	<ul>  
    &#160; &#160; <li>third</li>  
	&#160; &#160; <li>first</li>  
	&#160; &#160; <li>second</li>  
	<ul>  
这样first会与third比较,second会与first比较,每个节点都会被修改.为了解决这种低效的更新方式,React提供了key属性,React会根据key来匹配子节点.  
	<ul>  
	&#160; &#160;<li key="first">first</li>  
    &#160; &#160;<li key="second">second</li>  
	<ul>  
	与  
	<ul>  
    &#160; &#160;<li key="third">third</li>  
	&#160; &#160;<li key="first">first</li>  
	&#160; &#160;<li key="second">second</li>  
	<ul>  
有了key就能知道哪个节点是新增的了,避免了不必要的比较.
	
	

# 性能优化
>React通过虚拟DOM和Diff算法等技术极大地提高了操作DOM的效率,大多数场景下,不需要考虑React的性能问题,但还是有一些优化的措施.  

>1.使用生产环境版本的库  
我们使用create-react-app创建的项目,在以npm run start 启动时,使用的是React开发环境版本的React库,有很多debug的内容在里面,体积大且执行速度慢,不适合在生产环境中使用.  
使用生产环境版本的React库,只需要执行npm run build.  

>2.避免不必要的组件渲染  
当组件的props或state发生变化时,组件的render方法会被重新调用,返回一个新的虚拟DOM对象.但在一些情况下,是没有必要重新调用render方法的.例如,父组件的每一次render调用都会触发子组件componentWillReceiveProps的调用,进而子组件的render方法也会被调用,但是这时候子组件的props可能并没有发生改变,所以这一次render是没有必要的,不仅多了一次render方法执行的时间,还多了一次虚拟DOM比较的时间.  
React组件的生命周期方法中提供了一个shouldComponentUpdate方法,这个方法默认返回true,如果返回false组件的这次更新将会停止.我们可以用这个方法来做判断,来阻止不必要的更新.

>3.使用key  
如上文所说,使用key可在更新时减少DOM操作,提高DOM更新效率,当列表元素数量很多时,key的使用更显得重要.


# 性能检测工具
>我们可以通过一些性能检测工具更加方便地定位性能问题.  

>1.React Developer Tools for Chrome
这是一个Chrome插件,用来检测页面使用的React代码是否是生产环境版本,当访问网页时.插件的图标是黑色的则表示生产环境,红色的图标则表示开发环境.  

>2.Chrome Performance Tab  
在开发模式下,可以通过Chrome提供的Performance工具观察组件的挂载更新卸载过程及各阶段使用的时间.使用方式为:  
1.确保应用在开发模式下.  
2.打开Chrome开发者工具,切换到performance窗口,单击Record按钮开始统计.  
3.在页面上执行需要分析的操作,最好不要超过20秒,时间太长会导致Chrome卡死.  
4.单击Stop按钮结束统计,然后在User Timing里查看统计结果.  

>3.why-did-you-update
why-did-you-update会比较组件的state和props的变化,从而发现render方法不必要的调用.  
安装  
	npm install why-did-you-update --save-dev
使用  
	import React from 'react'  
	if(process.env.NODE_ENV !== 'production'){  
	&#160; &#160;const {whyDidYouUpdate} = require('why-did-you-update')  
	&#160; &#160;whyDidYouUpdate(React)  
	}  



# 小结
>本章介绍了React的虚拟DOM机制以及一些优化方面的内容.
