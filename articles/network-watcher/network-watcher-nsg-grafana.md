---
title: 使用網路監看員和 Grafana 管理網路安全性群組流量記錄 | Microsoft Docs
description: 使用網路監看員和 Grafana 在 Azure 中管理和分析網路安全性群組流量記錄。
services: network-watcher
documentationcenter: na
author: kumudD
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/15/2017
ms.author: kumud
ms.openlocfilehash: 44cf074223c88b8fa539144c0d948e68ae6cbd13
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23036523"
---
# <a name="manage-and-analyze-network-security-group-flow-logs-using-network-watcher-and-grafana"></a>使用網路監看員和 Grafana 來管理和分析網路安全性群組流量記錄

[網路安全性群組 (NSG) 流量記錄](network-watcher-nsg-flow-logging-overview.md)提供的資訊可讓您用來了解網路介面上的輸入和輸出 IP 流量。 這些流量記錄會顯示每一個 NSG 規則的輸出和輸入流量、套用流量的 NIC、有關流量的 5 Tuple 資訊 (來源/目的地 IP、來源/目的地連接埠、通訊協定)，以及流量是獲得允許還是遭到拒絕。

您可以在已啟用流量記錄的網路中擁有許多個 NSG。 這樣的記錄資料量會讓您在剖析以及從記錄中取得見解時變得很麻煩。 本文提供了可使用 Grafana (開放原始碼的繪圖工具)、ElasticSearch (分散式的搜尋和分析引擎) 以及 Logstash (開放原始碼的伺服器端資料處理管線) 來集中管理這些 NSG 流量記錄的解決方案。  

## <a name="scenario"></a>案例

NSG 流量記錄可使用網路監看員來啟用，並且會儲存在 Azure Blob 儲存體中。 Logstash 外掛程式可用來從 Blob 儲存體連線和處理流量記錄，並將記錄傳送到 ElasticSearch。  流量記錄儲存在 ElasticSearch 後，就可供進行分析並於 Grafana 的自訂儀表板中視覺化呈現。

![NSG 網路監看員 Grafana](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig1.png)

## <a name="installation-steps"></a>安裝步驟

### <a name="enable-network-security-group-flow-logging"></a>啟用網路安全性群組流量記錄

在此案例中，您必須在您的帳戶中至少一個網路安全性群組上啟用「網路安全性群組流量記錄」。 如需有關啟用網路安全性流量記錄的指示，請參閱下列文章︰[網路安全性群組的流量記錄簡介](network-watcher-nsg-flow-logging-overview.md)。

### <a name="setup-considerations"></a>設定考量

此範例會在部署於 Azure 的 Ubuntu 16.04 LTS 伺服器上設定 Grafana、ElasticSearch 和 Logstash。 我們會使用這個最小型的設定來執行這三個元件，它們全都會在相同的 VM 上執行。 此設定只應用於測試和非關鍵性的工作負載。 Logstash、Elasticsearch 和 Grafana 全都可以建構為可跨多個執行個體獨立擴充。 如需詳細資訊，請參閱這些元件各自的文件。

### <a name="install-logstash"></a>安裝 Logstash

您可以使用 Logstash 將 JSON 格式的流量記錄壓平合併至流量 Tuple 層級。

1. 若要安裝 Logstash，請執行下列命令︰

    ```bash
    curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
    sudo dpkg -i logstash-5.2.0.deb
    ```

2. 設定 Logstash 來剖析流量記錄，並將記錄傳送至 ElasticSearch。 使用下列命令建立 Logstash.conf 檔案︰

    ```bash
    sudo touch /etc/logstash/conf.d/logstash.conf
    ```

3. 將下列內容新增至該檔案。 變更儲存體帳戶名稱和存取金鑰，以反映您的儲存體帳戶詳細資料：

    ```bash
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

      mutate {
        split => { "[records][resourceId]" => "/"}
        add_field => { "Subscription" => "%{[records][resourceId][2]}"
          "ResourceGroup" => "%{[records][resourceId][4]}"
          "NetworkSecurityGroup" => "%{[records][resourceId][8]}" 
        }
        convert => {"Subscription" => "string"}
        convert => {"ResourceGroup" => "string"}
        convert => {"NetworkSecurityGroup" => "string"}
        split => { "[records][properties][flows][flows][flowTuples]" => "," }
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
      elasticsearch {
        hosts => "localhost"
        index => "nsg-flow-logs"
      }
    }
    ```

所提供的 Logstash 設定檔是由三個部分組成的：輸入、篩選和輸出。 「輸入」區段會指定 Logstash 將要處理之記錄的輸入來源，在此案例中，我們會使用「azureblob」輸入外掛程式 (會於後續步驟中安裝)，它能讓我們存取 Blob 儲存體中所儲存的 NSG 流量記錄 JSON 檔案。 

「篩選」區段接著會將每個流量記錄壓平合併，讓每個個別的流量 Tuple 和其相關聯的屬性變成個別的 Logstash 事件。

最後，「輸出」區段會將每個 Logstash 事件轉送到 ElasticSearch 伺服器。 請放心地修改 Logstash 設定檔以符合您的特定需求。

### <a name="install-the-logstash-input-plugin-for-azure-blob-storage"></a>安裝 Azure Blob 儲存體的 Logstash 輸入外掛程式

這個 Logstash 外掛程式可讓您直接從流量記錄的指定 Blob 儲存體帳戶存取流量記錄。 若要安裝此外掛程式，請從預設的 Logstash 安裝目錄 (在此案例中為 /usr/share/logstash/bin) 執行下列命令：

```bash
cd /usr/share/logstash/bin
sudo ./logstash-plugin install logstash-input-azureblob
```

如需此外掛程式的詳細資訊，請參閱 [Azure 儲存體 Blob 的 Logstash 輸入外掛程式](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob)。

### <a name="install-elasticsearch"></a>安裝 ElasticSearch

您可以使用下列指令碼來安裝 ElasticSearch。 如需有關安裝 ElasticSearch 的資訊，請參閱[彈性堆疊](https://www.elastic.co/guide/en/elastic-stack/current/index.html)。

```bash
apt-get install apt-transport-https openjdk-8-jre-headless uuid-runtime pwgen -y
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | apt-key add -
echo "deb https://packages.elastic.co/elasticsearch/5.x/debian stable main" | tee -a /etc/apt/sources.list.d/elasticsearch-5.x.list
apt-get update && apt-get install elasticsearch
sed -i s/#cluster.name:.*/cluster.name:\ grafana/ /etc/elasticsearch/elasticsearch.yml
systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
```

### <a name="install-grafana"></a>安裝 Grafana

若要安裝和執行 Grafana，請執行下列命令︰

```bash
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana_4.5.1_amd64.deb
sudo apt-get install -y adduser libfontconfig
sudo dpkg -i grafana_4.5.1_amd64.deb
sudo service grafana-server start
```

如需其他安裝資訊，請參閱[在 Debian/Ubuntu 上安裝](http://docs.grafana.org/installation/debian/)。

#### <a name="add-the-elasticsearch-server-as-a-data-source"></a>將 ElasticSearch 伺服器新增為資料來源

接下來，您必須將包含流量記錄的 ElasticSearch 索引新增為資料來源。 您可以藉由選取 [新增資料來源] 並在表單內填妥相關資訊，來新增資料來源。 您可以在下列螢幕擷取畫面找到這項設定的範例：

![新增資料來源](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig2.png)

#### <a name="create-a-dashboard"></a>建立儀表板

您已成功地將 Grafana 設定為讀取包含 NSG 流量記錄的 ElasticSearch 索引，接下來您可以建立儀表板並將其個人化。 若要建立新的儀表板，請選取 [建立第一個儀表板]。 下列圖形設定範例會顯示依 NSG 規則所區分的流量：

![儀表板圖形](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig3.png)

下列螢幕擷取畫面會描繪圖形和圖表來顯示最大的幾個流量及其頻率。 流量也會依 NSG 規則來顯示，以及依決策來顯示。 Grafana 具有高度自訂能力，因此建議您建立儀表板以符合您的特定監視需求。 下列範例顯示一般的儀表板：

![儀表板圖形](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig4.png)

## <a name="conclusion"></a>結論

藉由整合網路監看員與 ElasticSearch 和 Grafana，您現在可以方便且集中地管理 NSG 流量記錄以及其他資料，並加以視覺化。 Grafana 還有許多功能強大的繪圖功能，可供您用來進一步管理流量記錄並深入了解您的網路流量。 您已經設定好 Grafana 執行個體並將其連線到 Azure，接下來請放心地繼續瀏覽它所提供的其他功能。

## <a name="next-steps"></a>後續步驟

- 深入了解如何使用[網路監看員](network-watcher-monitoring-overview.md)。

