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