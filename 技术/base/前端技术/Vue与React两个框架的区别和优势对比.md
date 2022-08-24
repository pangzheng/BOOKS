# Vue与React两个框架的区别和优势对比
## 简单介绍

- React

	由 Facebook 创建的 JavaScript UI 框架——React。它支撑着包括 Instagram 在内的大多数 Facebook 网站。React 与当时流行的jQuery,Backbone.js 和 Angular 1等框架不同，它的诞生改变了 JavaScript 的世界。其中最大的变化是 React 推广了 Virtual DOM并创造了新的语法——JSX，JSX 允许开发者在 JavaScript 中书写 HTML（译者注：即HTML in JavaScript）。
- Vue	

	Vue 致力解决的问题与 Reac t一致，但却提供了另外一套解决方案。Vue 使用模板系统而不是JSX，使其对现有应用的升级更加容易。这是因为模板用的就是普通的 HTML，通过 Vue 来整合现有的系统是比较容易的，不需要整体重构。同时 Vue 声称它更容易学习，我最近才接触Vue，能证明所言非虚。
	
关于 Vue 还需要说的是，Vue 主要是由一位开发者进行维护的，而不像 React 一样由如 Facebook 这类大公司维护。

## 相似之处
React与Vue存在很多相似之处，例如他们都是JavaScript的UI框架，专注于创造前端的富应用。不同于早期的JavaScript框架“功能齐全”，Reat与Vue只有框架的骨架，其他的功能如路由、状态管理等是框架分离的组件。

### Virtual DOM
Vue.js (2.0版本)与React的其中最大一个相似之处，就是他们都使用了一种叫 'Virtual DOM' 的东西。所谓的 Virtual DOM 基本上说就是它名字的意思：虚拟 DOM，DOM 树的虚拟表现。它的诞生是基于这么一个概念：改变真实的 DOM 状态远比改变一个JavaScript对象的花销要大得多。

Virtual DOM 是一个映射真实 DOM 的 JavaScript 对象，如果需要改变任何元素的状态，那么是先在 Virtual DOM 上进行改变，而不是直接改变真实的 DOM。当有变化产生时，一个新的 Virtual DOM 对象会被创建并计算新旧 Virtual DOM 之间的差别。之后这些差别会应用在真实的 DOM 上。

相同例子，不同实现的区别
	
- html  

		<ul class="list">
		  <li>item 1</li>
		  <li>item 2</li>
		</ul>
- js

		{
		    type: 'ul', 
		    props: {'class': 'list'}, 
		    children: [
		        { type: 'li', props: {}, children: ['item 1'] },
		        { type: 'li', props: {}, children: ['item 2'] }
		    ]
		}

真实的 Virtual DOM 会比上面的例子更复杂，但它本质上是一个嵌套着数组的原生对象。

当新一项被加进去这个 JavaScript 对象时，一个函数会计算新旧 Virtual DOM 之间的差异并反应在真实的 DOM 上。计算差异的算法是高性能框架的秘密所在，React 和 Vue 在实现上有点不同。

- Vue

	宣称可以更快地计算出 Virtual DOM 的差异，这是由于它在渲染过程中，会跟踪每一个组件的依赖关系，不需要重新渲染整个组件树。

- React

	每当应用的状态被改变时，全部子组件都会重新渲染。当然这可以通过 `shouldComponentUpdate` 这个生命周期方法来进行控制，但Vue 将此视为默认的优化。

小结：如果你的应用中，交互复杂，需要处理大量的 UI 变化，那么使用 Virtual DOM 是一个好主意。如果你更新元素并不频繁，那么 Virtual DOM 并不一定适用，性能很可能还不如直接操控DOM。

### 组件化
React 与 Vue 都鼓励组件化应用。这本质上说是建议你将你的应用分拆成一个个功能明确的模块，每个模块之间可以通过合适的方式互相联系。关于组件化的例子可以在这篇文章的中间部分被找到:

	你可以认为组件就是用户界面中的一小块。如果让我来设计 Facebook 的UI界面，那么聊天窗口会是一个组件，评论会是另一个组件，不断更新的好友列表也会作为一个组件。

- vue

	在 Vue 中，如果你遵守一定的规则，你可以使用单文件组件.

		//PastaItem.vue
		
		<template>
		<li class="pasta-dish list-unstyled">
		    <div class="row">
		        <div class="col-md-3">
		            <img :src="this.item.image" :alt="this.item.name" />
		        </div>
		        <div class="col-md-9 text-left">
		            <h3>{{this.item.name}}</h3>
		            <p>
		                {{this.item.desc}}
		            </p>
		            <button v-on:click="addToOrderNew" class="btn btn-primary">Add to order</button> <mark>{{this.orders}}</mark>
		        </div>
		    </div>
		</li>
		</template>
		
		<script>
		
		export default {
		    name: 'pasta-item',
		    props: ['item'],
		    data:  function(){
		        return{
		            orders: 0
		        }
		    },
		    methods: {
		        addToOrderNew: function(y){
		            this.orders += 1;
		            this.$emit('order');
		        }
		    }
		}
		
		</script>
		
		<style src="./Pasta.css"></style>

	正如上面你看到的例子中，HTML, JavaScript和CSS都写在一个文件之中。你不再需要在 .vue 组件文件中引入 CSS，虽然这也是可以的。
- react 

	React也是非常相似的，JavaScript与JSX被写入同一个组件文件中。

		import React from "react";
		
		class PastaItem extends React.Component {
		
		    render() {
		        const { details, index } = this.props;
		
		        return (
		            <li className="pasta-dish list-unstyled">
		                <div className="row">
		                    <div className="col-md-3">
		                        <img src={details.image} alt={details.name} />
		                    </div>
		                    <div className="col-md-9 text-left">
		                        <h3>{details.name}</h3>
		                        <p>
		                            {details.desc}
		                        </p>
		                        <button onClick={() => this.props.addToOrder(index)} className="btn btn-primary">Add to order</button> <mark>{this.props.orders || 0}</mark>
		                    </div>
		                </div>
		            </li>
		        );
		    }
		}
		
		export default PastaItem;
- Props

	在上面两个例子中，我们可以看到 React 和 Vue 都有 'props' 的概念，这是 properties 的简写。props 在组件中是一个特殊的属性，允许父组件往子组件传送数据。

		Object.keys(this.state.pastadishes).map(key =>
		    <PastaItem index={key} key={key} details={this.state.pastadishes[key]} addToOrder={this.addToOrder} orders={this.state.orders[key]} />
		)
	上面的 JSX 库组中，`index,key,details,orders` 与 `addToOrder` 都是 props，数据会被下传到子组件 `PastaItem` 中去。
	
	- react 
	
		在React中，这是必须的，它依赖一个“单一数据源”作为它的“状态”（稍后有更多介绍）。
	- vue

		而在 Vue 中，props 略有不同。它们一样是在组件中被定义，但Vue依赖于模板语法，你可以通过模板的循环函数更高效地展示传入的数据。

			<pasta-item v-for="(item, key) in samplePasta" :item="item" :key="key" @order="handleOrder(key)"></pasta-item>
		这是模板的实现，但这代码完全能工作，然而在React中展现相同数据会更麻烦一点。

### 构建工具
React 和 Vue 都有自己的构建工具，你可以使用它快速搭建开发环境。

- React 可以使用 Create React App (CRA)
- 而 Vue 对应的则是 vue-cli。

两个工具都能让你得到一个根据最佳实践设置的项目模板。

由于 CRA 有很多选项，使用起来会稍微麻烦一点。这个工具会逼迫你使用 Webpack 和 Babel。而 vue-cli 则有模板列表可选，能按需创造不同模板，使用起来更灵活一点。

事实上说，两个工具都非常好用，都能为你建立一个好环境。而且如果可以不配置 Webpack 的话，我和Jeff认为这是天大的好事。

### Chrome 开发工具
React和Vue都有很好的 Chrome 扩展工具去帮助你找出 bug。它们会检查你的应用，让你看到 Vue 或者 React 中的变化。你也可以看到应用中的状态，并实时看到更新。

- [React的开发工具](https://link.segmentfault.com/?enc=9aKJjZSKEqt%2BVW%2Fn8diGdg%3D%3D.yWUZQZetyBVXfoNsCGanXlCMxGLcMZW%2FC111ZlUUBH3PCwrUMa8Wao%2BhFnZ%2BBuAWnceuBg%2FQe63rrSLPYqQDcxPmx3Mtx2mPOuxbHeNSX2w9c00bqzILRj7WDP5wig3h)
- [Vue的开发工具](https://link.segmentfault.com/?enc=qaZOq688ZjpIjNE8QNLl6Q%3D%3D.J%2FXWban7niM92LIKVPgCVmL70BT9stY6ryRxbS5RZ5MVCBKWaJCfO4MQ81nAp2dLgdqKGqKDD%2FEzWSyvUi%2B7KXOdJ1IeZkVtTUIFbSp0Q4vELXFmru1WESr9Q2WbuPv5)

### 配套框架
Vue与React最后一个相似但略有不同之处是它们配套框架的处理方法。相同之处在于，两个框架都专注于UI层，其他的功能如路由、状态管理等都交由同伴框架进行处理。

而不同之处是在于它们如何关联它们各自的配套框架。

- Vue的核心团队维护着 vue-router 和 vuex，它们都是作为官方推荐的存在。
- 而 React 的 react-router 和 react-redux 则是由社区成员维护，它们都不是官方维护的。

## 主要区别
Vue 与 react 有很多的相似之处，但他们也有完全不一致的地方。
### 模板 vs JSX
React与Vue最大的不同是模板的编写。

- Vue 鼓励你去写近似常规 HTML 的模板。写起来很接近标准HTML元素，只是多了一些属性。

		<ul>
		    <template v-for="item in items">
		        <li>{{ item.msg }}</li>
		        <li class="divider"></li>
		    </template>
		</ul>
	这些属性也可以被使用在单文件组件中，尽管它需要在在构建时将组件转换为合法的JavaScript和HTML。

		<ul>
		  <pasta-item v-for="(item, key) in samplePasta" :item="item" :key="key" @order="handleOrder(key)"></pasta-item>
		</ul>
	Vue鼓励你去使用HTML模板去进行渲染，使用相似于Angular风格的方法去输出动态的内容。因此，通过把原有的模板整合成新的Vue模板，Vue很容易提供旧的应用的升级。这也让新来者很容易适应它的语法。
- 另一方面，React推荐你所有的模板通用JavaScript的语法扩展——JSX书写。同样的代码，用JSX书写的例子如下：

		<ul className="pasta-list">
		    {
		        Object.keys(this.state.pastadishes).map(key =>
		            <PastaItem index={key} key={key} details={this.state.pastadishes[key]} addToOrder={this.addToOrder} orders={this.state.orders[key]} />
		        )
		    }
		</ul>

	React/JSX乍看之下，觉得非常啰嗦，但使用JavaScript而不是模板来开发，赋予了开发者许多编程能力。

能力越大，责任越大。 Ben Parker
J
SX只是JavaScript混合着XML语法，然而一旦你掌握了它，它使用起来会让你感到畅快。这可能只是我个人的意见，但我觉得这比Angular 1风格的属性好多了，Angular 1真的难以忍受。

而相反的观点是Vue的模板语法去除了往视图/组件中添加逻辑的诱惑，保持了关注点分离。

值得一提的是，与React一样，Vue在技术上也支持render函数和JSX，但只是不是默认的而已。

### 状态管理 vs 对象属性
- 如果你对React熟悉，你就会知道应用中的状态是（React）关键的概念。也有一些配套框架被设计为管理一个大的state对象，如Redux。此外，state对象在React应用中是不可变的，意味着它不能被直接改变（这也许不一定正确）。在React中你需要使用setState()方法去更新状态。

		 addToOrder(key) {
		        //Make a copy of this.state
		        const orders = { ...this.state.orders };
		
		        //update or add
		        orders[ key ] = orders[ key ] + 1 || 1;
		        this.setState( { orders } );
		 }
- 在Vue中，state对象并不是必须的，数据由data属性在Vue对象中进行管理。

		export default {
		  name: 'app',
		  data() {
		    return {
		      samplePasta: samplePasta,
		      orders: {}
		    }
		  },
		...
		  methods: {
		    handleOrder: function (key) {
		
		      if (!this.orders.hasOwnProperty(key)) {
		        this.$set(this.orders, key, { count: 0 });
		      }
		
		      this.orders[key].count += 1;
		    }
		  }
	}
	而在Vue中，则不需要使用如setState()之类的方法去改变它的状态，在Vue对象中，data参数就是应用中数据的保存者。

	对于管理大型应用中的状态这一话题而言，Vue.js的作者尤雨溪曾说过，（Vue的）解决方案适用于小型应用，但对于对于大型应用而言不太适合。

多数情况下，框架内置的状态管理是不足以支撑大型应用的，Redux或Vuex等状态管理方案是必须使用的。

## React Native vs. ?
React Native 能在手机上创建原生应用，React 在这方面处于领先位置。使用JavaScript, CSS和HTML创建原生移动应用，这是一个重要的革新。Vue社区与阿里合作开发Vue版的React Native——Weex也很不错，但仍处于开发状态且并没经过实际项目的验证。

## 参考
[Vue与React两个框架的区别和优势对比](https://segmentfault.com/a/1190000022816695)