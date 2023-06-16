安装 MySQL 服务器：

```sh
sudo yum install mysql-server
```

启动 MySQL 服务：

```sh
sudo systemctl start mysqld
```

运行 MySQL 安全脚本来设置 root 密码和其他安全选项：

```sh
sudo mysql_secure_installation
```

验证 MySQL 服务是否正在运行：

```sh
sudo systemctl status mysqld
```

退出mysql

```sh
exit
```

