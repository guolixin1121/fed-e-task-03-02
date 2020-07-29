## 三种类型Watcher
vm.$watch() in src/core/instance/state.js
* 计算属性
* 用户Watcher
* 渲染Watcher

## 创建及执行顺序
Computed -> Watcher -> Render   
调用堆栈：   
initState -> initComputed   
initSTate -> initWatcher   
$mount -> mountComponent 

## 