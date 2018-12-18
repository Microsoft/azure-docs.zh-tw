---
title: 快速入門：使用認知服務語音 SDK 在 Windows 上根據 .NET Core 以 C# 辨識語音
titleSuffix: Microsoft Cognitive Services
description: 了解如何使用認知服務語音 SDK 在 Windows 上根據 .NET Core 以 C# 辨識語音
services: cognitive-services
author: wolfma61
ms.service: cognitive-services
ms.component: speech-service
ms.topic: quickstart
ms.date: 10/12/2018
ms.author: wolfma
ms.openlocfilehash: 04cdfd487036e79eccbb4fb879902a12e077d104
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49341645"
---
# <a name="quickstart-recognize-speech-in-c-under-net-core-on-windows-by-using-the-speech-sdk"></a>快速入門：使用語音 SDK 在 Windows 上根據 .NET Core 以 C# 辨識語音

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

在本文中，您會透過使用認知服務[語音 SDK](speech-sdk.md)，在 Windows 上的 .NET Core 建立 C# 主控台應用程式。 您將來自電腦麥克風的語音即時轉譯為文字。 應用程式是利用[語音 SDK NuGet 套件](https://aka.ms/csspeech/nuget)與 Microsoft Visual Studio 2017 (任何版本) 所建置的。

> [!NOTE]
> .NET Core 是實作[.NET Standard](https://docs.microsoft.com/dotnet/standard/net-standard) 規格的開放原始碼、跨平台 .NET 平台。

您需要語音服務訂用帳戶金鑰，才能完成本快速入門。 您可以免費取得一個金鑰。 如需詳細資訊，請參閱[免費試用語音服務](get-started.md)。


## <a name="create-a-visual-studio-project"></a>建立 Visual Studio 專案

1. 啟動 Visual Studio 2017。

1. 請確定 **.NET 跨平台開發**工作負載可用。 從 Visual Studio 功能表列選擇 [工具]  >  [取得工具和功能]，以開啟 Visual Studio 安裝程式。 如果已啟用此工作負載，請關閉對話方塊。

    ![Visual Studio 安裝程式的螢幕擷取畫面，其中 [工作負載] 索引標籤已醒目提示](media/sdk/vs-enable-net-core-workload.png)

    否則，選取 [.NET Core 跨平台開發] 旁邊的方塊，然後選取對話方塊右下角的 [修改]。 安裝新功能需要一些時間。

1. 建立新的 Visual C# .NET Core 主控台應用程式。 在 [新增專案] 對話方塊中，從左窗格展開 [已安裝]  >  [Visual C#]  >  [.NET Core]。 然後選取 [主控台應用程式 (.NET Core)]。 針對專案名稱，請輸入 *helloworld*。

    ![[新增專案] 對話方塊的螢幕擷取畫面](media/sdk/qs-csharp-dotnetcore-windows-01-new-console-app.png "建立 Visual C# 主控台應用程式 (.NET Core)")

1. 安裝並參考 [Speech SDK NuGet 封裝](https://aka.ms/csspeech/nuget)。 在 [方案總管] 中，以滑鼠右鍵按一下解決方案，然後選取 [管理解決方案的 NuGet 套件]。

    ![方案總管的螢幕擷取畫面，其中 [管理解決方案的 NuGet 套件] 選項已醒目提示](media/sdk/qs-csharp-dotnetcore-windows-02-manage-nuget-packages.png "管理解決方案的 NuGet 套件")

1. 在右上角的 [套件來源] 欄位中，選取 [nuget.org]。搜尋 `Microsoft.CognitiveServices.Speech` 套件，然後將它安裝到 **helloworld** 專案。

    ![管理解決方案套件對話方塊的螢幕擷取畫面](media/sdk/qs-csharp-dotnetcore-windows-03-nuget-install-1.0.0.png "安裝 NuGet 套件")

1. 接受顯示的授權，才會開始安裝 NuGet 套件。

    ![接受授權對話方塊的螢幕擷取畫面](media/sdk/qs-csharp-dotnetcore-windows-04-nuget-license.png "接受授權")

安裝套件之後，[套件管理員] 主控台上會出現確認訊息。


## <a name="add-sample-code"></a>新增範例程式碼

1. 開啟 `Program.cs` 並將其中的所有程式碼取代為下列程式碼。

    [!code-csharp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/csharp-dotnetcore/helloworld/Program.cs#code)]

1. 在相同檔案中，以您的訂用帳戶金鑰取代 `YourSubscriptionKey` 字串。

1. 同時以與您的訂用帳戶 (例如，免費試用訂用帳戶的 `westus`) 相關聯的[區域](regions.md)取代 `YourServiceRegion` 字串。

1. 儲存專案的變更。

## <a name="build-and-run-the-app"></a>建置並執行應用程式

1. 建置應用程式。 從功能表列中，選擇 [建置] > [建置解決方案]。 這應該會編譯程式碼，而且不會出現任何錯誤。

    ![Visual Studio 應用程式的螢幕擷取畫面，其中 [建置解決方案] 選項已醒目提示](media/sdk/qs-csharp-dotnetcore-windows-05-build.png "成功建置")

1. 啟動應用程式。 從功能表列中，選擇 [偵錯] > [開始偵錯]，或按 **F5**。

    ![Visual Studio 應用程式的螢幕擷取畫面，其中 [開始偵錯] 選項已醒目提示](media/sdk/qs-csharp-dotnetcore-windows-06-start-debugging.png "啟動應用程式並進行偵錯")

1. 一個主控台視窗隨即顯示，提示您說話。 請說英文片語或句子。 您的語音會傳送到語音服務，並且轉譯為文字，該文字會出現在相同視窗中。

    ![成功辨識後主控台輸出的螢幕擷取畫面](media/sdk/qs-csharp-dotnetcore-windows-07-console-output.png "成功辨識後的主控台輸出")

[!INCLUDE [Download this sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]
在 `quickstart/csharp-dotnetcore-windows` 資料夾中尋找此範例。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用適用於 C# 的語音 SDK 從語音辨識意圖](how-to-recognize-intents-from-speech-csharp.md)

## <a name="see-also"></a>另請參閱

- [轉譯語音](how-to-translate-speech-csharp.md)
- [自訂原音模型](how-to-customize-acoustic-models.md)
- [自訂語言模型](how-to-customize-language-model.md)
