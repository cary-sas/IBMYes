# 梅林固件 armV7下， v2ray 无法使用 ws + tls + CDN 的解决方案

最近SSR 有点不好使了，开始尝试搭载v2ray, 现在最流行的加密方式是 ws+tls+CDN+Cloudflare 反代。 然而过程并非一帆风顺，摸索了个把月，直到最近才完美实现。

服务端的搭载其实在网上教程很多，大家随便找一个就行，不需要完全按照我的方法，在这里我主要分享一下梅林科学上网插件的配置和更新。 

### 具体做法：

**前期：** 准备工作（包括：1，[域名申请](https://my.freenom.com/)，2，用[cloudflare](https://cloudflare.com/)添加site，并绑定VPS的IP）。不熟悉的可以参考[这里](https://www.v2rayssr.com/v2raynginx.html)。



![QQ截圖20201126111156.png](https://i.loli.net/2020/11/26/bR8K2Zz9lxwjitP.png)  

  

**中期：** VPS 操作系统还是推荐Debian或者Ubuntu，版本越新越好。安装配置 v2ray, nigix 可以参考[这里](https://ssr.tools/1317)，读者只需要用到下面这个一键安装配置脚本：

```shell
curl -O https://raw.githubusercontent.com/atrandys/v2ray-ws-tls/master/v2ray_ws_tls1.3.sh && chmod +x v2ray_ws_tls1.3.sh && ./v2ray_ws_tls1.3.sh
```

脚本运行的时候只需要填入申请的域名，后面就全自动化了，耐心等待10分钟。 

当配置信息输出到终端上后，我们就要配置客户端信息了。 



本人比较喜欢用路由器客户端，因为这样可以一劳永逸，让局域网下面的每个设备都能科学上网。 

所用到的路由器都是 koolshare 改版的梅林系统，版本号为380.70_0-X7.9.1， 使用的科学上网插件是fancyss 开发的[shadowsocks ](https://hq450.github.io/fancyss/)(开发者已经好久没有更新了)， 对于SS/SSR完全没问题，但是对于v2ray, 由于内在的v2ray core 的版本太老，（自动更新只能更新到 4.22.1） 这就导致尽管你的信息填写无误，但是一旦保存并应用，不是提示“V2Ray配置文件没有通过测试，请检查设置!!! ” 就是开启后“国外连接”出现红色的大叉。 

经过不懈的研究和努力，这个问题终于得到了解决。

### 客户端搞起！

#### 1，手动更新路由器中的v2ray core，也就是v2ray和v2ctl两个二进制文件。

~~方法介绍在[这里](https://github.com/hq450/fancyss/issues/1028)。~~切记**不要**使用v2ray core[官方仓库](https://github.com/v2ray/v2ray-core/releases/tag/v4.31.0)下的 Linux-armv7a安装包，而是去尝试armv5的，或者直接下载这个](https://github.com/YUMEYA/v2ray-core/releases/)现成的 。

下载后建议用UPX压缩v2ray二进制文件以减小体积，然后再去替换老版本的文件。方法如下：

下载UPX；[官网链接](https://github.com/upx/upx/releases)，[百度网盘/提取码: 65vm](https://pan.baidu.com/s/1VEdD4hyPE1RwsghSm6JRbw)。解压，进入UPX文件夹，将v2ray和v2ctl两个文件复制到UPX文件夹，然后在UPX文件夹打开CMD窗口；

   输入 upx v2ray 即可压缩v2ray。

   再输入 upx v2ctl 来压缩v2ctl文件。

   压缩之后的文件体积大概是源文件的35%左右。

![QQ截圖20201119150731.png](https://i.loli.net/2020/11/26/18oZxsGetJlVXB6.png)

新手推荐使用[winscp](https://winscp.net/eng/download.php)操作去替换v2ray和v2ctl文件， 即

   建立session，协议选SCP, 地址为路由器网关地址，登录后点击界面右下角的“已隐藏”，即可以显示所有文件。

  进入 jffs/.koolshare/bin文件夹，拖到最下方就可以看到v2ray和v2ctl文件了，从左侧本机文件夹直接把同名文件直接拖到右侧，覆盖。 

  右击这两个文件，选择“属性”，修改权限为“0755”. 



#### 2，修改ss 配置脚本， **（最关键就是这步）**

 修改 /koolshare/ss/ssconfig.sh 中 

```
\"serverName\": null ---> \"serverName\": \"$ss_basic_v2ray_network_host\"
```

保存后重启插件。



贴一张流程图以便理解

![QQ截圖20201119113215.png](https://i.loli.net/2020/11/26/gXYbtmKdvn9E6Ru.png)

**上述两个步骤总结下来就这几行代码：**

```shell
wget https://github.com/cary-sas/IBMYes/raw/master/v2ray
wget https://github.com/cary-sas/IBMYes/raw/master/v2ctl
chmod 0755 v2ctl
chmod 0755 v2ray
mv v2ctl /jffs/.koolshare/bin
mv v2ray /jffs/.koolshare/bin

sed -i 's/\\"serverName\\"\: null/\\"serverName\\"\: \\"\$ss_basic_v2ray_network_host\\"/'  /koolshare/ss/ssconfig.sh 
```

到这里就基本结束了。 



#### 路由器里添加v2ray节点信息

记得，如果你只是单纯地使用你VPS所在的域名，那就只需要在第一行“地址”里面填上域名地址，如果你需要套用CF worker的话， 第一行“地址”需要填上 “cloudflare.com” 或者优选IP， 然后在“伪装域名(host)”里面填上CF中的worker域名。

![QQ截圖20201118184451.png](https://i.loli.net/2020/11/26/Zy8gU4zOaAG9slj.png)



### 一些后续优化

#### CF中worker反代域名

在cloudflare.com中创建一个worker， 编辑，使用下面的代码，替换其中的hostname 为你申请到的域名。保存并部署。

![QQ截圖20201126140123.png](https://i.loli.net/2020/11/26/1kijWHUBpyGtdsA.png)

![QQ截圖20201126140447.png](https://i.loli.net/2020/11/26/Ts9PIqZUWMXmRH2.png)

```
addEventListener(
"fetch",event => {
let url=new URL(event.request.url);
url.hostname="xxx.herokuapp.com";
let request=new Request(url,event.request);
event. respondWith(
 fetch(request)
 )
 }
)
```



#### 优选IP

电信用户推荐使用。方法很多，我是使用的[这个](https://github.com/badafans/better-cloudflare-ip)工具，Linux版的不能在梅林路由器上直接运行，所以还是老老实实用windows 版的，使用的时候建议关闭科学上网插件，以免受到干扰。 

具体操作自己看使用说明吧。 



优化后可以在电信网络下大大提高翻墙的速度。



##### 关于优选IP的思考：

手动筛选IP有点麻烦，有没有办法可以在科学上网插件中让某个DNS 自动解析一个最好的IP呢？