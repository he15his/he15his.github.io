**假设没有https，而你要用http进行可信并且保密的通信，你要怎么做？**

首先最简单的就是用js加密数据。
过程主要是：浏览器在js中加密数据；传给服务器；服务器接收数据；服务器解密；服务器返回加密的数据；浏览器接收数据；浏览器解密。这样完成一次通信。
显然用对称加密在这是不靠谱的。浏览器要解密你的数据需要你加密用的秘钥，而只可能是浏览器告诉服务器秘钥，这样，中间人很容就得知了秘钥，毫无安全可言。

为了避免这个问题，可以采用一种秘钥协商算法（比如DH），这样即便中间人获取了双方的通信，也没办法知道你的秘钥。不过即便这样，还是无法防范中间人攻击。如果中间人不但能获取通信，还能介入，就可以将自己伪造成服务器与浏览器通信，而对于真正的浏览器这伪装成浏览器，这样，中间人参与了秘钥协商的过程，那么知道秘钥，知道整个通信内容。

好，换一个思路，用公钥加密，这样就不需要传递秘钥了。
浏览器用服务器的公钥加密数据，并将自己的公钥加密给服务器；服务器用自己的私钥解密，返回的数据用浏览器的公钥加密；浏览器收到数据用自己的私钥解密。完成一次通信。
即使这样，也是不安全的。因为浏览器不可能存储所有服务器的公钥，所以只能是以某种途径获取。中间人也能获取到这个公钥，中间人可以欺骗浏览器，以自己的公钥替换服务器的公钥，这样就能伪装成服务器与浏览器进行通信，并且劫持双方的通信。

可以看出，重点是可信的问题。不过没关系，我们可以加入证书认证系统，这样就能确认双方的身份，避免中间人攻击。然后用前面提到的方式加密数据，然后再处理一些诸如加密算法选择，消息完整性检验等等，这样的通信基本上就安全了。不过这差不多就是讲https重新“发明”了一遍。

http是一点安全性都没有。

https，协议本身是安全的。但是使用https的通信不一定。造成不安全的原因主要有：

1、使用了不安全的证书。

2、使用了不安全的加密算法。

3、有缺陷的实现方式。

4、系统或者软件漏洞。