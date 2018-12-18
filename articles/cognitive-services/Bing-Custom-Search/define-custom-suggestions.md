---
title: Bing 自訂搜尋：定義自訂自動建議的建議 | Microsoft Docs
description: 描述如何使用自訂的建議設定自訂自動建議
services: cognitive-services
author: brapel
manager: ehansen
ms.service: cognitive-services
ms.technology: bing-custom-search
ms.topic: article
ms.date: 09/28/2017
ms.author: v-brapel
ms.openlocfilehash: 11d3b1c2d98caa8d6527c52bec1cc65ba22c6c3b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958742"
---
# <a name="configure-your-custom-autosuggest-experience"></a>設定您的自訂自動建議體驗

自訂自動建議會傳回與您的搜尋體驗相關之建議的搜尋查詢字串。 建議的查詢字串是以使用者在搜尋方塊中提供的部分查詢字串為基礎。 清單最多可包含 10 個建議。 

您可以指定只傳回自訂的建議，或者也要包含 Bing 的建議。 如果您包含 Bing 的建議，自訂的建議會在 Bing 的建議前面出現。 如果您提供足夠的相關建議，則傳回的建議清單可能不會包含 Bing 的建議。 Bing 的建議一律會在您的自訂搜尋執行個體內容中。 

若要為您的執行個體設定搜尋查詢建議，請按一下 [自動建議] 索引標籤。  

> [!NOTE]
> 若要使用此功能，您必須訂閱適當等級的「自訂搜尋」(請參閱[定價](https://azure.microsoft.com/pricing/details/cognitive-services/bing-custom-search/))。

建議最多可能需要 24 小時才會反映在服務端點中 (API 或託管的 UI)。

## <a name="enable-bing-suggestions"></a>啟用 Bing 的建議

若要啟用 Bing 的建議，請將 [自動 Bing 建議] 滑桿切換到開啟的位置。 滑桿會變成藍色。

## <a name="add-your-own-suggestions"></a>新增您自己的建議

若要新增您自己的查詢字串建議，請將它們新增到 [使用者定義建議] 底下的清單。 在清單中新增建議之後，請按 Enter 鍵或按一下 **+** 圖示。 您可以用任何語言指定建議。 您可以新增最多 5000 個查詢字串建議。

## <a name="upload-suggestions"></a>上傳建議

您可以選擇從檔案上傳建議清單。 檔案中的每一行都必須包含一個搜尋查詢字串。 若要上傳檔案，請按一下上傳圖示並選取要上傳的檔案。 服務會從檔案擷取建議並將它們新增到清單。

## <a name="remove-suggestions"></a>移除建議

若要移除查詢字串建議，請按一下您要移除之建議旁的移除圖示。

## <a name="block-suggestions"></a>封鎖建議

如果包含 Bing 的建議，您可以新增不希望 Bing 傳回的搜尋查詢字串清單。 若要新增封鎖的查詢字串，請按一下 [顯示封鎖的建議]。 將查詢字串新增到清單，然後按 Enter 鍵或按一下 **+** 圖示。 您可以新增最多 50 個封鎖的查詢字串。



[!INCLUDE[publish or revert](./includes/publish-revert.md)]

>[!NOTE]  
>這個程序可能需要 24 小時，自動建議設定的變更才會生效。


## <a name="enabling-autosuggest-in-hosted-ui"></a>在託管的 UI 中啟用自動建議

若要為託管的 UI 啟用查詢字串建議，請按一下 [託管的 UI]。 向下捲動到 [其他設定] 區段。 在 [Web 搜尋] 下，針對 [啟用自動建議] 選取 [開啟]。 若要啟用自動建議，您必須選取包含搜尋方塊的版面配置。


## <a name="calling-the-autosuggest-api"></a>呼叫自動建議 API

若要使用 Bing 自訂搜尋 API 取得建議的查詢字串，請傳送 `GET` 要求到下列端點。

```
GET https://api.cognitive.microsoft.com/bingcustomsearch/v7.0/Suggestions 
```

回應包含 `SearchAction` 物件清單，這些物件包含建議的查詢字串。

```
        {  
            "displayText" : "sailing lessons seattle",  
            "query" : "sailing lessons seattle",  
            "searchKind" : "CustomSearch"  
        },  
```

每個建議都包含 `displayText` 和 `query` 欄位。 `displayText` 欄位包含建議的查詢字串，您可以用來填入搜尋方塊的下拉式清單。

如果使用者從下拉式清單中選取建議的查詢字串，請在呼叫 [Bing 自訂搜尋 API](overview.md) 時，在 `query` 欄位中使用該查詢字串。


## <a name="next-steps"></a>後續步驟

- [取得自訂建議](./get-custom-suggestions.md)
- [搜尋自訂執行個體](./search-your-custom-view.md)
- [設定和使用自訂託管的 UI](./hosted-ui.md)