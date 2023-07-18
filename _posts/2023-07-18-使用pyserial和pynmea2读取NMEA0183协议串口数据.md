---
tags: Python nmea 串口
category: Python
---

pi使用外接定位模块，通过NMEA0183协议向串口中写入数据。使用pyserial读取串口数据，使用pynmea2解析数据，获取经纬度、海拔、速度等数据

<!--more-->

>   [pyserial](https://github.com/pyserial/pyserial)
>
>   [pynmea2](https://github.com/Knio/pynmea2)
>
>   [参考博客](https://dewey.dunnington.ca/post/2016/using-pyserial-pynmea2-and-raspberry-pi-to-log-nmea-output/)
>
>   [参考的介绍NMEA协议的文章]([NMEA协议解析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/434992232))

### GSP数据来源

外接模块通过USB连接pi，通过`/dev/ttyUSB0`可以读取数据

读取数据时需要主要波特率，如果波特率不对，从串口读取到的数据就会是乱码，调整波特率的方法如下

```sh
# 查看串口信息
stty -F /dev/ttyUSB0 -a
# 修改波特率为115200
stty -F /dev/ttyUSB0 ispeed 115200 ospeed 115200 cs8
# 读取串口数据（一开始有效，后面不知为啥不起效果了）
cat /dev/ttyUSB0
```

### 使用pyserial读取串口数据

注意：从串口中读取的数据需要解码为字符串，NMEA协议的字符编码是ASCII

```python
import serial

with serial.Serial('/dev/ttyUSB0', 115200, timeout=5.0) as ser:
        while 1:
            line = ser.readline().decode("ascii")
                if line is not None:
                    print(line)
```

### 使用pynmea2解析数据

使用`pynmea2.parse()`方法解析一行数据，并返回对应类型的结果

```python
import pynmea2
from pynmea2 import RMC, GGA


data = """\
$GPGGA,092750.000,5321.6802,N,00630.3372,W,1,8,1.03,61.7,M,55.2,M,,*76
$GPGSA,A,3,10,07,05,02,29,04,08,13,,,,,1.72,1.03,1.38*0A
$GPGSV,3,1,11,10,63,137,17,07,61,098,15,05,59,290,20,08,54,157,30*70
$GPGSV,3,2,11,02,39,223,19,13,28,070,17,26,23,252,,04,14,186,14*79
$GPGSV,3,3,11,29,09,301,24,16,09,020,,36,,,*76
$GPRMC,092750.000,A,5321.6802,N,00630.3372,W,0.02,31.66,280511,,,A*43
$GPGGA,092751.000,5321.6802,N,00630.3371,W,1,8,1.03,61.7,M,55.3,M,,*75
$GPGSA,A,3,10,07,05,02,29,04,08,13,,,,,1.72,1.03,1.38*0A
$GPGSV,3,1,11,10,63,137,17,07,61,098,15,05,59,290,20,08,54,157,30*70
$GPGSV,3,2,11,02,39,223,16,13,28,070,17,26,23,252,,04,14,186,15*77
$GPGSV,3,3,11,29,09,301,24,16,09,020,,36,,,*76
$GPRMC,092751.000,A,5321.6802,N,00630.3371,W,0.06,31.66,280511,,,A*45\
"""
for line in data.split("\n"):
    msg = pynmea2.parse(line)
    if isinstance(msg, RMC):
        print(f"""
        经度 : {msg.longitude}
        纬度 : {msg.latitude}
        时间 : {msg.datetime}
        速度 : {msg.spd_over_grnd}
        """)
    elif isinstance(msg, GGA):
        print(f"""
        经度 : {msg.longitude}
        纬度 : {msg.latitude}
        时间 : {msg.timestamp}
        海拔 : {msg.altitude}
        """)
        
```

以下为程序运行结果

```txt
经度 : -6.50562
纬度 : 53.361336666666666
时间 : 09:27:50+00:00
海拔 : 61.7


经度 : -6.50562
纬度 : 53.361336666666666
时间 : 2011-05-28 09:27:50+00:00
速度 : 0.02


经度 : -6.5056183333333335
纬度 : 53.361336666666666
时间 : 09:27:51+00:00
海拔 : 61.7


经度 : -6.5056183333333335
纬度 : 53.361336666666666
时间 : 2011-05-28 09:27:51+00:00
速度 : 0.06
```

