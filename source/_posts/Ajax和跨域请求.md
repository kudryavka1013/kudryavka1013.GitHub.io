---
title: Ajax和跨域请求
date: 2021-04-28 15:50:17
tags: [Ajax]
categories: 前端
---
#### Ajax是什么？

Ajax全称异步JavaScript和XML，这是一种能够向服务器请求额外数据而不用刷新页面的技术。它的核心是 **XMLHttpRequest（XHR）** 对象，这个对象提供了一些接口，能让我们以**异步**的方式从服务器获取信息，这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。也就是说可以用XHR对象取得新数据，然后通过DOM将新数据更新到页面上。

#### XMLHttpRequest 对象

对于IE7以下的浏览器，这里就不讨论了（求求你们让IE退休吧，现在都什么年代了），对于正常点的浏览器，都支持原生的XHR对象。

##### XHR的用法

要创建一个XHR对象，只要一句：`var xhr = new XMLHttpRequest()`

当然，只是创建出来的话，它是不会做任何工作的。使用XHR的大体流程如下：

- <u>创建</u>一个XHR对象
- XHR对象<u>启动</u>一个请求
- XHR对象<u>发送</u>一个请求
- XHR对象<u>接收</u>响应数据
- 接收工作完成

---

首先，我们需要使用名为 open() 的方法，启动请求。这个时候只是在做发送前的准备工作，设定一系列参数，**请求并没有发送出去**。让我们看一个例子：

```javascript
xhr.open("get", "example.txt", false)
```

这里有三个参数：第一个参数表示**请求的类型**（get、post等）；第二个参数表示**url**；第三个参数表示**是否异步发送请求**

这里要注意的是，URL是<u>相对于执行代码</u>的当前页面，当然你也可以用绝对路径。而且只能向同一个域中使用相同端口和协议的URL发送请求，否则会引发安全错误，这里牵扯到同源和跨域的问题，下面会讲到。

---

然后，我们需要使用叫 send() 的方法来把我们设置好的请求发送出去：

```javascript
xhr.send(null)
```

这个方法接收一个参数：**要作为请求主体发送的数据**，这个参数仅用于POST请求。如果不需要通过请求主体发送数据，这里就写一个null，当然你也可以不写，不过为了浏览器兼容我不建议你这么做。

---

到这里为止，请求已经发出去了，那我们还得等服务器给我们响应。在收到响应后，响应的数据会放在XHR对象里，我们通过检查这些属性来拿到我们想要的东西：

- responseText：作为响应主体返回的文本
- responseXML：如果响应的内容类型是`"text/xml"`或者是`"application/xml"`，这里就会保存拿到的XML文档
- status：响应的HTTP状态码
- statusText：HTTP状态说明

一般来说，我们首先要检查status，根据HTTP状态决定后续操作。对于同步请求，JS会等到服务器响应之后再继续执行。但是多数时候我们都是异步的，JS会继续执行而不会等待响应。此时，我们需要检测XHR的 **readyState** 属性，它表示XHR的当前活动阶段，有以下取值：

- 0：未初始化，尚未调用 open()
- 1：已启动，已调用 open()，未调用 send()
- 2：已发送，已调用 send()，还没收到响应
- 3：接收中，已经接收到部分数据（接收了，但没完全接收）
- 4：已完成，已经接收到全部数据，并且已经可以使用了

这些取值和XHR的流程是一一对应的。每当 readyState 的值发生了改变，就会触发一次 **readystatechange 事件**，我们利用这个事件来检测变化后 readyState 的值，不过通常都只对 4 作处理。

有一点注意，为了保证兼容性，事件处理程序的指定放在调用open()方法之前，我们看一下完整的流程

```javascript
var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function() {
  if(xhr.readyState == 4){
    if((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
      // 成功
      alert(xhr.responseText);
    }else{
      // 失败
      alert("Request error: " + xhr.status);
    }
  }
};
xhr.open("get", "example.txt", true);
xhr.send(null);
```

我们给 readystatechange 添加事件处理，先检查 readyState == 4，确定我们的响应数据已经全部拿到了；接着判断 status 并做相应的处理，200表示正常成功（有的浏览器会报204，奇怪的问题），304表示请求的资源没有被修改，可以直接用浏览器缓存的版本。

此外，在接收响应前可以用 <u>abort()</u> 方法来取消异步请求。调用之后，XHR对象会停止触发事件，而且也不允许访问任何与响应有关的对象属性。终止请求之后，还应该对XHR对象解除引用。

##### HTTP头部信息

每个HTTP请求和响应都会带有相应的头部信息，XHR对象也提供了方法来操作这些头部信息

默认情况下，在发送XHR请求时，也会发送一些头部消息：

- Accept：浏览器能够处理的内容类型
- Accept-Charset：浏览器支持的字符集
- Accept-Encoding：浏览器能够处理的压缩编码
- Accept-Language：浏览器当前语言
- Connection：浏览器和服务器之间连接的类型
- Cookie：当前页面设置的任何Cookie
- Host：发出请求的页面所在的域
- Referer：发出请求的页面的URI（笑死，HTTP规范拼写错了，正确写法应该是referrer）
- User-Agent：浏览器的用户代理字符串

实际上如果需求不相关，我们没必要去修改这些默认的东西，而且也很容易出问题（有的浏览器也不给改），用的比较多的字段其实是`Content-Type`

要检查响应的头部信息，可以使用 `getAllResponseHeaders()` 方法

要设置自定义头部信息，可以使用 `setRequestHeader("字段名", "字段值")` 方法。要注意的是，**这个方法必须在open()之后、send()之前调用**

```javascript
xhr.open("get", "example.txt", true);
xhr.setRequestHeader("MyHeader","MyValue");
xhr.send(null);
```

##### GET请求

最常见的GET请求，基本用来向服务器查询信息，查询的参数以一定格式放到URL的末尾。**最最重要的一件事，使用 encodeURIComponent() 对查询字符串编码**

查询字符串用 ? 和前面隔开，参数用键值对形式，多个参数用 & 连接，类似这样：
`example.php?name1=value1&name2=value2`

我们添加参数可以写一个方法，向某个url末尾加参数

```javascript
function addURLParam(url, name, value){
	url += (url.indexOf("?") == -1 ? "?" : "&");
	url += encodeURIComponent(name) + "=" + encodeURIComponent(value);
    return url;
}
```

##### POST请求

第二常见的POST请求，通常用于向服务器发送应该被保存的数据。和GET不同，POST请求把数据作为请求的主体提交，即数据不会放在URL里。

通常情况下，服务器对<u>POST请求和Web表单请求并不会一视同仁</u>，服务器自己要做处理。不过我们可以**用XHR模拟表单请求**。用上文提到的头部消息设置，并传入适当格式的数据，就能模仿一个表单请求。

1. 设置头部信息`"Content-Type"`为`"application/x-www-form-urlencoded"`
2. 将表单序列化（意思就是将某个表单的数据转换成查询字符串那种格式，并且编码，详情看[这里](https://blog.kudryavka.site/2021/04/27/表单和表单脚本/#表单序列化)）
3. 发送序列化后的表单数据

```javascript
xhr.open("post", "exmaple.php", true)
// 设置头部消息
xhr.setRequestHeader("Content-Type","application/x-www-form-urlencoded")
// 拿到一个表单
var form = document.getElementById("aform")
// 序列化后发送
xhr.send(serialize(form))         
```

#### XMLHTTPRequest 2级

XHR2级规范，进一步发展了XHR，但大部分浏览器都只实现了部分内容，其中最重要的也最常用的就是**FormData对象**。它为序列化表单以及创建与表单格式相同的数据（用于通过XHR传输）提供了方便。

通过 append() 方法，我们可以很轻松的给一个FormData对象传入键值对；或者如果你已经有一个表单了，你可以简单粗暴的把表单丢进FormData的构造函数里，它会识别数据并转换成键值对，还会给你编好码

```javascript
// 插入键值对
var data = new FormData();
data.append("name", "value");

// 直接给它一个表单
var data = new FormData(document.forms[0]);

// 前面配置XHR对象就省略
// ...

// 直接把这个FormData实例传给send发出去
xhr.open("post", "example.php", true)
xhr.send(data)
```

不知道你们注意到没有，这里没有设置头部消息。原因是<u>XHR对象能够识别传入的数据类型是FormData实例，并配置适当的头部信息</u>。

---

除了FormData之外，还有一个timeout属性，用来配置超时的时间。相应的还有timeout事件，可以通过ontimeout来指定事件处理。不过要注意和onreadystatechange不要发生冲突，因为前面说了，请求终止会导致一些属性无法访问，可以在try-catch里检查status。

#### 进度事件

上文提到，readyState 有接收中（3）和已完成（4），但是仅凭这个属性，我们是没法监控响应数据接收的整个过程的，因此后来XHR又专门给这个接收过程设计了一些进度事件：

- loadstart：接收到响应数据的第一个字节时触发
- progress：表示正在进行，会在接收期间<u>不断触发</u>
- error：请求发生错误时触发
- abort：调用abort()方法时触发
- load：收到完整的响应数据时触发
- loadend：通信完成或触发error、abort、load后触发（这个最好别用，因为很多浏览器不支持，而且也没什么用）

##### load事件

这其实就是用来代替readystatechange事件的。onload事件处理程序会收到event，event.target就指向了XHR对象实例。但是并非所有浏览器都实现了event.target，为了兼容性，我们没必要去使用event，依旧像之前一样访问status

##### progress事件

这个事件会在接收数据过程中周期性触发。对应的onprogress事件处理程序会收到event，event.target是XHR对象实例，里面还附带了额外属性：

- lengthComputable：进度信息是否可用的<u>布尔值</u>
- position：表示已经接收的字节数（你可以理解成<u>一条</u>数据传到什么位置了，这样想position这个词就好理解了）
- totalSize：预期字节数（其实就是总长），它是根据Content-Length响应头信息确定的

有了这些信息，我们可以做一个进度指示器出来：`event.position + "/" + event.totalSize + "bytes"`。另外，为了保证正常执行，<u>onprogress事件处理程序必须放在open()方法之前</u>



#### 跨域源资源共享

##### 什么是跨域/同域？

默认情况下，XHR受到跨域安全策略的限制，只能访问于包含它的页面位于同一个域中的资源，这就是**跨域**问题。同域，简单的说就是**域名相同，端口相同，协议相同**。

对此，W3C的大佬们弄出了**CORS（跨域源资源共享，Cross-origin resource sharing）**这么一个东西，解决了跨域的问题。它定义了在进行跨域访问时，浏览器和服务器应该怎么沟通。其中核心就是**使用自定义的HTTP头部**，根据这个头部来决定请求或响应是应该成功还是失败。

>引用阮一峰博客的一段话：
>
>整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的Ajax通信没有差别，代码完全一样。浏览器一旦发现Ajax请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。
>
>因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。

##### 简单请求

所谓简单请求，就是：

- 请求方法只能是：HEAD，GET，POST
- HTTP头部信息只能有这些字段：Accept，Accept-Language，Content-Language，Last-Event-ID，Content-Type（只限于`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`）
  这意味着 setRequestHeader() 和 getAllResponseHeaders() 失效，同时也不能发送和接收cookie

对于简单请求的CORS请求，我们写前端代码不用做额外的工作，浏览器的操作也很简单。它会给请求附带一个**额外的 Origin 头部字段**，其中包含请求页面的源信息（**协议，域名和端口**，就是上面说的判断是否同域的几个参数），比如：`Origin: http://www.nczonline.net`

服务器根据这个字段来判断是否给予响应：

- 如果认为这个请求可以接受，则在消息响应头会有这么一条：`Access-Control-Allow-Origin: 和请求头相同的源信息`（如果是公共资源，可以回发 `"*"`，表示接受任意域名的请求），例如：`Access-Control-Allow-Origin: http://www.nczonline.net`
- 如果认为这个请求不能接受，服务器会返回一个正常的HTTP回应，但不会有这么一个头部字段。浏览器发现，这个回应的头信息没有包含这个字段，就知道出错了，从而抛出一个错误。

接受请求的响应其实还可能会有一些可选值：

> **Access-Control-Allow-Credentials**
>
> 它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为`true`，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为`true`，如果服务器不要浏览器发送Cookie，删除该字段即可。
>
> **Access-Control-Expose-Headers**
>
> CORS请求时，XMLHttpRequest 对象的 getResponseHeader() 方法只能拿到6个基本字段：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`。如果想拿到其他字段，就必须在`Access-Control-Expose-Headers`里面指定。

###### 带凭据的请求

默认情况下，cookie、HTTP认证以及客户端SSL证明都不会被发送，如果我们想要发送这些东西，就要在XHR对象里设置：`xhr.withCredentials: true`；另一方面，服务器也得同意（即上面提到的 Access-Control-Allow-Credentials: true）

##### 非简单请求

那不是简单请求就是非简单请求咯，非简单请求是那种对服务器有特殊要求的请求，比如请求方法是 PUT 或 DELETE，或者 `Content-Type` 字段的类型是 `application/json` 。

对于这些非简单请求的CORS请求，会在正式通信之前，来一次 **“预检”** HTTP请求（preflight）。浏览器先通过预检请求询问服务器，请求头带上一些给服务器判断用的信息，只有得到肯定答复，浏览器才会发出正式的XHR请求，否则就报错。

我们看一下这个预检请求的头部有些什么东西：

- 首先这是一个 OPTIONS 类型的请求，表示询问
- Origin：最重要的字段，表明发送请求的页面的域名
- Access-Control-Request-Method：表明自己等一下的正式请求用的是什么方法，多个用逗号
- Access-Control-Request-Headers：自定义的头部信息，这个是可选的

---

服务器收到请求后，如果同意，响应的消息头部就会有如下的几个字段：

- Access-Control-Allow-Origin：同样是最重要的，会返回和请求的 Origin 相同的域名（或者 *）
- Access-Control-Allow-Methods：支持的所有方法，多个用逗号
- Access-Control-Allow-Headers：支持的自定义头部信息，多个用逗号（如果请求里有，则响应里必须写）
- Access-Control-Allow-Credentials：和上文提到的相同，控制凭据类的数据是否传输
- Access-Control-Max-Age：应该将这个 “预检” 请求缓存多长时间（单位秒），在这期间，不用再重新发预检请求

---

此后的请求和响应，就和简单请求的过程一样，都会带 Origin 和 Access-Control-Allow-Origin

##### 总结

简单请求：

```javascript
// 请求头
Origin

// 响应头
Acess-Control-Allow-Origin（和请求头的相同）
```

复杂请求：先预检再正式

```javascript
// 预检请求
// 请求头
Origin
Access-Control-Request-Method
Access-Control-Request-Headers（可选）
// 响应头
Access-Control-Allow-Origin（和请求头的相同）
Access-Control-Allow-Methods
Access-Control-Allow-Headers（看请求头有没有）
Access-Control-Max-Age

// 正式请求 = 简单请求
// 请求头
Origin
// 响应头
Acess-Control-Allow-Origin（和请求头的相同）
```



#### 参考

[跨域资源共享 CORS 详解 - 阮一峰的网络日志 (ruanyifeng.com)](http://www.ruanyifeng.com/blog/2016/04/cors.html)

《JavaScript高级程序设计（第3版）》21章