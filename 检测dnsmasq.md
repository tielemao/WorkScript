# 获取服务器指定应用的版本信息

by：铁乐猫

date：2020-10-30



以下代码用于检测dnsmasq 解析域名情况。

```python
import time
import datetime
import urllib.request
import socket
import sys
import ssl

class check_web_speed(object):
    def __init__(self, webs_line):
        self.web_headers = {
            "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:52.0) Gecko/20100101 Firefox/52.0"
        }
        self.webs = webs_line
        ssl._create_default_https_context = ssl._create_unverified_context

    def check_web(self):
        for web in self.webs:
            try:
                socket.gethostbyname(web)
            except socket.gaierror:
                span = "domain error!"
            else:
                req = urllib.request.Request(
                    "https://" + web,
                    headers=self.web_headers
                )
                start_time = datetime.datetime.now()
                try:
                    response = urllib.request.urlopen(req, timeout=5)
                except urllib.request.URLError as e:
                    span = e.reason
                else:
                    end_time = datetime.datetime.now()
                    span = str((end_time - start_time).total_seconds()).ljust(9) + " seconds"
            print(web.ljust(20), span)

if __name__ == "__main__":
    if len(sys.argv) >= 2:
        webs = sys.argv[1:]
    else:
        webs = [
            "www.facebook.com",
            "www.youtube.com",
            "www.mobile01.com",
            "www.google.com",
            "telegram.org",
            "www.baidu.com",
            "mail.163.com"
        ]

    print(time.strftime("%Y-%m-%d %H:%M:%S", time.localtime()))
    a = check_web_speed(webs)
    a.check_web()
```

