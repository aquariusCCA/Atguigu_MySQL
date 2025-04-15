# 1. 安装前说明
## 1.1 Linux 系统及工具的准备
- 安装并启动号俩台虚拟机：Centos 7

- 掌握克隆虚拟机的操作
    - mac 地址的修改
    - 主机名的修改
    - ip 地址的修改
    - uuid 的修改

- 安装 Xshell 和 Xftp 等访问 Centos 系统的工具
- Centos6 和 Centos7 在 MySql 的使用中的区别
    ```
    1、防火墙:6是iptables，7是firewalld
    2、启动服务的命令:6是service，7是systemctl
    ```

## 1.2 查看是否安装过 MySQL
##### 如果你是用rpm安装, 检查一下RPM PACKAGE：
```shell
rpm -qa | grep -i mysql # -i 忽略大小写
```

##### 检查mysql service：
```shell
systemctl status mysqld.service
```

##### 如果存在mysql-libs的旧版本包，显示如下：
![圖片名稱](./images/2402456-20220611162449369-1493301150.png "游標顯示")

##### 如果不存在mysql-lib的版本，显示如下：
![圖片名稱](./images//2402456-20220611162449140-1684510169.png "游標顯示")

## 1.3 MySQL的卸载

##### 关闭 mysql 服务
```shell
systemctl stop mysqld.service
```

##### 查看当前 mysql 安装状况
```shell
rpm -qa | grep -i mysql
# 或
yum list installed | grep mysql
```

##### 卸载上述命令查询出的已安装程序
```shell
yum remove mysql-xxx mysql-xxx mysql-xxx mysqk-xxxx
```
> 务必卸载干净，反复执行rpm -qa | grep -i mysql确认是否有卸载残留

##### 删除 mysql 相关文件
- 查找相关文件
    ```shell
    find / -name mysql
    ```
- 删除上述命令查找出的相关文件
    ```shell
    rm -rf xxx
    ```

##### 删除 my.cnf
```shell
rm -rf /etc/my.cnf
```

# 2. MySQL的Linux版安装
## 2.1 MySQL的4大版本
> **4大版本**
> - **MySQL Community Server 社区版本:** 开源免费，自由下载，但不提供官方技术支持，适用于大多数普通用户。
> - **MySQL Enterprise Edition 企业版本:** 需付费，不能在线下载，可以试用30天。提供了更多的功能和更完备的技术支持，更适合于对数据库的功能和可靠性要求较高的企业客户。
> - **MySQL Cluster 集群版:** 开源免费。用于架设集群服务器，可将几个MySQL Server封装成一个Server。需要在社区版或企业版的基础上使用。
> - **MySQL Cluster CGE 高级集群版:** 需付费

> 此外，官方还提供了 MySQL Workbench （GUITOOL）一款专为 MySQL 设计的 ER/数据库建模工具 。它是著名的数据库设计工具DBDesigner4的继任者。MySQLWorkbench又分为两个版本，分别是 社区版（MySQL Workbench OSS）、 商用版 （MySQL WorkbenchSE）。

## 2.2 下载MySQL指定版本
> **下载地址:** 官网：https://www.mysql.com

### 2.2.1 Windows下安裝 MySQL

- 打开官网，点击DOWNLOADS，然后，点击 MySQL Community(GPL) Downloads
    ![圖片名稱](./images/2402456-20220611162448868-267339285.png "游標顯示")

- 点击 MySQL Community Server
    ![](./images/2402456-20220611162448609-816157438.png "")

- 在General Availability(GA) Releases中选择适合的版本
    - 如果安装Windows 系统下MySQL ，推荐下载 MSI安装程序 ；点击 Go to Download Page 进行下载即可
    ![](./images/2402456-20220611162448338-1546620417.png "")

> **Windows下的MySQL安装有两种安装程序**
> - mysql-installer-web-community-8.0.25.0.msi 下载程序大小：2.4M；安装时需要联网安装组件。
> - mysql-installer-community-8.0.25.0.msi 下载程序大小：435.7M；安装时离线安装即可。推荐。

### 2.2.2 Linux系统下安装MySQL的几种方式
> Linux系统下安装软件的常用三种方式：

#### 方式1：rpm命令
> 使用 rpm 命令安装扩展名为 ".rpm" 的软件包。
> - .rpm 包的一般格式
> - ![](./images/2402456-20220611162448049-1794950942.png "")

#### 方式2：yum命令
> 需联网，从互联网获取的yum源，直接使用yum命令安装。

#### 方式3：编译安装源码包
> 针对 **tar.gz** 这样的压缩格式，要用tar命令来解压；如果是其它压缩格式，就使用其它命令。

#### Linux系统下安装MySQL，官方给出多种安装方式
- ![](./images/2402456-20220611162447813-163902580.png "")
- 这里不能直接选择CentOS 7系统的版本，所以选择与之对应的 Red Hat Enterprise Linux
- https://downloads.mysql.com/archives/community/ 直接点Download下载RPM Bundle全量
包。包括了所有下面的组件。不需要一个一个下载了。
- ![](./images/2402456-20220611162447539-635246543.png "")

#### 下载的tar包，用压缩工具打开
- ![](./images/2402456-20220611162447041-1175116571.png "")

## 2.3 CentOS7下检查MySQL依赖
### 检查/tmp临时目录权限（必不可少）
> 由于mysql安装过程中，会通过mysql用户在/tmp目录下新建tmp_db文件，所以请给/tmp较大的权限。执行 ：

```shell
chmod -R 777 /tmp
```

![](./images/2402456-20220611162446775-1577670458.png "")

### 安装前，检查依赖
```shell
rpm -qa|grep libaio
rpm -qa|grep net-tools
```

- 如果存在libaio包如下：
    ![](./images/2402456-20220611162446557-417375354.png "")

- 如果存在net-tools包如下：
    ![](./images/2402456-20220611162446319-1636905172.png "")

> 如果不存在需要到centos安装盘里进行rpm安装。安装linux如果带图形化界面，这些都是安装好的。

#### ✅ `rpm -qa | grep libaio`

這行的意思是：

- `rpm -qa`：列出所有已安裝的 RPM 套件（q = query，a = all）。
- `grep libaio`：過濾出名稱中包含 `libaio` 的套件。

##### 🔍 為什麼檢查 `libaio`？
MySQL（特別是使用 RPM 安裝的版本）在執行期間需要 **非同步 I/O 函式庫（`libaio`）** 來提高磁碟存取效能。如果系統沒有安裝 `libaio`，MySQL 安裝時會出錯，或啟動失敗。

👉 所以這個指令是「**確認是否已安裝 libaio**」的動作。  
若查無結果，通常會建議執行：
```bash
sudo yum install libaio
```

---

#### ✅ `rpm -qa | grep net-tools`

這行的意思是：

- `rpm -qa`：列出所有安裝的套件。
- `grep net-tools`：過濾出名稱中含有 `net-tools` 的套件。

##### 🔍 為什麼檢查 `net-tools`？
`net-tools` 是一組網路相關的工具套件，包含常見的指令像是：

- `ifconfig`：檢查網卡資訊
- `netstat`：查看連接埠、網路狀態
- `route`：查看路由表

這些工具對於伺服器管理者在設定、排錯 MySQL 網路連線時（例如：確認 MySQL 是否在監聽指定埠口）很有用。

👉 所以這個指令是「**確認是否具備網路工具**」。若沒有的話可以執行：
```bash
sudo yum install net-tools
```

---

#### ✅ 小結

| 指令 | 用途 | 重要原因 |
|------|------|-----------|
| `rpm -qa | grep libaio` | 檢查是否安裝 `libaio` 函式庫 | MySQL 執行依賴 |
| `rpm -qa | grep net-tools` | 檢查是否安裝網路工具套件 | 排查連線問題 |

## 2.4 CentOS7下MySQL安装过程

### 将安装程序拷贝到/opt目录下
> 在mysql的安装文件目录下执行：（必须按照顺序执行）
```shell
rpm -ivh mysql-community-common-8.0.25-1.el7.x86_64.rpm 
rpm -ivh mysql-community-client-plugins-8.0.25-1.el7.x86_64.rpm 
rpm -ivh mysql-community-libs-8.0.25-1.el7.x86_64.rpm 
rpm -ivh mysql-community-client-8.0.25-1.el7.x86_64.rpm 
rpm -ivh mysql-community-server-8.0.25-1.el7.x86_64.rpm
```

> ⚠️在安裝 `mysql-community-server` 前，**必須先確保系統上已安裝它所依賴的套件（依賴環境）**，否則會在安裝時發生錯誤，導致安裝失敗。

- rpm 是 Redhat Package Manage 缩写，通过 RPM 的管理，用户可以把源代码包装成以rpm为扩展名的文件形式，易于安装。
- **-i:** install 安装软件包
- **-v:** verbose 提供更多的详细信息输出
- **-h:** hash 软件包安装的时候列出哈希标记 (和 -v 一起使用效果更好)，展示进度条

![](./images/2402456-20220611162446070-840246302.png "")

#### 在你安裝 `mysql-community-server` 前，**必須先確保系統上已安裝它所依賴的套件（依賴環境）**，否則會在安裝時發生錯誤，導致安裝失敗。
##### 更具體地說明：

MySQL 的 RPM 安裝方式，這幾個 `.rpm` 套件之間是有依賴關係的。舉例來說：

- `mysql-community-server` **依賴**：
  - `mysql-community-client`
  - `mysql-community-libs`
  - `mysql-community-client-plugins`
  - `mysql-community-common`

所以你必須先按順序安裝前面這些元件（common → plugins → libs → client），**才能**安裝 `mysql-community-server`。

如果你直接跳過前面的步驟去安裝 `mysql-community-server`，但系統缺少這些依賴元件，就會報出錯誤，例如：

```bash
error: Failed dependencies:
    mysql-community-client is needed by mysql-community-server-8.0.25-1.el7.x86_64
    mysql-community-common is needed by mysql-community-client...
```

---

##### 對應這句的重點：

| 原文 | 說明 |
|------|------|
| 没有检查mysql依赖环境 | 指的是沒有事先安裝好需要的依賴（common, libs, client等） |
| 安装mysql-community-server会报错 | 最終安裝 `server` 這個主套件時會因依賴缺失而失敗 |

---

##### 建議補充步驟：
在安裝前，可以執行下列指令來檢查相關依賴是否存在：

```bash
rpm -qa | grep mysql
```

或使用這些工具檢查系統依賴：

```bash
rpm -qpR mysql-community-server-8.0.25-1.el7.x86_64.rpm
```

這會列出這個套件所需要的依賴清單。

---


## 2.5 安装过程截图
- ![](./images/2402456-20220611162445748-554154726.png "")

### 安装过程中可能的报错信息：
- ![](./images/2402456-20220611162445501-374187605.png "")
> 一个命令：`yum remove mysql-libs` 解决，清除之前安装过的依赖即可

###  查看MySQL版本
```shell
mysql --version 
#或
mysqladmin --version
```

### 服务的初始化
> 为了保证数据库目录与文件的所有者为 mysql 登录用户，如果你是以 root 身份运行 mysql 服务，需要执行下面的命令初始化：

```shell
mysqld --initialize --user=mysql
```

> 说明： --initialize 选项默认以 **安全** 模式来初始化，则会为 root 用户生成一个密码并将该密码标记为过期，登录后你需要设置一个新的密码。生成的临时密码会往日志中记录一份。

#### 查看密码：
```shell
cat /var/log/mysqld.log
```

![](./images/2402456-20220611162445259-687002224.png "")

> root@localhost: 后面就是初始化的密码

### 启动MySQL，查看状态

```shell
#加不加.service后缀都可以 
启动：systemctl start mysqld.service 
关闭：systemctl stop mysqld.service 
重启：systemctl restart mysqld.service 
查看状态：systemctl status mysqld.service
```

> mysqld 这个可执行文件就代表着 MySQL 服务器程序，运行这个可执行文件就可以直接启动一个服务器进程。

![](./images/2402456-20220611162444966-437773086.png "")

#### 查看进程:
```shell
ps -ef | grep -i mysql
```

### 查看MySQL服务是否自启动
```shell
systemctl list-unit-files|grep mysqld.service
```

![](./images/2402456-20220611162444718-1981029802.png "")

> 默认是 enabled。

- 如不是enabled可以运行如下命令设置自启动
    ```shell
    systemctl enable mysqld.service
    ```
    ![](./images/2402456-20220611162444458-801535320.png "")

- 如果希望不进行自启动，运行如下命令设置
    ```shell
    systemctl disable mysqld.service
    ```
    ![](./images/2402456-20220611162444216-1919150912.png "")

# 3. MySQL登录
## 3.1 首次登录
> 通过 `mysql -hlocalhost -P3306 -uroot -p` 进行登录，在Enter password：录入初始化密码

## 3.2 修改密码
- 当查看数据时show databases，报错;因为初始化密码默认是过期的，所以查看数据库会报错
- 修改密码：
    ```sql
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
    ```

- 5.7版本之后（不含5.7），mysql加入了全新的密码安全机制。设置新密码太简单会报错。
    ![](./images/2402456-20220611162443980-1702235916.png "")

- 改为更复杂的密码规则之后，设置成功，可以正常使用数据库了
    ![](./images/2402456-20220611162443759-531322726.png "")

## 3.3 设置远程登录
> **当前问题:**
> - 在用SQLyog或Navicat中配置远程连接Mysql数据库时遇到如下报错信息，这是由于Mysql配置了不支持远程连接引起的。
> - ![](./images/2402456-20220611162443456-2142832033.png "")
> - 要讓 MySQL 支援遠程連接，需完成以下幾個步驟來進行設定與授權。

### ✅ 1. 修改 MySQL 設定檔（`my.cnf` 或 `my.ini`）

##### Linux（通常為 `/etc/mysql/my.cnf` 或 `/etc/my.cnf`）

打開設定檔，找到以下段落：
```ini
[mysqld]
bind-address = 127.0.0.1
```

將 `127.0.0.1` 改為：
- `0.0.0.0` 表示允許所有 IP 連線
- 或填寫特定 IP 例如 `192.168.1.100` 表示只允許這個 IP 連線

範例：
```ini
[mysqld]
bind-address = 0.0.0.0
```

儲存並關閉。

##### Windows（通常為 `C:\ProgramData\MySQL\MySQL Server X.X\my.ini`）

做法一樣：找到 `bind-address`，設為 `0.0.0.0` 或指定 IP。

---

### ✅ 2. 開啟防火牆（或雲端安全群組）端口

MySQL 預設使用的 port 是 `3306`。

你需要在伺服器上：
- **開啟防火牆 port 3306**
- **雲端平台（如 AWS、GCP、阿里雲）需在安全群組中允許對應 IP 的 3306 port 通訊**

---

### ✅ 3. 建立允許遠端連接的 MySQL 使用者帳號

登入 MySQL：
```bash
mysql -u root -p
```

建立一個允許遠端的帳號或修改原有帳號授權：

##### 方法一：允許任意 IP 登入（不安全）
```sql
CREATE USER 'your_user'@'%' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON *.* TO 'your_user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

##### 方法二：僅允許特定 IP
```sql
CREATE USER 'your_user'@'192.168.1.50' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON *.* TO 'your_user'@'192.168.1.50' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

---

### ✅ 4. 重新啟動 MySQL 服務

##### Linux：
```bash
sudo systemctl restart mysql
```
或
```bash
sudo service mysql restart
```

##### Windows：
可在「服務管理」中重新啟動 MySQL。

---

### ✅ 5. 測試連線（如使用 SQLyog、Navicat）

填寫連線資訊：
- 主機名（Host）：伺服器 IP
- 使用者名稱
- 密碼
- port：3306

---

### ✅ 附註安全提醒

- **不要將 root 使用者暴露給遠端**，請創建單獨的帳號。
- 若一定要允許 root 遠端連接，務必設定強密碼並限制 IP。
- 建議使用 VPN 或 SSH 隧道加密連線。

---

## 3.4 Linux下修改配置

### 在Linux系统MySQL下测试：
```sql
use mysql;
select Host,User from user;
```
- ![](./images/2402456-20220611162442397-1961528809.png "")
> 可以看到root用户的当前主机配置信息为localhost。

### 修改Host为通配符%
> Host列指定了允许用户登录所使用的IP，比如 user=root Host=192.168.1.1。这里的意思就是说root用户只能通过192.168.1.1的客户端去访问。 user=root Host=localhost，表示只能通过本机客户端去访问。而 % 是个通配符 ，如果Host=192.168.1.%，那么就表示只要是IP地址前缀为“192.168.1.”的客户端都可以连接。如果 Host=% ，表示所有IP都有连接权限。

> ⚠️在生产环境下不能为了省事将host设置为%，这样做会存在安全问题，具体的设置可以根据生|产环境的IP进行设置。

```sql
update user set host = '%' where user ='root';
```
Host 设置了 **%** 后便可以允许远程访问。

Host 修改完成后记得执行 **flush privileges**使配置立即生效：

![](./images/2402456-20220611162442178-2128616185.png "")

### 测试
- 如果是 MySQL5.7 版本，接下来就可以使用SQLyog或者Navicat成功连接至MySQL了。

##### 如果是 MySQL8 版本，连接时还会出现如下问题：
![](./images/2402456-20220611162441910-1060340471.png "")

> 配置新连接报错：错误号码 2058，分析是 mysql 密码加密方法变了。

- **解决方法一：** 升级远程连接工具版本

- **解决方法二：**
    ```sql
    ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'abc123';
    ```

# 4. MySQL8的密码强度评估（了解）
從這裡開始

# 疑惑