+ [[1.响应式系统]]
+ [[编译器]]
+ [[渲染器]]
+ [[组件化]]


## 框架通用核心要素
+ 提升开发体验
	+ 直观的错误、日志输出信息
+ 控制代码体积
	+ 在不同环境下输出不同颗粒度的日志信息
+ 良好的tree-shaking
+ 构建产物输出形式
	+ esm
	+ commonjs
	+ umd
	+ iife
+ 特性开关
	+ 插件化的形式提供功能，按需将功能包插入或摘掉
+ 错误处理
+ TS支持



## Vue做了什么

1. Vue这个包的本质是什么？ 

创建了一个Vue对象，有很多属性、方法


```js
Vue对象

{
	// 创建应用
	createApp
	createSSRApp

	// 生命周期
	onMounted

	// 响应式
	ref
	reactive
	watch
	computed

	// 工具函数
	isProxy
}

// 创建应用实例
const app = createApp()
```


层级概念
+ renderer 渲染器 
+ 应用，需要挂载在一个DOM元素上
+ 组件 在应用之下的嵌套层级结构，每一个组件也是一个小的vue实例

### 创建renderer 渲染器

渲染器作用，把vdom渲染成特定平台上的真实元素

创建了一个通用renderer

baseCreateRenderer
	+ createRenderer 浏览器render
	+ createHydrationRenderer 支持服务端渲染的renderer

接收node、element操作作为参数，渲染器可以根据传递的特定平台的renderOptions参数，将vnode渲染成对应的真实元素

```js
renderOptions = {
	...patchProps, // props格式化
	...nodeOps // dom操作封装
}

const render = createRenderer()
```


renderer 渲染器主要干两件事情
1. dom diff 计算，根据新旧虚拟vnode，计算出最小的dom操作量
2. 根据第一部结果，调用平台api，操作dom更新

render创建完成之后，
dom diff 相关的方法、dom更新相关的方法，都已经在初始化完成，在内存中，等待调用

### 创建APP
针对应用，创建一个应用实例

**应用**也是一个对象

```js
// app 应用实例

const context = {
	app,
	config: {
		isNativeLog,
		performance,
		globalProperties,
		optionMergeStrategies,
		errorHandler,
		warnHanlder,
		compilerOptions
	},
	mixins: [],
	components: [],
	directives: [],
	provides: {},
	optionsCache: new WeakMap(),
	propsCache: new Weakmap(),
	emitsCache: new WeakMap()
}
const app = {
	// 应用基础信息
	_uid: uid++,
	_component: rootComponent,
	_props: rootProps,
	_container: null,
	_context: context,
	_instance: null
	
	version,
	
	get config() {
		return context.config
	},
	set config() {
		// 不可被修改
	},
	
	use() {
		// 插件注册
	},
	
	mixin(mixin) {
		context.mixins.push(mixin)
	},
	component(name, component) {
		context.components[name] = component
	},
	directive(name, directive) {
		context.directives[name] = directive
	},
	

	// 挂载
	mount(rootContainer) {
		
	},
	unmount() {
	
	},
	
	provide(key, value) {
		context.provides[key] = value
	}
}

// 创建app实例
const app = renderer.createApp(appConf)
```



createApp
+ createRenderer 创建 渲染器
	+ 初始化渲染
	+ DOM diff -- patch 计算出最小DOM变更
	+ DOM update 更新DOM
+ createApp 创建app实例
	+ createAppAPI  最终得到一个应用app实例
+ app.mount 
	+ 根据参数将根组件挂载到指定的DOM上
+ render

### app实例mount

```js
app.mount(container)
```

mount操作就是把vnode挂载到真实dom上的过程
在此之前，都是纯js的操作

mount需要两个关键点
1. vnode 要挂载的虚拟dom信息
2. container 挂载的目标点 


```js
/**
	mount 操作步骤:
	1. 格式化container 
	2. 创建根组件vnode
	3. 渲染
*/ 

const container = normalizeContainer()
const vnode = createVnode()

render(vnode, container)
```


#### createVnode vDom创建

createBaseVNode
```js
const vnode = {
	__v_isVNode: true,
	__v_skip: true,
	type,
	props,
	key,
	ref,
	scopedId,
	slotScopeIds,
	children,
	component,
	suspense,
	ssContent,
	ssFallback,
	dirs,
	transition,
	el,
	anchor,
	target,
	targetAnchor,
	staticCount,
	shapeFlag,
	patchFlag,
	dynamicProps,
	dynamicChildren,
	appContext,
	ctx
}
```

#### render 渲染
```js
render(vnode， container)
patch(container._vnode, vnode)

// 根据vnode类型 处理不同逻辑分支
// processElement()
processComponent(n1, n2, container)


// 根据组件不同状态
// activate() keep-alive 组件重新激活
mountComponent() // 初始化组件
// updateCOmponent() // 更新组件

createComponentInstance()
创建一个组件instance， 后续组件处理都是围绕这个对象来的

// 初始化组件 处理props、slots、setup函数
setupComponent()
	initProps()
	initSlots()
	setupStatefulComponent() // setup 执行
		setupResult
			isPromise // 客户端渲染不支持返回promise
		handleSetupResult()
			isFunction   赋值给组件的render
			isObject 赋值给组件的setupState
		finishComponentSetup
			applyOptions 兼容2.x版本选项数据处理
	
setupRenderEffect
	ReactiveEffect

```


setupRenderEffect 流程梳理


```js
processElement()
	mountElemnt()
		hostCreateElement() 创建本级dom
		hostSetElemntText() 
		mountChildren() 创建cildren
		hostInsert()
		
	patchElemnt()
		patchChildren()
			fast path
				patchKeyedChildren
				patchUnkeyedChildren
			full diff
					
```


```js
/**
	n1 n2 分别代表新旧children的vnode
	vnode可能有三种情况：text、array、null
	所以正常有 3*3 = 9 种情况
	但实际上只需要处理3种
	1. 新vnode是text
		1. 旧vnode是array 卸载即可
		2. 旧vnode是text 更新innerText即可
	2. 新vnode非text 并且 旧vnode是array
		1. 新vnode是array patchChildren 递归diff
		2. 新vnode是null 卸载旧vnode
	3. 旧node是text或者null 新node是array或null
		1. 卸载旧vnode，挂载新vnode 不需要复用
*/
patchChildren(n1, n2)
	if(shapeFlag === TEXT_CHILDREN) {
		if(prevShapeFlag === ARRAY_CHILDREN) {
			unmountChildren(n1)
		}
		if (n1 !== n2) {
			setElementText(n2)
		}
	} else {
		if(prevShapeFlag === ARRAY_CHILDREN) {
			if(shapeFlag === ARRAY_CHILDREN) {
				patchKeyedChildren()
			} else {
				unmountChildren()
			}
		}else {
			if(prevShapeFlag === TEXT_CHILDREN) {
				setElementText('')
			}
			if(shapeFlag === ARRAY_CHILDREN) {
				mountChildren()
			}
		}
	}


```

* [ ] app 实例是什么作用
* [ ] component/template 将不同格式的内容 统一成 component
	* [ ] setup render函数
	* [ ] template 
* [x] 在哪个阶段 响应式系统生效？ 将render函数与ref值依赖关系建立？ ✅ 2024-03-04
* [ ] 组件/element 逻辑分支 判断 处理
* [ ] 组件嵌套 完整流程
* [ ] 


### 关键概念
1. renderer 渲染器 把vdom渲染成特定平台上的真实元素
	1. createApp
	2. render
	3. hydrate 服务端渲染
	4. 自定义渲染器
		1. 对平台API抽象，作为参数传递
		2. 传递不同参数，实现不通平台的渲染
		3. 通用渲染器
2. render 渲染动作
3. mount 挂载 渲染器将vdom渲染成真实DOM节点的过程
	1. 需要指定vnode和挂载点container
4. patch 打补丁 更新
	1. 新老vnode，以及挂载点
	2. mount是一种特殊的patch，只不过oldVnode是null而已