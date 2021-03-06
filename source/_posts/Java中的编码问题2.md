---
title: Java 的中文编码问题(2)
date: 2017-02-09 14:18:20
tags: "Java"
categories: "Java"
---

引自[《深入分析Java中的中文编码问题》](https://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/)
***

## Java Web 涉及到的编码
对于使用中文来说，有 I/O 的地方就会涉及到编码，前面已经提到了 I/O 操作会引起编码，而大部分 I/O 引起的乱码都是网络 I/O，因为现在几乎所有的应用程序都涉及到网络操作，而数据经过网络传输都是以字节为单位的，所以所有的数据都必须能够被序列化为字节。在 Java 中数据被序列化必须继承 Serializable 接口。
原文中，作者提出这这样一个问题，我们是否认真考虑过一段文本它的实际大小应该怎么计算，他曾经碰到过一个问题：就是要想办法压缩 Cookie 大小，减少网络传输量，当时有选择不同的压缩算法，发现压缩后字符数是减少了，但是并没有减少字节数。所谓的压缩只是将多个单字节字符通过编码转变成一个多字节字符。减少的是 String.length()，而并没有减少最终的字节数。例如将“ab”两个字符通过某种编码转变成一个奇怪的字符，虽然字符数从两个变成一个，但是如果采用 UTF-8 编码这个奇怪的字符最后经过编码可能又会变成三个或更多的字节。同样的道理比如整型数字 1234567 如果当成字符来存储，采用 UTF-8 来编码占用 7 个 byte，采用 UTF-16 编码将会占用 14 个 byte，但是把它当成 int 型数字来存储只需要 4 个 byte 来存储。所以看一段文本的大小，看字符本身的长度是没有意义的，即使是一样的字符采用不同的编码最终存储的大小也会不同，所以从字符到字节一定要看编码类型。
另外一个问题，我们是否考虑过，当我们在电脑中某个文本编辑器里输入某个汉字时，它到底是怎么表示的？我们知道，计算机里所有的信息都是以 01 表示的，那么一个汉字，它到底是多少个 0 和 1 呢？我们能够看到的汉字都是以字符形式出现的，例如在 Java 中“淘宝”两个字符，它在计算机中的数值 10 进制是 28120 和 23453，16 进制是 6bd8 和 5d9d，也就是这两个字符是由这两个数字唯一表示的。Java 中一个 char 是 16 个 bit 相当于两个字节，所以两个汉字用 char 表示在内存中占用相当于四个字节的空间。
这两个问题搞清楚后，再来看一下 Java Web 中那些地方可能会存在编码转换？
用户从浏览器端发起一个 HTTP 请求，需要存在编码的地方是 URL、Cookie、Parameter。服务器端接受到 HTTP 请求后要解析 HTTP 协议，其中 URI、Cookie 和 POST 表单参数需要解码，服务器端可能还需要读取数据库中的数据，本地或网络中其它地方的文本文件，这些数据都可能存在编码问题，当 Servlet 处理完所有请求的数据后，需要将这些数据再编码通过 Socket 发送到用户请求的浏览器里，再经过浏览器解码成为文本。这些过程如下图所示：
!["HTTP请求"](/images/java-1-0.png)
如上图所示一次 HTTP 请求设计到很多地方需要编解码，它们编解码的规则是什么？下面将会重点阐述一下：
***
### URL的编解码
用户提交一个 URL，这个 URL 中可能存在中文，因此需要编码，如何对这个 URL 进行编码？根据什么规则来编码？有如何来解码？如下图一个 URL：
!["URL"](/images/java-1-1.png)
上图中以 Tomcat 作为 Servlet Engine 为例，它们分别对应到下面这些配置文件中：
Port 对应在 Tomcat 的 <Connector port="8080"/> 中配置，而 Context Path 在 <Context path="/examples"/> 中配置，Servlet Path 在 Web 应用的 web.xml 中的
```
 <servlet-mapping> 
        <servlet-name>junshanExample</servlet-name> 
        <url-pattern>/servlets/servlet/*</url-pattern> 
 </servlet-mapping>

```
<url-pattern> 中配置，PathInfo 是我们请求的具体的 Servlet，QueryString 是要传递的参数，注意这里是在浏览器里直接输入 URL 所以是通过 Get 方法请求的，如果是 POST 方法请求的话，QueryString 将通过表单方式提交到服务器端，这个将在后面再介绍。
上图中 PathInfo 和 QueryString 出现了中文，当我们在浏览器中直接输入这个 URL 时，在浏览器端和服务端会如何编码和解析这个 URL 呢？为了验证浏览器是怎么编码 URL 的我们选择 FireFox 浏览器并通过 HTTPFox 插件观察我们请求的 URL 的实际的内容，以下是 URL：
> HTTP://localhost:8080/examples/servlets/servlet/ 君山 ?author= 君山

在中文 FireFox3.6.12 的测试结果
!["Test"](/images/java-1-2.png)
君山的编码结果分别是：e5 90 9b e5 b1 b1，be fd c9 bd，查阅上一届的编码可知，PathInfo 是 UTF-8 编码而 QueryString 是经过 GBK 编码，至于为什么会有“%”？查阅 URL 的编码规范 RFC3986 可知浏览器编码 URL 是将非 ASCII 字符按照某种编码格式编码成 16 进制数字然后将每个 16 进制表示的字节前加上“%”，所以最终的 URL 就成了上图的格式了。
默认情况下中文 IE 最终的编码结果也是一样的，不过 IE 浏览器可以修改 URL 的编码格式在选项 -> 高级 -> 国际里面的发送 UTF-8 URL 选项可以取消。
从上面测试结果可知浏览器对 PathInfo 和 QueryString 的编码是不一样的，不同浏览器对 PathInfo 也可能不一样，这就对服务器的解码造成很大的困难，下面我们以 Tomcat 为例看一下，Tomcat 接受到这个 URL 是如何解码的。
解析请求的 URL 是在 org.apache.coyote.HTTP11.InternalInputBuffer 的 parseRequestLine 方法中，这个方法把传过来的 URL 的 byte[] 设置到 org.apache.coyote.Request 的相应的属性中。这里的 URL 仍然是 byte 格式，转成 char 是在 org.apache.catalina.connector.CoyoteAdapter 的 convertURI 方法中完成的：
```
protected void convertURI(MessageBytes uri, Request request) 
 throws Exception { 
        ByteChunk bc = uri.getByteChunk(); 
        int length = bc.getLength(); 
        CharChunk cc = uri.getCharChunk(); 
        cc.allocate(length, -1); 
        String enc = connector.getURIEncoding(); 
        if (enc != null) { 
            B2CConverter conv = request.getURIConverter(); 
            try { 
                if (conv == null) { 
                    conv = new B2CConverter(enc); 
                    request.setURIConverter(conv); 
                } 
            } catch (IOException e) {...} 
            if (conv != null) { 
                try { 
                    conv.convert(bc, cc, cc.getBuffer().length - 
 cc.getEnd()); 
                    uri.setChars(cc.getBuffer(), cc.getStart(), 
 cc.getLength()); 
                    return; 
                } catch (IOException e) {...} 
            } 
        } 
        // Default encoding: fast conversion 
        byte[] bbuf = bc.getBuffer(); 
        char[] cbuf = cc.getBuffer(); 
        int start = bc.getStart(); 
        for (int i = 0; i < length; i++) { 
            cbuf[i] = (char) (bbuf[i + start] & 0xff); 
        } 
        uri.setChars(cbuf, 0, length); 
 }
```
从上面的代码中可以知道对 URL 的 URI 部分进行解码的字符集是在 connector 的 <Connector URIEncoding="UTF-8"/> 中定义的，如果没有定义，那么将以默认编码 ISO-8859-1 解析。所以如果有中文 URL 时最好把 URIEncoding 设置成 UTF-8 编码。
QueryString 又如何解析？ GET 方式 HTTP 请求的 QueryString 与 POST 方式 HTTP 请求的表单参数都是作为 Parameters 保存，都是通过 request.getParameter 获取参数值。对它们的解码是在 request.getParameter 方法第一次被调用时进行的。request.getParameter 方法被调用时将会调用 org.apache.catalina.connector.Request 的 parseParameters 方法。这个方法将会对 GET 和 POST 方式传递的参数进行解码，但是它们的解码字符集有可能不一样。POST 表单的解码将在后面介绍，QueryString 的解码字符集是在哪定义的呢？它本身是通过 HTTP 的 Header 传到服务端的，并且也在 URL 中，是否和 URI 的解码字符集一样呢？从前面浏览器对 PathInfo 和 QueryString 的编码采取不同的编码格式不同可以猜测到解码字符集肯定也不会是一致的。的确是这样 QueryString 的解码字符集要么是 Header 中 ContentType 中定义的 Charset 要么就是默认的 ISO-8859-1，要使用 ContentType 中定义的编码就要设置 connector 的 <Connector URIEncoding=”UTF-8” useBodyEncodingForURI=”true”/> 中的 useBodyEncodingForURI 设置为 true。这个配置项的名字有点让人产生混淆，它并不是对整个 URI 都采用 BodyEncoding 进行解码而仅仅是对 QueryString 使用 BodyEncoding 解码，这一点还要特别注意。
从上面的 URL 编码和解码过程来看，比较复杂，而且编码和解码并不是我们在应用程序中能完全控制的，所以在我们的应用程序中应该尽量避免在 URL 中使用非 ASCII 字符，不然很可能会碰到乱码问题，当然在我们的服务器端最好设置 <Connector/> 中的 URIEncoding 和 useBodyEncodingForURI 两个参数。
***
### HTTP Header 的编解码
当客户端发起一个 HTTP 请求除了上面的 URL 外还可能会在 Header 中传递其它参数如 Cookie、redirectPath 等，这些用户设置的值很可能也会存在编码问题，Tomcat 对它们又是怎么解码的呢？
对 Header 中的项进行解码也是在调用 request.getHeader 是进行的，如果请求的 Header 项没有解码则调用 MessageBytes 的 toString 方法，这个方法将从 byte 到 char 的转化使用的默认编码也是 ISO-8859-1，而我们也不能设置 Header 的其它解码格式，所以如果你设置 Header 中有非 ASCII 字符解码肯定会有乱码。
我们在添加 Header 时也是同样的道理，不要在 Header 中传递非 ASCII 字符，如果一定要传递的话，我们可以先将这些字符用 org.apache.catalina.util.URLEncoder 编码然后再添加到 Header 中，这样在浏览器到服务器的传递过程中就不会丢失信息了，如果我们要访问这些项时再按照相应的字符集解码就好了。
***
### POST 表单的编解码
在前面提到了 POST 表单提交的参数的解码是在第一次调用 request.getParameter 发生的，POST 表单参数传递方式与 QueryString 不同，它是通过 HTTP 的 BODY 传递到服务端的。当我们在页面上点击 submit 按钮时浏览器首先将根据 ContentType 的 Charset 编码格式对表单填的参数进行编码然后提交到服务器端，在服务器端同样也是用 ContentType 中字符集进行解码。所以通过 POST 表单提交的参数一般不会出现问题，而且这个字符集编码是我们自己设置的，可以通过 request.setCharacterEncoding(charset) 来设置。
另外针对 multipart/form-data 类型的参数，也就是上传的文件编码同样也是使用 ContentType 定义的字符集编码，值得注意的地方是上传文件是用字节流的方式传输到服务器的本地临时目录，这个过程并没有涉及到字符编码，而真正编码是在将文件内容添加到 parameters 中，如果用这个编码不能编码时将会用默认编码 ISO-8859-1 来编码。
***
### HTTP BODY 的编解码
当用户请求的资源已经成功获取后，这些内容将通过 Response 返回给客户端浏览器，这个过程先要经过编码再到浏览器进行解码。这个过程的编解码字符集可以通过 response.setCharacterEncoding 来设置，它将会覆盖 request.getCharacterEncoding 的值，并且通过 Header 的 Content-Type 返回客户端，浏览器接受到返回的 socket 流时将通过 Content-Type 的 charset 来解码，如果返回的 HTTP Header 中 Content-Type 没有设置 charset，那么浏览器将根据 Html 的 <meta HTTP-equiv="Content-Type" content="text/html; charset=GBK" /> 中的 charset 来解码。如果也没有定义的话，那么浏览器将使用默认的编码来解码。
***
### 其它需要编码的地方
除了 URL 和参数编码问题外，在服务端还有很多地方可能存在编码，如可能需要读取 xml、velocity 模版引擎、JSP 或者从数据库读取数据等。
xml 文件可以通过设置头来制定编码格式
```
<?xml version="1.0" encoding="UTF-8"?>
```
Velocity 模版设置编码格式：
``` 
services.VelocityService.input.encoding=UTF-8
```
JSP 设置编码格式：
```
<%@page contentType="text/html; charset=UTF-8"%>
```
访问数据库都是通过客户端 JDBC 驱动来完成，用 JDBC 来存取数据要和数据的内置编码保持一致，可以通过设置 JDBC URL 来制定如 MySQL：
```
url="jdbc:mysql://localhost:3306/DB?useUnicode=true&characterEncoding=GBK"
```
***
### 常见问题分析
在了解了 Java Web 中可能需要编码的地方后，下面看一下，当我们碰到一些乱码时，应该怎么处理这些问题？出现乱码问题唯一的原因都是在 char 到 byte 或 byte 到 char 转换中编码和解码的字符集不一致导致的，由于往往一次操作涉及到多次编解码，所以出现乱码时很难查找到底是哪个环节出现了问题，下面就几种常见的现象进行分析。
+ **中文变成了看不懂的字符**
　　例如，字符串“淘！我喜欢！”变成了“Ì Ô £ ¡Î Ò Ï²»¶ £ ¡”编码过程如下图所示:
!["1"](/images/java-1-3.png)
　　字符串在解码时所用的字符集与编码字符集不一致导致汉字变成了看不懂的乱码，而且是一个汉字字符变成两个乱码字符。
+ **一个汉字变成一个问号**
　　例如，字符串“淘！我喜欢！”变成了“？？？？？？”编码过程如下图所示
!["2"](/images/java-1-4.png)
　　将中文和中文符号经过不支持中文的 ISO-8859-1 编码后，所有字符变成了“？”，这是因为用 ISO-8859-1 进行编解码时遇到不在码值范围内的字符时统一用 3f 表示，这也就是通常所说的“黑洞”，所有 ISO-8859-1 不认识的字符都变成了“？”。
+ **一个汉字变成两个问号**
　　例如，字符串“淘！我喜欢！”变成了“？？？？？？？？？？？？”编码过程如下图所示
!["3"](/images/java-1-5.png)
　　这种情况比较复杂，中文经过多次编码，但是其中有一次编码或者解码不对仍然会出现中文字符变成“？”现象，出现这种情况要仔细查看中间的编码环节，找出出现编码错误的地方。
+ **一种不正常的正确编码**
　　还有一种情况是在我们通过 request.getParameter 获取参数值时，当我们直接调用```String value = request.getParameter(name);```
　　会出现乱码，但是如果用下面的方式``` String value = String(request.getParameter(name).getBytes("ISO-8859-1"), "GBK");``` 
　　解析时取得的 value 会是正确的汉字字符，这种情况是怎么造成的呢？看下如所示：
!["4"](/images/java-1-6.png)
　　这种情况是这样的，ISO-8859-1 字符集的编码范围是 0000-00FF，正好和一个字节的编码范围相对应。这种特性保证了使用 ISO-8859-1 进行编码和解码可以保持编码数值“不变”。虽然中文字符在经过网络传输时，被错误地“拆”成了两个欧洲字符，但由于输出时也是用 ISO-8859-1，结果被“拆”开的中文字的两半又被合并在一起，从而又刚好组成了一个正确的汉字。虽然最终能取得正确的汉字，但是还是不建议用这种不正常的方式取得参数值，因为这中间增加了一次额外的编码与解码，这种情况出现乱码时因为 Tomcat 的配置文件中 useBodyEncodingForURI 配置项没有设置为”true”，从而造成第一次解析式用 ISO-8859-1 来解析才造成乱码的。
***
## 总结
　　本文首先总结了几种常见编码格式的区别，然后介绍了支持中文的几种编码格式，并比较了它们的使用场景。接着介绍了 Java 那些地方会涉及到编码问题，已经 Java 中如何对编码的支持。并以网络 I/O 为例重点介绍了 HTTP 请求中的存在编码的地方，以及 Tomcat 对 HTTP 协议的解析，最后分析了我们平常遇到的乱码问题出现的原因。
　　综上所述，要解决中文问题，首先要搞清楚哪些地方会引起字符到字节的编码以及字节到字符的解码，最常见的地方就是读取会存储数据到磁盘，或者数据要经过网络传输。然后针对这些地方搞清楚操作这些数据的框架的或系统是如何控制编码的，正确设置编码格式，避免使用软件默认的或者是操作系统平台默认的编码格式。

## 参考资料
[《Unicode 编码规范》](http://www.unicode.org/charts/)，详细描述了 Unicode 如何编码。
[《ISO-8859-1 编码》](https://en.wikipedia.org/wiki/ISO/IEC_8859-1)，详细介绍了 ISO-8859-1 的一些细节。
[《RFC3986 规范》](http://www.ietf.org/rfc/rfc3986.txt)，详细描述了 URL 编码规范。
[《HTTP 协议》](http://www.w3.org/Protocols/)，W3C 关于 HTTP 协议的详细描述。
[《Tomcat 系统架构与设计模式》](https://www.ibm.com/developerworks/cn/java/j-lo-tomcat1/)，了解 Tomcat 中容器的体系结构，基本的工作原理，以及 Tomcat 中使用的经典的设计模式介绍。
[《Servlet 工作原理解析》](http://www.ibm.com/developerworks/cn/java/j-lo-servlet/)，以 Tomcat 为例了解 Servlet 容器是如何工作的。
[developerWorks Java 技术专区](http://www.ibm.com/developerworks/cn/java/)