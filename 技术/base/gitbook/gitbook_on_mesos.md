# gitbook on mesos部署文档
    文档信息
    创建人 张馨文
    邮件地址 xwzhang@dataman-inc.com
    建立时间 2015年7月22号
    更新时间 2015年7月22号
### 1.拉取基础环境镜像
    需要npm下载gitbook, 需要nodejs环境，直接从dockerhub上拉取基础镜像
    docker pull marvambass/nodejs
### 2.基于基础镜像封装gitbook发布镜像
    docker run -it marvambass/nodejs /bin/bash  进入docker内部交互界面
    npm install gitbook -g
    执行: gitbook init

    设置软连接:
    /usr/bin/node -> /root/.nvm/versions/node/v0.12.2/bin/node
    /usr/bin/gitbook -> /root/.nvm/versions/node/v0.12.2/lib/node_modules/gitbook-cli/bin/gitbook.js

    在/opt/gitbook/下放置运行脚本:
    #!/bin/bash
    bookname=$1
    /usr/bin/wget http://10.3.6.5/gitbook/${bookname}.tar.gz -P /mnt/
    if [ $? -ne 0 ];then
        echo "wget ${bookname} file fail..." && exit 1
    fi
    /bin/tar -zxf /mnt/${bookname}.tar.gz -C /mnt/
    if [ $? -ne 0 ];then
        echo "tar ${bookname}.tar.gz fail..." && exit 1
    fi
    /usr/bin/gitbook serve /mnt/${bookname}
    if [ $? -ne 0 ];then
        echo "gitbook serve fail..." && exit 1
    fi

    退出docker
### 3.Marathon发布gitbook任务脚本编写
    curl -v -X POST http://10.3.2.3:8080/v2/apps?force=true \
            -H Content-Type:application/json -d '{
        "id”:"mesosbook",
        "container": {
        "type": "DOCKER",
        "docker": {
        "image": "10.3.6.3:5000/gitbook/dataman",
        "network": "BRIDGE",
        "portMappings": [
            { "containerPort": 4000, "hostPort": 0, "protocol": "tcp"}
                ]
            }
        },
        "healthChecks": [{
                 "protocol": "HTTP",
                 "gracePeriodSeconds": 60,
                 "intervalSeconds": 30,
                 "portIndex": 0,
                 "timeoutSeconds": 60,
                 "maxConsecutiveFailures": 3
                 }],
        "cmd": "/bin/sh /opt/gitbook/gitbook.sh Mesos-CN",
        "cpus": 0.1,
        "mem": 512,
        "instances":1
    }'

### 4.在crawlerconfig主机上设置定时cron任务拉取github上更新的gitbook
    脚本如下：
    #!/bin/bash
    books=("Mesos-CN" "Mesos-in-DataMan")
    basepath=/data/gitbook
    cd ${basepath} &&  rm -rf *.tar.gz
    for bookname in ${books[@]}
    do
        if [ -d ${basepath}/$bookname ];then
            echo "will git pull rebase..."
            cd ${basepath}/$bookname && /usr/bin/git pull --rebase
        else
            cd $basepath && /usr/bin/git clone https://github.com/Dataman-Cloud/${bookname}.git
        fi
        if [ $? -ne 0 ];then
            echo "git sync operation fail..." && exit 1
        fi
        cd ${basepath}/$bookname && /usr/local/bin/gitbook init
        if [ $? -ne 0 ];then
            echo "gitbook init operation fail..." && exit 1
        fi
        /bin/tar -zcvf ${basepath}/${bookname}.tar.gz ${basepath}/${bookname}
    done

    输入命令crontab -e
    添加下面一行:
    0 */1     * * * root    cd /root && /bin/sh gitbook_cron.sh   #每一小时执行一次脚本
