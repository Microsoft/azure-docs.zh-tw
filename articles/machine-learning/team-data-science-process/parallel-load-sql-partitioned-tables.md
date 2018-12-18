---
title: 建置和最佳化資料表，以便快速地將資料平行匯入到 Azure VM 上的 SQL Server | Microsoft Docs
description: 使用 SQL 資料分割資料表平行處理大量資料匯入
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: ff90fdb0-5bc7-49e8-aee7-678b54f901c8
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/09/2017
ms.author: deguhath
ms.openlocfilehash: f87bc1d8140bea9ebb09e45d42b27e201b474026
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39214337"
---
# <a name="parallel-bulk-data-import-using-sql-partition-tables"></a>使用 SQL 資料分割資料表平行處理大量資料匯入
本文件說明如何建置資料分割資料表，以快速的平行處理方式將大量資料匯入 SQL Server 資料庫。 若要將巨量資料載入/傳輸至 SQL Database，可使用*資料分割資料表和檢視*，來改善將資料匯入 SQL DB 和後續查詢的效能。 

## <a name="create-a-new-database-and-a-set-of-filegroups"></a>建立新的資料庫和一組檔案群組
* [建立新的資料庫](https://technet.microsoft.com/library/ms176061.aspx) (如果尚不存在)。
* 將資料庫檔案群組新增至將用來保留資料分割實體檔案的資料庫。 
* 這可透過 [CREATE DATABASE](https://technet.microsoft.com/library/ms176061.aspx) (如果是新的資料庫) 或 [ALTER DATABASE](https://msdn.microsoft.com/library/bb522682.aspx) (如果資料庫已經存在) 來完成。
* 將一或多個檔案 (視需要) 新增至每個資料庫檔案群組。
  
  > [!NOTE]
  > 指定將保留這個資料分割之資料的目標檔案群組，以及將儲存檔案群組資料的實體資料庫檔案名稱。
  > 
  > 

下列範例會建立含有三個檔案群組的新資料庫，這三個檔案群組不包括主要和記錄群組，且每個檔案群組中都會包含一個實體檔案。 資料庫檔案建立於預設的 SQL Server [資料] 資料夾中，如 SQL Server 執行個體中所設定。 如需關於預設檔案位置的詳細資訊，請參閱 [SQL Server 的預設和具名執行個體的檔案位置](https://msdn.microsoft.com/library/ms143547.aspx)。

    DECLARE @data_path nvarchar(256);
    SET @data_path = (SELECT SUBSTRING(physical_name, 1, CHARINDEX(N'master.mdf', LOWER(physical_name)) - 1)
      FROM master.sys.master_files
      WHERE database_id = 1 AND file_id = 1);

    EXECUTE ('
        CREATE DATABASE <database_name>
         ON  PRIMARY 
        ( NAME = ''Primary'', FILENAME = ''' + @data_path + '<primary_file_name>.mdf'', SIZE = 4096KB , FILEGROWTH = 1024KB ), 
         FILEGROUP [filegroup_1] 
        ( NAME = ''FileGroup1'', FILENAME = ''' + @data_path + '<file_name_1>.ndf'' , SIZE = 4096KB , FILEGROWTH = 1024KB ), 
         FILEGROUP [filegroup_2] 
        ( NAME = ''FileGroup1'', FILENAME = ''' + @data_path + '<file_name_2>.ndf'' , SIZE = 4096KB , FILEGROWTH = 1024KB ), 
         FILEGROUP [filegroup_3] 
        ( NAME = ''FileGroup1'', FILENAME = ''' + @data_path + '<file_name>.ndf'' , SIZE = 102400KB , FILEGROWTH = 10240KB ), 
         LOG ON 
        ( NAME = ''LogFileGroup'', FILENAME = ''' + @data_path + '<log_file_name>.ldf'' , SIZE = 1024KB , FILEGROWTH = 10%)
    ')

## <a name="create-a-partitioned-table"></a>建立資料分割資料表
為了要根據資料結構描述建立資料分割資料表，來對應到上一個步驟建立的資料庫檔案群組，您必須先建立資料分割函數和結構描述。 將資料大量匯入資料分割資料表時，記錄將根據資料分割配置分佈於檔案群組中，如下所述。

### <a name="1-create-a-partition-function"></a>1.建立資料分割函數
[建立資料分割函數](https://msdn.microsoft.com/library/ms187802.aspx)。此函數定義要在每個個別資料分割資料表中包含的值/界限範圍，例如，若要依 2013 年的月份來限制資料分割 (some\_datetime\_field)：
  
        CREATE PARTITION FUNCTION <DatetimeFieldPFN>(<datetime_field>)  
        AS RANGE RIGHT FOR VALUES (
            '20130201', '20130301', '20130401',
            '20130501', '20130601', '20130701', '20130801',
            '20130901', '20131001', '20131101', '20131201' )

### <a name="2-create-a-partition-scheme"></a>2.建立資料分割配置
[建立資料分割配置](https://msdn.microsoft.com/library/ms179854.aspx)。 此配置會將資料分割函數中的每個資料分割範圍對應至實體檔案群組，例如：
  
        CREATE PARTITION SCHEME <DatetimeFieldPScheme> AS  
        PARTITION <DatetimeFieldPFN> TO (
        <filegroup_1>, <filegroup_2>, <filegroup_3>, <filegroup_4>,
        <filegroup_5>, <filegroup_6>, <filegroup_7>, <filegroup_8>,
        <filegroup_9>, <filegroup_10>, <filegroup_11>, <filegroup_12> )
  
  若要根據函式/配置確認範圍會在每個資料分割中生效，請執行下列查詢：
  
        SELECT psch.name as PartitionScheme,
            prng.value AS ParitionValue,
            prng.boundary_id AS BoundaryID
        FROM sys.partition_functions AS pfun
        INNER JOIN sys.partition_schemes psch ON pfun.function_id = psch.function_id
        INNER JOIN sys.partition_range_values prng ON prng.function_id=pfun.function_id
        WHERE pfun.name = <DatetimeFieldPFN>

### <a name="3-create-a-partition-table"></a>3.建立資料分割資料表
根據您的資料結構描述來[建立資料分割資料表](https://msdn.microsoft.com/library/ms174979.aspx)，並指定用來為資料表進行資料分割的資料分割配置和條件約束欄位，例如：
  
        CREATE TABLE <table_name> ( [include schema definition here] )
        ON <TablePScheme>(<partition_field>)

如需詳細資訊，請參閱 [建立分割區資料表及索引](https://msdn.microsoft.com/library/ms188730.aspx)。

## <a name="bulk-import-the-data-for-each-individual-partition-table"></a>大量匯入每個個別資料分割資料表的資料

* 您可以使用 BCP、BULK INSERT 或其他方法，例如 [SQL Server 移轉精靈](http://sqlazuremw.codeplex.com/)。 所提供的範例會使用 BCP 方法。
* [修改資料庫](https://msdn.microsoft.com/library/bb522682.aspx)，將交易記錄配置變更為 BULK_LOGGED，以便將記錄額外負荷降到最低，例如：
  
        ALTER DATABASE <database_name> SET RECOVERY BULK_LOGGED
* 若要加速資料載入，可以平行方式啟動大量匯入作業。 如需加速將巨量資料大量匯入 SQL Server 資料庫的提示，請參閱 [載入 1 TB 的時間少於 1 小時](http://blogs.msdn.com/b/sqlcat/archive/2006/05/19/602142.aspx)(英文)。

下列 PowerShell 指令碼是使用 BCP 平行載入資料的範例。

    # Set database name, input data directory, and output log directory
    # This example loads comma-separated input data files
    # The example assumes the partitioned data files are named as <base_file_name>_<partition_number>.csv
    # Assumes the input data files include a header line. Loading starts at line number 2.

    $dbname = "<database_name>"
    $indir  = "<path_to_data_files>"
    $logdir = "<path_to_log_directory>"

    # Select authentication mode
    $sqlauth = 0

    # For SQL authentication, set the server and user credentials
    $sqlusr = "<user@server>"
    $server = "<tcp:serverdns>"
    $pass   = "<password>"

    # Set number of partitions per table - Should match the number of input data files per table
    $numofparts = <number_of_partitions>

    # Set table name to be loaded, basename of input data files, input format file, and number of partitions
    $tbname = "<table_name>"
    $basename = "<base_input_data_filename_no_extension>"
    $fmtfile = "<full_path_to_format_file>"

    # Create log directory if it does not exist
    New-Item -ErrorAction Ignore -ItemType directory -Path $logdir

    # BCP example using Windows authentication
    $ScriptBlock1 = {
       param($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $num)
       bcp ($dbname + ".." + $tbname) in ($indir + "\" + $basename + "_" + $num + ".csv") -o ($logdir + "\" + $tbname + "_" + $num + ".txt") -h "TABLOCK" -F 2 -C "RAW" -f ($fmtfile) -T -b 2500 -t "," -r \n
    }

    # BCP example using SQL authentication
    $ScriptBlock2 = {
       param($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $num, $sqlusr, $server, $pass)
       bcp ($dbname + ".." + $tbname) in ($indir + "\" + $basename + "_" + $num + ".csv") -o ($logdir + "\" + $tbname + "_" + $num + ".txt") -h "TABLOCK" -F 2 -C "RAW" -f ($fmtfile) -U $sqlusr -S $server -P $pass -b 2500 -t "," -r \n
    }

    # Background processing of all partitions
    for ($i=1; $i -le $numofparts; $i++)
    {
       Write-Output "Submit loading trip and fare partitions # $i"
       if ($sqlauth -eq 0) {
          # Use Windows authentication
          Start-Job -ScriptBlock $ScriptBlock1 -Arg ($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $i)
       } 
       else {
          # Use SQL authentication
          Start-Job -ScriptBlock $ScriptBlock2 -Arg ($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $i, $sqlusr, $server, $pass)
       }
    }

    Get-Job

    # Optional - Wait till all jobs complete and report date and time
    date
    While (Get-Job -State "Running") { Start-Sleep 10 }
    date


## <a name="create-indexes-to-optimize-joins-and-query-performance"></a>建立索引以將聯結和查詢效能最佳化
* 如果您會從多個資料表擷取資料來進行模型化，請在聯結索引鍵上建立索引來提升聯結效能。
* [建立索引](https://technet.microsoft.com/library/ms188783.aspx) (叢集或非叢集) 會將每個資料分割的目標設定為相同的檔案群組，例如：
  
        CREATE CLUSTERED INDEX <table_idx> ON <table_name>( [include index columns here] )
        ON <TablePScheme>(<partition)field>)
  或者，
  
        CREATE INDEX <table_idx> ON <table_name>( [include index columns here] )
        ON <TablePScheme>(<partition)field>)
  
  > [!NOTE]
  > 您可以選擇在大量匯入資料之前建立索引。 在大量匯入之前建立索引，會讓資料載入的速度變慢。
  > 
  > 

## <a name="advanced-analytics-process-and-technology-in-action-example"></a>進階分析程序和技術實務範例
如需搭配使用 Team Data Science Process 與公用資料集的端對端逐步解說範例，請參閱 [Team Data Science Process 實務：使用 SQL Server](sql-walkthrough.md)。

