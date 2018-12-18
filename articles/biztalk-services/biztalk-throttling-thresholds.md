---
title: 了解 BizTalk 服務中的節流 | Microsoft Docs
description: 了解 BizTalk 服務的節流閾值和產生的執行階段行為。 節流是以記憶體使用量和訊息數為依據。 MABS，WABS
services: biztalk-services
documentationcenter: ''
author: MandiOhlinger
manager: anneta
editor: ''
ms.assetid: f6663cf2-cda4-4bac-855e-27d2ad5c4fa4
ms.service: biztalk-services
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/07/2016
ms.author: mandia
ms.openlocfilehash: 39fc5ef36bb581c3a81c9948fda048f6cb75eb7e
ms.sourcegitcommit: dcf5f175454a5a6a26965482965ae1f2bf6dca0a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/10/2017
ms.locfileid: "24102081"
---
# <a name="biztalk-services-throttling"></a>BizTalk 服務：節流

> [!INCLUDE [BizTalk Services is being retired, and replaced with Azure Logic Apps](../../includes/biztalk-services-retirement.md)]

Azure BizTalk 服務根據兩個條件實作服務節流：記憶體使用量和同時訊息處理的數量。 本主題列出節流閾值，並說明發生節流狀況時的執行階段行為。

## <a name="throttling-thresholds"></a>節流閾值
下表列出節流來源和閾值：

|  | 說明 | 低閾值 | 高閾值 |
| --- | --- | --- | --- |
| 記憶體 |可用系統記憶體總計的 %/PageFileBytes。 <p><p>可用的 PageFileBytes 總計大約是系統 RAM 的 2 倍。 |60% |70% |
| 訊息處理 |同時處理的訊息數目 |40 * 核心數目 |100 * 核心數目 |

達到高閾值時，Azure BizTalk 服務就會開始節流。 達到低閾值時會停止節流。 例如，服務使用 65% 系統記憶體。 在此情況下，服務不會節流。 服務開始使用 70% 系統記憶體。 在此情況下，服務會節流，並持續節流直到服務使用 60% (低閾值) 系統記憶體為止。

Azure BizTalk 服務會追蹤節流狀態 (正常狀態與節流狀態) 和節流持續時間。

## <a name="runtime-behavior"></a>執行階段行為
Azure BizTalk 服務進入節流狀態時會發生下列情況：

* 依每一角色執行個體來節流。 例如︰<br/>
  RoleInstanceA 節流。 RoleInstanceB 未節流。 在此情況下，RoleInstanceB 中的訊息如預期般地處理。 RoleInstanceA 中的訊息會捨棄且失敗，並傳回下列錯誤：<br/><br/>
  **伺服器忙碌中。請再試一次。**<br/><br/>
* 所有提取來源不會輪詢或下載訊息。 例如︰<br/>
  管線會從外部 FTP 來源提取訊息。 進行提取的角色執行個體會進入節流狀態。 在此情況下，在角色執行個體停止節流之前，管線會停止下載其他訊息。
* 回應會傳送給用戶端，讓用戶端可以重新提交訊息。
* 您必須等待節流情況解決為止。 尤其，您必須等待到達到低閾值為止。

## <a name="important-notes"></a>重要事項
* 無法停用節流。
* 無法修改節流閾值。
* 節流的實作以全系統為範圍。
* Azure SQL Database Server 也有內建的節流。

## <a name="additional-azure-biztalk-services-topics"></a>其他 Azure BizTalk 服務主題
* [安裝 Azure BizTalk 服務 SDK](http://go.microsoft.com/fwlink/p/?LinkID=241589)<br/>
* [教學課程：Azure BizTalk 服務](http://go.microsoft.com/fwlink/p/?LinkID=236944)<br/>
* [如何開始使用 Azure BizTalk 服務 SDK](http://go.microsoft.com/fwlink/p/?LinkID=302335)<br/>
* [Azure BizTalk 服務](http://go.microsoft.com/fwlink/p/?LinkID=303664)<br/>

## <a name="see-also"></a>另請參閱
* [BizTalk 服務：開發人員、基本、標準和高級版本圖表](http://go.microsoft.com/fwlink/p/?LinkID=302279)<br/>
* [BizTalk 服務：佈建狀態圖](http://go.microsoft.com/fwlink/p/?LinkID=329870)<br/>
* [BizTalk 服務：儀表板、監視和調整索引標籤](http://go.microsoft.com/fwlink/p/?LinkID=302281)<br/>
* [BizTalk 服務：備份與還原](http://go.microsoft.com/fwlink/p/?LinkID=329873)<br/>
* [BizTalk 服務：簽發者名稱和簽發者金鑰](http://go.microsoft.com/fwlink/p/?LinkID=303941)<br/>

