# 获取Nexus系列交换机的cpu值



by: 铁乐猫

date: 2020-11-20



## 环境

交换机：Nexus 92160ycx



## 开启nxapi

```
switch24(config)# feature nxapi
```

验证开启

```bash
switch24# show nxapi 
nxapi enabled
NXAPI timeout 10
HTTPS Listen on port 443
```



## 获取cpu值

```python
#!/usr/bin/env python
# coding: utf-8

import requests
import json
import argparse
import re

def get_cpu(url):
    #For NXAPI to authenticate the client using client certificate, set 'client_cert_auth' to True.
    #For basic authentication using username & pwd, set 'client_cert_auth' to False.
    client_cert_auth=False
    #switchuser='USERID'
    switchuser='<此处填入交换机用户名>'
    #switchpassword='PASSWORD'
    switchpassword='<此处填入交换机密码>'
    client_cert='PATH_TO_CLIENT_CERT_FILE'
    client_private_key='PATH_TO_CLIENT_PRIVATE_KEY_FILE'
    ca_cert='PATH_TO_CA_CERT_THAT_SIGNED_NXAPI_SERVER_CERT'

    myheaders={'content-type':'application/json'}
    payload={
      "ins_api": {
      "version": "1.0",
      "type": "cli_conf",
      "chunk": "0",
      "sid": "sid",
      "input": "show processes cpu sort | grep \"CPU utilization\"",
      "output_format": "json",
      "rollback": "stop-on-error"
      }
    }
    if client_cert_auth is False:
        response = requests.post(url,data=json.dumps(payload), headers=myheaders,auth=(switchuser,switchpassword)).json()
    else:
        https_url = url.replace('http','https')
        response = requests.post(https_url,data=json.dumps(payload), headers=myheaders,auth=(switchuser,switchpassword),cert=(client_cert,client_private_key),verify=ca_cert).json()
    return response

def re_body(res):
    # e.g 'CPU utilization for five seconds: 8%/0%; one minute: 9%; five minutes: 10%\n'
    body = res["ins_api"]["outputs"]["output"]["body"]
    # 正则匹配上数字
    pattren = re.compile(r'\d+')
    result = pattren.findall(body)
    # 移除第二位数字（seconds：8%:/0%）
    result.pop(1)
    info = {"five_seconds":result[0],"one_minute":result[1],"five_minutes":result[2]}
    return info

def _argparse():
    """
    添加参数说明,-h 或 --help可以获取到帮助信息
    """
    parser = argparse.ArgumentParser(description="python get switch24 or switch25 cpu info")
    parser.add_argument("-s", "--switch", type=int, dest='switch', choices=[24,25], help="input 24 => switch24 or 25 => switch25")
    parser.add_argument("-t", "--time", dest='time', choices=['s','m','f'], help="select seconds or one minute or five minute")
    return parser.parse_args()

def main():
    sw24_url = 'http://192.168.1.38/ins'
    sw25_url = 'http://192.168.1.39/ins'
    parser = _argparse()
    sw = parser.switch
    slect_time = parser.time
    if sw == 24:
        res = get_cpu(sw24_url)
        info = re_body(res)
        print(info)
        if slect_time == "s":
            return info["five_seconds"]
        elif slect_time == "m":
            return info["one_minute"]
        elif slect_time == "f":
            return info["five_minutes"]
        else:
            # 交换机参数输入错误
            return(401)
    elif sw == 25:
        res = get_cpu(sw25_url)
        info = re_body(res)
        print(info)
        if slect_time == "s":
            return info["five_seconds"]
        elif slect_time == "m":
            return info["one_minute"]
        elif slect_time == "f":
            return info["five_minutes"]
        else:
            # 交换机参数输入错误
            return(401)

    else:
        # 交换机参数输入错误
        return(400)

if __name__ == '__main__':
    main()
```

效果

```
[root@czabbix1]# python3 nexus_cpu.py -s 24
{'five_seconds': '8', 'one_minute': '8', 'five_minutes': '9'}

[root@czabbix1]# python3 nexus_cpu.py -h
usage: nexus_cpu.py [-h] [-s {24,25}] [-t {s,m,f}]

python get switch24 or switch25 cpu info

optional arguments:
  -h, --help            show this help message and exit
  -s {24,25}, --switch {24,25}
                        input 24 => switch24 or 25 => switch25
  -t {s,m,f}, --time {s,m,f}
                        select seconds or one minute or five minute
```

这里设计的参数是-t后接s的话是秒，m是一分钟, f是5分钟的cpu占用百分比值。