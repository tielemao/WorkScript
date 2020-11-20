# 检查iptables规则

by:  铁乐猫

date：2020-11-20



```python
import os
import subprocess
import commands
list1 =[]
with open('./ip2.txt','r') as f:
    for line in f:
	cmd = 'iptables -nL | grep %s' % (line)
	numr = os.system(cmd)
        if numr == 256:
	    list1.append(line)
print list1
```



在ip2.txt中填入一行行的ip地址作为传参进py脚本中检测有没有符合的规则。