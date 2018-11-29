---
layout: post                        ## 文章使用的模板
title: React进阶之路 第七章 用React Router开发单页面应用		 		## 文章的标题
date: 2018-11-27					## 发布时间
author: "FaceWaller"                ## 作者
categories: React
tags: React
---

React中的路由系统.

>真实项目中,一般需要不同的URL标识不同的页面,也就是存在页面间路由的需求,这时候就该React Router发挥作用了.React Router是React技术栈中最常用的用于构建单页面应用的解决方案.

# 单页面应用和前端路由
## 后端路由
>在传统的web应用中,浏览器根据地址栏的URL向服务器发送一个HTTP请求,服务器根据URL返回一个HTML页面.这种情况下,一个URL对应一个HTML页面,一个Web应用包含很多HTML页面,这样的应用就是多页面应用.在多页面应用中,页面路由的控制由服务器端负责,这种路由方式称为后端路由.  

## 多页面应用
>在多页面应用中,每次页面切换都需要向服务器发送一次请求,页面使用到的静态资源也需要重新请求加载,存在一定的浪费.而且页面的整体刷新对用户体验也有影响,因为不同页面间往往存在共同的部分,页面整体刷新也会导致共用部分的刷新.  

## 单页面应用
>单页面应用.让Web应用只是看起来像多页面应用,实际URL的变化可以引起页面内容的变化,但不会向服务器发送新的请求.单页面应用的视觉感受是多页面的,因为URL发生变化,页面的内容也会发生变化,但至少逻辑上的多页面,实际上无论URL如何变化,对应的HTML文件都是同一个.  

## 前端路由
>在单页面应用中,URL发生变化不会向服务器发送新的请求,所以这个逻辑的路由只能由前端负责,这种路由方式称为前端路由.  

## React Router
>React Router就是一种前端路由的实现方式,通过使用React Router可以让Web应用根据不同的URL渲染不同的组件,这样的组件渲染方式可以解决更加复杂的业务场景.



# React Router的安装
>React Router包含3个库:react-router、react-router-dom和react-router-native.在实际使用时,根据环境去选择安装react-router-dom(浏览器)或react-router-native(react-native),他们都依赖react-router,所以react-router会跟着自动安装.

	npm install react-router-dom

# 路由器
>通过Router和Route两个组件完成路由功能,Router可以理解成路由器,一个应用中只需要一个Router实例,所有的路由配置组件Route定义为Router的子组件.在Web应用中,我们一般会使用对Router进行包装的BrowserRouter或HashRouter两个组件.
	
	BrowerRouter创建的URL形式如下:
	http://example.com/some/path
	HashRouter创建的URL形式如下:
	http://example.com/#/some/path
>使用HashRouter时,hash部分的内容会被服务器忽略,真正有效的信息是hash前面的部分,对于单页面来说,这部分内容是固定的.  


>Router会创建一个history对象,history用来跟踪URL,当URL发生变化时,Router的后代组件会重新渲染.这也隐含说明了React Router中的其他组件必须作为Router组件的后代组件使用.但Router中只能有唯一一个子元素

	//正确
	ReactDOM.render((
		<BrowserRouter>
			<App />
		</BrowserRouter>
	), document.getElementById('root'))
	
	//错误,Router中包含两个子元素
	ReactDOM.render((
		<BrowserRouter>
			<App1 />
			<App2 />
		</BrowserRouter>
	),document.getElementById('root'))


# 路由配置
>Route是ReactRouter中用于配置路由信息的组件,也是ReactRouter中使用频率最高的组件.每当有一个组件需要根据URL决定是否渲染时,就需要创建一个Route.

## path
>每个Route都需要定义一个path属性,当使用BrowserRouter时,path用来描述这个Route匹配的URL的pathname.当使用HashRouter时,path用来描述这个Route匹配的URL的hash.当URL匹配一个Route时,这个Route中顶一顶组件就会被渲染出来.反之,不进行渲染.

## match
>当URL和Route匹配时,Route会创建一个match对象作为props的一个属性传递给被渲染的组件.这个对象包含4个属性:

- params:Route的path可以包含参数,例如<Route path='/foo/:id'>包含一个参数id,params就是用于从匹配的URL中解析出path中的参数,当URL="http://example.com/foo/1"时,params={id:1}
- isExact:是一个布尔值,当URL完全匹配时,值为true,当URL部分匹配时,值为false.
- path:Route的path属性,构建嵌套路由时会使用到.
- url:URL的匹配部分

## Route渲染组件的方式
>Route提供3个属性,用于定义待渲染的组件:

- component  
	component的值是一个组件,当URL和Route匹配时,component属性定义的组件就会被渲染.  
	例如: <Route path='/foo' component={Foo}>  
	当URL="http://example.com/foo"时,Foo组件会被渲染.  
	
- render  
	render的值是一个函数,这个函数返回一个React元素.这种方式可以方便地为待渲染的组件传递额外的属性.例如:    
		<Route path='/foo' render={(props) => (  
			<Foo {... props} data={extraProps} />  
		)}>  
	Foo组件接收了一个额外的data属性.
	
- children  
	children的值也是一个函数,函数返回要渲染的React元素.与前两种方式不同之处是,无论是否匹配成功,children返回的组件都会被渲染.但是当匹配不成功时,match属性为null.例如:  
	<Route path='/foo' children={(props) =>  
		(  
			<div className={props.match?'active' : ''}>  
				<Foo />  
			</div>  
		)}>  
	</Route>
	如果Route匹配当前URL,待渲染元素的根节点div的class将被设置成active.
	
- Switch和exact
	当URL和多个Route匹配时,这些Route都会执行渲染操作.如果只想让第一个匹配的Route渲染,那么可以把这些Route包到第一个匹配的Route渲染,那么可以把这些Route包到一个Switch组件中.如果想让URL和Route完全匹配时,Route才渲染,那么可以使用Route的exact属性.Switch和exact常常联合使用,用于应用首页的导航.例如:  
	<Router>  
		<Switch>  
			<Route exact path='/' component={Home} />  //1
			<Route path='/posts' component={Posts} />  //2
			<Route path='/:user' component={User} />   //3
		</Switch>  
	</Router>  
	
	如果不使用Switch,当URL的pathname为"/posts"时,2和3都会被匹配,很明显我们不希望3被匹配,实际上也不会有叫posts的用户.  
	如果不使用exact,几乎所有的URL都会匹配1.
	
- 嵌套路由  
	嵌套路由是指在Route渲染的组件内部定义新的Route.例如,在上一个例子中,在Posts组件内再定义两个Route:  
		const Posts = ({match}) => { 
			return(  
				<div>  
				//这里match.url 等于/posts
					<Route path={`${match.url}/:id`} component={PostDetail} />  
					<Route exact path={match.url} component={PostList} />  
				</div>  
			)  
		}  
	当URL的pathname 为"/posts/react"时,PostDetail组件会被渲染;当URL的pathname为"/posts"时,PostList组件会被渲染.Route的嵌套使用让应用可以更加灵活地使用路由.
	

# 链接
>Link是ReactRouter提供的链接组件,一个Link组件定义了当点击该Link时,页面应该如何路由.例如:

	const Navigation = () => (  
		<header>  
			<nav>  
				<ul>  
					<li><Link to='/'>Home</Link></li>  
					<li><Link to='/posts'>posts</Link></li>  
				</ul>  
			</nav>  
		</header>  
	)
>Link使用to属性声明要导航到的URL地址.to可以是string或object类型,当to为object类型时,可以包含pathname search hash state 四个属性,例如:
	
	<Link to={  
			pathname:'/posts',  
			search:'?sort=name',
			hash:'#the-hash',
			state: { fromHome:true }
		} />

>除了使用Link外,我们还可以使用history对象手动实现导航.history中最常用的两个方法是push(path,[state])和replace(path,[state]),push会向浏览历史记录中新增一条记录,replace会用新纪录替换当前记录.例如:
	
	history.push('/posts')
	history.replace('/posts')

