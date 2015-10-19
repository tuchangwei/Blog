title: "[译]如何用Node.js实现SSL/TLS"
date: 2015-09-25 21:56:51
tags: HTTPS
categories: Node.js
---
最近iOS9发布，强制App使用HTTPS传输,做为一个入坑多年的iOS程序员，还真是不了解这其中的来龙去脉，刚好订阅的Node.js周报中推送了这篇[文章](http://www.sitepoint.com/how-to-use-ssltls-with-node-js/),赶紧翻译过来，既搞清了HTTPS又学习了Node.js，一举两得。

---
使用HTTPS正在变的越来越普遍，因此我们应该知道怎么在Node.js程序中实现SSL/TSL——不论是为了访问HTTPS资源还是为了提供加密的资源。HTTPS到底是什么意思呢？它暗示了什么？有什么限制和约束？我们将试着为所有的这些问题找到答案。

另外，我们不应该仅仅通过提供HTTPS来保护我们的客户端，而且我们也应该严格要求来自服务器的是加密过的连接。我们将会看到激活SSL/TLS层的可能性的存在，即使默认情况下不能激活。让我们先来浏览一下HTTPS的当前情况。

# HTTPS Everywhere
在2015年2月17日，HTTP/2协议被IESG许可作为推荐的标准发布。这是一个大的里程碑。现在我们可以更新我们的服务器去使用HTTP/2。一个重要的方面是它与HTTP1.1和握手机制的向后的兼容性。虽然这个标准并没有指明要强制加密，但大部分浏览器将仅仅支持TLS之上的HTTP/2协议。这给了HTTPS另一方面的促进。最终，HTTPS everywhere!

那我们的协议栈看起来是怎样的呢？从一个在浏览器中跑的网站的透视来看，我们大致有下面的一些层到达IP层。

1. 用户浏览器
2. HTTP
3. SSL/TLS
4. TCP
5. IP

对比SSL/TLS顶端的HTTP协议来说，HTTPS啥也不多。因此，所有对于HTTP的规则仍然适用。那额外的层给了我们什么呢？给了很多好处。我们通过拥有Keys和证书来获得授权。当连接以一种不对称的加密时，还有某种隐私和机密被保护。最后但不仅仅还有数据的真实性也会得到保护，比如，那些传输完成的数据在传输过程中是不能被改变的。

其中一个最普遍的错误观念是使用SSL/TLS需要太多的资源并且会拖慢服务器的速度。这无异是错误的。我们也不需要任何特制的拥有密码单元的硬件。甚至对于Google来说，SSL/TLS层的消耗也小于百分之一的CPU的加载。更进一步的讲，HTTPS的网络开支会低于HTTP的百分之二。总而言之，如果因为一点网络开支而放弃HTTPS，这是没有道理的。

{%asset_img 1442392075istlsfastyet.png %}

最新的版本是TLS1.2（目前1.3版本作为工作草案是可用的）。TLS是SSL的继承者，在最新发布的SSL3.0中可以使用。从SSL到TLS的改变杜绝的互通性。但是基本的流程是不变的。我们有三种不同的加密途径。第一个是证书链公钥基础结构，第二个是提供公钥密码作为Key交换。最后，第三种是对称。这里我们对于数据传输有密码逻辑。

我们可以使用很多哈希算法，比如MD5，SHA1，SHA256. MD5已经被降级了，我们不应该使用它了。SHA1仍然是可接受的，但可能不久后也会被降级。比较好的是SHA256，它当前拥有更多的用户。用SHA256有一个问题是老的系统是否支持它，比如，在Window XP上，SHA256只能通过安装Service Pack 3才能使用。

HTTPS也正在获得更多客户端的关注。隐私和安全问题一直存在，但随着在线访问数据和服务人数日益增长都越来越关受注。一个有用的浏览器插件是 `HTTPS Everywhere`,它可以加密我们与大多数网站之间的交流。

{%asset_img 1442392093everywhere.png %}

这个插件的作者认识到很多网站仅仅有一部分提供了HTTPS功能。这个插件允许我们重新请求这些只有部分支持HTTPS的网站。

可选的，我们也可以同时屏蔽HTTP请求（如上面的截图所示）

# Basic Communication
这个证书的认证过程包括验证证书签名的有效性和过期时间。此外我们还要验证它链接到可信任的根证书。最后，我们需要去检测它是否已经被撤销。

HTTPS握手的流程图像下图一样。我们从客户端的初始化开始，伴随其后的是一条带有证书和Key交换的信息。在服务器返回完整的包之后，客户端开始Key交换和密码详情的传输。这时，客户端已经完成。最后服务器确认对密码详情的挑选，然后关闭握手。

{%asset_img 1442392132sequence.png %}

这整个流程被触发独立于HTTP。如果我们决定使用HTTPS，仅仅改变的是对Socket的处理。客户端仍然发起的是HTTP请求，但是socket将会执行前面描叙的握手协议和数据加密。

所以，在Node.js中，什么包能处理SSL／TLS呢？下面我们将会简单的讲到三种标准的包。

## ssl-root-cas
当我们在处理自定义的或过期的证书遇到问题时，`ssl-root-cas`能够帮助我们。当许可证不可用时，它也很有帮助。

归结起来，大部分时间会用到下面的命令。这个`inject`方法修改`https.globalAgent.options.ca` 去包含一系列的证书。

```
require('ssl-root-cas').inject();
```

但是，有时也可能需要增加本地自定义的证书。

```
require('ssl-root-cas').addFile('my-cert.crt');
```

从 `ssl-root-cas` 模块返回的这个对象是可串的，因此能够用来增加多个文件或注入多个标准文件在一次调用中。

## https
类似于`http`,我们可以使用`https`. 这个主要的区别是如果我们需要发布我们的服务器，我们需要提供一个key。这将会在下一章节讨论。

现在我们仅仅看一下我们怎么创建一个HTTPS请求。

```
var https = require('https');

https.get('https://www.amazon.com/', function(res) {
   console.log('statusCode: ', res.statusCode);

   res.on('data', function(d) {
      process.stdout.write(d);
   });
});
```

基本上，`https`提供和`http`相同的功能。但是在最上层增加了SSL/TLS层。API大部分是相同的。

## tls
`tls`模块需要`openssl`工具包，下一章节将会讨论`openssl`工具包。这个模块为加密的流传输提供了SSL/TLS。本质上，`https`也使用了`tls`。所以，使用`tls`API时感觉会和`http`模块的方法相关就不是啥惊奇的事了。

下面简单的代码创建了一个在8000端口监听连接的服务器，进来的连接讲接收一个加密后的数据。

```
var tls = require('tls');
var fs = require('fs');

var options = {
   key  : fs.readFileSync('private.key'),
   cert : fs.readFileSync('public.cert')
};

var server = tls.createServer(options, function (res) {
   res.write('Hello World!');
   res.pipe(res);
}).listen(8000);
```
类似的，我们也可以连接这样一个服务器。
```
var tls = require('tls');
var fs = require('fs');

var options = {
   key  : fs.readFileSync('private.key'),
   cert : fs.readFileSync('public.cert')
};

var client = tls.connect(8000, options, function () {
   console.log(client.authorized ? 'Authorized' : 'Not authorized');
});

client.on('data', function (data) {
   console.log(data.toString());
   client.end();
});
```
现在，我们应该看一下怎样生成我们自己的key和证书文件。

# Generating Certificates
从技术的角度上来看，证书并不是强制的。不过，由于网络的开放性，如果不增加证书，中间人攻击就会变得太平常。因此，一个被可信赖证书认证（CA）的证书是按需求而定的。CA确保了证书的持有者就是这个证书的创建者。

下面，我们会使用`openssl`工具。这个程序将会给我们用RSA加密生成私有Key的能力。我们也能够发起一个证书签名的请求（CSR）。更进一步的，这个OpenSSL项目的核心库还能生成自签名的证书。这些证书可以用来做测试，它们不应该公开使用。

第一步是创建一个私有的Key。作为一个例子，我们可以使用下面的命令，生成一个叫server.enc.key的文件。

```
openssl genrsa -des3 -out server.enc.key 1024
```
这个生成的RSA Key是一个1024字节的用三倍DES加密的key。这个文件是人类可读的。作为一个轻量的变形，我们也可以省略掉`-des3`选项，并且改变加密的强度，比如`2048`。虽然在这个例子中我们使用了1024字节的key，但是对于真是的Key，我们应该最少使用2048字节的长度。

一旦我们拥有一个生成的Key，我们需要发起一个证书签名的请求。一个证书可以用如下的命令生成：

```
openssl req -new -key server.enc.key -out server.csr
```

生成的CSR存在于server.csr文件中。我们需要对这个文件进行自签名。

到这时，当使用server.enc.key文件中的key时，我们需要输入一个密码。由于我们仅在内部使用这个证书，所以我们可以移除密码保护。移除3倍的DES加密用下面的命令。

```
openssl rsa -in server.enc.key -out server.key
```

现在这个原始加密的server.enc.key文件被转换成非加密的文件server.key。

最后，我们要对这个证书进行自签名。自签名的证书将会在浏览器中生成一个错误。这是因为这个自签名的证书的权威机构是未知的，因此不被信任。为了生成一个临时的证书server.crt，我们需要发送下面的命令。

```
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```
这个证书将会在发出这个命令的365天内过期。

# Integration with Express

使用HTTPS代替HTTP是可能的也是被期望的。我们将开始使用Express编写一个小的Node.js应用。

```
var http = require('http');
var app = require('express')();

app.get('/', function (req, res) {
   res.send('Hello World!');
});

http.createServer(app).listen(3000, function () {
   console.log('Started!');
});
```

启动HTTPS我们需要做啥更改呢？显然，我们需要导入一个不同的包。最后我们也需要提供正确的Key／证书。所有文件的内容需要被展现，它们被传递到`https`包的`createServer`方法。

让我们用SSL/TLS重写我们的Express例子:

```
var fs = require('fs');
var https = require('https');
var app = require('express')();
var options = {
   key  : fs.readFileSync('server.key'),
   cert : fs.readFileSync('server.crt')
};

app.get('/', function (req, res) {
   res.send('Hello World!');
});

https.createServer(options, app).listen(3000, function () {
   console.log('Started!');
});
```

就是如此简单！在上面的例子中，我们假设key和证书都位于工作目录中。这仍然有很大的提升空间。一个可能性是放这段代码到创建server的module中。这个module将会根据选择不同的协议来选择正确的包。另外，这个端口，key和证书应该驱逐到一个`settongs`对象中。

一个简单的实现可能看起来如下：

```
var fs = require('fs');

function setup (ssl) {
   if (ssl && ssl.active) {
      return {
         key  : fs.readFileSync(ssl.key),
         cert : fs.readFileSync(ssl.certificate)
      };
   }
}

function start (app, options) {
   if (options)
      return require('https').createServer(options, app);

   return require('http').createServer(app);
}

module.exports = {
   create: function (settings, app, cb) {
      var options = setup(settings.ssl);
      return start(app, options).listen(settings.port, cb);
   }
};
```

关键的地方是`settings`中的`ssl`对象，如果这个对象存在并且有个叫`active`的参数设置为true，那么我们将会用提供的Key／证书启动SSL／TLS。

# Conclusions

在2015年，没有理由再去忽视HTTPS了。未来的方向是清晰可见的——到处都是HTTPS！在Node.js中， 我们有很多选择去管理SSL／TLS。我们可以发布我们HTTPS的网站，我们可以对加密的网站发起请求，我们也能授权哪怕不被信任的证书。

