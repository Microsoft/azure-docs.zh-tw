---
title: 認知服務語音 SDK 文件 | Microsoft Docs
description: 版本資訊 - 最近版本中的變更
titleSuffix: Microsoft Cognitive Services
services: cognitive-services
author: wolfma61
ms.service: cognitive-services
ms.component: speech-service
ms.topic: article
ms.date: 09/24/2018
ms.author: wolfma
ms.openlocfilehash: cfc8f973ac9bced9396a081f7fb2ab6a4a1b2f7f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46963078"
---
# <a name="release-notes"></a>版本資訊

## <a name="cognitive-services-speech-sdk-100-2018-september-release"></a>認知服務語音 SDK 1.0.0：2018 年 9 月版

**新功能**

* iOS 上對 Objective-C 的支援。 請參閱我們的[適用於 iOS 的 Objective-C 快速入門](quickstart-objectivec-ios.md)。
* 瀏覽器中對 JavaScript 的支援。 請參閱我們的 [JavaScript 快速入門](quickstart-js-browser.md)。

**重大變更**

* 此版本引進了幾個重大變更。
  如需詳細資訊，請參閱[此頁面](https://aka.ms/csspeech/breakingchanges_1_0_0)。

## <a name="cognitive-services-speech-sdk-060-2018-august-release"></a>認知服務語音 SDK 0.6.0：2018 年 8 月版

**新功能**

* 使用語音 SDK 所建置的 UWP 應用程式現在可以通過 Windows 應用程式認證套件 (WACK) 的認證。
  請參閱 [UWP 快速入門](quickstart-csharp-uwp.md)。
* 在 Linux (Ubuntu 16.04 x64) 上支援 .NET Standard 2.0。
* 實驗性：在 Windows (64 位元) 和 Linux (Ubuntu 16.04 x64) 上支援 Java 8。
  請參閱 [Java 執行階段環境快速入門](quickstart-java-jre.md)。

**功能性變更**

* 公開關於連線錯誤的其他錯誤詳細資訊。

**重大變更**

* 在 Java (Android) 上，`SpeechFactory.configureNativePlatformBindingWithDefaultCertificate` 函式不再需要路徑參數。 系統現在會自動在所有支援平台上偵測路徑。
* Java 和 C# 中 `EndpointUrl` 屬性的 get-accessor 已移除。

**錯誤修正**

* 在 Java 中，現已實作翻譯辨識器上的音訊合成結果。
* 已修正錯誤，此錯誤可能導致非作用中的執行緒，並增加已開啟且未使用的通訊端數目。
* 已修正問題，長時間執行的辨識可能會在傳輸途中終止。
* 已修正辨識器關機中的競爭條件。

## <a name="cognitive-services-speech-sdk-050-2018-july-release"></a>認知服務語音 SDK 0.5.0：2018 年 7 月版

**新功能**

* 支援 Android 平台 (API 23: Android 6.0 Marshmallow 或更新版本)。 請參閱 [Android 快速入門](quickstart-java-android.md)。
* 在 Windows 上支援 .NET Standard 2.0。 請參閱 [.NET Core 快速入門](quickstart-csharp-dotnetcore-windows.md)。
* 實驗性：在 Windows 上支援 UWP (1709 版或更新版本)。
  * 請參閱 [UWP 快速入門](quickstart-csharp-uwp.md)。
  * 注意：使用語音 SDK 建置的 UWP 應用程式尚未通過 Windows 應用程式認證套件 (WACK) 的認證。
* 使用自動重新連線支援長時間執行辨識。

**功能性變更**

* `StartContinuousRecognitionAsync()` 支援長時間執行辨識。
* 辨識結果包含更多欄位。 它們從已辨識文字的音訊開頭和持續時間 (兩者都以刻度為單位)，以及表示辨識狀態的其他值位移，例如 `InitialSilenceTimeout` 和 `InitialBabbleTimeout`。
* 支援 AuthorizationToken 以建立處理站執行個體。

**重大變更**

* 辨識事件：NoMatch 事件類型已合併到錯誤事件中。
* C# 中的 SpeechOutputFormat 已重新命名為 OutputFormat，以便與 C++ 保持一致。
* `AudioInputStream` 介面部分方法的傳回型別已稍微變更：
   * 在 Java 中，`read` 方法現在會傳回 `long` 而不是 `int`。
   * 在 C# 中，`Read` 方法現在會傳回 `uint` 而不是 `int`。
   * 在 C++ 中，`Read` 和 `GetFormat` 方法現在會傳回 `size_t` 而不是 `int`。
* C++：音訊輸入串流的執行個體現在只能當作 `shared_ptr` 傳遞。

**錯誤修正**

* 修正 `RecognizeAsync()` 逾時的時候，結果中不正確的傳回值。
* 已移除 Windows 上媒體基礎程式庫的相依性。 SDK 現在使用核心音訊 API。
* 文件修正：已新增[區域](regions.md)頁面來描述支援的區域。

**已知問題**

* 適用於 Android 的語音 SDK 不會回報用於翻譯的語音合成結果。 此問題將在下一版中修正。

## <a name="cognitive-services-speech-sdk-040-2018-june-release"></a>認知服務語音 SDK 0.4.0：2018 年 6 月版本

**功能性變更**

- AudioInputStream

  辨識器現在可以取用資料流作為音訊來源。 如需詳細資訊，請參閱相關的[操作說明指南](how-to-use-audio-input-streams.md)。

- 詳細的輸出格式

  建立 `SpeechRecognizer` 時，您可以要求 `Detailed` 或 `Simple` 輸出格式。 `DetailedSpeechRecognitionResult` 包含信賴分數、已辨識的文字、未經處理的語彙形式，以及標準化形式和加上遮罩的粗話。

**重大變更**

- 在 C# 中已從 `SpeechRecognitionResult.RecognizedText` 變更為 `SpeechRecognitionResult.Text`。

**錯誤修正**

- 已修正在關閉期間 USP 層中可能發生的回呼問題。

- 如果辨識器取用音訊輸入檔，其保留檔案控制代碼超過所需的時間。

- 已移除訊息幫浦與辨識器之間的數個死結。

- 如果來自服務的回應逾時，則引發 `NoMatch`。

- Windows 上的媒體基礎程式庫延遲載入。 只有麥克風輸入需要此程式庫。

- 音訊資料的上傳速度受限於大約兩倍的原始音訊速度。

- 在 Windows 上，C# .NET 組件現在使用強式名稱。

- 文件修正：`Region` 是建立辨識器時所需的資訊。

已新增更多範例，且持續更新。 如需最新的範例集，請參閱[語音 SDK 範例 GitHub 存放庫](https://aka.ms/csspeech/samples) \(英文\)。

## <a name="cognitive-services-speech-sdk-0212733-2018-may-release"></a>認知服務語音 0.2.12733：2018 年 5 月版本

此版本為認知服務語音 SDK 的第一個公開預覽版本。
