# Vue

## 四个导出Vue的模块
* src/platforms/web/entry-runtime-with-compiler.js
  * web平台相关的入口
  * 重写了平台相关的`$mount()`方法：将template转换为render函数
  * 注册了`Vue.compile()`**`方法，传递一个HTML字符串返回render函数
* src/platforms/web/runtime/index.js
  * web平台相关
  * 增加和平台相关的config
  * 注册和平台相关的全局指令：`v-model，v-show`
  * 注册和平台相关的全局组件：`v-transform，v-transform-group`
  * 增加**实例成员**
    * `__patch__`: 把虚拟DOM转换成真实DOM
    * `$mount()`: 挂载到DOM
* src/core/index.js
  * 与平台无关
  * 设置了Vue的**静态成员**，initGlobalApi(Vue) in `src/core/global-api/index.js`
    * 注册函数：`component，delete, directive, extend, filter, mixin, nextTick, observable, options(components, directives, filters), set, use, util` 
    * 加载内置components：keep-alive  
* src/core/instance/index.js
  * 与平台无关
  * 定义了Vue构造函数，其中调用了this._init(options)
  * 给Vue中混入常用的**实例成员**
    * `initMixin`: _init
    * `stateMixin`：$data(返回_data), $props(返回_props), $set, $delete, $watch
    * `eventMixin`: $on, $once, $off, $emit   
    * `lifestyleMixin`: _update, $forceUpdate, $destroy   
    * `renderMixin`: $nextTick, _render, 和一系列_x函数


## 初始化实例
定义`Vue.prototype._init`
代码位置： `src/core/instance/init.js`
``` javascript
    // vm 生命周期相关变量初始化
		// $children/$parent/$root/$refs
		initLifecycle(vm)
		// vm 事件监听初始化，父组件绑定在当前组件上的事件
		initEvents(vm)
		// vm 的编译render初始化
		// $slot/$scopeSlots/_c/$createElement/$attrs/$listeners
    initRender(vm)
    // 钩子函数
		callHook(vm, 'beforeCreate')
    // 把 inject 成员注入vm
    initInjections(vm) // resolve injections before data/props
    // 初始化 vm 的 _props/methods/_data/computed/watch
    initState(vm)
    // 初始化provide
		initProvide(vm) // resolve provide after data/props
		callHook(vm, 'created')
```

## 首次渲染过程
主要是`_init()`方法的执行。   

