---
title: 使用 Azure 網路監看員和開放原始碼工具執行網路入侵偵測 | Microsoft Docs
description: 本文說明如何使用 Azure 網路監看員和開放原始碼工具來執行網路入侵偵測
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 0f043f08-19e1-4125-98b0-3e335ba69681
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: aff1b5f9e8860d3b8dc09b37684bb8a4ac2bf134
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "23036733"
---
# <a name="perform-network-intrusion-detection-with-network-watcher-and-open-source-tools"></a>使用網路監看員和開放原始碼工具執行網路入侵偵測

封包擷取是實作網路入侵偵測系統 (ID) 和執行網路安全性監視 (NSM) 的重要元件。 有數個開放原始碼 IDS 工具，可處理封包擷取並尋找可能的網路入侵和惡意活動的簽章。 使用網路監看員所提供的封包擷取，您可以分析您的網路找出任何有害的入侵或安全性弱點。

一種開放原始碼工具是 Suricata，這是使用規則集監視網路流量且在可疑事件發生時就會觸發警示的 IDS 引擎。 Suricata 提供多執行緒的引擎，這表示它可以更高的速度和效率執行網路流量分析。 如需關於 Suricata 和其功能的詳細資訊，請瀏覽其網站：https://suricata-ids.org/。

## <a name="scenario"></a>案例

本文說明如何使用網路監看員、Suricata 和彈性堆疊來設定您的環境，以執行網路入侵偵測。 網路監看員會提供用來執行網路入侵偵測的封包擷取。 Suricata 會根據比對指定的威脅規則集處理封包擷取和觸發警示。 這些警示會儲存在本機電腦上的記錄檔。 使用彈性堆疊的 Suricata 所產生之記錄可以編製索引及用來建立 Kibana 儀表板，提供您視覺表示法的記錄，以便快速獲得潛在網路弱點的見解。  

![簡單的 web 應用程式案例][1]

可於 Azure VM 上設定這兩個開放原始碼工具，讓您能夠在您自己的 Azure 網路環境內執行這項分析。

## <a name="steps"></a>步驟

### <a name="install-suricata"></a>安裝 Suricata

如需其他所有安裝方法，請瀏覽 http://suricata.readthedocs.io/en/latest/install.html

1. 在您 VM 的命令列終端機執行下列命令︰

    ```
    sudo add-apt-repository ppa:oisf/suricata-stable
    sudo apt-get update
    sudo sudo apt-get install suricata
    ```

1. 若要確認您的安裝，請執行命令 `suricata -h` 以查看命令的完整清單。

### <a name="download-the-emerging-threats-ruleset"></a>下載新興的威脅規則集

在這個階段，我們不需要執行 Suricata 的任何規則。 如果有您想要在您的網路偵測到的特定威脅，可以建立自己的規則，或者您也可以從多個提供者使用開發的規則集，例如新興的威脅，或從 Snort 使用 VRT 規則。 我們在這裡使用自由存取的新興威脅規則集︰

下載規則集，並將它們複製到目錄︰

```
wget http://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz
tar zxf emerging.rules.tar.gz
sudo cp -r rules /etc/suricata/
```

### <a name="process-packet-captures-with-suricata"></a>使用 Suricata 處理封包擷取

若要使用 Suricata 處理封包擷取，請執行下列命令︰

```
sudo suricata -c /etc/suricata/suricata.yaml -r <location_of_pcapfile>
```
若要檢查產生的警示，請閱讀 fast.log 檔案︰
```
tail -f /var/log/suricata/fast.log
```

### <a name="set-up-the-elastic-stack"></a>設定彈性堆疊

雖然 Suricata 產生的記錄檔包含有關我們網路上所發生事件的重要資訊，並不容易閱讀並了解這些記錄檔。 藉由連線 Suricata 與彈性堆疊，我們可以建立 Kibana 儀表板，讓我們可從記錄搜尋、繪圖、分析和洞察。

#### <a name="install-elasticsearch"></a>安裝 Elasticsearch

1. 5.0 版和更新版本的彈性堆疊需要 Java 8。 執行命令 `java -version` 來檢查您的版本。 如果您沒有安裝 Java，請參閱 [Oracle 網站](http://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html)上的文件
1. 針對您的系統下載正確的二進位套件︰

    ```
    curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.0.deb
    sudo dpkg -i elasticsearch-5.2.0.deb
    sudo /etc/init.d/elasticsearch start
    ```

    可在 [Elasticsearch 安裝](https://www.elastic.co/guide/en/beats/libbeat/5.2/elasticsearch-installation.html)找到其他安裝方法

1. 使用下列命令確認 Elasticsearch 正在執行︰

    ```
    curl http://127.0.0.1:9200
    ```

    您應該會看到如下所示的回應：

    ```
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

如需安裝彈性搜尋的進一步指示，請參閱[安裝](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/_installation.html)頁面

### <a name="install-logstash"></a>安裝 Logstash

1. 若要安裝 Logstash，請執行下列命令︰

    ```
    curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
    sudo dpkg -i logstash-5.2.0.deb
    ```
1. 接下來我們需要設定 Logstash 以從 eve.json 檔案的輸出讀取。 建立 logstash.conf 檔案，使用︰

    ```
    sudo touch /etc/logstash/conf.d/logstash.conf
    ```

1. 將下列內容新增至檔案 (請確定 eve.json 檔案的路徑正確)︰

    ```ruby
    input {
    file {
        path => ["/var/log/suricata/eve.json"]
        codec =>  "json"
        type => "SuricataIDPS"
    }

    }

    filter {
    if [type] == "SuricataIDPS" {
        date {
        match => [ "timestamp", "ISO8601" ]
        }
        ruby {
        code => "
            if event.get('[event_type]') == 'fileinfo'
            event.set('[fileinfo][type]', event.get('[fileinfo][magic]').to_s.split(',')[0])
            end
        "
        }

        ruby{
        code => "
            if event.get('[event_type]') == 'alert'
            sp = event.get('[alert][signature]').to_s.split(' group ')
            if (sp.length == 2) and /\A\d+\z/.match(sp[1])
                event.set('[alert][signature]', sp[0])
            end
            end
            "
        }
    }

    if [src_ip]  {
        geoip {
        source => "src_ip"
        target => "geoip"
        #database => "/opt/logstash/vendor/geoip/GeoLiteCity.dat"
        add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
        add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
        }
        mutate {
        convert => [ "[geoip][coordinates]", "float" ]
        }
        if ![geoip.ip] {
        if [dest_ip]  {
            geoip {
            source => "dest_ip"
            target => "geoip"
            #database => "/opt/logstash/vendor/geoip/GeoLiteCity.dat"
            add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
            add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
            }
            mutate {
            convert => [ "[geoip][coordinates]", "float" ]
            }
        }
        }
    }
    }

    output {
    elasticsearch {
        hosts => "localhost"
    }
    }
    ```

1. 請務必提供正確權限給 eve.json 檔案，以便 Logstash 可內嵌檔案。
    
    ```
    sudo chmod 775 /var/log/suricata/eve.json
    ```

1. 若要啟動 Logstash，請執行命令︰

    ```
    sudo /etc/init.d/logstash start
    ```

如需安裝 Logstash 的進一步指示，請參閱[正式文件](https://www.elastic.co/guide/en/beats/libbeat/5.2/logstash-installation.html)

### <a name="install-kibana"></a>安裝 Kibana

1. 執行下列命令以安裝 Kibana：

    ```
    curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-5.2.0-linux-x86_64.tar.gz
    tar xzvf kibana-5.2.0-linux-x86_64.tar.gz

    ```
1. 若要執行 Kibana，請使用這些命令︰

    ```
    cd kibana-5.2.0-linux-x86_64/
    ./bin/kibana
    ```

1. 若要檢視 Kibana Web 介面，請瀏覽至`http://localhost:5601`
1. 此案例中，用於 Suricata 記錄的索引模式是 "logstash-*"

1. 如果您想要從遠端檢視 Kibana 儀表板，建立輸入 NSG 規則以允許存取**連接埠 5601**。

### <a name="create-a-kibana-dashboard"></a>建立 Kibana 儀表板

在本文中，我們提供了範例儀表板，讓您檢視警示中的趨勢和詳細資料。

1. 下載儀表板檔案 ([這裡](https://aka.ms/networkwatchersuricatadashboard))、視覺效果檔案 ([這裡](https://aka.ms/networkwatchersuricatavisualization))，以及儲存的搜尋檔案 ([這裡](https://aka.ms/networkwatchersuricatasavedsearch))。

1. 在 Kibana 的 [管理] 索引標籤下，瀏覽至 [儲存的物件] 並匯入這三個檔案。 然後您可以從 [儀表板] 索引標籤開啟並載入範例儀表板。

您也可以針對自己感興趣的計量，量身製作自己的視覺效果和儀表板。 從 Kibana 的[正式文件](https://www.elastic.co/guide/en/kibana/current/visualize.html)深入了解如何建立 Kibana 視覺效果。

![建立儀表板][2]

### <a name="visualize-ids-alert-logs"></a>以視覺化方式檢視 IDS 警示記錄

範例儀表板會提供 Suricata 警示記錄的數個視覺效果︰

1. 依 GeoIP 發佈警示 –根據地理位置 (由 IP 判斷) 依其國家/地區顯示警示分佈的地圖

    ![地理 IP][3]

1. 前 10 個警示 – 10 個最常見的觸發警示及其描述摘要。 按一下個別警示可向下篩選儀表板到該特定警示的相關資訊。

    ![影像 4][4]

1. 警示數目 – 依規則集所觸發的警示總計數

    ![影像 5][5]

1. 前 20 個來源/目的地 IP/連接埠 - 顯示前 20 個觸發警示的 IP 和連接埠的圓形圖。 您可以在特定的 IP/連接埠向下篩選，以查看所觸發警示的數目和類型。

    ![映像 6][6]

1. 警示摘要 – 彙總每一個個別警示的特定詳細資料的資料表。 您可以自訂這個資料表，以顯示每個警示的其他感興趣參數。

    ![映像 7][7]

如需有關建立自訂視覺效果和儀表板的文件，請參閱 [Kibana 的正式文件](https://www.elastic.co/guide/en/kibana/current/introduction.html)。

## <a name="conclusion"></a>結論

藉由結合網路監看員所提供的封包擷取和開放原始碼 IDS 工具 (例如 Suricata)，您可以執行各式各樣威脅的網路入侵偵測。 這些儀表板可讓您快速發現您網路內的趨勢和異常，並且深入瞭解來探索警示資料的根本原因，例如惡意的使用者代理程式或連接埠弱點。 利用這個擷取的資料，您可以做出關於如何回應和保護網路免於任何有害入侵嘗試，以及建立規則來避免未來入侵您網路的明智決策。

## <a name="next-steps"></a>後續步驟

若要了解如何根據警示觸發封包擷取，請造訪[使用封包擷取搭配 Azure Functions 進行主動式網路監視](network-watcher-alert-triggered-packet-capture.md)

若要了解如何利用 Power BI 將 NSG 流量記錄視覺化，請瀏覽[利用 Power BI 將 NSG 流量記錄視覺](network-watcher-visualize-nsg-flow-logs-power-bi.md)



<!-- images -->
[1]: ./media/network-watcher-intrusion-detection-open-source-tools/figure1.png
[2]: ./media/network-watcher-intrusion-detection-open-source-tools/figure2.png
[3]: ./media/network-watcher-intrusion-detection-open-source-tools/figure3.png
[4]: ./media/network-watcher-intrusion-detection-open-source-tools/figure4.png
[5]: ./media/network-watcher-intrusion-detection-open-source-tools/figure5.png
[6]: ./media/network-watcher-intrusion-detection-open-source-tools/figure6.png
[7]: ./media/network-watcher-intrusion-detection-open-source-tools/figure7.png
