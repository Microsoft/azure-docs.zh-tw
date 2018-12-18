---
title: 快速入門：使用認知服務語音 SDK 在瀏覽器上以 JavaScript 辨識語音
titleSuffix: Microsoft Cognitive Services
description: 了解如何使用認知服務語音 SDK 在瀏覽器上以 JavaScript 辨識語音
services: cognitive-services
author: fmegen
ms.service: cognitive-services
ms.technology: Speech
ms.topic: article
ms.date: 09/24/2018
ms.author: fmegen
ms.openlocfilehash: b925a382a32f320c5d6906bc3fa23f046d260cb0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46997432"
---
# <a name="quickstart-recognize-speech-in-javascript-in-a-browser-using-the-cognitive-services-speech-sdk"></a>快速入門：使用認知服務語音 SDK 在瀏覽器上以 JavaScript 辨識語音

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

在此文章中，您將了解如何使用認知服務語音 SDK 的 JavaScript 繫結，將語音謄寫為文字以建立網站。
應用程式是以「Microsoft 認知服務語音 SDK ([下載版本 1.0.0](https://aka.ms/csspeech/jsbrowserpackage))」為基礎。

## <a name="prerequisites"></a>必要條件

* 語音服務適用的訂用帳戶金鑰。 請參閱[免費試用語音服務](get-started.md)。
* PC 或 Mac 以及運作正常的麥克風。
* 文字編輯器。
* Chrome 或 Microsoft Edge 的目前版本。
* (選擇性) 支援裝載 PHP 指令碼的 Web 伺服器。

## <a name="create-a-new-website-folder"></a>建立新的網站資料夾

建立新的空白資料夾。 如果您想要在 Web 伺服器上裝載範例，請確定 Web 伺服器可以存取該資料夾。

## <a name="unpack-the-speech-sdk-for-javascript-into-that-folder"></a>將適用於 JavaScript 的語音 SDK 解壓縮到該資料夾

下載 [.zip 套件](https://aka.ms/csspeech/jsbrowserpackage)形式的語音 SDK，並將它解壓縮到新建立的資料夾。 這應該會使得兩個檔案解壓縮，亦即 `microsoft.cognitiveservices.speech.sdk.bundle.js` 與 `microsoft.cognitiveservices.speech.sdk.bundle.js.map`。
第二個檔案是選擇性的，可在必要時用來協助針對 SDK 程式碼進行偵錯。

## <a name="create-an-indexhtml-page"></a>建立 index.html 網頁

在資料夾中建立名為 `index.html` 的新檔案，並使用文字編輯器開啟此檔案。

1. 建立下列 HTML 基本架構：

  ```html
  <html>
  <head>
      <title>Microsoft Cognitive Service Speech SDK JavaScript Quickstart</title>
  </head>
  <body>
    <!-- UI code goes here -->

    <!-- SDK reference goes here -->

    <!-- Optional authorization token request goes here -->

    <!-- Sample code goes here -->
  </body>
  </html>
  ```

1. 將下列 UI 程式碼新增至您檔案中的第一個註解下方：

  [!code-html[](~/samples-cognitive-services-speech-sdk/quickstart/js-browser/index.html#uidiv)]

1. 新增對語音 SDK 的參考

  [!code-html[](~/samples-cognitive-services-speech-sdk/quickstart/js-browser/index.html#speechsdkref)]

1. 連接辨識按鈕的處理常式、辨識結果，以及 UI 程式碼定義的訂用帳戶相關欄位：

  [!code-html[](~/samples-cognitive-services-speech-sdk/quickstart/js-browser/index.html#quickstartcode)]

## <a name="create-the-token-source-optional"></a>建立權杖來源 (選擇性)

如果想要在 Web 伺服器上裝載網頁，可以選擇性提供示範應用程式的權杖來源。
這樣一來，訂用帳戶金鑰將永遠不會離開您的伺服器，同時可讓使用者不需要輸入任何授權代碼就能使用語音功能。

1. 建立名為 `token.php` 的新檔案。 在此範例中，我們會假設您的 Web 伺服器支援 PHP 指令碼語言。 輸入下列程式碼：

  [!code-php[](~/samples-cognitive-services-speech-sdk/quickstart/js-browser/token.php)]

1. 編輯 `index.html` 檔案，並將下列程式碼加到您的檔案中：

  [!code-html[](~/samples-cognitive-services-speech-sdk/quickstart/js-browser/index.html#authorizationfunction)]

> [!NOTE]
> 授權權杖的存留期是有限的。
> 這個簡化範例不會說明如何自動重新整理授權權杖。 如果您是使用者，可以手動重新載入或按 F5 以重新整理頁面。

## <a name="build-and-run-the-sample-locally"></a>在本機建置並執行範例

若要啟動應用程式，請按兩下 index.html 檔案或使用最愛的網頁瀏覽器開啟 index.html。 它將會提供簡單的 GUI，供您輸入訂用帳戶金鑰與[區域](regions.md)，並使用麥克風觸發辨識。

## <a name="build-and-run-the-sample-via-a-web-server"></a>透過 Web 伺服器建置並執行範例

若要啟動應用程式，請開啟您最愛的網頁瀏覽器並指向資料夾裝載所在的公用 URL，輸入您的[地區](regions.md)，並使用麥克風觸發辨識。 如果已設定，它將會從您的權杖來源取得權杖。

[!INCLUDE [Download the sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]
在 `quickstart/js-browser` 資料夾中尋找此範例。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [取得我們的範例](speech-sdk.md#get-the-samples)
