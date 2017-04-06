title: "SSL编程小札"
date: 2015-03-09 22:42:04
categories: "技术"  
tags: 
  - c
  - ssl 
type: "tags"
---

### SSL简介：
---

安全套接层(Secure Socket Layer，SSL)是一种在两台机器之间提供安全通道的协议。它具有保护传输数据以及识别通信机器的功能。安全通道是透明的，意思就是说它对传输的数据不加变更。客户端与服务器之间的数据是经过加密的，一端写入的数据完全是另一端读取的内容。透明性使得几乎所有基于TCP的协议稍加改动就可以在SSL上运行，非常方便。

SSL最初的版本是由网景(Netscape)设计的，但1996年由于市场份额下滑等因素，网景决定将SSL移交给因特网工程任务组（IETF）进行标准化制定，随后便产生了TLS，也被称作SSL3.1版本；

SSL/TLS的特性确保了通信双方可以进行端点认证(ndpoint authentication)互验对方身份，并保证了数据的消息完整性(message integrity)以及私密性(confidentiality)。
<!--more-->
### SSL编程套路：
---

SSL通信模型采用的是标准C/S结构，因此基于OpenSSL的程序就被分成两个部分：Client和Server。且程序遵循着以下几个重要步骤：

#### OpenSSL初始化

在使用OpenSSL之前，必须进行相应的初始化工作。完成初始化功能的函数原型如下： 
```cpp
int SSL_library_int(void);              // 初始化SSL算法库函数，调用SSL函数之前必须调用此函数
void OpenSSL_add_all_algorithms(void);  //加载SSL所有算法
void SSL_load_error_strings(void);      //错误信息的初始化  
```
在建立SSL连接之前，要为Client和Server分别指定本次连接采用的协议及其版本，目前能够使用的协议版本包括SSLv2、SSLv3、SSLv2/v3和TLSv1.0。SSL连接若要正常建立，则要求Client和Server必须使用相互兼容的协议。

#### 创建CTX

在OpenSSL中，CTX是指SSL会话环境。建立连接时使用不同的协议（ [*详见*](http://www.openssl.org/docs/ssl/SSL_CTX_new.html) ），其CTX也不一样。创建CTX的过程中会依次用到以下OpenSSL函数：
```cpp
//客户端、服务端都需要调用的
SSL_CTX_new()                               //申请SSL会话环境
//若有验证对方证书的需求，则需调用
SSL_CTX_set_verify()                        //指定证书验证方式
SSL_CTX_load_verify_location()              //为SSL会话环境加载本应用所信任的CA证书列表
//若有加载证书的需求，则需调用
SSL_CTX_use_certificate_file()              //为SSL会话加载本应用的证书
SSL_CTX_use_certificate_chain_file()        //为SSL会话加载本应用的证书所属的证书链
SSL_CTX_set_default_passwd_cb_userdata()    //设置私钥的密码(如果有的话);注：需要在加载私钥之前设置
SSL_CTX_use_PrivateKey_file()               //为SSL会话加载本应用的私钥
SSL_CTX_check_private_key()                 //验证所加载的私钥和证书是否相匹配
```

对于SSL证书的双向认证，这里需要补充的是，可能需要client以及server配置如下的接口函数：
```cpp
/* 设置验证证书链的最大长度 */
void SSL_CTX_set_verify_depth(SSL_CTX *ctx,int depth);
/* SSL_CTX_set_verify的回调函数，需要自己去写实现 */
int verify_callback(int preverify_ok, X509_STORE_CTX *x509_ctx);
/* 设置认证的模式及是否开启认证回调 */
SSL_CTX_set_verify(SSL_CTX* ctx,int mode,int (*verify_callback)(int,X509_STORE_CTX*));
/* 加载CA证书文件 */
SSL_CTX_load_verify_locations(SSL_CTX* ctx,const char* CAfile,const char* CApath);
```
另外还需要注意认证的模式（ [*详见*](https://www.openssl.org/docs/ssl/SSL_CTX_set_verify.html) ），常用的模式如下：

> 1. SSL_VERIFY_NONE:
    不会进行证书验证
> 2. SSL_VERIFY_PEER:
    要求对方提供证书进行验证的模式，该模式在服务端类似于一种自愿验证的模式，当遇到客户端不提供证书时，是否继续通信可自行决定；但该模式运用于客户端，若服务端无证书则会导致SSL握手失败。
> 3. SSL_VERIFY_FAIL_IF_NO_PEER_CERT:
    服务端使用的一种的模式，此模式下，相当于强制验证客户端的证书，若客户端无证书则SSL握手失败。

#### 建立SSL套接字

在此之前要先创建普通的流套接字，完成TCP三次握手，建立普通的TCP连接。然后创建SSL套接字，并将其与流套接字绑定。这一过程中会使用以下几个函数：
```cpp
SSL *SSl_new(SSL_CTX *ctx);         //申请一个SSL套接字
int SSL_set_fd(SSL *ssl,int fd);    //绑定读写套接字
int SSL_set_rfd(SSL *ssl,int fd);   //绑定只读套接字
int SSL_set_wfd(SSL *ssl,int fd);   //绑定只写套接字
```

#### 完成SSL握手

在这一步，我们需要在普通TCP连接的基础上，建立SSL连接。与普通流套接字建立连接的过程类似：
Client使用函数SSL_connect()发起握手（ *类似于流套接字中用的connect()* ），而Server使用函数SSL_accept()对握手进行响应（ *类似于流套接字中用的accept()* ），从而完成握手过程。两函数原型如下：
```cpp
int SSL_connect(SSL *ssl);
int SSL_accept(SSL *ssl);
```

#### 鉴别证书信息

握手过程完成以后，Client以及Server通常能够获取到对方的证书信息，并对信息进行鉴别。具体实现中会用到以下函数:
```cpp
X509 *SSL_get_peer_certificate(SSL *ssl);   //从SSL套接字中获取对方的证书信息
X509_NAME *X509_get_subject_name(X509 *a);  //获取证书所有者名字
X509_NAME *X509_get_issuer_name(X509  *a);  //获取证书颁发者名字
```

#### 进行数据传输

经过前面的一系列过程后，就可以进行安全的数据传输了。
在数据传输阶段，需要使用SSL_read( )和SSL_write( )来代替普通流套接字所使用的read( )和write( )函数，以此完成对SSL套接字的读写操作,两个新函数的原型分别如下：
```cpp
int SSL_read(SSL *ssl,void *buf,int num);           //从SSL套接字读取数据
int SSL_write(SSL *ssl,const void *buf,int num);    //向SSL套接字写入数据
```

#### 会话结束

当Client和Server之间的通信过程完成后，就使用以下函数来释放前面过程中申请的SSL资源： 
```cpp
int SSL_shutdown(SSL *ssl);         //关闭SSL套接字
void SSl_free(SSL *ssl);            //释放SSL套接字
void SSL_CTX_free(SSL_CTX *ctx);    //释放SSL会话环境
```

### 具体源码示范：

- [*SSLSocketDemo*](https://github.com/JiangInk/SSLSocketDemo)

---

*本文参考如下资料整理：*

*1. 《SSL与TLS》*

*2. 《[使用OpenSSL API 建立SSL安全通信的一般流程](http://blog.bccn.net/Ping_Fani07/13407)》*


