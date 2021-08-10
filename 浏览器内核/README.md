> 本文基本是复制该文章[浏览器专题系列 - 浏览器内核](https://juejin.cn/post/6926729677088227342)
> 作者：粥里有勺糖
> 链接：https://juejin.cn/post/6926729677088227342
> 来源：掘金



## 常见浏览器内核

| 浏览器名称 | 内核                  | 补充                                                         |
| ---------- | --------------------- | ------------------------------------------------------------ |
| IE         | Trident               | 主要包含在 window操作系统的 IE浏览器中                       |
| firefox    | Gecko                 | Gecko的特点是代码完全公开，因此，其可开发程度很高            |
| Safari     | webkit                | 苹果公司自己的内核，包含WebCore排版引擎及JavaScriptCore解析引擎 |
| chrome     | Chromium/Blink/webkit | Blink是开源引擎WebKit中WebCore组件的一个分支                 |
| Opera      | blink/Webkit/Presto   | 现在跟随chrome的步伐，同时参与开发                           |


最开始渲染引擎和JS引擎并没有明确区分，随着不断的迭代，JS引擎越来越独立，内核更就倾向于只指渲染引擎（渲染进程）



## 移动端内核

主要有：

- webkit: IOS内置浏览器，Android4.4之前
- chromium：Android4.4之后内置
- blink
- trident：~~Windows Phone 8 这个玩意儿感觉不用关注了~~
- u3: UC打造 -> UC浏览器
- x5: 腾讯打造 -> QQ浏览器，腾讯系App内置webview



## 总结

### 兼容问题

正由于内核的 `"多彩缤纷"`, 前端开发者最头疼的问题莫过于此

#### PC

最头痛的就是兼容IE,尽管js能力可以通过polyfill支持一部分，但部分样式能力的缺失是无法弥补的

#### 移动端

最头痛的就是在使用跨端开发框架时（uni-app,React Native等）样式在双端的差异有时比较大

尤其是在低版本的Android机上，需要花费大量的去做兼容测试与视图适配

**浏览器内核能力的统一，还有很长的路要走，需要一个契机**

### 现状

- google、opera拥抱的blink
- IOS拥抱webkit
- Firefox拥抱Gecko
- 微软新版Edge已经采用Chromium内核，旧版为EdgeHTML

