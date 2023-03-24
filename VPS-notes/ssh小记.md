# ssh小记



## 什么是SSH

SSH是一种网络协议，用于计算机之间的加密通信。



## 公钥Public Key与私钥Private Key

SSH需要生成公钥Public Key和私钥Private Key, 常用的是使用RSA算法生成`id_rsa.pub`和`id_rsa`。 公钥Public Key(`id_rsa.pub`)是可以暴露在网络传输上的，是不安全的。而私钥Private Key(`id_rsa`)是不可暴露的，只能存在客户端本机上。 所以公钥Public Key(`id_rsa.pub`)的权限是644，而私钥Private Key(`id_rsa`)的权限只能是600。如果权限不对，SSH会认为公钥Public Key(`id_rsa.pub`)和私钥Private Key(`id_rsa`)是不可靠的，就无法正常使用SSH登陆了。

同时在服务端会有一个`~/.ssh/authorized_keys`文件，里面存放了多个客户端的公钥Public Key(`id_rsa.pub`)，就表示拥有这些Public Key的客户端就可以通过SSH登陆服务端。

## SSH公钥登陆过程

1. 客户端发出公钥登陆的请求(`ssh user@host`)
2. 服务端返回一段随机字符串
3. 客户端用私钥Private Key(`id_rsa`)加密这个字符串，再发送回服务端
4. 服务端用`~/.ssh/authorized_keys`里面存储的公钥Public Key去解密收到的字符串。如果成功，就表明这个客户端是可信的，客户端就可以成功登陆

由此可见，只要多台电脑上的的公钥Public Key(`id_rsa.pub`)和私钥Private Key(`id_rsa`)是一样的，对于服务端来说着其实就是同一个客户端。所以可以通过复制公钥Public Key(`id_rsa.pub`)和私钥Private Key(`id_rsa`)到多台电脑来实现共享登陆。



## 多个电脑共用一个ssh

比如我们有多个设备，但不想每个设备上生成一个ssh key，然后去github或其他网站上添加，那样的话，ssh key会比较多，搞起来会比较乱，所以我们想在不同的设备上使用同一个ssh。

做法是，我们只需要将 id_rsa(私钥) 和 id_rsa.pub(公钥) 复制一份到其他电脑就好了。

有点需要注意：

确保两个文件的权限是正确的，id_rsa是600，id_rsa.pub是644，比如：**-rw------- 1  id_rsa**、**-rw-r--r-- 1  id_rsa.pub**

如果上述方法不可以，那就先在另外的设备上创建好同名的ssh，然后用之前有ssh的设备上的ssh去覆盖。但是，同样要注意 id_rsa 和 id_rsa.pub 文件的权限问题。