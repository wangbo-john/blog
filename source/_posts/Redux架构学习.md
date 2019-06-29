title: Redux架构学习
entitle: 'redux-study'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528023659
date: 2018-06-03 19:00:59
tags:
    - 前端
    - javascript
keywords: JavaScript, Redux
description: 来自2018.1.22的笔记：Redux架构学习笔记
photos:
    - /img/2018/15280237110062.jpg
---

## 整理自：

资料：https://segmentfault.com/a/1190000006742449

https://www.zhihu.com/question/41312576/answer/90782136


### 一、定义

Redux is a predictable state container for JavaScript apps.，其中predictable和state container体现了它的作用。那么如何来理解可预测化的呢？这里会有一些函数式编程方面的思想，在Redux中reducer函数是一个纯函数，相同输入一定会是一致的输出，所以确定输入的state那么reducer函数输出的state一定是可以被预测的，因为它只会进行单纯的计算，保证正确的输出。状态容器又是什么？说明Redux有一个专门管理state的地方，就是Store，并且一般情况下是唯一的，应用中所有state形成的一颗状态树就是Store。Redux由Flux演变而来，但受 Elm 的启发，避开了 Flux 的复杂性，我们看看其数据流向：

![](/img/2018/15280237110062.jpg)

不同于Flux架构，Redux中没有dispatcher这个概念，并且Redux设想你永远不会变动你的数据，你应该在reducer中返回新的对象来作为应用的新状态。但是它们都可以用(state, action) => newState来表述其核心思想，所以Redux可以被看成是Flux思想的一种实现，但是在细节上会有一些差异。

### 二、原则
1. 应用中所有的state都以一个object tree的形式存储在一个单一的store中；
2. 唯一能改变store的方法是触发Action，Action是动作行为的抽象；
3. 为了描述Action如何改变State树，需要编写reducer函数；

```javascript
function testReducer(state, action) {
  switch (action.type) {
    case ACTION_TYPE:
      // calc...
      return newState;
    default: return state;
  }
  return newState;
}
```

state是不可修改的，所以返回的新state应该是基于输入state副本的修改，而不是直接修改state后的返回。
可见
1、单一数据源，store
整个应用的state被存放在一棵Object tree树，并且整个object tree只存在唯一的一个store中。
2、State是只读的
唯一能改变State的方法是触发Action
3、使用纯函数来实现State归并操作，reducer
传入待修改的state和一个告知reducer如何修改state的action，reducer将返回action规则对应下操作后的新的state。

> reducer(state, action) => new state


### 三、数据流

严格的单向数据流是Redux设计的核心
Redux应用数据的生命周期遵循下面4个步骤：

调用store.dispatch(action), 可以在任何地方进行;
Redux store调用传入的reducer函数，并且将当前的state树与action传入。reducer是纯函数，只用于计算下一个state，它应该是完全可被预测的，相同的输入必定会有相同的输出，不能有副作用的操作，如API的调用或者路由跳转，这些应该都是在dispatch前产生；
根reducer将多个子reducer输出合并成一个单一的state树；
Redux store保存了根reducer返回的完整的state树。
新的state树就是应用的下一个状态，现在就可以根据新的state tree来渲染UI

1. React有props和state: props意味着父级分发下来的属性，state意味着组件内部可以自行管理的状态，并且整个React没有数据向上回溯的能力，也就是说数据只能单向向下分发，或者自行内部消化。
理解这个是理解React和Redux的前提。
2. 一般构建的React组件内部可能是一个完整的应用，它自己工作良好，你可以通过属性作为API控制它。但是更多的时候发现React根本无法让两个组件互相交流，使用对方的数据。
然后这时候不通过DOM沟通（也就是React体制内）解决的唯一办法就是提升state，将state放到共有的父组件中来管理，再作为props分发回子组件。
3. 子组件改变父组件state的办法只能是通过onClick触发父组件声明好的回调，也就是父组件提前声明好函数或方法作为契约描述自己的state将如何变化，再将它同样作为属性交给子组件使用。
这样就出现了一个模式：数据总是单向从顶层向下分发的，但是只有子组件回调在概念上可以回到state顶层影响数据。这样state一定程度上是响应式的。
4. 为了面临所有可能的扩展问题，最容易想到的办法就是把所有state集中放到所有组件顶层，然后分发给所有组件。
5. 为了有更好的state管理，就需要一个库来作为更专业的顶层state分发给所有React应用，这就是Redux。让我们回来看看重现上面结构的需求：
a. 需要回调通知state (等同于回调参数) -> action
b. 需要根据回调处理 (等同于父级方法) -> reducer
c. 需要state (等同于总状态) -> store
对Redux来说只有这三个要素：
a. action是纯声明式的数据结构，只提供事件的所有要素，不提供逻辑。
b. reducer是一个匹配函数，action的发送是全局的：所有的reducer都可以捕捉到并匹配与自己相关与否，相关就拿走action中的要素进行逻辑处理，修改store中的状态，不相关就不对state做处理原样返回。
c. store负责存储状态并可以被react api回调，发布action.
当然一般不会直接把两个库拿来用，还有一个binding叫react-redux, 提供一个Provider和connect。很多人其实看懂了redux卡在这里。
a. Provider是一个普通组件，可以作为顶层app的分发点，它只需要store属性就可以了。它会将state分发给所有被connect的组件，不管它在哪里，被嵌套多少层。
b. connect是真正的重点，它是一个科里化函数，意思是先接受两个参数（数据绑定mapStateToProps和事件绑定mapDispatchToProps），再接受一个参数（将要绑定的组件本身）：
mapStateToProps：构建好Redux系统的时候，它会被自动初始化，但是你的React组件并不知道它的存在，因此你需要分拣出你需要的Redux状态，所以你需要绑定一个函数，它的参数是state，简单返回你关心的几个值。
mapDispatchToProps：声明好的action作为回调，也可以被注入到组件里，就是通过这个函数，它的参数是dispatch，通过redux的辅助方法bindActionCreator绑定所有action以及参数的dispatch，就可以作为属性在组件里面作为函数简单使用了，不需要手动dispatch。这个mapDispatchToProps是可选的，如果不传这个参数redux会简单把dispatch作为属性注入给组件，可以手动当做store.dispatch使用。这也是为什么要科里化的原因。
做好以上流程Redux和React就可以工作了。

简单地说就是：
1.顶层分发状态，让React组件被动地渲染。
2.监听事件，事件有权利回到所有状态顶层影响状态。


