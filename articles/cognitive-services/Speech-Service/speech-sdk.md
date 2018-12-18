---
title: 關於認知服務語音 SDK
description: 提供語音服務的 SDK 概觀。
titleSuffix: Microsoft Cognitive Services
services: cognitive-services
author: v-jerkin
ms.service: cognitive-services
ms.component: speech-service
ms.topic: article
ms.date: 09/24/2018
ms.author: v-jerkin
ms.openlocfilehash: 169d421ddccf33ac239b69ab78ca7dca0f0b8261
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958408"
---
# <a name="about-the-cognitive-services-speech-sdk"></a>關於認知服務語音 SDK

認知服務語音軟體開發套件 (SDK) 提供您應用程式對語音服務功能的原生存取權，以更輕鬆地開發軟體。 目前，SDK 提供對**語音轉換文字**、**語音翻譯**和**意圖辨識**的存取權。

[!INCLUDE [Speech SDK Platforms](../../../includes/cognitive-services-speech-service-speech-sdk-platforms.md)]

[!INCLUDE [License Notice](../../../includes/cognitive-services-speech-service-license-notice.md)]

## <a name="get-the-sdk"></a>取得 SDK

### <a name="windows"></a>Windows

針對 Windows，我們支援下列語言：

* C# (UWP 和 .NET)、C++：您可以參考並使用語音 SDK NuGet 套件的最新版本。
  此套件包括 32 位元和 64 位元的程式庫，以及受控 (.NET) 程式庫。
  使用 NuGet，可以在 Visual Studio 中安裝 SDK；只要搜尋 `Microsoft.CognitiveServices.Speech`。

* Java：您可以參考並使用語音 SDK Maven 套件的最新版本，但其只支援 Windows x64。
  在 Maven 專案中，新增 `https://csspeechstorage.blob.core.windows.net/maven/` 作為額外的存放庫，並參考 `com.microsoft.cognitiveservices.speech:client-sdk:1.0.0` 作為相依性。 

### <a name="linux"></a>Linux

> [!NOTE]
> 目前，我們只支援 PC 上的 Ubuntu 16.04 (x86 或 x64 (適用於 C++ 開發)、x64 (適用於 .NET Core 和 Java))。

請執行下列殼層命令，確定您已安裝必要的編譯器和程式庫：

```sh
sudo apt-get update
sudo apt-get install build-essential libssl1.0.0 libcurl3 libasound2
```

* C#：您可以參考並使用語音 SDK NuGet 套件的最新版本。
  若要參考 SDK，請在專案中新增下列套件參考：

  ```xml
  <PackageReference Include="Microsoft.CognitiveServices.Speech" Version="1.0.0" />
  ```

* Java：您可以參考並使用語音 SDK Maven 套件的最新版本。
  在 Maven 專案中，新增 `https://csspeechstorage.blob.core.windows.net/maven/` 作為額外的存放庫，並參考 `com.microsoft.cognitiveservices.speech:client-sdk:1.0.0` 作為相依性。 

* C++：下載 [.tar 套件](https://aka.ms/csspeech/linuxbinary)形式的 SDK，並將其中的檔案解壓縮至您選擇的目錄中。 下表顯示 SDK 資料夾結構。

  |Path|說明|
  |-|-|
  |`license.md`|授權|
  |`ThirdPartyNotices.md`|協力廠商通知|
  |`include`|C 和 C++ 的標頭檔|
  |`lib/x64`|與應用程式連結的原生 x64 程式庫|
  |`lib/x86`|與應用程式連結的原生 x86 程式庫|

  若要建立應用程式，請將所需的二進位檔 (和程式庫) 複製或移至您的開發環境，並將它們包含為建置程序的必要項目。

### <a name="android"></a>Android

適用於 Android 的 Java SDK 封裝成 [AAR (Android 程式庫)](https://developer.android.com/studio/projects/android-library)，其中包含必要的程式庫，以及使用它的必要 Android 權限。
它在 `https://csspeechstorage.blob.core.windows.net/maven/` 的 Maven 存放庫中以套件 `com.microsoft.cognitiveservices.speech:client-sdk:1.0.0` 的形式裝載。
若要從 Android Studio 專案取用套件，請進行下列變更：

* 在專案層級的 `build.gradle` 檔案中，將下列內容加入至 `repository` 區段：

  ```text
  maven { url 'https://csspeechstorage.blob.core.windows.net/maven/' }
  ```

* 在模組層級的 `build.gradle` 檔案中，將下列內容加入至 `dependencies` 區段：

  ```text
  implementation 'com.microsoft.cognitiveservices.speech:client-sdk:1.0.0'
  ```

Java SDK 也是[語音裝置 SDK](speech-devices-sdk.md) 的一部分。

[!INCLUDE [Get the samples](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]

## <a name="next-steps"></a>後續步驟

* [試用認知服務](https://azure.microsoft.com/try/cognitive-services/)
* [了解如何以 C# 辨識語音](quickstart-csharp-dotnet-windows.md) (英文)
