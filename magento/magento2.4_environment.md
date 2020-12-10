# 安装 Magento 2.4 版本框架

## 安装条件

|依赖|版本要求|
|---|---|
|memory| >2GB of RAM |
|Apache|2.4|
|Nginx|1.x|
|PHP| > 7.2|
|MySQL|>=8.0|

## 环境准备

> 备注：因为本地和虚拟机中运行的项目 mysql 都是 mysql 5.7.32 ，本地运行的 magento2.3 的环境是 php7.2，所以决定在 虚拟机 中进行安装

### **PHP v7.4**

因为当前使用的是 laravel 中的 homestead 包，里面包含 php7.4，当前使用的是 php7.2，所以现在切换 php 版本

```bash
# 删除以前建立的软连接
sudo rm /usr/bin/php
# 建立软连接
sudo ln -s /usr/bin/php7.4 /usr/bin/php 
# 查看当前版本
php -v
```

### **MySQL v8.0**

```bash
# 安装 docker
sudo snap install docker
# 安装 mysql
sudo docker pull mysql(默认安装最新版本)
# 关闭虚拟机的 mysql 连接
sudo service mysql stop
# 运行 mysql
sudo docker run --name magento -p 3306:3306 -e MYSQL_ROOT_PASSWORD=admin  -d 
–name：给新创建的容器命名，此处命名为 magento
-e：配置信息，此处配置 mysql 的 root 用户的登陆密码
-p：端口映射，此处映射主机 3306 端口到容器 pwc-mysql 的 3306 端口
-d：成功启动容器后输出容器的完整ID .magento 
# 查看容器运行状态
sudo docker ps
# 查看 docker ip地址
ifconfig | grep inet
# 查看docker 运行的mysql ip地址
sudo docker inspect --format '{{ .NetworkSettings.IPAddress }}' magento
```

### **Magento v2.4** 

> 需要安装 **Elasticsearch**

``` bash
#在虚拟机中安装 Elasticsearch
# 安装 elasticsearch
sudo apt-get install  elasticsearch
# 启动 Elasticsearch
sudo service elasticsearch start
# 查看 elasticsearch 进程
ps aux | grep elasticsearch
# 如果 elasticsearch 不能正常启动，可以先删除再重新进行安装
sudo apt-get remove elasticsearch
```

### 使用 **Navicat** 连接 **mysql** 数据库

1. 此处的 host 为 mysql 镜像在 homestead 虚拟机中 docker 中的 ip 地址
  ![mysql_1](/images/mysql_1.png)

2. 此处的 host 为虚拟机的 ip 地址
  ![mysql_2](/images/mysql_2.png)

记得在虚拟机中配置项目访问地址(在 **Homestead.yaml 文件**)

```yaml
sites:
    - map: mayMagento.local
    to: /home/vagrant/Code/mayMagento2
```

### 虚拟机中配置访问地址（在 **/etc/hosts 文件**）

```conf
192.168.10.10  mayMagento.local
```

## 安装

### 在虚拟机中通过命令行，执行下列命令

```bash
# 切换到项目目录
cd /home/vagrant/Code/mayMagento2
# 执行下面三条指令
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
chmod u+x bin/magento
```

### 虚拟机项目目录，命令行执行：

```bash
php bin/magento setup:install \
--base-url=http://maymagento.local \
--db-host=172.17.0.2(此 ip 地址为 虚拟机中 docker 中 mysql 镜像的 ip 地址) \
--db-name=may_magento2 \
--db-user=root \
--db-password=admin \
--admin-firstname=admin \
--admin-lastname=admin \
--admin-email=bingqing0822@163.com \
--admin-user=admin \
--admin-password=admin123 \
--language=en_US \
--currency=USD \
--timezone=America/Chicago \
--use-rewrites=1
```

### 如果 **GD2** 报错，信息如下：

![gd2_error](/images/gd2.png)

修改 **vendor\magento\framework\Image\Adapter\Gd2.php** 文件中的 *validateURLScheme* 方法，改为：

```php
private function validateURLScheme(string $filename) : bool   {
    $allowed_schemes = ['ftp', 'ftps', 'http', 'https'];
    $url = parse_url($filename);
    if ($url && isset($url['scheme']) && !in_array($url['scheme'], $allowed_schemes) && !file_exists($filename)) {
        return false;
    }

    return true;   
}
```

### 报错 **memory_limit** 相关错误，如下图：

![memory_limit](/images/memory_limit.png)

```bash
# 可以在 php 命令后增加 -d memory_limit=-1
php -d memory_limit=-1  bin/magento s:d:c
```

### 成功

![success](/images/success_magento2.4.png)

### 检测

> 浏览器进行访问 http://maymagento.local/

![check](/images/check_1.jpg)

在项目目录 **/var/log/exception.log** 文件中查看错误信息

## **mysql 8.0** 相关问题报错

![error_1](/images/mysql_3.jpg)

解决方案(mysql8.0 认证方式不一样)：

```bash
# docker 下修改mysql配置文件
# 1、查找要修改的镜像
sudo docker ps
# 2、进入要修改的镜像
sudo docker exec -it 容器ID /bin/bash
# 3、进入修改目录
cd /etc/mysql
# 4、安装 vim
apt-get update
apt-get install vim
# 5、修改配置文件
vi my.cnf
# 在 【mysqld】下增加 default_authentication_plugin=mysql_native_password
# 6、给 my.cnf 赋权限
chmod 777 my.cnf
# 7、退出容器（二者选一）
# 1）Ctrl + d 退出并停止容器；
# 2）Ctrl + p + q 退出并在后台运行容器；
Ctrl + d
# 8、重启docker容器
sudo docker restart 容器ID
```

## 找不到 **host** 报错，信息如下：

![error_2](/images/mysql_4.jpg)

解决方案：

```bash
# 在本地服务器运行下面的命令
# 建立socket转发通道(将homestead虚拟机里的docker里的mysql镜像转发到本地)
ssh -C -f -N -g -L 3307:172.17.0.2:3306 vagrant@192.168.10.10
# 查看 3307 端口连接情况
lsof -i :3307
# 连接MySQL
mysql -h 127.0.0.1 -P 3307 -u root -p 
```

备注：**此方案问题尚未解决**

