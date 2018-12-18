---
title: 在 OMS Log Analytics 中收集 Linux 應用程式效能 | Microsoft Docs
description: 本文詳細說明如何設定 OMS Agent for Linux，以收集 MySQL 和 Apache HTTP Server 的效能計數器。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: f1d5bde4-6b86-4b8e-b5c1-3ecbaba76198
ms.service: log-analytics
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/04/2017
ms.author: magoedte
ms.component: ''
ms.openlocfilehash: 5120fa869d9c3fe28630b189b84b9c3e3f5577e2
ms.sourcegitcommit: 3856c66eb17ef96dcf00880c746143213be3806a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48044564"
---
# <a name="collect-performance-counters-for-linux-applications-in-log-analytics"></a>在 Log Analytics 中收集 Linux 應用程式的效能計數器 
本文詳細說明如何設定 [OMS Agent for Linux](https://github.com/Microsoft/OMS-Agent-for-Linux)，以收集特定應用程式的效能計數器。  本文包含的應用程式如下︰  

- [MySQL](#MySQL)
- [Apache HTTP Server](#apache-http-server)

## <a name="mysql"></a>MySQL
如果在安裝 OMS 代理程式時於電腦上偵測到 MySQL 伺服器或 MariaDB 伺服器，則會自動安裝 MySQL 伺服器的效能監視提供者。 此提供者會連接到本機 MySQL/MariaDB 伺服器，以公開效能統計資料。 必須設定 MySQL 使用者認證，提供者才能存取 MySQL 伺服器。

### <a name="configure-mysql-credentials"></a>設定 MySQL 認證
MySQL OMI 提供者需要預先設定的 MySQL 使用者，並已安裝 MySQL 用戶端程式庫，才能從 MySQL 執行個體查詢效能和健康情況資訊。  這些認證儲存在 Linux 代理程式上的驗證檔案。  驗證檔案指定 MySQL 執行個體所接聽的繫結位址和連接埠，以及要用來收集計量的認證。  

在 OMS Agent for Linux 安裝期間，MySQL OMI 提供者將掃描繫結位址和連接埠的 MySQL my.cnf 組態檔 (預設位置)，並局部設定 MySQL OMI 驗證檔案。

MySQL 驗證檔案儲存在 `/var/opt/microsoft/mysql-cimprov/auth/omsagent/mysql-auth`。


### <a name="authentication-file-format"></a>驗證檔案格式
以下是 MySQL OMI 驗證檔案的格式

    [Port]=[Bind-Address], [username], [Base64 encoded Password]
    (Port)=(Bind-Address), (username), (Base64 encoded Password)
    (Port)=(Bind-Address), (username), (Base64 encoded Password)
    AutoUpdate=[true|false]

下表說明驗證檔案中的項目。

| 屬性 | 說明 |
|:--|:--|
| Port | 代表 MySQL 執行個體目前正在接聽的連接埠。 連接埠 0 指定後面的屬性用於預設執行個體。 |
| 繫結位址| 目前的 MySQL 繫結位址。 |
| username| 用來監視 MySQL 伺服器執行個體的 MySQL 使用者。 |
| Base64 編碼的密碼| MySQL 監視使用者的密碼，以 Base64 編碼。 |
| AutoUpdate| 指定在升級 MySQL OMI 提供者時，是否重新掃描 my.cnf 檔案中的變更，並覆寫 MySQL OMI 驗證檔案。 |

### <a name="default-instance"></a>預設執行個體
MySQL OMI 驗證檔案可以定義預設執行個體和連接埠號碼，讓您更輕鬆地管理一台 Linux 主機上的多個 MySQL 執行個體。  連接埠為 0 的執行個體代表預設執行個體。 所有其他執行個體除非指定不同的值，否則會繼承預設執行個體的屬性集。 例如，如果加入接聽通訊埠 '3308' 的 MySQL 執行個體，預設執行個體的繫結位址、使用者名稱和 Base64 編碼的密碼將會用來嘗試和監視接聽 3308 的執行個體。 如果 3308 上的執行個體繫結至另一個位址，並使用相同的 MySQL 使用者名稱和密碼配對，則只需要繫結位址，其他屬性會繼承而來。

下表提供執行個體設定範例 

| 說明 | 檔案 |
|:--|:--|
| 預設執行個體和連接埠為 3308 的執行個體。 | `0=127.0.0.1, myuser, cnBwdA==`<br>`3308=, ,`<br>`AutoUpdate=true` |
| 預設執行個體，以及連接埠為 3308 且具有不同使用者名稱密碼的執行個體。 | `0=127.0.0.1, myuser, cnBwdA==`<br>`3308=127.0.1.1, myuser2,cGluaGVhZA==`<br>`AutoUpdate=true` |


### <a name="mysql-omi-authentication-file-program"></a>MySQL OMI 驗證檔案程式
安裝 MySQL OMI 提供者時會隨附 MySQL OMI 驗證檔案程式，可用來編輯 MySQL OMI 驗證檔案。 驗證檔案程式位於下列位置。

    /opt/microsoft/mysql-cimprov/bin/mycimprovauth

> [!NOTE]
> omsagent 帳戶必須可讀取認證檔案。 建議以 omsgent 身分執行 mycimprovauth 命令。

下表提供使用 mycimprovauth 語法的詳細資料。

| 作業 | 範例 | 說明
|:--|:--|:--|
| autoupdate *false or true* | mycimprovauth autoupdate false | 設定是否在重新啟動或更新時自動更新驗證檔案。 |
| default *bind-address username password* | mycimprovauth default 127.0.0.1 root pwd | 在 MySQL OMI 驗證檔案中設定預設執行個體。<br>應以純文字輸入 password 欄位 - MySQL OMI 驗證檔案中的密碼將會以 Base 64 編碼。 |
| delete *default or port_num* | mycimprovauth 3308 | 依預設值或依連接埠號碼刪除指定的執行個體。 |
| help | mycimprov help | 印出可使用的命令清單。 |
| print | mycimprov print | 印出方便閱讀的 MySQL OMI 驗證檔案。 |
| update port_num *bind-address username password* | mycimprov update 3307 127.0.0.1 root pwd | 更新指定的執行個體，而如果不存在，則新增執行個體。 |

下列命令範例會為 localhost 上的 MySQL 伺服器定義預設使用者帳戶。  應以純文字輸入 password 欄位 - MySQL OMI 驗證檔案中的密碼將會以 Base 64 編碼

    sudo su omsagent -c '/opt/microsoft/mysql-cimprov/bin/mycimprovauth default 127.0.0.1 <username> <password>'
    sudo /opt/omi/bin/service_control restart

### <a name="database-permissions-required-for-mysql-performance-counters"></a>MySQL 效能計數器所需的資料庫權限
MySQL 使用者需要存取下列查詢，才能收集 MySQL 伺服器的效能資料。 

    SHOW GLOBAL STATUS;
    SHOW GLOBAL VARIABLES:


MySQL 使用者需要下列預設資料表的 SELECT 存取權。

- information_schema
- mysql。 

執行下列 grant 命令，可以授與這些權限。

    GRANT SELECT ON information_schema.* TO ‘monuser’@’localhost’;
    GRANT SELECT ON mysql.* TO ‘monuser’@’localhost’;


> [!NOTE]
> 若要將權限授與 MySQL 監視使用者，則授與使用者必須具有 'GRANT option' 權限以及正在授與的權限。

### <a name="define-performance-counters"></a>定義效能計數器

設定 OMS Agent for Linux 將資料傳送至 Log Analytics 之後，您必須設定要收集的效能計數器。  使用 [Log Analytics 中的 Windows 和 Linux 效能資料來源](log-analytics-data-sources-windows-events.md)中的程序與下表中的計數器。

| 物件名稱 | 計數器名稱 |
|:--|:--|
| MySQL Database | Disk Space in Bytes |
| MySQL Database | 資料表 |
| MySQL Server | Aborted Connection Pct |
| MySQL Server | Connection Use Pct |
| MySQL Server | Disk Space in Bytes |
| MySQL Server | Full Table Scan Pct |
| MySQL Server | InnoDB Buffer Pool Hit Pct |
| MySQL Server | InnoDB Buffer Pool Use Pct |
| MySQL Server | InnoDB Buffer Pool Use Pct |
| MySQL Server | Key Cache Hit Pct |
| MySQL Server | Key Cache Use Pct |
| MySQL Server | Key Cache Write Pct |
| MySQL Server | Query Cache Hit Pct |
| MySQL Server | Query Cache Prunes Pct |
| MySQL Server | Query Cache Use Pct |
| MySQL Server | Table Cache Hit Pct |
| MySQL Server | Table Cache Use Pct |
| MySQL Server | Table Lock Contention Pct |

## <a name="apache-http-server"></a>Apache HTTP Server 
如果在安裝 omsagent 組合時於電腦上偵測到 Apache HTTP Server，則會自動安裝 Apache HTTP Server 的效能監視提供者。 此提供者所依賴的 Apache 模組必須載入至 Apache HTTP Server，才能存取效能資料。 您可以使用下列命令載入此模組：
```
sudo /opt/microsoft/apache-cimprov/bin/apache_config.sh -c
```

若要卸載 Apache 監視模組，請執行下列命令︰
```
sudo /opt/microsoft/apache-cimprov/bin/apache_config.sh -u
```

### <a name="define-performance-counters"></a>定義效能計數器

設定 OMS Agent for Linux 將資料傳送至 Log Analytics 之後，您必須設定要收集的效能計數器。  使用 [Log Analytics 中的 Windows 和 Linux 效能資料來源](log-analytics-data-sources-windows-events.md)中的程序與下表中的計數器。

| 物件名稱 | 計數器名稱 |
|:--|:--|
| Apache HTTP Server | Busy Workers |
| Apache HTTP Server | Idle Workers |
| Apache HTTP Server | Pct Busy Workers |
| Apache HTTP Server | Total Pct CPU |
| Apache Virtual Host | Errors per Minute - Client |
| Apache Virtual Host | Errors per Minute - Server |
| Apache Virtual Host | KB per Request |
| Apache Virtual Host | Requests KB per Second |
| Apache Virtual Host | Requests per Second |



## <a name="next-steps"></a>後續步驟
* 從 Linux 代理程式[收集效能計數器](log-analytics-data-sources-performance-counters.md)。
* 了解 [記錄搜尋](log-analytics-log-searches.md) ，其可分析從資料來源和方案所收集的資料。 
