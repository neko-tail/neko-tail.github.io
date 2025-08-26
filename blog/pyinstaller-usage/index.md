---
title: 'PyInstaller 使用'
date: 2024-08-27
authors: neko-tail
tags: 
  - tech
---

平时使用 Python 写的一些小工具，有时会需要提供给其他人使用，为了方便分发和使用，我通常会使用 PyInstaller 将程序打包为 exe 文件

<!-- truncate -->

## 关于 PyInstaller

[PyInstaller](https://pyinstaller.org/en/stable/) 会将代码和当前的 Python 环境打包，生成单个文件夹（包含所有依赖项，以及一个可执行文件）或单个可执行文件

具体的工作原理和两种模式的区别可以见 [What PyInstaller Does and How It Does It](https://pyinstaller.org/en/stable/operating-mode.html#what-pyinstaller-does-and-how-it-does-it)，下面是一些我在使用时的经验

### 生成单个文件夹还是生产单个可执行文件

1. 使用生成单个文件夹的方式，可以将文件夹压缩进行分发，用户只需解压后运行文件夹中的可执行文件即可

2. 使用生成单个可执行文件的方式，直接分发生成的可执行文件，用户可以直接启动该可执行文件

单个可执行文件有一些缺点，它实际的工作原理是：

- 程序启动时，在系统临时目录创建一个临时文件夹（名称中携带随机数，以确保可以多个相同程序同时执行），解压所需文件（与生成文件夹方式产生的文件相同）到此临时文件夹中，执行程序
- 程序关闭时，删除此临时文件夹

所以启动速度会比单个文件夹要慢很多，而且一旦程序崩溃，将不会删除产生的临时文件夹。此外，因为实际执行的目录是在临时文件夹，所以处理相对路径会比较麻烦

但对于没有足够计算机技能的用户来说，双击桌面上的 exe 文件执行更能让人接受

## 使用 conda 创建 Python 环境

PyInstaller 文档中说会识别项目 import 的包并导入，但在实际使用时发现，总是会将当前 Python 环境的所有包都打包进去，所以最好每一个项目单独一个虚拟环境，虚拟环境中只包含当前项目所依赖的包

一般来说，直接使用 [venv](https://docs.python.org/zh-cn/3/library/venv.html) 创建虚拟环境就可以了，不过我更常使用 [conda](https://docs.conda.io/projects/conda/en/stable/index.html)

conda 的常用命令如下：

```sh
# conda create 创建环境
# --name 指定环境名
# python=3.xx 指定python版本
# --no-default-packages 不安装默认的包
conda create --name myenv python=3.11

# 使用 environment.yml 中的配置来创建环境
call conda env create -f environment.yml

# 激活环境
conda activate myenv

# 安装依赖
conda install pyinstaller

# 安装依赖（指定 channel，我通常使用 conda-forge）
conda install -c conda-forge pyinstaller
```

通常会在项目中放置一个 environment.yml 指定项目环境，格式如下：

- name 环境名称
- channels 安装依赖使用的 channel
- dependencies 依赖

```yaml
name: nmea
channels:
  - conda-forge
dependencies:
  - python=3.11
  - pyinstaller
  - numpy
  - blas=[build=openblas]
  - matplotlib-base
  - pillow
  - pyside6
```

## 使用 PyInstaller 打包

一个我较为常用的打包语句：

```sh
# --clean 打包前清空之前打包产生的文件
# -F 生成单个可执行文件 / -D 生成单个文件夹（默认）
# -w 启动时不显示命令行窗口
# ./myapp.py 程序的启动文件
# --name 指定可执行文件的名称
# --add-data 将数据文件或目录打包进去，Linux 中参数写法为 SOURCE:DEST，Windows 中参数写法为 SOURCE;DEST，中间符号不同
pyinstaller --clean -Fw ./myapp.py --name myapp --add-data "conf/key.yaml;conf"
```

更多参数和用法可以查阅 [Using PyInstaller](https://pyinstaller.org/en/stable/usage.html#using-pyinstaller)，除了命令行参数之外，也可以通过修改打包时产生的`*.spec`文件来控制

另外在实际使用中，还遇到过一些问题，下面一一列出

### openpyxl

在使用 openpyxl 生成 Excel 文件时，需要在打包命令中加上 `--hidden-import openpyxl.cell._writer`，貌似是因为 openpyxl 部分 import 没有被 PyInstaller 识别到，所以打包时不会包含对应依赖。应该也有其他库会有类似的问题

### 单文件模式下获取数据文件

根据 [Bundling data files with PyInstaller (--onefile)](https://stackoverflow.com/questions/7674790/bundling-data-files-with-pyinstaller-onefile) 下面的回答，可以使用下面的代码在单文件模式下获取数据文件

```python
def resource_path(relative_path):
    """ Get absolute path to resource, works for dev and for PyInstaller """
    try:
        # PyInstaller creates a temp folder and stores path in _MEIPASS
        base_path = sys._MEIPASS
    except Exception:
        base_path = os.path.abspath(".")

    return os.path.join(base_path, relative_path)
```

### 操作系统

PyInstaller 在 Linux 中打包生成一个可执行文件，在 Windows 中打包生成一个 exe 文件

但除此之外，具体的系统版本也会有影响，比如在 Win10 中打包出的 exe 文件，在 Win7 中执行就会报错，原因貌似是缺少了一些系统 dll，需要在 Win7 中打包才行

### 打包体积

当 Python 项目中依赖了 [matplotlib](https://matplotlib.org/stable/)、[pillow](https://python-pillow.org/)、[numpy](https://numpy.org/) 等包时，PyInstaller 打出来的 exe 文件能达到 300M 左右，但通过一些方法，可以将 exe 文件的体积降到 50M 左右

#### numpy

根据 [参考](https://rick-huang1609.medium.com/python%E6%89%93%E5%8C%85%E6%88%90%E5%9F%B7%E8%A1%8C%E6%AA%94%E5%BE%8C%E6%AA%94%E6%A1%88%E5%A4%AA%E5%A4%A7-pyinstaller%E8%88%87numpy%E7%9A%84%E9%82%A3%E4%BA%9B%E4%BA%8B-dcc75ff9d42c)，conda 环境中使用下面的命令来安装 numpy，可以使 exe 文件体积降到 70M 左右

```sh
conda install -c conda-forge numpy blas=*=openblas
```

#### upx 压缩

[upx](https://github.com/upx/upx) 压缩可以进一步减小 exe 文件的体积，从上一步的 70M 降低到大约 50M

将下载解压出的 upx.exe 文件复制到 pyinstaller.exe 同级目录中，之后再使用 PyInstaller 打包时就会自动使用 upx 压缩

但要注意的是：upx 压缩会增加打包耗时，如果想要不使用 upx 压缩，可以在打包时加上 `--noupx` 参数
