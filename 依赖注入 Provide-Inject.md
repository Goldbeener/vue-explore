
## 解决的问题
解决**Prop逐级透传问题**


+ 应用层Provide 所有组件可用，编写插件有用
+ 组件层Provide 子组件可用


## 使用方式
```js
const app = createApp()
app.provide(key, value)


// .vue
import { provide inject} from vue

setup() {
	provide(key, value)
	inject(key)
	inject(key, defaultValue)
	inject(key, factoryFn, true)
}
```

### 限制
正常**必须在setup内使用**provide/inject

>可以借助runWithContext在非setup期间使用inject

因为在inject获取前提条件就是 有**活动的组件实例**或者是**活动的app实例**
runWithContext可以临时激活app实例

```js
app.runWithContext = function(fn){
	const lastApp = currentApp
	currentApp = app // 这一步激活app实例 后面fn函数调用时，可以使用inject
	try {
		return fn()
	} finally {
		currentApp = lastApp
	}
	
}
```

# 原理

```js
// 应用全局
context = { provides: {} } // 全局应用context
// 根组件
rootVnode.appContext = context

app.provide(key, value)
// app层面调用provide 存储在全局context里面


// 组件 createComponentInstance
function createComponentInstance() {
	appContext = parent ? parent.appContext : vnode.appContext
	// 根组件无parent 所以根组件会获取vnode.appCOntext 
	// 因为上一步已经在根组件上将全局context赋值给了根组件的appContext属性， 所以这时候有值

	instance = { /**组件实例*/}

	instance.provides = parent ? parent.provides : Object.create(appContext.provides)
	// 每个组件实例在创建的时候，会有一个provides属性
	// 根组件的这个属性值是以全局appContext.provides为原型的新对象
	// 非根组件，这个属性值直接指向父组件的provides属性
	// 子指父 父指根 根指app

	// 这样其实所有组件实例的provides都指向一个对象 达到共享数据的目的
}


// provide api 实现
function provide(key, value) {
	let provides = currentInstance.provides
	const parentProvides = currentInstance.parent.provides

	// 第一次设置组件的provide时候，组件的provides默认值等于父组件的provides，因为上一步初始化的时候是直接引用赋值
	// 这时候 需要使用create继承，基于父组件provides为原型创建一个新对象
	// 这样既能在不污染上层provides的前提下扩展当前组件provides
	// 也能利用原型链向上寻址 复用上层的provides
	if (provides === parentProvides) {
		provides = Object.create(parentProvides)
	}

	provides[key] = value
}

// inject api 实现
function inject(key) {
	const instance = currentInstance
	if (instance || currentApp) {
		// 有instance 和 currentApp 前提下 正常使用app
		const provides = instance
		 ? instance.parent == null
			 ? instance.vnode.appContext?.provides
			 : instance.parent.provides
		 : currentApp._context.provides // 没有活动组件有活动app实例时，inject会去全局provides获取 这时候在非setup，借助runWithContext
		return provides[key]
	}
}

```


### 关键技术点

**Object.create**(targetObj) 
创建一个以指定对象为原型的新对象
可以方便利用原型链 向上寻值

**currentInstance** 
这块用到了import/export对外暴露的是一个对象引用
模块内部动态更新，外部可以应用到最新值

currentInstance 始终指向当前正在创建的组件对象
因此只在组件创建的生命周期内可用，正常必须在setup内使用

在组件setup执行之前，会对currentInstance赋值
所以这个对象在setup内使用是比较安全的
可以使用**getCurrentInstance**方法获取
