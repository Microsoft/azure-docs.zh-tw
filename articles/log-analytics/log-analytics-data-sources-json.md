---
title: 在 OMS Log Analytics 中收集自訂 JSON 資料 | Microsoft Docs
description: 您可以使用 OMS Agent for Linux 將自訂 JSON 資料來源收集到 Log Analytics 中。  這些自訂資料來源可以是會傳回 JSON 的簡單指令碼，例如 curl 或 FluentD 的 300 個以上的外掛程式之一。 本文說明此資料收集所需的設定。
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
ms.date: 05/04/2017
ms.author: magoedte
ms.component: na
ms.openlocfilehash: d3c8807b7624e68ff55557922f97d51e24fc2c19
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37131798"
---
# <a name="collecting-custom-json-data-sources-with-the-oms-agent-for-linux-in-log-analytics"></a>在 Log Analytics 中使用 OMS Agent for Linux 收集自訂 JSON 資料來源
您可以使用 OMS Agent for Linux 將自訂 JSON 資料來源收集到 Log Analytics 中。  這些自訂資料來源可以是會傳回 JSON 的簡單指令碼，例如 [curl](https://curl.haxx.se/) 或 [FluentD 的 300 個以上的外掛程式](http://www.fluentd.org/plugins/all)之一。 本文說明此資料收集所需的設定。

> [!NOTE]
> 需要有 OMS Agent for Linux v1.1.0-217+ 才能收集自訂 JSON 資料

## <a name="configuration"></a>組態

### <a name="configure-input-plugin"></a>設定輸入外掛程式

若要在 Log Analytics 中收集 JSON 資料，請將 `oms.api.` 新增至輸入外掛程式中的 FluentD 標記開頭。

例如，以下是 `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.d/` 中的個別設定檔 `exec-json.conf`。  這會使用 FluentD 外掛程式 `exec` 每隔 30 秒執行一次 curl 命令。  此命令的輸出由 JSON 輸出外掛程式所收集。

```
<source>
  type exec
  command 'curl localhost/json.output'
  format json
  tag oms.api.httpresponse
  run_interval 30s
</source>

<match oms.api.httpresponse>
  type out_oms_api
  log_level info

  buffer_chunk_limit 5m
  buffer_type file
  buffer_path /var/opt/microsoft/omsagent/<workspace id>/state/out_oms_api_httpresponse*.buffer
  buffer_queue_limit 10
  flush_interval 20s
  retry_limit 10
  retry_wait 30s
</match>
```
對於在 `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.d/` 下新增的設定檔，必須使用下列命令變更其擁有權。

`sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/conf/omsagent.d/exec-json.conf`

### <a name="configure-output-plugin"></a>設定輸出外掛程式 
將下列輸出外掛程式設定新增至 `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.conf` 中的主要設定，或新增為個別的設定檔再放入 `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.d/` 中

```
<match oms.api.**>
  type out_oms_api
  log_level info

  buffer_chunk_limit 5m
  buffer_type file
  buffer_path /var/opt/microsoft/omsagent/<workspace id>/state/out_oms_api*.buffer
  buffer_queue_limit 10
  flush_interval 20s
  retry_limit 10
  retry_wait 30s
</match>
```

### <a name="restart-oms-agent-for-linux"></a>重新啟動 OMS Agent for Linux
使用下列命令重新啟動 OMS Agent for Linux 服務。

    sudo /opt/microsoft/omsagent/bin/service_control restart 

## <a name="output"></a>輸出
Log Analytics 中將會收集資料，記錄類型為 `<FLUENTD_TAG>_CL`。

例如，Log Analytics 中的自訂標記 `tag oms.api.tomcat`，記錄類型為 `tomcat_CL`。  您可以使用下列記錄搜尋來擷取此類型的所有記錄。

    Type=tomcat_CL

支援巢狀 JSON 資料來源，但編製索引是以父欄位作為基礎。 例如，Log Analytics 搜尋會以 `tag_s : "[{ "a":"1", "b":"2" }]` 傳回下列 JSON 資料。

```
{
    "tag": [{
        "a":"1",
        "b":"2"
    }]
}
```


## <a name="next-steps"></a>後續步驟
* 了解 [記錄搜尋](log-analytics-log-searches.md) ，其可分析從資料來源和方案所收集的資料。 
 