---
title: Azure 搜尋服務 REST API 版本 2017-11-11-Preview | Microsoft Docs
description: Azure 搜尋服務 REST API 版本 2017-11-11-Preview 包含同義字和 moreLikeThis 搜尋等實驗性功能。
services: search
author: HeidiSteen
manager: cgronlun
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: search
ms.date: 06/28/2018
ms.author: HeidiSteen
ms.openlocfilehash: b5cb60bf16a4c904c9a6060113eba8b4d3a671ef
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37112597"
---
# <a name="azure-search-service-rest-api-version-2017-11-11-preview"></a>Azure 搜尋服務 REST api-version 2017-11-11-Preview
本文描述 Azure 搜尋服務 REST API 的 `api-version=2017-11-11-Preview` 版本，並提供下列尚未提供的實驗性功能。

> [!NOTE]
> 預覽功能適用於目標為收集意見反應的測試和實驗，而且可能會有所變更。 我們強烈建議您避免在實際執行的應用程式中使用預覽 API。


## <a name="new-in-2017-11-11-preview"></a>2017-11-11-Preview 中的新功能

[**自動完成**](search-autocomplete-tutorial.md)會加入現有[建議 API](https://docs.microsoft.com/rest/api/searchservice/suggestions)，以將補充預先輸入體驗新增至搜尋列。 自動完成會傳回候選查詢字詞，使用者可以選擇這些字詞作為後續搜尋的查詢字串。 建議會傳回實際文件以回應部分輸入：搜尋結果會隨著輸入的搜尋字詞變長和更加明確，而立即動態變更。

[**認知搜尋**](cognitive-search-concept-intro.md)是 Azure 搜尋中的新擴充功能，可在非文字來源和無差異文字中尋找潛在資訊，並將其轉換成 Azure 搜尋服務中可全文檢索搜尋的內容。 在預覽 REST API 中導入或修改了下列資源。 無論您呼叫正式推出或預覽版本，所有其他 REST API 都是相同的。

+ [Skillset operations(api-version=2017-11-11-Preview)](https://docs.microsoft.com/rest/api/searchservice/skillset-operations)

+ [建立索引子 (api-version=2017-11-11-Preview)](https://docs.microsoft.com/rest/api/searchservice/create-indexer)

+ [預先定義的技能](cognitive-search-predefined-skills.md)

不論您如何設定 api-version，所有其他 REST API 都是相同的。 例如，`GET https://[service name].search.windows.net/indexes/hotels?api-version=2017-11-11-Preview` 和 `GET https://[service name].search.windows.net/indexes/hotels?api-version=2017-11-11` (不含`Preview`) 在功能上是相同的。

## <a name="other-preview-features"></a>其他預覽功能

舊版預覽中宣布的功能仍處於公開預覽。 如果您使用舊版預覽 api-version 呼叫 API，則可以繼續使用該版本或切換到 `2017-11-11-Preview`，而不會更改預期的行為。

+ [Azure Blob 索引中的 CSV 檔案](search-howto-index-csv-blobs.md) (在 `api-version=2015-02-28-Preview` 中導入) 仍是預覽功能。 這項功能是 Azure Blob 索引的一部分，並透過參數設定叫用。 CSV 檔案中的每一行都作為個別的文件編製索引。

+ [Azure Blob 索引中的 JSON 陣列](search-howto-index-json-blobs.md) (在 `api-version=2015-02-28-Preview` 中導入) 仍是預覽功能。 這項功能是 Azure Blob 索引的一部分，並透過參數設定叫用。 其中陣列中的每個元素都以個別文件來編製索引。

+ [moreLikeThis 查詢參數](search-more-like-this.md)，可尋找與特定文件相關的文件。 這個功能已存在舊版預覽中。 


## <a name="how-to-call-a-preview-api"></a>如何呼叫預覽 API

較舊的預覽仍可運作，但是經過一段時間後會變成過時。 如果您的程式碼呼叫 `api-version=2016-09-01-Preview` 或 `api-version=2015-02-28-Preview`，那些呼叫都仍然有效。 但是，只有最新預覽版本會重新整理改進功能。 

下列的範例語法說明對預覽 API 版本的呼叫。

    GET https://[service name].search.windows.net/indexes/[index name]/docs?search=*&api-version=2017-11-11-Preview

Azure 搜尋服務可以在多個版本中使用。 如需詳細資訊，請參閱 [API 版本](search-api-versions.md)。
