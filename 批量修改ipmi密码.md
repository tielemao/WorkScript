# 批量修改ipmi密码

by: 铁乐猫

date: 2020-7-31



```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2020/7/31 15:38
# @Update  : 2020/7/31 15:38
# @Author  : tielemao
# @Email   : tielemao@163.com
# @File    : ipmi_set_passwd.py
# @Desc    : 批量修改ipmi密码

import xlrd
import subprocess

xls_file = "2U_ipmi.xlsx"

def ipmi_xlsx(file):
    wb = xlrd.open_workbook(filename=file)
    sheet1 = wb.sheet_by_name("Sheet1")
    pwd = sheet1.col_values(2)
    host = sheet1.col_values(3)
    ipmi_dic = dict(zip(host, pwd))
    passwd = "<填入自定义的密码替换>"
    for h, p in ipmi_dic.items():
        cmd = "ipmitool -I lanplus -H " + h + " -U ADMIN -P " + p + " user set password 2 " + passwd
        print(cmd)
        subprocess.run(cmd, shell=True, capture_output=True)

ipmi_xlsx(xls_file)
```



配合记录有主机名/ipmi用户名/旧密码/ip的表格来实行，2U_ipmi.xlsx例如下：



| web233 | ADMIN | 123456 | 192.168.1.101 |
| ------ | ----- | ------ | ------------- |
| web234 | ADMIN | 123456 | 192.168.1.102 |