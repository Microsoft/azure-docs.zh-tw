---
title: Azure 媒體服務串流端點概觀 | Microsoft Docs
description: 本主題提供 Azure 媒體服務串流端點的概觀。
services: media-services
documentationcenter: ''
author: Juliako
writer: juliako
manager: cfowler
editor: ''
ms.assetid: 097ab5e5-24e1-4e8e-b112-be74172c2701
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/29/2017
ms.author: juliako
ms.openlocfilehash: 7b4077747d4c9ec581496786063fc8fbe0c6b247
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785947"
---
# <a name="streaming-endpoints-overview"></a>串流端點概觀 

## <a name="overview"></a>概觀

在「Microsoft Azure 媒體服務」(AMS) 中，**串流端點**代表可以直接將內容傳遞給用戶端播放程式應用程式，或傳遞給「內容傳遞網路」(CDN) 以進行進一步散發的串流服務。 媒體服務也提供順暢的 Azure CDN 整合。 來自 StreamingEndpoint 服務的輸出資料流可以是即時資料流、隨選視訊，也可以是媒體服務帳戶中漸進式的資產下載。 每個「Azure 媒體服務」帳戶皆包含一個預設的 StreamingEndpoint。 您可以在該帳戶下建立額外的 StreamingEndpoint。 StreamingEndpoint 有 1.0 和 2.0 兩個版本。 從 2017 年 1 月 10 日開始，所有新建立的 AMS 帳戶都會包含 2.0 版**預設** StreamingEndpoint。 您新增到此帳戶的額外串流端點也將會是 2.0 版。 這項變更不會影響現有的帳戶，現有的 StreamingEndpoint 會是 1.0 版並可升級到 2.0 版。 隨著這項變更，將會有行為、計費及功能變更 (如需詳細資訊，請參閱下列**串流類型和版本**一節)。

此外，從 2.15 版開始 (2017 年 1 月發行)，Azure 媒體服務將下列屬性新增至串流端點實體︰**CdnProvider**、**CdnProfile**、**FreeTrialEndTime**、**StreamingEndpointVersion**。 如需這些屬性的詳細概觀，請參閱[這裡](https://docs.microsoft.com/rest/api/media/operations/streamingendpoint)。 

當您建立 Azure 媒體服務帳戶時，系統會為您建立預設標準串流端點 (處於**已停止**狀態)。 您無法刪除預設串流端點。 根據目標區域中的 Azure CDN 可用性而定，依預設，新建立的預設串流端點也包含 "StandardVerizon" CDN 提供者整合。 

>[!NOTE]
>啟動串流端點之前可以停用 Azure CDN 整合。

本主題簡化串流端點提供的主要功能。

## <a name="streaming-types-and-versions"></a>串流類型和版本

### <a name="standardpremium-types-version-20"></a>標準/進階類型 (2.0 版)

從媒體服務的 2017 年 1 月版本開始，您有兩種串流類型︰**標準**和**進階**。 這些類型是串流端點 "2.0" 版的一部分。

類型|說明
---|---
**標準**|這是適用於大部分情況的預設選項。<br/>此選項可讓您取得固定/有限 SLA，在啟動串流端點之後，前 15 天免費。<br/>如果您建立多個串流端點，前 15 天只有第一個是免費，其他則一啟動就開始計費。 <br/>請注意，免費試用版僅適用於新建立的媒體服務帳戶和預設串流端點。 現有的串流端點和額外建立的串流端點即使升級至 2.0 版，或建立為 2.0 版，也不含免費試用期。
**高級**|這個選項適用於需要更大規模或控制的專業案例。<br/>以購買的進階串流單位 (SU) 容量為基礎的變動 SLA，專用的串流端點位於隔離的環境，不會競用資源。

如需詳細資訊，請參閱下列**比較串流類型**一節。

### <a name="classic-type-version-10"></a>傳統類型 (1.0 版)

對於在 2017 年 1 月 10 日之前建立 AMS 帳戶的使用者，您具有**傳統**類型的串流端點。 這個類型是串流端點 "1.0" 版的一部分。

如果您的 **"1.0" 版**串流端點有 >= 1 個進階串流單位 (SU)，則會是進階串流端點，且提供所有 AMS 功能 (就像**標準/進階**類型)，不需要任何額外的設定步驟。

>[!NOTE]
>**傳統**串流端點 ("1.0" 版和 0 SU) 提供有限的功能，且不包含 SLA。 建議移轉至**標準**類型，以獲得更好的體驗和使用動態封裝或加密等功能，以及**標準**類型隨附的其他功能。 若要移轉至**標準**類型，請移至 [Azure 入口網站](https://portal.azure.com/)，然後選取 [選擇標準]。 如需移轉的詳細資訊，請參閱[移轉文件](#migration-between-types)一節。
>
>請注意，這項作業無法回復，而且會影響價格。
>
 
## <a name="comparing-streaming-types"></a>比較串流類型

### <a name="versions"></a>版本

|類型|StreamingEndpointVersion|ScaleUnits|CDN|計費|SLA| 
|--------------|----------|-----------------|-----------------|-----------------|-----------------|    
|傳統|1.0|0|NA|免費|NA|
|標準串流端點|2.0|0|yes|付費|yes|
|進階串流單位|1.0|>0|yes|付費|yes|
|進階串流單位|2.0|>0|yes|付費|yes|

### <a name="features"></a>特性

功能|標準|進階
---|---|---
前 15 天免費| yes |否
Throughput |未使用 Azure CDN 時最多 600 Mbps。 隨著 CDN 調整。|每個串流單位 (SU) 200 Mbps。 隨著 CDN 調整。
SLA | 99.9|99.9 (每個 SU 200 Mbps)。
CDN|Azure CDN、協力廠商 CDN 或沒有 CDN。|Azure CDN、協力廠商 CDN 或沒有 CDN。
按比例計費| 每日|每日
動態加密|yes|yes
動態封裝|yes|yes
調整|自動相應增加至目標輸送量。|其他串流單位
IP 篩選/G20/自訂主機|yes|yes
漸進式下載|yes|yes
建議用法 |建議用於絕大多數的串流案例。|專業用法。<br/>如果認為您的需求已超過「標準」。 如果您預期有 50,000 位以上的觀眾同時觀看，請連絡我們 (amsstreaming@microsoft.com)。


## <a name="migration-between-types"></a>在類型之間移轉

從 | 至 | 動作
---|---|---
傳統|標準|需要選擇加入
傳統|進階| 調整 (其他串流單位)
標準/高階|傳統|無法使用 (如果串流端點版本為 1.0。 允許變更為傳統並將 scaleunits 設為 "0")
標準 (含/不含 CDN)|進階搭配相同的設定|在**已啟動**狀態下允許。 (透過 Azure 入口網站)
進階 (含/不含 CDN)|標準搭配相同的設定|在**已啟動**狀態下允許 (透過 Azure 入口網站)
標準 (含/不含 CDN)|進階搭配不同的設定|在**已停止**狀態下允許 (透過 Azure 入口網站)。 在執行中狀態下不允許。
進階 (含/不含 CDN)|標準搭配不同的設定|在**已停止**狀態下允許 (透過 Azure 入口網站)。 在執行中狀態下不允許。
1.0 版，SU >= 1，含 CDN|標準/進階，不含 CDN|在**已停止**狀態下允許。 在**已啟動**狀態下不允許。
1.0 版，SU >= 1，含 CDN|標準，含/不含 CDN|在**已停止**狀態下允許。 在**已啟動**狀態下不允許。 將會刪除 1.0 版 CDN 並建立和啟動新的 CDN。
1.0 版，SU >= 1，含 CDN|進階，含/不含 CDN|在**已停止**狀態下允許。 在**已啟動**狀態下不允許。 將會刪除傳統 CDN 並建立和啟動新的 CDN。

## <a name="next-steps"></a>後續步驟
檢閱媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

