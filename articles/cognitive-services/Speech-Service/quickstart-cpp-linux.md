---
title: 快速入門：使用認知服務語音 SDK 在 Linux 上以 C++ 辨識語音
titleSuffix: Microsoft Cognitive Services
description: 了解如何使用認知服務語音 SDK 在 Linux 上以 C++ 辨識語音
services: cognitive-services
author: wolfma61
ms.service: cognitive-services
ms.component: Speech
ms.topic: quickstart
ms.date: 10/12/2018
ms.author: wolfma
ms.openlocfilehash: f23e25ec117d798bae81945104fa3fb53f320f1c
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49344078"
---
# <a name="quickstart-recognize-speech-in-c-on-linux-by-using-the-speech-sdk"></a>快速入門：使用語音 SDK 在 Linux 上以 C++ 辨識語音

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

在本文中，您會建立適用於 Ubuntu Linux 16.04 的 C++ 主控台應用程式。 您使用認知服務[語音 SDK](speech-sdk.md)，即時從電腦的麥克風將語音轉譯為文字。 此應用程式是使用[適用於 Linux 的語音 SDK](https://aka.ms/csspeech/linuxbinary) 和 Linux 散發套件的 C++ 編譯器 (例如 `g++`) 所建置。

## <a name="prerequisites"></a>必要條件

您需要語音服務訂用帳戶金鑰，才能完成本快速入門。 您可以免費取得一個金鑰。 如需詳細資訊，請參閱[免費試用語音服務](get-started.md)。

## <a name="install-speech-sdk"></a>安裝語音 SDK

[!INCLUDE [License Notice](../../../includes/cognitive-services-speech-service-license-notice.md)]

認知服務語音 SDK 目前的版本為 `1.0.1`。

適用於 Linux 的語音 SDK 可用來建置 64 位元和 32 位元應用程式。 您可從 https://aka.ms/csspeech/linuxbinary 以 tar 檔案格式下載必要的程式庫和標頭檔。

下載並安裝 SDK，如下所示：

1. 請確定已安裝 SDK 的相依性。

   ```sh
   sudo apt-get update
   sudo apt-get install build-essential libssl1.0.0 libcurl3 libasound2 wget
   ```

1. 選擇語音 SDK 檔案應解壓縮至的目錄，並將 `SPEECHSDK_ROOT` 環境變數設定為指向該目錄。 此變數方便在未來命令中參考此目錄。 例如，如果您想要在主目錄中使用目錄 `speechsdk`，請使用類似下列的命令：

   ```sh
   export SPEECHSDK_ROOT="$HOME/speechsdk"
   ```

1. 在該目錄不存在的情況下建立它。

   ```sh
   mkdir -p "$SPEECHSDK_ROOT"
   ```

1. 下載並解壓縮包含語音 SDK 二進位檔的 `.tar.gz` 封存：

   ```sh
   wget -O SpeechSDK-Linux.tar.gz https://aka.ms/csspeech/linuxbinary
   tar --strip 1 -xzf SpeechSDK-Linux.tar.gz -C "$SPEECHSDK_ROOT"
   ```

1. 驗證已壓縮套件之最上層目錄的內容：

   ```sh
   ls -l "$SPEECHSDK_ROOT"
   ```

   目錄清單應包含第三方聲明和授權檔案，以及包含標頭 (`.h`) 檔案的 `include` 目錄和包含程式庫的 `lib` 目錄。

   [!INCLUDE [Linux Binary Archive Content](../../../includes/cognitive-services-speech-service-linuxbinary-content.md)]

## <a name="add-sample-code"></a>新增範例程式碼

1. 建立名為 `helloworld.cpp` 的 C++ 原始程式檔，然後在其中貼上下列程式碼。

   [!code-cpp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/cpp-linux/helloworld.cpp#code)]

1. 在這個新檔案中，以您的語音服務訂用帳戶金鑰取代 `YourSubscriptionKey` 字串。

1. 以與您的訂用帳戶 (例如，免費試用訂用帳戶的 `westus`) 相關聯的[區域](regions.md)取代 `YourServiceRegion` 字串。

## <a name="build-the-app"></a>建置應用程式

> [!NOTE]
> 請務必以「單一命令行」的形式輸入下面的命令。 若要這麼做，最簡單的方式就是使用每個命令旁邊的 [複製] 按鈕來複製命令，然後將它貼在殼層提示字元。

* 在 **x64** (64 位元) 系統上，執行下列命令以建置應用程式。

  ```sh
  g++ helloworld.cpp -o helloworld -I "$SPEECHSDK_ROOT/include/cxx_api" -I "$SPEECHSDK_ROOT/include/c_api" --std=c++14 -lpthread -lMicrosoft.CognitiveServices.Speech.core -L "$SPEECHSDK_ROOT/lib/x64" -l:libssl.so.1.0.0 -l:libcurl.so.4 -l:libasound.so.2
  ```

* 在 **x86** (32 位元) 系統上，執行下列命令以建置應用程式。

  ```sh
  g++ helloworld.cpp -o helloworld -I "$SPEECHSDK_ROOT/include/cxx_api" -I "$SPEECHSDK_ROOT/include/c_api" --std=c++14 -lpthread -lMicrosoft.CognitiveServices.Speech.core -L "$SPEECHSDK_ROOT/lib/x86" -l:libssl.so.1.0.0 -l:libcurl.so.4 -l:libasound.so.2
  ```

## <a name="run-the-app"></a>執行應用程式

1. 將載入程式的程式庫路徑設定為指向語音 SDK 程式庫。

   * 在 **x64** (64 位元) 系統上，輸入下列命令。

     ```sh
     export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$SPEECHSDK_ROOT/lib/x64"
     ```

   * 在 **x86** (32 位元) 系統上，輸入此命令。

     ```sh
     export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$SPEECHSDK_ROOT/lib/x86"
     ```

1. 執行應用程式。

   ```sh
   ./helloworld
   ```

1.  主控台視窗會出現提示，要求您說一些話。 請說英文片語或句子。 您的語音會傳送到語音服務，並且轉譯為文字，該文字會出現在相同視窗中。

   ```text
   Say something...
   We recognized: What's the weather like?
   ```

[!INCLUDE [Download this sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]
在 `quickstart/cpp-linux` 資料夾中尋找此範例。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用適用於 C++ 的語音 SDK 從語音辨識意圖](how-to-recognize-intents-from-speech-cpp.md)

## <a name="see-also"></a>另請參閱

- [轉譯語音](how-to-translate-speech-csharp.md)
- [自訂原音模型](how-to-customize-acoustic-models.md)
- [自訂語言模型](how-to-customize-language-model.md)
