---
tags: pyinstaller
category: Python
---

python程序使用到了**matplotlib**, **pillow**, **numpy**等依赖时，pyinstaller打包出的exe文件体积有300M左右，以下为将exe文件体积降至50M左右的方法

<!--more-->

>   [参考](https://rick-huang1609.medium.com/python%E6%89%93%E5%8C%85%E6%88%90%E5%9F%B7%E8%A1%8C%E6%AA%94%E5%BE%8C%E6%AA%94%E6%A1%88%E5%A4%AA%E5%A4%A7-pyinstaller%E8%88%87numpy%E7%9A%84%E9%82%A3%E4%BA%9B%E4%BA%8B-dcc75ff9d42c)

### 背景

程序使用到了matplotlib和pillow，虽然没有直接使用numpy，但是这两个库依赖于numpy

所以即使已经单独创建了虚拟环境，并只安装程序需要的库，最终打包出来的文件还是大于300M

### 解决方案

1.   根据[参考](https://rick-huang1609.medium.com/python%E6%89%93%E5%8C%85%E6%88%90%E5%9F%B7%E8%A1%8C%E6%AA%94%E5%BE%8C%E6%AA%94%E6%A1%88%E5%A4%AA%E5%A4%A7-pyinstaller%E8%88%87numpy%E7%9A%84%E9%82%A3%E4%BA%9B%E4%BA%8B-dcc75ff9d42c)，使用下面的命令来安装numpy，可以使最终exe文件大小减小至70M左右

     ```sh
     conda install -c conda-forge numpy blas=*=openblas
     ```

2.   使用[UPX](https://github.com/upx/upx)可以进一步减小exe文件的大小，从70M压缩到50M

     从[release](https://github.com/upx/upx/releases)下载最新版本的压缩包，解压，将upx.exe文件移动到pyinstaller.exe的同级目录中，使用pyinstaller打包时就会自动使用upx压缩了，不过会增加打包的时间，可以加上`--noupx`参数来不使用upx压缩

完整流程如下：

```sh
# 创建虚拟环境
conda create --name myenv python=3.11
# 激活环境
conda activate myenv
# 安装pyinstaller
conda install -c conda-forge pyinstaller
# 安装numpy
conda install -c conda-forge numpy blas=*=openblas
# 安装其他依赖
conda install -c conda-forge matplotlib pillow requests
# 打包
pyinstaller --clean -Fw my.py --name myApp
```

最终exe文件大小为50M
