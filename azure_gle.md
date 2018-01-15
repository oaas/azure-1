
## Grafana + Logstash + ElasticSearch 收集Azure SG的日志

1 . 目前G，L，E三个组件已经安装在srv-lz-log1服务器上，ssh登陆机器查看对应的进程
  
 ```
 # 可以看到elasticsearch和grafana已经在运行
 root@srv-lz-log1:~# ps aux | grep elastic
elastic+   1485  0.4 14.1 4189048 498220 ?      Ssl  01:32   0:14 /usr/bin/java -Xms256m -Xmx2g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -server -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -Dio.netty.noUnsafe=true -XX:+HeapDumpOnOutOfMemoryError -Des.path.home=/usr/share/elasticsearch -cp /usr/share/elasticsearch/lib/elasticsearch-5.0.0-alpha5.jar:/usr/share/elasticsearch/lib/* org.elasticsearch.bootstrap.Elasticsearch -p /var/run/elasticsearch/elasticsearch.pid -Edefault.path.logs=/var/log/elasticsearch -Edefault.path.data=/var/lib/elasticsearch -Edefault.path.conf=/etc/elasticsearch
root       7140  0.0  0.0  12920   936 pts/0    S+   02:21   0:00 grep --color=auto elastic
root@srv-lz-log1:~# ps aux | grep grafana
grafana    1337  0.0  0.8 568464 29668 ?        Ssl  01:32   0:00 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini --pidfile=/var/run/grafana/grafana-server.pid cfg:default.paths.logs=/var/log/grafana cfg:default.paths.data=/var/lib/grafana cfg:default.paths.plugins=/var/lib/grafana/plugins
root       7153  0.0  0.0  12920  1012 pts/0    S+   02:21   0:00 grep --color=auto grafana

 ```
 
 
 2 . 配置logstash日志输入源。目前是azureblob作为日志输入源，因此需要配置azureblob的account， access_key, 和container
 
 ```
 root@srv-lz-log1:~# vim /etc/logstash/conf.d/logstash.conf
 input {
  azureblob
  {
    storage_account_name => "mystorageaccount"  # 更改account_name
    storage_access_key => "VGhpcyBpcyBhIGZha2Uga2V5Lg=="   # 更改access_key
    container => "insights-logs-networksecuritygroupflowevent"  # 更改container
    codec => "json"
    ......
    ......
 ```  
 
 3 . 启动logstash
 
 ```
 root@srv-lz-log1:~# logstash -f /etc/logstash/conf.d/logstash.conf
 ```
 
 4 . 如若lotstash运行成功，并无任何报错，浏览器访问grafana的dashboard，去创建图标，看是否有数据产生
 
 ```
 http://40.118.134.160:3000/
 ```