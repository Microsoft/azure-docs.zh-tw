---
title: 了解 Azure 時間序列深入解析環境中的資料保留 | Microsoft Docs
description: 本文說明「Azure 時間序列深入解析」環境中，控制資料保留的兩個設定。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: jasonh, kfile, anshan
ms.workload: big-data
ms.topic: conceptual
ms.date: 02/09/2018
ms.openlocfilehash: e265a66b841530d1133d760ebdcdf56046d1aee1
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364099"
---
# <a name="understand-data-retention-in-time-series-insights"></a>了解時間序列深入解析中的資料保留

本文說明時間序列深入解析 (TSI) 環境中，影響資料保留的兩個設定。

## <a name="video"></a>視訊： 

### <a name="in-this-video-we-cover-time-series-insights-data-retention-and-how-to-plan-for-itbr"></a>在本影片中，我們將說明時間序列深入解析資料保留以及如何進行規劃。</br>

> [!VIDEO https://www.youtube.com/embed/03x6zKDQ6DU]

每個 TSI 環境都有可控制**資料保留時間**的設定。 可設定的值為 1 到 400 天。 資料會依環境儲存容量或保留持續時間 (1-400) 進行刪除，以先達到者為準。

每個 TSI 環境都有一個額外的設定 [超過儲存空間限制的行為]。 此設定可控制當達到環境的容量上限時，要執行的輸入與清除行為。 有兩個行為可供選擇：
- **清除舊資料** (預設)  
- **暫停輸入**

> [!NOTE]
> 根據預設，建立新環境時，保留期會設定為**清除舊資料**。 這個設定可以在建立後視需要使用 Azure 入口網站，在 TSI 環境的 [設定] 頁面上切換。

如需切換保留行為的相關資訊，請檢閱[在時間序列深入解析中設定保留](time-series-insights-how-to-configure-retention.md)。

比較資料保留行為：

## <a name="purge-old-data"></a>清除舊資料
- 此行為是 TSI 環境的預設行為，而且會表現出和 TSI 環境相同的行為，因為它會在公開預覽中啟動。  
- 當使用者想要在其 TSI 環境中一律看到「最新的資料」時，建議使用此行為。 
- 在達到環境限制 (保留時間、大小或計數，以先達到者為準) 時，此行為就會「清除」資料。 保留期預設值為 30 天。 
- 最舊的內嵌資料會先清除 (FIFO 方法)。

### <a name="example-1"></a>範例 1：
參考保留行為設為 [繼續輸入並清除舊資料] 的範例環境：在這個範例中，[資料保留時間] 是設定為 400 天。 [容量] 是設定為 S1 單位，其中包含 30 GB 的總容量。   假設傳入的資料平均每天會累積至 500 MB。 在給定的資料傳入速率下，此環境只能將資料保留 60 天，因為會在 60 天時達到容量上限。 傳入資料會累積為：500 MB 每天 x 60 天 = 30 GB。 

在這個範例中，在第 61 天時，環境會顯示最新的資料，但會清除 60 天以前的最舊資料。 清除動作可釋出空間供傳入的新資料使用，因此可以繼續瀏覽新資料。 

如果使用者希望將資料保留更久一點，可以透過增加額外單位容量以提高環境的資料容納量，或減少傳入的資料量。  

### <a name="example-2"></a>範例 2：
考慮也設定 [繼續輸入並清除舊資料] 保留行為的環境。 在這個範例中，[資料保留時間] 是設定為較低的值：180 天。 [容量] 是設定為 S1 單位，其中包含 30 GB 的總容量。 為將資料儲存 180 天，每日輸入量不能超過 0.166 GB (166 MB)。  

只要此環境的每日輸入速率超過 0.166 GB，資料就無法儲存 180 天，因為某些資料會被清除。 請考慮此相同環境在忙碌的時間範圍內會發生什麼狀況。 假設環境的資料輸入速率可能會增加到每日平均 0.189 GB。 在該忙碌時間範圍內，約可保留 158 天的資料 (30 GB/0.189 = 158.73 保留天數)。 此時間短於所需的資料保留時間範圍。

## <a name="pause-ingress"></a>暫停輸入
- 此行為的設計可確保在達到保留期間之前先達到大小和計數限制時，不會清除資料。  
- 此行為可為使用者提供額外的緩衝時間，方便使用者在資料因為達到保留期間而被刪除前，增加環境的容量。
- 此行為有助於防止舊資料遺失，但如果在達到事件來源保留期間之前暫停輸入，可能會造成您遺失最新的資料。
- 但是，一旦達到環境的最大容量，環境就會暫停資料輸入，直到採取以下額外動作為止： 
   - 您提高了環境最大容量。 如需詳細資訊，請參閱[如何調整您的時間序列深入解析環境規模](time-series-insights-how-to-scale-your-environment.md)以增加更多縮放單位。
   - 已達到資料保留期間且已清除資料，因此讓環境容量使用量降低到容量上限以下。

### <a name="example-3"></a>範例 3：
考慮保留期行為設定為**暫停輸入**的環境。 在此範例中，[資料保留期間] 設定為 60 天。 [容量] 設定為 S1 的 3 個單位。 假設此環境每天輸入 2 GB 資料。 在此環境中，一旦達到最大容量就會暫停輸入。 屆時，環境會顯示相同的資料集，直到恢復輸入或啟用 [繼續輸入] \(這會清除較舊的資料，以騰出空間給新的資料使用\) 為止。 

恢復繼續輸入時：
- 資料會依事件來源收到資料的順序輸入
- 系統會依據事件的時間戳記為事件編製索引，除非您已超出事件來源的保留原則。 如需有關事件來源保留設定的詳細資訊，請參閱[事件中樞常見問題集](../event-hubs/event-hubs-faq.md)

> [!IMPORTANT]
> 您應該設定警示，以提供可協助避免暫停輸入資料的通知。 因為 Azure 事件來源的保留期預設為 1 天，所以是有可能會遺失資料的。 因此，輸入暫停之後，除非您採取其他動作，否則就可能會遺失最新資料。 您必須增加容量，或將行為切換為 [清除舊資料]，以避免發生資料遺失。

當事件中樞受到影響時，請考慮調整 [訊息保留期] 屬性，以在「時間序列深入解析」中發生暫停輸入時，將資料遺失機率降到最低。

![事件中樞訊息保留。](media/time-series-insights-contepts-retention/event-hub-retention.png)

如果事件來源 (timeStampPropertyName) 中沒有設定屬性，TSI 預設將抵達事件中樞時的時間戳記設為 X 軸。 如果將 timeStampPropertyName 設為其他設定，環境會在事件暫停時，在資料封包中尋找設定的 timeStampPropertyName。 

如果您需要相應增加環境以容納更多容量，或需要延長保留長度，請參閱[如何調整您的時間序列深入解析環境規模](time-series-insights-how-to-scale-your-environment.md)以了解詳細資訊。  

## <a name="next-steps"></a>後續步驟
如需切換保留行為的相關資訊，請檢閱[在時間序列深入解析中設定保留](time-series-insights-how-to-configure-retention.md)。
