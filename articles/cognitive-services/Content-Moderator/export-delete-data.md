---
title: 匯出或刪除您的資料 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 了解如何匯出或刪除 Content Moderator 中的資料。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 05/25/2018
ms.author: pafarley
ms.openlocfilehash: f4da7cdffc2d2aad21c2ea4cfc67939d0dbba530
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339367"
---
# <a name="export-or-delete-user-data-in-content-moderator"></a>匯出或刪除 Content Moderator 中的使用者資料

Content Moderator 會收集使用者資料來操作本服務，但是客戶可以完整控制資料的檢視、匯出和刪除，並使用 [Review UI](http://contentmoderator.cognitive.microsoft.com/) 和 [API](https://docs.microsoft.com/azure/cognitive-services/content-moderator/api-reference)。

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

如需如何匯出和刪除 Content Moderator 中使用者資料的詳細資訊，請參閱下表。

| 資料 | 匯出作業 | 刪除作業 |
| ---- | ---------------- | ---------------- |
| 帳戶資訊 (訂用帳戶金鑰) | N/A | 使用 Azure 入口網站刪除 (Azure 訂用帳戶)。 或者，使用 [Review UI](http://contentmoderator.cognitive.microsoft.com/) Team 設定頁面中的 [刪除小組] 按鈕。 |
| 自訂比對的影像 | [取得影像識別碼](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f676)。 影像會以單向專屬雜湊格式儲存，而且沒有任何方法可以擷取實際影像。 | [搜尋所有影像](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f686)。 或者，使用 Azure 入口網站刪除 Content Moderator 資源。 |
| 自訂比對的字詞 | [取得所有字詞](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67e) | [刪除所有字詞](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67d)。 或者，使用 Azure 入口網站刪除 Content Moderator 資源。 |
| 標記 | N/A | 使用 [Review UI Tag] \(Review UI 標籤\) 設定頁面中每個標籤可用的**刪除**圖示。 或者，使用 [Review UI](http://contentmoderator.cognitive.microsoft.com/) Team 設定頁面中的 [刪除小組] 按鈕。 |
| 評論 | [取得檢閱](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c2) | 使用 [Review UI](http://contentmoderator.cognitive.microsoft.com/) 小組設定頁面中的 [刪除小組] 按鈕。
| 使用者 | N/A | 使用 [Review UI](http://contentmoderator.cognitive.microsoft.com/) 小組設定頁面中每個使用者可用的**刪除**圖示。 或者，使用 [Review UI](http://contentmoderator.cognitive.microsoft.com/) Team 設定頁面中的 [刪除小組] 按鈕。 |

