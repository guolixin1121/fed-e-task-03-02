
## 一、简答题

### 1、请简述 Vue 首次渲染的过程。
1. Vue初始化，定义实例成员和静态成员
   通过四个主要入口函数，为Vue注入静态成员和实例成员
   * core/instance/index.js: 定义Vue，并注册跟init/state/event/lifestyle/render相关的实例成员
   * core/index.js: 通过gloabl-api为Vue注册静态成员
   * plateform/web/runtime/index.js: 注册与平台相关的config/components/directives/__patch__/$mount
   * platform/web/entry-rumtime-with-compiler.js: 重新改写$mount, 对template进行编译
2. new Vue()实例化对象
   主要是在`this._init`里通过xxInit()实例化vue对象的实例成员，最后调用`$mount()`函数渲染页面
3. 渲染页面
   **第一步：判断是否需要将tempalte编译为render函数**
   代码位置：`src/platform/web/entry-runtime-with-compiler.js`
   * 判断options是否有render函数
   * 没有render，判断是否有template选项
     * 有template，判断是否是带#号的string，是则把该id对应的内部html文本取出作为template
     * 有template，判断是有有nodeType，是则表明是DOM节点，取出内部html文件作为template   
     * 没有template，判断是否有el，取出el对应的DOM的html作为template
   * 拿到template后，执行compileToFunctions将template编译为render函数并存入options
  
   **第二步：用render函数渲染组件**
   代码位置：`src/platform/web/runtime/index.js`调用`src/core/instance/lifestyle.js`的mountComponent函数挂载组件
   * 是否有render，如果没有且传入了模板并在开发环境则发送警告
   * 触发`beforeMount`
   * 定义updateComponent渲染页面的函数
     * 用户或compiler生成的render()生成VDOM
     * _update通过调用__patch()__将VDOM变成DOM并渲染到this.$el中
   * 定义Watcher，并传递updateComponent。真正的渲染会在watcher构造函数中调用其getter方法触发
   * 触发`mounted`

### 2、请简述 Vue 响应式原理。
第一阶段：初始化vm实例时将数据变为响应式
1. 在`vm._init中`调用`initData()`将data注入到vm实例，并通过`observe()`将其变为响应式
2. `observe()`为data创建Observer实例使其变为可响应对象
3. `Observer class`遍历对象每个属性，调用`defineReactive()`将每个属性变为响应式
4. `defineReactive()`
   * 为每个属性创建Dep对象用于收集依赖，依赖的收集需要通过`Watcher`触发
   * 为属性定义getter方法，收集依赖并返回值
   * 为属性滴定仪setter方法，保存性质，并调用dep.notify发送更新

第二阶段：使用data时收集依赖和触发更新
5. `Watcher`的`get()`触发收集依赖
   * `$mount->$mountComponet->new Watcher()->watcher.get()->updateComponet()->render()`首次渲染时
   * `dep.notify()->watcher.update()->watcher.run()->watcher.get()->updateComponent()->render()`更新组件时
   * 将Dep.target设置为自身
   * 渲染页面时会调用属性的getter从而触发依赖的收集
   * 属性的getter将当前`Watcher`收集到到dep的subs数组中
   * 如果属性值是对象，则为对象每个属性创建observer
6. 触发更新通知
   * 为属性赋值时，触发setter的dep.notify方法通知每个依赖Watcher的`update()`
   * `queueWatcher()`判断watcher是否被处理，没有则添加到queue队列，并调用`flushSchedulerQueue()`
   * `flushSchedularQueue()` 触发`beforeUpdate`钩子，并调用`watcher.run()`更新视图，触发`actived, updated`钩子函数
   * `watcher.run()`会依次触发`get()-->getter()->updateComponent()`更新视图 

### 3、请简述虚拟 DOM 中 Key 的作用和好处。
对于列表元素，循环依次对比每对新旧节点时，认为都是相同节点，如果内容不同则替换。这会导致不断的进行不必要的更新dom操作。 而设置为key后，每个元素被设置独立标识，仅会对新增修改删除的节点进行DOM操作，这样能避免不必要的节点操作。   
比如在列表头部插入新节点，不设置key会导致整个列表全部进行DOM更新操作，设置key只会进行一次DOM插入操作。

### 4、请简述 Vue 中模板编译的过程。
模板的编译工作是将HTML模板转换成render函数的过程
1. compileToFunctions()从缓存中加载编译好的render函数。如果没有缓存则调用compile()编译
2. compile()合并好选项后，调用baseCompile()进行实际的编译
3. baseCompile()编译template模板为js代码，并返回compileToFunctions
   * parse：把template转换成AST
   * optimize：标记静态树，patch时可直接跳过静态树从而提高性能
   * generate：将AST生成js的创建代码
4. compileToFunctions()将generate生成的字符串js代码转换为render函数