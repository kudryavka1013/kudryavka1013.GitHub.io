#### 缓存是什么东西？

首先，缓冲就是一个**资源副本**。我们向服务器请求资源后，会根据情况将资源复制一份副本<u>放在本地</u>，以便下次读取。

和本地存储localStorage、cookie不同的是，本地存储一般是**数据记录**，存储量比较小。而缓存主要用途是为了<u>减少资源请求</u>，多用于<u>存储文件</u>，存储量比较大。



#### 缓存有什么用呢？

最根本的作用就是**减少没有必要的请求**。有的资源，<u>很久才会发生一次改变</u>，如果我们每次都去请求它，请求拿到的东西又是一样的。一来一回，页面显示要的时间长了不说，频繁的服务器请求还给服务器增加压力。那我们把这个资源直接缓存到本地，那每次要用了就本地读取，不用再发起请求。

这样，**缓存减少页面显示需要的时间，提高了用户体验。同时，减少了服务器请求，减轻服务器压力。**



##### 浏览器缓存机制

按浏览器读取优先级顺序依次为：Memory Cache、Service Worker Cache、**HTTP Cache**、Push Cache



### HTTP Cache

浏览器与服务器通信的方式为应答模式，即是：浏览器发起HTTP请求 – 服务器响应该请求。那么浏览器<u>第一次</u>向服务器发起该请求后拿到请求结果，**会根据响应报文中HTTP头的<u>缓存标识</u>，决定是否缓存结果**，<u>是</u>则将请求结果和缓存标识存入浏览器缓存中

- 浏览器每次发起请求，都先在缓存中查找该请求的结果以及缓存标识
- 浏览器每次拿到返回的请求结果，都会把结果和缓存标识存进浏览器缓存



![img](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411000747.webp)

缓存标识分为**强缓存**，**协商缓存**，优先级最高的是强缓存，命中强缓存失败时，才会去走协商缓存

- 强缓存：在**本地**读取比对，而<u>不去请求服务器</u>，返回状态码为<u>200</u>
- 协商缓存：把本地缓存发送到**服务器**进行比对，若<u>没有改变</u>才直接读取本地缓存，返回状态码为<u>304</u>



#### 强缓存（强制缓存）

只在本地进行的操作，<u>向浏览器本地缓存查找请求结果</u>，根据结果的<u>缓存规则</u>来决定是否使用这个缓存结果。

##### 分为三种情况

- 不存在该缓存结果和缓存标识（命中失败），<u>直接向服务器发起请求</u>。
  ![img](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411150850.png)
- 存在该缓存结果和缓存标识（命中），但是<u>结果失效</u>了，<u>转而使用协商缓存</u>
  ![img](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411150900.png)
- 存在该缓存结果和缓存标识（命中），且该<u>结果尚未失效</u>，则<u>直接返回该结果</u>
  ![img](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411150916.png)

那么缓存的规则是什么？浏览器向服务器发起请求时，服务器会将缓存规则放入HTTP响应报文的HTTP头中和请求结果一起返回给浏览器。控制强制缓存的字段分别是Expires和Cache-Control，其中Cache-Control优先级比Expires高，此外还有一个Pragma，比它们的优先级都高

##### Expires

Expires是HTTP/1.0版本控制缓存的字段。

它的值是**<u>服务器</u>返回该请求结果的<u>到期时间</u>**。如果再次发起该请求时，<u>客户端的时间</u>小于这个值（没到时间），说明没过期，直接使用缓存结果

![image-20210411152910058](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411152910.png)

但是会有问题，由于到期时间是根据服务器决定的，而比较时用客户端时间比较，当服务器和客户端时间不一致的时候，缓存有效期就会不准

##### Cache-Control

Cache-Control 是 HTTP/1.1 中新增的属性，解决了Expires的问题，它不依赖两个特定时间，而是使用时间长度来记录有效期

它有几个值

- max-age= num，s-maxage= num
  一个数字，表示该资源过了**多少秒**之后无效，从第一次拿到这个资源开始计算。浏览器中，它们两个都起作用，s-maxage优先级更高。代理服务器里只有s-maxage起作用。可以设置maxage为0表示立即过期来向服务器请求资源
- public，private
  public 表示该资源**可以被所有客户端和代理服务器缓存**，private 表示该资源**只能被客户端缓存**。
  默认值是private，当设置了s-maxage 的时候表示允许代理服务器缓存，相当于public
  private可以指定
- no-cache，no-store
  no-cache 表示**不询问浏览器本地缓存情况**，而去**向服务器验证**当前资源有没有更新（协商缓存）。no-store 表示**完全不使用缓存策略**，不缓存 请求或响应的任何内容，直接向服务器请求最新资源。
  只要设置了这两个值其一，就是跳过本地缓存的检查直接向服务器请求，就直接忽略max-age等属性了。
- must-revalidate
  在缓存过期前可以使用，过期后必须向服务器验证

![image-20210411162054798](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411162054.png)

##### Pragma

它的值只有no-cache，意思和Cache-Control是一样的。

优先级高于 `cache-control` 和 `expires`，即三者同时出现时，先看 `pragma` -> `cache-control` -> `expires`

![image-20210411161926027](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411161926.png)



#### 协商缓存

协商缓存就是强制缓存失效（包括设置不走强缓存）后，同时设置了相关属性，浏览器携带缓存标识向服务器发起请求，由**服务器根据缓存标识决定是否使用缓存**的过程

##### 分为两种情况

- 协商缓存生效，返回304（成功），就是表示本地的资源还能继续用，不用从服务器拿资源
  ![img](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411163250.webp)

- 协商缓存失效，返回200和请求结果（就是返回相关资源）
  ![img](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411163341.webp)

同样，协商缓存的标识也是在响应报文的HTTP头中和请求结果一起返回给浏览器的。控制协商缓存的字段分别有：Last-Modified / If-Modified-Since和Etag / If-None-Match，**其中Etag / If-None-Match的优先级比Last-Modified / If-Modified-Since高**。

##### Last-Modified / If-Modified-Since

last-modified 记录该**资源文件在服务器最后被修改的时间**。启用后，请求资源之后的响应头会增加一个 `last-modified` 字段

![image-20210411165212908](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411165212.png)

当再次请求该资源时，请求头中会带有 `if-modified-since` 字段，值是之前返回的 `last-modified` 的值

![image-20210411165830530](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411165830.png)

通过此字段值告诉服务器该资源上次请求返回的最后被修改时间。服务器收到该请求，发现请求头含有If-Modified-Since字段，则会<u>根据If-Modified-Since的字段值与该资源在服务器的最后被修改时间做对比</u>，若服务器的资源最后被修改时间大于If-Modified-Since的字段值，则重新返回资源，状态码为200；否则则返回304，代表资源无更新，可继续使用缓存文件

但是有两个缺点：

- 只要编辑了，<u>不管内容是否真的有改变，都会以这最后修改的时间作为判断依据</u>，当成新资源返回，从而导致了没必要的请求响应，而这正是缓存本来的作用即避免没必要的请求。
- 时间的精确度只能到秒，如果在一秒内的修改是检测不到更新的，仍会告知浏览器使用旧的缓存

##### ETag / If-None-Match

ETag就解决了上面的问题，它**基于资源的内容编码生成一串唯一的表示字符串**。这其实是一串hash码，当服务器的资源文件变化的时候，它的hash码也会随之改变。

![image-20210411174017425](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411194250.png)

服务器在收到某个资源请求的时候，根据该请求，**服务器上根据对应资源生成hash码（每次请求都会生成一次，会增加服务器的开销），将请求中的ETag和生成的hash码进行对比**，若一致则代表未改变可直接使用本地缓存并返回 304；若不一致则返回新的资源（状态码200）并修改返回的 ETag 字段为新的值

![image-20210411194242891](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411194242.png)

ETag 又有强弱校验之分，如果 hash 码是以 "W/" 开头的一串字符串，说明此时协商缓存的校验是**弱校验**的，<u>只有服务器上的文件差异（根据 ETag 计算方式来决定）达到能够触发 hash 值后缀变化的时候，才会真正地请求资源</u>，否则返回 304 并加载浏览器缓存



#### 总结

![img](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411194112.jpeg)

强制缓存优先于协商缓存进行，若强制缓存（Expires和Cache-Control）生效则直接使用缓存，若不生效则进行协商缓存（Last-Modified / If-Modified-Since和Etag / If-None-Match），协商缓存由服务器决定是否使用缓存，若协商缓存失效，那么代表该请求的缓存失效，重新获取请求结果，再存入浏览器缓存中；生效则返回304，继续使用缓存。



#### 访问/刷新时，会进行怎样的缓存访问？

- 标签页进入，输入URL回车进入
  根据实际设计的缓存策略去判断，即走正常情况下的缓存策略，根据第一次访问时拿到的响应信息里的字段去判断怎么操作，该怎么来就怎么来
- 刷新按钮，F5刷新，网页右键重新加载
  浏览器在这种操作下将 `cache-control` 的 `max-age` 直接设置成了 0，让缓存立即过期，直接走协商缓存路线
  ![image-20210411195514026](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411195514.png)
- Ctrl + F5 强制刷新
  浏览器会强行设置 `no-cache`，强制获取最新的资源，就连 `if-modified-since` 等其他缓存协议字段都会被忽略
  ![image-20210411195526562](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210411195526.png)



### 参考

https://juejin.cn/post/6844903593275817998

https://blog.csdn.net/qianyu6200430/article/details/115535067

https://www.jianshu.com/p/fb59c770160c