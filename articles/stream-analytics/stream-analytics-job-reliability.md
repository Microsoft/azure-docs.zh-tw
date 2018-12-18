---
title: 避免 Azure 串流分析作業中發生服務中斷
description: 本文說明可讓串流分析作業升級具有彈性的指引。
services: stream-analytics
author: jseb225
manager: kfile
ms.author: jeanb
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/28/2017
ms.openlocfilehash: 47ccfe99d2ee6576dbb70324eb383f52d2a1b2e7
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30902706"
---
# <a name="guarantee-stream-analytics-job-reliability-during-service-updates"></a>在服務更新期間確保串流分析工作可靠性

完全受控服務的其中一個特性，就是能夠快速導入新的服務功能和改良功能。 因此，「串流分析」可以每週 (或更頻繁地) 進行服務更新部署。 不論做了多少測試，現有的執行中工作仍然可能有因引入錯誤而導致中斷的風險。 對於執行重要串流處理工作的客戶來說，必須避免這些風險。 有一個可供客戶用來降低此風險的機制，就是 Azure 的**[配對區域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)** 模型。 

## <a name="how-do-azure-paired-regions-address-this-concern"></a>Azure 配對區域如何解決這個問題？

「串流分析」可確保以個別的批次更新在配對區域中的工作。 如此一來，在找出潛在重大錯誤與進行補救的更新之間，便有足夠的時間間隔。

「印度中部是一個例外」(其配對區域印度南部並沒有提供「串流分析」服務)，「串流分析」更新部署不會在一組配對區域同時發生。 **相同群組內**多個區域中的部署可能**同時**發生。

關於**[可用性與配對區域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)** 的文章提供了有關配對區域的最新資訊。

建議客戶在兩個配對區域都部署相同的工作。 除了「串流分析」內部監視功能之外，也建議客戶將「兩個」工作都當作生產環境工作來監視。 如果中斷情況經識別是「串流分析」服務更新所造成，請適當地呈報此問題，並將所有下游取用者容錯移轉至狀況良好的工作輸出。 向支援服務呈報此問題將可防止配對區域受到新部署作業影響，並可維護配對工作的完整性。
