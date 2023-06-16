---
tags: CentOS Python HyperLPR3
categories: linux
---

需求：在一台CentOS 7服务器安装HyperLPR3（一个开源中文车牌识别框架），在线安装和离线安装都需要

<!--more-->

实现：使用rpm包安装依赖，源码编译安装python3，使用whl安装HyperLPR3

>   该流程已弃用，仅供参考，新方案见本文底部Attach章节
>
>   降低Python3版本，从而可以直接通过rpm包安装python3

### 实现方式

#### 1.在线安装

```sh
# 更新yum
yum update -y
# 安装工具
yum groupinstall 'Development Tools' -y
yum -y install openssl-devel libffi-devel bzip2-devel mesa-libGL
# 安装openssl-1.1.1
wget https://www.openssl.org/source/openssl-1.1.1q.tar.gz --no-check-certificate
tar -zxvf openssl-1.1.1q.tar.gz
cd openssl-1.1.1q
./config --prefix=/usr/local/openssl-1.1.1 && make -j 4 && make install
# 安装python
wget https://www.python.org/ftp/python/3.10.0/Python-3.10.0.tgz
tar -zxvf Python-3.10.0.tgz
cd Python-3.10.0
./configure --enable-optimizations --with-openssl=/usr/local/openssl-1.1.1 --with-openssl-rpath=auto --prefix=/usr/local/python3.10 && make -j 4 && make install
ln -s /usr/local/python3.10/bin/python3.10 /usr/bin/python3
# 更新pip
python3 -m pip install --upgrade pip
# 安装hyperlpr3
python3 -m pip install hyperlpr3
# 建立链接
ln -s /usr/local/python3.10/bin/lpr3 /usr/bin/lpr3
# 启动，并指定端口
nohup lpr3 rest --port 8715 --host 0.0.0.0 >lpr3.log 2>&1 &
```

#### 2.离线安装

写成了脚本，方便执行

```sh
#!/bin/bash

# 获取脚本所在目录
cd $(dirname $0)
dir=$(pwd)
echo "脚本所在目录: $dir"
echo "准备开始安装hyperlpr3..."


# 安装基础工具，用于后续编译
echo "============开始安装基础工具============"
cd $dir/lib/rpm
rpm -Uvh *.rpm --nodeps --force
echo "============基础工具安装完毕============"

# 安装OpenSSL-1.1.1
# 供python3.10使用
echo "============开始安装OpenSSL-1.1.1============"
cd $dir/lib
tar -zxvf openssl-1.1.1q.tar.gz
cd openssl-1.1.1q
./config --prefix=/usr/local/openssl-1.1.1 && make -j 4 && make install
echo "============OpenSSL-1.1.1安装完毕============"

# 安装python3.10
# 经测试，自带的python2.7和使用yum直接安装的python3.6无法正常安装hyperlpr3
echo "============开始安装python3.10============"
cd $dir/lib
tar -zxvf Python-3.10.0.tgz
cd Python-3.10.0
./configure --prefix=/usr/local/python3.10 --enable-optimizations --with-openssl=/usr/local/openssl-1.1.1 --with-openssl-rpath=auto && make -j 4 && make install
ln -s /usr/local/python3.10/bin/python3.10 /usr/bin/python3
echo "============python3.10安装完毕============"

# 更新pip
# 老版本pip无法安装对应依赖
echo "============开始更新pip============"
cd $dir/lib/pip
python3 -m pip install pip-23.1.2-py3-none-any.whl
echo "============pip更新完毕============"

# 安装hyperlpr3
echo "============开始安装hyperlpr3============"
cd $dir/lib/lpr
python3 -m pip install -r requirement.txt
ln -s /usr/local/python3.10/bin/lpr3 /usr/bin/lpr3
echo "============hyperlpr3安装完毕============"

echo "脚本执行完毕"
echo "执行以下命令启动lpr rest接口:"
echo "nohup lpr3 rest --port 8715 --host 0.0.0.0 >lpr3.log 2>&1 &"
```

### 流程

#### 1.自测

>   环境：Debian, Python3.10

直接安装并使用，正常，命令为

```sh
python3 -m pip install hyperlpr3
lpr3 rest --port 8715 --host 0.0.0.0
```

#### 2.服务器

>   环境：CentOS 7, Python2.7

-   尝试使用Python2.7安装hyperlpr3，失败

-   使用yum安装Python3，发现版本为Python3.6，尝试使用Python3.6安装hyperlpr3，莫名卡住，失败

-   尝试通过编译源码安装Python3.10：
    -   编译报错，安装'Development Tools', openssl-devel, libffi-devel, bzip2-devel, mesa-libGL解决
    -   编译过程显示警告，pip安装hyperlpr3报错，lpr使用rest命令报错，系统的openssl版本过低，安装openssl-1.1.1解决
    -   为不干扰原有Python2.7，编译前进行相关设置，并手动建立软链接
    -   成功

### 命令解释

#### 1.在线安装

1.   更新yum

     ```sh
     yum update -y
     ```

2.   安装工具

     ```sh
     # 安装Development Tools组的所有软件包，主要是需要其中的gcc，用于编译
     yum groupinstall 'Development Tools' -y
     # 分别安装这四个软件包，也是用于编译
     yum -y install openssl-devel libffi-devel bzip2-devel mesa-libGL
     ```

3.   安装openssl-1.1.1

     ```sh
     # 使用wget下载源码，使用--no-check-certificate指定不验证证书有效性
     wget https://www.openssl.org/source/openssl-1.1.1q.tar.gz --no-check-certificate
     # 解压
     tar -zxvf openssl-1.1.1q.tar.gz
     cd openssl-1.1.1q
     # 该命令分为三个部分
     # ./config监测系统环境，生成用于编译的配置，使用--prefix指定安装的位置
     # make编译源码，-j 4指定使用4个并行线程来加速编译过程
     # make install安装
     ./config --prefix=/usr/local/openssl-1.1.1 && make -j 4 && make install
     ```

4.   安装python

     ```sh
     # 下载源码
     wget https://www.python.org/ftp/python/3.10.0/Python-3.10.0.tgz
     # 解压
     tar -zxvf Python-3.10.0.tgz
     cd Python-3.10.0
     # --enable-optimizations 开启优化，使python运行更快
     # --with-openssl 指定使用的openssl，不使用默认的openssl
     # --with-openssl-rpath=auto 自动设置openssl库的运行时路径
     # --prefix 指定安装路径
     ./configure --enable-optimizations --with-openssl=/usr/local/openssl-1.1.1 --with-openssl-rpath=auto --prefix=/usr/local/python3.10 && make -j 4 && make install
     # 建立软链接
     ln -s /usr/local/python3.10/bin/python3.10 /usr/bin/python3
     ```

5.   安装hyperlpr3

     ```sh
     # 更新pip
     python3 -m pip install --upgrade pip
     # 安装hyperlpr3
     python3 -m pip install hyperlpr3
     # 建立链接
     ln -s /usr/local/python3.10/bin/lpr3 /usr/bin/lpr3
     ```

#### 2.离线安装

>   先从联网的机器上下载对应的软件包，然后再进行离线安装
>
>   注意：下载rpm包和whl包的机器，需要和离线服务器环境一致

##### 下载离线软件包

1.   下载rpm包

     ```sh
     # 安装工具，主要是yumdownloader，用于下载rpm包
     yum -y install yum-utils
     # 下载'Development Tools'群组的rpm包
     yumdownloader --resolve --destdir=lpr/lib/rpm "@Development Tools"
     # 下载几个相关依赖的rpm包
     yumdownloader --resolve --destdir=lpr/lib/rpm openssl-devel libffi-devel bzip2-devel mesa-libGL
     # 有的服务器连perl都无，也需提供
     yumdownloader --resolve --destdir=tmp perl-core perl perl-libs perl-Carp perl-constant perl-Encode perl-Exporter perl-File-Path
     ```

2.   下载whl包

     ```sh
     # 下载最新版本的pip
     cd lpr/lib/pip
     pip download pip
     # 下载hyperlpr3及其依赖的包
     # 将命令输出的所有包名记录到requirement.txt文件，用于之后进行安装
     # 命令输出：Saved ...\hyperlpr3-0.1.3-py3-none-any.whl
     # 文件中添加行：hyperlpr3-0.1.3-py3-none-any.whl
     cd lpr/lib/lpr
     pip download hyperlpr3
     ```

3.   下载源码

     ```sh
     # 直接使用wget下载源码
     wget https://www.openssl.org/source/openssl-1.1.1q.tar.gz --no-check-certificate
     wget https://www.python.org/ftp/python/3.10.0/Python-3.10.0.tgz
     ```

##### 进行离线安装

文件夹内结构

-   lib - 离线软件包
    -   lpr - hyperlpr3及其依赖的whl包
    -   pip - 最新版本pip的whl包
    -   rpm - 所有工具的rpm包
    -   openssl-1.1.1q.tar.gz - openssl-1.1.1的源码
    -   Python-3.10.0.tgz - python3.10的源码
-   install.sh - 安装脚本

脚本使用方式

```sh
sh install.sh
```

##### 脚本内容

```sh
#!/bin/bash

# 获取脚本所在目录
cd $(dirname $0)
dir=$(pwd)
echo "脚本所在目录: $dir"
echo "准备开始安装hyperlpr3..."


# 安装基础工具，用于后续编译
echo "============开始安装基础工具============"
cd $dir/lib/rpm
# -U 如软件包已安装，则替换为新的版本
# -v 显示安装过程中的详细信息
# -h 用#号表示安装进度
# *.rpm 所有软件包
# --nodeps 不检查软件包之间的依赖关系，强制安装
# --force 强制覆盖已存在的文件或软件包
rpm -Uvh *.rpm --nodeps --force
echo "============基础工具安装完毕============"

# 安装OpenSSL-1.1.1
# 供python3.10使用
echo "============开始安装OpenSSL-1.1.1============"
cd $dir/lib
tar -zxvf openssl-1.1.1q.tar.gz
cd openssl-1.1.1q
# 编译安装方式同上方在线安装
./config --prefix=/usr/local/openssl-1.1.1 && make -j 4 && make install
echo "============OpenSSL-1.1.1安装完毕============"

# 安装python3.10
# 经测试，自带的python2.7和使用yum直接安装的python3.6无法正常安装hyperlpr3
echo "============开始安装python3.10============"
cd $dir/lib
tar -zxvf Python-3.10.0.tgz
cd Python-3.10.0
# 编译安装方式同上方在线安装
./configure --prefix=/usr/local/python3.10 --enable-optimizations --with-openssl=/usr/local/openssl-1.1.1 --with-openssl-rpath=auto && make -j 4 && make install
ln -s /usr/local/python3.10/bin/python3.10 /usr/bin/python3
echo "============python3.10安装完毕============"

# 更新pip
# 老版本pip无法安装对应依赖
echo "============开始更新pip============"
cd $dir/lib/pip
# 指定whl包，通过whl包离线安装
python3 -m pip install pip-23.1.2-py3-none-any.whl
echo "============pip更新完毕============"

# 安装hyperlpr3
echo "============开始安装hyperlpr3============"
cd $dir/lib/lpr
# -r 指定配置文件，配置文件里列出了要安装的软件包和版本，pip会根据配置来安装指定的软件包
python3 -m pip install -r requirement.txt
ln -s /usr/local/python3.10/bin/lpr3 /usr/bin/lpr3
echo "============hyperlpr3安装完毕============"

echo "脚本执行完毕"
echo "执行以下命令启动lpr rest接口:"
echo "nohup lpr3 rest --port 8715 --host 0.0.0.0 >lpr3.log 2>&1 &"
```

### Attach

在新的服务器上测试，报错，缺少perl，安装，重新测试，编译报错...

尝试放弃python3.10，转回去解决python3.6安装opencv失败的问题

找到解决方案，原来只是pip版本太低

以下为新方案

#### 下载依赖

```sh
# python3.6
yumdownload --resolve python3
# 新版本pip
# 注意:
# python3具体版本要相同
# pip要是最新版本，更新语句 python3 -m pip install --upgrade pip
python3 -m pip download pip
# hyperlpr3
python3 -m pip download hyperlpr3
# 发现最新的opencv-python没有py36的whl
# 手动下载老版本的，并替换掉上面下载的opencv-python源码
python3 -m pip download opencv-python==4.6.0.66
```

#### 安装脚本

```sh
#!/bin/bash

# 获取脚本所在目录
cd $(dirname $0)
dir=$(pwd)
echo "脚本所在目录: $dir"

echo "============开始安装hyperlpr3============"
cd $dir/lib/py
rpm -Uvh *.rpm --nodeps --force
cd $dir/lib/pip
python3 -m pip install pip-21.3.1-py3-none-any.whl
cd $dir/lib/lpr
python3 -m pip install -r requirement.txt
echo "============hyperlpr3安装完毕============"

echo "脚本执行完毕"
echo "执行以下命令启动lpr rest接口:"
echo "nohup lpr3 rest --port 8715 --host 0.0.0.0 >lpr3.log 2>&1 &"
```
