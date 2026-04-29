# 智学教学助手平台部署说明文档（华为云）

## 一、服务器准备

### 1. 注册华为云账号
访问官网：https://www.huaweicloud.com/

### 2. 创建云服务器 ECS
进入控制台 → 搜索【ECS】→ 创建弹性云服务器

充值金额建议：5元

建议配置：
- 计费模式：按需计费
- 可用区：随机分配
- CPU架构：鲲鹏计算
- 实例筛选：1vCPUs、1GiB
- 镜像：OpenEuler
- 关闭"开启主机安全防护"
- 磁盘类型：通用型SSD、10GiB
- 关闭"开启备份"
- 默认虚拟私有云（如没有则跳转至附录问题1）
- 公网带宽选择"按流量计费" 带宽100Mbit
- 登陆凭证选择密码（一定要记住）
- ![q](C:\Users\18380\Desktop\q.png)

### 3. 服务器开机

成功购买服务器后，点击远程登录就可以打开服务器进行后续操作

![2](C:\Users\18380\Desktop\2.png)

### 4. 远程连接服务器

ssh root@你的公网IP 如下显示则说明配置成功

![image-20260429114534672](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429114534672.png)

---

## 二、环境部署

本项目采用：
- 后端：Spring Boot
- 前端：React
- 数据库：MySQL
- 反向代理：Nginx

---

## 三、JDK环境安装（重点）

大部分同学需要安装jdk1.8，但是有部分同学运行大模型会导致版本不能满足需求，但是不用担心，版本很容易切换，如果需要换高版本的操作见附录问题2

### 安装 JDK 1.8

``` 
yum -y install java-1.8*
```

### 验证
``` 
java -version
```

---

## 四、MySQL 安装与配置

### 安装
``` 
sudo dnf install -y mysql-server
```

### 启动
``` 
sudo systemctl start mysqld
sudo systemctl enable mysqld
```

### 进行验证是否启动成功

``` 
sudo systemctl status mysqld
```

### 初始化

``` 
mysql_secure_installation
// 按照下图进行配置，密码一定要是你本地的MySQL密码，否则要改后端配置类！
```

![image-20260429120422702](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429120422702.png)

![image-20260429120209232](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429120209232.png)

### 开启远程访问

``` 
mysql -uroot -p
USE mysql;
UPDATE user SET host='%' WHERE user='root';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### 修改配置

先去找 my.cnf 文件的位置

``` 
find / -name my.cnf
```

![image-20260429120646919](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429120646919.png)

跳转到当前文件夹并进行修改文件让所有路径都可访问

``` 
cd /etc/my.cnf
ls
vim /etc/my.cnf.d/mysql-server.cnf
```

![image-20260429120809264](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429120809264.png)

``` 
添加：（一定要手敲！）
bind-address = 0.0.0.0
```

![3](C:\Users\18380\Desktop\3.png)

### 保存退出

按：

```
ESC
:wq
```

------

### 重启 MySQL（必须）

``` 
systemctl restart mysqld
```

---

## 五、数据库导入

接下来进入到navicat（如果是其他可视化界面也是可以的，只要接口对全都对）进行测试连接发现连接成功

### 用 Navicat 连接

填写：（如果连接超时则跳转至附录问题3）

| 项目   | 内容                        |
| ------ | --------------------------- |
| 主机   | 1.92.84.157（你自己的主机） |
| 端口   | 3306                        |
| 用户名 | root                        |
| 密码   | 你的MySQL密码               |

![image-20260429121148049](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429121148049.png)

接下来就可以进行可视化界面操作后台数据库了，导入成功即可

---

## 六、后端部署（Spring Boot）

项目打包：成功后项目目录会有target/XXX.jar

``` 
mvn clean package -DskipTests
```

![屏幕截图 2026-04-29 111812](C:\Users\18380\Pictures\Screenshots\屏幕截图 2026-04-29 111812.png)

![image-20260429121610947](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429121610947.png)

上传至服务器：
scp target/*.jar root@服务器IP:/root/

``` 
scp target/TAP_SpringBoot-0.0.1-SNAPSHOT.jar root@1.92.84.157:/root/
```

![image-20260429121831043](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429121831043.png)

服务器运行：
``` 
java -jar TAP_SpringBoot-0.0.1-SNAPSHOT.jar
```

后台运行：
``` 
nohup java -jar TAP_SpringBoot-0.0.1-SNAPSHOT.jar > app.log 2>&1 &
```

---

## 七、前端部署（React）

### 打包前必须检查

### API地址要改成服务器IP

你代码里可能是：

```
http://localhost:8080/api
```

必须改成：

```
http://你的服务器IP:8080/api
```

否则上线后：前端连不上后端

![image-20260429123014543](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429123014543.png)

### 1、安装依赖（第一次必须）

```
npm install
```

### 2、打包

```
npm run build
```

### 3、打包成功标志

生成一个目录：

```
build/
```

### 4、上传到服务器

在本地执行：scp -r build root@你的服务器IP:/root/

```
scp -r build root@1.92.84.157:/root/
```

![image-20260429123722680](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429123722680.png)

---

## 八、Nginx 部署

### 1、服务器安装 Nginx

```
dnf install -y nginx
```

### 2、修改配置

```
vim /etc/nginx/nginx.conf
```

找到 `server`，改成👇

```
server {
    listen 80;
    server_name localhost;

    location / {
        root /root/build;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://localhost:8080;
    }
}
```

### 3、启动

```
systemctl start nginx
systemctl enable nginx
```

### 4、访问

浏览器打开：http://你的服务器IP

```
http://1.92.84.157:5171
```

![image-20260429140516173](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429140516173.png)看到页面 = 成功

---

## 九、总结

完成前后端分离项目部署，实现上线运行。

## 十、附录

### 问题1：没有虚拟私有云

点击新建私有云默认配置立即添加即可

![image-20260429141249986](C:\Users\18380\AppData\Roaming\Typora\typora-user-images\image-20260429141249986.png)

### 问题2：需要切换高版本JDK

#### 安装 JDK 21

``` 
dnf install -y java-latest-openjdk java-latest-openjdk-devel
```

#### 切换版本

``` 
alternatives --config java
```

### 问题3：MySQL访问连接超时

```
华为云控制台
→ 云服务器 ECS
→ 你的服务器
→ 安全组
→ 添加入方向规则
```

------

### 添加规则：

| 项目 | 填写                |
| ---- | ------------------- |
| 协议 | TCP                 |
| 端口 | 3306                |
| 来源 | 0.0.0.0/0（测试用） |

保存
