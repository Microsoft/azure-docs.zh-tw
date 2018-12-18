---
title: 使用開放原始碼工具將 Azure 網路監看員 NSG 流量記錄視覺化 | Microsoft Docs
description: 此頁面說明如何使用開放原始碼工具將 NSG 流量記錄視覺化。
services: network-watcher
documentationcenter: na
author: mattreatMSFT
manager: vitinnan
editor: ''
ms.assetid: e9b2dcad-4da4-4d6b-aee2-6d0afade0cb8
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: mareat
ms.openlocfilehash: aa83ba1f428e70cd78cba2af6d39989179d5b30f
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42145762"
---
# <a name="visualize-azure-network-watcher-nsg-flow-logs-using-open-source-tools"></a>使用開放原始碼工具將 Azure 網路監看員 NSG 流量記錄視覺化

網路安全性群組流量記錄提供的資訊可用於了解網路安全性群組上的輸入和輸出 IP 流量。 這些流量記錄顯示每一個規則的輸出和輸入流量、套用流量的 NIC、有關流量的 5 Tuple 資訊 (來源/目的地 IP、來源/目的地連接埠、通訊協定)，以及允許或拒絕流量。

這些流量記錄難以手動剖析和獲得見解。 不過，有幾種開放原始碼工具可協助您將此資料視覺化。 本文提供的解決方案使用「彈性堆疊」將這些記錄視覺化，該功能可讓您快速編製索引並以視覺化方式在 Kibana 儀表板上呈現流量記錄。

## <a name="scenario"></a>案例

在本文中，我們將設定一個解決方案，讓您使用「彈性堆疊」將網路安全性群組流量記錄視覺化。  Logstash 輸入外掛程式會直接從為了容納流量記錄而設定的儲存體 Blob 取得流量記錄。 然後，使用「彈性堆疊」，替流量記錄編製索引並用來建立 Kibana 儀表板，以將資訊視覺化。

![案例][scenario]

## <a name="steps"></a>步驟

### <a name="enable-network-security-group-flow-logging"></a>啟用網路安全性群組流量記錄
在此案例中，您必須在您的帳戶中至少一個網路安全性群組上啟用「網路安全性群組流量記錄」。 如需有關啟用網路安全性流量記錄的指示，請參閱下列文章︰[網路安全性群組的流量記錄簡介](network-watcher-nsg-flow-logging-overview.md)。

### <a name="set-up-the-elastic-stack"></a>設定彈性堆疊
藉由連線 NSG 流量記錄與彈性堆疊，我們可以建立 Kibana 儀表板，以便從記錄搜尋、繪圖、分析和洞察。

#### <a name="install-elasticsearch"></a>安裝 Elasticsearch

1. 5.0 版和更新版本的彈性堆疊需要 Java 8。 執行命令 `java -version` 來檢查您的版本。 如果您沒有安裝 Java，請參閱 [Oracle 網站](http://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html)上的文件。
2. 針對您的系統下載正確的二進位套件︰

   ```bash
   curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.0.deb
   sudo dpkg -i elasticsearch-5.2.0.deb
   sudo /etc/init.d/elasticsearch start
   ```

   可在 [Elasticsearch 安裝](https://www.elastic.co/guide/en/beats/libbeat/5.2/elasticsearch-installation.html)找到其他安裝方法

3. 使用下列命令確認 Elasticsearch 正在執行︰

    ```bash
    curl http://127.0.0.1:9200
    ```

    您應該會看到如下所示的回應：

    ```json
    {
    "name" : "Angela Del Toro",
    "cluster_name" : "elasticsearch",
    "version" : {
        "number" : "5.2.0",
        "build_hash" : "8ff36d139e16f8720f2947ef62c8167a888992fe",
        "build_timestamp" : "2016-01-27T13:32:39Z",
        "build_snapshot" : false,
        "lucene_version" : "6.1.0"
    },
    "tagline" : "You Know, for Search"
    }
    ```

如需安裝彈性搜尋的進一步指示，請參閱[安裝指示](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/_installation.html)。

### <a name="install-logstash"></a>安裝 Logstash

1. 若要安裝 Logstash，請執行下列命令︰

    ```bash
    curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
    sudo dpkg -i logstash-5.2.0.deb
    ```
2. 接下來，我們必須設定 Logstash 以存取並剖析流量記錄。 建立 logstash.conf 檔案，使用︰

    ```bash
    sudo touch /etc/logstash/conf.d/logstash.conf
    ```

3. 將下列內容新增至檔案：

   ```
   input {
      azureblob
        {
            storage_account_name => "mystorageaccount"
            storage_access_key => "VGhpcyBpcyBhIGZha2Uga2V5Lg=="
            container => "insights-logs-networksecuritygroupflowevent"
            codec => "json"
            # Refer https://docs.microsoft.com/azure/network-watcher/network-watcher-read-nsg-flow-logs
            # Typical numbers could be 21/9 or 12/2 depends on the nsg log file types
            file_head_bytes => 12
            file_tail_bytes => 2
            # Enable / tweak these settings when event is too big for codec to handle.
            # break_json_down_policy => "with_head_tail"
            # break_json_batch_count => 2
        }
      }

      filter {
        split { field => "[records]" }
        split { field => "[records][properties][flows]"}
        split { field => "[records][properties][flows][flows]"}
        split { field => "[records][properties][flows][flows][flowTuples]"}

     mutate{
      split => { "[records][resourceId]" => "/"}
      add_field => {"Subscription" => "%{[records][resourceId][2]}"
                    "ResourceGroup" => "%{[records][resourceId][4]}"
                    "NetworkSecurityGroup" => "%{[records][resourceId][8]}"}
      convert => {"Subscription" => "string"}
      convert => {"ResourceGroup" => "string"}
      convert => {"NetworkSecurityGroup" => "string"}
      split => { "[records][properties][flows][flows][flowTuples]" => ","}
      add_field => {
                  "unixtimestamp" => "%{[records][properties][flows][flows][flowTuples][0]}"
                  "srcIp" => "%{[records][properties][flows][flows][flowTuples][1]}"
                  "destIp" => "%{[records][properties][flows][flows][flowTuples][2]}"
                  "srcPort" => "%{[records][properties][flows][flows][flowTuples][3]}"
                  "destPort" => "%{[records][properties][flows][flows][flowTuples][4]}"
                  "protocol" => "%{[records][properties][flows][flows][flowTuples][5]}"
                  "trafficflow" => "%{[records][properties][flows][flows][flowTuples][6]}"
                  "traffic" => "%{[records][properties][flows][flows][flowTuples][7]}"
                   }
      convert => {"unixtimestamp" => "integer"}
      convert => {"srcPort" => "integer"}
      convert => {"destPort" => "integer"}        
     }

     date{
       match => ["unixtimestamp" , "UNIX"]
     }
    }
   output {
     stdout { codec => rubydebug }
     elasticsearch {
       hosts => "localhost"
       index => "nsg-flow-logs"
     }
   }  
   ```

如需安裝 Logstash 的進一步指示，請參閱[正式文件](https://www.elastic.co/guide/en/beats/libbeat/5.2/logstash-installation.html)。

### <a name="install-the-logstash-input-plugin-for-azure-blob-storage"></a>安裝 Azure blob 儲存體的 Logstash 輸入外掛程式

此 Logstash 外掛程式可讓您直接從流量記錄的指定儲存體帳戶存取它們。 若要安裝此外掛程式，請從預設的 Logstash 安裝目錄 (在此案例中為 /usr/share/logstash/bin) 執行下列命令：

```bash
logstash-plugin install logstash-input-azureblob
```

若要啟動 Logstash，請執行命令︰

```bash
sudo /etc/init.d/logstash start
```

如需此外掛程式的詳細資訊，請參閱[文件](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob)。

### <a name="install-kibana"></a>安裝 Kibana

1. 執行下列命令以安裝 Kibana：

   ```bash
   curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-5.2.0-linux-x86_64.tar.gz
   tar xzvf kibana-5.2.0-linux-x86_64.tar.gz
   ```

2. 若要執行 Kibana，請使用這些命令︰

   ```bash
   cd kibana-5.2.0-linux-x86_64/
   ./bin/kibana
   ```

3. 若要檢視 Kibana Web 介面，請瀏覽至`http://localhost:5601`
4. 在此案例中，用於流量記錄的索引模式為 "nsg-flow-logs"。 您可以變更 logstash.conf 檔案的 [輸出] 區段中的索引模式。
5. 如果您想要從遠端檢視 Kibana 儀表板，建立輸入 NSG 規則以允許存取**連接埠 5601**。

### <a name="create-a-kibana-dashboard"></a>建立 Kibana 儀表板

下圖顯示用來檢視警示中趨勢和詳細資料的範例儀表板：

![圖 1][1]

下載[儀表板檔案](https://aka.ms/networkwatchernsgflowlogdashboard)、[視覺效果檔案](https://aka.ms/networkwatchernsgflowlogvisualizations)，以及[儲存的搜尋檔案](https://aka.ms/networkwatchernsgflowlogsearch)。

在 Kibana 的 [管理] 索引標籤下，瀏覽至 [儲存的物件] 並匯入這三個檔案。 然後您可以從 [儀表板] 索引標籤開啟並載入範例儀表板。

您也可以針對自己感興趣的計量，量身製作自己的視覺效果和儀表板。 從 Kibana 的[正式文件](https://www.elastic.co/guide/en/kibana/current/visualize.html)深入了解如何建立 Kibana 視覺效果。

### <a name="visualize-nsg-flow-logs"></a>將 NSG 流量記錄視覺化

範例儀表板會提供流量記錄的數個視覺效果︰

1. 一段時間各決策/方向的流量 - 顯示一段期間內流量數目的時間序列圖。 您可以編輯這些視覺效果的時間單位和範圍。 「各決策的流量」顯示允許或拒絕所做決策的比例，而「各方向的流量」則顯示輸入和輸出流量的比例。 使用這些視覺效果，您可以檢查一段時間的流量趨勢，並尋找任何突增狀況或不尋常的模式。

   ![圖 2][2]

2. 各目的地/來源連接埠的流量 – 圓形圖，可顯示個別連接埠的流量分解。 在此檢視中，您可以查看最常使用的連接埠。 如果您按一下圓形圖內的特定連接埠，則儀表板的其餘部分會進一步篩選至該連接埠的流量。

   ![圖 3][3]

3. 流量數目和最早記錄時間 – 顯示已記錄流量數目和最舊記錄擷取日期之計量。

   ![圖 4][4]

4. 各 NSG 和規則的流量 – 長條圖，可顯示每個 NSG 內的流量分布，以及每個 NSG 內的規則。 您可以在這裡查看哪些 NSG 和規則產生最多流量。

   ![圖 5][5]

5. 前 10 個來源/目的地 IP – 長條圖，可顯示前 10 個來源和目的地 IP。 您可以調整這些圖表以顯示更多或更少的 IP 排名。 您可以在這裡查看最常出現的 IP，以及針對每個 IP 進行的流量決策 (允許或拒絕)。

   ![圖 6][6]

6. 流量 Tuple – 下表顯示每個流量 Tuple 內含的資訊，以及其對應的 NGS 和規則。

   ![圖 7][7]

使用儀表板頂端的查詢列，您可以根據任何流量參數 (例如訂用帳戶識別碼、資源群組、規則或任何其他感興趣的變數)，進一步篩選儀表板。 如需 Kibana 查詢與篩選器的詳細資訊，請參閱[正式文件](https://www.elastic.co/guide/en/beats/packetbeat/current/kibana-queries-filters.html)

## <a name="conclusion"></a>結論

結合網路安全性群組流量記錄與彈性堆疊，我們提供功能強大且可自訂的方式來將網路流量視覺化。 這些儀表板可讓您快速取得和分享您的網路流量深入解析，以及進一步篩選和調查任何潛在的異常狀況。 您可以使用 Kibana 量身製作這些儀表板並建立特定視覺效果，以符合任何安全性、稽核和合規性需求。

## <a name="next-steps"></a>後續步驟

若要了解如何利用 Power BI 將 NSG 流量記錄視覺化，請瀏覽[利用 Power BI 將 NSG 流量記錄視覺](network-watcher-visualize-nsg-flow-logs-power-bi.md)

<!--Image references-->

[scenario]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/scenario.png
[1]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure1.png
[2]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure2.png
[3]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure3.png
[4]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure4.png
[5]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure5.png
[6]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure6.png
[7]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure7.png
