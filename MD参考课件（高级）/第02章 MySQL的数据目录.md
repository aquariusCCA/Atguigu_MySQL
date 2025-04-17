# 1. MySQL8的主要目录结构
## 1.1  ✅ MySQL 8 的主要目录结构說明（以 Linux 環境為主）
> 當你安裝好 **MySQL 8** 之後，整體目錄結構會依據安裝方式（例如是使用 **apt/yum** 安裝、**Docker** 安裝，或是 **tar.gz** 二進位壓縮包）有所不同。不過在 **大多數 Linux 系統** 下，MySQL 8 的目錄結構會包含以下幾個重要目錄：

| 目錄路徑 | 功能說明 |
|----------|----------|
| `/etc/mysql/` 或 `/etc/my.cnf` | 配置檔目錄或主配置檔案位置（如 `my.cnf`） |
| `/usr/bin/` 或 `/usr/sbin/` | 存放 MySQL 可執行指令，如 `mysql`、`mysqld`、`mysqldump` 等 |
| `/var/lib/mysql/` | **資料庫實體檔案儲存位置（data directory）**，包含所有資料表與索引 |
| `/var/log/mysql/` 或 `/var/log/` | 存放錯誤日誌、查詢日誌、慢查詢日誌等 |
| `/usr/share/mysql/` | 儲存字元集、SQL 範本、錯誤訊息檔案等資源 |
| `/run/mysqld/` 或 `/tmp/` | 儲存 mysqld 的 PID 檔與 socket 檔案（如 `mysqld.sock`） |

---

### 1.1.1 🔍 如何使用 `find` 指令找出 mysql 相關目錄

你可以使用如下指令查找 MySQL 所在目錄：
```bash
sudo find / -name mysql
```

常見輸出結果可能會包括：
```
/etc/mysql
/usr/bin/mysql
/usr/lib/mysql
/usr/share/mysql
/var/lib/mysql
/var/log/mysql
```

---

### 1.1.2 📦 額外補充：使用 `mysqld --verbose --help | grep -A 1 "Default options"` 查詢預設目錄

這個指令可用來查看 MySQL 預設會讀取哪些配置檔：
```bash
mysqld --verbose --help | grep -A 1 "Default options"
```

輸出範例：
```
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf
```

---

### 1.1.3 🛠 若是 tar.gz 安裝版 MySQL，可能的目錄如下：

| 路徑 | 說明 |
|------|------|
| `/usr/local/mysql` | 安裝主目錄 |
| `/usr/local/mysql/bin` | 可執行指令 |
| `/usr/local/mysql/data` | 資料庫儲存目錄 |
| `/usr/local/mysql/support-files` | 預設配置檔與啟動腳本 |

---

## 1.2 📁 MySQL 数据库文件的存放路径（data directory）

> MySQL 的所有數據（包含資料庫、資料表、日誌、索引等）都儲存在稱為 **數據目錄（data directory）** 的路徑中。

---

### 1.2.1 🔹 預設資料路徑：`/var/lib/mysql/`

這個資料夾中會包含：
- 每個資料庫一個子資料夾（資料庫名）
- 每個資料表對應的 `.frm`、`.ibd` 等檔案（視儲存引擎而定）
- 系統資料庫（如 `mysql`, `performance_schema`, `information_schema`）

---

### 1.2.2 🔍 如何查詢 MySQL 實際的數據目錄路徑？

在 MySQL 客戶端中執行：
```sql
SHOW VARIABLES LIKE 'datadir';
```

輸出範例：
```
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
```

---

### 1.2.3 🧱 MySQL 數據目錄中常見的內容

假設你有一個名為 `testdb` 的資料庫，其在 `/var/lib/mysql/testdb/` 目錄下的內容可能包括：

| 檔案類型 | 檔案範例 | 說明 |
|----------|----------|------|
| `.frm` | `users.frm` | 資料表的結構定義（表結構）<br>（MySQL 8.0 後已棄用） |
| `.ibd` | `users.ibd` | InnoDB 獨立表空間文件，儲存數據與索引 |
| `.MYD` | `users.MYD` | MyISAM 引擎資料文件 |
| `.MYI` | `users.MYI` | MyISAM 引擎索引文件 |
| `ibdata1` | — | InnoDB 的共享表空間（如未使用 `innodb_file_per_table`） |
| `ib_logfile0`, `ib_logfile1` | — | InnoDB 重做日誌 |
| `mysql.ibd` | — | MySQL 系統資料庫表空間 |

> 📌 注意：從 **MySQL 5.6 以後**，開啟了 `innodb_file_per_table`（預設為 ON），每個表都會有自己的 `.ibd` 文件。

---

### 1.2.4 ⚙️ 變更資料目錄的方法（僅限進階用戶）

如果你想將資料儲存於其他磁碟或自訂目錄，例如 `/data/mysql/`，可透過以下步驟修改：

1. 停止 MySQL 服務：
   ```bash
   sudo systemctl stop mysqld
   ```

2. 修改配置檔（如 `/etc/my.cnf`）：
   ```ini
   [mysqld]
   datadir=/data/mysql
   socket=/data/mysql/mysql.sock
   ```

3. 移動資料並調整權限：
   ```bash
   sudo mv /var/lib/mysql /data/mysql
   sudo chown -R mysql:mysql /data/mysql
   ```

4. 重啟服務並確認：
   ```bash
   sudo systemctl start mysqld
   mysql -u root -p
   SHOW VARIABLES LIKE 'datadir';
   ```

---

## 1.3 📂 MySQL 相關命令目錄：`/usr/bin` 和 `/usr/sbin`

MySQL 安裝完成後，其 **命令工具程式（可執行檔）** 主要分佈在這兩個系統目錄中：

| 目錄路徑 | 主要用途 |
|----------|----------|
| `/usr/bin/` | 用戶層級常用指令，例如：`mysql`、`mysqldump`、`mysqladmin` 等 |
| `/usr/sbin/` | 系統層級指令，例如：`mysqld`（MySQL 伺服器啟動主程式）等 |

---

### 1.3.1 📁 MySQL 安裝目錄中的 `bin/` 資料夾（適用於 tar.gz 安裝方式）

若你是從 **二進位壓縮包安裝（tar.gz）**，則所有命令會集中在：
```
/usr/local/mysql/bin/
```
或自定義安裝路徑下的 `bin/` 目錄。

---

### 1.3.2 🧰 常見 MySQL 可執行命令工具說明

| 命令工具 | 所在目錄 | 說明 |
|-----------|------------|------|
| `mysql` | `/usr/bin/` | MySQL 的命令列用戶端工具，用於連線和操作資料庫 |
| `mysqld` | `/usr/sbin/` | MySQL 伺服器核心程式，負責提供服務 |
| `mysqld_safe` | `/usr/bin/` or `/usr/local/mysql/bin/` | 一種以「安全方式」啟動 mysqld 的 wrapper script |
| `mysqladmin` | `/usr/bin/` | 管理工具，可用於查看伺服器狀態、關閉服務等 |
| `mysqldump` | `/usr/bin/` | 備份工具，用於導出資料庫成 SQL 檔 |
| `mysqlimport` | `/usr/bin/` | 批次匯入資料至資料表 |
| `mysqlshow` | `/usr/bin/` | 查看資料庫、表格等結構資訊 |
| `mysqlbinlog` | `/usr/bin/` | 解析 binlog（二進位日誌）內容 |
| `mysql_upgrade` | `/usr/bin/` | 用於資料庫升級後的自動維護 |

---

### 1.3.3 ⚠️ bin 目錄 ≠ 資料目錄（`datadir`）

這裡特別說明：

- `bin/` 目錄 👉 放的是 **可執行檔案與命令工具**
- `datadir`（如 `/var/lib/mysql/`）👉 放的是 **資料庫檔案與日誌**

兩者需嚴格區分，**一個是程式操作層級，一個是資料儲存層級**。

---

### 1.3.4 ✅ 如何查找命令工具實際路徑

可以使用 `which` 或 `whereis` 命令查詢，例如：
```bash
which mysql
# /usr/bin/mysql

whereis mysqld
# mysqld: /usr/sbin/mysqld
```

---

## 1.4 配置文件目錄

> MySQL 的配置文件（通常是 `my.cnf` 或 `my.ini`）用於定義 MySQL 伺服器的啟動參數、資料目錄位置、緩衝區大小、日誌路徑等設定。

### 常見配置文件位置：

| 路徑 | 說明 |
|------|------|
| `/etc/mysql/` | Debian/Ubuntu 系統預設的配置資料夾，常見的配置檔為 `/etc/mysql/my.cnf`，該檔案可能會引用其他設定檔（例如 `/etc/mysql/mysql.conf.d/` 目錄下的檔案）。 |
| `/etc/my.cnf` 或 `/etc/my.cnf.d/` | Red Hat/CentOS 系統常見位置，通常這裡是主要配置檔所在位置。 |
| `/usr/share/mysql-8.0/` | 安裝目錄中包含範本設定檔（如 `my-default.cnf`）與預設 SQL 指令及幫助文件等。 |
| `/usr/local/mysql/etc/` | 若 MySQL 是從官方原始碼安裝，則可能會在這個目錄中找設定檔。 |
| `~/.my.cnf` | 使用者層級的設定檔，可以自定義連線參數，僅對目前使用者生效（例如自動登入的帳密設定）。 |

### 檢查實際使用的配置文件：

可以透過以下指令確認 MySQL 當前實際讀取的設定檔：

```bash
mysql --help | grep -A 1 "Default options"
```

或使用：

```bash
mysqld --verbose --help | grep -A 1 "Default options"
```

這會列出系統啟動時會依序查找的 `my.cnf` 位置。

---

# 2. 数据库和文件系统
## 2.1 🔧 MySQL 資料庫與檔案系統的關係

MySQL 是一個資料庫管理系統，但它本身並不直接管理磁碟資料的儲存細節，而是 **透過作業系統的檔案系統（File System）來儲存資料**。

當你在 MySQL 中建立一個資料表或插入資料時，實際上資料會根據你使用的 **儲存引擎（Storage Engine）** 被寫入到作業系統的檔案中。

這些檔案就儲存在你的硬碟中，而作業系統（如 Windows、Linux）會透過檔案系統（如 NTFS、EXT4）來負責這些檔案的讀寫與管理。

<img width="300" height="300" src="./images/數據庫系統調用數據.png" />

---

### 📦 不同儲存引擎如何使用檔案系統

#### 1. InnoDB 儲存引擎
- 預設儲存引擎，支援 **交易（Transaction）**、**外鍵（Foreign Key）** 等功能。
- 資料和索引一般儲存在 `.ibd` 檔案中（如果開啟 `innodb_file_per_table` 參數）。
- 系統也會使用一個共享的資料空間（如 `ibdata1`）來管理 metadata 或 redo log。

##### 範例：
假設你建立一個資料庫 `shop`，並建立一張名為 `products` 的資料表：

```sql
CREATE TABLE products (
  id INT PRIMARY KEY,
  name VARCHAR(100)
) ENGINE=InnoDB;
```

🔹 在資料夾 `/var/lib/mysql/shop/` 下，你可能會看到：

```
products.ibd      ← 儲存表格資料與索引
```

#### 2. MyISAM 儲存引擎
- 傳統引擎，不支援交易，但速度快，適合查詢多的場景。
- 一張表會對應**三個檔案**：

| 檔案 | 說明 |
|------|------|
| `.frm` | 表的結構定義（InnoDB 也會有） |
| `.MYD` | MyISAM 資料檔（MyData） |
| `.MYI` | MyISAM 索引檔（MyIndex） |

##### 範例：
```sql
CREATE TABLE customers (
  id INT,
  name VARCHAR(100)
) ENGINE=MyISAM;
```

🔹 在 `/var/lib/mysql/shop/` 中你會看到：

```
customers.frm      ← 表結構
customers.MYD      ← 資料內容
customers.MYI      ← 索引內容
```

---

### 🧠 總結關係圖（簡化）

```
MySQL 資料表
   ↓
儲存引擎（InnoDB 或 MyISAM）
   ↓
將資料寫入磁碟檔案（如 .ibd、.MYD）
   ↓
透過作業系統的檔案系統（如 EXT4）來實際管理磁碟資料
```

---

### 🎓 小結

- ✅ **MySQL 資料是以檔案的形式存放在作業系統的檔案系統上**。
- ✅ 不同的儲存引擎使用不同的檔案格式與架構。
- ✅ 作業系統的檔案系統負責實際的資料讀寫與磁碟空間管理。

--- 

## 2.2 查看默认数据库
查看一下在我的计算机上当前有哪些数据库：`show databases;`
```shell
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```
> 可以看到有4个数据库是属于MySQL自带的系统数据库。


### 2.2.1 mysql 資料庫
#### ✅ 一、什麼是 `mysql` 數據庫？

`mysql` 數據庫是 **MySQL 系統自帶的內建資料庫**，用來 **儲存系統級別的設定與管理資訊**，它不是給使用者用來儲存業務資料的，而是讓 MySQL 本身能夠正常運作。

---

#### ✅ 二、主要內容分類

| 資料表 | 功能簡介 |
|--------|----------|
| `user` | 儲存所有用戶帳號與其權限，例如誰能連線、能操作哪些資料庫 |
| `db`、`tables_priv` | 管理特定資料庫或資料表的使用權限 |
| `procs_priv` | 管理儲存過程（Stored Procedure）的權限 |
| `time_zone`、`time_zone_name` | 儲存時區設定資訊 |
| `help_topic`、`help_category` | 提供 `HELP` 指令查詢的內容 |
| `event` | 儲存計畫排程事件（EVENT）的定義 |
| `general_log`、`slow_log` | 儲存操作日誌與慢查詢記錄（需開啟 LOG 功能） |

---

#### ✅ 三、實際操作例子

##### 📌 範例：查詢 MySQL 所有用戶帳號

```sql
SELECT User, Host FROM mysql.user;
```

📘 解釋：
這表示你要看目前有哪些帳號能登入 MySQL，以及這些帳號能從哪個主機登入（本機或遠端）。

---

##### 📌 範例：查詢某個帳號的權限

```sql
SHOW GRANTS FOR 'myuser'@'localhost';
```

這會列出 `myuser` 帳號擁有的所有權限，例如可以對哪個資料庫執行 SELECT、INSERT 等操作。

---

##### 📌 範例：查看時區表內容（如果有初始化）

```sql
SELECT * FROM mysql.time_zone LIMIT 5;
```

這可以查到系統支援的時區資料（前提是你已用 `mysql_tzinfo_to_sql` 工具匯入時區）。

---

#### ✅ 四、補充說明：使用建議

- `mysql` 資料庫裡的資料**千萬不要隨便修改**，否則可能導致系統登入失敗或功能異常。
- 權限管理（GRANT、REVOKE）基本上都會對 `mysql` 資料庫做出變動。

---

#### ✅ 五、小結

| 你記住這句話就夠了！|
|------------------|
|「`mysql` 資料庫是 MySQL 自己的『設定大腦』，用來記錄用戶、權限、時區、事件、日誌等資訊。」|

---

### 2.2.2 information_schema 數據庫
> MySQL 系统自带的数据库，这个数据库保存着 MySQL 服务器 **维护的所有其他数据库的信息** ，比如有哪些表、哪些视图、哪些触发器、哪些列、哪些索引。这些信息并不是真实的用户数据，而是一些描述性信息，有时候也称之为 元数据 。在系统数据库 `information_schema` 中提供了一些以 `innodb_sys` 开头的表，用于表示内部系统表。

```shell
mysql> USE information_schema;
Database changed
mysql> SHOW TABLES LIKE 'innodb_sys%';
+--------------------------------------------+
| Tables_in_information_schema (innodb_sys%) |
+--------------------------------------------+
| INNODB_SYS_DATAFILES            |
| INNODB_SYS_VIRTUAL             |
| INNODB_SYS_INDEXES             |
| INNODB_SYS_TABLES             |
| INNODB_SYS_FIELDS             |
| INNODB_SYS_TABLESPACES           |
| INNODB_SYS_FOREIGN_COLS          |
| INNODB_SYS_COLUMNS             |
| INNODB_SYS_FOREIGN             |
| INNODB_SYS_TABLESTATS           |
+--------------------------------------------+
10 rows in set (0.00 sec)
```

#### ✅ 一、什麼是 `information_schema` 數據庫？

`information_schema` 是 MySQL 自帶的「虛擬系統資料庫」，**不儲存真實資料，而是用來查詢資料庫、資料表、欄位、索引、約束等資訊**。

可以把它當作「MySQL 的目錄或總表」，是開發人員、資料庫管理員常用來查詢系統結構的地方。

---

#### ✅ 二、常見資料表與說明

| 資料表名稱 | 說明 |
|------------|------|
| `SCHEMATA` | 系統中有哪些資料庫 |
| `TABLES` | 所有資料庫中的資料表 |
| `COLUMNS` | 每張資料表的欄位資訊（欄位名稱、型別等） |
| `STATISTICS` | 資料表的索引資訊 |
| `KEY_COLUMN_USAGE` | 主鍵、外鍵相關資訊 |
| `VIEWS` | 所有的視圖 |
| `TRIGGERS` | 所有的觸發器 |
| `ROUTINES` | 儲存程序（Stored Procedures）與函式（Functions） |
| `INNODB_SYS_*` | InnoDB 引擎的內部結構資訊（通常進階使用者才會查） |

---

#### ✅ 三、簡單查詢範例

##### 📌 查詢目前所有資料庫名稱

```sql
SELECT SCHEMA_NAME FROM information_schema.SCHEMATA;
```

---

##### 📌 查詢某個資料庫中的所有資料表

```sql
SELECT TABLE_NAME 
FROM information_schema.TABLES 
WHERE TABLE_SCHEMA = 'mydb';
```

（這裡的 `'mydb'` 是你的資料庫名稱）

---

##### 📌 查詢某資料表的欄位資訊（例如：欄位名稱、資料型別）

```sql
SELECT COLUMN_NAME, DATA_TYPE 
FROM information_schema.COLUMNS 
WHERE TABLE_NAME = 'employees' AND TABLE_SCHEMA = 'mydb';
```

---

##### 📌 查詢某資料表的所有索引

```sql
SELECT * 
FROM information_schema.STATISTICS 
WHERE TABLE_NAME = 'employees' AND TABLE_SCHEMA = 'mydb';
```

---

##### 📌 查詢所有觸發器

```sql
SELECT TRIGGER_NAME, EVENT_MANIPULATION, EVENT_OBJECT_TABLE 
FROM information_schema.TRIGGERS;
```

---

#### ✅ 四、補充說明

- `information_schema` 是一個只讀資料庫，不能對其做 `INSERT`、`UPDATE` 或 `DELETE` 操作。
- 很適合用在：
  - 資料庫結構分析
  - 動態生成 SQL（例如：做資料匯出工具）
  - 程式自動化處理資料表結構變動

---

#### ✅ 五、小結

| 你可以這樣理解：|
|-----------------|
|「`information_schema` 就是 MySQL 的黃頁，記錄了整個資料庫系統的『結構』，例如哪裡有什麼表、表裡有什麼欄位、有什麼索引等資訊。」|

---

### 2.2.3 performance_schema 數據庫
> MySQL 系统自带的数据库，这个数据库里主要保存MySQL服务器运行过程中的一些状态信息，可以用来 监控 MySQL 服务的各类性能指标 。包括统计最近执行了哪些语句，在执行过程的每个阶段都花费了多长时间，内存的使用情况等信息。

#### ✅ 一、什麼是 `performance_schema` 數據庫？

`performance_schema` 是 MySQL 自帶的 **性能監控資料庫**，記錄 MySQL 執行中的各種指標與資源使用狀況，例如：

- 查詢語句耗時多長？
- 哪個帳號連線最多？
- 哪個 SQL 最慢？
- 哪個表鎖最常出現？
- 內存與執行緒的使用情況如何？

> 🧠 與 `information_schema` 記錄的是「結構資訊」不同，`performance_schema` 記錄的是「動態行為」與「效能指標」。

---

#### ✅ 二、常見資料表分類

| 資料表範例 | 說明 |
|------------|------|
| `events_statements_*` | 記錄 SQL 語句的執行時間與狀態 |
| `events_waits_*` | 記錄等候鎖（如 I/O、表鎖）所花費的時間 |
| `events_stages_*` | 記錄 SQL 執行各階段所花時間（如解析、執行） |
| `threads` | 記錄目前執行緒資訊 |
| `accounts`、`users`、`hosts` | 統計不同使用者/主機的效能資訊 |
| `memory_summary_*` | 內存使用統計 |
| `file_summary_*` | 檔案存取相關耗時統計 |
| `setup_*` | 控制哪些資訊要收集，例如啟用哪些監控功能 |

---

#### ✅ 三、簡單查詢範例

##### 📌 1. 查詢最耗時的前 5 條 SQL 語句

```sql
SELECT digest_text, COUNT_STAR, SUM_TIMER_WAIT/1000000000000 AS total_time_sec
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 5;
```

🔍 解釋：
- `digest_text`: SQL 模板
- `COUNT_STAR`: 執行次數
- `SUM_TIMER_WAIT`: 總耗時（奈秒 → 除以 1 兆轉成秒）

---

##### 📌 2. 查詢目前有哪些執行緒正在跑（像 `SHOW PROCESSLIST`）

```sql
SELECT * 
FROM performance_schema.threads
WHERE PROCESSLIST_STATE IS NOT NULL;
```

---

##### 📌 3. 查詢哪些資料表等待最久（例如鎖表）

```sql
SELECT OBJECT_SCHEMA, OBJECT_NAME, COUNT_STAR, SUM_TIMER_WAIT/1000000000000 AS wait_time_sec
FROM performance_schema.table_io_waits_summary_by_table
ORDER BY wait_time_sec DESC
LIMIT 5;
```

---

##### 📌 4. 查詢記憶體分配最多的模組

```sql
SELECT EVENT_NAME, SUM_NUMBER_OF_BYTES/1024/1024 AS memory_mb
FROM performance_schema.memory_summary_global_by_event_name
ORDER BY memory_mb DESC
LIMIT 5;
```

---

#### ✅ 四、補充說明

- `performance_schema` 需要在 MySQL 啟動時就開啟（`performance_schema=ON`），某些版本預設是啟用的。
- 雖然可以查資料，但這個資料庫本身不能手動新增/刪除資料。
- 查出來的資料都是「分析用途」，不適合用在應用程式邏輯中。

---

#### ✅ 五、小結比較

| 系統資料庫 | 用途 | 重點 |
|-------------|------|------|
| `mysql` | 系統設定 | 儲存帳號、權限、事件 |
| `information_schema` | 結構資訊 | 儲存資料表、欄位、索引定義等「元資料」 |
| `performance_schema` | 性能監控 | 儲存查詢行為、等待事件、內存耗用等「效能資料」 |

---

#### ✅ 六、記憶法

📌「`performance_schema` 是幫你看 *MySQL 目前在忙什麼、慢在哪裡* 的地方！」

---

### 2.2.4 sys 數據庫
> MySQL 系统自带的数据库，这个数据库主要是通过 视图 的形式把 `information_schema` 和 `performance_schema` 结合起来，帮助系统管理员和开发人员监控 MySQL 的技术性能。

#### ✅ 一、什麼是 `sys` 數據庫？
> `sys` 數據庫就是 MySQL 官方為了讓 **系統管理員與開發人員更方便使用**，基於 `information_schema` 和 `performance_schema` 所建立的一組「**預先整理好的視圖（View）**」。

`sys` 是 MySQL 自帶的一個資料庫（從 MySQL 5.7 起引入），它：

- ✅ 封裝了許多 **複雜查詢**（來自 `information_schema` 和 `performance_schema`）
- ✅ 使用 **視圖（View）** 提供更簡潔、可讀性高的方式來查看效能資料
- ✅ 方便 DBA、工程師快速做 **效能分析、問題診斷、瓶頸排查**

📌 換句話說：  
> 「`performance_schema` 給你很多數據，但 `sys` 幫你整理好；就像原始數據 vs. 分析報表。」

---

#### ✅ 二、常見 `sys` 資料表（視圖）功能分類

| 視圖名稱 | 說明 |
|----------|------|
| `host_summary`、`user_summary` | 每個主機/用戶的執行統計 |
| `statement_analysis` | 語句分析，幫你找出最常用或最慢的 SQL |
| `io_by_thread_by_latency` | 哪些執行緒花最多時間在 I/O 上 |
| `innodb_buffer_stats_by_table` | 各資料表在 InnoDB buffer pool 中的使用情況 |
| `wait_classes_global_by_avg_latency` | 哪一類等待時間最長（例如鎖、I/O） |
| `processlist` | 類似 `SHOW PROCESSLIST`，但更清晰 |
| `ps_check_lost_instrumentation` | 檢查 `performance_schema` 是否有遺失設定 |

---

#### ✅ 三、簡單查詢範例

##### 📌 1. 查詢目前執行中耗時最長的 SQL

```sql
SELECT * 
FROM sys.statement_analysis 
ORDER BY avg_latency DESC 
LIMIT 5;
```

---

##### 📌 2. 查詢哪些資料表的 I/O 最慢

```sql
SELECT * 
FROM sys.io_by_file_by_latency 
ORDER BY total_latency DESC 
LIMIT 5;
```

---

##### 📌 3. 查詢哪些帳號最常執行操作

```sql
SELECT * 
FROM sys.user_summary 
ORDER BY statements DESC 
LIMIT 5;
```

---

##### 📌 4. 查詢哪種類型的等待最花時間（鎖？I/O？）

```sql
SELECT * 
FROM sys.wait_classes_global_by_avg_latency;
```

---

##### 📌 5. 查詢 InnoDB 中哪些資料表佔用最多緩衝區

```sql
SELECT * 
FROM sys.innodb_buffer_stats_by_table 
ORDER BY pages DESC 
LIMIT 5;
```

---

#### ✅ 四、優點與補充說明

- 不需要自己寫複雜 SQL，`sys` 視圖幫你包裝好。
- 適合日常效能巡檢（Performance Health Check）。
- `sys` 是只讀的，你不能修改資料，但可以查詢視圖。

---

#### ✅ 五、小結對比四大系統資料庫

| 資料庫 | 作用 | 特點 |
|--------|------|------|
| `mysql` | 系統設定 | 儲存帳號、權限、事件等 |
| `information_schema` | 結構資料 | 所有資料表、欄位、索引的描述資料（元資料） |
| `performance_schema` | 性能監控 | 詳細記錄執行狀況、等待、資源耗用等 |
| `sys` | 分析報表 | 整合上兩者資訊，透過視圖簡化查詢，幫助診斷與監控 |

---

#### ✅ 六、記憶方式（口訣）

```
mysql 是「系統腦」
information_schema 是「資料架構表」
performance_schema 是「行為監控器」
sys 是「報表分析王」
```

---

## 2.3 数据库在文件系统中的表示
當我們談到「**数据库在文件系统中的表示**」時，其實就是在探討 MySQL 如何在硬碟中儲存每個資料庫與資料表。這對於理解 MySQL 的底層運作、進行備份或還原操作很有幫助。

---

### 2.3.1 🗂️ 数据库在文件系统中的表示

MySQL 安裝完成後，會在系統中設定一個「**數據目錄（data directory）**」，這個目錄是用來儲存所有資料庫資料的實體位置。這個路徑可以透過 `my.cnf` 或 `my.ini` 中的 `datadir` 參數查到，例如：

```ini
[mysqld]
datadir=/var/lib/mysql
```

你可以用以下指令查看你的 MySQL 資料目錄：
```bash
SHOW VARIABLES LIKE 'datadir';
```

---

### 2.3.2 📁 数据目录下的内容補充

在這個目錄下，你會看到很多子目錄與檔案，像這樣：

```
/var/lib/mysql/
├── ibdata1
├── ib_logfile0
├── ib_logfile1
├── performance_schema/
├── mysql/
├── sys/
├── temp/       <-- 自己建立的資料庫
├── test/
└── ibtmp1
```

- `information_schema/`：這是虛擬資料庫，不會有實體目錄。
- `mysql/`：MySQL 的核心系統資料，如使用者權限等。
- `performance_schema/` 和 `sys/`：系統統計資訊與診斷用的資料庫。
- `temp/`：使用者自己建立的資料庫，每個資料庫會對應一個同名的目錄。
- `ibdata1`, `ib_logfile*`：這些是 InnoDB 的共享表空間與日誌檔。

---

### 2.3.3 🧪 以 temp 数据库为例

#### ✅ 在 MySQL 5.7 中

在 MySQL 5.7，預設情況下使用「**共享表空間**」，所有的 InnoDB 資料表的資料和索引都儲存在 `ibdata1` 檔案中。資料表的 `.frm` 檔案會儲存在該資料庫目錄中。

假設我們建立了一個資料表：
```sql
CREATE DATABASE temp;
USE temp;
CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100));
```

則在檔案系統中會看到：
```
/var/lib/mysql/temp/
├── db.opt              <-- 資料庫選項（字符集等）
├── users.frm           <-- 表結構（格式定義）
```

如果是 MyISAM 引擎：
```
├── users.MYD           <-- 資料檔
├── users.MYI           <-- 索引檔
```

---

#### ✅ 在 MySQL 8.0 中

從 MySQL 8.0 開始，啟用「**每表表空間（file-per-table）**」為預設行為，InnoDB 會將每個表的資料儲存在獨立的 `.ibd` 檔案中。

繼續以剛才的範例：
```
/var/lib/mysql/temp/
├── db.opt
├── users.frm （已廢棄）
├── users.ibd   <-- 表的資料與索引都儲存在這
```

> ✅注意：從 MySQL 8.0 開始，`*.frm` 格式已不再使用，改為儲存在 `data dictionary`（資料字典）中，這些資訊不再是以獨立檔案形式存在，而是內建於系統內部。
> - 👉 表的結構資料不再用外部的 .frm 檔案存起來，而是統一存在 MySQL 自己的內部資料表裡。
> - 你可以把這個想像成：
> - 把所有表格的設計圖「統一集中管理」起來，保存在 MySQL 自己的資料庫裡，這樣更安全、更穩定，也更好管理。
---

### 2.3.4 🧠 小結

| 版本 | 表結構檔案 | 表資料檔案 | 表索引檔案 | 額外檔案 |
|------|-------------|-------------|-------------|-------------|
| MySQL 5.7 | `*.frm` | `ibdata1`（InnoDB）或 `*.MYD`（MyISAM） | `ibdata1` 或 `*.MYI` | `db.opt` |
| MySQL 8.0 | 改為儲存在系統內部 | `*.ibd`（InnoDB） | 同上 | `db.opt` |

---

## 2.4 表在文件系统中的表示
### 2.4.1 InnoDB存储引擎模式
#### 2.4.1.1 表結構
> 为了保存表结构，InnoDB 在数据目录下对应的数据库子目录下创建了一个专门用于描述表结构的文件

> `InnoDB` 引擎在建立表格時會在資料庫目錄下建立一個 `.frm` 檔案用來保存**表結構的描述**。不過，從 MySQL 8.0 開始，這個儲存結構已經有所改變。因此我會分別說明：

---

##### ✅ 一、MySQL InnoDB 存儲引擎表結構說明（MySQL 5.x 時代）

在 MySQL 5.x 或之前的版本，**每當你使用 InnoDB 建立一個表格，會建立以下幾個檔案：**

###### 1. `.frm` 文件（**表結構描述**）

- 儲存表的欄位資訊、資料型別、主鍵、索引等。
- 位於 `data directory` 的該資料庫子目錄下，例如：
  ```
  /usr/local/mysql/data/atguigu/test.frm
  ```

- 雖然你可以看到這個檔案，但它是 **二進位格式（binary）**，無法直接編輯。

###### 2. `.ibd` 文件（**表資料與索引**）

- 如果你啟用了 `innodb_file_per_table=ON`（預設為 ON），每個 InnoDB 表會有自己的 `.ibd` 檔案，用來儲存 **資料與索引**。
- 範例路徑：
  ```
  /usr/local/mysql/data/atguigu/test.ibd
  ```

---

###### 🔧 範例操作（MySQL 5.x）

```sql
-- 進入資料庫
USE atguigu;

-- 建立一個表格
CREATE TABLE test (
  c1 INT,
  c2 VARCHAR(50)
);
```

在此情況下，MySQL 在系統中產生的內容如下：

| 檔案名     | 功能說明                   |
|------------|----------------------------|
| `test.frm` | 記錄表格結構（欄位、主鍵） |
| `test.ibd` | 儲存資料與索引             |

---

##### ✅ 二、MySQL 8.0 的變化（更現代化）

MySQL 8.0 移除了 `.frm` 檔案，改為將表結構資訊儲存在 **系統資料字典（Data Dictionary）中**，這個字典儲存在 `InnoDB` 的共享表空間中。

###### ❌ 不再產生 `.frm` 檔案  
取而代之的是以下幾種檔案組合：

| 檔案類型            | 說明                                      |
|---------------------|---------------------------------------------|
| `mysql.ibd`         | 儲存資料字典、元資料                       |
| `table_name.ibd`    | 每張表的資料與索引（前提是 `innodb_file_per_table = ON`） |

🔍 你可以用以下 SQL 指令查詢表結構：

```sql
SELECT * FROM information_schema.tables WHERE table_name = 'test';
```

---

##### 🧠 總結對照表

| MySQL 版本 | 表結構儲存方式 | 資料與索引儲存方式 |
|-------------|------------------|--------------------|
| 5.x 及以下  | `.frm` 文件        | `.ibd`（每表一檔） 或共享 `ibdata1` |
| 8.0 起      | 系統資料字典        | `.ibd`（預設）     |

---

##### ✅ 額外補充：如何查看資料存放位置？

你可以使用以下 SQL 指令來確認資料與表檔案是否獨立儲存：

```sql
SHOW VARIABLES LIKE 'innodb_file_per_table';
```

如果是 `ON`，代表每個表有自己的 `.ibd` 檔案。

---

如果你想，我也可以示範如何用命令列查看資料庫實際儲存的檔案，例如：

```bash
ls -lh /usr/local/mysql/data/atguigu/
```

---

#### 2.4.1.2 表中數據和索引

##### 系统表空间（system tablespace）
###### ✅ 一、什麼是 InnoDB 系統表空間（System Tablespace）

####### 🧱 系統表空間的定義
系統表空間是一個用來**儲存系統級別資料**的特殊表空間，包含：

- 系統資料字典（例如：表的定義、索引結構）
- undo log（回滾日誌）
- 某些內部 temporary table
- 在特定版本中，也包含使用者表的資料（如果 `innodb_file_per_table=OFF`）

---

###### ✅ 二、預設檔案名稱：`ibdata1`

####### 📁 在文件系統上的表現：
當你第一次啟動 MySQL 並使用 InnoDB 時，預設會在資料目錄下產生一個叫：

```bash
ibdata1
```

的檔案，大小為 12MB，自動擴充（autoextend）。

範例路徑：
```
/usr/local/mysql/data/ibdata1
```

這個檔案就是「**系統表空間在磁碟上的映射**」。

---

###### ✅ 三、如何自定義系統表空間的結構（多個檔案）

你可以在 `my.cnf` 配置文件中修改這樣的設定：

```ini
[mysqld]
innodb_data_file_path=data1:512M;data2:512M:autoextend
```

####### 🔍 解釋：
| 設定           | 意思                                      |
|----------------|---------------------------------------------|
| `data1:512M`   | 建立一個固定大小 512MB 的檔案 `data1`       |
| `data2:512M:autoextend` | 建立一個初始大小為 512MB 且會自動擴充的 `data2` |

這樣 MySQL 啟動後會產生這兩個檔案，並共同組成系統表空間。

---

###### ✅ 四、版本差異與作用變化

| MySQL 版本       | 系統表空間的內容                                       |
|------------------|--------------------------------------------------------|
| MySQL 5.5.7 以前 | 所有表的資料與索引、資料字典、undo log 都放這裡       |
| MySQL 5.5.7~5.6.6| 支援 `innodb_file_per_table`，可以選擇分開儲存         |
| MySQL 5.6.6 起   | 預設開啟 `innodb_file_per_table`，系統表空間儲存 undo log 與字典等系統資料 |
| MySQL 8.0 起     | 完全移除 `.frm` 檔案，資料字典也改存在系統表空間中的 ibd 檔案或共享空間中 |

---

###### ✅ 五、簡單實例演示

####### 📋 步驟 1：修改 my.cnf（模擬多個檔案）

```ini
[mysqld]
innodb_data_file_path=data1:512M;data2:512M:autoextend
```

####### 📋 步驟 2：啟動 MySQL 後產生檔案

```bash
ls -lh /usr/local/mysql/data/
```

輸出可能會看到：

```
data1      # 固定 512MB
data2      # 初始 512MB，可自動變大
```

這兩個檔案就構成了「系統表空間」。

---

###### ✅ 六、如何驗證表資料是否進入系統表空間？

```bash
SHOW VARIABLES LIKE 'innodb_file_per_table';
```

- 如果是 `OFF`，則所有 InnoDB 表都放進 `ibdata1`（系統表空間）
- 如果是 `ON`（預設），每個表會有自己的 `.ibd` 檔案，而系統表空間只存 undo log 等系統資訊

---

###### ✅ 七、補充：你什麼時候會看到 `ibdata1` 很大？

- 長期使用 InnoDB、刪除大量資料卻沒重建表空間 → **ibdata1 不會縮小**
- 解法：必須 `mysqldump` + `drop database` + `restart` 才能清掉

---

###### 🧠 總結

| 名稱        | 說明                                  |
|-------------|-----------------------------------------|
| `ibdata1`   | 預設系統表空間檔案，儲存系統資料與日誌 |
| `innodb_file_per_table` | 控制是否將表資料儲存在單獨的 `.ibd` 檔中        |
| 可多檔設定   | 可透過 `innodb_data_file_path` 指定多個檔案形成表空間 |

---

##### 独立表空间(file-per-table tablespace)
###### ✅ 一、什麼是「獨立表空間」？

**獨立表空間（file-per-table tablespace）** 是 InnoDB 在 MySQL 中的一種儲存策略：

> 為**每一張表格單獨建立一個儲存檔案（`.ibd` 檔）**，用來儲存該表的資料和索引。

---

###### ✅ 二、啟用方式

從 **MySQL 5.6.6 起，`innodb_file_per_table=ON` 為預設值**，所以你只要用 MySQL 5.6.6 以後的版本，且沒修改過這個設定，就是啟用「獨立表空間」。

```sql
SHOW VARIABLES LIKE 'innodb_file_per_table';
```

若查詢結果為 `ON`，表示啟用中。

---

###### ✅ 三、儲存檔案結構（MySQL 5.x）

假設你在 `atguigu` 資料庫中建立一個表格：

```sql
USE atguigu;

CREATE TABLE test (
  id INT PRIMARY KEY,
  name VARCHAR(50)
);
```

則在你的資料目錄（例如 `/usr/local/mysql/data/atguigu/`）中會產生兩個檔案：

| 檔名        | 說明                         |
|-------------|------------------------------|
| `test.frm`  | 儲存表的結構定義（欄位、索引） |
| `test.ibd`  | 儲存表的資料與索引資料         |

####### 📌 注意：
- `.frm` 為 MySQL 5.x 使用的表結構檔案（非純文字，無法直接開啟）
- `.ibd` 為 InnoDB 表資料與索引的獨立檔案（file-per-table）

---

###### ✅ 四、MySQL 8.0 中的變化

MySQL 8.0 移除了 `.frm` 檔案，把表結構資訊也整合進 `.ibd` 檔中，這表示：

> `test.ibd` 同時包含：**表結構定義 + 資料 + 索引**

範例：
```bash
# 路徑為 /var/lib/mysql/atguigu/
test.ibd  ← 包含所有資訊（MySQL 8.0 起）
```

---

###### ✅ 五、為什麼要用獨立表空間？

####### ✅ 優點
| 優點 | 說明 |
|------|------|
| 資料易於管理 | 每張表都有獨立檔案，可以單獨備份、移除 |
| ibdata1 不再無限變大 | 刪除表格也能釋放磁碟空間 |
| 支援 `DISCARD TABLESPACE` | 可以丟棄/導入 `.ibd` 檔案 |

####### ❌ 缺點
| 缺點 | 說明 |
|------|------|
| 產生大量 `.ibd` 檔案 | 當表很多時，檔案數量會變多 |
| 多表操作稍微複雜 | 跨表的表空間操作需留意每個 `.ibd` 狀態 |

---

###### ✅ 六、簡單範例：檢查、建立與查看表檔案

####### 📋 1. 建立表格
```sql
CREATE DATABASE atguigu;
USE atguigu;

CREATE TABLE test (
  id INT PRIMARY KEY,
  name VARCHAR(100)
);
```

####### 📋 2. 在 Linux 中查看檔案

```bash
cd /var/lib/mysql/atguigu/
ls -lh test.*
```

輸出結果會看到（MySQL 8.0）：

```
-rw-r----- 1 mysql mysql  X MB test.ibd   # 儲存 test 表全部資訊
```

MySQL 5.x 則會看到：

```
-rw-r----- 1 mysql mysql   9 KB test.frm
-rw-r----- 1 mysql mysql  X MB test.ibd
```

---

###### ✅ 七、如何確認表格使用的是獨立表空間？

```sql
SELECT TABLE_NAME, TABLESPACE_NAME
FROM information_schema.tables
WHERE TABLE_SCHEMA = 'atguigu';
```

若 `TABLESPACE_NAME` 欄為 `innodb_file_per_table` 預設表空間名，則表示該表有自己 `.ibd` 檔案。

---

###### 🧠 小結

| 名稱                     | 說明                                               |
|--------------------------|----------------------------------------------------|
| 獨立表空間 `.ibd`        | 每張表都有自己的 `.ibd`，儲存資料 + 索引         |
| `innodb_file_per_table`  | 控制是否啟用獨立表空間（MySQL 5.6.6+ 預設為 ON） |
| `.frm`（MySQL 5.x）       | 表結構儲存在 `.frm`（MySQL 8.0 起改整合進 .ibd）  |

---

##### 系统表空间与独立表空间的设置
###### ✅ 一、什麼是「表空間（Tablespace）」？

在 MySQL 的 InnoDB 存儲引擎中，**表空間是儲存資料（數據、索引、undo log 等）在磁碟上的邏輯容器**。不同的表空間設定，會影響表格資料最終儲存在硬碟的方式。

---

###### ✅ 二、主要三種 InnoDB 表空間類型

| 表空間類型       | 說明                                                                 |
|------------------|----------------------------------------------------------------------|
| ✅ 系統表空間     | 預設名稱為 `ibdata1`，早期版本會將所有表的資料、索引都存在這裡            |
| ✅ 獨立表空間     | 每個表建立 `.ibd` 檔案儲存自己資料與索引（需 `innodb_file_per_table=ON`） |
| ✅ 通用表空間     | MySQL 5.7+ 引入，可自訂資料與索引放哪個表空間（較進階，常用於大型系統）     |

本題聚焦的是前兩種：**系統表空間與獨立表空間的設定切換**

---

###### ✅ 三、系統表空間 vs 獨立表空間（重點對比）

| 對比項目           | 系統表空間（System）               | 獨立表空間（File-per-table）     |
|--------------------|-------------------------------------|-----------------------------------|
| 設定參數           | `innodb_file_per_table=0`          | `innodb_file_per_table=1`（預設）|
| 儲存方式           | 所有表共用 `ibdata1` 檔案          | 每表對應 `.ibd` 檔案              |
| 空間回收           | 刪表後不會釋放空間（需手動重建）   | 刪表即釋放對應 `.ibd` 空間        |
| 管理便利性         | 備份困難，所有表混在一個檔案中     | 備份容易，可單表操作              |

---

###### ✅ 四、怎麼切換儲存方式？
> 我们可以自己指定使用系统表空间还是独立表空间来存储数据，这个功能取決於參數 **`innodb_file_per_table`**

####### 📌 設定方法

######## 🔹 MySQL 配置（永久）：
```ini
# my.cnf or my.ini 中加入：
[mysqld]
innodb_file_per_table=1  # 開啟獨立表空間（預設為 1，可不寫）
```

######## 🔹 即時查詢（目前值）：
```sql
SHOW VARIABLES LIKE 'innodb_file_per_table';
```

######## 🔹 動態修改（暫時性）：
```sql
SET GLOBAL innodb_file_per_table = 0; -- 關閉，使用系統表空間
```

> ⚠️ 注意：修改後只會影響**之後建立的表格**，已存在的表不會受影響。

---

###### ✅ 五、簡單範例：實驗兩種表空間效果

####### 📋 步驟 1：查詢當前表空間設定

```sql
SHOW VARIABLES LIKE 'innodb_file_per_table';
```

若為 `ON`，表示新建的表將使用「獨立表空間（.ibd 檔案）」。

---

####### 📋 步驟 2：建立表格

```sql
CREATE DATABASE atguigu;
USE atguigu;

CREATE TABLE demo (
  id INT PRIMARY KEY,
  name VARCHAR(50)
);
```

---

####### 📋 步驟 3：檢查磁碟上的檔案（假設 MySQL 資料目錄為 `/var/lib/mysql/`）

```bash
cd /var/lib/mysql/atguigu
ls -lh demo.*
```

######## 🔸 如果 `innodb_file_per_table = 1`（預設）：
```
demo.ibd      ← 儲存資料與索引
demo.frm      ← MySQL 5.x 才會有（MySQL 8.0 整合進 .ibd）
```

######## 🔸 如果 `innodb_file_per_table = 0`：
```
只會有 demo.frm，資料寫入 ibdata1，不會有 .ibd
```

---

###### ✅ 六、切換實例（完整流程）

####### 假設你想讓新建表用「系統表空間」，流程如下：

######## 📝 修改參數（方式一：動態設定）
```sql
SET GLOBAL innodb_file_per_table = 0;
```

######## 🚀 建立新表
```sql
CREATE TABLE sys_demo (
  id INT PRIMARY KEY,
  content TEXT
);
```

######## 🔍 驗證結果
```bash
ls /var/lib/mysql/atguigu/
```

你會發現沒有 `sys_demo.ibd`，表資料寫到 `ibdata1` 中。

---

###### 🧠 總結整理

| 操作                          | 結果                              |
|-------------------------------|-----------------------------------|
| `innodb_file_per_table=0`     | 表資料進入共享 ibdata1 表空間    |
| `innodb_file_per_table=1`     | 表資料存在自己對應的 `.ibd` 檔案 |
| 查詢當前狀態                  | `SHOW VARIABLES LIKE 'innodb_file_per_table'` |
| 修改當前狀態（僅影響新表）    | `SET GLOBAL innodb_file_per_table = 1` or `0` |

---

### 2.4.2 MyISAM存储引擎模式
#### 2.4.2.1 表結構
> 在存储表结构方面， MyISAM 和 InnoDB 一样，也是在数据目录下对应的数据库子目录下创建了一个专门用于描述表结构的文件

```shell
表名.frm
```

##### ✅ 一、MyISAM 的表結構存放方式

MyISAM 引擎是 MySQL 早期的預設儲存引擎，它不支援事務與外鍵，但擁有較快的查詢速度，特別適合讀取密集的場景。

當你建立一張表並使用 MyISAM 引擎時，MySQL 會為該表在資料庫目錄中建立三個檔案：

| 檔案類型 | 副檔名 | 功能說明 |
|----------|--------|-----------|
| 表結構檔案 | `.frm` | 儲存表的欄位定義、索引資訊等結構資料（**InnoDB 也用此檔案儲存表結構**） |
| 資料檔案 | `.MYD`（MyData） | 儲存表中的實際資料（Data） |
| 索引檔案 | `.MYI`（MyIndex） | 儲存索引資料，例如 PRIMARY KEY、UNIQUE KEY 等 |

---

##### ✅ 二、簡單範例

```sql
CREATE TABLE students (
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    age INT UNSIGNED,
    PRIMARY KEY (id)
) ENGINE=MyISAM;
```

假設這張 `students` 表建立在資料庫 `school` 裡，則在 MySQL 的資料目錄中會看到：

```
/var/lib/mysql/school/
├── students.frm    <-- 儲存表的欄位、主鍵等結構
├── students.MYD    <-- 儲存實際的學生資料
└── students.MYI    <-- 儲存主鍵、索引等資訊
```

---

##### ✅ 三、補充說明

1. **不支援事務與外鍵**  
   MyISAM 不支援 `BEGIN`, `COMMIT`, `ROLLBACK`，也不支援 `FOREIGN KEY` 約束。

2. **表鎖 (table-level lock)**  
   MyISAM 使用的是**表級鎖**，並非行鎖 (如 InnoDB)，因此在寫入時整張表會被鎖住，這會影響並發性。

3. **壓縮表**  
   可以使用 `myisampack` 工具來壓縮 MyISAM 表，節省空間。

---

##### ✅ 四、何時使用 MyISAM？

- **大量查詢、少量寫入** 的場景（例如報表查詢）
- 對事務、資料一致性要求不高的場景
- 對磁碟使用率要求嚴格（可搭配壓縮功能）

---

#### 2.4.2.2 表中數據和索引
##### ✅ 一、MyISAM 的資料與索引儲存特性

MyISAM 的設計非常直觀：**資料與索引是分開儲存的**，這一點跟 InnoDB 的聚簇索引（資料與主鍵索引存一起）不同。

| 類型 | 副檔名 | 說明 |
|------|--------|------|
| 表結構 | `.frm`（MySQL 5.x 及以下）或 `.sdi`（MySQL 8.0） | 儲存表的欄位名稱、型別、主鍵資訊等結構定義 |
| 資料檔案 | `.MYD`（MYData） | 儲存實際的資料列內容（如 name, age） |
| 索引檔案 | `.MYI`（MYIndex） | 儲存所有索引，包括主鍵、普通索引，皆為**二級索引** |

> MyISAM **所有索引都是二級索引（Secondary Index）**，包含主鍵。這代表：
> 索引儲存的是「鍵值 + 指向資料位置的指標（row pointer）」，資料和索引是分離的。

---

##### ✅ 二、簡單範例說明

```sql
CREATE TABLE `student_myisam` (
  `id` BIGINT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(64) DEFAULT NULL,
  `age` INT DEFAULT NULL,
  `sex` VARCHAR(2) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MYISAM AUTO_INCREMENT=0 DEFAULT CHARSET=utf8mb3;
```

---

##### ✅ 三、資料存放方式（檔案示意）

假設你是在 `test` 資料庫下建立這張表，MySQL 的資料目錄下會出現以下檔案：

```
/var/lib/mysql/test/
├── student_myisam.frm   <-- 儲存表的結構（MySQL 5.x）
├── student_myisam.MYD   <-- 儲存實際資料內容（Data）
└── student_myisam.MYI   <-- 儲存索引資訊（Index）
```

---

##### ✅ 四、資料與索引的存取流程示意

###### 🔹 插入資料
```sql
INSERT INTO student_myisam (name, age, sex) VALUES ('Alice', 22, 'F');
```

1. 這筆資料會寫入 `.MYD` 檔案。
2. 主鍵 `id` 被自動設為 1，並建立對應的索引項目，寫入 `.MYI` 檔案：
   - 索引鍵：`1`
   - 指標：指向 `.MYD` 裡這筆資料的位移位置

---

###### 🔹 查詢資料
```sql
SELECT * FROM student_myisam WHERE id = 1;
```

1. MySQL 使用 `.MYI` 檔案查找索引 `id = 1`，取得對應的 `.MYD` 資料位置。
2. 根據該位置，從 `.MYD` 中抓出整筆資料回傳。

> 所有索引查詢都走二級索引 → 再根據指標去 `.MYD` 找出資料 → 多一個跳轉步驟

---

##### ✅ 五、MyISAM 索引特性與限制

| 特性 | 說明 |
|------|------|
| 📄 所有索引皆為二級索引 | 包含主鍵也是，需多跳一次找資料位置 |
| 📌 多欄索引與全文索引支援好 | MyISAM 支援 FULLTEXT 全文索引 |
| 🔒 鎖機制為**表級鎖** | 寫入時會鎖整張表，並發效能差於 InnoDB |
| ❌ 不支援外鍵與事務 | 若需要資料一致性、回滾處理，建議用 InnoDB |

---

##### ✅ 六、補充：MySQL 8.0 的 .sdi

在 MySQL 8.0 中，`.frm` 檔案已不再使用。無論是 InnoDB 還是 MyISAM，引擎的表結構資訊統一放入 `.sdi`（Serialized Dictionary Information） 檔案中。

範例：

```
/var/lib/mysql/test/
├── student_myisam.sdi   <-- MySQL 8.0 結構資訊
├── student_myisam.MYD
└── student_myisam.MYI
```

---

##### ✅ 小結

| 項目 | MyISAM 行為 |
|------|-------------|
| 資料與索引 | 分開存放 (`.MYD`, `.MYI`) |
| 主鍵索引類型 | 二級索引 |
| 結構檔案 | `.frm`（舊）或 `.sdi`（新） |
| 適用情境 | 查詢密集、大量讀取、簡易應用 |
| 不支援 | 外鍵、事務、行級鎖 |

---

## 2.5 小結
> 举例：数据库a ， 表b 。

### 如果表b采用 InnoDB： `data\a` 中会产生1个或者2个文件：
- b.frm ：描述表结构文件，字段长度等
- 如果采用 系统表空间 模式的，数据信息和索引信息都存储在 ibdata1 中
- 如果采用 独立表空间 存储模式，data\a中还会产生 b.ibd 文件（存储数据信息和索引信息）

- 此外：
    - ① MySQL5.7 中会在 `data/a` 的目录下生成 `db.opt` 文件用于保存数据库的相关配置。比如：字符集、比较规则。而 `MySQL8.0` 不再提供 `db.opt` 文件。
    - ② MySQL8.0 中不再单独提供 `b.frm`，而是合并在 `b.ibd` 文件中。

### 如果表b采用 MyISAM：`data\a` 中会产生 3 个文件：
- MySQL5.7 中： b.frm ：描述表结构文件，字段长度等。
- MySQL8.0 中  b.xxx.sdi ：描述表结构文件，字段长度等
- b.MYD (MYData)：数据信息文件，存储数据信息(如果采用独立表存储模式)
- b.MYI (MYIndex) ：存放索引信息文件

## 2.6 视图在文件系统中的表示
> 我们知道 MySQL 中的视图其实是虚拟的表，也就是某个查询语句的一个别名而已，所以在存储视图的时候是不需要存储真实的数据的，只需要把它的结构存储起来就行了。和表一样，描述视图结构的文件也会被存储到所属数据库对应的子目录下边，只会存储一个视图名.frm的文件。

---

### 一、視圖是什麼？

**視圖（View）** 本質上是：
- 一個「虛擬表格」
- 它不保存實際數據
- 它保存的是查詢語句（SQL）作為定義
- 每次查詢視圖時，其實是執行那段 SQL 查詢

---

### 二、視圖在檔案系統的表示方式（MySQL 5.x 為例）

當你建立一個視圖時，MySQL 會在資料庫對應的目錄中存下一些與視圖定義有關的文件（這些視圖文件會根據你使用的儲存引擎與 MySQL 版本而不同）。

一般來說會包含以下檔案：

| 檔名                  | 說明 |
|-----------------------|------|
| `view_name.frm`       | 定義視圖的欄位結構（字段的資料型別等）|
| `view_name.trg`       | （可選）與該視圖有關的觸發器（Trigger）定義檔案 |
| `view_name.ibd`       | 視圖不會產生這個檔案，因為不儲存資料 |
| `db.opt`              | 資料庫的設定檔（字符集等）|

> **補充**：從 MySQL 8.0 開始，已不再使用 `.frm` 文件，而是使用 [data dictionary] 儲存資料庫結構資訊，但這對理解歷史系統仍重要。

---

### 三、舉例說明

假設你建立以下資料庫和視圖：

```sql
CREATE DATABASE company;

USE company;

CREATE TABLE employees (
    emp_id INT,
    name VARCHAR(50),
    salary DECIMAL(10,2)
);

-- 建立一個簡單的視圖
CREATE VIEW high_salary_emps AS
SELECT emp_id, name
FROM employees
WHERE salary > 50000;
```

#### 在文件系統中（假設資料目錄為 `/var/lib/mysql/`）：

```bash
/var/lib/mysql/company/
├── employees.frm       # 表的欄位結構定義
├── high_salary_emps.frm # 視圖的結構定義
├── db.opt              # 資料庫設定檔
```

#### `high_salary_emps.frm` 檔案的作用：

這個 `.frm` 檔案中不包含資料，而是：
- 定義視圖的欄位結構（例如：emp_id 是 INT、name 是 VARCHAR(50)）
- 包含定義這個視圖的 SQL 查詢語句（即 `SELECT emp_id, name FROM ...`）

---

## 2.7 其他的文件
> 除了我们上边说的这些用户自己存储的数据以外，数据目录下还包括为了更好运行程序的一些额外文件，主要包括这几种类型的文件:

### 2.7.1 一、伺服器進程文件（Server PID File）

#### 📌 說明：
- 當你啟動 MySQL 伺服器時，會有一個對應的「**進程ID（PID）**」。
- MySQL 會把這個 PID 寫進一個 `.pid` 的檔案，通常命名為 `{hostname}.pid`。
- 這對於系統管理員關閉伺服器或監控伺服器狀態非常重要。

#### 🧩 範例：
如果你的主機名為 `localhost`，那麼 PID 檔可能是：
```
/usr/local/mysql/data/localhost.pid
```

---

### 2.7.2 二、伺服器日誌文件（Server Log Files）

這類檔案用於記錄 MySQL 的運行狀況和各種操作紀錄。主要有以下幾種：

#### 1. 錯誤日誌（Error Log）
- 記錄啟動/關閉資訊、錯誤訊息、崩潰資訊等。
- 檔名類似 `hostname.err`。
- 例如：
  ```
  /usr/local/mysql/data/localhost.err
  ```

#### 2. 查詢日誌（General Query Log）📝
- 記錄所有 SQL 語句（可選擇性啟用）。
- 用於開發除錯或審計用途。

#### 3. 慢查詢日誌（Slow Query Log）🐢
- 記錄執行時間超過指定秒數的 SQL 語句。
- 用來發現效率不佳的查詢。

#### 4. 二進位日誌（Binary Log）🧱
- 記錄所有資料修改操作（如 `INSERT`, `UPDATE`）。
- 用於主從複製、資料還原。
- 檔案範例：
  ```
  mysql-bin.000001
  mysql-bin.000002
  ```

#### 5. 重做日誌（Redo Log）
- 用於 InnoDB 引擎的崩潰恢復機制。
- 檔案像：
  ```
  ib_logfile0
  ib_logfile1
  ```

---

### 2.7.3 三、SSL 與 RSA 憑證與金鑰檔案

#### 📌 說明：
- 從 MySQL 5.7 起，安裝時會自動生成 SSL/TLS 憑證與金鑰。
- 用於保護伺服器與用戶端之間的通訊安全。
- 包括 `.pem` 檔案，例如：

#### 🧩 範例：
```
ca-key.pem         # CA 的私鑰
ca.pem             # CA 憑證
server-key.pem     # 伺服器私鑰
server-cert.pem    # 伺服器憑證
client-key.pem     # 客戶端私鑰
client-cert.pem    # 客戶端憑證
```

這些檔案會放在：
```
/usr/local/mysql/data/
```
或者某個指定的安全目錄中。

---

### 2.7.4 四、其他補充類型的檔案

#### 1. 套件資訊（如：`auto.cnf`）
- 初始化資料庫時產生，記錄 server UUID。

#### 2. InnoDB 系統表空間（預設模式）
- 像是：
  ```
  ibdata1   # 存放 InnoDB 表結構與資料的共享空間（除非用獨立表空間）
  ```

---

### 2.7.5 🔍 小總結

| 類型       | 代表檔案/範例 | 功能描述 |
|------------|----------------|----------|
| 進程檔案    | `localhost.pid`     | 儲存 MySQL 伺服器 PID |
| 錯誤日誌    | `localhost.err`     | 記錄錯誤、啟動、崩潰資訊 |
| 二進位日誌  | `mysql-bin.000001`  | 資料操作紀錄，支援複製與還原 |
| redo log    | `ib_logfile0`       | InnoDB 引擎寫入紀錄、用於恢復 |
| 通用查詢日誌| `hostname.log`      | 所有 SQL 操作紀錄（可選） |
| 慢查詢日誌  | `hostname-slow.log` | 執行效率差的查詢紀錄 |
| SSL 憑證    | `*.pem` 檔案        | 加密連線所需之金鑰與憑證 |

---

## 補充內容
### 資料字典 Data Dictionary

> 你提到的「InnoDB 的內建資料字典（Data Dictionary）」，是在 MySQL 8.0 起出現的重要架構變革，它用來取代過去 `.frm`、`.trn`、`.par` 等外部檔案儲存方式，以下我會清楚說明什麼是 **資料字典（Data Dictionary）**，並舉例幫助你理解。

---

#### 📚 一、什麼是資料字典（Data Dictionary）？

**資料字典（Data Dictionary）** 是 **InnoDB 儲存在內部系統表中的元資料庫（Metadata Database）**，記錄所有資料庫中：

| 類別 | 說明 |
|------|------|
| 表格 | 表名、欄位、資料型別、主鍵、索引等 |
| 索引 | 主索引、次索引的定義與屬性 |
| 結構關聯 | 外鍵、約束條件等 |
| 統計資訊 | 表大小、行數估算、索引選擇性等 |
| 表空間資訊 | `.ibd` 檔案資訊、檔案大小與位置 |

👉 這些資訊**不再存於檔案（.frm）**，而是儲存在 InnoDB 系統表中。

---

#### 🏛️ 二、資料字典儲存在哪裡？

在 MySQL 8.0 中，這些資訊會儲存在內部的 InnoDB 系統表（**資料字典表**）中，例如：

| 系統表名 | 內容說明 |
|----------|-----------|
| `mysql.innodb_table_stats` | 各表的統計資訊（例如行數） |
| `mysql.innodb_index_stats` | 索引統計資料 |
| `mysql.tables` | 所有表格的定義 |
| `mysql.columns` | 表格欄位的詳細資訊 |
| `mysql.indexes` | 所有索引的結構 |
| `mysql.foreign_keys` | 外鍵定義 |

這些表**使用 InnoDB 儲存引擎自己管理，使用者無法直接更新**，但你可以查詢它們以了解系統內部運作。

---

#### 🧪 三、查詢資料字典範例

假設你建立了一個表 `employees`，你可以查詢 `information_schema` 或 `mysql` 資料庫來看到其元資料：

```sql
-- 查詢表結構資訊
SELECT * FROM information_schema.tables
WHERE table_schema = 'your_database' AND table_name = 'employees';

-- 查詢欄位定義
SELECT * FROM information_schema.columns
WHERE table_schema = 'your_database' AND table_name = 'employees';
```

---

#### 🧠 四、為什麼 InnoDB 改用資料字典？

MySQL 8.0 將元資料統一存入 InnoDB 資料字典，有以下幾個重要優點：

| 優點 | 說明 |
|------|------|
| ✅ 一致性提升 | 不再依賴外部 `.frm`、`.ibd` 配對一致性 |
| ✅ 元資料 ACID 化 | 可使用交易保護表結構變更（CREATE/ALTER/DROP） |
| ✅ 效能更穩定 | 表格結構和資料同樣存在 InnoDB，操作更快 |
| ✅ 減少檔案碎片問題 | 避免多個檔案在檔案系統中造成混亂 |

---

#### ✅ 小結

| 問題 | 回答 |
|------|------|
| 資料字典是什麼？ | 儲存所有表格與索引的結構資訊的系統表 |
| 哪個版本開始使用？ | MySQL 8.0 |
| 儲存在什麼地方？ | InnoDB 系統表空間的系統表中 |
| 可以查詢嗎？ | 可透過 `information_schema` 或 `mysql` 資料庫查詢 |

---

###