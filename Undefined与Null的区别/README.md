> 本文基本是复制该文章[JS 基础篇(七)：Undefined与Null的区别](https://juejin.cn/post/6844903777506426893)
> 作者：水木同学_
> 链接：https://juejin.cn/post/6844903777506426893
> 来源：掘金

##  JS 基础篇(七)：Undefined与Null的区别

### 一、基本数据类型

在介绍undefined与null之前，我们先来了解一下ECMAScript中的数据类型。在ECMAScript中有七种简单数据类型(也称为基本数据类型): Undefined、Null、Boolean、Number 和 String、Symbol (ES6中引入) 。还有一种复杂数据类型——Object。后面还加入了bigInt。

Undefined和Null都只有一个值，分别对应着undefined和null。这两种不同类型的值，既有着不同的语义和场景，又表现出较为相似的行为。

### 二、undefined

undefined 的字面意思就是：未定义的值 。这个值的语义是，希望表示一个变量最原始的状态，而非人为操作的结果 。 这种原始状态会在以下 4 种场景中出现：

#### **1、声明一个变量，但是没有赋值**

```js
var foo;
console.log(foo); // undefined
```

访问 foo，返回了 undefined，表示这个变量自从声明了以后，就从来没有使用过，也没有定义过任何有效的值。

#### **2、访问对象上不存在的属性或者未定义的变量**

```js
console.log(Object.foo); // undefined
console.log(typeof demo); // undefined
```

访问 Object 对象上的 foo 属性，返回 undefined ， 表示Object 上不存在或者没有定义名为 foo 的属性；对未声明的变量执行typeof操作符返回了undefined值。

#### **3、函数定义了形参，但没有传递实参**

```js
//函数定义了形参 a
function fn(a) {
    console.log(a); // undefined
}
fn(); //未传递实参
```

函数 fn 定义了形参 a，但 fn 被调用时没有传递参数，因此，fn 运行时的参数 a 就是一个原始的、未被赋值的变量。

#### **4、使用void对表达式求值**

```js
void 0 ; // undefined
void false; // undefined
void []; // undefined
void null; // undefined
void function fn(){} ; // undefined
```

ECMAScript 明确规定 void 操作符 对任何表达式求值都返回 undefined ，这和函数执行操作后没有返回值的作用是一样的，JavaScript 中的函数都有返回值，当没有 return 操作时，就默认返回一个原始的状态值，这个值就是 undefined，表明函数的返回值未被定义。

因此，undefined 一般都来自于某个表达式最原始的状态值，不是人为操作的结果。当然，你也可以手动给一个变量赋值 undefined，但这样做没有意义，因为一个变量不赋值就是 undefined 。



### 三、null

null 的字面意思是：空值 。这个值的语义是，希望表示一个对象被人为的重置为空对象，而非一个变量最原始的状态 。 在内存里的表示就是，栈中的变量没有指向堆中的内存对象。

#### **2、特殊的typeof null**

当我们使用typeof操作符检测`null`值，我们理所应当地认为应该返"Null"类型呀，但是事实返回的类型却是`object`。

```js
var data = null;
console.log(typeof data); // "object"
```

是不是很奇怪？其实我们可以从两方面来理解这个结果:

- 一方面从逻辑角度来看，null值表示一个空对象指针，它代表的其实就是一个空对象，所以使用`typeof`操作符检测时返回`object`也是可以理解的。
- 另一方面，其实在JavaScript 最初的实现中，JavaScript 中的值是由一个表示类型的标签和实际数据值表示的(对象的类型标签是 0)。由于 null 代表的是空指针（大多数平台下值为 0x00），因此，null的类型标签也成为了 0，typeof null就错误的返回了"object"。在ES6中，当时曾经有提案为历史平凡, 将type null的值纠正为null, 但最后提案被拒了,所以还是保持"object"类型。

### 四、总结

用一句话总结两者的区别就是：undefined 表示一个变量自然的、最原始的状态值，而 null 则表示一个变量被人为的设置为空对象，而不是原始状态。所以，在实际使用过程中，为了保证变量所代表的语义，不要对一个变量显式的赋值 undefined，当需要释放一个对象时，直接赋值为 null 即可。
