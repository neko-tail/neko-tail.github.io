---
title: '使用 VMware 创建 macOS 虚拟机'
date: 2024-07-08T14:31:08+08:00
draft: false
categories:
  - tech
tags:
  - macOS
  - 虚拟机
---

为了用 flutter 打包 ios app，尝试创建一个 macOS 虚拟机

最终使用的方案是 VMware 创建 macOS Sonoma 虚拟机，并使用 Clover Configurator 生成设备信息，成功登录 Apple ID，下载 xcode

可以直接跳到 [VMware](#vmware) 章节

## VirtualBox

一开始打算使用 VMware 来创建虚拟机，但在 VMware 宣布个人使用免费之后，下载就需要注册账号了，有点麻烦，所以换成了 VirtualBox，照着 youtube 的[这个视频](https://www.youtube.com/watch?v=DkeJuv7LXLY)来操作

### 自定义安装路径问题

安装时发现，最新版本的 VirtualBox 不能自定义安装路径了（见 [Can't install Virtualbox 7.0.16 outside of C:\Program Files](https://www.mfpud.com/topics/11002/)），可以选择下载旧版本（如 7.0.14）安装

### 创建虚拟机

使用视频给出的 iso 文件创建 macOS Big Sur 虚拟机，结果报错了，根据 reddit 的几个帖子（找不到链接了），创建虚拟机时配置 cpu 核心数为 2，显存为 128 M，能成功创建

### 安装 macos

关闭 VirtualBox，在具有管理员权限的 powershell 窗口，先 cd 到 VirtualBox 的安装目录，再执行视频中给出的命令（`"macos"`为自定义的虚拟机名称）设置虚拟机硬件信息进行伪装

```powershell
.\VBoxManage.exe modifyvm "macos" --cpuidset 00000001 000106e5 00100800 0098e3fd bfebfbff
.\VBoxManage.exe setextradata "macos" "VBoxInternal/Devices/efi/0/Config/DmiSystemProduct" "iMac19,1"
.\VBoxManage.exe setextradata "macos" "VBoxInternal/Devices/efi/0/Config/DmiBoardProduct" "Mac-551B86E5744E2388"
.\VBoxManage.exe setextradata "macos" "VBoxInternal/Devices/smc/0/Config/DeviceKey" "ourhardworkbythesewordsguardedpleasedontsteal(c)AppleComputerInc"
.\VBoxManage.exe setextradata "macos" "VBoxInternal/Devices/smc/0/Config/GetKeyFromRealSMC" 1
```

之后重新打开虚拟机，可以正常走安装流程，但要注意：**安装引导中不要登录 Apple ID**

安装完毕之后发现虚拟机的分辨率较低，使用下面的命令来修改屏幕分辨率

```powershell
.\VBoxManage.exe setextradata "macos" VBoxInternal2/EfiGraphicsResolution "1920x1080"
```

到此步骤，macOS 虚拟机已经可以使用，但在我点进 App Store 准备下载 xcode 时，却发现上面提示要更新到 macos Sonoma

直接在虚拟机中更新系统，结果陷入了反复重启的问题当中，最后放弃，选择尝试使用 VMware

## VMware

> 参考 [Windows上でVMwareを使ってMacOSをインストールする方法](https://www.partitionwizard.jp/resizepartition/mac-emulator-for-windows.html) 进行

### 软件准备

在 VMware 官网注册账号下载 VMware Workstations 并安装

安装后的 VMware 无法创建 macOS 虚拟机，需要进行解锁

下载 [unlocker](https://github.com/paolo-projects/unlocker) 并解压，使用管理员权限执行里面的`win-install.cmd`，即可解锁 VMware 创建虚拟机中的 macos 选项，此外还会自动下载`com.vmware.fusion.zip.tar`文件并从中解压出`darwin.iso`和`darwinPre15.iso`两个文件，放在`tools`目录下

### iso 文件

这里需要一份 macOS Sonoma 的 iso 文件，因为在 [VirtualBox](#virtualbox) 章节中已经下载了 macos Big Sur 的 iso 文件，所以这里我打算在 macos Big Sur 虚拟机中自行创建 macos Sonoma 的 iso 文件

下载并创建 iso 文件的方法可以参考 youtube 视频 [How to Download and Create macOS Sonoma ISO File](https://www.youtube.com/watch?v=Uun55VXu72M&t=0s)

首先，在 macos 的 APP Store 中搜索下载 macos Sonoma，下载完成之后不要更新，直接退出

然后在 macos 命令行中依次执行下面的命令生成 Sonoma.iso 文件

```sh
hdiutil create -o /tmp/Sonoma -size 16384m -volname Sonoma -layout SPUD -fs HFS+J
hdiutil attach /tmp/Sonoma.dmg -noverify -mountpoint /Volumes/Sonoma
sudo /Applications/Install\ macOS\ Sonoma.app/Contents/Resources/createinstallmedia --volume /Volumes/Sonoma --nointeraction
hdiutil eject -force /Volumes/Install\ macOS\ Sonoma
hdiutil convert /tmp/Sonoma.dmg -format UDTO -o ~/Desktop/Sonoma
mv -v ~/Desktop/Sonoma.cdr ~/Desktop/Sonoma.iso
rm -fv /tmp/Sonoma.dmg
```

将桌面的 Sonoma.iso 文件复制到 win 主机中，即可用来创建一个 macOS Sonoma 虚拟机

### 创建 Mac 虚拟机

使用 Sonoma.iso 文件创建一个虚拟机（这里可以正常设置 CPU 核心数等配置），找到该虚拟机的配置文件如`macOS 14.vmx`打开

在`smc.present = "TRUE"`行下面添加`smc.version = "0"`，最终如下

```toml
smc.present = "TRUE"
smc.version = "0"
```

保存修改后，启动虚拟机

根据引导，使用磁盘工具抹掉磁盘，安装 MacOS Sonoma 即可

### VMware Tools

虚拟机创建完毕之后，发现也有分辨率过低的问题，可以安装 VMware Tools 解决

直接使用 VMware 给 macOS 虚拟机安装 VMware Tools 会失败，这里需要使用`darwin.iso`手动安装

在[软件准备](#软件准备)一节中，unlocker 会自动下载`darwin.iso`和`darwinPre15.iso`，没有的话可以手动下载：

- 下载最新版本的 [com.vmware.fusion.zip.tar](https://softwareupdate.vmware.com/cds/vmw-desktop/fusion/13.5.2/23775688/universal/core/com.vmware.fusion.zip.tar)
- 解压后于`com.vmware.fusion\payload\VMware Fusion.app\Contents\Library\isoimages\x86_x64`目录中可以找到找到`darwin.iso`和`darwinPre15.iso`两个文件

将这两个 iso 文件放于 VMware 根目录（不清楚是否为必须操作）

在 macOS 虚拟机设置的 CD/DVD 选项中使用`darwin.iso`文件，重启虚拟机

虚拟机桌面会出现一个光盘，双击并选择`安装 VMware Tools`，等待安装结束后在`系统设置-隐私与安全性`里面启用

重启之后虚拟机分辨率变为 1920x1080

### 登录 Apple ID

在虚拟机中无法登录 Apple ID，这里需要进行伪装，具体方法可以参考下面两个链接

- [How to run iMessage/FaceTime on VMWare (or should I say: how to run iMessage/FaceTime on a Windows PC, and for those that can't afford to run macOS on Bare Metal or whatever)](https://www.reddit.com/r/hackintosh/comments/c613ia/how_to_run_imessagefacetime_on_vmware_or_should_i/)
- [Deploying macOS in VMWare on Windows (Full Guide)](https://docs.bluebubbles.app/server/advanced/macos-virtualization/running-a-macos-vm/deploying-macos-in-vmware-on-windows-full-guide#fixing-iservices)

首先在虚拟机中下载并启动 [Clover Configurator](https://mackie100projects.altervista.org/download-clover-configurator/)

打开机型设置，右下角选择机型（如 MacPro7,1），随后点击`序列表`文本旁的`生成新的`按钮来随机生成机型信息

点击右侧的`检查覆盖范围`按钮，会自动打开 Apple 的一个网页，输入序列号，检查此序列号是否有效（注意：这里必须要是**无效**的序列号）

记录当前页面的几个数据：

- Product Name（如：MacPro7,1）
- Board-ID（如：Mac-27AD2F918AE68F61）
- 序列号（如：F5KY6EYJP7QM）

然后打开`变量设置`，点击`ROM`文本旁的`生成`按钮，随机生成信息

记录信息中的两个数据：

- ROM（如：72A85F41745B）
- MLB（如：F5K905130QXFHDDFB）

之后就可以关闭 Clover Configurator，并关闭虚拟机

打开虚拟机目录下的`.vmx`文件，添加下面的内容，其中的部分值替换为记录的数据

```toml
board-id = "记录的 Board-ID"
hw.model.reflectHost = "FALSE"
hw.model = "记录的 Product Name"
serialNumber.reflectHost = "FALSE"
serialNumber = "记录的序列号"
smbios.reflectHost = "FALSE"
efi.nvram.var.ROM.reflectHost = "FALSE"
efi.nvram.var.MLB.reflectHost = "FALSE"
efi.nvram.var.ROM = "记录的 ROM"
efi.nvram.var.MLB = "记录的 MLB"
```

打开 [HWaddress](https://hwaddress.com/company/apple-inc/)，选择任意一条数据，记录其 OUI 字段（如：00-1F-F3）

接着进行几项修改：

- `board-id.reflectHost`设置为`"TRUE"`
- `ethernet0.addressType`从`"generated"`改为`"static"`
- `ethernet0.generatedAddress`值的前三段替换为记录的 OUI
  如：值为`00:0c:29:bb:91:7f`，OUI 为`00-1F-F3`，修改后的值为`00:1f:f3:bb:91:7f`
- `ethernet0.generatedAddress`重命名为`ethernet0.Address`
- `ethernet0.generatedAddressOffset = "0"`替换为`ethernet0.checkMACAddress = "FALSE"`

最后保存并关闭`.vmx`文件

重启虚拟机，现在可以正常登录 Apple ID 了

另外，在我尝试时，直接在设置或者 APP Store 中登录，会在设置手机号码双重认证之后报错，后来先在 appleid.apple.com 中登录账号，设置双重认证，再在 APP Store 中登录就正常了
