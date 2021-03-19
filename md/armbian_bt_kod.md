# Armbian搭建可道云kodbox

## 安装armbian

安装Armbian可参考恩山大佬的帖子    
https://www.right.com.cn/forum/thread-510423-1-1.html


## 安装宝塔

1. 安装宝塔面板

一条命令搞定    

`wget https://gitee.com/ryuukarin/shell_scripts/raw/master/shell/armbian_bt.sh && bash armbian_bt.sh`    

出现下图红框绿色字体，宝塔面板安装成功，并给出面板的ip地址、用户名及密码，用于web登陆。    
输入`bt`命令，可以修改面板配置    

![bt安装成功](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210318_001.png "在这里输入图片标题")    

web登陆上面提供的内网面板地址，输入用户名、密码进行登陆（面板地址应该一致，一张是之前的截图）    

![bt登陆界面](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210318_002.png "在这里输入图片标题")    



2. 搭建LAMP环境

登陆塔成功后，选择安装环境LAMP，急速安装，需要花很长时间（5小时左右），自己安排时间，等待环境安装完成

![安装环境](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210318_003.png "在这里输入图片标题")

安装完成（可以参考我安装所花的时间，算了下大概5.2个小时，可以晚上安装睡一觉）    

![LAMP完成](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210318_004.png "在这里输入图片标题")

*如果后面步骤的一键部署出现如下错误信息，是因为Apache没有启动成功（armbian的openssl版本在官方源中只有1.1.0，但宝塔已经帮你装好了openssl）*    

>httpd: Syntax error on line 130 of /www/server/apache/conf/httpd.conf: Cannot load modules/mod_ssl.so into server: /usr/lib/aarch64-linux-gnu/libssl.so.1.1: version `OPENSSL_1_1_1′ not found (required by /www/server/apache/modules/mod_ssl.so)    

*确认Apache服务状态*

![查看Apache状态](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210318_006.png "在这里输入图片标题")

*所以执行以下命令把opnessl替换即可解决，参考http://www.apod.cc/index.php/post/197.html*
```
mv /usr/bin/openssl /usr/bin/openssl—old
ln -s /usr/local/openssl111/bin/openssl  /usr/bin/openssl
mv /usr/include/openssl /usr/include/openssl—old
ln -s /usr/local/openssl111/include/openssl   /usr/include/openssl
mv /usr/lib/aarch64-linux-gnu/libssl.so.1.1 /usr/lib/aarch64-linux-gnu/libssl.so.1.1—old
ln -s /usr/local/openssl111/lib/libssl.so.1.1 /usr/lib/aarch64-linux-gnu/libssl.so.1.1
mv /usr/lib/aarch64-linux-gnu/libcrypto.so.1.1 /usr/lib/aarch64-linux-gnu/libcrypto.so.1.1—old
ln -s /usr/local/openssl111/lib/libcrypto.so.1.1 /usr/lib/aarch64-linux-gnu/libcrypto.so.1.1
```
*上面命令完成后，到宝塔面板重启Apache服务*

![重启Apache](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210318_005.png "在这里输入图片标题")



##  部署可道云

1. 一键部署可道云

宝塔面板找到**软件商店 ---> 一键部署 ---> 可道云KODBOX（一键部署）**

![部署可道云](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210318_007.png "在这里输入图片标题")

在可道云部署好之后，需要设置域名，数据库帐号和密码等信息（记住，后面登陆可道云会用到），点提交

![可道云数据库配置](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210318_008.png "在这里输入图片标题")

提交成功后，如下界面，点击访问站点（或者直接输入主机ip），进入可道云登陆界面

![可道云配置成功](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210318_009.png "在这里输入图片标题")


2. 登录可道云

进入可道云登录界面，输入账号密码登陆（就是上一步设置的数据库名和密码）

![登陆可道云](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210318_010.png "在这里输入图片标题")

登陆成功后，进入可道云私人网盘界面，如下图

![进入可道云](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210318_011.png "在这里输入图片标题")


3. 挂载外接磁盘

默认的可道云存储在`/www/wwwroot/数据库名/`目录下（也就是可道云网盘的根目录，之前设置的/www/wwwroot/kodbox/）。但是，我用的是斐讯N1、玩客云这样的小主机，存储也就只有8GB容量，系统和环境已经占了一大部分了，要是当网盘存储，那肯定是不行的。所以，需要外接一块硬盘挂载，当作云盘的存储。具体配置如下：

- 把硬盘接入主机

- 用putty远程连接登陆到主机armbian（root帐号，root密码）

![putty链接主机](https://gitee.com/ryuukarin/shell_scripts/raw/master/img/210319_001.png "在这里输入图片标题")

- 查看硬盘

输入命令`lsblk`，找到要挂载的硬盘

```
root@aml:~# lsblk
NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda            8:0    1  29.3G  0 disk
mmcblk1      179:0    0   7.3G  0 disk
|-mmcblk1p1  179:1    0   122M  0 part /boot
`-mmcblk1p2  179:2    0   6.5G  0 part /
mmcblk1boot0 179:32   0     4M  1 disk
mmcblk1boot1 179:64   0     4M  1 disk
zram0        253:0    0    50M  0 disk /var/log
zram1        253:1    0 919.2M  0 disk [SWAP]
```
*sda就是需要挂载的磁盘29.3GB（32GB的U盘）*

- 格式化磁盘

输入命令`mkfs.ext4 /dev/sda`，把硬盘sda格式化成ext4格式

```
root@aml:~# mkfs.ext4 /dev/sda
mke2fs 1.43.4 (31-Jan-2017)
Creating filesystem with 7682304 4k blocks and 1921360 inodes
Filesystem UUID: d03d4044-b17f-42a0-8dd8-8071aa2ccc6a
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

```

- 新建挂载目录

输入命令`cd /www/wwwroor/数据库名/`（我设置的数据库位置`cd /www/wwwroor/kodbox/`），进入可道云的根目录下    
继续输入命令`ls`，查看目录内容，data目录为可道云数据库默认存储数据的位置    
再输入命令`mkdir NAS`，新建一个目录，用于挂载硬盘（目录名随意，此处目录为NAS）  

```
root@aml:~# cd /www/wwwroot/kodbox/
root@aml:/www/wwwroot/kodbox# ls
app  config  data  index.php  plugins  static
root@aml:/www/wwwroot/kodbox# mkdir NAS
```

- 挂载磁盘

输入命令`mount /dev/sda NAS/`，挂载磁盘sda到/www/wwwroot/数据库名/NAS/下（此处为/www/wwwroot/kodbox/NAS/）    
输入命令`lsblk`，查看硬盘挂载是否成功，sda的MOUNTPOINT挂载点是否显示/www/wwwroot/kodbox/NAS/
输入命令`chown -hR www NAS/`，更改NAS目录的的所属用户为www    
输入命令`chgrp -hR www NAS/`，更改NAS目录的的所属组为www    

```
root@aml:/www/wwwroot/kodbox# mount /dev/sda ./NAS/
root@aml:/www/wwwroot/kodbox# lsblk
NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda            8:0    1  29.3G  0 disk /www/wwwroot/kodbox/NAS
mmcblk1      179:0    0   7.3G  0 disk
|-mmcblk1p1  179:1    0   122M  0 part /boot
`-mmcblk1p2  179:2    0   6.5G  0 part /
mmcblk1boot0 179:32   0     4M  1 disk
mmcblk1boot1 179:64   0     4M  1 disk
zram0        253:0    0    50M  0 disk /var/log
zram1        253:1    0 919.2M  0 disk [SWAP]
root@aml:/www/wwwroot/kodbox# chown -hR www NAS/
root@aml:/www/wwwroot/kodbox# chgrp -hR www NAS/
```

- 设置开机自动挂载硬盘

输入命令`blkid /dev/sda`，查看硬盘的信息UUID和TYPE    
再输入命令`nano /etc/fstab`，编辑/etc/fstab文件，最后一行加入如下信息：    
>UUID=d03d4044-b17f-42a0-8dd8-8071aa2ccc6a       /www/wwwroot/kodbox/NAS ext4    defaults        0 0    
Ctrl+O写入文件；Ctrl+O退出文件    
以后当主机断电重启时，这块硬盘就在启动时自动挂载到/www/wwwroot/kodbox/NAS目录下     

```
root@aml:/www/wwwroot/kodbox# blkid /dev/sda
/dev/sda: UUID="d03d4044-b17f-42a0-8dd8-8071aa2ccc6a" TYPE="ext4"
root@aml:/www/wwwroot/kodbox# nano /etc/fstab

#/var/swap none swap sw 0 0
#/dev/root      /               auto            noatime,errors=remount-ro       0 1
#proc           /proc           proc            defaults                                0 0

/dev/root       /               ext4            defaults,noatime,errors=remount-ro      0 1
tmpfs           /tmp            tmpfs           defaults,nosuid                         0 0
LABEL=BOOT_EMMC /boot           vfat            defaults                                0 2
UUID=d03d4044-b17f-42a0-8dd8-8071aa2ccc6a       /www/wwwroot/kodbox/NAS ext4    defaults        0 0
```

4. web界面设置

- 进入可道云web界面,点击左下角的图标弹出设置菜单,选择**后台管理**
- 新界面点击**存储文件--->存储管理**
- 在右侧界面点击**新增**进入新设置界面
- 如下设置
- 存储类型:**本地**(FTP应该也可以,没测试)
- 名称:**随意命名**
- 空间大小:**不要超过磁盘大小**
- 存储目录:**点击右侧文件夹图标,选择之前新建的NAS文件夹**
- 设置好之后保存,回到之前界面,就多了一个NAS的磁盘
- 点击NAS磁盘上**文件管理**右侧的下拉三角,选择设为默认,跳出确认窗口选确定

以上,基本的设置酒完成了.

后面还有新建账户,权限,外网访问(需要公网IP或内网穿透),以后再慢慢研究




