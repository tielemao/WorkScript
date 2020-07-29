# python nmap扫描服务器端口

by：铁乐与猫

update：2019-07-17



## 环境

centos6.5

python 2.7

nmap


## 背景

需要使用python批量去调用nmap去扫描一批服务器的指定端口，获取其有无开放的信息。



## 脚本

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2019/7/16 19:44
# @Author  : wuweizeng
# @Email   : tielemao@163.com
# @Site    :
# @File    : serverinfo.py
# @Software: PyCharm

from __future__ import print_function
import os
import sys
import json
import nmap
import argparse


class NmapScan():

    def __init__(self, jsonPath):
        self.jsonPath = jsonPath

    def read_json(self):
        """
        读取预设好格式的储存有host与ip,prot的json文件
        :return: dict
        """
        try:
            with open(self.jsonPath, 'r') as f:
                em_dict = json.load(f)
                return em_dict
        except IOError:
            print("No such file or directory.")
            sys.exit(0)

    def nmap_scan(self, em_dict):
        """
        读取己转换回字典的json格式的配置文件，输出扫描结果
        :param em_dict:
        :return:
        """
        for k in em_dict:
            hosts = em_dict[k]["hosts"]
            port = em_dict[k]["port"]

            try:
                nm = nmap.PortScanner()
            except nmap.PortScannerError:
                print('Nmap not found', sys.exc_info()[0])
                sys.exit(0)
            except:
                print("Unexpected error:", sys.exc_info()[0])
                sys.exit(0)

            try:
                nm.scan(hosts=hosts, arguments=' -v -sS -p ' + port)  # -v 详细输出 -sS TCP SYN 扫描 -p 指定特定端口
            except Exception, e:
                print("Scan erro:" + str(e))

            for host in nm.all_hosts():
                print('----------------------------------------------------')
                print('Host : %s  %s (%s)' % (host, k, nm[host].hostname()))  # 输出主机及主机名
                print('State : %s' % nm[host].state())  # 输出主机状态，如up、down

                for proto in nm[host].all_protocols():
                    print('Protocol : %s' % proto)  # 输出协议名
                    lport = nm[host][proto].keys()
                    lport.sort()

                    for port in lport:
                        print('port : %s\tstate : %s' % (port, nm[host][proto][port]['state']))  # 输出端口状态


def _argparse():
    """
    添加参数说明，-h 或 --help 获取帮助信息
    :return:
    """
    parser = argparse.ArgumentParser(description=u"python-nmap 读取json文件去进行nmap扫描，返回扫描结果")
    parser.add_argument('-f', '--file', action='store', dest='file', required=True, help='json file path')
    return parser.parse_args()


def main():
    try:
        parser = _argparse()
        jsonPath = parser.file
        print("Load json file path : %s " % (jsonPath))
        clas = NmapScan(jsonPath)
        em_dict = clas.read_json()
        clas.nmap_scan(em_dict)
    except IOError:
        print("No such file or directory.")
        sys.exit(0)


if __name__ == '__main__':
    main()

```

json文件例子如下：

```json
{
    "server_name1":{
        "hosts": "10.17.2.59",
         "port": "9000"
    },
    "server_name2":{
        "hosts": "10.17.2.29",
        "port": "9000"
    },
    "server_name3":{
        "hosts": "10.17.2.203",
         "port": "9000"
    }
}
```

**执行方式是**：

```bash
`python emcheck.py ``-f` `相应json文件的path路径`
```

效果形如下：

```bash
(ENV27) [root@web6 python_code]# python emcheck.py -f accept_host.json
Load json file path : accept_host.json
----------------------------------------------------
Host : 192.168.17.136  evaluatel ()
State : up
Protocol : tcp
port : 80       state : open
port : 443      state : open
----------------------------------------------------
Host : 192.168.17.25  web1 ()
State : up
Protocol : tcp
port : 80       state : open
port : 443      state : open
----------------------------------------------------
Host : 192.168.17.135  web5 ()
State : up
Protocol : tcp
port : 80       state : open
port : 443      state : open
----------------------------------------------------
```



## 附：

nmap scan py

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys
import nmap

scan_row = []
input_data = raw_input('Please input hosts and port: ')  # 输入主机和端口
scan_row = input_data.split(" ")  # 分割空格
if len(scan_row) != 2:  # 判断输入的字符长度不等于2
    print
    "Input errors,example \"192.168.1.0/24 80,443,22\""  # 输出 输入错误
    sys.exit(0)
hosts = scan_row[0]  # 接收用户输入的主机
port = scan_row[1]  # 接收用户输入的端口

try:
    nm = nmap.PortScanner()  # 创建端口扫描对象
except nmap.PortScannerError:
    print('Nmap not found', sys.exc_info()[0])
    sys.exit(0)
except:
    print("Unexpected error:", sys.exc_info()[0])
    sys.exit(0)

try:
    nm.scan(hosts=hosts, arguments=' -v -sS -p ' + port)  # 调用扫描方法，参数指定扫描主机hosts，nmap扫描命令行参数arguments
except Exception, e:
    print
    "Scan erro:" + str(e)

for host in nm.all_hosts():  # 遍历扫描主机
    print('----------------------------------------------------')
    print('Host : %s (%s)' % (host, nm[host].hostname()))  # 输出主机及主机名
    print('State : %s' % nm[host].state())  # 输出主机状态，如up、down

    for proto in nm[host].all_protocols():  # 遍历扫描协议，如tcp、udp
        print('----------')
        print('Protocol : %s' % proto)  # 输入协议名

        lport = nm[host][proto].keys()  # 获取协议的所有扫描端口
        lport.sort()  # 端口列表排序
        for port in lport:  # 遍历端口及输出端口与状态
            print('port : %s\tstate : %s' % (port, nm[host][proto][port]['state']))

```

