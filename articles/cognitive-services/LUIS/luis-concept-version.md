---
title: 了解 LUIS 中的版本設定
titleSuffix: Azure Cognitive Services
description: 了解如何在 Language Understanding (LUIS) 中使用版本來管理變更
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 5d78172ae441300cdc39df8b911fd7ecaa42df9f
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47032687"
---
# <a name="versions"></a>版本
使用[版本](luis-how-to-manage-versions.md)來建立相同應用程式的不同模型。 

## <a name="version-id"></a>版本識別碼
版本識別碼由字元、數字或 '.' 所組成，且長度不可超過 10 個字元。

## <a name="initial-version"></a>初始版本
初始版本 (0.1) 是預設的作用中版本。 

## <a name="active-version"></a>作用中版本
將[版本設定為作用中](luis-how-to-manage-versions.md#set-active-version)，意謂著它目前在 [LUIS](luis-reference-regions.md) 網站中處於編輯和測試狀態。 將版本設定為作用中來存取其資料、進行更新，以及測試和發佈它。

目前作用中版本的名稱會顯示在左上方面板應用程式名稱的後面。 

[ ![變更作用中版本](./media/luis-concept-version/version-in-nav-bar-inline.png) ](./media/luis-concept-version/version-in-nav-bar-expanded.png#lightbox)

## <a name="versions-and-publishing-slots"></a>版本和發佈位置
您可以發佈至預備位置和產品位置。 每個位置的版本可以不同，也可以相同。 這在透過端點 (Bot 或其他 LUIS 呼叫應用程式可用的端點) 確認模型版本之間的變更時，相當有用。 

## <a name="clone-a-version"></a>複製版本
複製版本以建立現有版本的複本，並將其儲存成新版本。 複製版本以使用現有版本的相同內容作為新版本的起始點。 複製版本之後，新版本就會變成**作用中**版本。 

## <a name="import-and-export-a-version"></a>匯入及匯出版本
您可以在應用程式層級匯入版本。 該版本會變成作用中版本，並使用應用程式檔案 "versionId" 屬性中的版本識別碼。 您也可以在版本層級匯入至現有應用程式。 新版本會變成作用中版本。 

您可以在應用程式層級匯出版本，也可以在版本層級匯出版本。 唯一的差別在於，在應用程式層級匯出的版本是目前作用中的版本，而在版本層級，您則可以在 [[Settings](luis-how-to-manage-versions.md)] \(設定\) 頁面上選擇任何要匯出的版本。 

匯出的檔案不會包含機器學習資訊，因為應用程式在匯出後會重新定型。 匯出的檔案不會包含共同作業者 -- 您必須在將版本匯入新應用程式後，將這些資訊加回。

## <a name="export-each-version-as-app-backup"></a>將每個版本匯出作為應用程式備份
若要備份您的 LUIS 應用程式，請在 [[Settings](luis-how-to-manage-versions.md)] \(設定\) 頁面上匯出每個版本。

## <a name="delete-a-version"></a>刪除版本
您可以從 [Settings] \(設定\) 頁面上的 [Versions] \(版本\) 清單中刪除作用中版本以外的所有版本。 

## <a name="version-availability-at-the-endpoint"></a>端點上的版本可用性
系統會自動在您的應用程式[端點](luis-glossary.md#endpoint)上提供已定型的版本。 若要在您的應用程式端點上提供某個版本，您必須[發佈](luis-how-to-publish-app.md)或重新發佈該版本。 您可以發佈成 [Staging] \(預備\) 和 [Production] \(生產\)，這可在端點上最多為您提供兩個應用程式版本。 如果您需要在端點上提供更多應用程式版本，則應該將版本匯出，然後再重新匯入至新的應用程式。 新應用程式會有不同的應用程式識別碼。

## <a name="collaborators"></a>共同作業者
擁有者和所有[共同作業者](luis-how-to-collaborate.md)都具備所有應用程式版本的完整存取權。

## <a name="next-steps"></a>後續步驟

了解如何在應用程式設定頁面上新增[版本設定](luis-how-to-manage-versions.md)。 

了解如何將[意圖](luis-concept-intent.md)設計成模型。