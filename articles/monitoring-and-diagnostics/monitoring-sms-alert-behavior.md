---
title: 動作群組中的 SMS 警示行為
description: SMS 訊息格式和回應 SMS 訊息以取消訂閱、重新訂閱，或要求說明。
author: dkamstra
services: monitoring
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 02/16/2018
ms.author: dukek
ms.component: alerts
ms.openlocfilehash: f2f463f6c428ce6c72e2640472376fa17a2bfe5a
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35263002"
---
# <a name="sms-alert-behavior-in-action-groups"></a>動作群組中的 SMS 警示行為
## <a name="overview"></a>概觀 ##
動作群組可讓您設定動作清單。 定義警示時會使用這些群組；確保觸發警示時，特定動作群組可以收到通知。 SMS 是其中一個支援的動作；SMS 通知示支援雙向通訊。 使用者可以回應 SMS 以便：

- **取消訂閱警示：** 使用者可取消訂閱所有動作群組或單一動作群組的 SMS 警示。
- **重新訂閱警示：** 使用者可重新訂閱所有動作群組或單一動作群組的 SMS 警示。  
- **要求說明︰** 使用者可在 SMS 上要求其他資訊。 系統會將他們導向本文。

本文涵蓋 SMS 警示的行為，以及使用者可依據其所在位置採取的回應動作：

## <a name="receiving-an-sms-alert"></a>接收 SMS 警示
設定成動作群組一部分的 SMS 收件者，將會在觸發警示時收到 SMS。 SMS 包含下列資訊：
* 會收到此警示的動作群組簡稱
- 警示的標題

| REPLY | 說明 |
| ----- | ----------- |
| DISABLE <Action Group Short name> | 停用來自動作群組的其他 SMS |
| ENABLE <Action Group Short name> | 重新啟用來自動作群組的 SMS |
| STOP | 停用來自所有動作群組的其他 SMS |
| START | 重新啟用來自所有動作群組的 SMS |
| HELP | 使用者將會收到包含本文連結的回應。 |

>[!NOTE]
>如果使用者已取消訂閱 SMS 警示，但又加入新的動作群組；則他們會接收到新動作群組的 SMS 警示，但對於所有之前的動作群組，仍是取消訂閱狀態。

## <a name="next-steps"></a>後續步驟
取得[活動記錄警示的概觀](monitoring-overview-alerts.md)，並了解如何收到警示  
深入了解 [SMS 速率限制](monitoring-alerts-rate-limiting.md)  
深入了解[動作群組](monitoring-action-groups.md)
