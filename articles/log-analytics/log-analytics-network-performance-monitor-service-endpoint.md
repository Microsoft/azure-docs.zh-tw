---
title: Azure Log Analytics 中的網路效能監視器方案 | Microsoft Docs
description: 使用網路效能監控中的服務連線能力監視功能，對任何具有已開啟 TCP 連接埠的端點進行網路連線監視。
services: log-analytics
documentationcenter: ''
author: abshamsft
manager: carmonm
editor: ''
ms.assetid: 5b9c9c83-3435-488c-b4f6-7653003ae18a
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 02/20/2018
ms.author: abshamsft
ms.component: na
ms.openlocfilehash: 3c9352e8e4aee7817b1195c15f74503e86e597ea
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37434914"
---
# <a name="service-connectivity-monitor"></a>服務連線能力監視

您可以使用[網路效能監控](log-analytics-network-performance-monitor.md)中的服務連線能力監視功能，對任何具有已開啟 TCP 連接埠的端點進行網路連線監視。 這類端點包括網站、SaaS 應用程式、PaaS 應用程式和 SQL 資料庫。 

您可以使用服務連線能力監視來執行下列功能： 

- 監視多個分公司或位置與您的應用程式和網路服務的網路連線。 應用程式和網路服務包括 Office 365、Dynamics CRM、內部企業營運系統應用程式及 SQL 資料庫。
- 使用內建測試來監視 Office365 和 Dynamics365 端點的網路連線。 
- 判斷連線至端點時的回應時間、網路延遲及封包遺失。
- 判斷應用程式效能低落是因為網路還是因為應用程式提供者端的某些問題所造成。
- 檢視拓撲地圖上每個躍點所提供的延遲，識別可能導致應用程式效能低落的作用點。


![服務連線能力監視](media/log-analytics-network-performance-monitor/service-endpoint-intro.png)


## <a name="configuration"></a>組態 
若要開啟網路效能監控的組態，請開啟[網路效能監控解決方案](log-analytics-network-performance-monitor.md)，然後選取 [設定]。

![設定網路效能監控](media/log-analytics-network-performance-monitor/npm-configure-button.png)


### <a name="configure-operations-management-suite-agents-for-monitoring"></a>設定用於監視的 Operations Management Suite 代理程式
在用於監視的節點上啟用下列防火牆規則，讓解決方案可以探索從您的節點到服務端點的拓撲： 

```
netsh advfirewall firewall add rule name="NPMDICMPV4Echo" protocol="icmpv4:8,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV6Echo" protocol="icmpv6:128,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV4DestinationUnreachable" protocol="icmpv4:3,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV6DestinationUnreachable" protocol="icmpv6:1,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV4TimeExceeded" protocol="icmpv4:11,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV6TimeExceeded" protocol="icmpv6:3,any" dir=in action=allow 
```

### <a name="create-service-connectivity-monitor-tests"></a>建立服務連線能力監視測試 

開始建立您的測試，來監視服務端點的網路連線。

1. 選取 [服務連線能力監視] 索引標籤。
2. 選取 [新增測試] 並輸入測試名稱和描述。 
3. 選取測試類型：<br>

    * 選取 [Web] 以監視回應 HTTP/S 要求之服務的連線，例如 outlook.office365.com 或 bing.com。<br>
    * 選取 [網路] 以監視回應 TCP 要求但不會回應 HTTP/S 要求之服務的連線，例如 SQL 伺服器、FTP 伺服器或 SSH 連接埠。 
4. 如果您不想要執行網路量測，例如網路延遲、封包遺失和拓撲探索，請清除 [執行網路量測] 核取方塊。 保持選取狀態可充分發揮功能。 
5. 在 [目標] 中，輸入您要監視網路連線的 URL/FQDN/IP 位址。
6. 在 [連接埠號碼] 中，輸入目標服務的連接埠號碼。 
7. 在 [測試頻率] 中，輸入您想要執行測試的頻率值。 
8. 選取您要監視服務網路連線的來源節點。 

    >[!NOTE]
    > 對於以 Windows Server 為基礎的節點，功能會使用以 TCP 為基礎的要求來執行網路測量。 對於以 Windows 用戶端為基礎的節點，功能會使用以 ICMP 為基礎的要求來執行網路測量。 在某些情況下，目標應用程式會在節點是以 Windows 用戶端為基礎時，封鎖以連入 ICMP 為基礎的要求。 此解決方案無法執行網路量測。 建議您在此情況下使用以 Windows Server 為基礎的節點。 

9. 如果您不想要建立所選項目的健康情況事件，則清除 [在此測試所涵蓋的目標上啟用健康情況監視]。 
10. 選擇監視條件。 您可以輸入閾值，以設定健康情況事件產生的自訂閾值。 只要條件的值高於針對所選網路或子網路配對選取的閾值時，就會產生健康情況事件。 
11. 選取 [儲存] 以儲存組態。 

    ![服務連線能力監視測試組態](media/log-analytics-network-performance-monitor/service-endpoint-configuration.png)



## <a name="walkthrough"></a>逐步介紹 

移至 [網路效能監控] 儀表板檢視。 若要對您所建立的不同測試取得健康情況摘要，請查看 [服務連線能力監視] 頁面。 

![服務連線能力監視頁面](media/log-analytics-network-performance-monitor/service-endpoint-blade.png)

選取圖格以檢視 [測試] 頁面上的測試詳細資料。 在左側資料表中，您可以檢視所有測試的時間點健康情況和服務回應時間的值、網路延遲及封包遺失。 使用 [網路狀態錄製器] 控制項，檢視過去另一個時間的網路快照集。 在資料表中選取您要調查的測試。 您可以在右側窗格的圖表中，檢視遺失、延遲和回應時間值的歷史趨勢。 選取 [測試詳細資料] 連結，即可檢視每個節點的效能。

![服務連線能力監視測試](media/log-analytics-network-performance-monitor/service-endpoint-tests.png)

在 [測試節點] 檢視中，您可以觀察來自每個節點的網路連線。 選取效能降低的節點。 這是能觀察到應用程式執行速度緩慢的節點。

若要判斷應用程式效能低落是因為網路還是因為應用程式提供者端的某些問題所造成，可觀察應用程式回應時間與網路延遲之間的相互關聯。 

* **應用程式問題：** 如果回應時間突然增加，但網路延遲是一致的，則表示網路運作正常，而問題可能是應用程式端的問題所致。 

    ![服務連線能力監視應用程式問題](media/log-analytics-network-performance-monitor/service-endpoint-application-issue.png)

* **網路問題：** 如果回應時間尖峰與對應的網路延遲尖峰同時出現，則表示回應時間增加可能是網路延遲的增加所致。 

    ![服務連線能力監視網路問題](media/log-analytics-network-performance-monitor/service-endpoint-network-issue.png)

在您判定問題是因為網路之後，選取 [拓撲] 檢視連結，以識別拓撲地圖上的問題躍點。 範例如下圖所示。 節點與應用程式端點之間的 105 毫秒延遲總計之中，有 96 毫秒是因為躍點標示為紅色。 識別出問題躍點之後，您可以採取矯正措施。 

![服務連線能力監視測試](media/log-analytics-network-performance-monitor/service-endpoint-topology.png)

## <a name="diagnostics"></a>診斷 

如果您觀察到異常，請依照下列步驟：

* 如果服務回應時間、網路遺失和延遲顯示為 NA，可能是因為下列一個或多個原因：

    - 應用程式已關閉。
    - 用於檢查服務之網路連線的節點已關閉。
    - 測試組態中輸入的目標不正確。
    - 節點沒有任何網路連線。

* 如果顯示有效的服務回應時間，但網路遺失和延遲顯示為 NA，可能是因為下列一個或多個原因：

    - 如果用於檢查服務之網路連線的節點為 Windows 用戶端機器，則目標服務會封鎖 ICMP 要求，或是網路防火牆會封鎖源自該節點的 ICMP 要求。
    - [執行網路量測] 核取方塊在測試組態中為空白。 

* 如果服務回應時間為 NA，但是網路遺失和延遲都是有效的，則目標服務可能並非 Web 應用程式。 編輯測試組態，然後選擇 [網路]而不是 [Web] 作為測試類型。 

* 如果應用程式執行緩慢，請判斷應用程式效能低落是因為網路還是應用程式提供者端的某些問題所造成。


## <a name="next-steps"></a>後續步驟
[搜尋記錄檔](log-analytics-log-searches.md)以檢視詳細的網路效能資料記錄。
