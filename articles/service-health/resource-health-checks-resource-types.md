---
title: 透過 Azure 資源健康狀態支援的資源類型 | Microsoft Docs
description: 透過 Azure 資源健康狀態支援的資源類型
services: Resource health
documentationcenter: ''
author: BernardoAMunoz
manager: ''
editor: ''
ms.assetid: 85cc88a4-80fd-4b9b-a30a-34ff3782855f
ms.service: service-health
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: Supportability
ms.date: 10/09/2017
ms.author: BernardoAMunoz
ms.openlocfilehash: 633501b03a362fbd07e14e308e18488f03af5af5
ms.sourcegitcommit: 0b05bdeb22a06c91823bd1933ac65b2e0c2d6553
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39069194"
---
# <a name="resource-types-and-health-checks-in-azure-resource-health"></a>Azure 資源健康狀態中的資源類型和健康情況檢查
以下是依資源類型透過資源健康狀態執行之所有檢查的完整清單。

## <a name="microsoftanalysisservicesservers"></a>Microsoft.AnalysisServices/servers
|執行的檢查|
|---|
|<ul><li>伺服器是否已啟動且執行中？</li><li>伺服器是否已耗盡記憶體？</li><li>伺服器正在啟動嗎？</li><li>伺服器正在復原嗎？</li></ul>|

## <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service
|執行的檢查|
|---|
|<ul><li>Api 管理服務是否已啟動並執行？</li></ul>|

## <a name="microsoftcacheredisredis"></a>Microsoft.CacheRedis/Redis
|執行的檢查|
|---|
|<ul><li>所有快取節點是否已啟動且執行中？</li><li>可以從資料中心內觸達快取嗎？</li><li>快取是否已觸達連線數目上限？</li><li> 快取是否已用盡其可用的記憶體？ </li><li>快取是否發生大量頁面錯誤？</li><li>快取是否負載過重？</li></ul>|

## <a name="microsoftcdnprofile"></a>Microsoft.CDN/profile
|執行的檢查|
|---|
|<ul> <li>是否可存取補充入口網站進行 CDN 組態作業？</li><li>CDN 端點有進行中的傳遞問題嗎？</li><li>使用者可以將其 CDN 資源的組態進行變更嗎？</li><li>組態變更是否會以預期的速度進行傳播？</li><li>使用者可以使用 Azure 入口網站、PowerShell 或 API 管理 CDN 組態嗎？</li> </ul>|

## <a name="microsoftclassiccomputevirtualmachines"></a>Microsoft.classiccompute/virtualmachines
|執行的檢查|
|---|
|<ul><li>主機伺服器是否已啟動且執行中？</li><li>主機 OS 是否已開機完成？</li><li>虛擬機器容器是否已佈建和啟動？</li><li>主機和儲存體帳戶之間是否有網路連線能力？</li><li>客體 OS 是否已完成開機？</li><li>是否有持續性的規劃維護？</li></ul>|

## <a name="microsoftcognitiveservicesaccounts"></a>Microsoft.cognitiveservices/accounts
|執行的檢查|
|---|
|<ul><li>可以從資料中心內觸達帳戶嗎？</li><li>是否有可用的認知服務資源提供者？</li><li>適當區域中是否有可用的辨識服務？</li><li>可以在保存資源中繼資料的儲存體帳戶上執行讀取作業嗎？</li><li>是否已觸達 API 呼叫配額？</li><li>是否已觸達 API 呼叫讀取上限？</li></ul>|

## <a name="microsoftcomputevirtualmachines"></a>Microsoft.compute/virtualmachines
|執行的檢查|
|---|
|<ul><li>裝載此虛擬機器的伺服器是否已啟動且執行中？</li><li>主機 OS 是否已開機完成？</li><li>虛擬機器容器是否已佈建和啟動？</li><li>主機和儲存體帳戶之間是否有網路連線能力？</li><li>客體 OS 是否已完成開機？</li><li>是否有持續性的規劃維護？</li></ul>|

## <a name="microsoftdatalakeanalyticsaccounts"></a>Microsoft.datalakeanalytics/accounts
|執行的檢查|
|---|
|<ul><li>使用者在提交或列出其 Data Lake Analytics 作業時是否遇到問題？</li><li>Data Lake Analytics 作業是否因為系統錯誤而無法完成？</li></ul>|


## <a name="microsoftdatalakestoreaccounts"></a>Microsoft.datalakestore/accounts
|執行的檢查|
|---|
|<ul><li>使用者將資料上傳至 Data Lake Store 時是否遇到問題？</li><li>使用者從 Data Lake Store 下載資料時是否遇到問題？</li></ul>|

## <a name="microsoftdevicesiothubs"></a>Microsoft.devices/iothubs

|執行的檢查|
|---|
|<ul><li>IoT 中樞是否已啟動且正在執行中？</li></ul>|

## <a name="microsoftdocumentdbdatabaseaccounts"></a>Microsoft.documentdb/databaseAccounts
|執行的檢查|
|---|
|<ul><li>是否有因 Azure Cosmos DB 服務無法使用而不提供的任何資料庫或集合要求？</li><li>是否有因 Azure Cosmos DB 服務無法使用而不提供的任何文件要求？</li></ul>|

## <a name="microsoftkeyvaultvaults"></a>Microsoft.KeyVault/vaults
|執行的檢查|
|---|
|<ul><li>對金鑰保存庫執行的要求是否因為 Azure KeyVault 平台問題而失敗？</li><li>對金鑰保存庫執行的要求是否因為客戶發出太多要求而遭到節流？</li></ul>|

## <a name="microsoftnetworkconnections"></a>Microsoft.network/connections
|執行的檢查|
|---|
|<ul><li>VPN 通道是否已連線？</li><li>連線中有設定衝突嗎？</li><li>是否正確設定預先共用的金鑰？</li><li>是否可連線到 VPN 內部部署裝置？</li><li>IPSec/IKE 安全性原則中是否有不相符之處？</li><li>S2S VPN 連線是否正確佈建，還是處於失敗狀態？</li><li>VNET 對 VNET 連線是否正確佈建，還是處於失敗狀態？</li></ul>|

## <a name="microsoftnetworkvirtualnetworkgateways"></a>Microsoft.network/virtualNetworkGateways
|執行的檢查|
|---|
|<ul><li>是否可從網際網路觸達 VPN 閘道？</li><li>VPN 閘道是否處於待命模式？</li><li>VPN 服務是否在閘道上執行？</li></ul>|

## <a name="microsoftnotificationhubsnamespace"></a>Microsoft.NotificationHubs/namespace
|執行的檢查|
|---|
|<ul><li> 可以在命名空間上執行註冊、安裝或傳送等執行階段作業嗎？</li></ul>|

## <a name="microsoftpowerbiworkspacecollections"></a>Microsoft.PowerBI/workspaceCollections
|執行的檢查|
|---|
|<ul><li>主機作業系統是否已啟動且執行中？</li><li>是否可從資料中心外部觸達 workspaceCollection？</li><li>PowerBI 資源提供者是否可用？</li><li>適當區域中是否有可用的 PowerBI 服務？</li></ul>|

## <a name="microsoftsearchsearchservices"></a>Microsoft.search/searchServices
|執行的檢查|
|---|
|<ul><li>可以在叢集上執行診斷作業嗎？</li></ul>|

## <a name="microsoftsqlserverdatabase"></a>Microsoft.SQL/Server/database
|執行的檢查|
|---|
|<ul><li> 是否已經登入資料庫？</li></ul>|

## <a name="microsoftstoragestorageaccounts"></a>Microsoft.Storage/storageAccounts
|執行的檢查|
|---|
|<ul><li>從儲存體帳戶讀取資料的要求是否因為 Azure 儲存體的平台問題而失敗？</li><li>將資料寫入儲存體帳戶的要求是否因為 Azure 儲存體平台問題而失敗？</li><li>儲存體帳戶所在的儲存體叢集無法使用嗎？</li></ul>|

## <a name="microsoftstreamanalyticsstreamingjobs"></a>Microsoft.StreamAnalytics/streamingjobs
|執行的檢查|
|---|
|<ul><li>作業執行中的所有主機是否已啟動且執行中？</li><li>作業無法啟動嗎？</li><li>有進行中的執行階段升級嗎？</li><li>作業是否為預期的狀態 (例如執行中或由客戶停止)？</li><li>作業是否發生記憶體不足的例外狀況？</li><li>是否有進行中的排程計算更新？</li><li>執行管理員 (控制計畫) 是否可用？</li></ul>|

## <a name="microsoftwebserverfarms"></a>Microsoft.web/serverFarms
|執行的檢查|
|---|
|<ul><li>主機伺服器是否已啟動且執行中？</li><li>網際網路資訊服務是否執行中？</li><li>負載平衡器是否執行中？</li><li>可以從資料中心內觸達 App Service 方案嗎？</li><li>裝載 serverFarm 網站內容的儲存體帳戶是否可用？</li></ul>|

## <a name="microsoftwebsites"></a>Microsoft.web/sites
|執行的檢查|
|---|
|<ul><li>主機伺服器是否已啟動且執行中？</li><li>網際網路資訊服務是否執行中？</li><li>負載平衡器是否執行中？</li><li>可以從資料中心內觸達 Web 應用程式嗎？</li><li>裝載網站內容的儲存體帳戶是否可用？</li></ul>|

# <a name="next-steps"></a>後續步驟
-  請參閱 [Azure 服務健康狀態儀表板的簡介](service-health-overview.md)和 [Azure 資源健康狀態的簡介](resource-health-overview.md)來了解更多相關資訊。 
-  [關於 Azure 資源健康狀態的常見問題集](resource-health-faq.md)
- 設定警示，如此就能收到健康情況問題的通知。 如需詳細資訊，請參閱[設定適用於服務健康情況事件的警示](../monitoring-and-diagnostics/monitoring-activity-log-alerts-on-service-notifications.md)。 
