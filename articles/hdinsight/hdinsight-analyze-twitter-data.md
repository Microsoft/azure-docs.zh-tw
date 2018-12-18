---
title: 在 HDInsight 中使用 Hadoop 分析 Twitter 資料 - Azure
description: 了解如何在 HDInsight 中的 Hadoop 上使用 Hive 來分析 Twitter 資料，以找出特定單字的使用頻率。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/25/2017
ms.author: jasonh
ROBOTS: NOINDEX
ms.openlocfilehash: 294353cfcfba617ab19e703f11f35402bcf7ea82
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387575"
---
# <a name="analyze-twitter-data-using-hive-in-hdinsight"></a>在 HDInsight 中使用 Hive 分析 Twitter 資料
社群網站是驅使採用巨量資料的其中一個主要動力。 像 Twitter 之類的網站所提供的公開 API，是分析和了解流行趨勢的一項實用的資料來源。
在本教學課程中，您將使用 Twitter 串流 API 取得推文，然後使用 Apache Hive on Azure HDInsight 取得傳送了最多內含特定文字之推文的 Twitter 使用者清單。

> [!IMPORTANT]
> 此文件中的步驟需要 Windows 型 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。 如需 Linux 型叢集的特定步驟，請參閱 [在 HDInsight (Linux) 中使用 Hive 分析 Twitter 資料](hdinsight-analyze-twitter-data-linux.md)。

## <a name="prerequisites"></a>必要條件
開始進行本教學課程之前，您必須具備下列條件：

* **工作站** 。

    若要執行 Windows PowerShell 指令碼，您必須以系統管理員的身分執行 Azure PowerShell，並將執行原則設為 *RemoteSigned*。 請參閱[執行 Windows PowerShell 指令碼][powershell-script]。

    執行 Windows PowerShell 指令碼之前，請確定您已使用下列 Cmdlet 連接到 Azure 訂用帳戶：

    ```powershell
    Connect-AzureRmAccount
    ```

    如果您有多個 Azure 訂閱，請使用下列 Cmdlet 設定目前的訂閱：

    ```powershell
    Select-AzureRmSubscription -SubscriptionID <Azure Subscription ID>
    ```

    > [!IMPORTANT]
    > 使用 Azure Service Manager 管理 HDInsight 資源的 Azure PowerShell 支援已**被取代**，並已在 2017 年 1 月 1 日移除。 本文件中的步驟會使用可與 Azure Resource Manager 搭配使用的新 HDInsight Cmdlet。
    >
    > 請遵循 [安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs) 中的步驟來安裝最新版的 Azure PowerShell。 如果您需要修改指令碼才能使用適用於 Azure Resource Manager 的新 Cmdlet，請參閱 [移轉至以 Azure Resource Manager 為基礎的開發工具 (適用於 HDInsight 叢集)](hdinsight-hadoop-development-using-azure-resource-manager.md) ，以取得詳細資訊。

* **Azure HDInsight 叢集**。 如需叢集佈建的指示，請參閱[開始使用 HDInsight][hdinsight-get-started] 或[佈建 HDInsight 叢集][hdinsight-provision]。 稍後在教學課程中會用到此叢集名稱。

下表列出本教學課程中使用的檔案：

| 檔案 | 說明 |
| --- | --- |
| /tutorials/twitter/data/tweets.txt |Hive 工作的來源資料。 |
| /tutorials/twitter/output |Hive 工作的輸出資料夾。 預設 Hive 工作的輸出檔案名稱為 **000000_0**。 |
| tutorials/twitter/twitter.hql |HiveQL 指令碼檔案。 |
| /tutorials/twitter/jobstatus |Hadoop 工作狀態。 |

## <a name="get-twitter-feed"></a>取得 Twitter 摘要
在本教學課程中，您將使用 [Twitter 串流 API][twitter-streaming-api]。 您將使用的特定 Twitter 串流 API 為 [statuses/filter][twitter-statuses-filter]。

> [!NOTE]
> 已在公用 Blob 容器中上傳含有 10,000 則推文的檔案和 Hive 指令碼檔案 (下一節說明)。 如果想要使用上傳的檔案，可以略過這一節。

推文資料會以包含複雜巢狀結構的 JavaScript 物件標記法 (JSON) 格式儲存。 您可以不要使用慣用的程式設計語言撰寫多行程式碼，而將此巢狀結構轉換成 Hive 資料表，以利用 HiveQL 這種類似結構化查詢語言 (SQL) 的語言來查詢資料表。

Twitter 會使用 OAuth 提供對其 API 的授權存取。 OAuth 是一項驗證通訊協定，可讓使用者在無須共用其密碼的情況下，允許應用程式代表他們執行動作。 如需詳細資訊，請至 [oauth.net](http://oauth.net/)，或參考 Hueniverse 的 [OAuth 的入門指南](http://hueniverse.com/oauth/) (英文)。

使用 OAuth 的第一個步驟，是在 Twitter 開發人員網站上建立新的應用程式。

**建立 Twitter 應用程式**

1. 登入 [https://apps.twitter.com/](https://apps.twitter.com/)。 如果您沒有 Twitter 帳戶，請按一下[立即註冊]  連結。
2. 按一下 [建立新的應用程式] 。
3. 輸入 [名稱]、[說明]、[網站]。 您可以在 [網站] 欄位中自行設定 URL。 下表列出部分要使用的範例值：

   | 欄位 | 值 |
   | --- | --- |
   |  名稱 |MyHDInsightApp |
   |  說明 |MyHDInsightApp |
   |  網站 |http://www.myhdinsightapp.com |
4. 核取 [是，我同意] 然後按一下 [建立 Twitter 應用程式]。
5. 按一下 [權限]  索引標籤。預設權限為 [唯讀] 。 本教學課程使用預設值即可。
6. 按一下 **[金鑰和存取權杖** ] 索引標籤。
7. 按一下 [Create my access token] 。
8. 按一下位於頁面右上角的 [測試 OAuth]  。
9. 記下**消費者金鑰**、**消費者祕密**、**存取權杖**和**存取權杖祕密**。 稍後在教學課程中會用到這些值。

在本教學課程中，您將使用 Windows PowerShell 發出 Web 服務呼叫。 另一項常用來發出 Web 服務呼叫的工具是 [*Curl*][curl]。 您可以從[這裡][curl-download]下載 Curl。

> [!NOTE]
> 在 Windows 上使用 curl 命令時，對選項值請使用雙引號，而不要使用單引號。

**取得推文**

1. 開啟 Windows PowerShell 整合式指令碼環境 (ISE)。 (在 Windows 8 的 [開始] 畫面上輸入 **PowerShell_ISE**，然後按一下 [Windows PowerShell ISE]。 請參閱[在 Windows 8 和 Windows 上啟動 Windows PowerShell](https://docs.microsoft.com/powershell/scripting/setup/starting-windows-powershell?view=powershell-6)
2. 將下列指令碼複製到指令碼窗格中：

    ```powershell
    #region - variables and constants
    $clusterName = "<HDInsightClusterName>" # Enter the HDInsight cluster name

    # Enter the OAuth information for your Twitter application
    $oauth_consumer_key = "<TwitterAppConsumerKey>";
    $oauth_consumer_secret = "<TwitterAppConsumerSecret>";
    $oauth_token = "<TwitterAppAccessToken>";
    $oauth_token_secret = "<TwitterAppAccessTokenSecret>";

    $destBlobName = "tutorials/twitter/data/tweets.txt" # This script saves the tweets into this blob.

    $trackString = "Azure, Cloud, HDInsight" # This script gets the tweets containing these keywords.
    $track = [System.Uri]::EscapeDataString($trackString);
    $lineMax = 10000  # The script will get this number of tweets. It is about 3 minutes every 100 lines.
    #endregion

    #region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
    Connect-AzureRmAccount
    #endregion

    #region - Create a block blob object for writing tweets into Blob storage
    Write-Host "Get the default storage account name and Blob container name using the cluster name ..." -ForegroundColor Green
    $myCluster = Get-AzureRmHDInsightCluster -Name $clusterName
    $resourceGroupName = $myCluster.ResourceGroup
    $storageAccountName = $myCluster.DefaultStorageAccount.Replace(".blob.core.windows.net", "")
    $containerName = $myCluster.DefaultStorageContainer
    Write-Host "`tThe storage account name is $storageAccountName." -ForegroundColor Yellow
    Write-Host "`tThe blob container name is $containerName." -ForegroundColor Yellow

    Write-Host "Define the Azure storage connection string ..." -ForegroundColor Green
    $storageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName)[0].Value
    $storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=$storageAccountName;AccountKey=$storageAccountKey"
    Write-Host "`tThe connection string is $storageConnectionString." -ForegroundColor Yellow

    Write-Host "Create block blob object ..." -ForegroundColor Green
    $storageAccount = [Microsoft.WindowsAzure.Storage.CloudStorageAccount]::Parse($storageConnectionString)
    $storageClient = $storageAccount.CreateCloudBlobClient();
    $storageContainer = $storageClient.GetContainerReference($containerName)
    $destBlob = $storageContainer.GetBlockBlobReference($destBlobName)
    #end region

    # region - Format OAuth strings
    Write-Host "Format oauth strings ..." -ForegroundColor Green
    $oauth_nonce = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes([System.DateTime]::Now.Ticks.ToString()));
    $ts = [System.DateTime]::UtcNow - [System.DateTime]::ParseExact("01/01/1970", "dd/MM/yyyy", $null)
    $oauth_timestamp = [System.Convert]::ToInt64($ts.TotalSeconds).ToString();

    $signature = "POST&";
    $signature += [System.Uri]::EscapeDataString("https://stream.twitter.com/1.1/statuses/filter.json") + "&";
    $signature += [System.Uri]::EscapeDataString("oauth_consumer_key=" + $oauth_consumer_key + "&");
    $signature += [System.Uri]::EscapeDataString("oauth_nonce=" + $oauth_nonce + "&");
    $signature += [System.Uri]::EscapeDataString("oauth_signature_method=HMAC-SHA1&");
    $signature += [System.Uri]::EscapeDataString("oauth_timestamp=" + $oauth_timestamp + "&");
    $signature += [System.Uri]::EscapeDataString("oauth_token=" + $oauth_token + "&");
    $signature += [System.Uri]::EscapeDataString("oauth_version=1.0&");
    $signature += [System.Uri]::EscapeDataString("track=" + $track);

    $signature_key = [System.Uri]::EscapeDataString($oauth_consumer_secret) + "&" + [System.Uri]::EscapeDataString($oauth_token_secret);

    $hmacsha1 = new-object System.Security.Cryptography.HMACSHA1;
    $hmacsha1.Key = [System.Text.Encoding]::ASCII.GetBytes($signature_key);
    $oauth_signature = [System.Convert]::ToBase64String($hmacsha1.ComputeHash([System.Text.Encoding]::ASCII.GetBytes($signature)));

    $oauth_authorization = 'OAuth ';
    $oauth_authorization += 'oauth_consumer_key="' + [System.Uri]::EscapeDataString($oauth_consumer_key) + '",';
    $oauth_authorization += 'oauth_nonce="' + [System.Uri]::EscapeDataString($oauth_nonce) + '",';
    $oauth_authorization += 'oauth_signature="' + [System.Uri]::EscapeDataString($oauth_signature) + '",';
    $oauth_authorization += 'oauth_signature_method="HMAC-SHA1",'
    $oauth_authorization += 'oauth_timestamp="' + [System.Uri]::EscapeDataString($oauth_timestamp) + '",'
    $oauth_authorization += 'oauth_token="' + [System.Uri]::EscapeDataString($oauth_token) + '",';
    $oauth_authorization += 'oauth_version="1.0"';

    $post_body = [System.Text.Encoding]::ASCII.GetBytes("track=" + $track);
    #endregion

    #region - Read tweets
    Write-Host "Create HTTP web request ..." -ForegroundColor Green
    [System.Net.HttpWebRequest] $request = [System.Net.WebRequest]::Create("https://stream.twitter.com/1.1/statuses/filter.json");
    $request.Method = "POST";
    $request.Headers.Add("Authorization", $oauth_authorization);
    $request.ContentType = "application/x-www-form-urlencoded";
    $body = $request.GetRequestStream();

    $body.write($post_body, 0, $post_body.length);
    $body.flush();
    $body.close();
    $response = $request.GetResponse() ;

    Write-Host "Start stream reading ..." -ForegroundColor Green

    Write-Host "Define a MemoryStream and a StreamWriter for writing ..." -ForegroundColor Green
    $memStream = New-Object System.IO.MemoryStream
    $writeStream = New-Object System.IO.StreamWriter $memStream

    $sReader = New-Object System.IO.StreamReader($response.GetResponseStream())

    $inrec = $sReader.ReadLine()
    $count = 0
    while (($inrec -ne $null) -and ($count -le $lineMax))
    {
        if ($inrec -ne "")
        {
            Write-Host "`n`t $count tweets received." -ForegroundColor Yellow

            $writeStream.WriteLine($inrec)
            $count ++
        }

        $inrec=$sReader.ReadLine()
    }
    #endregion

    #region - Write tweets to Blob storage
    Write-Host "Write to the destination blob ..." -ForegroundColor Green
    $writeStream.Flush()
    $memStream.Seek(0, "Begin")
    $destBlob.UploadFromStream($memStream)

    $sReader.close()
    #endregion

    Write-Host "Completed!" -ForegroundColor Green
    ```

3. 設定指令碼中的前五到八個變數：

    變數|說明
    ---|---
    $clusterName|這是您要執行應用程式的 HDInsight 叢集名稱。
    $oauth_consumer_key|這是您先前在建立 Twitter 應用程式時所記下的 Twitter 應用程式 **消費者金鑰** 。
    $oauth_consumer_secret|這是您先前記下的 Twitter 應用程式 **消費者密碼** 。
    $oauth_token|這是您先前記下的 Twitter 應用程式 **存取權杖** 。
    $oauth_token_secret|這是您先前記下的 Twitter 應用程式 **存取權杖密碼** 。
    $destBlobName|這是輸出 Blob 名稱。 預設值為 **tutorials/twitter/data/tweets.txt**。 如果您變更預設值，則 Windows PowerShell 指令碼也必須隨之變更。
    $trackString|Web 服務會傳回這些關鍵字的相關推文。 預設值為 **Azure, Cloud, HDInsight**。 如果您變更預設值，則 Windows PowerShell 指令碼也要隨之變更。
    $lineMax|此值會決定指令碼所將讀取的推文數。 讀取 100 則推文大約需要三分鐘。 您可以設定更大的數目，但下載時間將會較久。

1. 按 **F5** 以執行指令碼。 如果發生問題，請選取所有程式碼行，然後按 **F8**，以解決問題。
2. 輸出的結尾應會顯示「完成！ 」。 錯誤訊息會顯示為紅色。

在驗證程序中，您可以使用 Azure 儲存體總管或 Azure PowerShell，在您的 Azure Blob 儲存體上查看輸出檔案 **/tutorials/twitter/data/tweets.txt**。 如需用於列出這些檔案的範例 Windows PowerShell 指令碼，請參閱[搭配 HDInsight 使用 Blob 儲存體][hdinsight-storage-powershell]。

## <a name="create-hiveql-script"></a>建立 HiveQL 指令碼
使用 Azure PowerShell 可讓您逐一執行多個 HiveQL 陳述式，或將 HiveQL 陳述式封裝到指令碼檔案中。 在本教學課程中，您將會建立 HiveQL 指令碼。 指令碼檔案必須上傳至 Azure Blob 儲存體。 在下一節中，您將使用 Azure PowerShell 執行指令碼檔案。

> [!NOTE]
> 已在公用 Blob 容器中上傳 Hive 指令碼檔案和含有 10,000 則推文的檔案。 如果想要使用上傳的檔案，可以略過這一節。

HiveQL 指令碼將執行下列作業：

1. **捨棄 tweets_raw 資料表** (若此資料表已存在)。
2. **建立 tweets_raw Hive 資料表**。 這個暫時的 Hive 結構化資料表會保存要進一步進行擷取、轉換和載入 (ETL) 處理的資料。 如需資料分割的相關資訊，請參閱 [Hive 教學課程][apache-hive-tutorial]。
3. **載入資料** 。 巢狀 JSON 格式的大型 Tweets 資料集現在已轉換成暫時的 Hive 資料表結構。
4. **捨棄 tweets 資料表** (若此資料表已存在)。
5. **建立 tweets 資料表**。 您必須先執行另一個 ETL 程序，才能使用 Hive 來查詢 Tweets 資料集。 此 ETL 程序針對您在 "twitter_raw" 資料表中儲存的資料，定義了更詳細的資料表結構描述。
6. **插入覆寫資料表**。 這個複雜的 Hive 指令碼會啟動一組冗長的 MapReduce 工作 (由 Hadoop 叢集執行)。 根據資料集和叢集大小而定，這可能需要 10 分鐘。
7. **插入覆寫目錄**。 執行查詢，並將資料集輸出至檔案。 此查詢會傳回一份 Twitter 使用者清單，這些使用者傳送了最多含有 "Azure" 一字的推文。

**建立 Hive 指令碼並將它上傳至 Azure**

1. 開啟 Windows PowerShell ISE。
2. 將下列指令碼複製到指令碼窗格中：

    ```powershell
    #region - variables and constants
    $clusterName = "<Existing HDInsight Cluster Name>" # Enter your HDInsight cluster name
    $subscriptionID = "<Azure Subscription ID>"

    $sourceDataPath = "/tutorials/twitter/data"
    $outputPath = "/tutorials/twitter/output"
    $hqlScriptFile = "tutorials/twitter/twitter.hql"

    $hqlStatements = @"
    set hive.exec.dynamic.partition = true;
    set hive.exec.dynamic.partition.mode = nonstrict;

    DROP TABLE tweets_raw;
    CREATE EXTERNAL TABLE tweets_raw (
        json_response STRING
    )
    STORED AS TEXTFILE LOCATION '$sourceDataPath';

    DROP TABLE tweets;
    CREATE TABLE tweets
    (
        id BIGINT,
        created_at STRING,
        created_at_date STRING,
        created_at_year STRING,
        created_at_month STRING,
        created_at_day STRING,
        created_at_time STRING,
        in_reply_to_user_id_str STRING,
        text STRING,
        contributors STRING,
        retweeted STRING,
        truncated STRING,
        coordinates STRING,
        source STRING,
        retweet_count INT,
        url STRING,
        hashtags array<STRING>,
        user_mentions array<STRING>,
        first_hashtag STRING,
        first_user_mention STRING,
        screen_name STRING,
        name STRING,
        followers_count INT,
        listed_count INT,
        friends_count INT,
        lang STRING,
        user_location STRING,
        time_zone STRING,
        profile_image_url STRING,
        json_response STRING
    );

    FROM tweets_raw
    INSERT OVERWRITE TABLE tweets
    SELECT
        cast(get_json_object(json_response, '$.id_str') as BIGINT),
        get_json_object(json_response, '$.created_at'),
        concat(substr (get_json_object(json_response, '$.created_at'),1,10),' ',
        substr (get_json_object(json_response, '$.created_at'),27,4)),
        substr (get_json_object(json_response, '$.created_at'),27,4),
        case substr (get_json_object(json_response, '$.created_at'),5,3)
            when "Jan" then "01"
            when "Feb" then "02"
            when "Mar" then "03"
            when "Apr" then "04"
            when "May" then "05"
            when "Jun" then "06"
            when "Jul" then "07"
            when "Aug" then "08"
            when "Sep" then "09"
            when "Oct" then "10"
            when "Nov" then "11"
            when "Dec" then "12" end,
        substr (get_json_object(json_response, '$.created_at'),9,2),
        substr (get_json_object(json_response, '$.created_at'),12,8),
        get_json_object(json_response, '$.in_reply_to_user_id_str'),
        get_json_object(json_response, '$.text'),
        get_json_object(json_response, '$.contributors'),
        get_json_object(json_response, '$.retweeted'),
        get_json_object(json_response, '$.truncated'),
        get_json_object(json_response, '$.coordinates'),
        get_json_object(json_response, '$.source'),
        cast (get_json_object(json_response, '$.retweet_count') as INT),
        get_json_object(json_response, '$.entities.display_url'),
        array(
            trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
            trim(lower(get_json_object(json_response, '$.entities.hashtags[1].text'))),
            trim(lower(get_json_object(json_response, '$.entities.hashtags[2].text'))),
            trim(lower(get_json_object(json_response, '$.entities.hashtags[3].text'))),
            trim(lower(get_json_object(json_response, '$.entities.hashtags[4].text')))),
        array(
            trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
            trim(lower(get_json_object(json_response, '$.entities.user_mentions[1].screen_name'))),
            trim(lower(get_json_object(json_response, '$.entities.user_mentions[2].screen_name'))),
            trim(lower(get_json_object(json_response, '$.entities.user_mentions[3].screen_name'))),
            trim(lower(get_json_object(json_response, '$.entities.user_mentions[4].screen_name')))),
        trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
        trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
        get_json_object(json_response, '$.user.screen_name'),
        get_json_object(json_response, '$.user.name'),
        cast (get_json_object(json_response, '$.user.followers_count') as INT),
        cast (get_json_object(json_response, '$.user.listed_count') as INT),
        cast (get_json_object(json_response, '$.user.friends_count') as INT),
        get_json_object(json_response, '$.user.lang'),
        get_json_object(json_response, '$.user.location'),
        get_json_object(json_response, '$.user.time_zone'),
        get_json_object(json_response, '$.user.profile_image_url'),
        json_response
    WHERE (length(json_response) > 500);

    INSERT OVERWRITE DIRECTORY '$outputPath'
    SELECT name, screen_name, count(1) as cc
        FROM tweets
        WHERE text like "%Azure%"
        GROUP BY name,screen_name
        ORDER BY cc DESC LIMIT 10;
    "@
    #endregion

    #region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green

    Try{
        Get-AzureRmSubscription
    }
    Catch{
        Connect-AzureRmAccount
    }

    Select-AzureRmSubscription -SubscriptionId $subscriptionID

    #endregion

    #region - Create a block blob object for writing the Hive script file
    Write-Host "Get the default storage account name and container name based on the cluster name ..." -ForegroundColor Green
    $myCluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroupName = $myCluster.ResourceGroup
    $defaultStorageAccountName = $myCluster.DefaultStorageAccount.Replace(".blob.core.windows.net", "")
    $defaultBlobContainerName = $myCluster.DefaultStorageContainer
    Write-Host "`tThe storage account name is $defaultStorageAccountName." -ForegroundColor Yellow
    Write-Host "`tThe blob container name is $defaultBlobContainerName." -ForegroundColor Yellow

    Write-Host "Define the connection string ..." -ForegroundColor Green
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value
    $storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=$defaultStorageAccountName;AccountKey=$defaultStorageAccountKey"

    Write-Host "Create block blob objects referencing the hql script file" -ForegroundColor Green
    $storageAccount = [Microsoft.WindowsAzure.Storage.CloudStorageAccount]::Parse($storageConnectionString)
    $storageClient = $storageAccount.CreateCloudBlobClient();
    $storageContainer = $storageClient.GetContainerReference($defaultBlobContainerName)
    $hqlScriptBlob = $storageContainer.GetBlockBlobReference($hqlScriptFile)

    Write-Host "Define a MemoryStream and a StreamWriter for writing ... " -ForegroundColor Green
    $memStream = New-Object System.IO.MemoryStream
    $writeStream = New-Object System.IO.StreamWriter $memStream
    $writeStream.Writeline($hqlStatements)
    #endregion

    #region - Write the Hive script file to Blob storage
    Write-Host "Write to the destination blob ... " -ForegroundColor Green
    $writeStream.Flush()
    $memStream.Seek(0, "Begin")
    $hqlScriptBlob.UploadFromStream($memStream)
    #endregion

    Write-Host "Completed!" -ForegroundColor Green
    ```

3. 設定指令碼中的前兩個變數：

   | 變數 | 說明 |
   | --- | --- |
   |  $clusterName |# 提供您要在其中執行 Hive 工作的 HDInsight 叢集名稱 |
   |  $subscriptionID |輸入您的 Azure 訂用帳戶 ID。 |
   |  $sourceDataPath |Hive 查詢將從中讀取資料的 Azure Blob 儲存體位置。 您無須變更此變數。 |
   |  $outputPath |Hive 查詢將輸出結果的 Azure Blob 儲存體位置。 您無須變更此變數。 |
   |  $hqlScriptFile |HiveQL 指令碼檔案的位置和檔案名稱。 您無須變更此變數。 |
4. 按 **F5** 以執行指令碼。 如果發生問題，請選取所有程式碼行，然後按 **F8**，以解決問題。
5. 輸出的結尾應會顯示「完成！ 」。 錯誤訊息會顯示為紅色。

在驗證程序中，您可以使用 Azure 儲存體總管或 Azure PowerShell，在您的 Azure Blob 儲存體上查看輸出檔案 **/tutorials/twitter/twitter.hql**。 如需用於列出這些檔案的範例 Windows PowerShell 指令碼，請參閱[搭配 HDInsight 使用 Blob 儲存體][hdinsight-storage-powershell]。

## <a name="process-twitter-data-by-using-hive"></a>使用 Hive 處理 Twitter 資料
您已完成所有準備工作。 現在，您可以叫用 Hive 指令碼，並查看結果。

### <a name="submit-a-hive-job"></a>提交 Hive 工作
使用下列 Window PowerShell 指令碼可執行 Hive 指令碼。 您必須設定第一個變數。

> [!NOTE]
> 若要使用您在上兩節中上傳的推文和 HiveQL 指令碼，請將 $hqlScriptFile 設定為 "/tutorials/twitter/twitter.hql"。 若要使用已上傳至公用 Blob 的項目，請將 $hqlScriptFile 設定為 "wasb://twittertrend@hditutorialdata.blob.core.windows.net/twitter.hql"。

```powershell
#region variables and constants
$clusterName = "<Existing Azure HDInsight Cluster Name>"
$httpUserName = "admin"
$httpUserPassword = "<HDInsight Cluster HTTP User Password>"

#use one of the following
$hqlScriptFile = "wasb://twittertrend@hditutorialdata.blob.core.windows.net/twitter.hql"
$hqlScriptFile = "/tutorials/twitter/twitter.hql"

$statusFolder = "/tutorials/twitter/jobstatus"
#endregion

$myCluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
$resourceGroupName = $myCluster.ResourceGroup
$defaultStorageAccountName = $myCluster.DefaultStorageAccount.Replace(".blob.core.windows.net", "")
$defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value

$defaultBlobContainerName = $myCluster.DefaultStorageContainer

#region - Invoke Hive
Write-Host "Invoke Hive ... " -ForegroundColor Green

# Create the HDInsight cluster
$pw = ConvertTo-SecureString -String $httpUserPassword -AsPlainText -Force
$httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$pw)

Use-AzureRmHDInsightCluster -ResourceGroupName $resourceGroupName -ClusterName $clusterName -HttpCredential $httpCredential
$response = Invoke-AzureRmHDInsightHiveJob -DefaultStorageAccountName $defaultStorageAccountName -DefaultStorageAccountKey $defaultStorageAccountKey -DefaultContainer $defaultBlobContainerName -file $hqlScriptFile -StatusFolder $statusFolder #-OutVariable $outVariable

Write-Host "Display the standard error log ... " -ForegroundColor Green
$jobID = ($response | Select-String job_ | Select-Object -First 1) -replace �\s*$� -replace �.*\s�
Get-AzureRmHDInsightJobOutput -ClusterName $clusterName -JobId $jobID -DefaultContainer $defaultBlobContainerName -DefaultStorageAccountName $defaultStorageAccountName -DefaultStorageAccountKey $defaultStorageAccountKey -HttpCredential $httpCredential
#endregion
```

### <a name="check-the-results"></a>查看結果
使用下列 Windows PowerShell 指令碼可查看 Hive 工作輸出。 您必須設定前兩個變數。

```powershell
#region variables and constants
$clusterName = "<Existing Azure HDInsight Cluster Name>"

$blob = "tutorials/twitter/output/000000_0" # The name of the blob to be downloaded.
#endregion

#region - Create an Azure storage context object
Write-Host "Get the default storage account name and container name based on the cluster name ..." -ForegroundColor Green
$myCluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
$resourceGroupName = $myCluster.ResourceGroup
$defaultStorageAccountName = $myCluster.DefaultStorageAccount.Replace(".blob.core.windows.net", "")
$defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value
$defaultBlobContainerName = $myCluster.DefaultStorageContainer

Write-Host "`tThe storage account name is $defaultStorageAccountName." -ForegroundColor Yellow
Write-Host "`tThe blob container name is $defaultBlobContainerName." -ForegroundColor Yellow

Write-Host "Create a context object ... " -ForegroundColor Green
$storageContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey
#endregion

#region - Download blob and display blob
Write-Host "Download the blob ..." -ForegroundColor Green
cd $HOME
Get-AzureStorageBlobContent -Container $defaultBlobContainerName -Blob $blob -Context $storageContext -Force

Write-Host "Display the output ..." -ForegroundColor Green
Write-Host "==================================" -ForegroundColor Green
cat "./$blob"
Write-Host "==================================" -ForegroundColor Green
#end region
```

> [!NOTE]
> Hive 資料表會使用 \001 做為欄位分隔符號。 此分隔符號不會顯示在輸出中。

分析結果列示在 Azure Blob 儲存體之後，您可以將資料匯出至 Azure SQL Database/SQL Server，使用 Power Query 將資料匯出至 Excel，或使用 Hive ODBC 驅動程式將應用程式連接到資料。 如需詳細資訊，請參閱[搭配 HDInsight 使用 Sqoop][hdinsight-use-sqoop]、[使用 HDInsight 分析航班延誤資料][hdinsight-analyze-flight-delay-data]、[使用 Power Query 將 Excel 連接到 HDInsight][hdinsight-power-query] 和[使用 Microsoft Hive ODBC 驅動程式將 Excel 連接到 HDInsight][hdinsight-hive-odbc]。

## <a name="next-steps"></a>後續步驟
本教學課程中，我們看到如何將非結構化 JSON 資料集轉換成結構化 Hive 資料表，然後在 Azure 上使用 HDInsight 來查詢、探索和分析來自 Twitter 的資料。 若要深入了解，請參閱：

* [開始使用 HDInsight][hdinsight-get-started]
* [使用 HDInsight 分析航班延誤資料][hdinsight-analyze-flight-delay-data]
* [使用 Power Query 將 Excel 連接到 HDInsight][hdinsight-power-query]
* [使用 Microsoft Hive ODBC 驅動程式將 Excel 連接到 HDInsight][hdinsight-hive-odbc]
* [搭配 HDInsight 使用 Sqoop][hdinsight-use-sqoop]

[curl]: http://curl.haxx.se
[curl-download]: http://curl.haxx.se/download.html

[apache-hive-tutorial]: https://cwiki.apache.org/confluence/display/Hive/Tutorial

[twitter-streaming-api]: https://developer.twitter.com/en/docs/api-reference-index
[twitter-statuses-filter]: https://developer.twitter.com/en/docs/tweets/filter-realtime/api-reference/post-statuses-filter

[powershell-start]: http://technet.microsoft.com/library/hh847889.aspx
[powershell-install]: /powershell/azureps-cmdlets-docs
[powershell-script]: http://technet.microsoft.com/library/ee176961.aspx

[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-get-started]:hadoop/apache-hadoop-linux-tutorial-get-started.md
[hdinsight-storage-powershell]:hdinsight-hadoop-use-blob-storage.md#access-blobs-using-azure-powershell
[hdinsight-analyze-flight-delay-data]: hdinsight-analyze-flight-delay-data.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-use-sqoop]:hadoop/hdinsight-use-sqoop.md
[hdinsight-power-query]:hadoop/apache-hadoop-connect-excel-power-query.md
[hdinsight-hive-odbc]:hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md
