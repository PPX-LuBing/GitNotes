## 网站开发中，如何实现图片的懒加载
懒加载，顾名思义，在当前网页，滑动页面到能看到图片的时候再加载图片

故问题拆分成两个：

1. 如何判断图片出现在了当前视口 （即如何判断我们能够看到图片）
2. 如何控制图片的加载  

**方案一: 位置计算 + 滚动事件 (Scroll) + DataSet API**
**如何判断图片出现在了当前视口**
`clientTop`，`offsetTop`，`clientHeight`以及 `scrollTop` 各种关于图片的高度作比对

这些高度都代表了什么意思？

这我以前有可能是知道的，那时候我比较单纯，喜欢死磕。我现在想通了，背不过的东西就不要背了

所以它有一个问题：复杂琐碎不好理解！

仅仅知道它静态的高度还不够，我们还需要知道动态的

如何动态？监听 window.scroll 事件

**如何控制图片的加载**  
```html <img data-src="shanyue.jpg" />```  
首先设置一个临时 Data 属性 `data-src`，控制加载时使用 src 代替 `data-src`，可利用 `DataSet API` 实现

```img.src = img.datset.src```

**方案二: getBoundingClientRect API + Scroll with Throttle + DataSet API
改进一下**

**如何判断图片出现在了当前视口**  
引入一个新的 API， Element.`getBoundingClientRect()` 方法返回元素的大小及其相对于视口的位置。

![](https://mdn.mozillademos.org/files/15087/rect.png)
那如何判断图片出现在了当前视口呢，根据示例图示意，代码如下，这个就比较好理解了，就可以很容易地背会(就可以愉快地去面试了)。  
``` javascript
// clientHeight 代表当前视口的高度
img.getBoundingClientRect().top < document.documentElement.clientHeight;
```
监听 `window.scroll` 事件也优化一下

加个节流器，提高性能。工作中一般使用 lodash.throttle 就可以了，万能的 lodash 啊！
``` javascript
_.throttle(func, [(wait = 0)], [(options = {})]);
```  
参考  [什么是防抖和节流，他们的应用场景有哪些](https://github.com/shfshanyue/Daily-Question/issues/3)，或者[前端面试题](https://q.shanyue.tech/fe/js/3.html)  

**方案三: IntersectionObserver API + DataSet API**  
再改进一下

**如何判断图片出现在了当前视口**
方案二使用的方法是: `window.scroll` 监听 `Element.getBoundingClientRect()` 并使用 `_.throttle` 节流

一系列组合动作太复杂了，于是浏览器出了一个三合一事件: `IntersectionObserver API`，一个能够监听元素是否到了当前视口的事件，一步到位！

事件回调的参数是 [IntersectionObserverEntry] (https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry)的集合，代表关于是否在可见视口的一系列值

其中，entry.isIntersecting 代表目标元素可见
``` javascript
const observer = new IntersectionObserver((changes) => {
  // changes: 目标元素集合
  changes.forEach((change) => {
    // intersectionRatio
    if (change.isIntersecting) {
      const img = change.target;
      img.src = img.dataset.src;
      observer.unobserve(img);
    }
  });
});

observer.observe(img);
```
当然，`IntersectionObserver` 除了给图片做懒加载外，还可以对单页应用资源做预加载。

如在 next.js v9 中，会对视口内的资源做预加载，可以参考 next 9 production optimizations(opens new window)
``` javascript
<Link href="/about">
  <a>关于山月</a>
</Link>
```
**方案四: LazyLoading 属性**
浏览器觉得懒加载这事可以交给自己做，你们开发者加个属性就好了。实在是...！
``` javascript
<img src="shanyue.jpg" loading="lazy" />
```
不过目前浏览器兼容性不太好，关于 loading 属性的文章也可以查看 [Native image lazy-loading for the web!](https://addyosmani.com/blog/lazy-loading/)
## CSP 是干什么用的了
CSP 只允许加载指定的脚本及样式，最大限度地防止 XSS 攻击，是解决 XSS 的最优解。CSP 的设置根据加载页面时 http 的响应头 Content Security Policy 在服务器端控制。

1. 外部脚本可以通过指定域名来限制：`Content-Security-Policy: script-src 'self'，self 代表只加载当前域名
2. 如果网站必须加载内联脚本 (inline script) ，则可以提供一个 nonce 才能执行脚本，攻击者则无法注入脚本进行攻击。Content-Security-Policy: script-src 'nonce-xxxxxxxxxxxxxxxxxx'  
通过 devtools -> network 可见 github 的 CSP 配置

Content Security Policy (CSP)
介绍：

1. 解决 XSS 最优办法  
2. 可以设置信任域名才可以访问 script / audio / video / image ...
防止 XSS 例子： 攻击者通过 恶意脚本(假设有执行外部脚本) 注入到系统内，显示给访问用户，以此来获取用户信息 我们可以通过 CSP 来设置信任域名才可以执行 .js 脚本。

如何设置：

1. HTTP 请求头
2. Meta 标签
MDN：https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP

兼容性：IE >= 10
## 如何实现页面文本不可复制

有 CSS 和 JS 两种方法，以下任选其一或结合使用

使用 CSS 如下：
`user-select: none;`

或使用 JS 如下，监听 selectstart 事件，禁止选中。

当用户选中一片区域时，将触发 selectstart 事件，Selection API 将会选中一片区域。禁止选中区域即可实现页面文本不可复制。

```javascript
document.body.onselectstart = (e) => {
	e.preventDefault();
};

document.body.oncopy = (e) => {
	e.preventDefault();
};
```

## 异步加载 js 脚本时，async 与 defer 有何区别

以下图片取自 whatwg 的规范，可以说是最权威的图文解释了，详细[参考原文](https://html.spec.whatwg.org/multipage/scripting.html#the-script-element)

<img src="https://html.spec.whatwg.org/images/asyncdefer.svg" alt="async 与 defer 区别">

async 与 defer 区别

在正常情况下，即 `<script> `没有任何额外属性标记的情况下，有几点共识

JS 的脚本分为加载、解析、执行几个步骤，简单对应到图中就是 fetch (加载) 和 execution (解析并执行)
JS 的脚本加载(fetch)且执行(execution)会阻塞 DOM 的渲染，因此 JS 一般放到最后头
而 defer 与 async 的区别如下:

相同点: 异步加载 (fetch)

不同点:
async 加载(fetch)完成后立即执行 (execution)，因此可能会阻塞 DOM 解析；

defer 加载(fetch)完成后延迟到 DOM 解析完成后才会执行(execution)\*\*，但会在事件 DomContentLoaded 之前 #拓展
当以下 index.js 加载时，属性是 async 与 defer 时，输出有何不同？

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="UTF-8" />
		<title></title>
	</head>
	<body>
		<script src="./defer.js" defer></script>
		<script src="./async.js" async></script>
		<script>
			console.log("Start");
			document.addEventListener("DOMContentLoaded", () => {
				console.log("DCL");
			});
		</script>
	</body>
</html>
```

**derfer.js**

`console.log("Defer Script")`;

**async.js**

`console.log("Async Script");`

> 答：defer 总是在 DCL 之前输出，但是 async 有可能之前也有可能之后

## load 事件与 DomContentLoaded 事件的先后顺序

当初始的 HTML 文档被完全加载和解析完成之后，DOMContentLoaded 事件被触发，而无需等待样式表、图像和子框架的完全加载.

当整个页面及所有依赖资源如样式表和图片都已完成加载时，将触发load事件

## React/Vue 中的 router 实现原理如何

前端路由实现的本质是监听 url 变化，实现方式有两种：Hash 模式和 History 模式，无需刷新页面就能重新加载相应的页面。 Hash url 的格式为`www.a.com/#/`，当#后的哈希值发生变化时，通过 hashchange 事件监听，然后页面跳转。 History url 通过`history.pushState`和`history.replaceState`改变 url。 两种模式的区别：

- hash 只能改变#后的值，而 history 模式可以随意设置同源 url；
- hash 只能添加字符串类的数据，而 history 可以通过 API 添加多种类型的数据；
- hash 的历史记录只显示之前的www.a.com而不会显示 hash 值，而 history 的每条记录都会进入到历史记录；
- hash 无需后端配置且兼容性好，而 history 需要配置index.html用于匹配不到资源的情况。

前端路由有两种实现方式:

**history API**
- 通过 `history.pushState()` 跳转路由
- 通过 `popstate event` 监听路由变化，但无法监听到 `history.pushState()` 时的路由变化
**hash**
- 通过 `location.hash` 跳转路由
- 通过 `hashchange event` 监听路由变化

## 前端如何实现文件上传功能

将 `input` 的类型设置为 `file`，再加一个按钮就行
``` html
<input type="file" ref="referenceUpload" @change="onUpload"></input>
<button type="primary" style="margin: 0px 0px 0px -83px;">上传文件</button>
```
## 什么是 HTML 的实体编码 (HTML Entity Encode)
HTML 实体是一段以连字号（&）开头、以分号（;）结尾的字符串。用以显示不可见字符及保留字符 (如 HTML 标签)

在前端，一般为了避免 XSS 攻击，会将 <> 编码为 &lt; 与 &gt;，这些就是 HTML 实体编码。

在 [whatwg](https://html.spec.whatwg.org/multipage/named-characters.html#named-character-references) 中可查看实体编码数据。

在 HTML 转义时，仅仅只需要对六个字符进行编码: ` &, <, >, ", ', ``` `。可使用 [he](https://npm.devtool.tech/he)这个库进行编码及转义
```
// 实体编码
> he.encode('<img src=""></img>')
< "&#x3C;img src=&#x22;&#x22;&#x3E;&#x3C;/img&#x3E;"

// 转义
> he.escape('<img src=""></img>')
< "&lt;img src=&quot;&quot;&gt;&lt;/img&gt;"
```
## 如何取消请求的发送
根据发送网络请求的 API 不同，取消方法不同

- xhr
- fetch
- axios

**XHR 使用 `xhr.abort()`**
``` javascript
const xhr = new XMLHttpRequest(),
method = "GET",
url = "https://developer.mozilla.org/";
xhr.open(method, url, true);
xhr.send();
// 取消发送请求
xhr.abort();
```
**fetch 使用 AbortController**
> `AbortController`文档见[AbortSignal - MDN](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)，它不仅可以取消`Fetch`请求发送，同样也可以取消事件的监听(通过 `addEventListener` 的第三个参数 `signal` 控制)

1. 发送请求时使用一个`signal`选项控制`fetch`请求
2. `control.abort()`用以取消请求发送
3. 取消请求发送之后会得到异常 `AbortError`
``` javascript
const controller = new AbortController()
const signal = controller.signal

const downloadBtn = document.querySelector('.download');
const abortBtn = document.querySelector('.abort');

downloadBtn.addEventListener('click', fetchVideo);

// 点击取消按钮时，取消请求的发送
abortBtn.addEventListener('click', function() {
  controller.abort();
  console.log('Download aborted');
});
function fetchVideo() {
  ...
  fetch(url, {signal}).then(function(response) {
    ...
  }).catch(function(e) {
   // 请求被取消之后将会得到一个 AbortError
    reports.textContent = 'Download error: ' + e.message;
  })
}
```
**Axios: `xhr` 与 `http/https`**  
`Axios`中通过`cancelToken`取消请求发送
``` javascript
const CancelToken = axios.CancelToken;
const source = CancelToken.source();
axios
  .get("/user/12345", {
    cancelToken: source.token,
  })
  .catch(function (thrown) {
    if (axios.isCancel(thrown)) {
      console.log("Request canceled", thrown.message);
    } else {
      // handle error
    }
  });

axios.post(
  "/user/12345",
  {
    name: "new name",
  },
  {
    cancelToken: source.token,
  }
);

// cancel the request (the message parameter is optional)
source.cancel("Operation canceled by the user.");
```  
而其中的原理可分为两部分

- 浏览器端: 基于 XHR，`xhr.abort()`，见源码[axios/lib/adapters/xhr.js]((https://github.com/axios/axios/blob/v0.21.1/lib/adapters/xhr.js#L165))
- Node 端: 基于 http/https/follow-redirects，使用 `request.abort()`，见源码[axios/lib/adapters/http.js](https://github.com/axios/axios/blob/v0.21.1/lib/adapters/http.js#L289)  
## DOM 中如何阻止事件默认行为，如何判断事件否可阻止？  
- `e.preventDefault()`: 取消事件
- `e.cancelable`: 事件是否可取消  

如果 `passive` 设置为 true 那其实 `preventDefault` 就会无效 因为 `passive` 为 true 会导致初始化的时候 `cancelable` 为 false

`event.preventDefault` 不能取消的没有固定哪一个类 主要是在规范中有没有定义 `Default Action` 还有即使是定义了 `Default Action` 那在实际中可能也会在不同的触发时间存在或不存在默认行为 所以可以依赖 event.cancelable 来处理 Default Action

如果只是简单列举下 具体可以去 https://w3c.github.io/uievents/#events-wheelevents 自己看看 当然不是全部 event 这里都有我就不都粘这里了  
## 什么是事件委托，e.currentTarget 与 e.target 有何区别  
> 事件委托指当有大量子元素触发事件时，将事件监听器绑定在父元素进行监听，此时数百个事件监听器变为了一个监听器，提升了网页性能。    
另外，React 把所有事件委托在 Root Element，用以提升性能。  

[event.currentTarget](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/currentTarget)

> Event 接口的只读属性 currentTarget 表示的，标识是当事件沿着 DOM 触发时事件的当前目标。它总是指向事件绑定的元素，而 Event.target 则是事件触发的元素。
## 浏览器中 cookie 有哪些字段
- Domain
- Path
- Expire/MaxAge
- HttpOnly
- Secure
- SameSite
## DOM 中 Element 与 Node 有何区别
[浅析 Node 与 Element 的区别](https://zhuanlan.zhihu.com/p/165422508)  
1. Node是节点，其中包含不同类型的节点，Element只是Node节点的一种。
2. Element继承与Node，可以调用Node的方法。
3. 给所有DOM元素添加方法，只需要污染Node或者Element的原型链就行。
## SameSite Cookie 有哪些值，是如何预防 CSRF 攻击的
见文档 [SameSite Cookie - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)见文章 [Cookie 的 SameSite 属性](http://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html)

- None: 任何情况下都会向第三方网站请求发送 Cookie
- Lax: 只有导航到第三方网站的 Get 链接会发送 Cookie，跨域的图片、iframe、form 表单都不会发送 Cookie
- Strict: 任何情况下都不会向第三方网站请求发送 Cookie
> 目前，主流浏览器 Same-Site 的默认值为 Lax，而在以前是 None，将会预防大部分 `CSRF` 攻击，如果需要手动指定 `Same-Site` 为 `None`，需要指定 `Cookie` 属性 `Secure`，即在 https 下发送
## sessionStorage 与 localStorage 有何区别
-  localStorage 生命周期是永久除非自主清除 sessionStorage 生命周期为当前窗口或标签页，关闭窗口或标签页则会清除数据  
- 他们均只能存储字符串类型的对象  

- 不同浏览器无法共享 localStorage 或 sessionStorage 中的信息。相同浏览器的不同页面间可以共享相同的 localStorage（页面属于相同域名和端口），但是不同页面或标签页间无法共享 sessionStorage 的信息。这里需要注意的是，页面及标 签页仅指顶级窗口，如果一个标签页包含多个 iframe 标签且他们属于同源页面，那么他们之间是可以共享 sessionStorage 的。 https://www.php.cn/faq/463215.html
## 如何统计当前页面出现的所有标签
- document.querySelectorAll('*')
- document.getElementsByTagName('*')
- \$$('*')，可在浏览器控制台使用
- document.all，已废弃，不建议使用
## 如何监听 localStorage 的变动
[StorageEvent](https://caniuse.com/?search=StorageEvent)  

## Data URL 的应用场景及如何生成
Data URLs 由四个部分组成：

1. 前缀(data:)
2. 指示数据类型的 MIME 类型
3. 如果二进制数据则为可选的 base64 标记，比如图片
4. 数据  
```data:[<mediatype>][;base64],<data>```  

应用场景:   
1. base 数据
2. 生成设备指纹

## 浏览器中如何读取二进制信息
使用 JavaScript 处理二进制数据和文件。

1. ArrayBuffer，二进制数组
2. TextDecoder 和 TextEncoder
3. Blob
4. File 和 FileReader

## 在浏览器中点击 a 标签保存为文件如何做
有两种方式:
1. `a.download`当指定 a 标签的 `download` 属性时，点击该链接会直接保存为文件，文件名为 `download` 属性
2. 通过对 a 标签指定的 URL 在服务器设置响应头 `Content-Disposition: attachment;` `filename="filename.jpg"` 可直接下载
## 如何禁止打开浏览器控制台
https://github.com/AEPKILL/devtools-detector
## 简述下 WebWorker，它如何进行通信
js 多线程通信，只能访问 navigator、setTimeout 等有限的 api

通过 onmessage 和 postmessage 通信，全局对象是 self

## 浏览器中监听事件函数 addEventListener 第三个参数有那些值
- capture。监听器会在时间捕获阶段传播到 event.target 时触发。
- passive。监听器不会调用 `preventDefault()`。
- once。监听器只会执行一次，执行后移除。
- singal。调用 `abort()`移除监听器。
## 浏览器中 Frame 与 Event Loop 的关系是什么
浏览器组成中有两大引擎，JS 引擎和渲染引擎。

Frame(帧)是渲染引擎每隔 16ms(默认 60fps)将渲染树渲染、合成成位图的结果

每次 `Event Loop` 是 JS 引擎执行的一个周期，执行过程中可能依赖渲染引擎的执行结果，比如访问 `DOM` 和 `CSSOM`，也可能影响渲染引擎绘制帧，比如调用 `requestAnimationFrame`，在每个帧开始绘制时执行一段回调函数(通常包含影响渲染结果的代码)

因此 `Frame` 和 `Event Loop` 是相对独立运行的，但是 `Event Loop` 中执行的代码可能依赖或影响 `Frame`
## 浏览器中如何使用原生的 ESM

**Native Import: Import from URL**  
通过 `script[type=module]`，可直接在浏览器中使用原生 ESM。这也使得前端不打包 (Bundless) 成为可能。
``` javascript
<script type="module">
  import lodash from "https://cdn.skypack.dev/lodash";
</script>
```
由于前端跑在浏览器中，因此它也只能从 `URL` 中引入 `Package`

1. 绝对路径: https://cdn.sykpack.dev/lodash
2. 相对路径: ./lib.js  

现在打开浏览器控制台，把以下代码粘贴在控制台中。由于 `http import` 的引入，你发现你调试 `lodash` 此列工具库更加方便了。
``` javascript
 lodash = await import('https://cdn.skypack.dev/lodash')

 lodash.get({ a: 3 }, 'a')
```

**ImportMap**  
但 `Http Import` 每次都需要输入完全的 URL，相对以前的裸导入 (`bare import specifiers`)，很不太方便，如下例:

`import lodash from "lodash";`  
它不同于 `Node.JS` 可以依赖系统文件系统，层层寻找 `node_modules`
```
/home/app/packages/project-a/node_modules/lodash/index.js
/home/app/packages/node_modules/lodash/index.js
/home/app/node_modules/lodash/index.js
/home/node_modules/lodash/index.js
```
在 ESM 中，可通过 `importmap` 使得裸导入可正常工作:
```  javascript
<script type="importmap">
  {
    "imports": {
      "lodash": "https://cdn.skypack.dev/lodash",
      "ms": "https://cdn.skypack.dev/ms"
    }
  }
</script>
```
此时可与以前同样的方式进行模块导入
``` javascript
import lodash from 'lodash'

import("lodash").then(_ => ...)
```

那么通过裸导入如何导入子路径呢？
``` javascript
<script type="importmap">
  {
    "imports": {
      "lodash": "https://cdn.skypack.dev/lodash",
      "lodash/": "https://cdn.skypack.dev/lodash/"
    }
  }
</script>
<script type="module">
  import get from "lodash/get.js";
</script>
```
**Import Assertion**  
通过 script[type=module]，不仅可引入 Javascript 资源，甚至可以引入 JSON/CSS，示例如下
``` javascript
<script type="module">
  import data from "./data.json" assert { type: "json" };

  console.log(data);
</script>
```
1. module 默认是 defer 的加载和执行方式

2. 这里会存在单独的 module 的域不会污染到全局

3. 直接是 strict
## 