---
title: 資料科學虛擬機器的資料平台 - Azure | Microsoft Docs
description: 資料科學虛擬機器的資料平台。
keywords: 資料科學工具、資料科學虛擬機器、資料科學工具、linux 資料科學
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/16/2018
ms.author: gokuma
ms.openlocfilehash: b3f340006801287383c2afb2924706affbd77a51
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31411024"
---
# <a name="data-platforms"></a>資料平台

資料科學虛擬機器 (DSVM) 可讓您遵循各種不同的資料平台建立您的分析。 除了遠端資料平台的介面之外，DSVM 也提供快速開發和原型設計的本機執行個體。 

以下是 DSVM 上支援的資料平台工具。 

## <a name="sql-server-2016-developer-edition"></a>SQL Server 2016 Developer Edition

| | |
| ------------- | ------------- |
| 這是什麼？   | 本機關聯式資料庫執行個體      |
| 支援的 DSVM 版本      | Windows      |
| 典型的使用案例      | 在本機使用較小的資料集進行快速的開發 <br/> 執行資料庫內 R   |
| 範例的連結      |    New York City 資料集的一個小型範例載入至 SQL 資料庫 `nyctaxi`。 <br/> 顯示 Microsoft R 和資料庫中內分析的 Jupyter 範例位於：<br/> `~notebooks/SQL_R_Services_End_to_End_Tutorial.ipynb`  |
| DSVM 上的相關工具       | SQL Server Management Studio <br/> ODBC/JDBC 驅動程式<br/> pyodbc、RODBC<br />Apache 深入探詢      |

> [!NOTE]
> SQL Server 2016 Developer Edition 只能用於進行開發和測試。 您需要授權或其中一個 SQL Server VM，才能在生產環境中執行。 


### <a name="setup"></a>設定

資料庫伺服器經過預先設定，而且與 SQL Server 相關的 Windows 服務 (例如 `SQL Server (MSSQLSERVER)`) 是設定為自動執行。 唯一將執行的手動步驟是使用 Microsoft R 啟用資料庫內分析。您以機器系統管理員身分登入後，可以在 SQL Server Management Studio (SSMS) 中執行屬於一次性動作的下列命令，在 SSMS 中開啟 [新增查詢]，確定選取的資料庫是 `master`，然後執行： 

        CREATE LOGIN [%COMPUTERNAME%\SQLRUserGroup] FROM WINDOWS 

        (Please replace the %COMPUTERNAME% with your VM name)
       
若要執行 SQL Server Management Studio，您可以在程式清單上搜尋「SQL Server Management Studio」，也可以使用 Windows 搜尋來尋找並執行它。 提示輸入認證時，請選擇「Windows 驗證」，並在 SQL Server 名稱中使用機器名稱或 ```localhost```。 

### <a name="how-to-use--run-it"></a>如何使用/執行它？  

有預設資料庫執行個體的資料庫伺服器預設為自動執行。 您可以在 VM 上使用 SQL Server Management Studio 之類的工具在本機存取 SQL Server 資料庫。 本機系統管理員帳戶具有資料庫的系統管理員存取權。 

DSVM 也提供 ODBC 驅動程式和 JDBC 驅動程式，能夠從以包含 Python、R 在內的多種語言撰寫的應用程式中與 SQL Server，Azure SQL 資料庫和 Azure SQL 資料倉儲進行通訊。 

### <a name="how-is-it-configured--installed-on-the-dsvm"></a>它是如何在 DSVM 上設定/安裝的？ 

SQL Server 是以標準方式安裝的。 它位於 `C:\Program Files\Microsoft SQL Server`。 在 `C:\Program Files\Microsoft SQL Server\MSSQL13.MSSQLSERVER\R_SERVICES` 可以找到資料庫內 R 執行個體。 DSVM 也有安裝在 `C:\Program Files\Microsoft\R Server\R_SERVER` 的個別獨立 R 伺服器執行個體。 這兩個 R 執行個體不會共用程式庫。


## <a name="apache-spark-2x-standalone"></a>Apache Spark 2.x (獨立)

| | |
| ------------- | ------------- |
| 這是什麼？   | 常用 Apache Spark 平台的獨立 (單一節點內含式) 執行個體，這是快速大規模資料處理和機器學習的系統     |
| 支援的 DSVM 版本      | Linux <br /> Windows (實驗性)      |
| 典型的使用案例      | * 使用較小的資料集在本機快速開發 Spark/PySpark 應用程式，然後將它部署在 Azure HDInsight 之類的大型 Spark 叢集上<br/> * 測試 Microsoft R Server Spark 內容 <br />* 使用 SparkML 或 Microsoft 的開放原始碼 [MMLSpark](https://github.com/Azure/mmlspark) 程式庫建置 ML 應用程式  |
| 範例的連結      |    Jupyter 範例： <br />&nbsp;&nbsp;* ~/notebooks/SparkML/pySpark <br /> &nbsp;&nbsp;* ~/notebooks/MMLSpark <br /> Microsoft R Server (Spark 內容)：/dsvm/samples/MRS/MRSSparkContextSample.R |
| DSVM 上的相關工具       | PySpark、Scala<br/>Jupyter (Spark/PySpark 核心)<br/>Microsoft R Server、SparkR、Sparklyr <br />Apache 深入探詢      |

### <a name="how-to-use-it"></a>如何使用它
您可以使用 `spark-submit` 或 `pyspark` 命令在命令列上提交 Spark 工作，以便執行 Spark。 您也可以使用 Spark 核心建立新的 Notebook，以便建立 Jupyter Notebook。 

您可以使用 DSVM 上提供的 SparkR、Sparklyr 或 Microsoft R Server 等程式庫從 R 使用 Spark。 請參閱上表中範例的指標。 

> [!NOTE]
> 僅 Ubuntu Linux DSVM 版本支援在 DSVM 的 Spark 內容中執行 Microsoft R Server。 



### <a name="setup"></a>設定
在 Ubuntu Linux DSVM 版本上，於 Microsoft R 伺服器的 Spark 內容中執行之前，您必須執行一次性設定步驟，以啟用本機單一節點 Hadoop HDFS 和 Yarn 執行個體。 根據預設，Hadoop 服務已安裝但是在 DSVM 上已停用。 若要啟用它，您需要在第一次以 root 身分執行下列命令︰

    echo -e 'y\n' | ssh-keygen -t rsa -P '' -f ~hadoop/.ssh/id_rsa
    cat ~hadoop/.ssh/id_rsa.pub >> ~hadoop/.ssh/authorized_keys
    chmod 0600 ~hadoop/.ssh/authorized_keys
    chown hadoop:hadoop ~hadoop/.ssh/id_rsa
    chown hadoop:hadoop ~hadoop/.ssh/id_rsa.pub
    chown hadoop:hadoop ~hadoop/.ssh/authorized_keys
    systemctl start hadoop-namenode hadoop-datanode hadoop-yarn

您可以在不需要它們時停止 Hadoop 相關服務，方法是執行 ````systemctl stop hadoop-namenode hadoop-datanode hadoop-yarn```` 示範如何在遠端 Spark 內容中開發和測試 MRS (這是 DSVM 上的獨立 Spark 執行個體) 的範例，以及 `/dsvm/samples/MRS` 目錄中的可用項目。 


### <a name="how-is-it-configured--installed-on-the-dsvm"></a>它是如何在 DSVM 上設定/安裝的？ 
|平台|安裝位置 ($SPARK_HOME)|
|:--------|:--------|
|Windows | c:\dsvm\tools\spark-X.X.X-bin-hadoopX.X|
|Linux   | /dsvm/tools/spark-X.X.X-bin-hadoopX.X|


從 Azure Blob 或 Azure Data Lake 儲存體 (ADLS) 存取資料的程式庫和 Microsoft 的 MMLSpark 機器學習程式庫預先安裝於 $SPARK_HOME/jars。 Spark 啟動時，這些 JAR 會自動載入。 Spark 預設使用本機磁碟上的資料。 若要讓 DSVM 上的 Spark  執行個體存取 Azure Blob 或 ADLS 上儲存的資料，您需要遵循在 $SPARK_HOME/conf/core-site.xml.template (其中包含 Blob 和 ADLS 組態的預留位置) 中找到的範本，使用 Azure Blob 和 Azure Data Lake 儲存體的適當認證建立/設定 `core-site.xml` 檔案。 您可以在[這裡](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-authenticate-using-active-directory#create-an-active-directory-application)找到建立 ADLS 服務認證的詳細步驟。 在 core-site.xml 檔案中輸入 Azure Blob 或 ADLS 的認證時，您可以使用 wasb:// 或 adl:// 的 URI 前置詞參照這些來源中儲存的資料。 

