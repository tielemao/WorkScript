# 检查elk状态脚本

**背景：**

cbamboo2这台服务器上面运行有kibana与elasticsearch。

**需求：**

需要对kibana和elasticsearch的运行状态做一个脚本监控。

而报警脚本我们使用的是skypealarm脚本。通过这个脚本可以直接发报警到飞书上（最初是skype上）

脚本如下：

```shell
#==========================================================
#   Copyright (C) 2020 Sangfor Ltd. All rights reserved.
#
#  Author:     WuWeiZeng
#  Email:      tielemao@163.com
#  Time:       2020-04-24
#  Use:        check elk status
#==========================================================
check_kibana(){
    # check kibana running status
    kibana_status=`/etc/init.d/kibana status`
    echo "$kibana_status"
    if [ "$kibana_status" != "kibana is running" ]; then
        echo "cbamboo2 kibana is no running" | /root/bin/skypealarm -w
        # restart kibana
        /etc/init.d/kibana restart
        if [ $? -ne 0 ]; then
            echo "cbamboo2 kibana is no running and restart failed, Pls call wuweizeng." | /root/bin/skypealarm -c
        fi;
        # double check
        sleep 30s
        kibana_status2=`/etc/init.d/kibana status`
        echo "$kibana_status2"
        if [ "$kibana_status2" != "kibana is running" ]; then
            echo "cbamboo2 kibana is no running, Pls call wuweizeng." | /root/bin/skypealarm -c
        fi;
    fi;
}

check_es(){
    # check es running status
    es_status=`/etc/init.d/elasticsearch status`
    # eg. elasticsearch (pid 33061) is running
    es_running='is running'
    if [[ "$es_status" =~ "$es_running" && "$es_status" =~ "elasticsearch" ]]; then
        echo "$es_status"
    else
        echo "cbamboo2 es is no running" | /root/bin/skypealarm -w
        # restart es
        /etc/init.d/elasticsearch restart
        if [ $? -ne 0 ]; then
            echo "cbamboo2 es is no running and restart failed, Pls call wuweizeng." | /root/bin/skypealarm -c
        fi;
        # double check
        sleep 60s
        es_status2=`/etc/init.d/elasticsearch status`
        if [[ "$es_status2" =~ "$es_running" && "$es_status2" =~ "elasticsearch" ]]; then
            echo "$es_status2"
        else
            echo "cbamboo2 es is no running, Pls call wuweizeng." | /root/bin/skypealarm -c
        fi;
    fi;
}

check_kibana
check_es
```

