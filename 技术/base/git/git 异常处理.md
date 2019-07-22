# Git 异常处理
## 大文件 push 问题

remote: warning: See http://git.io/iEPt8g for more information.
remote: warning: File 运维文档/技术文档/mesos/mesosondocker/src/mesos/mesos-marathon/mesos-marathon-0.10.1/marathon-0.10.1-1.0.416.el7.x86_64.rpm is 54.06 MB; this is larger than GitHub's recommended maximum file size of 50.00 MB
remote: warning: File 运维文档/8-SRE/SRE -实践汇总.pptx is 50.34 MB; this is larger than GitHub's recommended maximum file size of 50.00 MB

## 删除大文件
首先查找到你想要删除的文件
    
        git rev-list --objects --all ｜ grep 删除的文件名
样例
        
        $ git rev-list --objects --all | grep DXSHURENPOC03
        d00491fd7e5bb6fa28c517a0bb32b8b506539d4d 运维文档/9-对外项目故障/浦发12.9日日志错误log/log03/DXSHURENPOC03.dataman-marathon.log
        5a9df5b93fbef8b99b8a2996474a26b620a9174b 运维文档/9-对外项目故障/浦发12.9日日志错误log/log03/DXSHURENPOC03.dataman-borgsphere.log
        37312d7967ec78f5e9be5930f10c81450a087285 运维文档/9-对外项目故障/浦发12.9日日志错误log/log03/DXSHURENPOC03.dataman-marathon.log
        e139f47bfca1d194aa62eb5d485bcaaf2da7a79d 运维文档/9-对外项目故障/浦发12.9日日志错误log/log03/DXSHURENPOC03.dataman-mesos-master.log
        51a91a942224632648e0e1aab8925179fb82d33e 运维文档/9-对外项目故障/浦发12.9日日志错误log/log03/DXSHURENPOC03.dataman-zookeeper.log
