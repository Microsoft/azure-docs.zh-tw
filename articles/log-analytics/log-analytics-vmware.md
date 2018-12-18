---
title: Log Analytics 中的 VMware 監視解決方案 | Microsoft Docs
description: 了解 VMware 監視解決方案如何協助您管理記錄檔和監視 ESXi 主機。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: 16516639-cc1e-465c-a22f-022f3be297f1
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/04/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 7cb78647b8ae17e066f05acc32668d5c0c8b4ff5
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43127672"
---
# <a name="vmware-monitoring-preview-solution-in-log-analytics"></a>Log Analytics 中的 VMware 監視 (預覽) 解決方案

![VMware 符號](./media/log-analytics-vmware/vmware-symbol.png)

> [!NOTE]
> VMware 監控解決方案已被取代。  已安裝此解決方案的客戶可以繼續使用，但無法將 VMware 監控加入至任何新的工作區。

Log Analytics 中的 VMware 監視解決方案是一個可協助您針對大型 VMware 記錄檔建立集中記錄和監視方法的解決方案。 本文說明如何使用此解決方案在單一位置進行疑難排解、擷取和管理 ESXi 主機。 有了這個解決方案，您可以在單一位置查看所有 ESXi 主機的詳細資料。 您可以看到 VM 和 ESXi 主機上前幾名的事件計數、狀態和趨勢，透過 ESXi 主機記錄檔提供。 您可以檢視及搜尋 ESXi 主機集中記錄檔，來進行疑難排解。 而且，您可以根據記錄檔搜尋查詢來建立警示。

解決方案會使用 ESXi 主機的原生 syslog 功能來將資料推播至具有 OMS 代理程式的目標 VM。 但是，解決方案不會將檔案寫入目標 VM 內部的 syslog。 OMS 代理程式會開啟連接埠 1514 並接聽該連接埠。 在收到資料之後，OMS 代理程式就會將資料推送至 Log Analytics 中。

## <a name="install-and-configure-the-solution"></a>安裝和設定解決方案
請使用下列資訊來安裝和設定方案。

* 使用[安裝管理解決方案](../monitoring/monitoring-solutions.md#install-a-management-solution)中所述的流程，將 VMware 監控解決方案新增至您的訂用帳戶。

#### <a name="supported-vmware-esxi-hosts"></a>支援的 VMware ESXi 主機
vSphere ESXi 主機 5.5、6.0 和 6.5

#### <a name="prepare-a-linux-server"></a>準備 Linux 伺服器
建立 Linux 作業系統 VM 來接收來自 ESXi 主機的所有 syslog 資料。 [OMS Linux 代理程式](log-analytics-linux-agents.md)是所有 ESXi 主機 syslog 資料的收集點。 您可以使用多個 ESXi 主機將記錄檔轉送到單一 Linux 伺服器，如下列範例所示。  

   ![syslog 流程](./media/log-analytics-vmware/diagram.png)

### <a name="configure-syslog-collection"></a>設定 syslog 收集
1. 設定 VSphere 的 syslog 轉送。 如需協助設定 syslog 轉送的詳細資訊，請參閱[設定 ESXi 5.0 和更新版本上的 syslog (2003322)](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2003322)。 移至 [ESXi 主機組態]  >  [軟體]  >  [進階設定]  >  [Syslog]。
   ![vsphereconfig](./media/log-analytics-vmware/vsphere1.png)  
1. 在 [Syslog.global.logHost] 欄位中，新增您的 Linux 伺服器和連接埠號碼 1514。 例如，`tcp://hostname:1514` 或 `tcp://123.456.789.101:1514`。
1. 為 syslog 開啟 ESXi 主機防火牆。 [ESXi 主機組態]  >  [軟體]  >  [安全性設定檔]  >  [防火牆]，然後開啟 [屬性]。  

    ![vspherefw](./media/log-analytics-vmware/vsphere2.png)  

    ![vspherefwproperties](./media/log-analytics-vmware/vsphere3.png)  
1. 檢查 vSphere 主控台，確認 syslog 設定正確。 確認 ESXI 主機上已設定連接埠 **1514**。
1. 在 Linux 伺服器上下載並安裝 OMS Agent for Linux。 如需詳細資訊，請參閱 [OMS Agent for Linux 的文件](https://github.com/Microsoft/OMS-Agent-for-Linux)。
1. 安裝 OMS Agent for Linux 後，移至 /etc/opt/microsoft/omsagent/sysconf/omsagent.d 目錄，將 vmware_esxi.conf 檔複製到 /etc/opt/microsoft/omsagent/conf/omsagent.d directory 目錄，並變更檔案的擁有者/群組和權限。 例如︰

    ```
    sudo cp /etc/opt/microsoft/omsagent/sysconf/omsagent.d/vmware_esxi.conf /etc/opt/microsoft/omsagent/conf/omsagent.d
   sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/conf/omsagent.d/vmware_esxi.conf
    ```
1. 執行 `sudo /opt/microsoft/omsagent/bin/service_control restart` 啟動 OMS Agent for Linux。
1. 在 ESXi 主機上使用 `nc`命令測試 Linux 伺服器和 ESXi 主機之間的連線。 例如︰

    ```
    [root@ESXiHost:~] nc -z 123.456.789.101 1514
    Connection to 123.456.789.101 1514 port [tcp/*] succeeded!
    ```

1. 在 Azure 入口網站中，執行 `VMware_CL` 的記錄搜尋。 Log Analytics 在收集 syslog 資料時，會保留 syslog 格式。 在入口網站中，會擷取某些特定欄位，例如 Hostname 和 ProcessName。  

    ![type](./media/log-analytics-vmware/type.png)  

    如果您的檢視記錄搜尋結果類似上圖，表示您已設定為可使用 VMware 監控解決方案儀表板。  

## <a name="vmware-data-collection-details"></a>VMware 資料收集詳細資料
VMware 監視解決方案會使用您已啟用的 OMS Agents for Linux，從 ESXi 主機收集各種效能度量和記錄檔資料。

下表顯示資料收集方法和其他資料收集方式的詳細資料。

| 平台 | OMS Agent for Linux | SCOM 代理程式 | Azure 儲存體 | SCOM 是否為必要項目？ | 透過管理群組傳送的 SCOM 代理程式資料 | 收集頻率 |
| --- | --- | --- | --- | --- | --- | --- |
| Linux |&#8226; |  |  |  |  |每隔 3 分鐘 |

下表顯示由 VMware 監視解決方案收集的資料欄位範例︰

| 欄位名稱 | 說明 |
| --- | --- |
| Device_s |VMware 儲存裝置 |
| ESXIFailure_s |失敗類型 |
| EventTime_t |事件發生的時間 |
| HostName_s |ESXi 主機名稱 |
| Operation_s |建立 VM或刪除 VM |
| ProcessName_s |事件名稱 |
| ResourceId_s |VMware 主機的名稱 |
| ResourceLocation_s |VMware |
| ResourceName_s |VMware |
| ResourceType_s |Hyper-V |
| SCSIStatus_s |VMware SCSI 的狀態 |
| SyslogMessage_s |syslog 資料 |
| UserName_s |建立或刪除 VM 的使用者 |
| VMName_s |VM 名稱 |
| 電腦 |主機電腦 |
| TimeGenerated |產生資料的時間 |
| DataCenter_s |VMware 資料中心 |
| StorageLatency_s |儲存體延遲 (毫秒) |

## <a name="vmware-monitoring-solution-overview"></a>VMware 監視解決方案概觀
[VMware] 圖格會出現在 Log Analytics 工作區中。 它提供任何失敗的高階檢視。 當您按一下圖格時，會進入儀表板檢視。

![圖格](./media/log-analytics-vmware/tile.png)

#### <a name="navigate-the-dashboard-view"></a>瀏覽儀表板檢視
在 VMware 儀表板檢視中，各刀鋒視窗組織如下︰

* 失敗狀態計數
* 依事件計數的主機前幾名
* 前幾名事件計數
* 虛擬機器活動
* ESXi 主機磁碟的事件

![解決方案1](./media/log-analytics-vmware/solutionview1-1.png)

![解決方案2](./media/log-analytics-vmware/solutionview1-2.png)

按一下任何刀鋒視窗以開啟 Log Analytics 搜尋窗格，窗格中會顯示該刀鋒視窗的詳細資訊。

您可以在此編輯搜尋查詢，修改查核來尋找特定目標。 如需有關建立記錄搜尋的詳細資料，請參閱[在 Log Analytics 中使用記錄搜尋以尋找資料](log-analytics-log-searches.md)。

#### <a name="find-esxi-host-events"></a>尋找 ESXi 主機事件
單一 ESXi 主機會產生多個記錄檔，取決於其程序。 VMware 監視解決方案會將它們集中在一起，並總結事件計數。 這個集中式的檢視可幫助您了解哪些 ESXi 主機有大量的事件，以及在您的環境中最常發生哪些事件。

![事件](./media/log-analytics-vmware/events.png)

您可以按一下 ESXi 主機或事件類型，進一步深入探詢。

當您按一下的 ESXi 主機名稱時，可以檢視該 ESXi 主機的資訊。 如果想要限縮事件類型的結果，在搜尋查詢中新增 `“ProcessName_s=EVENT TYPE”`。 您可以在搜尋篩選中選取 **ProcessName**。 這會為您限縮資訊。

![深入探詢](./media/log-analytics-vmware/eventhostdrilldown.png)

#### <a name="find-high-vm-activities"></a>尋找高 VM 活動
在任何 ESXi 主機上皆可以建立及刪除虛擬機器。 能識別 ESXi 主機建立多少個 VM，對系統管理員很有幫助， 進而幫助了解效能和容量規劃。 管理您的環境時，持續追蹤 VM 活動事件極為重要。

![深入探詢](./media/log-analytics-vmware/vmactivities1.png)

如果您想要查看其他 ESXi 主機 VM 建立資料，按一下 ESXi 主機名稱。

![深入探詢](./media/log-analytics-vmware/createvm.png)

#### <a name="common-search-queries"></a>常見的搜尋查詢
這個解決方案包含其他實用的查詢，可協助您管理您的 ESXi 主機，例如高儲存量空間、儲存體延遲、路徑失敗。

![查詢](./media/log-analytics-vmware/queries.png)


#### <a name="save-queries"></a>儲存查詢
儲存搜尋查詢是 Log Analytics 中的標準功能，可協助您保留任何您認為有用的查詢。 建立您覺得有用的查詢之後，按一下 [我的最愛] 儲存它。 儲存的查詢讓您之後可從 [我的儀表板](log-analytics-dashboards.md) 頁面輕鬆地重複使用它們，您也可以在此建立您自己自訂的儀表板。

![DockerDashboardView](./media/log-analytics-vmware/dockerdashboardview.png)

#### <a name="create-alerts-from-queries"></a>從查詢建立警示
建立您的查詢後，您可能想要使用該查詢在特定事件發生時發出警示。 如需有關如何建立警示的資訊，請參閱 [Log Analytics 中的警示](log-analytics-alerts.md)。 如需警示查詢和其他查詢的範例，請參閱部落格文章[使用 Log Analytics 監視 VMware](https://blogs.technet.microsoft.com/msoms/2016/06/15/monitor-vmware-using-oms-log-analytics)。

## <a name="frequently-asked-questions"></a>常見問題集
### <a name="what-do-i-need-to-do-on-the-esxi-host-setting-what-impact-will-it-have-on-my-current-environment"></a>我需要在 ESXi 主機設定上做什麼設定？ 它會對我目前的環境造成什麼影響？
解決方案會使用原生 ESXi 主機 Syslog 轉送機制。 您在 ESXi 主機上不需要任何額外的 Microsoft 軟體就可以擷取記錄檔。 它對您現有的環境影響不大。 但是，您需要設定 syslog 轉送，這是 ESXI 功能。

### <a name="do-i-need-to-restart-my-esxi-host"></a>我需要重新啟動 ESXi 主機嗎？
否。 此處理序不需要重新啟動。 有時候，vSphere 不會正確更新 syslog。 在這種情況下，請登入 ESXi 主機並重新載入 syslog。 同樣地，您不需要重新啟動主機，所以此處理序不會干擾到您的環境。

### <a name="can-i-increase-or-decrease-the-volume-of-log-data-sent-to-log-analytics"></a>可以增加或減少傳送至 Log Analytics 的記錄資料量嗎？
是，您可以這麼做。 您可以使用 vSphere 中的 ESXi 主機記錄層級設定。 記錄集合是以 *info* 層級為基礎。 所以，如果您想要稽核 VM 建立或刪除，您需要在 Hostd 上維持 *info* 層級。 如需詳細資訊，請參閱 [VMware 知識庫](https://kb.vmware.com/selfservice/microsites/search.do?&cmd=displayKC&externalId=1017658)。

### <a name="why-is-hostd-not-providing-data-to-log-analytics-my-log-setting-is-set-to-info"></a>為什麼 Hostd 沒有將資料提供給 Log Analytics？ 我的記錄檔設定是設為 info。
syslog 時間戳記有一個 ESXi 主機錯誤。 如需詳細資訊，請參閱 [VMware 知識庫](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2111202)。 在您套用因應措施之後，Hostd 應該就能正常運作。

### <a name="can-i-have-multiple-esxi-hosts-forwarding-syslog-data-to-a-single-vm-with-omsagent"></a>我可以使用 OMS 代理程式讓多部 ESXi 主機轉送 syslog 資料至單一 VM 嗎？
是。 您可以使用 OMS 代理程式讓多部 ESXi 主機轉送資料至單一 VM。

### <a name="why-dont-i-see-data-flowing-into-log-analytics"></a>為什麼我沒有看到資料流入 Log Analytics？
這有幾個原因：

* ESXi 主機目前沒有推播資料至執行 OMS 代理程式的 VM。 若要測試，請執行下列步驟：

  1. 若要確認，請使用 SSH 登入 ESXi 主機並執行以下命令：`nc -z ipaddressofVM 1514`

      如果這沒有成功，表示進階組態中的 vSphere 設定可能不正確。 請參閱[設定 syslog 集合](#configure-syslog-collection)，以瞭解如何設定 ESXi 主機來進行 syslog 轉送的相關資訊。
  1. 如果 syslog 連接埠連線成功，但您還是沒有看到任何資料，請使用 ssh 並執行以下命令來於 ESXi 主機上重新載入 syslog：` esxcli system syslog reload`
* 未正確設定具有 OMS 代理程式的 VM。 若要測試，請執行下列步驟：

  1. Log Analytics 會接聽連接埠 1514。 若要確認它是否已經開啟，請執行以下命令：`netstat -a | grep 1514`
  1. 您應該會看到連接埠 `1514/tcp` 已開啟。 如果沒有，請確認是否已正確安裝 OMS 代理程式。 如果您沒有看到連接埠資訊，表示 VM 上沒有開啟 syslog 連接埠。

    a. 請使用 `ps -ef | grep oms` 確認 OMS 代理程式是否在執行中。 如果它沒有執行，請執行命令 ` sudo /opt/microsoft/omsagent/bin/service_control start`

    b. 開啟 `/etc/opt/microsoft/omsagent/conf/omsagent.d/vmware_esxi.conf` 檔案。

    c. 確認適當的使用者和群組設定有效，類似於：`-rw-r--r-- 1 omsagent omiusers 677 Sep 20 16:46 vmware_esxi.conf`

    d. 如果檔案不存在或使用者和群組設定錯誤，請透過[準備 Linux 伺服器](#prepare-a-linux-server)來採取更正動作。

## <a name="next-steps"></a>後續步驟
* 使用 Log Analytics 中的 [Log Analytics](log-analytics-log-searches.md) 檢視詳細的 VMware 主機資料。
* [建立您自己的儀表板](log-analytics-dashboards.md)來顯示 VMware 主機的資料。
* 在特定的 VMware 主機事件發生時[建立警示](log-analytics-alerts.md)。
