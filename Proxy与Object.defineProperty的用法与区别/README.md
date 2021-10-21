> 本文基本是复制该文章[ES6 系列之 defineProperty 与 proxy](https://juejin.cn/post/6844903710410162183)
> 作者：冴羽
> 链接：https://juejin.cn/post/6844903710410162183
> 来源：掘金

> 本文基本是复制该文章[重学JS | Proxy与Object.defineProperty的用法与区别](https://juejin.cn/post/6973636618515120165)
> 作者：梁龙先森
> 链接：https://juejin.cn/post/6973636618515120165
> 来源：掘金

# Proxy与Object.defineProperty的用法与区别

## 前言

我们或多或少都听过“数据绑定”这个词，“数据绑定”的关键在于监听数据的变化，可是对于这样一个对象：`var obj = {value: 1}`，我们该怎么知道 obj 发生了改变呢？

## definePropety

ES5 提供了 Object.defineProperty 方法，该方法可以在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回这个对象。

### **语法**

> Object.defineProperty(obj, prop, descriptor)

### **参数**

```
obj: 要在其上定义属性的对象。

prop:  要定义或修改的属性的名称。

descriptor: 将被定义或修改的属性的描述符。
```


函数的第三个参数 descriptor 所表示的属性描述符有两种形式：**数据描述符和存取描述符**。

**一个描述符只能是这两者中的一个，不能同时是两者，且两种描述符都是对象。**

描述符共享以下属性：

1. configurable（默认false）

   是否可配置，为`true`时，属性描述符才能够被改变，同时属性可以从对应对象上被删除。

2. enumerable（默认false）

   是否可枚举，为`true`时，属性才会出现在对象的枚举属性中

### **数据描述符**

数据描述符可选键值：

1. value（默认undefined）

   属性对应的值，可以是任何有效的`JavaScript`值。

2. writable（默认false） 键值为`true`时，`value`才能被赋值运算符改变。

### 存取描述符可选键值

1. get（默认undefined）

   属性的`getter()`函数，当访问该属性时，会调用此函数。返回的返回值会被用作属性的值。

2. set（默认undefined）

   属性的`setter()`函数，当属性被修改时，会调用此函数。

`Object.defineProperty`只能代理对象上的某个属性，因此存在对内部属性进行代理的时候，只能一次性递归完成对所有属性的代理。

此外，所有的属性描述符都是非必须的，但是 descriptor 这个字段是必须的，如果不进行任何配置，你可以这样：

```js
var obj = Object.defineProperty({}, "num", {});
console.log(obj.num); // undefined
```

### Setters 和 Getters

之所以讲到 defineProperty，是因为我们要使用存取描述符中的 get 和 set，这两个方法又被称为 getter 和 setter。由 getter 和 setter 定义的属性称做”存取器属性“。

当程序查询存取器属性的值时，JavaScript 调用 getter方法。这个方法的返回值就是属性存取表达式的值。当程序设置一个存取器属性的值时，JavaScript 调用 setter 方法，将赋值表达式右侧的值当做参数传入 setter。从某种意义上讲，这个方法负责“设置”属性值。可以忽略 setter 方法的返回值。

举个例子：

```js
var obj = {}, value = null;
Object.defineProperty(obj, "num", {
    get: function(){
        console.log('执行了 get 操作')
        return value;
    },
    set: function(newValue) {
        console.log('执行了 set 操作')
        value = newValue;
    }
})

obj.num = 1 // 执行了 set 操作

console.log(obj.num); // 执行了 get 操作 // 1
```

这不就是我们要的监控数据改变的方法吗？我们再来封装一下：

```js
function Archiver() {
    var value = null;
    // archive n. 档案
    var archive = [];

    Object.defineProperty(this, 'num', {
        get: function() {
            console.log('执行了 get 操作')
            return value;
        },
        set: function(value) {
            console.log('执行了 set 操作')
            value = value;
            archive.push({ val: value });
        }
    });

    this.getArchive = function() { return archive; };
}

var arc = new Archiver();
arc.num; // 执行了 get 操作
arc.num = 11; // 执行了 set 操作
arc.num = 13; // 执行了 set 操作
console.log(arc.getArchive()); // [{ val: 11 }, { val: 13 }]
```

### watch API

既然可以监控数据的改变，那我可以这样设想，即当数据改变的时候，自动进行渲染工作。举个例子：

HTML 中有个 span 标签和 button 标签

```js
<span id="container">1</span>
<button id="button">点击加 1</button>
```

当点击按钮的时候，span 标签里的值加 1。

传统的做法是：

```js
document.getElementById('button').addEventListener("click", function(){
    var container = document.getElementById("container");
    container.innerHTML = Number(container.innerHTML) + 1;
});
```

如果使用了 defineProperty：

```js
var obj = {
    value: 1
}

// 储存 obj.value 的值
var value = 1;

Object.defineProperty(obj, "value", {
    get: function() {
        return value;
    },
    set: function(newValue) {
        value = newValue;
        document.getElementById('container').innerHTML = newValue;
    }
});

document.getElementById('button').addEventListener("click", function() {
    obj.value += 1;
});
```

代码看似增多了，但是当我们需要改变 span 标签里的值的时候，直接修改 obj.value 的值就可以了。

然而，现在的写法，我们还需要单独声明一个变量存储 obj.value 的值，因为如果你在 set 中直接 `obj.value = newValue` 就会陷入无限的循环中。此外，我们可能需要监控很多属性值的改变，要是一个一个写，也很累呐，所以我们简单写个 watch 函数。使用效果如下：

```js
var obj = {
    value: 1
}

watch(obj, "value", function(newvalue){
    document.getElementById('container').innerHTML = newvalue;
})

document.getElementById('button').addEventListener("click", function(){
    obj.value += 1
});
```

我们来写下这个 watch 函数：

```js
(function(){
    var root = this;
    function watch(obj, name, func){
        var value = obj[name];

        Object.defineProperty(obj, name, {
            get: function() {
                return value;
            },
            set: function(newValue) {
                value = newValue;
                func(value)
            }
        });

        if (value) obj[name] = value
    }

    this.watch = watch;
})()
```

现在我们已经可以监控对象属性值的改变，并且可以根据属性值的改变，添加回调函数，棒棒哒~



## proxy

使用 defineProperty 只能重定义属性的读取（get）和设置（set）行为，到了 ES6，提供了 Proxy，可以重定义更多的行为，比如 in、delete、函数调用等更多行为。

Proxy主要用于改变对象的默认访问行为，实际上是在访问对象之前增加一层拦截，在任何对对象的访问行为都会通过这层拦截。在这层拦截中，我们可以增加自定义的行为。

基本语法如下：

```js
/*
 * target: 目标对象
 * handler: 配置对象，用来定义拦截的行为
 * proxy: Proxy构造器的实例
 */
var proxy = new Proxy(target,handler)
```

#### 基本用法

看个简单例子：

```js
// 目标对象
var target = {
	num:1
}
// 自定义访问拦截器
var handler = {
  // receiver: 操作发生的对象，通常是代理
  get:function(target,prop,receiver){
    console.log(target,prop,receiver)
  	return target[prop]*2
  },
  set:function(trapTarget,key,value,receiver){
    console.log(trapTarget.hasOwnProperty(key),isNaN(value))
  	if(!trapTarget.hasOwnProperty(key)){
    	if(typeof value !== 'number'){
      	throw new Error('入参必须为数字')
      }
      return Reflect.set(trapTarget,key,value,receiver)
    }
  }
}
// 创建target的代理实例dobuleTarget
var dobuleTarget = new Proxy(target,handler)
console.log(dobuleTarget.num) // 2

dobuleTarget.count = 2
// 代理对象新增属性，目标对象也跟着新增
console.log(dobuleTarget) // {num: 1, count: 2}
console.log(target)  // {num: 1, count: 2}
// 目标对象新增属性，Proxy能监听到
target.c = 2
console.log(dobuleTarget.c)  // 4 能监听到target新增的属性
```

例子里，我们通过Proxy构造器创建了target的代理dobuleTarget，即是代理了整个target对象，此时通过对dobuleTarget属性的访问都会转发到target身上，并且针对访问的行为配置了自定义handler对象。因此任何通过dobuleTarget访问target对象的属性，都会执行handler对象自定义的拦截操作。

这里面专业的描述是：

代理可以拦截JavaScript引擎内部目标的底层对象操作，这些操作被拦截后会触发响应特定操作的陷阱函数。例子里的陷阱函数就是get函数。

#### 陷阱函数汇总

总结下Proxy的陷阱函数：

| 陷阱函数                 | 覆写的特性                                                   |
| ------------------------ | ------------------------------------------------------------ |
| get                      | 读取一个值                                                   |
| set                      | 写入一个值                                                   |
| has                      | in操作符                                                     |
| deleteProperty           | Object.getPrototypeOf()                                      |
| getPrototypeOf           | Object.getPrototypeOf()                                      |
| setPrototypeOf           | Object.setPrototypeOf()                                      |
| isExtensible             | Object.isExtensible()                                        |
| preventExtensions        | Object.preventExtensions()                                   |
| getOwnPropertyDescriptor | Object.getOwnPropertyDescriptor()                            |
| defineProperty           | Object.defineProperty                                        |
| ownKeys                  | Object.keys() Object.getOwnPropertyNames()和Object.getOwnPropertySymbols() |
| apply                    | 调用一个函数                                                 |
| construct                | 用new调用一个函数                                            |

#### 陷阱函数应用

隐藏私有属性，以及不允许删除

```js
var obj = {
  // 以"_"下划线开头的为私有属性
  _type:'obj',
  name:'hello world'
}
var handler = {
  // 判断的是hasProperty,不是hasOwnProperty，拦截的是in操作符
   has:function(trapTarget,prop){
  	if(prop[0]=== '_'){
    	return false
    }
    return prop in trapTarget
  },
  // 拦截的是delete操作符
  deleteProperty:function(trapTarget,prop){
  	if(prop[0]=== '_'){
    	throw new Error('私有属性不能删除')
    }
    return true
  }
}
var proxy = new Proxy(obj,handler)
'_type' in proxy // false
delete proxy._type  // 报错：私有属性不能删除
```

#### Proxy递归代理

Proxy只代理对象的外层属性。例子如下：

```js
var target = {
  a:1,
  b:{
    c:2,
    d:{e:3}
  }
}
var handler = {
  get:function(trapTarget,prop,receiver){
    console.log('触发get:',prop)
    return Reflect.get(trapTarget,prop)
  },
  set:function(trapTarget,key,value,receiver){
    console.log('触发set:',key,value)
    return Reflect.set(trapTarget,key,value,receiver)
  }
}
var proxy = new Proxy(target,handler)

proxy.b.d.e = 4 
// 输出  触发get:b , 由此可见Proxy仅代理了对象外层属性。
```

如何解决呢？递归设置代理

```js
var target = {
  a:1,
  b:{
  	c:2,
    d:{e:3}
  }
}
var handler = {
  get:function(trapTarget,prop,receiver){
    var val = Reflect.get(trapTarget,prop)
    console.log('get',prop)
    if(val !== null && typeof val==='object'){
    	return new Proxy(val,handler) // 代理内层
    }
    return Reflect.get(trapTarget,prop)
  },
  set:function(trapTarget,key,value,receiver){
    console.log('触发set:',key,value)
    return Reflect.set(trapTarget,key,value,receiver)
  }
}
var proxy = new Proxy(target,handler)
proxy.b.d.e
// 输出： 均被代理
// get b
// get d
// get e 
```

从递归代理可以看出，如果对象内部要全部递归代理，Proxy可以只在调用时递归设置代理。



## 总结

1. Proxy是对整个对象的代理，而Object.defineProperty只能代理某个属性。
2. 对象上新增属性，Proxy可以监听到，Object.defineProperty不能。
3. 数组新增修改，Proxy可以监听到，Object.defineProperty不能。
4. 若对象内部属性要全部递归代理，Proxy可以只在调用的时候递归，而Object.definePropery需要一次完成所有递归，性能比Proxy差。
5. Proxy不兼容IE，Object.defineProperty不兼容IE8及以下
6. Proxy使用上比Object.defineProperty方便多。
