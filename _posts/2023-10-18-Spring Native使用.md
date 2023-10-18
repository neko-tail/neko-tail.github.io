---
tag: 
    - Spring Native
    - graalvm
category: Spring
---

尝试Spring Native功能，将一个SpringBoot 3项目编译为二进制文件，提升程序启动速度

<!--more-->

>   相关链接：
>
>   [graalvm](https://www.graalvm.org/)
>
>   [GraalVM Native Image Support](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html)
>
>   使用版本：
>
>   graalvm:  graalvm-jdk-17.0.8+9.1
>
>   spring-boot: 3.1.4
>
>   native-maven-plugin: 0.9.27

## 优缺点

优点：

-   程序启动时间段
-   运行时使用内存少
-   运行环境不需要安装jvm

缺点：

-   编译时间长
-   部分依赖不支持（或需要进行特殊配置）
-   需要生成提示文件来支持反射等功能
-   生成的二进制文件较大

## GraalVM

在下载页面，选择对应的Java版本和平台，下载对应GraalVM（这里以graalvm-jdk17 Windows平台为例）

![image-20231018111110132](https://cdn.thmaster.net/pic/202310181111391.png)

自行解压、配置环境变量，可以将其作为jdk使用

## 项目

### 创建项目

选择JDK为graalvm

![image-20231018112421028](https://cdn.thmaster.net/pic/202310181124070.png)

选择GraalVM Native Support，创建项目

这里我额外选择了Spring Web和Lombok，来写一个用于测试的功能

![image-20231018112723143](https://cdn.thmaster.net/pic/202310181127177.png)

### 编写代码（用于测试）

修改NativeDemoApplication，编写一个接口

```java
@RestController
@SpringBootApplication
public class NativeDemoApplication {
    
    @GetMapping
    public String hello() {
        return "Hello World!";
    }
    
    public static void main(String[] args) {
        SpringApplication.run(NativeDemoApplication.class, args);
    }
    
}
```

直接运行，请求正常，接下来将项目编译为二进制文件

![image-20231018113334711](https://cdn.thmaster.net/pic/202310181133745.png)

## 编译环境

### Linux

>   在Linux上进行编译，需要有gcc，但是有部分服务器上会没有

以Ubuntu为例，安装gcc：

```sh
apt install gcc
```

### Windows

>   在Windows上进行编译，需要额外安装Visual Studio，并配置相关环境变量
>
>   同时编译命令不能直接在cmd执行

#### Visual Studio

自行从官网下载安装

以下以安装VS在D:\opt为例，替换为自己VS安装路径，注意版本号可能不同

-   Path

    添加下列内容

    ```
    D:\opt\Microsoft Visual Studio\2022\Enterprise\VC\Tools\MSVC\14.35.32215\bin\Hostx64\x64
    ```

-   INCLUDE

    新增此环境变量，内容如下

    ```txt
    D:\opt\Microsoft Visual Studio\2022\Enterprise\VC\Tools\MSVC\14.35.32215\include
    D:\Windows Kits\10\Include\10.0.22000.0\shared
    D:\Windows Kits\10\Include\10.0.22000.0\ucrt
    D:\Windows Kits\10\Include\10.0.22000.0\um
    D:\Windows Kits\10\Include\10.0.22000.0\winrt
    ```

-   LIB

    新增此环境变量，内容如下

    ```txt
    D:\opt\Microsoft Visual Studio\2022\Enterprise\VC\Tools\MSVC\14.35.32215\lib\x64
    D:\Windows Kits\10\Lib\10.0.22000.0\um\x64
    D:\Windows Kits\10\Lib\10.0.22000.0\ucrt\x64
    ```

#### 命令行窗口

打开**x64 Native Tools Command Prompt for VS 2022**（可以通过Win+S搜索打开）

![image-20231018113927085](https://cdn.thmaster.net/pic/202310181139122.png)

## 编译

>   编译耗时较长，需要高CPU和高内存，建议在性能好的机器上进行
>
>   目前不支持交叉编译，如果需要aarch64的二进制文件，需要在对应架构的服务器上进行编译

cd到项目的目录中，执行编译指令

```sh
mvn -Pnative native:compile
```

在我的电脑上执行了3分43秒，生成的二进制文件大小为81.1MB

在生成的target目录下，可以找到**native-demo.exe**（Windows）或**native-demo**（Linux）文件

执行后请求接口，功能正常

![image-20231018115013049](https://cdn.thmaster.net/pic/202310181150089.png)



