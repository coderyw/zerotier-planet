
# 使用zeroiter自建跨域局域网

这里使用zeroiter工具，具体了解可看官网或者自行`google`。

## 要求

1. 自建moon需要一台云服务器或者一台有公网ip的自己主机（要保持长期开机）。
2. 安装docker
3. 安装docker-compose
4. 防火墙开放端口 TCP:4000/9993/3180和UDP：9993

## 云端服务器或家庭主机操作

首先在机器上拉下代码，具体位置可自定义。以下云服务器或有公网ip的主机都用云服务器表述，后面不在赘述。

```bash
git clone https://github.com/coderyw/zerotier-planet.git
```

打开到具体目录内，首先去docker-compose.yaml内修改云服务器ip，这步很重要，不然后面客户端连不上。同时也可以在文件内修改后面控制台默认登录密码。如图

![image-20230928102116868](./使用zeroiter自建跨域局域网.assets/image-20230928102116868.png)

然后执行如下面命令

```bash
cd zerotier-planet

# 这个可能会超时，多试几次，如果不想等，可自行去找第三方源，换源
docker-compose up -d
```

创建planet和moon

```bash
docker cp mkmoonworld-x86_64 ztncui:/tmp
docker cp patch.sh ztncui:/tmp
docker exec -it ztncui bash /tmp/patch.sh
docker restart ztncui
```

防火墙打开4000端口后就可以进入控制台了`http://云服务器ip:4000`

防火墙打开3180端口后，可以进入plaent和moon文件下载页面`http://云服务器ip:3180`，也可以在云服务器目录中下载`你git clone到的目录/zerotier-planet/ztncui/etc/myfs/`。以上两个文件保存下来，后面在配置客户的时候会使用到。

控制台默认账号密码是`admin/coderyw.code`，具体可以在`docker-compose.yaml`里面配置。登录管理台后会提示修改密码，修改为自己密码即可。

## 控制台配置

登录控制台后，先点击`Add network`创建一个网络

![image-20230928102318164](./使用zeroiter自建跨域局域网.assets/image-20230928102318164.png)

然后输入网络名称直接点击`Create Network`，成功后就可以看到你创建的网络控制台面板了。
![image-20230928102509954](./使用zeroiter自建跨域局域网.assets/image-20230928102509954.png)

接下来需要配置局域相关信息，点击上方的`Easy setup`，进入`network address`配置界面，这里就类似于本地机器局域网配置，第一个配置路由，第二个配置起始ip地址，第三个配置结束ip地址即可。

![image-20230928102920038](./使用zeroiter自建跨域局域网.assets/image-20230928102920038.png)

## 客户端配置

客户端主要分为`windows`，`Mac`，`Linux`和`Android`，本文只涉及到`Windows,Mac,Linux`。

### Windows

首先去`zerotier`官网下载一个`zerotier`客户端
将 `planet` 文件覆盖粘贴到 `C:\ProgramData\ZeroTier\One` 中(这个目录是个隐藏目录，需要运允许查看隐藏目录才行)
`Win+S` 搜索 服务
![invalid image(图片无法加载)](./使用zeroiter自建跨域局域网.assets/2022-08-14_212736_6452990.768645577804645.png)
找到`ZeroTier One`，并且重启服务
![invalid image(图片无法加载)](./使用zeroiter自建跨域局域网.assets/2022-08-14_212808_0024340.8742051046893466.png)
使用管理员身份打开`PowerShell`
执行如下命令，看到`join ok`字样就成功了

```bash
PS C:\Windows\system32> zerotier-cli.bat join 网络id(就是在网页里面创建的那个网络)
200 join OK
PS C:\Windows\system32>
```

登录管理后台可以看到有个个新的客户端，勾选Authorized就行
![invalid image(图片无法加载)](./使用zeroiter自建跨域局域网.assets/2022-08-14_213116_8760760.10632687026803656.png)
执行如下命令：

```bash
PS C:\Windows\system32> zerotier-cli.bat peers
200 peers
<ztaddr>   <ver>  <role> <lat> <link> <lastTX> <lastRX> <path>
fcbaeb9b6c 1.8.7  PLANET    52 DIRECT 16       8994     1.1.1.1/9993
fe92971aad 1.8.7  LEAF      14 DIRECT -1       4150     2.2.2.2/9993
PS C:\Windows\system32>
```

可以看到有一个 `PLANTET` 和 `LEAF` 角色，连接方式均为 `DIRECT`(直连)
到这里就加入网络成功了

### Linux

首先登录linux后安装客户端。

基于 Debian 和 RPM 的发行版（包括 Debian、Ubuntu、CentOS、RHEL、Fedora 等）通过脚本支持，该脚本添加正确的存储库并安装软件包。其他 Linux 发行版可能有自己的软件包。如果没有尝试 [从源代码构建和安装](https://github.com/zerotier/ZeroTierOne)。

如果您愿意依靠 SSL 来验证站点，可以通过以下方式完成一行安装：

```bash
curl -s https://install.zerotier.com | sudo bash
```

*如果您安装了 GPG，则可以使用更安全的选项：*

```bash
curl -s 'https://raw.githubusercontent.com/zerotier/ZeroTierOne/master/doc/contact%40zerotier.com.gpg' | gpg --import && \  
if z=$(curl -s 'https://install.zerotier.com/' | gpg); then echo "$z" | sudo bash; fi
```

安装成功后进入目录`/var/lib/zeroiter-one`，替换目录下的`planet`文件，就是前文中下载的文件。

重启`zeroiter-one`服务，并设置开启启动：

```bash
service zeroiter-one start
systemctl enable zeroiter-one
```

然后就可以让客户端加入网络`zeroiter-cli join 控制台你创建的Network的Id`,然后去控制台同意加入请求即可。

![image-20230928103823306](./使用zeroiter自建跨域局域网.assets/image-20230928103823306.png)

同意请求只要勾选`Authorized`即可。

`Linux`终端执行`zeroiter-cli peers` 既可以看到`planet`角色。

### Mac

下载安装包`https://download.zerotier.com/dist/ZeroTier%20One.pkg`。然后直接安装，打开。

替换`planet`文件，`mac`所在的文件目录是`/Library/Application Support/ZeroTier/One`，直接替换然后重启客户端即可。

复制创建好的`Network`的`ID`，然后可以客户端点击Join New Network`，同上客户端同意即可。

## 使用方式

### ssh已经加入的局域网机器

从控制台选择一个已经分配了ip的机器，然后复制ip，按着正常ssh方式即可。`ssh root@ip`，密码为访问的机器密码。
