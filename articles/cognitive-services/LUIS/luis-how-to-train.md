---
title: 訓練 LUIS 版本
titleSuffix: Azure Cognitive Services
description: 訓練程序會教導您的 Language Understanding (LUIS) 應用程式版本，以改善其自然語言理解。 在模型的更新之後訓練 LUIS 應用程式，例如新增、編輯、加上標籤或刪除實體、意圖或語調。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/24/2018
ms.author: diberry
ms.openlocfilehash: f27716cc416b162a5b2df5542d709058f3b3e903
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47182025"
---
# <a name="train-your-luis-app-version"></a>訓練 LUIS 應用程式版本

訓練程序會教導您的 Language Understanding (LUIS) 應用程式，以改善其自然語言理解。 在模型的更新之後訓練 LUIS 應用程式，例如新增、編輯、加上標籤或刪除實體、意圖或語調。 

<!--
When you train a LUIS app by example, LUIS generalizes from the examples you have labeled, and it learns to recognize the relevant intents and entities. This teaches LUIS to improve classification accuracy in the future. -->

訓練和[測試](luis-concept-test.md)應用程式是一種反覆程序。 在您訓練 LUIS 應用程式之後，請使用範例語調來測試它，查看是否正確地辨識意圖和實體。 若為否，請更新 LUIS 應用程式、訓練，並重新測試。 

## <a name="how-to-train"></a>如何訓練
若要開始反覆程序，您至少需要先訓練 LUIS 應用程式一次。 請在訓練之前，確定每個意圖都至少有一個語調。

1. 在 [我的應用程式] 頁面上選取應用程式名稱，來存取應用程式。 

2. 在應用程式中，選取上方面板中的 [訓練]。 

3. 訓練完成時，綠色通知列會顯示在瀏覽器頂端。

<!-- The following note refers to what might cause the error message "Training failed: FewLabels for model: <ModelName>" -->

>[!NOTE]
>若應用程式中有一或多個未包含範例語調的意圖，則無法訓練應用程式。 新增所有意圖的語調。 如需詳細資訊，請參閱[新增範例語調](luis-how-to-add-example-utterances.md)。

## <a name="train-with-all-data"></a>以所有資料進行訓練
訓練會使用一小部分的負面取樣。 如果您想要使用所有資料，而不是使用小部分的負面取樣，請使用[版本設定 API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings)，並將 `UseAllTrainingData` 設為 true，來關閉這項功能。 

## <a name="next-steps"></a>後續步驟

* [使用 LUIS 標示建議的語調](luis-how-to-review-endoint-utt.md) 
* [使用功能來改善 LUIS 應用程式效能](luis-how-to-add-features.md) 