> 本文基本是复制该文章[你真的了解ES6的Set，WeakSet，Map和WeakMap吗？](https://juejin.cn/post/6844903587764502536)
> 作者：MuffinFish34765
> 链接：https://juejin.cn/post/6844903587764502536
> 来源：掘金

> 关于 `cookie` ，更多请移步 [深挖前端 JavaScript 知识点 —— 史上最全面、最详细的 Cookie 总结](https://juejin.cn/post/6877133657228869639)

### localStorage和sessionStorage

#### 两者的共同点在于：

1、存储大小均为5M左右
2、都有同源策略限制
3、仅在客户端中保存，不参与和服务器的通信

#### 两者的不同点在于：

1、`生命周期` —— 数据可以存储多少时间

- localStorage: 存储的数据是永久性的，除非用户人为删除否则会一直存在。
- sessionStorage: 与存储数据的脚本所在的标签页的有效期是相同的。一旦窗口或者标签页被关闭，那么所有通过 sessionStorage 存储的数据也会被删除。

2、`作用域` —— 谁拥有数据的访问权

- localStorage: 在同一个浏览器内，`同源文档`之间共享 localStorage 数据，可以互相读取、覆盖。
- sessionStorage: 与 localStorage 一样需要同一浏览器同源文档这一条件。不仅如此，sessionStorage 的作用域还被限定在了窗口中，也就是说，只有同一浏览器、同一窗口的同源文档才能共享数据。 

为了更好的理解`sessionStorage`,我们来看个例子：

例如你在浏览器中打开了两个相同地址的页面A、B,虽然这两个页面的源完全相同，但是他们还是不能共享数据，因为他们是不同窗口中的。但是如果是一个窗口中，有两个同源的`iframe`元素的话，这两个`iframe`的 sessionStorage 是可以互通的。

#### API

```js
//sessionStorage用法相同
localStorage.setItem("name",1);   // 以"x"为名字存储一个数值
localStorage.getItem("name");     // 获取数值
localStorage.key(i);              // 获取第i对的名字
localStorage.removeItem("name");  // 获取该对的值
localStorage.clear();             // 全部删除
```



### Cookie

##### 基本概念

Cookie是小甜饼的意思，主要有以下特点：

1、顾名思义，Cookie 确实非常小，它的大小限制为4KB左右

2、主要用途是保存登录信息和标记用户(比如购物车)等，不过随着localStorage的出现，现在购物车的工作Cookie承担的较少了

3、一般由服务器生成，可设置失效时间。如果在浏览器端生成Cookie，默认是关闭浏览器后失效

4、每次都会携带在HTTP头中，如果使用cookie保存过多数据会带来性能问题

5、原生API不如storage友好，需要自己封装函数

##### 用法(API)

服务端向客户端发送的cookie(HTTP头,不带参数)：
`Set-Cookie: <cookie-name>=<cookie-value>` (name可选)

服务端向客户端发送的cookie(HTTP头，带参数)：
`Set-Cookie: <cookie-name>=<cookie-value>;(可选参数1);(可选参数2)`

客户端设置cookie：

```js
document.cookie = "<cookie-name>=<cookie-value>;(可选参数1);(可选参数2)"
复制代码
```



可选参数：
`Expires=<date>`：cookie的最长有效时间，若不设置则cookie生命期与会话期相同

`Max-Age=<non-zero-digit>`：cookie生成后失效的秒数

`Domain=<domain-value>`：指定cookie可以送达的主机域名，若一级域名设置了则二级域名也能获取。

`Path=<path-value>`：指定一个URL，例如指定path=/docs，则”/docs”、”/docs/Web/“、”/docs/Web/Http”均满足匹配条件

`Secure`：必须在请求使用SSL或HTTPS协议的时候cookie才回被发送到服务器

`HttpOnly`：客户端无法更改Cookie，客户端设置cookie时不能使用这个参数，一般是服务器端使用

示例：

```js
Set-Cookie: sessionid=aes7a8; HttpOnly; Path=/

document.cookie = "KMKNKK=1234;Sercure"
复制代码
```



可选前缀：
`__Secure-`：以`__Secure-`为前缀的cookie，必须与secure属性一同设置，同时必须应用于安全页面（即使用HTTPS）

`__Host-`：以`__Host-`为前缀的cookie，必须与secure属性一同设置，同时必须应用于安全页面（即使用HTTPS）。必须不能设置domian属性（这样可以防止二级域名获取一级域名的cookie），path属性的值必须为”/“。

前缀使用示例：

```js
Set-Cookie: __Secure-ID=123; Secure; Domain=example.com
Set-Cookie: __Host-ID=123; Secure; Path=/

document.cookie = "__Secure-KMKNKK=1234;Sercure"
document.cookie = "__Host-KMKNKK=1234;Sercure;path=/"
```



### Session

##### 基本概念

Session是在无状态的HTTP协议下，服务端记录用户状态时用于标识具体用户的机制。它是在服务端保存的用来跟踪用户的状态的数据结构，可以保存在文件、数据库或者集群中。

在浏览器关闭后这次的Session就消失了，下次打开就不再拥有这个Session。其实并不是Session消失了，而是Session ID变了，服务器端可能还是存着你上次的Session ID及其Session 信息，只是他们是无主状态，也许一段时间后会被删除。

大多数的应用都是用Cookie来实现Session跟踪的，第一次创建Session的时候，服务端会在HTTP协议中告诉客户端，需要在Cookie里面记录一个SessionID，以后每次请求把这个会话ID发送到服务器

#### 与Cookie的关系与区别：

1、`Session`是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中，`Cookie`是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现Session的一种方式。

2、`Cookie`的`安全性`一般，他人可通过分析存放在本地的`Cookie`并进行`Cookie`欺骗。在安全性第一的前提下，选择`Session`更优。重要交互信息比如权限等就要放在`Session`中，一般的信息记录放`Cookie`就好了。 

3、单个`Cookie`保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个`Cookie`。 

4、当访问增多时，`Session`会较大地占用服务器的性能。考虑到减轻服务器性能方面，应当适时使用`Cookie`。 

5、`Session`的运行依赖`Session ID`，而`Session ID`是存在 Cookie 中的。也就是说，如果浏览器禁用了`Cookie`,`Session`也会失效（但是可以通过其它方式实现，比如在`url`中传递`Session ID`,即sid=xxxx）。
