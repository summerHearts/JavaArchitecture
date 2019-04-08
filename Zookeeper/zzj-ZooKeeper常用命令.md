
# ZooKeeper常用命令

意义 | 命令
---|---
启动ZK服务:  | bin/zkServer.sh start 
查看ZK服务状态: | bin/zkServer.sh status 
停止ZK服务:   |    bin/zkServer.sh stop 
重启ZK服务:   |   bin/zkServer.sh restart
进入zk服务    |   zkCli.sh -server 127.0.0.1:2181
显示根目录下、文件  |  ls  /
显示根目录下、文件  |  ls2 /
创建文件，并设置初始内容  |    create /zk "test" 
获取文件内容  |   get /zk
修改文件内容  |   set /zk "zkbak"
删除文件   |  delete /zk
退出客户端  |  quit