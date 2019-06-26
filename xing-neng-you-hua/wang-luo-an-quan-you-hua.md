## **App网络安全实战**

在App安全上的投入再多也不会过，不过安全问题上所投入的开发资源应该根据开发团队技术积累，产品发布deadline，用户规模及产品关注度等综合因素考量。结合这些因素我把App分为三类，各类App对安全级别的要求不同，投入产出也不同。

### **第一类，作坊式创业App**

这些年伴随着移动互联网的创业潮，各式各样的app出现在用户的手机端。对于创业初期的团队来说，能把业务模型尽快实现上线当然是重中之重。但很多创业团队在安全上的投入几乎为零，所导致的安全问题比想象中的要严重。我见过不少使用http明文传输用户名密码的app，其中甚至包括一些知名传统企业。其实只要照顾到一些基础方面就能过滤掉大部分的安全漏洞了。这里提供一些小tip供创业初期团队参考：

**Tip 1:尽量使用https**

https可以过滤掉大部分的安全问题。https在证书申请，服务器配置，性能优化，客户端配置上都需要投入精力，所以缺乏安全意识的开发人员容易跳过https，或者拖到以后遇到问题再优化。https除了性能优化麻烦一些以外其他都比想象中的简单，如果没精力优化性能，至少在注册登录模块需要启用https，这部分业务对性能要求比较低。

**Tip 2:不要传输密码**

不知道现在还有多少app后台是明文存储密码的。无论客户端，server还是网络传输都要避免明文密码，要使用hash值。客户端不要做任何密码相关的存储，hash值也不行。存储token进行下一次的认证，而且token需要设置有效期，使用refresh token去申请新的token。

**Tip 3:Post并不比Get安全**

事实上，Post和Get一样不安全，都是明文。参数放在QueryString或者Body没任何安全上的差别。在Http的环境下，使用Post或者Get都需要做加密和签名处理。

**Tip 4:不要使用301跳转**

301跳转很容易被Http劫持攻击。移动端http使用301比桌面端更危险，用户看不到浏览器地址，无法察觉到被重定向到了其他地址。如果一定要使用，确保跳转发生在https的环境下，而且https做了证书绑定校验。

**Tip 5:http请求都带上MAC**

所有客户端发出的请求，无论是查询还是写操作，都带上MAC（Message Authentication Code）。MAC不但能保证请求没有被篡改（Integrity），还能保证请求确实来自你的合法客户端（Signing）。当然前提是你客户端的key没有被泄漏，如何保证客户端key的安全是另一个话题。MAC值的计算可以简单的处理为hash（request params＋key）。带上MAC之后，服务器就可以过滤掉绝大部分的非法请求。MAC虽然带有签名的功能，和RSA证书的电子签名方式却不一样，原因是MAC签名和签名验证使用的是同一个key，而RSA是使用私钥签名，公钥验证，MAC的签名并不具备法律效应。

**Tip 6:http请求使用临时密钥**

高延迟的网络环境下，不经优化https的体验确实会明显不如http。在不具备https条件或对网络性能要求较高且缺乏https优化经验的场景下，http的流量也应该使用AES进行加密。AES的密钥可以由客户端来临时生成，不过这个临时的AES key需要使用服务器的公钥进行加密，确保只有自己的服务器才能解开这个请求的信息，当然服务器的response也需要使用同样的AES key进行加密。由于http的应用场景都是由客户端发起，服务器响应，所以这种由客户端单方生成密钥的方式可以一定程度上便捷的保证通信安全。

**Tip 7:AES使用CBC模式**

不要使用ECB模式，原因前面已经分析过，记得设置初始化向量，每个block加密之前要和上个block的秘文进行运算。

### **第二类，正规军App**

**All Traffic HTTPS**

全站使用HTTPS，而且是强制使用。baidu到今天（2016.04.13）还没有强制使用HTTPS。所有的流量都应该在HTTPS上产生，没有人可以决定哪些流量是可以不用考虑安全问题的。如果自建长连接使用tcp，udp或者其他网络协议，也应该实现类似HTTPS的密钥协商流程。

**Certificate Pinning**

RSA的签名机制虽然看着安全，一旦出现上游证书颁发机构私钥泄漏，或者签名流程发现漏洞等情况，中间人攻击还是会导致数据被第三方破解甚至被钓鱼。Certificate Pinning是一种与服务器证书强绑定的机制，要么绑定证书本身，需要证书更新机制配合加强安全性，要么使用公钥绑定，这样更新证书的时候只要保证私钥不变即可。现在流行的HTTP framework，iOS端如AFNetworking，Android端如OKHttp都支持Certificate Pinning。

**Perfect Forward Secrecy**

很多人会觉得非对称加密算法足够安全，只要使用了RSA或者AES，加密过后的数据就认为安全。但没有绝对的安全，无论是RSA或者AES算法本身都有可能在未来某一天被破解，正如当年的DES，甚至有传言NSA正如当年掌握了differential cryptanalysis一样，现在已经获取了某种方法来破解当前互联网当中的部分网络流量，至于到底是RSA还是AES就不得而知了。

未来计算机的计算能力是个未知数，或许某一天brute force能够暴力破解的密钥长度会远超128bits（现阶段上限应该在80bits）。

2014年1月3日，美国国家安全局（NSA）正在研发一款用于破解加密技术的量子计算机，希望破解几乎所有类型的加密技术。

即使算法本身没有被破解，密钥也有可能被泄漏，技术上的原因或者政策上的因素都可能导致RSA或者ECC的私钥被泄漏。所以尽可能针对不同的session使用不同的key能够使的我们的数据更佳安全。

Forward Secrecy就是为了避免某个私钥的泄漏或者被破解而导致历史数据一起泄漏。现在google的https配置所使用的是TLS_ECDHE_RSA算法，每次对称密钥的协商都是使用ECC生成临时的公钥私钥对（之前提到过ECC在快速生成密钥对上有优势），身份验证使用RSA算法进行签名。

**每天跟踪信息安全动态**

安全的攻防战不会有穷尽的一天，算法的更替会伴随着人类对知识的无尽渴望延绵至不可预知的未来。AES说不定哪天被破解了，openSSL可能又出现新的漏洞了，google又提倡新的安全模型了，NSA的量子计算机说不准已经在悄悄解密google的流量了，每天跟踪八卦最新业界动态才是码农避免因bug而背黑锅的不二法宝。

### **第三类，带节操正规军App**

现在互联网早已渗入每个人的平常生活当中，当我们的行为越来越多的迁移到互联网这个媒介当中之后，行为本身及所产生的关联数据都将被滴水不漏的记录起来，特别是在大数据研究兴起的当下，服务提供商总是希望尽可能多的记录用户所有的行为数据。每个互联网产品的使用者都成了样本，你的购物记录，商品浏览历史，搜索引擎搜索记录，打车记录，租房记录，股票记录，甚至聊天记录等等都是样本，毫不夸张的说，如果将淘宝，微信，支付宝，快滴，美团等等高频次产品数据统一分析，基本上可以将你的身高，性别，年龄，三维，家庭住址，恋爱史，家庭成员，甚至是个人喜好，性格等等完美的呈现出来，其后果远不是一个骚扰电话带来的隐私泄漏那么简单。

移动互联网的大部分使用者还不具备强烈的安全意识，当你用手机号作为登录id方便记忆的同时，骚扰电话就可能随时来临，你在百度输入租房关键字，下一秒中介就已经电话打上门。当你允许app上传通讯录匹配可能认识的好友同时，你认识哪些人就变得一清二楚，你p2p借贷未及时归还时，你的亲朋好友第二天就收到了催债电话。我们在享受移动互联网的便利同时，付出的是个人隐私这种隐形成本。下一次，当我们感叹新app好用便利的同时，静思三分钟，好好想想我们的哪些隐私又被当白菜卖了。

在互联网受众的安全意识普遍觉醒之前，只能靠app开发商，服务提供商的节操来保证用户信息隐私安全。

带节操的App在打算记录用户行为或者数据之前会考虑下是不是真的有需要，用户的确会有需要查询历史购买记录，但有多少人会在意自己几年前花几个小时浏览了杜蕾斯的产品。

服务器作为数据存储或者转发的媒介是不是真的需要了解真实的数据为何？现在WhatsApp，Telegram都已经支持端到端的加密聊天方式，服务器本身看到的都是秘文，只做秘文转发处理，带着这样的节操设计产品，用户才会觉得安全。

WhatsApp的端到端加密安全模型是怎么样实现的呢？非常值得学习。

简单来说是严格遵循forward secrecy。每个用户在注册成功之后会在服务器存一对永久的Identity Key，一对临时的Signed Pre Key（Signed Pre Key由Identity Key签名，每隔一段时间变化一次），n对临时的One-Time Pre Key（每次建立session消耗一个）。

每次session开始建立的时候使用Identity Key，Signed Pre Key， One Time Key生成Master Secret。Master Secret再通过HKDF算法生成对称加密使用的Root Key，Chain Key，Message Key。

Forward Secrecy体现在每次sender发送的消息被ack后，都会交换新的临时ECC Key对，并更新Root Key，Chain Key，Message Key。这样网络中的流量即使被第三方缓存起来，而且某一天某个Key Pair的私钥被破解，也不会对之前的流量产生安全影响。ECC Key对会随着消息的发送不停的“Ratcheting”。这是属于非对称加密的Forward Secrecy。

在sender的消息被ack之前，也就是新的ECC Key对交换成功之前，Message Key也会通过HKDF算法不停的“Ratcheting”，确保每条消息所使用的对称密钥也不相同。这是属于对称加密的Forward Secrecy。

有兴趣深入了解的同学可以自己google：WhatsApp Security WhitePaper。