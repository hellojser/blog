---
title: Vue 2.5.9 源码分析（三） ~ Vue初始化之选项的合并策略
date: 2018-08-10 17:00:22
tags: 
  - 源码分析
  - Vue源码分析
  - 技能知识图谱
---
上一篇，在Vue的合并选项过程中，对`props`,`inject`,`directives`进行了统一的规范化处理，不管框架的使用者在传入初始化选项的时候传入了何种格式，在框架的内部都会转换成统一的数据格式，在提升了框架使用上的灵活性的同时，又保证了框架在内部对数据更加严谨的处理。在之前的分析过程中，我们也遇到了些许疑问，并加上了相应的问题备注，不过，这些暂时不会影响我们继续阅读源码，随着对整体框架的深入的了解，再回看这些问题，这些问题也就不再是问题。

实际上，本文和上一篇算是一部分，选项合并主要包含了两部分：一部分是对数据的规范化，另一部分就是对不同选项的策略的挂载。这篇，我们紧接着上一篇，对选项的策略挂载进行分析。
<!-- more -->
定义options，这也是最终选项统一化处理和合并策略阶段处理完毕之后，最终的返回值。
```
const options = {}
```
其次对parent进行遍历，还记得parent是什么么？在我们之前的示例中代表的是Vue构造函数的静态属性'options'.
```
  for (key in parent) {
    mergeField(key)
  }
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
  function mergeField (key) {
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
```
在示例中，首先对`components`,`directives`,`filters`,`_base`进行遍历，首先从`strats`中获取到对应的策略方法，找不到的话将默认策略赋给它。我们先看下strats是从哪里来的？
```
import config from '../config'
const strats = config.optionMergeStrategies
```
在最开始，定义了strats，其值是从config中导入的optionMergeStrategies，还记得config在哪里用过么？那就是我们在第一篇文章挂载静态方法的阶段，如下：
```
Vue.config = {
  // user
  optionMergeStrategies: { [key: string]: Function };
  ...
}
```
同时，也可以在官方文档中找到这个配置的相关说明（[戳这里](https://cn.vuejs.org/v2/api/#optionMergeStrategies)），先从config.optionMergeStrategies拿到用户自定义的合并策略方法。在这基础之上，开始定义对各个初始化选项的策略方法。

#### defaultStrat

```
/**
 * Default strategy.
 */
const defaultStrat = function (parentVal: any, childVal: any): any {
  return childVal === undefined
    ? parentVal
    : childVal
}
```
默认策略，接受两个参数父实例和子实例，内部先判断childVal是否被传入，传入的话返回childVal，不存在的话，返回父实例。

#### el & propsData
```
if (process.env.NODE_ENV !== 'production') {
  strats.el = strats.propsData = function (parent, child, vm, key) {
    if (!vm) {
      warn(
        `option "${key}" can only be used during instance ` +
        'creation with the `new` keyword.'
      )
    }
    return defaultStrat(parent, child)
  }
}
```
在非生产环境下，对options中的el和propsData进行处理的时候，会校验当前的实例是否存在，不存在会有相应的提示。非生产环境下会直接返回默认策略`defaultStrat`，生产环境下也会因为`const strat = strats[key] || defaultStrat`，先是在strats下找不到el和propsData,之后直接执行默认策略。

#### data
对data的策略方法的一开始，便首先判断了当前是否存在实例：
```
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      )

      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }

  return mergeDataOrFn(parentVal, childVal, vm)
}
```
- 不存在实例vm：
	- 判断当前传入的初始化data选项是否是一个函数*（之前的示例，data一直都是一个对象，官方文档中有提到data应该是一个返回对象的函数，后面我们在分析其到底是为什么）*,不是函数的话直接返回父实例的data项
	- 是一个函数的话，会调用mergeDataOrFn,并将父实例和子实例作为参数传进去
- 存在实例vm,会调用mergeDataOrFn,并将构造函数和传入的当前配置项、vm实例、作为参数传进去

在我们这里示例中，会直接执行`mergeDataOrFn(parentVal, childVal, vm)`,来看`mergeDataOrFn`:
```
  if (!vm) {
  	...
  } else {
    return function mergedInstanceDataFn () {
      // instance merge
      const instanceData = typeof childVal === 'function'
        ? childVal.call(vm)
        : childVal
      const defaultData = typeof parentVal === 'function'
        ? parentVal.call(vm)
        : parentVal
      if (instanceData) {
        return mergeData(instanceData, defaultData)
      } else {
        return defaultData
      }
    }
  }
```
因为在Vue的初始化过程中，第一次初始化data时执行到这里，是可以获取到vm的,从代码原作者的注释中*(// in a Vue.extend merge, both should be functions)*也可以看出，上面省略的部分跟Vue.extend有关，我们先按照示例走，在这里会返回一个`mergedInstanceDataFn`函数，所以最终`options.data = mergedInstanceDataFn(){}`。

#### 生命周期函数策略
首先从`shared/onstant.js`中，引入了所有的生命周期方法：
```
export const LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated',
  'errorCaptured'
]
```
之后为所有的生命周期函数挂载了统一的策略函数`mergeHook`，我们来看下`mergeHook`:
```
function mergeHook (
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>
): ?Array<Function> {
  return childVal
    ? parentVal
      ? parentVal.concat(childVal)
      : Array.isArray(childVal)
        ? childVal
        : [childVal]
    : parentVal
}
```
- 在子实例中是否有该项生命周期函数：
	- 在父实例中是否有该项生命周期函数：
		- 是，将父实例和子实例进行合并
		- 否，判断当前子实例中该项是否为一个数组，不为数组的转换成一个数组
- 没有相关项，直接返回父实例的该项

所以，最终options[生命周期名称] = [数组项]

#### component & directive & filter
首先从`shared/onstant.js`中，引入了静态类型的选项名称：
```
export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
```
统一的策略函数`mergeAssets`，我们来看下`mergeAssets`:
```
function mergeAssets (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): Object {
  const res = Object.create(parentVal || null)
  if (childVal) {
    process.env.NODE_ENV !== 'production' && assertObjectType(key, childVal, vm)
    return extend(res, childVal)
  } else {
    return res
  }
}
```
首先从父实例上缓存相关项的值，没有的话通过Object.create(null)创建一个空对象。之后在判断子实例中是否存在该项，在非生产环境下还需要通过`assertObjectType`校验格式并有相应的提示：
- 如果含有该项，需要将子实例的该项复制合并到父实例的该项
- 无该项，直接返回父实例的该项或空对象

#### watch
```
  // work around Firefox's Object.prototype.watch...
  if (parentVal === nativeWatch) parentVal = undefined
  if (childVal === nativeWatch) childVal = undefined
```
判断parentVal或者childVal是否是Object.prototype.watch，在Gecko下有watch的API的实现，详细信息戳这里([MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/watch)),避免Vue的watch方法和原型上的watch方法冲突，如果是FireFox下的，先将parentVal或者childVal重置为undefined。

```
if (!childVal) return Object.create(parentVal || null)
```
如果子实例不存在该项，直接返回父实例的该项或者一个空对象。
```
  if (process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
```
非生产环境下还需要校验当前的类型，给给出相应的提示信息。
```
if (!parentVal) return childVal
```
如果父实例不存在该项，直接返回子实例的该项，在我们的例子中便直接返回传入的watch项。

```
  const ret = {}
  extend(ret, parentVal)
  for (const key in childVal) {
    let parent = ret[key]
    const child = childVal[key]
    if (parent && !Array.isArray(parent)) {
      parent = [parent]
    }
    ret[key] = parent
      ? parent.concat(child)
      : Array.isArray(child) ? child : [child]
  }
```
最后是父实例和子实例中都含有watch项，那么要进行合并处理。合并的主要过程就是：
- 首先定义一个常量ret，将父实例的watch项合并到ret.
- 遍历子实例的watch
- 在遍历的过程中，判断父实例是否拥有很子实例同样名称的watch：
	- 有，则将父实例和子实例的watch合并成一个数组。
	- 没有，直接将子实例返回就行

其中，返回的都是一个数组，如果在处理的过程中，父实例和子实例的watch不是Array的时候，会通过[]进行包起来转换成数组。

#### props & methods & inject & computed
接下来会对这四个选项进行处理，先判断其类型，并在生产环境下给出提示：
```
  if (childVal && process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
```
父实例不包含该项，直接返回子实例即可：
```
if (!parentVal) return childVal
```
父子实例都存在该项的话，将两个进行合并返回：
```
  const ret = Object.create(null)
  extend(ret, parentVal)
  if (childVal) extend(ret, childVal)
  return ret
```

#### provide
最后是provide的合并策略，其策略函数是`mergeDataOrFn`，这个函数在data的合并策略介绍过。

## 完整示例
最后，我们用之前的例子，将所有选项填满，通过运行Vue在Debug中整体看一下options的返回值：
```
var demo = new Vue({
  data(){
  	return {
	    a:1
	  }
  },
  el:'#demo',
  propsData: {
    msg: 'hello'
  },
  mounted(){
  	console.log("创建喽！");
  },
  directives: {
	focus: {
	// 指令的定义
	inserted: function (el) {
	  el.focus()
	}
	}
  },
  watch:{
  	watchA(){
  		console.log('watchA');
  	},
  	watchB(){
  		console.log('watchB');
  	}
  },
  methods:{
  	sayHello(){
  		console.log('hello MIFE');
  	}
  },
  provide: {
    foo: 'bar'
  },
  props:['js-bridge','vueprops'],
  inject:['inject-test1','inject-test2']
})
```

![debugger_options](/images/2018/8/vue-3/debugger_options.png)

(完)









