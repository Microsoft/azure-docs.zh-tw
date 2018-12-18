---
title: 教學課程：異常偵測 Java
titlesuffix: Azure Cognitive Services
description: 探索會使用異常偵測 API 的 Java 應用程式。 將原始資料點傳送到 API，並取得預期的值和異常點。
services: cognitive-services
author: wenya
manager: bix
ms.service: cognitive-services
ms.component: anomaly-detection
ms.topic: tutorial
ms.date: 05/01/2018
ms.author: wenya
ms.openlocfilehash: 4b544e2e59a40cebf75042c4040b84bceebcecf7
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48887695"
---
# <a name="tutorial-anomaly-detection-with-java-application"></a>教學課程：使用 Java 應用程式進行異常偵測

[!INCLUDE [PrivatePreviewNote](../../../../../includes/cognitive-services-anomaly-finder-private-preview-note.md)]

本文示範如何使用簡單的 Java 應用程式來叫用異常偵測 API。  
此範例會使用您的訂用帳戶金鑰來將時間序列資料提交到異常偵測 API，然後從該 API 針對每個資料點取得所有異常點和預期的值。

## <a name="prerequisites"></a>必要條件

### <a name="platform-requirements"></a>平台需求

本教學課程是使用 [IntelliJ IDEA](https://www.jetbrains.com/idea) \(英文\) 來開發的。 此外，您也必須安裝 [Java 開發套件 (JDK)](http://www.oracle.com/technetwork/java/javase/downloads/index.html) \(英文\) 1.8 以上版本以及 [Apache 的 Maven](http://maven.apache.org/) \(英文\) 最新建置工具。

### <a name="subscribe-to-anomaly-detection-and-get-a-subscription-key"></a>訂閱異常偵測並取得訂用帳戶金鑰 

[!INCLUDE [GetSubscriptionKey](../includes/get-subscription-key.md)]
 

## <a name="download-the-tutorial-project"></a>下載教學課程專案

1. 移至 MicrosoftAnomalyDetection [Java 存放庫](https://github.com/MicrosoftAnomalyDetection/java-sample)。
2. 按一下 [複製] 或 [下載] 按鈕。
3. 按一下 [下載 ZIP] 以下載教學課程專案的 .zip 檔案。

<a name="Step1"></a>
### <a name="open-the-tutorial-project"></a>開啟教學課程專案

1. 將教學課程專案的 .zip 檔案解壓縮。
2. 在 IntelliJ IDEA 中，按一下 [檔案] > [開啟]，隨即會出現 [開啟檔案或專案] 對話方塊。
3. 選取已解壓縮之專案的根路徑，然後按一下 [確定]。
4. 在 [專案] 面板中，展開 [src] > [main] > [java]。
5. 按兩下 com.microsoft.cognitiveservice.anomalydetection.Main.java，將檔案下載到編輯器。

<a name="Step2"></a>
### <a name="replace-subscriptionkey-and-uri-region"></a>取代 subscriptionKey 和 URI 區域

```
// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace the subscriptionKey string value with your valid subscription key.
public static final String subscriptionKey = "<Subscription Key>";

public static final String uriBase = "https://api.labs.cognitive.microsoft.com/anomalyfinder/v1.0/anomalydetection";

```

<a name="Step3"></a>
### <a name="build-and-run-the-tutorial-project"></a>建置並執行教學課程專案

1. 以滑鼠右鍵按一下 com.microsoft.cognitiveservice.anomalydetection.Main.java 原始程式碼索引標籤中的任何位置來顯示功能表。 
2. 選取 [執行 'Main.main()']
3. 範例要求的結果將傳回並顯示於終端機中。

### <a name="result-of-the-tutorial-project"></a>教學課程專案的結果

[!INCLUDE [diagrams](../includes/diagrams.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [REST API 參考資料](https://dev.labs.cognitive.microsoft.com/docs/services/anomaly-detection/operations/post-anomalydetection)
