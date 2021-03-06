---
title: Android中的Https那些事
---

![how+does+https+work](D:\MyDocument\Picture\how+does+https+work.png)

# 加密算法简介

1. **对称加密**：采用对称密码编码技术，也就是编码和解码采用相同描述字符，及加密和解密使用相同的密钥，实现这种加密技术的算法称对称加密算法。对称加密使用简单、密钥较短，加密和解密过程较快，耗时短。
2. **非对称加密**：这种加密算法需要两个密钥：公钥（public key）和私钥（private key），两者是一对，如果用公钥加密，只能用私钥才能解密。非对称加密保密性好，但加密和解密花费的时间较长，不适合对大文件加密而只适合对少量的数据加密。
3. **Hash算法**：单向算法，通过Hash算法可以对目标数据生成一段特定长度、唯一的hash值，但是不能通过这个hash值重新计算出原始的数据，因此称之为摘要算法。经常被用在不需要数据还原的密码加密以及数据完整性校验上。

# Https传输数据的流程

传输流程如下所示：

![https传输过程](D:\MyDocument\Picture\https传输过程.jpg)

1. **客户端发起https请求**

2. **服务端的配置**：采用HTTPS协议的服务器必须要有一套数字证书，可以自己制作或者CA证书。

   证书就是一对公钥和私钥，公钥给别人加密使用，私钥给自己解密使用

3. **传送证书**：这个证书其实就是公钥，只是包含了很多信息，如证书的颁发机构，过期时间等。

4. **客户端解析证书**：这部分的工作由客户端的TLS来完成，首先会验证公钥是否有效，比如颁发机构，过期时间等，如果发现异常，则会弹出一个警告框，提示证书存在问题；如果证书没问题，则生成一个随机值，然后用证书（公钥）对该随机值进行加密

5. **传送加密信息**：服务端获取客户端用证书加密后的随机值，以后客户端和服务端的通信就可以通过这个随机值来进行加密解密了。

6. **服务端解密信息**：服务端用私钥解密后，得到客户端生成的随机值，然后把内容通过该值进行对称加密。

7. **传输加密后的信息**： 这部分信息是服务段用私钥（客户端生成的随机值）加密后的信息，可以在客户端被还原。

8. **客户端解密信息**：客户端用之前生成的私钥解密服务段传过来的信息，于是获取了解密后的内容。