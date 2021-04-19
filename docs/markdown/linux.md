linux 对外端口访问
firewall-cmd --add-port=5601/tcp --permanent
firewall-cmd --reload
firewall-cmd --query-port=5601/tcp

查看端口的进程占用号

netstat -nltp | grep 9100

mysql 操作

sudo systemctl start mysqld.service
sudo systemctl restart mysqld.service

解压

tar  -xvf

seata  启动

nohup ./seata-server.sh -h 192.168.153.129 -p 9001 >log.out 2>&1 &
