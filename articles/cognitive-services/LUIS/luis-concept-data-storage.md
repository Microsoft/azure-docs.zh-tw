---
title: LUIS 中的資料儲存 - Language Understanding
titleSuffix: Azure Cognitive Services
description: 了解 Language Understanding (LUIS) 中的資料儲存方式。 LUIS 會將資料以加密方式儲存在與金鑰所指定區域對應的 Azure 資料存放區中。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 6d42246b3078e0f4684dfe29bca720e8f8d779cd
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47034700"
---
# <a name="data-storage-and-removal-in-language-understanding-luis-cognitive-services"></a>Language Understanding (LUIS) 認知服務中的資料儲存和移除
LUIS 會將資料以加密方式儲存在與金鑰所指定區域對應的 Azure 資料存放區中。 此資料會儲存 30 天。 

## <a name="export-and-delete-app"></a>匯出及刪除應用程式
使用者可以完全控制應用程式的[匯出](luis-how-to-start-new-app.md#export-app)和[刪除](luis-how-to-start-new-app.md#delete-app)。 

## <a name="utterances-in-an-intent"></a>意圖中的語句
刪除用於將 [LUIS](luis-reference-regions.md) 定型的範例語句。 若您從 LUIS 應用程式刪除範例語句，則會從 LUIS Web 服務中予以移除，而且無法用於匯出。

## <a name="utterances-in-review"></a>檢閱中的語句
您可以在 [[Review endpoint utterances](luis-how-to-review-endoint-utt.md)] \(檢閱端點語句\) 頁面中，從 LUIS 所建議的使用者語句清單中刪除語句。 從此清單中刪除語句可防止系統建議這些語句，但並不會從記錄中刪除它們。

## <a name="accounts"></a>帳戶
若您刪除帳戶，則會刪除所有應用程式，以及其範例語句和記錄。 資料會保留 60 天，之後系統就會將帳戶和資料永久刪除。

刪除帳戶時，是從 [Settings] \(設定\) 頁面執行刪除。 請選取右上方導覽列中您的帳戶名稱，以移至 [Settings] \(設定\) 頁面。

## <a name="data-inactivity-as-an-expired-subscription"></a>資料非作用中狀態視為過期訂用帳戶
基於資料保留和刪除的目的，非作用中 LUIS 應用程式可能「由 Microsoft 斟酌決定」視為過期的訂用帳戶。 應用程式如果在過去 90 天符合下列條件，即可視為非作用中： 

* **沒有**任何對它發出的呼叫。
* 尚未經過修改。
* 沒有已指派的目前金鑰。
* 沒有已指派的使用者。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解如何匯出及刪除應用程式](luis-how-to-start-new-app.md)