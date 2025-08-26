---
title: 'HyperLPR3 使用'
date: 2024-08-26
authors: neko-tail
tags: 
  - tech
---

[HyperLPR](https://github.com/szad670401/HyperLPR) 是一个中文车牌识别工具，[HyperLPR3](https://github.com/szad670401/HyperLPR/releases/tag/v3.0) 为其当前最新版本，提供命令行、WebApi、python 等多种方式调用，本文记录之前安装和使用 HyperLPR3 的过程

<!-- truncate -->

HyperLPR3 的命令行工具为 lpr3，python 示例中导入的依赖包别名也为 lpr3，因此后文都将 HyperLPR3 简称为 lpr3，HyperLPR 简称为 lpr

在写这篇文章的一年之前，我在服务器上部署了 lpr3 用来测试车牌识别，几个月前，又用到了一次，这里将两次使用过程中的文档汇总，记录如下

## 安装

### 在线安装

最简单的方式为直接通过 pip 安装

```sh
python3 -m pip install hyperlpr3
```

### 离线安装方法一

在没有网络，且没有 python3 环境的 CentOS 7 服务器上部署，编译安装 python3，并最终安装 lpr3，操作较为复杂，建议参考 [离线安装方法二](#离线安装方法二)

1. 准备一台相同系统的联网服务器，用于下载相关依赖
2. 在联网服务器上下载需要的依赖
    - 使用 yumdownloader 下载以下 rpm 包
        - \'Development Tools\' 群组的 rpm 包
        - openssl-devel libffi-devel bzip2-devel mesa-libGL 包
        - perl-core perl perl-libs perl-Carp perl-constant perl-Encode perl-Exporter perl-File-Path 等 perl 相关包（仅待部署服务器无 perl 的情况下需要）

        ```sh
        # 安装工具，主要是 yumdownloader，用于下载 rpm 包
        yum -y install yum-utils
        # 下载 'Development Tools' 群组的 rpm 包
        yumdownloader --resolve --destdir=lpr/lib/rpm "@Development Tools"
        # 下载几个相关依赖的 rpm 包
        yumdownloader --resolve --destdir=lpr/lib/rpm openssl-devel   libffi-devel bzip2-devel mesa-libGL
        # 有的服务器连 perl 都无，也需提供
        yumdownloader --resolve --destdir=tmp perl-core perl perl-libs   perl-Carp perl-constant perl-Encode perl-Exporter perl-File-Path
        ```

    - 使用 wget 下载 python3 相关源码（以 python3.10 为例）

        ```sh
        # openssl-1.1.1（编译安装 python3 需要）
        wget https://www.openssl.org/source/openssl-1.1.1q.tar.gz
        # python3
        wget https://www.python.org/ftp/python/3.10.0/Python-3.10.0.tgz
        ```

    - 下载最新版本的 pip whl 包

        ```sh
        # 更新 pip
        python3 -m pip install --upgrade pip
        # 下载 pip
        python3 -m pip download pip
        ```

    - 下载 lpr3 及其依赖的 whl 包

        ```sh
        python3 -m pip download hyperlpr3
        ```

        将命令输出的所有包名记录到 `requirement.txt` 文件，用于之后进行安装，格式如：

        ```text
        hyperlpr3-0.1.3-py3-none-any.whl
        fastapi-0.83.0-py3-none-any.whl
        starlette-0.19.1-py3-none-any.whl
        ...
        ```

3. 在待部署服务器上编译安装 python3，并安装 lpr3
    1. 安装所有之前下载的 rpm 包

         ```sh
         rpm -Uvh *.rpm --nodeps --force
         ```

    2. 编译安装 openssl-1.1.1

        ```sh
        tar -zxvf openssl-1.1.1q.tar.gz
        cd openssl-1.1.1q
        ./config --prefix=/usr/local/openssl-1.1.1 && make -j 4 && make install
        ```

    3. 编译安装 python3

        ```sh
        tar -zxvf Python-3.10.0.tgz
        cd Python-3.10.0
        ./configure --prefix=/usr/local/python3.10 --enable-optimizations --with-openssl=/usr/local/openssl-1.1.1 --with-openssl-rpath=auto && make -j 4 && make install
        ln -s /usr/local/python3.10/bin/python3.10 /usr/bin/python3
        ```

    4. 更新 pip（否则无法安装 lpr3 依赖）

        ```sh
        python3 -m pip install pip-23.1.2-py3-none-any.whl
        ```

    5. 安装 lpr3

        ```sh
        python3 -m pip install -r requirement.txt
        ln -s /usr/local/python3.10/bin/lpr3 /usr/bin/lpr3
        ```

### 离线安装方法二

之前的离线安装方式过于复杂，且要编译安装 python3，会有很多编译上的问题，尝试了一下，可以使用 CentOS 已有的 python3.6 rpm 包来安装 python3，并执行一些处理来安装 lpr3

1. 准备一台相同系统的联网服务器，用于下载相关依赖
2. 在联网服务器上下载依赖
    - 下载 python3.6 rpm 包

        ```sh
        yumdownload --resolve python3
        ```

    - 下载最新版本的 pip whl 包

        ```sh
        # 更新 pip
        python3 -m pip install --upgrade pip
        # 下载 pip
        python3 -m pip download pip
        ```

    - 下载 lpr3 及其依赖的 whl 包

        ```sh
        python3 -m pip download hyperlpr3
        ```

        最新的 opencv-python 没有 py36 的 whl，这里手动下载老版本的，并替换掉上面下载的 opencv-python

        ```sh
        python3 -m pip download opencv-python==4.6.0.66
        ```

3. 在待部署服务器上安装
    1. 安装 python3.6

        ```sh
        rpm -Uvh *.rpm --nodeps --force
        ```

    2. 安装 pip

        ```sh
        python3 -m pip install pip-21.3.1-py3-none-any.whl
        ```

    3. 安装 lpr3

        ```sh
        python3 -m pip install -r requirement.txt
        ```

## 使用

### 命令行使用

命令行工具 lpr3 有两个命令 `rest` 和 `sample`，`rest` 用于开启 WebApi 服务，`sample` 则用于在命令行识别车牌

```sh
Usage: lpr3 [OPTIONS] COMMAND [ARGS]...

Options:
  -h, --help  Show this message and exit.

Commands:
  rest    Exec HyperLPR3 WebApi Server.
  sample  Exec HyperLPR3 Test Sample.
```

对于 `sample` 命令

- 用 `-src` 指定图片，参数为 url 地址或本地文件路径
- 用 `-det` 指定检测等级，有 `low` 和 `high` 两种，没有找到相关的说明，但经测试，`low` 能返回置信度更低的识别结果，在原始图片角度较大或较模糊的情况下，仍能给出识别结果

```sh
Usage: lpr3 sample [OPTIONS]

  Exec HyperLPR3 Test Sample.

Options:
  -src, --src TEXT
  -det, --det [low|high]
  -h, --help              Show this message and exit.
```

执行命令打印日志格式为 `[车牌类型]车牌号码 置信度 车牌box`，如：

```log
2024-08-26 14:01:09.052 | INFO     | hyperlpr3.command.sample:sample:70 - 共检测到车牌: 1       
2024-08-26 14:01:09.058 | SUCCESS  | hyperlpr3.command.sample:sample:73 - [绿牌新能源]XXXXXXXX 0.9979625940322876 [145, 1140, 384, 1221]
```

### WebApi

lpr3 提供了 `rest` 命令来启动一个 WebApi 服务

- 文档示例中使用 8715 作为服务端口
- `-workers` 参数没有用过，也未在文档中找到其作用

```sh
Usage: lpr3 rest [OPTIONS]

  Exec HyperLPR3 WebApi Server.

Options:
  -host, --host TEXT
  -port, --port INTEGER
  -workers, --workers INTEGER
  -h, --help                   Show this message and exit.
```

开启服务后，可通过 [http://localhost:8715/api/v1/docs](http://localhost:8715/api/v1/docs)（若指定端口为 8751）查看 SwaggerUI 的接口文档

使用 [http://localhost:8715/api/v1/rec](http://localhost:8715/api/v1/rec) 接口进行车牌识别，提供单张图片，返回结果格式如下（`plate_list` 中的车牌可能为零个或多个）

```json
{
    "result": {
        "plate_list": [
            {
                "code": "XXXXXXXX",
                "conf": 0.9979625940322876,
                "plate_type": "绿牌新能源",
                "box": [
                    145,
                    1140,
                    384,
                    1221
                ]
            }
        ]
    },
    "code": 5000,
    "msg": "请求成功"
}
```

但 WebApi 无法指定 det，且貌似默认为 high，如果需要使用 low 的话，可以转为使用 python 调用

### python 调用

直接参考文档中给出的示例：

```python
# 导入 opencv 库
import cv2
# 导入依赖包
import hyperlpr3 as lpr3

# 实例化识别对象
catcher = lpr3.LicensePlateCatcher()
# 读取图片
image = cv2.imread("images/test_img.jpg")
# 识别结果
print(catcher(image))
```

`LicensePlateCatcher` 可传入一些参数，如使用 `detect_level` 指定检测等级，相当于命令行工具中的 `-det`，默认为 low

`catcher` 返回的结果为 `[[车牌号码, 置信度, 车牌类型, 车牌box]]`，更多信息可以直接查看其源码

### 其他调用方式

在 [文档](https://github.com/szad670401/HyperLPR/blob/master/README_CH.md) 中有提及 C/C++ 调用和 Android SDK 调用，貌似可以指定更多参数，功能更全，不过我没有试过
