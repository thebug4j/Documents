
### 离线yum源制作 ###
- 1、在一台有网服务器上下载需要的安装包和依赖(例：下载tcpdump、telnet)

```
yum install  --downloadonly --downloaddir=/mnt/localrepo tcpdump telnet
```

- 2、安装生成yum源的工具

```
yum install createrepo -y
```

- 3、配置本地仓库信息  

###### 方法1：vi手动输入
```
# vim /etc/yum.repos.d/local.repo
[localyum]
name=my local repository
baseurl=file:///mnt/localrepo
gpgcheck=0
enabled=1
```
###### 方法2：一键创建配置信息

```
echo -e '# vim /etc/yum.repos.d/local.repo\n[localyum]\nname=my local repository\nbaseurl=file:///mnt/localrepo\ngpgcheck=0\nenabled=1' >/etc/yum.repos.d/local.repo
```

- 4、使用createrepo命令生成repodata信息

```
createrepo /mnt/localrepo
```

- 5、检查yum源是否成功

```
yum repoinfo localyum
```


### 离线yum源使用 ###
>- 1、将/etc/yum.repos.d/local.repo 传到离线服务器/etc/yum.repos.d/目录下
>- 2、将yum源【==cd /mnt&&tar -zcvf localrepo.tar.gz localrepo==】压缩后上传到离线服务器的/mnt/ 下解压
>- 3、yum安装验证

### 离线yum源更新 ###


```
对本地仓库进行更新
下载一个新的rpm软件包到本地仓库，此时我们使用yum repoinfo localyum查看会发现软件包的数量并没有增加，我们安装新增的软件包也会提示，找不到次软件包的现象，可以按照下述步骤，更新仓库信息。

查看旧的软件包总数 yum repoinfo localyum | grep pkgs
更新本地仓库 createrepo --update /mnt/localrepo/
清除所有缓存 yum clean all
查看新的软件包总数 yum repoinfo localyum | grep pkgs
如果软件包的数量增加，说明仓库更新成功。
```

