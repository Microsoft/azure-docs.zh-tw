---
title: Team Data Science Process 實務：使用 SQL 資料倉儲 | Microsoft Docs
description: 進階分析程序和技術實務
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 88ba8e28-0bd7-49fe-8320-5dfa83b65724
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/24/2017
ms.author: deguhath
ms.openlocfilehash: 6178c4a55d24bb37aae787129c9a0d390a2e536b
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39226221"
---
# <a name="the-team-data-science-process-in-action-using-sql-data-warehouse"></a>Team Data Science Process 實務：使用 SQL 資料倉儲
在本教學課程中，我們將引導您使用 SQL 資料倉儲 (SQL DW)，針對可公開使用的資料集 ( [NYC 計程車車程](http://www.andresmh.com/nyctaxitrips/) 資料集) 建置和部署機器學習服務模型。 所建構的二元分類模型可預測是否已針對某趟車程支付小費，並且也會討論預測支付的小費金額分佈的多元分類模型和迴歸模型。

此程序會遵循 [Team Data Science Process (TDSP)](https://azure.microsoft.com/documentation/learning-paths/cortana-analytics-process/) 工作流程。 我們會示範如何設定資料科學環境、如何將資料載入 SQL DW，以及如何使用 SQL DW 或 IPython Notebook 來探索要模型化的資料和工程功能。 然後，我們會示範如何使用 Azure Machine Learning 建置和部署模型。

## <a name="dataset"></a>NYC 計程車車程資料集
「NYC 計程車車程」資料是由約 20GB 的 CSV 壓縮檔 (未壓縮時可達 48GB) 所組成，裡面記錄了超過 1 億 7300 萬筆個別車程及針對每趟車程所支付的費用。 每趟車程記錄包括上車和下車的位置與時間、匿名的計程車司機駕照號碼，以及計程車牌照 (計程車的唯一識別碼) 號碼。 資料涵蓋 2013 年的所有車程，並且每月會在下列兩個資料集中加以提供：

1. **trip_data.csv** 檔案包含車程的詳細資訊，例如，乘客數、上車和下車地點、車程持續時間，以及車程長度。 以下是一些範例記錄：
   
        medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude
        89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868
2. **trip_fare.csv** 檔案包含針對每趟車程所支付之費用的詳細資訊，例如付款類型、費用金額、銷售稅和稅金、小費和服務費，以及支付的總金額。 以下是一些範例記錄：
   
        medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount
        89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5

聯結 trip\_data 和 trip\_fare 的「唯一索引鍵」是由下列三個欄位所組成：

* medallion、
* hack\_license 和
* pickup\_datetime。

## <a name="mltasks"></a>處理三種類型的預測工作
我們根據 *tip\_amount* 將三個預測問題公式化來說明三種類型的模型化工作：

1. **二元分類**：預測是否已支付某趟車程的小費 (即大於美金 $0 元的 *tip\_amount* 為正面範例)，而等於美金 $0 元的 *tip\_amount* 為負面範例。
2. **多類別分類**：預測針對該車程所支付之小費的金額範圍。 我們將 tip\_amount 分成五個分類收納組或類別：
   
        Class 0 : tip_amount = $0
        Class 1 : tip_amount > $0 and tip_amount <= $5
        Class 2 : tip_amount > $5 and tip_amount <= $10
        Class 3 : tip_amount > $10 and tip_amount <= $20
        Class 4 : tip_amount > $20
3. **迴歸工作**：預測針對某趟車程支付的小費金額。  

## <a name="setup"></a>設定適用於進階分析的 Azure 資料科學環境
若要設定您的 Azure 資料科學環境，請遵循下列步驟。

**建立自己的 Azure Blob 儲存體帳戶**

* 當您在佈建自己的 Azure Blob 儲存體時，請為 Azure Blob 儲存體選擇位於或最接近「美國中南部」 的地理位置 (即儲存 NYC 計程車資料的位置)。 該資料會使用 AzCopy 從公用 Blob 儲存體容器複製到您自己的儲存體帳戶中的容器。 您的 Azure Blob 儲存體越接近美國中南部，就能越快完成這項工作 (步驟 4)。
* 若要建立自己的 Azure 儲存體帳戶，請遵循 [關於 Azure 儲存體帳戶](../../storage/common/storage-create-storage-account.md)中概述的步驟。 請務必記下下列儲存體帳戶認證的值，因為我們會在本逐步解說稍後的地方用到它們。
  
  * **儲存體帳戶名稱**
  * **儲存體帳戶金鑰**
  * **容器名稱** (您想要在 Azure Blob 儲存體中用來儲存資料的容器)

**佈建 Azure SQL DW 執行個體。**
遵循 [建立 SQL 資料倉儲](../../sql-data-warehouse/sql-data-warehouse-get-started-provision.md) 中的說明來佈建 SQL 資料倉儲執行個體。 請務必記下下列 SQL 資料倉儲認證以用於稍後的步驟。

* **伺服器名稱**：<server Name>.database.windows.net
* **SQLDW (資料庫) 名稱**
* **使用者名稱**
* **密碼**

**安裝 Visual Studio 和 SQL Server Data Tools。** 如需指示，請參閱 [安裝適用於 SQL 資料倉儲的 Visual Studio 2015 及/或 SSDT (SQL Server Data Tools)](../../sql-data-warehouse/sql-data-warehouse-install-visual-studio.md)中概述的步驟。

**使用 Visual Studio 連接到 Azure SQL DW。** 如需指示，請參閱[使用 Visual Studio 連接到 Azure SQL 資料倉儲](../../sql-data-warehouse/sql-data-warehouse-connect-overview.md)中的步驟 1 和 2。

> [!NOTE]
> 在您於 SQL 資料倉儲中建立的資料庫上執行下列 SQL 查詢 (而不是連接主題的步驟 3 中所提供的查詢)，以 **建立主要金鑰**。
> 
> 

    BEGIN TRY
           --Try to create the master key
        CREATE MASTER KEY
    END TRY
    BEGIN CATCH
           --If the master key exists, do nothing
    END CATCH;

**在 Azure 訂用帳戶下建立 Azure Machine Learning 工作區。** 如需指示，請參閱 [建立 Azure Machine Learning 工作區](../studio/create-workspace.md)中概述的步驟。

## <a name="getdata"></a>將資料載入 SQL 資料倉儲
開啟 Windows PowerShell 命令主控台。 執行下列 PowerShell 命令，將我們在 GitHub 上與您分享的範例 SQL 指令碼檔案，下載到您使用 *-DestDir* 參數指定的本機目錄中。 您可以將 *-DestDir* 參數的值變更為任何本機目錄。 如果 *-DestDir* 不存在，PowerShell 指令碼會加以建立。

> [!NOTE]
> 如果需要系統管理員權限才能建立或寫入 **DestDir** 目錄，您可能需要在執行下列 PowerShell 指令碼時 *以系統管理員身分執行* 。
> 
> 

    $source = "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/SQLDW/Download_Scripts_SQLDW_Walkthrough.ps1"
    $ps1_dest = "$pwd\Download_Scripts_SQLDW_Walkthrough.ps1"
    $wc = New-Object System.Net.WebClient
    $wc.DownloadFile($source, $ps1_dest)
    .\Download_Scripts_SQLDW_Walkthrough.ps1 –DestDir 'C:\tempSQLDW'

成功執行之後，目前的工作目錄會變更為 *-DestDir*。 您應該會看到如下畫面：

![][19]

在 *-DestDir*中，以系統管理員模式執行下列 PowerShell 指令碼：

    ./SQLDW_Data_Import.ps1

當 PowerShell 指令碼第一次執行時，系統會要求您輸入 Azure SQL DW 和 Azure Blob 儲存體帳戶的資訊。 當這個 PowerShell 指令碼完成第一次執行時，您所輸入的認證即已寫入現在的工作目錄中的組態檔 SQLDW.conf。 日後再次執行這個 PowerShell 指令碼檔案時，就會有從這個組態檔讀取所有所需參數的選項。 如果您需要變更一些參數，您可以選擇在提示時於畫面上輸入參數 (方法是刪除此組態檔，並依提示輸入參數值)，或是變更參數值 (方法是編輯 *-DestDir* 目錄中的 SQLDW.conf 檔案)。

> [!NOTE]
> 為了避免結構描述名稱與 Azure SQL DW 中現存的名稱發生衝突，在直接從 SQLDW.conf 檔案讀取參數時，都會對 SQLDW.conf 檔案中的結構描述名稱加上 3 位數的隨機數字，以做為每次執行的預設結構描述名稱。 PowerShell 指令碼可能會提示您輸入結構描述名稱：使用者可自行指定此名稱。
> 
> 

此 **PowerShell 指令碼** 檔案會完成下列工作：

* 如果尚未安裝 AzCopy，請**下載並安裝 AzCopy**
  
        $AzCopy_path = SearchAzCopy
        if ($AzCopy_path -eq $null){
               Write-Host "AzCopy.exe is not found in C:\Program Files*. Now, start installing AzCopy..." -ForegroundColor "Yellow"
            InstallAzCopy
            $AzCopy_path = SearchAzCopy
        }
            $env_path = $env:Path
            for ($i=0; $i -lt $AzCopy_path.count; $i++){
                if ($AzCopy_path.count -eq 1){
                    $AzCopy_path_i = $AzCopy_path
                } else {
                    $AzCopy_path_i = $AzCopy_path[$i]
                }
                if ($env_path -notlike '*' +$AzCopy_path_i+'*'){
                    Write-Host $AzCopy_path_i 'not in system path, add it...'
                    [Environment]::SetEnvironmentVariable("Path", "$AzCopy_path_i;$env_path", "Machine")
                    $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine")
                    $env_path = $env:Path
                }
* **將資料複製到私人 Blob 儲存體帳戶** 
  
        Write-Host "AzCopy is copying data from public blob to yo storage account. It may take a while..." -ForegroundColor "Yellow"
        $start_time = Get-Date
        AzCopy.exe /Source:$Source /Dest:$DestURL /DestKey:$StorageAccountKey /S
        $end_time = Get-Date
        $time_span = $end_time - $start_time
        $total_seconds = [math]::Round($time_span.TotalSeconds,2)
        Write-Host "AzCopy finished copying data. Please check your storage account to verify." -ForegroundColor "Yellow"
        Write-Host "This step (copying data from public blob to your storage account) takes $total_seconds seconds." -ForegroundColor "Green"
* **使用 Polybase 載入資料 (藉由執行 LoadDataToSQLDW.sql) 到您的 Azure SQL DW** 。
  
  * 建立結構描述
    
          EXEC (''CREATE SCHEMA {schemaname};'');
  * 建立資料庫範圍認證
    
          CREATE DATABASE SCOPED CREDENTIAL {KeyAlias}
          WITH IDENTITY = ''asbkey'' ,
          Secret = ''{StorageAccountKey}''
  * 建立 Azure 儲存體 Blob 的外部資料來源。
    
          CREATE EXTERNAL DATA SOURCE {nyctaxi_trip_storage}
          WITH
          (
              TYPE = HADOOP,
              LOCATION =''wasbs://{ContainerName}@{StorageAccountName}.blob.core.windows.net'',
              CREDENTIAL = {KeyAlias}
          )
          ;
    
          CREATE EXTERNAL DATA SOURCE {nyctaxi_fare_storage}
          WITH
          (
              TYPE = HADOOP,
              LOCATION =''wasbs://{ContainerName}@{StorageAccountName}.blob.core.windows.net'',
              CREDENTIAL = {KeyAlias}
          )
          ;
  * 建立 CSV 檔案的外部檔案格式。 資料是未經壓縮的，而欄位會以縱線字元分隔。
    
          CREATE EXTERNAL FILE FORMAT {csv_file_format}
          WITH
          (   
              FORMAT_TYPE = DELIMITEDTEXT,
              FORMAT_OPTIONS  
              (
                  FIELD_TERMINATOR ='','',
                  USE_TYPE_DEFAULT = TRUE
              )
          )
          ;
  * 為 Azure Blob 儲存體的 NYC 計程車資料集建立外部 fare 和 trip 資料表。
    
          CREATE EXTERNAL TABLE {external_nyctaxi_fare}
          (
              medallion varchar(50) not null,
              hack_license varchar(50) not null,
              vendor_id char(3),
              pickup_datetime datetime not null,
              payment_type char(3),
              fare_amount float,
              surcharge float,
              mta_tax float,
              tip_amount float,
              tolls_amount float,
              total_amount float
          )
          with (
              LOCATION    = ''/nyctaxifare/'',
              DATA_SOURCE = {nyctaxi_fare_storage},
              FILE_FORMAT = {csv_file_format},
              REJECT_TYPE = VALUE,
              REJECT_VALUE = 12     
          )  

            CREATE EXTERNAL TABLE {external_nyctaxi_trip}
            (
                   medallion varchar(50) not null,
                   hack_license varchar(50)  not null,
                   vendor_id char(3),
                   rate_code char(3),
                   store_and_fwd_flag char(3),
                   pickup_datetime datetime  not null,
                   dropoff_datetime datetime,
                   passenger_count int,
                   trip_time_in_secs bigint,
                   trip_distance float,
                   pickup_longitude varchar(30),
                   pickup_latitude varchar(30),
                   dropoff_longitude varchar(30),
                   dropoff_latitude varchar(30)
            )
            with (
                LOCATION    = ''/nyctaxitrip/'',
                DATA_SOURCE = {nyctaxi_trip_storage},
                FILE_FORMAT = {csv_file_format},
                REJECT_TYPE = VALUE,
                REJECT_VALUE = 12         
            )

    - 從 Azure Blob 儲存體將外部資料表中的資料載入 SQL 資料倉儲

            CREATE TABLE {schemaname}.{nyctaxi_fare}
            WITH
            (   
                CLUSTERED COLUMNSTORE INDEX,
                DISTRIBUTION = HASH(medallion)
            )
            AS
            SELECT *
            FROM   {external_nyctaxi_fare}
            ;

            CREATE TABLE {schemaname}.{nyctaxi_trip}
            WITH
            (   
                CLUSTERED COLUMNSTORE INDEX,
                DISTRIBUTION = HASH(medallion)
            )
            AS
            SELECT *
            FROM   {external_nyctaxi_trip}
            ;

    - 建立範例資料的資料表 (NYCTaxi_Sample)，並透過選取 trip 和 fare 資料表上的 SQL 查詢對範例資料表插入資料。 (本逐步解說的某些步驟需要使用此範例資料表。)

            CREATE TABLE {schemaname}.{nyctaxi_sample}
            WITH
            (   
                CLUSTERED COLUMNSTORE INDEX,
                DISTRIBUTION = HASH(medallion)
            )
            AS
            (
                SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount, f.total_amount, f.tip_amount,
                tipped = CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END,
                tip_class = CASE
                        WHEN (tip_amount = 0) THEN 0
                        WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
                        WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
                        WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
                        ELSE 4
                    END
                FROM {schemaname}.{nyctaxi_trip} t, {schemaname}.{nyctaxi_fare} f
                WHERE datepart("mi",t.pickup_datetime) = 1
                AND t.medallion = f.medallion
                AND   t.hack_license = f.hack_license
                AND   t.pickup_datetime = f.pickup_datetime
                AND   pickup_longitude <> ''0''
                AND   dropoff_longitude <> ''0''
            )
            ;

儲存體帳戶的地理位置會影響載入時間。

> [!NOTE]
> 根據私人 Blob 儲存體帳戶所在的地理位置，將公用 Blob 中的資料複製到私人儲存體帳戶的程序可能需要大約 15 分鐘或更久時間，而將儲存體帳戶中的資料載入到 Azure SQL DW 的程序則可能需要 20 分鐘或更久時間。  
> 
> 

您必須決定當您有重複的來源和目的地檔案時該如何做。

> [!NOTE]
> 如果私人 Blob 儲存體帳戶中已有要從公用 Blob 儲存體複製到私人 Blob 儲存體帳戶的 .csv 檔案，AzCopy 會詢問您是否要加以覆寫。 如果不想加以覆寫，在出現提示時輸入 **n** 。 如果想要**全部**覆寫，請在出現提示時輸入 **a**。 您也可以輸入 **y** 來個別覆寫 .csv 檔案。
> 
> 

![圖 #21][21]

您可以使用自己的資料。 如果資料位於內部部署電腦的現實應用程式中，您仍可以使用 AzCopy 將內部部署資料上傳至私人 Azure Blob 儲存體。 您只需要在 AzCopy 命令中，將 PowerShell 指令碼檔案中的 **Source** 位置 (`$Source = "http://getgoing.blob.core.windows.net/public/nyctaxidataset"`) 變更為包含您的資料的本機目錄。

> [!TIP]
> 如果資料已位於私人 Azure Blob 儲存體的現實應用程式中，您可以略過 PowerShell 指令碼中的 AzCopy 步驟，並直接將資料上傳到 Azure SQL DW。 這將需要另外編輯指令碼，使它符合您的資料格式。
> 
> 

這個 Powershell 指令碼也會將 Azure SQL DW 資訊插入資料探索範例檔案 SQLDW_Explorations.sql、SQLDW_Explorations.ipynb 和 SQLDW_Explorations_Scripts.py，讓這三個檔案能夠在 PowerShell 指令碼完成後就立即提供試用。

成功執行之後，您會看到如下畫面：

![][20]

## <a name="dbexplore"></a>Azure SQL 資料倉儲中的資料探索和特徵工程
在本節中，我們會使用 **Visual Studio Data Tools**直接對 Azure SQL DW 執行 SQL 查詢，以探索資料和產生特徵。 本節中使用的所有 SQL 查詢都能在名為 *SQLDW_Explorations.sql* 的範例指令碼中找到。 PowerShell 指令碼已將此檔案下載到您的本機目錄。 您也可以從 [GitHub](https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/SQLDW/SQLDW_Explorations.sql) 擷取此檔案。 但 GitHub 中的檔案並未插入 Azure SQL DW 資訊。

使用 Visual Studio 與 SQL DW 登入名稱和密碼連接到您的 Azure SQL DW，然後開啟 **SQL 物件總管** 確認資料庫和資料表已匯入。 擷取 *SQLDW_Explorations.sql* 檔案。

> [!NOTE]
> 若要開啟 Parallel Data Warehouse (PDW) 查詢編輯器，請於在 [SQL 物件總管] 中選取 PDW 時使用「新增查詢」命令。 PDW 不支援標準的 SQL 查詢編輯器。
> 
> 

以下是本節所執行之資料探索和功能產生工作的類型：

* 在變動的時間範圍中探索數個欄位的資料分佈。
* 調查經度和緯度欄位的資料品質。
* 根據 **tip\_amount** 產生二進位和多類別分類標籤。
* 產生功能，並計算或比較車程距離。
* 聯結這兩個資料表，並擷取將用來建置模型的隨機取樣。

### <a name="data-import-verification"></a>資料匯入驗證
這些查詢可以快速驗證先前使用 Polybase 的平行大量匯入所填入的資料表中的資料列與資料行數目，

    -- Report number of rows in table <nyctaxi_trip> without table scan
    SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_trip>')

    -- Report number of columns in table <nyctaxi_trip>
    SELECT COUNT(*) FROM information_schema.columns WHERE table_name = '<nyctaxi_trip>' AND table_schema = '<schemaname>'

**輸出：** 您應該有 173,179,759 個資料列和 14 個資料行。

### <a name="exploration-trip-distribution-by-medallion"></a>探索：依據 medallion 的車程分佈
此範例查詢可找出在指定期間內完成超過 100 趟車程的計程車牌照 (計程車號碼)。 資料分割資料表存取的條件是以 **pickup\_datetime** 資料分割配置為依據，因為可為查詢帶來好處。 查詢完整資料集也會使用資料分割資料表及 (或) 索引掃描。

    SELECT medallion, COUNT(*)
    FROM <schemaname>.<nyctaxi_fare>
    WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
    GROUP BY medallion
    HAVING COUNT(*) > 100

**輸出：** 查詢應該會傳回包含資料列的資料表，資料列中指出 13,369 個圓形徽章 (計程車) 及它們於 2013 年完成的車程數。 最後一個資料行包含所完成之車程數的計數。

### <a name="exploration-trip-distribution-by-medallion-and-hacklicense"></a>探索：依據 medallion 和 hack_license 的車程分佈
此範例可找出在指定期間內完成超過 100 趟車程的圓形徽章 (計程車號碼) 和計程車駕照號碼 (司機)。

    SELECT medallion, hack_license, COUNT(*)
    FROM <schemaname>.<nyctaxi_fare>
    WHERE pickup_datetime BETWEEN '20130101' AND '20130131'
    GROUP BY medallion, hack_license
    HAVING COUNT(*) > 100

**輸出：** 查詢應該會傳回包含 13,369 個資料列的資料表，資料列中指出於 2013 年完成超過 100 趟車程之 13,369 部車/司機的識別碼。 最後一個資料行包含所完成之車程數的計數。

### <a name="data-quality-assessment-verify-records-with-incorrect-longitude-andor-latitude"></a>資料品質評估：驗證含有不正確經度和/或緯度的記錄
此範例會檢查是否有任何經度和 (或) 緯度欄位包含無效值 (弧度角度應介於-90 和 90 之間)，或是具有 (0，0) 座標。

    SELECT COUNT(*) FROM <schemaname>.<nyctaxi_trip>
    WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
    AND  (CAST(pickup_longitude AS float) NOT BETWEEN -90 AND 90
    OR    CAST(pickup_latitude AS float) NOT BETWEEN -90 AND 90
    OR    CAST(dropoff_longitude AS float) NOT BETWEEN -90 AND 90
    OR    CAST(dropoff_latitude AS float) NOT BETWEEN -90 AND 90
    OR    (pickup_longitude = '0' AND pickup_latitude = '0')
    OR    (dropoff_longitude = '0' AND dropoff_latitude = '0'))

**輸出：** 查詢傳回 837,467 個經度和/或緯度欄位無效的車程。

### <a name="exploration-tipped-vs-not-tipped-trips-distribution"></a>探索：已支付小費和未支付小費的車程分佈
此範例會尋找在指定期間內 (或者，如果涵蓋一整年，則如這裡所設定是在整個資料集內)，已收到小費的車程數目，以及未收到小費的車程數目。 此分佈會反映二進位標籤分佈，以便稍後用來將二進位分類模型化。

    SELECT tipped, COUNT(*) AS tip_freq FROM (
      SELECT CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped, tip_amount
      FROM <schemaname>.<nyctaxi_fare>
      WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
    GROUP BY tipped

**輸出：** 查詢應該會傳回下列 2013 年的小費頻率：90、447、622 已付小費及 82,264,709 未付小費。

### <a name="exploration-tip-classrange-distribution"></a>探索：小費類別/範圍分佈
此範例會計算在指定期間內 (或者，如果涵蓋一整年，則是在整個資料庫中) 小費範圍的分佈。 這是標籤類別的分佈，會在稍後用來將多類別分類模型化。

    SELECT tip_class, COUNT(*) AS tip_freq FROM (
        SELECT CASE
            WHEN (tip_amount = 0) THEN 0
            WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
            WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
            WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
            ELSE 4
        END AS tip_class
    FROM <schemaname>.<nyctaxi_fare>
    WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
    GROUP BY tip_class

**輸出：**

| tip_class | tip_freq |
| --- | --- |
| 1 |82230915 |
| 2 |6198803 |
| 3 |1932223 |
| 0 |82264625 |
| 4 |85765 |

### <a name="exploration-compute-and-compare-trip-distance"></a>探索：計算並比較車程距離
此範例會將上車和下車的經緯度轉換為 SQL 地理位置點、使用 SQL 地理位置點的差距來計算車程距離，然後傳回結果的隨機取樣以進行比較。 此範例只會使用稍早所提供的資料品質評估查詢，將結果限制為有效座標。

    /****** Object:  UserDefinedFunction [dbo].[fnCalculateDistance] ******/
    SET ANSI_NULLS ON
    GO

    SET QUOTED_IDENTIFIER ON
    GO

    IF EXISTS (SELECT * FROM sys.objects WHERE type IN ('FN', 'IF') AND name = 'fnCalculateDistance')
      DROP FUNCTION fnCalculateDistance
    GO

    -- User-defined function to calculate the direct distance  in mile between two geographical coordinates.
    CREATE FUNCTION [dbo].[fnCalculateDistance] (@Lat1 float, @Long1 float, @Lat2 float, @Long2 float)

    RETURNS float
    AS
    BEGIN
          DECLARE @distance decimal(28, 10)
          -- Convert to radians
          SET @Lat1 = @Lat1 / 57.2958
          SET @Long1 = @Long1 / 57.2958
          SET @Lat2 = @Lat2 / 57.2958
          SET @Long2 = @Long2 / 57.2958
          -- Calculate distance
          SET @distance = (SIN(@Lat1) * SIN(@Lat2)) + (COS(@Lat1) * COS(@Lat2) * COS(@Long2 - @Long1))
          --Convert to miles
          IF @distance <> 0
          BEGIN
            SET @distance = 3958.75 * ATAN(SQRT(1 - POWER(@distance, 2)) / @distance);
          END
          RETURN @distance
    END
    GO

    SELECT pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude,
    dbo.fnCalculateDistance(pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude) AS DirectDistance
    FROM <schemaname>.<nyctaxi_trip>
    WHERE datepart("mi",pickup_datetime)=1
    AND CAST(pickup_latitude AS float) BETWEEN -90 AND 90
    AND CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
    AND pickup_longitude != '0' AND dropoff_longitude != '0'

### <a name="feature-engineering-using-sql-functions"></a>使用 SQL 函式的功能工程
有時 SQL 函式會是有效率進行功能工程的合適選項。 在此逐步解說中，我們已定義 SQL 函式來計算上車與下車位置之間的直線距離。 您可以在 **Visual Studio Data Tools**中執行下列 SQL 指令碼。

以下是定義距離函式的 SQL 指令碼。

    SET ANSI_NULLS ON
    GO

    SET QUOTED_IDENTIFIER ON
    GO

    IF EXISTS (SELECT * FROM sys.objects WHERE type IN ('FN', 'IF') AND name = 'fnCalculateDistance')
      DROP FUNCTION fnCalculateDistance
    GO

    -- User-defined function calculate the direct distance between two geographical coordinates.
    CREATE FUNCTION [dbo].[fnCalculateDistance] (@Lat1 float, @Long1 float, @Lat2 float, @Long2 float)

    RETURNS float
    AS
    BEGIN
          DECLARE @distance decimal(28, 10)
          -- Convert to radians
          SET @Lat1 = @Lat1 / 57.2958
          SET @Long1 = @Long1 / 57.2958
          SET @Lat2 = @Lat2 / 57.2958
          SET @Long2 = @Long2 / 57.2958
          -- Calculate distance
          SET @distance = (SIN(@Lat1) * SIN(@Lat2)) + (COS(@Lat1) * COS(@Lat2) * COS(@Long2 - @Long1))
          --Convert to miles
          IF @distance <> 0
          BEGIN
            SET @distance = 3958.75 * ATAN(SQRT(1 - POWER(@distance, 2)) / @distance);
          END
          RETURN @distance
    END
    GO

以下是呼叫此函式以在 SQL 查詢中產生功能的範例：

    -- Sample query to call the function to create features
    SELECT pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude,
    dbo.fnCalculateDistance(pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude) AS DirectDistance
    FROM <schemaname>.<nyctaxi_trip>
    WHERE datepart("mi",pickup_datetime)=1
    AND CAST(pickup_latitude AS float) BETWEEN -90 AND 90
    AND CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
    AND pickup_longitude != '0' AND dropoff_longitude != '0'

**輸出：** 此查詢會產生包含上下車的經緯度及所對應之直接距離 (英里)的資料表 (包含 2,803,538 個資料列) 。 前 3 個資料列的結果如下：

|  | pickup_latitude | pickup_longitude | dropoff_latitude | dropoff_longitude | DirectDistance |
| --- | --- | --- | --- | --- | --- |
| 1 |40.731804 |-74.001083 |40.736622 |-73.988953 |.7169601222 |
| 2 |40.715794 |-74,010635 |40.725338 |-74.00399 |.7448343721 |
| 3 |40.761456 |-73.999886 |40.766544 |-73.988228 |0.7037227967 |

### <a name="prepare-data-for-model-building"></a>準備資料以進行模型建置
下列查詢可聯結 **nyctaxi\_trip** 和 **nyctaxi\_fare** 資料表、產生二進位分類標籤 **tipped**、多類別分類標籤 **tip\_class**，以及從完整聯結的資料集中擷取樣本。 根據上車時間擷取車程子集即可完成取樣。  您可以複製此查詢，再直接貼到 [Azure Machine Learning Studio](https://studio.azureml.net) 的[匯入資料][import-data]模組中，以便從 Azure 中的 SQL 資料庫執行個體直接內嵌資料。 查詢會排除含有不正確 (0, 0) 座標的記錄。

    SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount,     f.total_amount, f.tip_amount,
        CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped,
        CASE WHEN (tip_amount = 0) THEN 0
            WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
            WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
            WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
            ELSE 4
        END AS tip_class
    FROM <schemaname>.<nyctaxi_trip> t, <schemaname>.<nyctaxi_fare> f
    WHERE datepart("mi",t.pickup_datetime) = 1
    AND   t.medallion = f.medallion
    AND   t.hack_license = f.hack_license
    AND   t.pickup_datetime = f.pickup_datetime
    AND   pickup_longitude != '0' AND dropoff_longitude != '0'

當您準備好繼續進行 Azure Machine Learning，您可以：  

1. 儲存最後一個 SQL 查詢以擷取和取樣資料，然後複製該查詢並直接貼到 Azure Machine Learning 中的[匯入資料][import-data]模組，或者
2. 將您打算用來建置模型的取樣和工程設計資料保存在新的 SQL DW 資料表中，然後在 Azure Machine Learning 的[匯入資料][import-data]模組中使用該新的資料表。 先前步驟中的 PowerShell 指令碼已為您完成此作業。 您可以在「匯入資料」模組中直接讀取此資料表。

## <a name="ipnb"></a>IPython Notebook 中的資料探索和特徵工程設計
在本節中，我們將在先前建立的 SQL DW 中進行 Python 和 SQL 查詢，藉此探索資料和產生功能。 名為 **SQLDW_Explorations.ipynb** 的 IPython Notebook 範例和 Python 指令碼檔案 **SQLDW_Explorations_Scripts.py** 已下載到您的本機目錄中。 您也可以在 [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/SQLDW)上取得這兩個檔案。 在 Python 指令碼中，這兩個檔案是相同的。 我們會提供 Python 指令碼檔案給您，以免您沒有 IPython Notebook 伺服器。 這兩個範例 Python 檔案是以 **Python 2.7**設計。

範例 IPython Notebook 和 Python 指令碼檔案中下載到本機電腦的所需 Azure SQL DW 資訊先前已由 PowerShell 指令碼插入。 因此，不必進行任何修改就可以執行。

如果您已經設定 AzureML 工作區，則可以直接將範例 IPython Notebook 上傳至 AzureML IPython Notebook 服務，並開始執行。 以下是上傳至 AzureML IPython Notebook 服務的步驟：

1. 登入 AzureML 工作區，按一下頂端的 [Studio]，然後按一下網頁左側的 [NOTEBOOKS]。
   
    ![圖 #22][22]
2. 按一下網頁左下角的 [新增]，接著選取 [Python 2]。 然後，提供 Notebook 的名稱，並按一下核取記號以建立新的空白 IPython Notebook。
   
    ![圖 #23][23]
3. 按一下新的 IPython Notebook 左上角的 [Jupyter] 符號。
   
    ![圖 #24][24]
4. 將範例 IPython Notebook 拖放到 AzureML IPython Notebook 服務的 [樹狀結構] 頁面，然後按一下 [上傳]。 然後，範例 IPython Notebook 就會上傳到 AzureML IPython Notebook 服務。
   
    ![圖 #25][25]

若要執行範例 IPython Notebook 或 Python 指令碼檔案，您需要下列 Python 封裝。 如果您使用 AzureML IPython Notebook 服務，則已預先安裝這些封裝。

    - pandas
    - numpy
    - matplotlib
    - pyodbc
    - PyTables

以下是在 AzureML 中使用大型資料建置進階分析解決方案時的建議順序：

* 將小型資料取樣讀取至記憶體中的資料框架。
* 使用取樣的資料來執行一些視覺化操作和探索。
* 使用取樣的資料來試驗功能工程。
* 如為較大型的資料探索、資料操作及功能工程，請使用 Pythont，針對 SQL DW 直接發出 SQL 查詢。
* 決定適合用於 Azure Machine Learning 模型建置的取樣大小。

以下是數個資料探索、資料視覺化及功能工程範例。 您可以在範例 IPython Notebook 和範例 Python 指令碼檔案中找到更多資料探索。

### <a name="initialize-database-credentials"></a>初始化資料庫認證
使用下列變數來初始化資料庫連接設定：

    SERVER_NAME=<server name>
    DATABASE_NAME=<database name>
    USERID=<user name>
    PASSWORD=<password>
    DB_DRIVER = <database driver>

### <a name="create-database-connection"></a>建立資料庫連接
以下是建立資料庫連接的連接字串。

    CONNECTION_STRING = 'DRIVER={'+DRIVER+'};SERVER='+SERVER_NAME+';DATABASE='+DATABASE_NAME+';UID='+USERID+';PWD='+PASSWORD
    conn = pyodbc.connect(CONNECTION_STRING)

### <a name="report-number-of-rows-and-columns-in-table-nyctaxitrip"></a>報告資料表 <nyctaxi_trip> 中資料列和資料行的數目
    nrows = pd.read_sql('''
        SELECT SUM(rows) FROM sys.partitions
        WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_trip>')
    ''', conn)

    print 'Total number of rows = %d' % nrows.iloc[0,0]

    ncols = pd.read_sql('''
        SELECT COUNT(*) FROM information_schema.columns
        WHERE table_name = ('<nyctaxi_trip>') AND table_schema = ('<schemaname>')
    ''', conn)

    print 'Total number of columns = %d' % ncols.iloc[0,0]

* 資料列總數 = 173179759  
* 資料行總數 = 14

### <a name="report-number-of-rows-and-columns-in-table-nyctaxifare"></a>報告資料表 <nyctaxi_fare> 中資料列和資料行的數目
    nrows = pd.read_sql('''
        SELECT SUM(rows) FROM sys.partitions
        WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_fare>')
    ''', conn)

    print 'Total number of rows = %d' % nrows.iloc[0,0]

    ncols = pd.read_sql('''
        SELECT COUNT(*) FROM information_schema.columns
        WHERE table_name = ('<nyctaxi_fare>') AND table_schema = ('<schemaname>')
    ''', conn)

    print 'Total number of columns = %d' % ncols.iloc[0,0]

* 資料列總數 = 173179759  
* 資料行總數 = 11

### <a name="read-in-a-small-data-sample-from-the-sql-data-warehouse-database"></a>從 SQL 資料倉儲資料庫讀入小型資料取樣
    t0 = time.time()

    query = '''
        SELECT TOP 10000 t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax,
            f.tolls_amount, f.total_amount, f.tip_amount
        FROM <schemaname>.<nyctaxi_trip> t, <schemaname>.<nyctaxi_fare> f
        WHERE datepart("mi",t.pickup_datetime) = 1
        AND   t.medallion = f.medallion
        AND   t.hack_license = f.hack_license
        AND   t.pickup_datetime = f.pickup_datetime
    '''

    df1 = pd.read_sql(query, conn)

    t1 = time.time()
    print 'Time to read the sample table is %f seconds' % (t1-t0)

    print 'Number of rows and columns retrieved = (%d, %d)' % (df1.shape[0], df1.shape[1])

讀取取樣資料表的時間為 14.096495 秒。  
擷取的資料列和資料行數目 = (1000, 21)。

### <a name="descriptive-statistics"></a>描述性統計資料
現在您已做好準備可以探索取樣的資料。 一開始我們會先查看 **trip\_distance** (或您選擇指定的任何其他欄位) 的一些描述性統計資料。

    df1['trip_distance'].describe()

### <a name="visualization-box-plot-example"></a>視覺效果：盒狀圖範例
接下來將查看車程距離的盒狀圖，以視覺化方式檢視分位數。

    df1.boxplot(column='trip_distance',return_type='dict')

![圖 #1][1]

### <a name="visualization-distribution-plot-example"></a>視覺效果：分佈的盒狀圖範例
能以視覺化方式顯示取樣車程距離之分佈和長條圖的繪圖。

    fig = plt.figure()
    ax1 = fig.add_subplot(1,2,1)
    ax2 = fig.add_subplot(1,2,2)
    df1['trip_distance'].plot(ax=ax1,kind='kde', style='b-')
    df1['trip_distance'].hist(ax=ax2, bins=100, color='k')

![圖 #2][2]

### <a name="visualization-bar-and-line-plots"></a>視覺效果：長條圖和折線圖
在此範例中，我們可以將車程距離分類收納為五個分類收納組，並將分類收納結果視覺化。

    trip_dist_bins = [0, 1, 2, 4, 10, 1000]
    df1['trip_distance']
    trip_dist_bin_id = pd.cut(df1['trip_distance'], trip_dist_bins)
    trip_dist_bin_id

我們可以用下列命令在長條圖或折線圖中繪製上述分類收納組的分佈：

    pd.Series(trip_dist_bin_id).value_counts().plot(kind='bar')

![圖 #3][3]

和

    pd.Series(trip_dist_bin_id).value_counts().plot(kind='line')

![圖 #4][4]

### <a name="visualization-scatterplot-examples"></a>視覺效果：散佈圖範例
這會顯示 **trip\_time\_in\_secs** 與 **trip\_distance** 之間的散佈圖，供您查看當中是否有任何關聯性

    plt.scatter(df1['trip_time_in_secs'], df1['trip_distance'])

![圖 #6][6]

也可以同樣的方式查看 **rate\_code** 與 **trip\_distance** 之間的關聯性。

    plt.scatter(df1['passenger_count'], df1['trip_distance'])

![圖 #8][8]

### <a name="data-exploration-on-sampled-data-using-sql-queries-in-ipython-notebook"></a>在 IPython Notebook 中使用 SQL 查詢對取樣資料進行資料探索
在本節中，我們將使用前面所建立的新資料表中保存的取樣資料，來探索資料分佈。 請注意，您也可以使用原始資料表執行類似探索。

#### <a name="exploration-report-number-of-rows-and-columns-in-the-sampled-table"></a>探索：報告取樣資料表中資料列和資料行的數目
    nrows = pd.read_sql('''SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_sample>')''', conn)
    print 'Number of rows in sample = %d' % nrows.iloc[0,0]

    ncols = pd.read_sql('''SELECT count(*) FROM information_schema.columns WHERE table_name = ('<nyctaxi_sample>') AND table_schema = '<schemaname>'''', conn)
    print 'Number of columns in sample = %d' % ncols.iloc[0,0]

#### <a name="exploration-tippednot-tripped-distribution"></a>探索：已支付小費/未支付小費的分佈
    query = '''
        SELECT tipped, count(*) AS tip_freq
        FROM <schemaname>.<nyctaxi_sample>
        GROUP BY tipped
        '''

    pd.read_sql(query, conn)

#### <a name="exploration-tip-class-distribution"></a>探索：小費類別分佈
    query = '''
        SELECT tip_class, count(*) AS tip_freq
        FROM <schemaname>.<nyctaxi_sample>
        GROUP BY tip_class
    '''

    tip_class_dist = pd.read_sql(query, conn)

#### <a name="exploration-plot-the-tip-distribution-by-class"></a>探索：依類別繪製小費分佈
    tip_class_dist['tip_freq'].plot(kind='bar')

![圖 #26][26]

#### <a name="exploration-daily-distribution-of-trips"></a>探索：車程的每日分佈
    query = '''
        SELECT CONVERT(date, dropoff_datetime) AS date, COUNT(*) AS c
        FROM <schemaname>.<nyctaxi_sample>
        GROUP BY CONVERT(date, dropoff_datetime)
    '''

    pd.read_sql(query,conn)

#### <a name="exploration-trip-distribution-per-medallion"></a>探索：根據 medallion 的車程分佈
    query = '''
        SELECT medallion,count(*) AS c
        FROM <schemaname>.<nyctaxi_sample>
        GROUP BY medallion
    '''

    pd.read_sql(query,conn)

#### <a name="exploration-trip-distribution-by-medallion-and-hack-license"></a>探索：依據計程車牌照和計程車駕照的車程分佈
    query = '''select medallion, hack_license,count(*) from <schemaname>.<nyctaxi_sample> group by medallion, hack_license'''
    pd.read_sql(query,conn)


#### <a name="exploration-trip-time-distribution"></a>探索：車程時間分佈
    query = '''select trip_time_in_secs, count(*) from <schemaname>.<nyctaxi_sample> group by trip_time_in_secs order by count(*) desc'''
    pd.read_sql(query,conn)

#### <a name="exploration-trip-distance-distribution"></a>探索：車程距離分佈
    query = '''select floor(trip_distance/5)*5 as tripbin, count(*) from <schemaname>.<nyctaxi_sample> group by floor(trip_distance/5)*5 order by count(*) desc'''
    pd.read_sql(query,conn)

#### <a name="exploration-payment-type-distribution"></a>探索：付款類型分佈
    query = '''select payment_type,count(*) from <schemaname>.<nyctaxi_sample> group by payment_type'''
    pd.read_sql(query,conn)

#### <a name="verify-the-final-form-of-the-featurized-table"></a>驗證功能化表格的最後形式
    query = '''SELECT TOP 100 * FROM <schemaname>.<nyctaxi_sample>'''
    pd.read_sql(query,conn)

## <a name="mlmodel"></a>在 Azure Machine Learning 中建置模型
我們現在已準備好在 [Azure Machine Learning](https://studio.azureml.net)中建置和部署模型。 資料已經準備好用於稍早所識別的任何預測問題，也就是：

1. **二元分類**：預測是否已支付某趟車程的小費。
2. **多元分類**：根據先前定義的類別，預測所支付的小費範圍。
3. **迴歸工作**：預測針對某趟車程支付的小費金額。  

若要開始進行模型化練習，請登入 **Azure Machine Learning** 工作區。 如果您尚未建立機器學習服務工作區，請參閱「 [建立 Azure ML 工作區](../studio/create-workspace.md)」。

1. 若要開始使用 Azure Machine Learning，請參閱「 [什麼是 Azure Machine Learning Studio？](../studio/what-is-ml-studio.md)
2. 登入 [Azure Machine Learning Studio](https://studio.azureml.net)。
3. Studio 首頁會提供豐富的資訊、影片、教學課程、與模組參考的連結，以及其他資源。 如需 Azure Machine Learning 的詳細資訊，請參閱 [Azure 機器學習服務文件中心](https://azure.microsoft.com/documentation/services/machine-learning/)。

典型的訓練實驗包含下列步驟：

1. 建立 **+NEW** 實驗。
2. 將資料匯入 Azure ML。
3. 視需要前置處理、轉換和操作資料。
4. 視需要產生功能。
5. 將資料分割為訓練/驗證/測試資料集 (或讓每一個擁有個別的資料集)。
6. 根據要解決的學習問題，選取一或多個機器學習服務演算法。 例如，二進位分類、多類別分類、迴歸。
7. 使用訓練資料集來訓練一或多個模型。
8. 使用訓練的模型，為驗證資料集計分。
9. 評估模型來計算適用於學習問題的相關度量。
10. 微調模型，並選取要部署的最佳模型。

在這個練習中，我們已經探索了 SQL 資料倉儲中的資料並進行處理，並且決定了要在 Azure ML 中擷取的取樣大小。 以下是建置一或多個預測模型的程序：

1. 使用[匯入資料][import-data]模組 (可從**資料輸入和輸出**區段取得)，將資料匯入 Azure ML。 如需詳細資訊，請參閱[匯入資料][import-data]模組參考頁面。
   
    ![Azure ML 匯入資料][17]
2. 在 [屬性] 面板中，選取 [Azure SQL Database] 做為 [資料來源]。
3. 在 [ **資料庫伺服器名稱** ] 欄位中輸入資料庫的 DNS 名稱。 格式： `tcp:<your_virtual_machine_DNS_name>,1433`
4. 在對應欄位中輸入 **資料庫名稱** 。
5. 在 [伺服器使用者帳戶名稱] 中輸入「SQL 使用者名稱」，並在 [伺服器使用者帳戶密碼] 中輸入「密碼」。
7. 在 [ **資料庫查詢** ] 中編輯文字區域、貼上可擷取必要資料庫欄位的查詢 (包括任何經過計算的欄位，例如標籤)，以及向下取樣所需大小的資料。

下圖是直接從 SQL 資料倉儲資料庫讀取資料的二元分類實驗範例 (請記得用您在逐步解說中所使用的結構描述名稱和資料表名稱來取代資料表名稱 nyctaxi_trip 和 nyctaxi_fare)。 您可以針對多類別分類和迴歸問題建構類似的實驗。

![Azure ML 訓練][10]

> [!IMPORTANT]
> 在前幾節中提供的模型化資料擷取和取樣查詢範例中， **這三個模型化練習的所有標籤都包含於此查詢中**。 每一個模型化練習的重要 (必要) 步驟都是針對其他兩個問題**排除**不需要的標籤，以及任何其他的**目標流失**。 例如，使用二進位分類時，請用 **tipped** 標籤，並排除 **tip\_class**、**tip\_amount** 和 **total\_amount** 欄位。 後者為目標流失，因為它們意指支付的小費。
> 
> 若要排除任何不必要的資料行或目標流失，您可以使用[選取資料集中的資料行][select-columns]模組或[編輯中繼資料][edit-metadata]。 如需詳細資訊，請參閱[選取資料集中的資料行][select-columns]和[編輯中繼資料][edit-metadata]參考頁面。
> 
> 

## <a name="mldeploy"></a>在 Azure Machine Learning 中部署模型
當您備妥模型時，可以輕鬆地直接從實驗中將它部署為 Web 服務。 如需關於部署 Azure ML Web 服務的詳細資訊，請參閱 [部署 Azure 機器學習 Web 服務](../studio/publish-a-machine-learning-web-service.md)。

若要部署新的 Web 服務，您需要：

1. 建立計分實驗。
2. 部署 Web 服務。

若要從「已完成」的訓練實驗建立評分實驗，請按一下下方動作列中的 [建立評分實驗]。

![Azure 評分][18]

Azure Machine Learning 將根據訓練實驗的元件來建立計分實驗。 特別是，它將：

1. 儲存訓練的模型，並移除模型訓練模組。
2. 識別邏輯 **輸入連接埠** ，表示預期的輸入資料結構描述。
3. 識別邏輯 **輸出連接埠** ，表示預期的 Web 服務輸出結構描述。

建立計分實驗時，請檢閱它並視需要進行調整。 典型的調整是使用某一個會排除標籤欄位的輸入資料集和 (或) 查詢來取代它們，因為在呼叫服務時將無法使用這些欄位。 若要將輸入資料集和 (或) 查詢的大小縮減為只有幾筆足以表示輸入結構描述的記錄，這也是個很好的練習。 針對輸出連接埠，通常會使用[選取資料集中的資料行][select-columns]模組，在輸出中排除所有輸入欄位，只包含 [評分標籤]和 [評分機率]。

下圖提供評分實驗範例。 準備部署時，請按下方動作列中的 [發佈 Web 服務]  按鈕。

![Azure ML 發佈][11]

## <a name="summary"></a>總結
讓我們回顧一下已在此逐步解說教學課程中完成的工作，您已經建立 Azure 資料科學環境、使用大型公用資料集，並在 Team Data Science Process 的整個過程中使用它，而您在這個過程中擷取資料、進行模型定型，然後部署 Azure Machine Learning Web 服務。

### <a name="license-information"></a>授權資訊
此逐步解說範例及其隨附的指令碼和 IPython Notebook 是在 MIT 授權下由 Microsoft 所共用。 如需詳細資料，請查看 GitHub 上範例程式碼目錄中的 LICENSE.txt 檔案。

## <a name="references"></a>參考
•    [Andrés Monroy NYC 計程車車程下載頁面](http://www.andresmh.com/nyctaxitrips/) \(英文\)  
•    [FOILing NYC 的計程車車程資料 (作者為 Chris Whong)](http://chriswhong.com/open-data/foil_nyc_taxi/)   
•    [NYC 計程車和禮車委託研究和統計資料](http://www.nyc.gov/html/tlc/html/technology/aggregated_data.shtml)

[1]: ./media/sqldw-walkthrough/sql-walkthrough_26_1.png
[2]: ./media/sqldw-walkthrough/sql-walkthrough_28_1.png
[3]: ./media/sqldw-walkthrough/sql-walkthrough_35_1.png
[4]: ./media/sqldw-walkthrough/sql-walkthrough_36_1.png
[5]: ./media/sqldw-walkthrough/sql-walkthrough_39_1.png
[6]: ./media/sqldw-walkthrough/sql-walkthrough_42_1.png
[7]: ./media/sqldw-walkthrough/sql-walkthrough_44_1.png
[8]: ./media/sqldw-walkthrough/sql-walkthrough_46_1.png
[9]: ./media/sqldw-walkthrough/sql-walkthrough_71_1.png
[10]: ./media/sqldw-walkthrough/azuremltrain.png
[11]: ./media/sqldw-walkthrough/azuremlpublish.png
[12]: ./media/sqldw-walkthrough/ssmsconnect.png
[13]: ./media/sqldw-walkthrough/executescript.png
[14]: ./media/sqldw-walkthrough/sqlserverproperties.png
[15]: ./media/sqldw-walkthrough/sqldefaultdirs.png
[16]: ./media/sqldw-walkthrough/bulkimport.png
[17]: ./media/sqldw-walkthrough/amlreader.png
[18]: ./media/sqldw-walkthrough/amlscoring.png
[19]: ./media/sqldw-walkthrough/ps_download_scripts.png
[20]: ./media/sqldw-walkthrough/ps_load_data.png
[21]: ./media/sqldw-walkthrough/azcopy-overwrite.png
[22]: ./media/sqldw-walkthrough/ipnb-service-aml-1.png
[23]: ./media/sqldw-walkthrough/ipnb-service-aml-2.png
[24]: ./media/sqldw-walkthrough/ipnb-service-aml-3.png
[25]: ./media/sqldw-walkthrough/ipnb-service-aml-4.png
[26]: ./media/sqldw-walkthrough/tip_class_hist_1.png


<!-- Module References -->
[edit-metadata]: https://msdn.microsoft.com/library/azure/370b6676-c11c-486f-bf73-35349f842a66/
[select-columns]: https://msdn.microsoft.com/library/azure/1ec722fa-b623-4e26-a44e-a50c6d726223/
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
