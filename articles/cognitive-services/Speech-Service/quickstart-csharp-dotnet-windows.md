---
title: 快速入門：使用認知服務語音 SDK 在 Windows 上根據 .NET Framework 以 C# 辨識語音
titleSuffix: Microsoft Cognitive Services
description: 了解如何使用認知服務語音 SDK 在 Windows 上根據 .NET Framework 以 C# 辨識語音
services: cognitive-services
author: wolfma61
ms.service: cognitive-services
ms.component: speech-service
ms.topic: quickstart
ms.date: 09/24/2018
ms.author: wolfma
ms.openlocfilehash: 32b484451c4ee2264c25cca92b1d03d91b955a29
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47053991"
---
# <a name="quickstart-recognize-speech-in-c-under-net-framework-on-windows-by-using-the-speech-sdk"></a>快速入門：使用語音 SDK 在 Windows 上根據 .NET Framework 以 C# 辨識語音

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

在本文中，您會使用[語音 SDK](speech-sdk.md)，在 Windows 上建立適用於 .NET Framework 的 C# 主控台應用程式。 您從電腦的麥克風即時將語音轉譯為文字。 應用程式是利用[語音 SDK NuGet 套件](https://aka.ms/csspeech/nuget)與 Microsoft Visual Studio 2017 (任何版本) 所建置的。

## <a name="prerequisites"></a>必要條件

您需要語音服務訂用帳戶金鑰，才能完成本快速入門。 您可以免費取得一個金鑰。 如需詳細資訊，請參閱[免費試用語音服務](get-started.md)。

## <a name="create-a-visual-studio-project"></a>建立 Visual Studio 專案

[!INCLUDE [Create project ](../../../includes/cognitive-services-speech-service-create-speech-project-vs-csharp.md)]

## <a name="add-sample-code"></a>新增範例程式碼

1. 開啟 `Program.cs` 並將其中的所有程式碼取代為下列程式碼。

    [!code-csharp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/csharp-dotnet-windows/helloworld/Program.cs#code)]

1. 在相同檔案中，以您的語音服務訂用帳戶金鑰取代 `YourSubscriptionKey` 字串。

1. 同時以與您的訂用帳戶 (例如，免費試用訂用帳戶的 `westus`) 相關聯的[區域](regions.md)取代 `YourServiceRegion` 字串。

1. 儲存專案的變更。

## <a name="build-and-run-the-app"></a>建置並執行應用程式

1. 建置應用程式。 從功能表列中，選取 [建置]  >  [建置解決方案]。 現在會編譯程式碼，而且不會出現任何錯誤。

    ![Visual Studio 應用程式的螢幕擷取畫面，其中 [建置解決方案] 選項已醒目提示](media/sdk/qs-csharp-dotnet-windows-08-build.png "成功建置")

1. 啟動應用程式。 從功能表列中，選取 [偵錯]  >  [開始偵錯]，或按 **F5**。

    ![Visual Studio 應用程式的螢幕擷取畫面，其中 [開始偵錯] 選項已醒目提示](media/sdk/qs-csharp-dotnet-windows-09-start-debugging.png "將應用程式啟動至偵錯")

1. 一個主控台視窗隨即顯示，提示您說話。 請說英文片語或句子。 您的語音會傳送到語音服務，並且轉譯為文字，該文字會出現在相同視窗中。

    ![成功辨識後主控台輸出的螢幕擷取畫面](media/sdk/qs-csharp-dotnet-windows-10-console-output.png "成功辨識後的主控台輸出")

[!INCLUDE [Download this sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]
在 `quickstart/csharp-dotnet-windows` 資料夾中尋找此範例。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用適用於 C# 的語音 SDK 從語音辨識意圖](how-to-recognize-intents-from-speech-csharp.md)

## <a name="see-also"></a>另請參閱

- [轉譯語音](how-to-translate-speech-csharp.md)
- [自訂原音模型](how-to-customize-acoustic-models.md)
- [自訂語言模型](how-to-customize-language-model.md)
