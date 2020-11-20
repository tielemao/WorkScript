# python收集网卡信息

by: 铁乐猫


### 背景

需要对全服服务器的网卡进行统计信息，收集达到TiB级别的主机信息。



参考脚本如下：

```python
#!/usr/bin/python
#coding:utf-8
import datetime
import psutil
 
'''
     查看系统基本信息
'''
 
def get_cpu_info():  #获取cpu状态
    print('\033[1;31;40m-------------------------cpu状态信息-----------------------------------\033[0m')
    print('cpu 物理逻辑为 ：%s' % psutil.cpu_count())
    print('cpu 物理个数为 ：%s' % psutil.cpu_count(logical=False))
    print('cpu 执行用户进程时间百分比 ：%% %s' % (psutil.cpu_times_percent().user) )
    print('cpu 处于空闲时间状态百分比 ：%% %s' % (psutil.cpu_times_percent().idle) )
    print('cpu 执行内核进程和中断时间百分比 ：%%%s' % psutil.cpu_times_percent().system)
    print('cpu 处于io等待状态百分比 ：%%%s' % psutil.cpu_times_percent().iowait)
def get_mem_info():   #获取内存状态信息
    print('\033[1;31;40m-------------------------内存状态信息-----------------------------------\033[0m')   
    mem = psutil.virtual_memory()
    print('内存总大小：%.1fG' % (mem.total/1024/1024/1024))
    print('空闲内存大小：%.2fG' % (mem.free/1024/1024/1024))
    swap = psutil.swap_memory()
    print('swap分区总大小：%.1fG' % (swap.total/1024/1024/1024))
    print('swap空闲分区大小：%.1fG' % (swap.free/1024/1024/1024))
    
def get_disk_info():   #获取磁盘信息
    print('\033[1;31;40m-------------------------磁盘信息-----------------------------------\033[0m')   
    for i in psutil.disk_partitions():
        print(i)
        disk_usage = psutil.disk_usage(i[1])
        print('%s 使用情况如下：' % i[1])
        print('----------分区总大小：%.1fG' % (disk_usage[0]/1024/1024/1024))   
        print('----------已使用空间：%.1fG' % (disk_usage[1]/1024/1024/1024)) 
        print('----------剩余空间：  %.1fG' % (disk_usage[2]/1024/1024/1024))
def get_network_info():   #获取网络信息
    print('\033[1;31;40m-------------------------网络信息-----------------------------------\033[0m') 
    network = psutil.net_io_counters()
    print('已发送字节数：%.1fM' % (network[0]/1024/1024/1024))
    print('已接收字节数：%.1fM' % (network[1]/1024/1024/1024))
    print('已发送数据包个数：%.1f' % network[2])        
    print('已接收数据包个数：%.1f' % network[3])
def get_user_info():    #获取用户信息    
    print('\033[1;31;40m-------------------------用户信息-----------------------------------\033[0m') 
    print('系统开机时间为：%s' % datetime.datetime.fromtimestamp(psutil.boot_time()).strftime('%Y-%m-%d %H:%M:%S'))
    user = psutil.users()
    for i in range(0,len(user)):    
        print('用户名 ：%s,终端 ：%s,主机 ：%s，登录时间 %s' %(user[i][0],user[i][1],user[i][2],datetime.datetime.fromtimestamp(user[i][3]).strftime('%Y-%m-%d %H:%M:%S')))
        
if  __name__ == '__main__' :
    get_cpu_info()
    get_mem_info()
    get_disk_info()
    get_network_info()
    get_user_info()
```

运行结果如下：

```python
tielemao:/home/operation # python network_info.py 
-------------------------cpu状态信息-----------------------------------
cpu 物理逻辑为 ：1
cpu 物理个数为 ：1
cpu 执行用户进程时间百分比 ：% 0.0
cpu 处于空闲时间状态百分比 ：% 0.0
cpu 执行内核进程和中断时间百分比 ：%0.0
cpu 处于io等待状态百分比 ：%0.0
-------------------------内存状态信息-----------------------------------
内存总大小：1.0G
空闲内存大小：0.00G
swap分区总大小：0.0G
swap空闲分区大小：0.0G
-------------------------磁盘信息-----------------------------------
sdiskpart(device='/dev/vda1', mountpoint='/', fstype='ext4', opts='rw,relatime,stripe=32639,data=ordered')
/ 使用情况如下：
----------分区总大小：39.0G
----------已使用空间：32.0G
----------剩余空间：  4.0G
-------------------------网络信息-----------------------------------
已发送字节数：2.0M
已接收字节数：5.0M
已发送数据包个数：4902956.0
已接收数据包个数：7182500.0
-------------------------用户信息-----------------------------------
系统开机时间为：2020-03-14 12:21:02
用户名 ：operation,终端 ：tty1,主机 ：，登录时间 2020-03-14 12:26:40
用户名 ：operation,终端 ：pts/0,主机 ：10.8.0.6，登录时间 2020-03-25 17:29:36
```



### 脚本

后来发现psutil没法获取总的计数器，便又新写了一个：

```python
#!/usr/bin/python
#coding:utf-8

"""
by:    wuweizeng
email: tielemao@163.com
date:  2020-03-26
用于检测服务器网卡统计信息达到TiB的主机
"""

import os

cmd = 'ifconfig'
cmd2 = 'hostname'
network_info = os.popen(cmd).readlines()
for line in network_info:
    if "TiB" in line:
        hostname = os.popen(cmd2).readline()
        print(hostname)
        break
```

