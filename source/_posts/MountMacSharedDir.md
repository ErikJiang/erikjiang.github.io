title: "Mac共享目录挂载"
date: 2015-04-05 15:16:59
categories: "技术"   
tags: [Mac,mount] 
---

最近，这个共享目录挂载的问题一直困扰着我，妈蛋！
我的目标很简单，就是需要将一个Mac共享目录mount到Linux中去，
但为什么这般折磨……
「上帝啊！請寬恕這個狂躁的機智少年！」

<!--more-->

####  第一步，共享目录：

1、 Mac系统“系统偏好设置”中，有个“共享”功能；
2、进入“共享”后，勾选“文件共享”服务，然后在右侧，我们可以添加需要共享的文件夹以及相关用户访问权限；
3、如果你需要“SMB”和“AFP”，那就点击“选项...”，添加该服务以及账户；

####  第二步，创建用户：

1、Mac系统“系统偏好设置”中，有个“用户与群组”功能，添加一个共享用户；
2、在添加前，需要解锁才能操作，解锁后，点击“＋”号，新账户选择“仅限共享”，其他随意，然后“创建用户”；

####  第三步，其余操作：

1、需要讲你新创建的“仅限共享”的用户，添加到共享文件夹的用户中去，设置“读与写”权限，这样，该用户才能够访问该文件夹；
2、在Linux终端中，首先需要安装cifs： 
    ``` bash    
    yum install cifs-utils
    ``` 
3、Linux创建挂载点： 
    ``` bash 
    sudo mkdir -p /mnt/MountPointName
    ```  
4、开始挂载： 
    ``` bash 
    mount -t cifs //MacHostOrMacIP/SharedDirectory /mnt/MountPointName/ -o username=UserName,password=Password,nounix,sec=ntlmssp
    ``` 
#### 但是，最终当我执行的时候，它不好使了～ 
    ``` bash 
    # mount -t cifs //192.168.1.101/Github /mnt/jiangink/ -o username=jiangink,password=123456,nounix,sec=ntlmssp
    mount error(13): Permission denied
    Refer to the mount.cifs(8) manual page (e.g. man mount.cifs)
    ``` 
