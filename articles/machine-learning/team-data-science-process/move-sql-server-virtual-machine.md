---
title: 移動資料至 Azure 虛擬機器上的 SQL Server | Microsoft Docs
description: 從一般檔案或內部部署的 SQL Server 移動資料至 Azure VM 上的 SQL Server
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 2c9ef1d3-4f5c-4b1f-bf06-223646c8af06
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/04/2017
ms.author: deguhath
ms.openlocfilehash: a87d02732196af7471636346fa802f9a58869c95
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49393568"
---
# <a name="move-data-to-sql-server-on-an-azure-virtual-machine"></a>移動資料至 Azure 虛擬機器上的 SQL Server
本主題概述從一般檔案 (CSV 或 TSV 格式) 或是內部部署的 SQL Server，將資料移動至 Azure 虛擬機器上之 SQL Server 的選項。 將資料移到雲端的這些工作是 Team Data Science Process 的一部分。

如需概述移動資料至機器學習的 Azure SQL Database 之選項的主題，請參閱 [移動資料至 Azure Machine Learning 的 Azure SQL Database](move-sql-azure.md)。

以下 **功能表** 所連結的主題會說明如何將資料內嵌至其他目標環境，以在 Team Data Science Process (TDSP) 期間儲存和處理該資料。

[!INCLUDE [cap-ingest-data-selector](../../../includes/cap-ingest-data-selector.md)]

下表摘要說明移動資料至 Azure 虛擬機器上之 SQL Server 的選項。

| <b>來源</b> | <b>目的地：Azure VM 上的 SQL Server</b> |
| --- | --- |
| <b>一般檔案</b> |1.<a href="#insert-tables-bcp">命令列大量複製公用程式 (BCP)</a><br> 2.<a href="#insert-tables-bulkquery">大量插入 SQL 查詢</a><br> 3.<a href="#sql-builtin-utilities">SQL Server 中的圖形化內建公用程式</a> |
| <b>內部部署的 SQL Server</b> |1.<a href="#deploy-a-sql-server-database-to-a-microsoft-azure-vm-wizard">將 SQL Server Database 部署到 Microsoft Azure VM 精靈</a><br> 2.<a href="#export-flat-file">匯出到一般檔案</a><br> 3.<a href="#sql-migration">SQL Database 移轉精靈</a> <br> 4.<a href="#sql-backup">資料庫備份和還原</a><br> |

請注意，本文件假設 SQL 命令是從 SQL Server Management Studio 或 Visual Studio 資料庫總管中執行。

> [!TIP]
> 或者，您可以使用 [Azure Data Factory](https://azure.microsoft.com/services/data-factory/) 建立並排程管線，以將資料移至 Azure 上的 SQL Server VM。 如需詳細資訊，請參閱 [使用 Azure Data Factory 複製資料 (複製活動)](../../data-factory/copy-activity-overview.md)。
>
>

## <a name="prereqs"></a>必要條件
本教學課程假設您有：

* **Azure 訂用帳戶**。 如果您沒有訂用帳戶，可以註冊 [免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure 儲存體帳戶**。 在本教學課程中，您將使用 Azure 儲存體帳戶來儲存資料。 如果您沒有 Azure 儲存體帳戶，請參閱 [建立儲存體帳戶](../../storage/common/storage-quickstart-create-account.md) 一文。 建立儲存體帳戶之後，您必須取得用來存取儲存體的帳戶金鑰。 請參閱[管理儲存體存取金鑰](../../storage/common/storage-account-manage.md#access-keys)。
* 已佈建 **Azure VM 上的 SQL Server**。 如需指示，請參閱 [將 Azure SQL Server 虛擬機器設定為 IPython Notebook 伺服器供進階分析使用](../data-science-virtual-machine/setup-sql-server-virtual-machine.md)。
* 已在本機上安裝和設定 **Azure PowerShell** 。 如需指示，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

## <a name="filesource_to_sqlonazurevm"></a> 從一般檔案來源移動資料至 Azure VM 上的 SQL Server
如果您的資料位於一般檔案 (使用資料列或資料行格式排列) 中，可透過下列方法，將它移到 Azure 上的 SQL Server VM：

1. [命令列大量複製公用程式 (BCP)](#insert-tables-bcp)
2. [大量插入 SQL 查詢 ](#insert-tables-bulkquery)
3. [SQL Server 中的圖形化內建公用程式 (匯入/匯出，SSIS)](#sql-builtin-utilities)

### <a name="insert-tables-bcp"></a>命令列大量複製公用程式 (BCP)
BCP 是與 SQL Server 一起安裝的命令列公用程式，是最快速移動資料的其中一種方式。 它的運作方式可跨越這三個 SQL Server 版本 (內部部署的 SQL Server、SQL Azure，以及 Azure 上的 SQL Server VM)。

> [!NOTE]
> **我應該針對 BCP 將資料放置於何處？**  
> 雖然並非必要，但是，擁有包含與目標 SQL Server 位於相同電腦上之來源資料的檔案可讓傳輸速度 (網路速度與本機磁碟 IO 速度) 變得更快。 您可以使用像是 [AZCopy](../../storage/common/storage-use-azcopy.md)、[Azure 儲存體總管](http://storageexplorer.com/)或透過遠端桌面通訊協定 (RDP) 進行的 Windows 複製/貼上等各種檔案複製工具，將包含資料的一般檔案移到安裝 SQL Server 的電腦上。
>
>

1. 確定已在目標 SQL Server 資料庫上建立資料庫和資料表。 以下是如何使用 `Create Database` 和 `Create Table` 命令來執行此作業的範例：

        CREATE DATABASE <database_name>

        CREATE TABLE <tablename>
        (
            <columnname1> <datatype> <constraint>,
            <columnname2> <datatype> <constraint>,
            <columnname3> <datatype> <constraint>
        )
2. 從安裝 BCP 的電腦命令列中發出下列命令，以產生說明資料表結構描述的格式檔案。

    `bcp dbname..tablename format nul -c -x -f exportformatfilename.xml -S servername\sqlinstance -T -t \t -r \n`
3. 使用 BCP 命令來將資料插入資料庫，如下所示。 若假設 SQL Server 安裝於同一部電腦上，這應該就能從命令列順利運作：

    `bcp dbname..tablename in datafilename.tsv -f exportformatfilename.xml -S servername\sqlinstancename -U username -P password -b block_size_to_move_in_single_attemp -t \t -r \n`

> **最佳化 BCP 插入** ：請參閱 [最佳化大量匯入的指導方針](https://technet.microsoft.com/library/ms177445%28v=sql.105%29.aspx) 這篇文章，以將這類插入最佳化。
>
>

### <a name="insert-tables-bulkquery-parallel"></a>平行插入以進行更快速的資料移動
如果您要移動的資料很大，就可以在 PowerShell 指令碼中同時平行執行多個 BCP 命令來加快運作速度。

> [!NOTE]
> **巨量資料擷取** ：若要將大型和超大型資料集的資料載入予以最佳化，可以使用多個檔案群組和資料分割資料表來進行邏輯與實體資料庫資料表的資料分割。 如需關於建立和載入資料至資料分割資料表的詳細資訊，請參閱 [平行載入 SQL 資料分割資料表](parallel-load-sql-partitioned-tables.md)。
>
>

下列 PowerShell 指令碼範例將示範使用 BCP 平行插入：

    $NO_OF_PARALLEL_JOBS=2

     Set-ExecutionPolicy RemoteSigned #set execution policy for the script to execute
     # Define what each job does
       $ScriptBlock = {
           param($partitionnumber)

           #Explictly using SQL username password
           bcp database..tablename in datafile_path.csv -F 2 -f format_file_path.xml -U username@servername -S tcp:servername -P password -b block_size_to_move_in_single_attempt -t "," -r \n -o path_to_outputfile.$partitionnumber.txt

            #Trusted connection w.o username password (if you are using windows auth and are signed in with that credentials)
            #bcp database..tablename in datafile_path.csv -o path_to_outputfile.$partitionnumber.txt -h "TABLOCK" -F 2 -f format_file_path.xml  -T -b block_size_to_move_in_single_attempt -t "," -r \n
      }


    # Background processing of all partitions
    for ($i=1; $i -le $NO_OF_PARALLEL_JOBS; $i++)
    {
      Write-Debug "Submit loading partition # $i"
      Start-Job $ScriptBlock -Arg $i      
    }


    # Wait for it all to complete
    While (Get-Job -State "Running")
    {
      Start-Sleep 10
      Get-Job
    }

    # Getting the information back from the jobs
    Get-Job | Receive-Job
    Set-ExecutionPolicy Restricted #reset the execution policy


### <a name="insert-tables-bulkquery"></a>大量插入 SQL 查詢
[大量插入 SQL 查詢](https://msdn.microsoft.com/library/ms188365)可用來將資料從以資料列/資料行為基礎的檔案匯入資料庫 (支援類型請參閱[準備大量匯出或匯入的資料 (SQL Server)](https://msdn.microsoft.com/library/ms188609) 主題中的說明)。

此處提供一些大量插入的命令範例，如下所示：  

1. 分析您的資料，並在匯入之前設定任何自訂選項，以確定 SQL Server 資料庫會針對任何特殊欄位 (例如日期) 假設相同的格式。 以下範例示範如何將日期格式設為「年-月-日」 (如果您的資料包含「年-月-日」格式的日期)：

        SET DATEFORMAT ymd;    
2. 使用大量匯入陳述式來匯入資料：

        BULK INSERT <tablename>
        FROM    
        '<datafilename>'
        WITH
        (
        FirstRow=2,
        FIELDTERMINATOR =',', --this should be column separator in your data
        ROWTERMINATOR ='\n'   --this should be the row separator in your data
        )

### <a name="sql-builtin-utilities"></a>SQL Server 中的內建公用程式
您可以使用 SQL Server Integration Services (SSIS)，將資料從一般檔案匯入 Azure 上的 SQL Server VM。
SSIS 適用於兩種 Studio 環境。 如需詳細資料，請參閱 [Integration Services (SSIS) 和 Studio 環境](https://technet.microsoft.com/library/ms140028.aspx)：

* 如需 SQL Server Data Tools 的詳細資料，請參閱 [Microsoft SQL Server Data Tools](https://msdn.microsoft.com/data/tools.aspx)  
* 如需匯入/匯出精靈的詳細資料，請參閱 [SQL Server 匯入和匯出精靈](https://msdn.microsoft.com/library/ms141209.aspx)

## <a name="sqlonprem_to_sqlonazurevm"></a>從內部部署的 SQL Server 移動資料至 Azure VM 上的 SQL Server
您也可以使用下列移轉策略：

1. [將 SQL Server Database 部署到 Microsoft Azure VM 精靈](#deploy-a-sql-server-database-to-a-microsoft-azure-vm-wizard)
2. [匯出至一般檔案](#export-flat-file)
3. [SQL Database 移轉精靈](#sql-migration)
4. [資料庫備份和還原](#sql-backup)

我們將在下方說明這每一項內容：

### <a name="deploy-a-sql-server-database-to-a-microsoft-azure-vm-wizard"></a>將 SQL Server Database 部署到 Microsoft Azure VM 精靈
**將 SQL Server Database 部署到 Microsoft Azure VM 精靈** 是簡單且建議的方式，可用於將資料從內部部署 SQL Server 執行個體移至 Azure VM 上的 SQL Server。 如需詳細的步驟以及其他替代方案的討論，請參閱[將資料庫移轉至 Azure VM 上的 SQL Server](../../virtual-machines/windows/sql/virtual-machines-windows-migrate-sql.md)。

### <a name="export-flat-file"></a>匯出至一般檔案
有各種方法可用來從內部部署的 SQL Server 大量匯出資料，如 [資料的大量匯入及匯出 (SQL Server)](https://msdn.microsoft.com/library/ms175937.aspx) 主題所述。 本文件將提供大量複製程式 (BCP) 做為範例。 一旦將資料匯出至一般檔案之後，就可以使用大量匯入功能來將它匯入另一部 SQL Server。

1. 使用 BCP 公用程式，從內部部署的 SQL Server 將資料匯出至檔案，如下所示

    `bcp dbname..tablename out datafile.tsv -S    servername\sqlinstancename -T -t \t -t \n -c`
2. 針對步驟 1 中匯出的資料表結構描述使用 `create database` 和 `create table`，在 Azure 的 SQL Server VM 上建立資料庫和資料表。
3. 建立格式檔案，用以說明要匯出/匯入之資料的資料表結構描述。 [建立格式檔案 (SQL 伺服器)](https://msdn.microsoft.com/library/ms191516.aspx)中說明了格式檔案的詳細資料。

    從 SQL Server 電腦執行 BCP 時產生格式檔案

        bcp dbname..tablename format nul -c -x -f exportformatfilename.xml -S servername\sqlinstance -T -t \t -r \n

    針對 SQL Server 遠端執行 BCP 時產生格式檔案

        bcp dbname..tablename format nul -c -x -f  exportformatfilename.xml  -U username@servername.database.windows.net -S tcp:servername -P password  --t \t -r \n
4. 使用 [從檔案來源移動資料](#filesource_to_sqlonazurevm) 一節中所述的任何方法，將一般檔案中的資料移至 SQL Server。

### <a name="sql-migration"></a>SQL Database 移轉精靈
[SQL Server 資料庫移轉精靈](http://sqlazuremw.codeplex.com/) 提供方便使用的方式，讓您在兩個 SQL Server 執行個體之間移動資料。 它讓使用者能夠對應來源與目的地資料表之間的資料結構描述，選擇資料行類型和其他各種功能。 它會在幕後使用大量複製 (BCP) 功能。 SQL Database 移轉精靈歡迎畫面的螢幕擷取畫面如下所示。  

![SQL Server 移轉精靈][2]

### <a name="sql-backup"></a>資料庫備份和還原
SQL Server 支援：

1. [資料庫備份和還原功能](https://msdn.microsoft.com/library/ms187048.aspx) (兩者皆可為本機檔案或以 bacpac 匯出至 Blob) 和[資料層應用程式](https://msdn.microsoft.com/library/ee210546.aspx) (使用 bacpac)。
2. 使用複製的資料庫直接在 Azure 上建立 SQL Server VM，或者複製到現有 SQL Azure 資料庫的能力。 如需詳細資料，請參閱 [使用複製資料庫精靈](https://msdn.microsoft.com/library/ms188664.aspx)。

SQL Server Management Studio 的資料庫備份/還原選項的螢幕擷取畫面如下所示。

![SQL Server 匯入工具][1]

## <a name="resources"></a>資源
[將資料庫移轉至 Azure VM 上的 SQL Server](../../virtual-machines/windows/sql/virtual-machines-windows-migrate-sql.md)

[Azure 虛擬機器上的 SQL Server 概觀](../../virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview.md)

[1]: ./media/move-sql-server-virtual-machine/sqlserver_builtin_utilities.png
[2]: ./media/move-sql-server-virtual-machine/database_migration_wizard.png
