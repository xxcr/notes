

## 前言

看这篇文章之前希望你掌握：

+ `JavaScript`的事件循环
+ 浏览器进程和线程
+ `Promise`相关知识



## 简单说下JS的异步编程

### JS单线程、同步和异步

#### JS单线程

`Javascript`语言的执行环境是"单线程"。也就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务。

这种模式虽然实现起来比较简单，执行环境相对单纯，但是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段`Javascript`代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。

为什么`JavaScript`是单线程？

JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？

所以，为了避免复杂性，从一诞生，JavaScript就是单线程。

WebWorker，JS的多线程？

MDN的官方解释是：

```js
Web Worker为Web内容在后台线程中运行脚本提供了一种简单的方法。线程可以执行任务而不干扰用户界面

一个worker是使用一个构造函数创建的一个对象(e.g. Worker()) 运行一个命名的JavaScript文件 

这个文件包含将在工作线程中运行的代码; workers 运行在另一个全局上下文中,不同于当前的window

因此，使用 window快捷方式获取当前全局的范围 (而不是self) 在一个 Worker 内将返回错误
```

这样理解下：

- 创建`Worker`时，JS引擎向浏览器申请开一个子线程（子线程是浏览器开的，完全受主线程控制，而且不能操作DOM）
- JS引擎线程与worker线程间通过特定的方式通信（postMessage API，需要通过序列化对象来与线程交互特定的数据）

所以，如果有非常耗时的工作，请单独开一个Worker线程，这样里面不管如何翻天覆地都不会影响JS引擎主线程， 只待计算出结果后，将结果通信给主线程即可，perfect!

而且注意下，**JS引擎是单线程的**，这一点的本质仍然未改变，Worker可以理解是浏览器给JS引擎开的外挂，专门用来解决那些大量计算问题。



#### 同步和异步

1. 同步：连续不间断得执行多个任务，具有阻塞效应；
2. 异步：不连续得执行多个任务，不具有阻塞效应。

简单来理解就是：同步按你的代码顺序执行，异步不按照代码顺序执行。



### 1. 回调函数

回调函数是异步操作最基本的方法。以下代码就是一个回调函数的例子：

```js
ajax(url, () => {
    // 处理逻辑
})
```

但是回调函数有一个致命的弱点，就是容易写出**回调地狱（Callback hell）**。假设多个请求存在依赖性，你可能就会写出如下代码：

```js
ajax(url, () => {
    // 处理逻辑
    ajax(url1, () => {
        // 处理逻辑
        ajax(url2, () => {
            // 处理逻辑
        })
    })
})
```

- 优点：简单、容易理解和实现;

- 缺点：

- - 不利于代码的阅读和维护，各个部分之间高度耦合，使得程序结构混乱、流程难以追踪（尤其是多个回调函数嵌套的情况）;
  - 每个任务只能指定一个回调函数;
  - 不能使用 try catch 捕获错误，不能直接 return；
  - 极其容易写出回调地狱（Callback hell）。

### 2. 事件监听

异步任务的执行不取决于代码的顺序，而取决于某个事件是否发生。

例如：fnB必须等到fnA执行完成后才能执行。（这里采用的jQuery的写法）

```js
fnA.on('done', fnB) // 为fnA绑定一个事件，当fnA发生done事件，就执行fnB。

function fnA() {
  setTimeout(function () {

    ... // 处理逻辑

    fnA.trigger('done') // 执行完成后，立即触发done事件，开始执行fnB。
  }, 1000)
}
```

- 优点：

- - 可绑定多个事件，每个事件可以指定多个回调函数；
  - "去耦合"，有利于实现模块化。

- 缺点：

- - 整个程序都要变成事件驱动型，运行流程会变得很不清晰
  - 可读性较差，难以梳理出程序主流程。

### 3. 发布订阅

存在一个"信号中心"，某个任务执行完成，就向信号中心"发布"（publish）一个信号，其他任务可以向信号中心"订阅"（subscribe）这个信号，从而知道什么时候自己可以开始执行。这就叫做"发布/订阅模式"（publish-subscribe pattern），又称"观察者模式"（observer pattern）。

例如：fnB通过订阅done信号，触发执行。

```js
Watcher.subscribe('done', fnB) // fnB向信号中心Watcher订阅done信号。

function fnA() {
  setTimeout(function () {

    ... // 处理逻辑

    Watcher.publish('done') // 执行完成后，向信号中心Watcher发布done信号，从而引发fnB的执行。
  }, 1000)
}

function fnB(){

    ... // 处理逻辑

    Watcher.unsubscribe('done', fnB); // fnB完成执行后取消订阅
}
```

- 优点：

- - 支持简单的广播通信，当对象状态发生改变时，会自动通知已经订阅过的对象；
  - 发布者与订阅者耦合性降低，发布者只管发布一条消息出去，它不关心这条消息如何被订阅者使用，同时，订阅者只监听发布者的事件名，只要发布者的事件名不变，它不管发布者如何改变；
  - 可以通过查看“消息中心”，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。

- 缺点：

- - 创建“消息中心”需要消耗一定的时间和内存；
  - 虽然可以弱化对象之间的联系，如果过度使用的话，反而使代码可读性及可维护性降低。

### 4. Promise/A+

> 这个可以写一大篇文章了，这里简单过一下，毕竟主角是async/await

1. `Promise`本意是承诺，在程序中的意思就是承诺我过一段时间后会给你一个结果。

2. `Promise`的三种状态

   - Pending----`Promise`对象实例创建时候的初始状态
   - Fulfilled----可以理解为成功的状态
   - Rejected----可以理解为失败的状态

   **这个承诺一旦从等待状态变成为其他状态就永远不能更改状态了**

3. `promise`的链式调用

   - 每次调用返回的都是一个新的`Promise`实例(这就是then可用链式调用的原因)
   - 如果`then`中返回的是一个结果的话会把这个结果传递下一次`then`中的成功回调
   - 如果`then`中出现异常,会走下一个`then`的失败回调
   - 在 `then`中使用了`return`，那么 `return` 的值会被`Promise.resolve()` 包装
   - `then`中可以不传递参数，如果不传递会透到下一个`then`中
   - `catch` 会捕获到没有捕获的异常

4. 把之前的回调地狱例子改写为如下代码：

   ```js
   ajax(url)
     .then(res => {
         console.log(res)
         return ajax(url2) // 包装成 Promise.resolve(ajax(url2))
     }).then(res => {
         console.log(res)
         return ajax(url3)
     }).then(res => console.log(res))
   ```

   - 优点：

   - - 解决了回调地狱
     - 能够通过回调函数捕获错误

   - 缺点：

   - - 无法取消`Promise`，一旦新建它就会执行，无法中途取消
     - 如果不设置回调函数，Promise内部抛出的错误，不会反应到外部
     - 当处于Pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

### 5. Generator（生成器）

1. `Generator` 最大的特点就是可以控制函数的执行。
2. ES6 新引入了 `Generator` 函数，可以通过 `yield` 关键字，把函数的执行流挂起，通过`next()`方法可以切换到下一个状态，为改变执行流程提供了可能，从而为异步编程提供解决方案。
   - function 关键字与函数名之间有一个星号。
   - 语法上，首先可以把它理解成，Generator 函数是一个状态机，封装了多个内部状态。
   - Generator 函数除了状态机，还是一个遍历器对象生成函数。
   - 可暂停函数, yield可暂停，next方法可启动，每次返回的是yield后的表达式结果。
   - yield表达式本身没有返回值，或者说总是返回undefined。**next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值**。

2. 我们先来看个例子：

   ```js
   function *foo(x) {
     let y = 2 * (yield (x + 1))
     let z = yield (y / 3)
     return (x + y + z)
   }
   let it = foo(5)
   console.log(it.next())   // => {value: 6, done: false}
   console.log(it.next(12)) // => {value: 8, done: false}
   console.log(it.next(13)) // => {value: 42, done: true}
   ```

   我们逐行代码分析：

   - 首先 Generator 函数调用和普通函数不同，它会返回一个迭代器
   - 当执行第一次 next 时，传参会被忽略，并且函数暂停在 yield (x + 1) 处，所以返回 5 + 1 = 6
   - 当执行第二次 next 时，传入的参数12就会被当作上一个yield表达式的返回值，如果你不传参，yield 永远返回 undefined。此时 let y = 2 * 12，所以第二个 yield 等于 2 * 12 / 3 = 8
   - 当执行第三次 next 时，传入的参数13就会被当作上一个yield表达式的返回值，所以 z = 13, x = 5, y = 24，相加等于 42

3. 同样可以解决回调地狱的问题。

   ```js
   function *fetch() {
       yield ajax(urlA, () => {})
       yield ajax(urlB, () => {})
       yield ajax(urlC, () => {})
   }
   let it = fetch()
   let result1 = it.next()
   let result2 = it.next()
   let result3 = it.next()
   ```

   - 优点：

   - - 可分步执行并得到异步操作的结果；
     - 可知晓异步操作所处的过程；
     - 可切入修改异步操作的过程。

   - 缺点：

   - - 仍然需要使用异步的思维去阅读代码；
     - 手动迭代Generator函数较为麻烦。

### 6. Async/Await

> 这个是主角，下面详细讲一下

```js
async function fetch() {
    await ajax(url1)
    await ajax(url2)
    await ajax(url3)
}
```



## async/await

async 是“异步”的简写，而 await 可以认为是 async wait 的简写。

async 用于申明一个 function 是异步的，而 await 用于等待一个异步方法执行完成。

### 什么是async？

`async` 函数是 `Generator` 函数的语法糖。使用 关键字 `async` 来表示，在函数内部使用 `await` 来表示异步。

`async`是ES7新出的特性，表明当前函数是异步函数，不会阻塞线程导致后续代码停止运行。



 #### 怎么用？

申明之后就可以进行调用了

```js
async function asyncFn() {
　　return 'hello world'
}
asyncFn()
```

这样就表示这是异步函数，返回一个promise对象，如果function中返回的是一个值，`async`直接会用`Promise.resolve()`包裹一下返回。如果没有返回值，`Promise.resolve(undefined)`

> `Promise.resolve(x)` 可以看作是 `new Promise(resolve => resolve(x))` 的简写，可以用于快速封装字面量对象或其他对象，将其封装成 `Promise` 实例。

如果函数内部抛出异常或者是返回`reject`，都会使函数的`promise`状态为失败`reject`。

```js
async function e() {    
    throw new Error('1')
}
e().then(success => console.log('成功', success))   
   .catch(error => console.log('失败', error)) // 1

```



#### async 做一件什么事情？

**带 async 关键字的函数，它使得你的函数的返回值必定是 promise 对象**

也就是：

- 如果async关键字函数返回的不是promise，会自动用Promise.resolve()包装。

- 如果async关键字函数显式地返回promise，那就以你返回的promise为准。

  所以如果某个函数返回的本身就是promise的话，不需要使用async声明。



### 什么是await

只能在使用`async`定义的函数里面使用。

`await`是等待的意思，那么他在等什么呢？ 在MDN上写的是：

```js
[return_value] = await expression;
```

等的是一个表达式，那么表达式，可以是一个常量，变量，promise，函数等。

正常情况下，await 命令后面跟着的是 Promise ，如果不是的话，也会被转换成一个 立即 `resolve` 的 `Promise`

```js
async function  f() {
    return await 1
}
f().then( v => console.log(v)) // 1
```

当 `async` 函数执行到 `await` 的时候，`await`后面的函数会先执行一遍(比如await Fn()的Fn ,并非是下一行代码)，然后就会跳出整个`async`函数，出让其线程，来执行后面js代码，只有当其等待的基于`Promise` 的异步操作被兑现或被拒绝之后才会恢复线程。

**async 函数返回的 Promise 对象，必须等到内部所有的 await 命令的 Promise 对象执行完，才会发生状态改变**。



### 为什么要用async/await

解决回调地狱这个不用说了。

#### 相对于**Promise** 

1. 更好地处理 then 链

   - 看上面异步编程`Promise`、`async`代码： `Promise`这种方式充满了 `then()` 方法，如果处理流程复杂的话，整段代码将充满 `then`。语义化不明显，代码流程不能很好的表示执行流程。
   - 如何停止`Promise`链，是一大难点，是整个`Promise`最复杂的地方。比如：你想在第一个`then`就跳出链式，后面的不想执行了。

2. `Pomise`传递参数太过麻烦，看着很晕。

   举个栗子：假设一个业务，分多个步骤完成，每个步骤都是异步的，而且每一个步骤都需要之前每个步骤的结果。

   ```js
   function takeLongTime(n) {
       return new Promise(resolve => {
           setTimeout(() => resolve(n + 200), n)
       })
   }
   
   function step1(n) {
       console.log(`step1 with ${n}`)
       return takeLongTime(n)
   }
   
   function step2(m, n) {
       console.log(`step2 with ${m} and ${n}`)
       return takeLongTime(m + n)
   }
   
   function step3(k, m, n) {
       console.log(`step3 with ${k}, ${m} and ${n}`)
       return takeLongTime(k + m + n)
   }
   
   
   function doIt() {
       console.time("doIt")
       const time1 = 300;
       step1(time1)
           .then(time2 => {
               return step2(time1, time2)
                   .then(time3 => [time1, time2, time3])
           })
           .then(times => {
               const [time1, time2, time3] = times
               return step3(time1, time2, time3)
           })
           .then(result => {
               console.log(`result is ${result}`)
           })
   }
   doIt()
   ```

   有没有感觉有点复杂的样子？那一堆参数处理，就是 Promise 方案的死穴—— 参数传递太麻烦了，看着就晕！

   然后用`async/await` 来实现：

   ```javascript
   async function doIt() {
       console.time("doIt")
       const time1 = 300
       const time2 = await step1(time1)
       const time3 = await step2(time2)
       const result = await step3(time3)
       console.log(`result is ${result}`)
   }
   ```

   是不是感觉舒服多了。



#### 相对于Generator

看上面异步编程的`Generator`、`Promise`、`async`代码

- `Generator` 的方式解决了 `Promise` 的一些问题，流程更加直观、语义化。

- `*/yield`和`async/await`看起来其实已经很相似了，它们都提供了暂停执行的功能。

但是相较于 `Generator`，`async` 函数的改进在于下面四点：

- **内置执行器**。`Generator` 函数的执行必须依靠执行器，而 `async` 函数自带执行器，调用方式跟普通函数的调用一样
- **更好的语义**。`async` 和 `await` 相较于 `*` 和 `yield` 更加语义化
- **更广的适用性**。`co` 模块约定，`yield` 命令后面只能是 `Thunk` 函数或 `Promise`对象。而 `async` 函数的 `await` 命令后面则可以是 Promise 或者 原始类型的值（Number，string，boolean，但这时等同于同步操作）
- **返回值是 Promise**。`async` 函数返回值是 `Promise` 对象，比 `Generator` 函数返回的 `Iterator` 对象方便，可以直接使用 `then()` 方法进行调用。



这里的重点是自带了执行器，相当于把我们要额外做的(写执行器/依赖co模块)都封装了在内部。



#### 用同步的思路写异步逻辑

async/await 最大的优势就是我们可以用同步的思路来写异步的业务逻辑，所以代码整体看起来更加容易看懂。



### async/await的执行顺序

结合js的事件循环机制，我们来看看`async/await`的执行顺序：

```js
console.log('script start')

async function async1() {
await async2()
console.log('async1 end')
}
async function async2() {
console.log('async2 end')
}
async1()

setTimeout(function() {
console.log('setTimeout')
}, 0)

new Promise(resolve => {
console.log('Promise')
resolve()
})
.then(function() {
console.log('promise1')
})
.then(function() {
console.log('promise2')
})

console.log('script end')

/*
script start
async2 end
Promise
script end
async1 end
promise1
promise2
setTimeout
*/
```

分析一下：

1. 执行代码，输出`script start`。
2. 执行async1函数，此函数中又调用了async2函数，输出`async2 end`。回到async1函数，遇到了await，让出线程，其后的代码放入微任务队列。
3. 遇到setTimeout，产生一个宏任务
4. 执行Promise，输出`Promise`。遇到then，产生第一个微任务
5. 继续执行代码，输出`script end`
6. 代码逻辑执行完毕(当前宏任务执行完毕)，开始执行当前宏任务产生的微任务队列，输出第二步被扔到微任务队列的任务`async1 end`。
7. 执行第 4 步被扔到微任务队列的任务，输出`promise1`，又产生一个微任务，加在后面。
8. 执行产生的微任务，输出`promise2`,当前微任务队列执行完毕。
9. 最后，执行下一个宏任务，即执行setTimeout，输出`setTimeout`



### async/await注意的3个点

1. async/await中错误处理
2. 小心 await 阻塞
3. forEach 中用 await 

下面详细说一下。



### async/await中错误处理

先来看下面的例子：

```js
let a
async function f() {
    await Promise.reject('error')
    a = await 1 // 这段 await 并没有执行
}
f().then(v => console.log(a))
```


上面的代码，当 `async` 函数中只要一个 `await` 出现 reject 状态，则后面的都不会被执行。

`async`函数接收到返回的值，发现不是`异常`或者`reject`，则判定成功，这里可以`return`各种数据类型的值，`false`,`NaN`,`undefined`...总之，都是`resolve`

但是返回如下结果会使`async`函数判定失败`reject`

1. 内部含有直接使用并且未声明的变量或者函数。
2. 内部抛出一个错误`throw new Error`或者返回`reject`状态`return Promise.reject('执行失败')`
3. 函数方法执行出错（🌰：Object使用push()）等等...



那上面的代码，我希望 `await` 出现 reject 状态，我还是需要后面代码执行，怎么办？

1. 用try-catch来做错误捕捉

   ```js
   let a
   async function correct() {
       try {
           await Promise.reject('error')
       } catch (error) {
           console.log(error)
       }
       a = await 1
       return a
   }
   
   correct().then(v => console.log(a)) // 1
   ```

2. 用promise的catch来做错误捕捉

   ```js
   let a
   async function correct() {
       await Promise.reject('error').catch((err) => {
           console.log(err)
       })
       a = await 1
       return a
   }
   
   correct().then(v => console.log(a)) // 1
   ```

3. 更懒更高阶的方法：通过一个 `webpack loader` 来自动注入 `try/catch` 代码。



### 小心 await 阻塞

由于 await 能够阻塞 async 函数的运行，所以代码看起来更像同步的代码，更容易阅读和理解。

但是要小心 await 阻塞，因为有些阻塞是不必要的，不恰当使用可能会影响代码的性能。

看一个错误的栗子：

```js
async function Fn() {
    let a = await ajax(urla)
    let b = await ajax(urlb)
    console.log(a + b)
}
```

上面这个代码是想拼接两个接口请求回来的值，看上去好像没什么问题，但是：请求a接口的时候，会阻塞掉Fn方法，b接口就不会去请求，要等a接口返回数据、本次宏任务都执行完，将请求回来的数据赋值给a后，才会去请求b接口。这样写等于每次异步`http`请求线程每次只要维护一个请求。

这样严重影响性能，可能我请求b接口的时候，js没有什么要执行的了，就要一直等b接口请求回来数据。异步`http`请求线程明明可以维护多个请求。

所以我们可以这样写：

```js
async function Fn() {
    let aPromise = ajax(urla)
    let bPromise = ajax(urlb)
    let a = await aPromise
    let b = await bPromise
    console.log(a + b)
}

```



这样写a请求和b请求可以同时请求，提高性能。

当然，如果你熟悉 Promise 的话，可以直接使用 `Promise.all` 的方式来处理，或者 await 后面跟 `Promise.all` 这里就不展开讲了。

```js
async function Fn() {
    Promise.all([ajax(urla), ajax(urlb)]).then((values) => {
      console.log(values) // [a, b]
    })
}
```





### forEach 中用 await 

#### 问题

对于异步代码，`forEach` 并不能保证按顺序执行。

举个栗子：

三个请求，按顺序循环得到数据：

```js
const urls = [
  'https://1',
  'https://2',
  'https://3'
]

async function test() {
  await urls.forEach(async item => { 
    const res = await ajax(item)
    console.log(res)
  })
  
  console.log('结束')
}

test()
```



我们期望的结果是：

```js
1
2
3
结束
```



但是实际上可能会输出：

一开始打印出结束

```js
结束
2
1
3
```



#### 原因

这是为什么呢？我想我们有必要看看`forEach`底层怎么实现的。

```js
// 核心逻辑
for (var i = 0; i < length; i++) {
  if (i in array) {
    var element = array[i]
    callback(element, i, array)
  }
}
```

可以看到，`forEach` 拿过来直接执行了，这就导致它无法保证异步任务的执行顺序。

`for`第一个时，在内部又创建了一个 `async 、await` 形式的方法，`await`阻塞后面的代码，让出当前`async`方法的线程，然后`for`第二个。

当 `forEach` 函数执行完的时候，相当于创建了 3 个 方法。比如后面的任务用时短，那么异步`http`线程就先将其回调放到宏任务队尾。所以先执行。

`map`方法也会。



#### 解决方案

如何来解决这个问题呢？

其实也很简单, 

1. 我们利用普通的`for`循环或者`for...of`就能轻松解决。这种方法是串行的，也就是发送一个请求得到结果，再去发送下一个请求。

   > 使用`for...of`循环的一个主要缺点是它与Javascript中的其他循环选项相比性能不够好。但是，将性能参数用于await异步调用时，性能参数可以忽略不计。

   ```js
   const urls = [
     'https://1',
     'https://2',
     'https://3'
   ]
   
   async function test() {
     for(const item of urls) {
   	const res = await ajax(item)
   	console.log(res)
     }
     
     console.log('结束')
   }
   
   test()
   ```

1. 可以用`map`方法返回一个和url数组对应的一个`promise` 数组。然后用`Promise.all`。这种方法是并行的。

   > `Promise.all` 方法会按照并行的模式，将所有请求一次性全部发送出去，然后等待接收到全部结果后，按照顺序打印出来而已。它并不会按照顺序发送一个请求，收到结果后再发送下一个请求。

   ```js
   const urls = [
     'https://1',
     'https://2',
     'https://3'
   ]
   
   async function test() {
     let a = urls.map(item => ajax(item))
     
     console.log(await Promise.all(a))
     
     console.log('结束')
   }
   
   test()
   ```

   



#### for...of 解决原理——Iterator

这个问题看起来好像很简单就能搞定，你有想过这么做为什么可以成功吗？

我们都知道：`for...of`并不像`forEach`那么简单粗暴的方式去遍历执行，而是采用一种特别的手段——`迭代器`去遍历。

首先，对于数组来讲，它是一种`可迭代数据类型`。那什么是`可迭代数据类型`呢？

> 原生具有[Symbol.iterator]属性数据类型为可迭代数据类型。如数组、类数组（如arguments、NodeList）、Set和Map。

可迭代对象可以通过迭代器进行遍历。

```js
const urls = [
  'https://1',
  'https://2',
  'https://3'
]
// 这就是迭代器
let iterator = urls[Symbol.iterator]()
console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())


// {value: 'https://1', done: false}
// {value: 'https://2', done: false}
// {value: 'https://3', done: false}
// {value: undefined, done: true}
```

因此，我们的代码可以这样来组织:

```js
async function test() {
  const urls = [
      'https://1',
      'https://2',
      'https://3'
  ]
  // 这就是迭代器
  let iterator = urls[Symbol.iterator]()
  let res = iterator.next()
  while(!res.done) {
    let value = res.value
    console.log(value)
    await ajax(value)
    res = iterater.next()
  }
  console.log('结束')
}
// 1
// 2
// 3
// 结束
```

多个任务成功地按顺序执行！其实刚刚的for...of循环代码就是这段代码的语法糖。



#### 重新认识生成器

回头再看看用`iterator`遍历`urls`这个数组的代码。

咦？返回值有`value`和`done`属性，生成器也可以调用 `next`,返回的也是这样的数据结构，这么巧?!

没错，**生成器**本身就是一个**迭代器**。

既然属于迭代器，那它就可以用`for...of`遍历了吧？

当然没错。这里就不展开了。



## 参考文献

1. [JS 异步编程六种方案](https://juejin.cn/post/6844903760280420366)
2. [[译]async-await 数组循环的几个坑](https://juejin.cn/post/6844903793117626376)
3. [理解async/await](https://juejin.cn/post/6844903960910757902)
4. [一次性让你懂async/await，解决回调地狱](https://juejin.cn/post/6844903621360943118)
5. [细说 async/await 相较于 Promise 的优势](https://juejin.cn/post/6844903760217505805)
6. [为什么要使用 async/await ？](https://juejin.cn/post/6967000766032658440)
7. [理解 async/await](https://juejin.cn/post/6844903487805849613)



