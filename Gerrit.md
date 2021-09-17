# 开发环境搭建

## Linux服务器id

```c
ssh 172.16.83.28 和 172.16.80.24   账号密码都是jiwansu
hdu6相关在172.16.80.24上开发  其他的在83.28上
```

## 服务器映射

1、首先将两个服务器映射到windows的网络磁盘当中用来进行linux和windows之间的文件传输

![](.\Gerrit.assets\image-20210913145416168.png)

2、将文件路径输入

![](.\Gerrit.assets\image-20210913145517476.png)



## git环境配置搭建

### 1、根据开发环境判断是否需要实现安装git

```c
$ sudo apt-get update
$ sudo apt-get dist-upgrade
$ sudo apt-get install openssh-client
$ sudo apt-get install git-core
$ sudo apt-get install curl
```

### 2、生成公钥，用于系统认证

```c
$ ssh-keygen -t -rsa                 //一路回车到底
$ cd ~/.ssh                          //查看是否生成公私钥对（id_rsa.pub、id_rsa、know_hosts）
```

### 3、在$HOME/.shh目录下创建config文件

```c
$ touch ~/.ssh/config
```

### 4、将以下内容添加到config文件中

```c
$ touch ~/.ssh/config
$ chmod 644 ~/.ssh/config
$ vi ~/.ssh/config              //增加如下内容，注意修改自己的名字，若有新服务器添加，末端添加即可
host 172.16.8.9
Hostname 172.16.8.9
Port 29418
User 自己的名字
PubkeyAuthentication yes
IdentitiesOnly yes
PasswordAuthentication no
IdentityFile ~/.ssh/id_rsa
 
host 172.16.6.148			//有新的服务器就继续在下面添加
Hostname 172.16.6.148
Port 29418
User 自己的名字
PubkeyAuthentication yes
IdentitiesOnly yes
PasswordAuthentication no
IdentityFile ~/.ssh/id_rsa
```

### 5、将之前生成的公钥添加到Gerrit系统中

```c
cat ~/.ssh/id_rsa.pub		//注意*.pub后缀的公钥文件，选择公钥内容并拷贝
```

#### （1）登录Gerrit web页面

- http://172.16.8.9:8080/            # 基础组件Gerrit系统
- [http://172.16.7.105:8080](http://172.16.7.105:8080/)          # Rockchip Android平台Gerrit系统
- [http://172.16.6.148:8080](http://172.16.7.105:8080/)          # Hi3798C、QualComm定制Android平台Gerrit系统
- [http://172.16.1.12:8080](http://172.16.7.105:8080/)           # QualComm Android平台 Gerrit系统

#### （2）选择用户名，进入设置模式

![](.\Gerrit.assets\image-20210913151636989.png)

#### （3）选择ssh key

![](.\Gerrit.assets\image-20210913151723680.png)

#### （4）将公钥添加

![](.\Gerrit.assets\image-20210913151839826.png)

#### （5）查看开发服务器权限

```c
$ssh 172.16.8.9           //注意 这里的IP为对应Gerrit服务器IP，出现welcome 提示即表示成功。
```

### 6、下载开发环境仓库

```c
$ mkdir -p ~/bin #创建bin目录
$ cd ~/bin
$ git clone 172.16.8.9:sysdev/devenv          #下载sysdev/devenv仓库至$HOME/bin/devenv下
```

### 7、配置环境变量，能够直接使用repo & codereview工具

以下方式，我选择了第一种方式，因为下面方式中的repo工具有好多版本不知道应该移动哪一个，所以干脆将其路径放入环境变量中

```c
$ vi  ~/.profile        # 修改自定义文件，增加如下内容
 
if [ -d "$HOME/bin" ]; then
PATH="$HOME/bin:$HOME/bin/devenv/scripts:$PATH"
fi
 
$ source ~/.profile
```

或者

```c
$ cp devenv/scripts/repo  ~/bin/                   # 拷贝repo引导文件至$HOME/bin目录
$ cp devenv/scripts/codereview  ~/bin/             # 拷贝repo引导文件至$HOME/bin目录
$ vi  ~/.profile                                   # 修改自定义文件，增加如下内容
 
if [ -d "$HOME/bin" ]; then
PATH="$HOME/bin:$PATH"
fi
 
$ source ~/.profile
```

# 代码获取及更新

## 基础库权限申请

- 账号开通：由LTM或导师发邮件给CMO为新员工申请账号

   账号申请邮件格式

- 基础组件（172.16.8.9）仓库，线上申请，访问[cmo.kedacom.com](http://cmo.kedacom.com/)

- android代码（非基础库），线下申请，邮件格式如下，由项目负责人审核通过后cmo打开权限

- 发送：android项目负责人

  抄送：LTM，CMO

  标题：申请android_xx项目读权限，请审核

  邮件正文：

  - - android项目名称
    - android的repo名称
    - 申请原因：

1、以基础组件（172.16.8.9）仓库为例，我需要申请hdu6的代码，需要先进入CMO在公共信息里的系统部repo信息

![](.\Gerrit.assets\image-20210913164005181.png)

2、搜索相关代码譬如hdu6相关代码需要搜索rk3568（用的是这个的芯片）

![](.\Gerrit.assets\image-20210913164406630.png)

选择你需要的版本，注意看好服务器IP，如代码所在服务器之前没有添加过公钥，将会导致代码拉不下来

点击路径查看内容

![](.\Gerrit.assets\image-20210914091033945.png)

发现这个代码分在了两个库中存放，分别是一个基础数据库，一个私有的，基础数据库我们在CMO上申请就可以了，私有的需要邮件申请，接下来我们先申请基础数据库

3、申请权限

![](.\Gerrit.assets\image-20210913164607792.png)

3、选择工程

![](.\Gerrit.assets\image-20210913164736525.png)

红框部分都要进行选择，权限记得选择读写提交，审核人能找认识的就找认识的，点击提交等待审核批准

## 下载程序

### init

程序的权限申请成功之后，复制你所申请权限

```c
repo init -u 172.16.8.9:manifest/sysdevrepo.git -b sysdev_rockchip_v1.x -m rk356x.xml
```

运行该命令之后会在当前目录下创建一个.repo子目录

```c
.repo
|- manifests		//一个git库，包含default.xml文件，用于描述repo所管理的git库的信息
|- manifests.git	//manifest这个git库的实体，manifest/.git目录下的所有文件都会被链接到该目录
|- manifest.xml		//manifest/default.xml的一个软连接
|- repo    			//一个git库，包含repo运行的所有脚本
```

首先，在当前目录下创建了.repo子目录，后续所有的操作都将在.repo子目录下完成

然后，clone了两个git库，其中一个是**-u**参数指定的manifests，本地git库的名称是manifest.git;另一个是默认的repo，后面我们会看到这个URL也可以通过参数来指定;

接着，创建了manifest/.git目录，里面的所有文件都是到manifests.git这个目录的链接，这个是为了方便对manifests目录执行git命令，紧接着，就会将manifest切换到**-b**参数指定的分支;

最后，在.repo目录下，创建了一个软链接，链接到**-m**参数制定的清单文件，默认情况是manifests/default.xml。

这样，就完成了一个多git库的初始化，之后，就可以执行其他的repo命令了。

### sync

```c
$ repo sync
```

下载远程代码，并将本地代码更新到最新，这个过程被称为同步，类似于git pull

## 申请私有数据库

==注意==：当代码被两个以上库包含时，需要获得所有库的权限才能将项目的完整代码下载下来

![](.\Gerrit.assets\image-20210914091253875.png)

### 添加公钥到Gerrit

根据这张图我们可以知道私有数据库的IP是172.168.7.205，进入这个IP的Gerrit添加公钥

### 配置config

在.ssh/config的文件里配置添加相关信息

```c
host 172.16.7.105
Hostname 172.16.7.105
Port 29418
User jiwansu
PubkeyAuthentication yes
IdentitiesOnly yes
PasswordAuthentication no
IdentityFile ~/.ssh/id_rsa
```

### 测试服务器链接

```c
ssh 172.16.7.205		//出现welcome表示链接成功
```

### 通过邮箱申请权限

申请格式如下：

发送：android项目负责人

抄送：LTM，CMO

标题：申请android_xx项目读权限，请审核

邮件正文：

- - android项目名称
  - android的repo名称
  - 申请原因：

![](.\Gerrit.assets\image-20210914102646519.png)

### 重新下载

授权成功后将.repo删除重新repo init

```c
$ rm -rf .repo
$ repo init -u 172.16.8.9:manifest/sysdevrepo.git -b sysdev_rockchip_v1.x -m rk356x.xml
```

## 移动分支

当我们将代码下载下来之后，记得查看分支是否和你在库里看到的分支吻合

```c
$ git brach -a
```

如果不吻合，需要将分支移动到和下载分支相同的分支上去

```c
$ git checkout -t brach_name
```



## 代码提交

举个例子：现在有个需求，需要在hdu6的内核中将cma的设备树上的内存从208MB修改为512MB

1、移动分支

按照需求修改代码之前，注意，分支信息一定要对应上

```c
$ git branch -a
$ git checkout -t sysdev_kdv_rk356x_v1.x		//移动到这个名字的分支    
```

2、修改代码

```c
 $ vi	/arch/arm64/boot/dts/rockchip/rk3568-nvr-hdu6.dtsi

     linux_cma: linux-cma {
         compatible = "shared-dma-pool";
         reusable;
         //size = <0x0 0xd000000>;	将原来的208MB修改为下面的512MB
         size = <0x0 0x20000000>;
         alignment = <0x0 0x1000>;
         linux,cma-default;
     };
```

3、追踪修改后的代码

```c
$ git add /arch/arm64/boot/dts/rockchip/rk3568-nvr-hdu6.dtsi
```

4、创建Readmine任务

需要在Readmine的web段添加一个关于这次修改的任务，为之后git commit 添加注释可以写Readmine编号

5、上传本地仓库

```c
$ git commit -s
//添加以下注释
    Redmine#695719 hdu6:modify for cma.		//这里的Readmine变化一定要和你之前web上创建的相同
    
            1.extend reserve to 512M for cma.

```

==注意==：如果上传本地仓库步骤有地方错了可以用git reset HEAD^	取消已经暂存的消息

6、使用/usr/bin/codereview脚本进行上传到Gerrit

```c
$ /usr/bin/codereview
```

7、进入Gerrit网站，点击YOUR—Changes

![](.\Gerrit.assets\image-20210916173811642.png)

8、点击选中之前提交的代码

![](.\Gerrit.assets\image-20210916173857372.png)

9、添加审核人

![](.\Gerrit.assets\image-20210916173259692.png)

10、自评按照以下方式填写，添加审核人，填写自评按照以下格式填写，添加+2，+1

![](.\Gerrit.assets\image-20210916173456353.png)





