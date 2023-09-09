## 1.简介

Wireshark是非常流行的网络封包分析软件，可以截取各种网络数据包，并显示数据包详细信息

## 2. 使用wireshark抓包

1. 终端运行wireshark

```
sudo wireshark
```

2. 如果界面上没有显示网卡名称，并且wifi显示wifi dump，输入以下命令进行配置

```
sudo dpkg-reconfigure wireshark-common //然后选no
sudo usermod -a -G wireshark <username> //username填写自己ubuntu的用户名
```

3. 在wireshark界面上选择网卡，点击start，抓包成功

## 3. 过滤

1. 在终端输入以下命令向baidu.com发送请求

```
ping baidu.com
```

2. 在wireshark上方display filter栏输入

```
ip.addr==39.156.66.10 and icmp
```

3. wireshark上显示出过滤后的数据包

## 4. wireshark界面分析

1. [文章在这](https://www.cnblogs.com/linyfeng/p/9496126.html)