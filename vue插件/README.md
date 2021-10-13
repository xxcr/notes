

## 插件



### 定义

> 原生插件 `vue-router`、`vuex`

插件通常用来为 `Vue` 添加全局功能。—— `Vue.js` 官网



那他和组件的区别：

组件是可复用的 `Vue` 实例，且带有一个名字。—— `Vue.js` 官网



其实， `Vue 插件` 和 `Vue组件` 只是在 `Vue.js` 中包装的两个概念而已，不管是插件还是组件，最终目的都是为了实现逻辑复用。它们的本质都是对代码逻辑的封装，只是封装方式不同而已。在必要时，组件也可以封装成插件，插件也可以改写成组件，就看实际哪种封装更方便使用了。



插件一般有下面几种：

- 添加全局方法或者属性。如: `vue-custom-element`
- 添加全局资源：指令/过滤器/过渡等。如 `vue-touch`
- 通过全局混入来添加一些组件选项。如 `vue-router`
- 添加 `Vue` 实例方法，通过把它们添加到 `Vue.prototype` 上实现。
- 一个库，提供自己的 `API`，同时提供上面提到的一个或多个功能。如 `vue-router`

—— `Vue.js` 官网

```js
export default {
    install(Vue, options) {
        Vue.myGlobalMethod = function () {  // 1. 添加全局方法或属性，如:  vue-custom-element
            // 逻辑...
        }

        Vue.directive('my-directive', {  // 2. 添加全局资源：指令/过滤器/过渡等，如 vue-touch
            bind (el, binding, vnode, oldVnode) {
                // 逻辑...
            }
            ...
        })

        Vue.mixin({
            created: function () {  // 3. 通过全局 mixin方法添加一些组件选项，如: vuex
                // 逻辑...
            }
            ...
        })    

        Vue.prototype.$myMethod = function (options) {  // 4. 添加实例方法，通过把它们添加到 Vue.prototype 上实现
            // 逻辑...
        }
    }
}
```

`install`是注册插件主要调用的方法，包含了两个参数（`Vue`实例和自定义配置属性`options`）。



### 使用

插件需要通过 `Vue.use()` 方法注册到全局，并且需要在调用 `new Vue()` 启动应用之前完成。之后在其他 `Vue` 实例里面就可以通过 `this.$xxx` 来调用插件中提供的 `API` 了。





## 参考文献

1. [开发一个Vue插件](https://segmentfault.com/a/1190000021959058)
2. [Vue.use(plugin)详解](https://juejin.cn/post/6844903946343940104)
3. [Vue.extend 编程式插入组件](https://juejin.cn/post/6844903998672076813)

