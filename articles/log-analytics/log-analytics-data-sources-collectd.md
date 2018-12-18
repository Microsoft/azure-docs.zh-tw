---
title: 在 OMS Log Analytics 中從 CollectD 收集資料 | Microsoft Docs
description: CollectD 是開放原始碼 Linux 精靈，可定期收集來自應用程式的資料和系統等級資訊。  本文提供如何在 Log Analytics 中從 CollectD 收集資料的相關資訊。
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
ms.date: 05/02/2017
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 59b6f8b82d0f714d4526147b42f68e14bf0aa2bd
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37127691"
---
# <a name="collect-data-from-collectd-on-linux-agents-in-log-analytics"></a>在 Log Analytics 中，從 Linux 代理程式上的 CollectD 收集資料
[CollectD](https://collectd.org/) 是開放原始碼 Linux 精靈，可定期收集來自應用程式的效能計量和系統等級資訊。 範例應用程式包括 Java 虛擬機器 (JVM)、MySQL 伺服器和 Nginx。 本文提供如何在 Log Analytics 中從 CollectD 收集效能資料的相關資訊。

您可以在[外掛程式表](https://collectd.org/wiki/index.php/Table_of_Plugins)中找到可用之外掛程式的完整清單。

![CollectD 概觀](media/log-analytics-data-sources-collectd/overview.png)

OMS Agent for Linux 包含下列 CollectD 設定，可將 CollectD 資料傳遞至 OMS Agent for Linux。

    LoadPlugin write_http

    <Plugin write_http>
         <Node "oms">
         URL "127.0.0.1:26000/oms.collectd"
         Format "JSON"
         StoreRates true
         </Node>
    </Plugin>

此外，如果是使用 5.5 之前的 collectD 版本，請改為使用下列設定。

    LoadPlugin write_http

    <Plugin write_http>
       <URL "127.0.0.1:26000/oms.collectd">
        Format "JSON"
         StoreRates true
       </URL>
    </Plugin>

CollectD 設定使用預設的 `write_http` 外掛程式，可將效能計量資料透過連接埠 26000 傳送至 OMS Agent for Linux。 

> [!NOTE]
> 如有需要，您可以將此連接埠設定為自行定義的連接埠。

OMS Agent for Linux 也會在連接埠 26000 接聽 CollectD 計量，然後將它們轉換成 OMS 結構描述計量。 以下是 OMS Agent for Linux 設定 `collectd.conf`。

    <source>
      type http
      port 26000
      bind 127.0.0.1
    </source>

    <filter oms.collectd>
      type filter_collectd
    </filter>


## <a name="versions-supported"></a>支援的版本
- Log Analytics 目前支援 CollectD 4.8 版和更新版本。
- 需要有 OMS Agent for Linux v1.1.0-217 或以上才能收集 CollectD 計量。


## <a name="configuration"></a>組態
以下是在 Log Analytics 中設定收集 CollectD 資料的基本步驟。

1. 使用 write_http 外掛程式，設定 CollectD 將資料傳送至 OMS Agent for Linux。  
2. 設定 OMS Agent for Linux 在適當的連接埠上接聽 CollectD 資料。
3. 重新啟動 CollectD 和 OMS Agent for Linux。

### <a name="configure-collectd-to-forward-data"></a>設定 CollectD 來轉送資料 

1. 若要將 CollectD 資料傳遞至 OMS Agent for Linux，需要將 `oms.conf` 新增至 CollectD 的設定目錄。 這個檔案的目的地取決於您電腦的 Linux 散發版本。

    如果您的 CollectD 設定目錄位於 /etc/collectd.d/：

        sudo cp /etc/opt/microsoft/omsagent/sysconf/omsagent.d/oms.conf /etc/collectd.d/oms.conf

    如果您的 CollectD 設定目錄位於 /etc/collectd/collectd.conf.d/：

        sudo cp /etc/opt/microsoft/omsagent/sysconf/omsagent.d/oms.conf /etc/collectd/collectd.conf.d/oms.conf

    >[!NOTE]
    >如果是 5.5 以前的 CollectD 版本，您就必須修改 `oms.conf` 中的標記，如上所示。
    >

2. 將 collectd.conf 複製到所需之工作區的 omsagent 設定目錄。

        sudo cp /etc/opt/microsoft/omsagent/sysconf/omsagent.d/collectd.conf /etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.d/
        sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.d/collectd.conf

3. 使用下列命令重新啟動 CollectD 和 OMS Agent for Linux。

    sudo service collectd restart  sudo /opt/microsoft/omsagent/bin/service_control restart

## <a name="collectd-metrics-to-log-analytics-schema-conversion"></a>CollectD 計量至 Log Analytics 的結構描述轉換
在 OMS Agent for Linux 已收集的基礎結構計量和 CollectD 所收集的新計量之間，為了維持一種熟悉的模型，我們使用下列結構描述對應：

| CollectD 計量欄位 | Log Analytics 欄位 |
|:--|:--|
| host | 電腦 |
| plugin | None |
| plugin_instance | 執行個體名稱<br>If **plugin_instance** is *null* then InstanceName="*_Total*" |
| type | ObjectName |
| type_instance | CounterName<br>If **type_instance** is *null* then CounterName=**blank** |
| dsnames[] | CounterName |
| dstypes | None |
| values[] | CounterValue |

## <a name="next-steps"></a>後續步驟
* 了解 [記錄搜尋](log-analytics-log-searches.md) ，其可分析從資料來源和方案所收集的資料。 
* 使用 [自訂欄位](log-analytics-custom-fields.md) ，以將來自 syslog 記錄的資料剖析至個別欄位。

