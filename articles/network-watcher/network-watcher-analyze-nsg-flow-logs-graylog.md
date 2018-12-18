---
title: 分析 Azure 網路安全性群組流量記錄 - Graylog | Microsoft Docs
description: 了解如何使用網路監看員和 Graylog 在 Azure 中管理和分析網路安全性群組流量記錄。
services: network-watcher
documentationcenter: na
author: mattreatMSFT
manager: vitinnan
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/19/2017
ms.author: mareat
ms.openlocfilehash: db3b08ae8092661e6ffa0f2dd7e460f341a8d013
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666054"
---
# <a name="manage-and-analyze-network-security-group-flow-logs-in-azure-using-network-watcher-and-graylog"></a>使用網路監看員和 Graylog 在 Azure 中管理和分析網路安全性群組流量記錄

[網路安全性群組流量記錄](network-watcher-nsg-flow-logging-overview.md)提供的資訊可讓您用來了解 Azure 網路介面的輸入和輸出 IP 流量。 流量記錄會顯示每一個網路安全性群組規則的輸出和輸入流量、套用流量的網路介面、有關流量的 5 Tuple 資訊 (來源/目的地 IP、來源/目的地連接埠、通訊協定)，以及流量是獲得允許還是遭到拒絕。

您可以在已啟用流量記錄的網路中擁有許多個網路安全性群組。 已啟用流量記錄的網路安全性群組如果有好幾個，在剖析以及從記錄中取得見解時就會變得很麻煩。 本文提供了可使用 Graylog (開放原始碼的記錄管理和分析工具) 以及 Logstash (開放原始碼的伺服器端資料處理管線) 來集中管理這些網路安全性群組流量記錄的解決方案。

## <a name="scenario"></a>案例

網路安全性群組流量記錄可使用網路監看員來啟用。 流量記錄會流入 Azure Blob 儲存體。 Logstash 外掛程式可用來從 Blob 儲存體連線和處理流量記錄，並將記錄傳送到 Graylog。 流量記錄儲存在 Graylog 後，就可供進行分析並於自訂儀表板中視覺化呈現。

![Graylog 工作流程](./media/network-watcher-analyze-nsg-flow-logs-graylog/workflow.png)

## <a name="installation-steps"></a>安裝步驟

### <a name="enable-network-security-group-flow-logging"></a>啟用網路安全性群組流量記錄

在此案例中，您必須在您的帳戶中至少一個網路安全性群組上啟用「網路安全性群組流量記錄」。 如需有關啟用網路安全性群組流量記錄的指示，請參閱下列文章︰[網路安全性群組流量記錄簡介](network-watcher-nsg-flow-logging-overview.md)。

### <a name="setting-up-graylog"></a>設定 Graylog

此範例會在部署於 Azure 的 Ubuntu 14.04 伺服器上同時設定 Graylog 和 Logstash。

- 請參閱 Graylog 所提供的[文件](http://docs.graylog.org/en/2.2/pages/installation/os/ubuntu.html)，以取得如何安裝到 Ubuntu 的逐步指示。
- 請務必同時遵循該[文件](http://docs.graylog.org/en/2.2/pages/configuration/web_interface.html#configuring-webif)來設定 Graylog Web 介面。

此範例會使用最小型的 Graylog 安裝 (亦即 單一 Graylog 執行個體)，但您可以將 Graylog 建構為可依據您的系統和生產環境需求來調整資源。 如需架構考量或深度架構指南的詳細資訊，請參閱 Graylog 的[文件](http://docs.graylog.org/en/2.2/pages/architecture.html)和[架構指南](https://www.slideshare.net/Graylog/graylog-engineering-design-your-architecture)。

根據平台和喜好設定，Graylog 可以進行多種方式的安裝。 如需完整的可行安裝方法清單，請參閱 Graylog 的官方[文件](http://docs.graylog.org/en/2.2/pages/installation.html)。 Graylog 伺服器應用程式會在 Linux 發行版本上執行，並具有下列必要條件：

-  Oracle Java SE 8 或更新版本 – [Oracle 的安裝文件](http://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html)
-  ElasticSearch 2.x (2.1.0 或更新版本) - [Elasticsearch 安裝文件](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/_installation.html)
-  MongoDB 2.4 或更新版本 – [MongoDB 安裝文件](https://docs.mongodb.com/manual/administration/install-on-linux/)

### <a name="install-logstash"></a>安裝 Logstash

我們會使用 Logstash 將 JSON 格式的流量記錄壓平合併至流量 Tuple 層級。 將流量記錄壓平合併可讓您更輕鬆地在 Graylog 中組織和搜尋記錄。

1. 若要安裝 Logstash，請執行下列命令︰

   ```bash
   curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
   sudo dpkg -i logstash-5.2.0.deb
   ```

2. 設定 Logstash 來剖析流量記錄，並將記錄傳送至 Graylog。 建立 Logstash.conf 檔案︰

   ```bash
   sudo touch /etc/logstash/conf.d/logstash.conf
   ```

3. 將下列內容新增至該檔案。 變更醒目提示的值，以反映您的儲存體帳戶詳細資料：

   ```
    input {
        azureblob
        {
            storage_account_name => "mystorageaccount"
            storage_access_key => "NrUZmx7pJSKaRJzvQbeiZWi5nBRWOTr7Wwr9DrvK7YtDBrADYxT1y0oEExtSlkDnGRt7qcRiZzEBCCyRYND8SxSt"
            container => "insights-logs-networksecuritygroupflowevent"
            registry_create_policy => "start_over"
            codec => "json"
            file_head_bytes => 21
            file_tail_bytes => 9
            # Possible options: `do_not_break`, `with_head_tail`, `without_head_tail`
            break_json_down_policy  => 'with_head_tail'
            break_json_batch_count => 2
            interval => 5
        }
    }
    
    filter {
        split { field => "[records]" }
        split { field => "[records][properties][flows]"}
        split { field => "[records][properties][flows][flows]"}
        split { field => "[records][properties][flows][flows][flowTuples]"
    }
    
     mutate {
        split => { "[records][resourceId]" => "/"}
        add_field =>{
                    "Subscription" => "%{[records][resourceId][2]}"
                    "ResourceGroup" => "%{[records][resourceId][4]}"
                    "NetworkSecurityGroup" => "%{[records][resourceId][8]}"
        }
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
        add_field => {
                    "time" => "%{[records][time]}"
                    "systemId" => "%{[records][systemId]}"
                    "category" => "%{[records][category]}"
                    "resourceId" => "%{[records][resourceId]}"
                    "operationName" => "%{[records][operationName}}"
                    "Version" => "%{[records][properties][Version}}"
                    "rule" => "%{[records][properties][flows][rule]}"
                    "mac" => "%{[records][properties][flows][flows][mac]}"
        }
        convert => {"unixtimestamp" => "integer"}
        convert => {"srcPort" => "integer"}
        convert => {"destPort" => "integer"}
        add_field => { "message" => "%{Message}" }
    }
        date {
            match => ["unixtimestamp" , "UNIX"]
        }
    }
    output {
        stdout { codec => rubydebug }
        udp {
            host => "127.0.0.1"
            port => 12201
        }
    }
    ```
所提供的 Logstash 設定檔是由三個部分組成的：輸入、篩選和輸出。 「輸入」區段會指定 Logstash 將要處理的記錄輸入來源，在此案例中，您會使用 Azure 部落格輸入外掛程式 (會於後續步驟中安裝)，它能讓我們存取 Blob 儲存體中所儲存的網路安全性群組流量記錄 JSON 檔案。

「篩選」區段接著會將每個流量記錄壓平合併，讓每個個別的流量 Tuple 和其相關聯的屬性變成個別的 Logstash 事件。

最後，輸出區段會將每個 Logstash 事件轉送到 Graylog 伺服器。 若要符合您的特定需求，請視需要修改 Logstash 設定檔。

   > [!NOTE]
   > 先前的組態檔假設 Graylog 伺服器已經在本機主機回送 IP 位址 127.0.0.1 上設定。 如果沒有，請務必將輸出區段中的主機參數變更為正確的 IP 位址。

如需 Logstash 的進一步安裝指示，請參閱 Logstash [文件](https://www.elastic.co/guide/en/beats/libbeat/5.2/logstash-installation.html)。

### <a name="install-the-logstash-input-plug-in-for-azure-blob-storage"></a>安裝 Azure blob 儲存體的 Logstash 輸入外掛程式

Logstash 外掛程式可讓您直接從流量記錄的指定 Blob 儲存體帳戶存取流量記錄。 若要安裝此外掛程式，請從預設的 Logstash 安裝目錄 (在此案例中為 /usr/share/logstash/bin) 執行下列命令：

```bash
cd /usr/share/logstash/bin
sudo ./logstash-plugin install logstash-input-azureblob
```

如需此外掛程式的詳細資訊，請參閱[文件](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob)。

### <a name="set-up-connection-from-logstash-to-graylog"></a>設定從 Logstash 到 Graylog 的連線

您已使用 Logstash 建立了流量記錄的連線並已設定 Graylog 伺服器，接下來您需要將 Graylog 設定為接受內送記錄檔。

1. 使用您為 Graylog 伺服器 Web 介面所設定的 URL 來瀏覽至該介面。 您可以將瀏覽器導向至 `http://<graylog-server-ip>:9000/` 來存取該介面

2. 若要瀏覽至設定頁面，請選取頂端導覽列右邊的 [系統] 下拉式功能表，然後按一下 [輸入]。
   或者，您也可以瀏覽至 `http://<graylog-server-ip>:9000/system/inputs`

   ![開始使用](./media/network-watcher-analyze-nsg-flow-logs-graylog/getting-started.png)

3. 若要啟動新的輸入，請在 [選取輸入] 下拉式清單中選取 [GELF UDP]，然後填寫表單。 GELF 代表 Graylog Extended Log Format。 GELF 格式是由 Graylog 所開發的。 若要深入了解其優點，請參閱 Graylog [文件](http://docs.graylog.org/en/2.2/pages/gelf.html)。

   請務必將輸入繫結至 Graylog 伺服器設定所在的 IP。 IP 位址應符合 Logstash 設定檔 UDP 輸出的 [主機] 欄位。 預設連接埠應為 12201。 請確保連接埠符合 Logstash 設定檔所指定之 UDP 輸出中的 [連接埠] 欄位。

   ![輸入](./media/network-watcher-analyze-nsg-flow-logs-graylog/inputs.png)

   在啟動輸入後，您應該就會看到它出現在 [本機輸入] 區段底下，如下圖所示：

   ![](./media/network-watcher-analyze-nsg-flow-logs-graylog/local-inputs.png)

   若要深入了解 Graylog 訊息輸入，請參閱[文件](http://docs.graylog.org/en/2.2/pages/sending_data.html#what-are-graylog-message-inputs)。

4. 在完成這些設定後，您就可以使用下列命令啟動 Logstash 以開始讀取流量記錄：`sudo systemctl start logstash.service`。

### <a name="search-through-graylog-messages"></a>搜尋 Graylog 訊息

在給予 Graylog 伺服器一些時間來收集訊息之後，您就可以搜尋所有訊息。 若要檢查傳送給 Graylog 伺服器的訊息，請從 [輸入] 設定頁面中按一下您所建立之 GELF UDP 輸入的 [顯示所收到的訊息] 按鈕。 系統會將您導向至類似下圖的畫面： 

![長條圖](./media/network-watcher-analyze-nsg-flow-logs-graylog/histogram.png)

按一下藍色的 “%{Message}” 連結便可展開每個訊息，以顯示每個流量 Tuple 的參數，如下圖所示：

![訊息](./media/network-watcher-analyze-nsg-flow-logs-graylog/messages.png)

依預設，如果您沒有選取要搜尋特定的訊息欄位，則系統會在搜尋中包含所有訊息欄位。 如果您想要搜尋特定訊息 (亦即 來自特定來源 IP 的流量 Tuple)，則可以如[記載](http://docs.graylog.org/en/2.2/pages/queries.html)內容所述使用 Graylog 搜尋查詢語言

## <a name="analyze-network-security-group-flow-logs-using-graylog"></a>使用 Graylog 分析網路安全性群組流量記錄

它所設定的 Graylog 已在執行，接下來您可使用它的某些功能來深入了解流量記錄資料。 其中一種方法是使用儀表板來為資料建立特定檢視。

### <a name="create-a-dashboard"></a>建立儀表板

1. 在頂端導覽列中選取 [儀表板] 或瀏覽至 `http://<graylog-server-ip>:9000/dashboards/`

2. 從該處按一下綠色的 [建立儀表板] 按鈕，然後在簡短的表單中填寫儀表板的標題和說明。 按 [儲存] 按鈕以建立新的儀表板。 您會看到類似下圖的儀表板：

    ![儀表板](./media/network-watcher-analyze-nsg-flow-logs-graylog/dashboards.png)

### <a name="add-widgets"></a>新增小工具

您可以按一下儀表板的標題來查看儀表板，由於我們還沒有新增任何小工具，所以儀表板內現在空無一物。 有一個簡單又實用的類型小工具可新增至儀表板，那就是**快速值**圖表，此圖表會顯示所選欄位值的清單和其分佈情形。

1. 藉由在頂端導覽列中選取 [搜尋]，以瀏覽回到會接收流量記錄之 UDP 輸入的搜尋結果。

2. 在畫面左邊的 [搜尋結果] 面板底下尋找 [欄位] 索引標籤，它會列出每個內送流量 Tuple 訊息的各個欄位。

3. 選取要據以視覺化的任何所需參數 (在此範例中，已選取 IP 來源)。 若要顯示可用小工具的清單，請按一下欄位左邊的藍色下拉箭頭，然後選取 [快速值] 來產生小工具。 您應該會看到類似下圖的輸出：

   ![來源 IP](./media/network-watcher-analyze-nsg-flow-logs-graylog/srcip.png)

4. 在該處，您可以選取小工具右上方的 [新增至儀表板] 按鈕，然後選取要新增的對應儀表板。

5. 瀏覽回到儀表板，以查看您剛才新增的小工具。

   您可以在儀表板中新增各種其他小工具 (例如長條圖和計數) 來追蹤重要計量，如下圖中所示的儀表板範例：

   ![流量記錄儀表板](./media/network-watcher-analyze-nsg-flow-logs-graylog/flowlogs-dashboard.png)

    如需儀表板和其他類型之小工具的進一步說明，請參閱 Graylog 的[文件](http://docs.graylog.org/en/2.2/pages/dashboards.html)。

藉由整合網路監看員與 Graylog，您現在可以方便且集中地管理網路安全性群組流量記錄並加以視覺化。 Graylog 還有許多功能強大的功能 (例如資料流和警示)，可供您用來進一步管理流量記錄並深入了解您的網路流量。 您已經設定好 Graylog 並將其連線到 Azure，接下來請放心地繼續瀏覽它所提供的其他功能。

## <a name="next-steps"></a>後續步驟

若要了解如何利用 Power BI 將網路安全性群組流量記錄視覺化，請瀏覽[利用 Power BI 將網路安全性群組流量記錄視覺化](network-watcher-visualize-nsg-flow-logs-power-bi.md)。
