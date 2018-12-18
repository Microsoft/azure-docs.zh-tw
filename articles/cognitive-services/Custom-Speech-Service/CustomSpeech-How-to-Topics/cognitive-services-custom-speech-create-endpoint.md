---
title: 使用 Azure 上的自訂語音服務來建立自訂語音端點 | Microsoft Docs
description: 了解如何使用「認知服務」中的「自訂語音服務」來建立自訂的語音轉文字端點。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: article
ms.date: 07/08/2017
ms.author: panosper
ms.openlocfilehash: 7b7bbf89f04e4c8b12df46c2a7100a08001eb7d1
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340796"
---
# <a name="create-a-custom-speech-to-text-endpoint"></a>建立自訂的語音轉文字端點

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

在您建立自訂原音模型或語言模型之後，您可以將這些模型部署至自訂的語音轉文字端點。 

## <a name="create-an-endpoint"></a>建立端點
若要建立新的自訂端點，請在頁面頂端的 [自訂語音] 功能表上選取 [部署]。 這個動作會帶您前往 [部署] 頁面，內含目前自訂端點的資料表。 如果您尚未建立任何端點，此資料表會是空白的。 資料表標題會反映目前的地區設定。 

若要建立不同語言的部署，請選取 [變更地區設定]。 如需所支援語言的詳細資訊，請參閱[自訂語音服務所支援的地區設定](cognitive-services-custom-speech-change-locale.md)。

若要建立新的端點，請選取 [新建]。 在 [建立部署] 窗格中，輸入自訂部署的 [名稱] 和 [描述] 方塊中所含的資訊。

在 [訂用帳戶] 下拉式方塊中，選取您要使用的訂用帳戶。 如果是 S2 訂用帳戶，您可以選取縮放單位和內容記錄。 如需縮放單位和記錄的詳細資訊，請參閱[自訂語音服務的計量和配額](../cognitive-services-custom-speech-meters.md)。

下表顯示縮放單位與可用並行要求的對應情形：

| 縮放單位 | 並行要求數 |
| ------ | ----- |
| 0 | 1 |
| 1 | 5 |
| 2 | 10 |
| 3 | 15 |
| n | 5 * n |

您也可以選取要開啟還是關閉內容記錄。 也就是說，您可以選取是否要儲存端點流量以供 Microsoft 內部使用。 如果未選取，則會隱藏儲存流量的選項。 隱藏內容記錄會導致額外的成本。 如需詳細資訊，請參閱[定價資訊頁面](https://azure.microsoft.com/pricing/details/cognitive-services/custom-speech-service/)。

> [!NOTE]
> 在定價頁面上，內容記錄的名稱為「無追蹤」。
>


此外，Microsoft 也會提供粗略估計的成本，讓您了解縮放單位和內容記錄會對成本產生什麼影響。 這只是粗略的估計值，因此可能會與實際成本不同。

> [!NOTE]
> 這些設定不適用於 F0 (免費層) 訂用帳戶。
>

在 [原音模型] 清單中，選取所需的原音模型，然後在 [語言模型] 清單中，選取所需的語言模型。 原音和語言模型的選項一律包含基本 Microsoft 模型。 所選取的基本模型會限制組合內容。 您無法混合使用交談式基本模型與搜尋服務，也無法決定基本模型。

![[建立部署] 頁面](../../../media/cognitive-services/custom-speech-service/custom-speech-deployment-create2.png)

> [!NOTE]
> 請務必選取核取方塊來接受使用規定和定價資訊。
>

在選取了原音和語言模型之後，請選取 [建立]。 這個動作會讓您回到 [部署] 頁面。 資料表中現在包含與新的端點對應的項目。 端點的狀態會反映其建立當下的狀態。 可能需要 30 分鐘的時間才能具現化具有自訂模型的新端點。 當部署狀態為 [完成] 時，端點就可供使用。

![[部署] 頁面](../../../media/cognitive-services/custom-speech-service/custom-speech-deployment-ready.png)

部署準備就緒時，部署名稱就會變成連結。 選取連結會顯示 [部署資訊] 頁面，其中會顯示要與 HTTP 要求或 Microsoft 認知服務語音用戶端程式庫 (其使用 Web 通訊端) 搭配使用的自訂端點 URL。

![[部署資訊] 頁面](../../../media/cognitive-services/custom-speech-service/custom-speech-deployment-info2.png)

## <a name="next-steps"></a>後續步驟

如需更多教學課程，請參閱：
* [使用自訂的語音轉文字端點](cognitive-services-custom-speech-use-endpoint.md)
* [建立自訂原音模型](cognitive-services-custom-speech-create-acoustic-model.md)
* [建立自訂語言模型](cognitive-services-custom-speech-create-language-model.md)
