---
title: 收集與分析 OMS Log Analytics 中的 Syslog 訊息 | Microsoft Docs
description: Syslog 是通用於 Linux 的事件記錄通訊協定。 本文說明如何在 Log Analytics 中設定收集 Syslog 訊息，以及它們在 OMS 儲存機制中建立的記錄詳細資料。
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
ms.date: 09/28/2017
ms.author: magoedte;bwren
ms.component: na
ms.openlocfilehash: 1e7806e802f9b1dda16a9d5d477783663d03d416
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37131781"
---
# <a name="syslog-data-sources-in-log-analytics"></a>Log Analytics 中的 Syslog 資料來源
Syslog 是通用於 Linux 的事件記錄通訊協定。  應用程式將傳送的訊息可能會儲存在本機電腦上，或傳遞到 Syslog 收集器。  安裝 OMS Agent for Linux 時，它會設定本機 Syslog 精靈來將訊息轉送到代理程式。  接著，代理程式會將訊息傳送到 Log Analytics，其中對應的記錄會建立於 OMS 儲存機制中。  

> [!NOTE]
> Log Analytics 支援收集由 rsyslog 或 syslog-ng 所傳送的訊息，其中 rsyslog 是預設精靈。 Red Hat Enterprise Linux 第 5 版、CentOS 和 Oracle Linux 版本 (sysklog) 不支援預設 syslog 精靈，進行 syslog 事件收集。 若要從這些散發套件的這個版本收集 syslog 資料，應該安裝並設定 [rsyslog 精靈](http://rsyslog.com) 來取代 sysklog。
>
>

![Syslog 收集](media/log-analytics-data-sources-syslog/overview.png)

## <a name="configuring-syslog"></a>設定 Syslog
OMS Agent for Linux 只會收集具有其組態中指定之設備和嚴重性的事件。  您可以透過 Azure 入口網站，或藉由管理您 Linux 代理程式上的組態檔來設定 Syslog。

### <a name="configure-syslog-in-the-azure-portal"></a>在 Azure 入口網站中設定 Syslog
從 [Log Analytics 進階設定中的 [資料] 功能表](log-analytics-data-sources.md#configuring-data-sources)設定 Syslog。  這個組態會傳遞到每個 Linux 代理程式上的組態檔。

您可以輸入新設備的名稱，然後按一下 **+** 設定 Syslog。  針對每個設備，僅會收集包含所選嚴重性的訊息。  請檢查您想要收集之特定設備的嚴重性。  您無法提供任何其他準則來篩選訊息。

![設定 Syslog](media/log-analytics-data-sources-syslog/configure.png)

根據預設，所有設定變更都會自動發送給所有代理程式。  如果您想在每個 Linux 代理程式上手動設定 Syslog，則可取消核取 *[Apply below configuration to my Linux machines]* \(將下列設定套用至我的 Linux 機器) 方塊。

### <a name="configure-syslog-on-linux-agent"></a>在 Linux 代理程式上設定 Syslog
當 [OMS 代理程式安裝於 Linux 用戶端](log-analytics-linux-agents.md)時，它會安裝預設的 syslog 組態檔，其中會定義所收集之資訊的設備和嚴重性。  您可以修改此檔案來變更組態。  組態檔會根據用戶端已安裝的 Syslog 精靈而有所不同。

> [!NOTE]
> 如果您編輯 syslog 組態，則必須重新啟動 syslog 精靈，變更才會生效。
>
>

#### <a name="rsyslog"></a>rsyslog
Rsyslog 的組態檔位於 **/etc/rsyslog.d/95-omsagent.conf**。  其預設內容如下所示。  這會針對層級為警告或以上的所有設備收集傳送自本機代理程式的 syslog 訊息。

    kern.warning       @127.0.0.1:25224
    user.warning       @127.0.0.1:25224
    daemon.warning     @127.0.0.1:25224
    auth.warning       @127.0.0.1:25224
    syslog.warning     @127.0.0.1:25224
    uucp.warning       @127.0.0.1:25224
    authpriv.warning   @127.0.0.1:25224
    ftp.warning        @127.0.0.1:25224
    cron.warning       @127.0.0.1:25224
    local0.warning     @127.0.0.1:25224
    local1.warning     @127.0.0.1:25224
    local2.warning     @127.0.0.1:25224
    local3.warning     @127.0.0.1:25224
    local4.warning     @127.0.0.1:25224
    local5.warning     @127.0.0.1:25224
    local6.warning     @127.0.0.1:25224
    local7.warning     @127.0.0.1:25224

您可以藉由移除組態檔的設備區段來移除該設備。  您可以藉由修改特定設備的項目，來限制針對該設備所收集的嚴重性。  例如，若要將使用者設備限制為嚴重性為錯誤或以上的訊息，您要將組態檔的那一行修改為下列內容：

    user.error    @127.0.0.1:25224


#### <a name="syslog-ng"></a>syslog-ng
Syslog-ng 的組態檔位於 **/etc/syslog-ng/syslog-ng.conf**。  其預設內容如下所示。  這會針對所有設備和所有嚴重性收集傳送自本機代理程式的 syslog 訊息。   

    #
    # Warnings (except iptables) in one file:
    #
    destination warn { file("/var/log/warn" fsync(yes)); };
    log { source(src); filter(f_warn); destination(warn); };

    #OMS_Destination
    destination d_oms { udp("127.0.0.1" port(25224)); };

    #OMS_facility = auth
    filter f_auth_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(auth); };
    log { source(src); filter(f_auth_oms); destination(d_oms); };

    #OMS_facility = authpriv
    filter f_authpriv_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(authpriv); };
    log { source(src); filter(f_authpriv_oms); destination(d_oms); };

    #OMS_facility = cron
    filter f_cron_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(cron); };
    log { source(src); filter(f_cron_oms); destination(d_oms); };

    #OMS_facility = daemon
    filter f_daemon_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(daemon); };
    log { source(src); filter(f_daemon_oms); destination(d_oms); };

    #OMS_facility = kern
    filter f_kern_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(kern); };
    log { source(src); filter(f_kern_oms); destination(d_oms); };

    #OMS_facility = local0
    filter f_local0_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(local0); };
    log { source(src); filter(f_local0_oms); destination(d_oms); };

    #OMS_facility = local1
    filter f_local1_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(local1); };
    log { source(src); filter(f_local1_oms); destination(d_oms); };

    #OMS_facility = mail
    filter f_mail_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(mail); };
    log { source(src); filter(f_mail_oms); destination(d_oms); };

    #OMS_facility = syslog
    filter f_syslog_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(syslog); };
    log { source(src); filter(f_syslog_oms); destination(d_oms); };

    #OMS_facility = user
    filter f_user_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(user); };
    log { source(src); filter(f_user_oms); destination(d_oms); };

您可以藉由移除組態檔的設備區段來移除該設備。  您可以限制針對特定設備收集的嚴重性，方法是從其清單中移除它們。  例如，若要將使用者設備限制為只有警示和重大訊息，您要將組態檔的那一個區段修改為下列內容：

    #OMS_facility = user
    filter f_user_oms { level(alert,crit) and facility(user); };
    log { source(src); filter(f_user_oms); destination(d_oms); };


### <a name="collecting-data-from-additional-syslog-ports"></a>從其他 Syslog 連接埠收集資料
OMS 代理程式會在本機用戶端的連接埠 25224 上接聽 Syslog 訊息。  安裝代理程式時，會套用預設 syslog 組態，並可在下列位置找到：

* Rsyslog：`/etc/rsyslog.d/95-omsagent.conf`
* Syslog-ng：`/etc/syslog-ng/syslog-ng.conf`

您可以變更連接埠號碼，方法是建立兩個組態檔：FluentD 組態檔和 rsyslog-or-syslog-ng 檔案，依您已安裝的 Syslog 精靈而定。  

* FluentD 組態檔應該是新的檔案，位於：`/etc/opt/microsoft/omsagent/conf/omsagent.d`，且會將**連接埠**項目中的值取代為自訂連接埠號碼。

        <source>
          type syslog
          port %SYSLOG_PORT%
          bind 127.0.0.1
          protocol_type udp
          tag oms.syslog
        </source>
        <filter oms.syslog.**>
          type filter_syslog
        </filter>

* 針對 rsyslog，您應建立新的組態檔，位於：`/etc/rsyslog.d/`，並將 %SYSLOG_PORT% 值取代為您的自訂連接埠號碼。  

    > [!NOTE]
    > 如果您在 `95-omsagent.conf` 組態檔中修改這個值，就會在代理程式套用預設組態時將它覆寫。
    >

        # OMS Syslog collection for workspace %WORKSPACE_ID%
        kern.warning              @127.0.0.1:%SYSLOG_PORT%
        user.warning              @127.0.0.1:%SYSLOG_PORT%
        daemon.warning            @127.0.0.1:%SYSLOG_PORT%
        auth.warning              @127.0.0.1:%SYSLOG_PORT%

* 應修改 syslog ng 組態，方法是複製如下所示的範例組態，並將自訂修改的設定新增至位於 `/etc/syslog-ng/` 之 syslog ng.conf 組態檔的結尾。  請**勿**使用預設標籤 **%workspace_id%_oms** 或 **%workspace_id_oms**，定義自訂標籤可協助您辨別變更。  

    > [!NOTE]
    > 如果您在組態檔中修改預設值，就會在代理程式套用預設組態時將它覆寫。
    >

        filter f_custom_filter { level(warning) and facility(auth; };
        destination d_custom_dest { udp("127.0.0.1" port(%SYSLOG_PORT%)); };
        log { source(s_src); filter(f_custom_filter); destination(d_custom_dest); };

完成變更後，必須將 Syslog 和 OMS 代理程式服務重新啟動，才能確保組態變更生效。   

## <a name="syslog-record-properties"></a>Syslog 記錄屬性
Syslog 記錄具有 **Syslog** 類型，以及下表中的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| 電腦 |收集事件的來源電腦。 |
| Facility |定義產生訊息之系統的一部分。 |
| HostIP |傳送訊息之系統的 IP 位址。 |
| HostName |傳送訊息之系統的名稱。 |
| SeverityLevel |事件的嚴重性層級。 |
| SyslogMessage |訊息的文字。 |
| ProcessID |產生訊息的處理序識別碼。 |
| EventTime |產生事件的日期和時間。 |

## <a name="log-queries-with-syslog-records"></a>含有 Syslog 記錄的記錄查詢
下表提供擷取 Syslog 記錄的不同記錄查詢範例。

| 查詢 | 說明 |
|:--- |:--- |
| syslog |所有的 Syslog。 |
| Syslog &#124; where SeverityLevel == "error" |嚴重性為錯誤的所有 Syslog 記錄。 |
| Syslog &#124; summarize AggregatedValue = count() by Computer |依電腦的 Syslog 記錄計數。 |
| Syslog &#124; summarize AggregatedValue = count() by Facility |依設備的 Syslog 記錄計數。 |

## <a name="next-steps"></a>後續步驟
* 了解 [記錄搜尋](log-analytics-log-searches.md) ，其可分析從資料來源和方案所收集的資料。
* 使用 [自訂欄位](log-analytics-custom-fields.md) ，以將來自 syslog 記錄的資料剖析至個別欄位。
* [設定 Linux 代理程式](log-analytics-linux-agents.md) ，以收集其他類型的資料。
