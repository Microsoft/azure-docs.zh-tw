---
title: Azure 資訊安全中心搜尋 | Microsoft Docs
description: 了解 Azure 資訊安全中心如何使用 Log Analytics 搜尋來擷取和分析您的安全性資料。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: 45b9756b-6449-49ec-950b-5ed1e7c56daa
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/11/2017
ms.author: terrylan
ms.openlocfilehash: 513c98237a322dabd6b2bf13443e8998ca843b1d
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23040433"
---
# <a name="azure-security-center-search"></a>Azure 資訊安全中心搜尋
Azure 資訊安全中心使用 [Log Analytics 搜尋](../log-analytics/log-analytics-log-searches.md)來擷取和分析您的安全性資料。 Log Analytics 包含可快速擷取及彙總資料的查詢語言。 您可以從安全性中心利用 Log Analytics 搜尋來建構查詢，並分析收集的資料。

資訊安全中心的免費層和標準層皆可使用搜尋。  記錄搜尋中的可用資料取決於套用至您工作區的層級。  如需詳細資訊，請參閱資訊安全中心[價格頁面](../security-center/security-center-pricing.md)。


> [!NOTE]
> 資訊安全中心不會儲存免費層下工作區的安全性資料。 您可以將各種不同的記錄傳送到在免費層下的工作區，並在該資料上搜尋，但是搜尋結果不會包含資訊安全中心的資料。 資訊安全中心只會儲存標準層下工作區的資料。
>
>

## <a name="access-search"></a>存取搜尋
1. 在資訊安全中心主功能表下，選取 [搜尋]。

  ![選取記錄搜尋][1]

2. 資訊安全中心會列出您 Azure 訂用帳戶下的所有工作區。 選取工作區。 (如果您只有一個工作區，這個工作區選取器不會出現。)

  ![選取工作區][2]

3. [記錄搜尋] 隨即開啟。 若要查詢所選取工作區下的詳細資料，請輸入此範例查詢：

  SecurityEvent | 其中 EventID = = 4625 | summarize count() by TargetAccount

  結果會顯示無法登入的所有帳戶 (事件 4625)。

  ![搜尋結果][3]

如需進一步了解如何查詢所選取工作區下的，請參閱 [Log Analytics 查詢語言](../log-analytics/log-analytics-search-reference.md)。

## <a name="next-steps"></a>後續步驟
您會在本文中了解如何存取資訊安全中心的搜尋。 資訊安全中心使用 Log Analytics 搜尋。 若要深入了解 Log Analytics 搜尋，請參閱：

- [什麼是 Log Analytics？](../log-analytics/log-analytics-overview.md) – Log Analytics 概觀
- [了解 Log Analytics 中的記錄搜尋](../log-analytics/log-analytics-log-search-new.md) - 說明記錄搜尋在 Log Analytics 中的使用方式，並且提供在建立記錄搜尋之前應該了解的概念
- [在 Log Analytics 中使用記錄搜尋以尋找資料](../log-analytics/log-analytics-log-searches.md) – 使用記錄搜尋的教學課程
- [Log Analytics 搜尋參考](../log-analytics/log-analytics-search-reference.md) – 說明 Log Analytics 中的查詢語言

若要深入了解資訊安全中心，請參閱：

- [資訊安全中心概觀](security-center-intro.md) – 說明資訊安全中心的主要功能

<!--Image references-->
[1]: ./media/security-center-search/search.png
[2]: ./media/security-center-search/workspace-selector.png
[3]: ./media/security-center-search/log-search.png
