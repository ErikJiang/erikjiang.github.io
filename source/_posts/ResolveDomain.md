title: "有关域名解析"
date: 2015-03-18 21:08:06
categories: "技术" 
tags:
  - c
  - DNS 
type: "tags"
---


*这里纪录一下服务端环境，单个域名在对应的多条IP信息情况下，配置与解析处理。*

<!--more-->

#### 关于hosts允许主机对应多条IP的参数配置
---
我们知道 *gethostbyname()* 函数会使用到 hosts 文件进行域名解析，而在解析过程中将按照顺序读取文件内容；

`/etc/hosts` 文件内容如下所示： 

````
    192.168.1.1		www.dns.com
    192.168.1.2		www.dns.com
    192.168.1.3		www.dns.com
    192.168.1.4		www.dns.com

````

一般情况下，若未做任何host参数配置的话，仅能够解析到第一条内容(即：192.168.1.1)，
但如果在 */etc/host.conf* 配置文件中添加 *multi on* (允许主机存在多条IP)，则能够解析到关于该域名的多条IP信息；

#### 主机信息及IP地址列表
---
hostent(host entry)，该结构体记录主机的信息，包括主机名、别名、地址类型、地址长度和地址列表。之所以主机的地址是一个列表的形式，原因是当一个主机对应多个网络接口时，就存在多个地址。

````
    struct hostent
    {
        char*	h_name;         /* official name of host */
        char*	h_aliases;      /* alias list */
        short	h_addrtype;     /* host address type */
        short	h_length;       /* length of address */
        char**	h_addr_list;    /* list of addresses */
#define	h_addr	h_addr_list[0]  /* address, for backward compat */
    };

````

*h\_addr\_list* 为指针数组，数组中每个元素都是 *in\_addr* 型指针。

具体域名IP解析的代码如下：

```cpp
void resolve_domain_name(const unsigned char* host_name)
{
    struct hostent *host_info = NULL;
    char target_ip[32];
    char **pptr;
    char *address = NULL;

    memset(target_ip, 0, sizeof(target_ip));
    //获取主机信息
    host_info = gethostbyname(host_name);
    if (NULL == host_info)
    {
        printf("DNS resolve error![%s]\n", host_name);
        return;
    }

    //获取IP地址列表
    pptr = host_info->h_addr_list;
    while(NULL != *pptr)
    {
        //[二进制整数] 转换为 [点分十进制]
        address = inet_ntoa(*(struct in_addr *)*pptr);
        //address = inet_ntop(host_info->h_addrtype, *pptr, target_ip, sizeof(target_ip));
        if (NULL == address)
        {
            printf("Addr convert error!\n");
            return;
        }
        printf("IP Address = [%s]\n", address);
        pptr ++;
    }
}

``` 

嗯，到这里，另外需要补充的是二进制整数转点分十进制时，“inet_ntoa函数陷阱”的问题，
inet_ntoa() 函数返回值为一个指向字符的指针，且是一个由inet_ntoa()函数控制的静态固定指针，
在每次调用 inet_ntoa() 后，它将会覆盖上次调用时所得的IP地址。
所以如果上一次返回的结果仍然需要，则最好定义一个缓存来做保留。


