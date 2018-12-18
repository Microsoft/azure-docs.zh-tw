---
title: Azure Application Insights 遙測資料模型 - 遙測內容 | Microsoft Docs
description: Application Insights 遙測內容資料模型
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: TBD
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 05/15/2017
ms.reviewer: sergkanz
ms.author: mbullwin
ms.openlocfilehash: b6cfae20f09b19a57cf411777e78abb1dbbf0484
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35633466"
---
# <a name="telemetry-context-application-insights-data-model"></a>遙測內容：Application Insights 資料模型

每個遙測項目都可能具有強型別的內容欄位。 每個欄位都具有特定的監視案例。 請使用自訂屬性集合來儲存自訂或應用程式特定的內容資訊。


## <a name="application-version"></a>應用程式版本

應用程式內容欄位中一律是與傳送遙測之應用程式相關的資訊。 應用程式版本可用於分析應用程式行為及其與部署相互關聯中的趨勢變更。

最大長度：1024


## <a name="client-ip-address"></a>用戶端 IP 位址

用戶端裝置的 IP 位址。 支援 IPv4 和 IPv6。 從服務傳送遙測時，位置內容與服務中起始作業的使用者相關。 Application Insights 會從用戶端 IP 擷取地理位置資訊並予以截斷。 因此用戶端 IP 本身無法作為終端使用者識別資訊。 

最大長度：46


## <a name="device-type"></a>裝置類型

原本此欄位是用於表示應用程式終端使用者正在使用的裝置類型。 目前主要用於區別裝置類型為「瀏覽器」的 JavaScript 遙測與裝置類型為「電腦」的伺服器端遙測。

最大長度：64


## <a name="operation-id"></a>作業 id

根作業的唯一識別碼。 此識別碼可讓群組遙測跨越多個元件。 如需詳細資訊，請參閱[遙測相互關聯](application-insights-correlation.md)。 作業識別碼會由要求或頁面檢視所建立。 所有其他遙測會將此欄位設定為包含要求或頁面檢視的值。 

最大長度：128


## <a name="parent-operation-id"></a>父作業識別碼

遙測項目直屬父代的唯一識別碼。 如需詳細資訊，請參閱[遙測相互關聯](application-insights-correlation.md)。

最大長度：128


## <a name="operation-name"></a>作業名稱

作業的名稱 (群組)。 作業名稱會由要求或頁面檢視所建立。 所有其他遙測項目會將此欄位設定為包含要求或頁面檢視的值。 作業名稱可用於尋找作業群組的所有遙測項目 (例如 'GET Home/Index')。 此內容屬性可用於回答「此頁面擲回的一般例外狀況為何」等問題。

最大長度：1024


## <a name="synthetic-source-of-the-operation"></a>作業的綜合來源

綜合來源的名稱。 來自可能代表綜合流量之應用程式的一些遙測。 其可能是來自 Application Insights SDK 本身等診斷程式庫之編製網站索引的網頁編目程式、網站可用性測試或追蹤。

最大長度：1024


## <a name="session-id"></a>工作階段識別碼

工作階段識別碼 - 與應用程式進行使用者互動的執行個體。 工作階段內容欄位中一律是與終端使用者相關的資訊。 從服務傳送遙測時，工作階段內容與服務中起始作業的使用者相關。

最大長度：64


## <a name="anonymous-user-id"></a>匿名使用者識別碼

匿名使用者識別碼。代表應用程式的終端使用者。 從服務傳送遙測時，使用者內容與服務中起始作業的使用者相關。

[取樣](app-insights-sampling.md)是將收集的遙測量降到最低的其中一個方法。 取樣演算法會嘗試取樣輸入或輸出所有相互關聯的遙測。 匿名使用者識別碼可用於產生取樣分數。 因此匿名使用者識別碼應該為足夠隨機的值。 

使用匿名使用者識別碼儲存使用者名稱是濫用欄位的做法。 使用已驗證的使用者識別碼。

最大長度：128


## <a name="authenticated-user-id"></a>已驗證的使用者識別碼

已驗證的使用者識別碼。與匿名使用者識別碼相反，此欄位代表具有易記名稱的使用者。 由於其 PII 資訊，根據預設，大部分的 SDK 都不會收集該資訊。

最大長度：1024


## <a name="account-id"></a>帳戶識別碼

在多租用戶應用程式中，此為使用者使用的帳戶識別碼或名稱。 範例包括 Azure 入口網站的訂用帳戶識別碼或部落格平台的部落格名稱。

最大長度：1024


## <a name="cloud-role"></a>雲端角色

應用程式角色的名稱是其中的一部分。 在 Azure 中直接對應到角色名稱。 您也可以將其用於區別微服務，其為單一應用程式的一部分。

最大長度：256


## <a name="cloud-role-instance"></a>雲端角色執行個體

執行應用程式的執行個體名稱。 內部部署的電腦名稱、Azure 的執行個體名稱。

最大長度：256


## <a name="internal-sdk-version"></a>內部：SDK 版本

SDK 版本。 如需相關資訊，請參閱 https://github.com/Microsoft/ApplicationInsights-Home/blob/master/SDK-AUTHORING.md#sdk-version-specification。

最大長度：64


## <a name="internal-node-name"></a>內部：節點名稱

此欄位代表用於計費的節點名稱。 您可以將其用於覆寫節點的標準偵測。

最大長度：256


## <a name="next-steps"></a>後續步驟

- 了解如何[擴充和篩選遙測](app-insights-api-filtering-sampling.md)。
- 如需 Application Insights 類型和資料模型，請參閱[資料模型](application-insights-data-model.md)。
- 請查看標準內容屬性集合[組態](app-insights-configuration-with-applicationinsights-config.md#telemetry-initializers-aspnet)。
