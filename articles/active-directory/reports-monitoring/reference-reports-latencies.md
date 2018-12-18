---
title: Azure Active Directory 報告延遲 | Microsoft Docs
description: 深入了解在您的 Azure 入口網站中針對顯示報告事件所花費的時間長度
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
editor: ''
ms.assetid: 9b88958d-94a2-4f4b-a18c-616f0617a24e
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: report-monitor
ms.date: 12/15/2017
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: f0de2f8700bef83b5a8a9303e90c97aab29722a3
ms.sourcegitcommit: 1af4bceb45a0b4edcdb1079fc279f9f2f448140b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "42141021"
---
# <a name="azure-active-directory-reporting-latencies"></a>Azure Active Directory 報告延遲

透過 Azure Active Directory 中的[報告](../active-directory-preview-explainer.md)，您可取得判斷您的環境執行狀況所需的所有資訊。 Azure 入口網站中針對顯示報告資料所花費的時間長度也稱為延遲。 

本主題列出 Azure 入口網站中所有報告類別的延遲資訊。 


## <a name="activity-reports"></a>活動報告

有兩個活動報告的區域︰

- **登入活動** – 受控應用程式和使用者登入活動的使用情況相關資訊
- **稽核記錄檔** - 使用者和群組管理、受控應用程式和目錄活動的相關系統活動資訊

下表列出活動報告的延遲資訊。

| 報告 | 延遲 (P95) |延遲 (P99)|
| :-- | --- | --- | 
| 稽核記錄 | 2 分鐘  | 5 分鐘  |
| 登入 | 2 分鐘  | 5 分鐘 |







## <a name="security-reports"></a>安全性報告

有兩個安全性報告的區域︰

- **有風險的登入** - 有風險的登入表示非使用者帳戶合法擁有者的某人嘗試登入。 
- **標幟為有風險的使用者** - 有風險的使用者表示可能被盜用的使用者帳戶。 

下表列出安全性報告的延遲資訊。

| 報告 | 最小值 | 平均值 | 最大值 |
| :-- | --- | --- | --- |
| 有風險的使用者          | 5 分鐘   | 15 分鐘  | 2 小時  |
| 有風險的登入         | 5 分鐘   | 15 分鐘  | 2 小時  |

## <a name="risk-events"></a>風險事件

Azure Active Directory 使用調適性機器學習服務演算法和啟發學習法，來偵測與您使用者帳戶相關的可疑動作。 偵測到的每個可疑動作都會儲存在名為風險事件的記錄中。

下表列出風險事件的延遲資訊。

| 報告 | 最小值 | 平均值 | 最大值 |
| :-- | --- | --- | --- |
| 從匿名 IP 位址登入 |5 分鐘 |15 分鐘 |2 小時 |
| 從不熟悉的位置登入 |5 分鐘 |15 分鐘 |2 小時 |
| 認證外洩的使用者 |2 小時 |4 小時 |8 小時 |
| 不可能到達非典型位置的移動 |5 分鐘 |1 小時 |8 小時  |
| 從受感染的裝置登入 |2 小時 |4 小時 |8 小時  |
| 從具有可疑活動的 IP 位址登入 |2 小時 |4 小時 |8 小時  |



## <a name="next-steps"></a>後續步驟

如果您想要深入了解 Azure 入口網站中的活動報告，請參閱︰

- [Azure Active Directory 入口網站中的登入活動報告](concept-sign-ins.md)
- [Azure Active Directory 入口網站中的稽核活動報告](concept-audit-logs.md)

如果您想要深入了解 Azure 入口網站中的安全性報告，請參閱︰

- [Azure Active Directory 入口網站中具有風險的使用者安全性報告](concept-user-at-risk.md)
- [Azure Active Directory 入口網站中的風險登入報告](concept-risky-sign-ins.md)

如果您想要深入了解風險事件，請參閱 [Azure Active Directory 風險事件](concept-risk-events.md)。
