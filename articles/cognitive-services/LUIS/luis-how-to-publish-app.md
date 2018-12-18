---
title: 將 LUIS 應用程式發佈至預測端點
titleSuffix: Azure Cognitive Services
description: 當您完成作用中 LUIS 應用程式的建置和測試時，將它發佈至端點以供用戶端應用程式使用。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 54d3b66f6a452e97e1d354fa75eb1b21065ab3e2
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47031266"
---
# <a name="publish-your-trained-app"></a>發佈訓練的應用程式

當您完成作用中 LUIS 應用程式的建置和測試時，將它發佈至端點以供用戶端應用程式使用。 

<a name="publish-your-trained-app-to-an-http-endpoint"></a>

## <a name="publishing"></a>發佈

若要發佈至端點，請選取右側面板頂端的 [發佈]。 

![右上方導覽列](./media/luis-how-to-publish-app/publish-top-nav-bar.png)

在快顯視窗顯示時選取正確的位置：預備或生產。 透過使用兩個發佈位置，可讓您具有兩個具有已發佈端點的不同版本，或兩個不同端點上的相同版本。 

應用程式會發佈至與 LUIS 入口網站中新增的 LUIS 資源相關聯的所有區域。 例如，對於在 [www.luis.ai](https://www.luis.ai) 上建立的應用程式，如果您在 **westus** 中建立 LUIS 資源並將它當作資源新增至應用程式，則會在該區域中發佈應用程式。 如需 LUIS 區域的相關資訊，請參閱[區域](luis-reference-regions.md)。
 
![發佈快顯視窗](./media/luis-how-to-publish-app/publish-pop-up.png)

成功發佈應用程式時，綠色成功通知會顯示在瀏覽器上方。 綠色通知列也包含端點的連結。 

![發佈快顯視窗](./media/luis-how-to-publish-app/publish-success.png)

如果您需要端點 URL，請選取此連結。 您也可以選取頂端功能表中的 [管理]，然後選取左側功能表中的 [金鑰和端點]，以取得端點 URL。 

## <a name="configuring-publish-settings"></a>配置發佈設定

選取右上方導覽列中的 [管理]，然後選取 [發佈設定]，以配置發佈設定。 

![發佈設定](./media/luis-how-to-publish-app/publish-settings.png)

### <a name="publish-after-enabling-sentiment-analysis"></a>在啟用情感分析之後發佈

<a name="enable-sentiment-analysis"></a>

情感分析可讓 LUIS 與[文字分析](https://azure.microsoft.com/services/cognitive-services/text-analytics/)整合，以提供情感和關鍵片語分析。 

您不需要提供文字分析金鑰，而且不會向您的 Azure 帳戶收取此服務的費用。 在您核取此設定之後，此設定就是永久性的。 

情感資料具有 1 與 0 之間的分數，指出資料的正 (較接近 1) 或負 (較接近 0) 情感。

如需使用情感分析之 JSON 端點回應的詳細資訊，請參閱[情感分析](luis-concept-data-extraction.md#sentiment-analysis)



## <a name="next-steps"></a>後續步驟

* 請參閱[管理金鑰](./luis-how-to-manage-keys.md)，將金鑰新增至 LUIS 的 Azure 訂用帳戶金鑰，以及了解如何設定 Bing 拼字檢查金鑰並且在結果中包含所有意圖。
* 如需如何在測試主控台中測試已發佈應用程式的指示，請參閱[訓練和測試應用程式](luis-interactive-test.md)。

