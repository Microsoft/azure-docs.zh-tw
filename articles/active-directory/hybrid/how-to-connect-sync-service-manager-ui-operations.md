---
title: Azure AD Connect Synchronization Service Manager 作業 | Microsoft Docs
description: 了解 Azure AD Connect 的 Synchronization Service Manager 中的 [作業] 索引標籤。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 97a26565-618f-4313-8711-5925eeb47cdc
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.component: hybrid
ms.author: billmath
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 92de95cb3192fb9f551bea631ef878070f9173cf
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310317"
---
# <a name="using-the-sync-service-manager-operations-tab"></a>使用 Sync Service Manager 作業索引標籤

![Sync Service Manager](./media/how-to-connect-sync-service-manager-ui-operations/operations.png)

[作業] 索引標籤顯示最新作業的結果。 此索引標籤主要是用來了解及疑難排解問題。

## <a name="understand-the-information-visible-in-the-operations-tab"></a>了解 [作業] 索引標籤中顯示的資訊
上半部會依時間順序顯示所有執行。 根據預設，作業記錄會保留最後 7 天的相關資訊，但是您可以利用 [排程器](how-to-connect-sync-feature-scheduler.md)來變更此設定。 若您想要尋找任何未顯示成功狀態的執行。 您可以按一下標頭來變更排序。

[狀態]  欄位是最重要的資訊，並顯示最嚴重的執行問題。 以下快速摘要依照優先順序來調查的最常見狀態 (其中 * 表示數個可能的錯誤字串)。

| 狀態 | 註解 |
| --- | --- |
| stopped-\* |執行無法完成。 例如，如果遠端系統已關閉且無法聯繫。 |
| stopped-error-limit |有 5,000 個以上的錯誤。 執行因錯誤數量過多而自動停止。 |
| completed-\*-errors |執行已完成，但發生應調查的錯誤 (數量少於 5,000 個)。 |
| completed-\*-warnings |執行完成，但某些資料並未處於預期的狀態。 如果您遇到錯誤，則此訊息通常只是一個徵狀。 在您解決錯誤之前，不應該調查警告。 |
| 成功 |沒有問題。 |

當您選取某個資料列時，底部會更新以顯示該執行的詳細資料。 在底部的最左邊，可能會有一份顯示 [步驟 #] 的清單。 如果您的樹系中有多個網域，而每個網域都以一個步驟來代表，則只會顯示此清單。 您可以在 [分割區] 標題下方找到網域名稱。 在 [同步處理統計資料] 下方，您可以找到關於已處理的變更次數詳細資訊。 您可以按一下連結，以取得變更的物件清單。 如果您有物件發生錯誤，這些會顯示於 [同步處理錯誤] 下方。

如需詳細資訊，請參閱[針對未同步的物件進行疑難排解](tshoot-connect-object-not-syncing.md)

## <a name="next-steps"></a>後續步驟
深入了解 [Azure AD Connect 同步](how-to-connect-sync-whatis.md) 組態。

深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
