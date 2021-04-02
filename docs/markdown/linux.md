linux 对外端口访问
firewall-cmd --add-port=5601/tcp --permanent
firewall-cmd --reload
firewall-cmd --query-port=5601/tcp

查看端口的进程占用号

netstat -nltp | grep 9100
