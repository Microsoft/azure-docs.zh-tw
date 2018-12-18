---
title: 快速入門：使用認知服務語音 SDK 在 UWP 應用程式中以 C# 辨識語音
titleSuffix: Microsoft Cognitive Services
description: 了解如何使用認知服務語音 SDK 在 UWP 應用程式中辨識語音
services: cognitive-services
author: wolfma61
ms.service: cognitive-services
ms.component: speech-service
ms.topic: quickstart
ms.date: 09/24/2018
ms.author: wolfma
ms.openlocfilehash: d9a90869e060d2f8f1a1c522a4528e74841caada
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339609"
---
# <a name="quickstart-recognize-speech-in-a-uwp-app-by-using-the-speech-sdk"></a>快速入門：使用語音 SDK 在 UWP 應用程式中辨識語音

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

在本文中，您會使用認知服務[語音 SDK](speech-sdk.md)來建立 C# 通用 Windows 平台 (UWP) 應用程式。 您會將來自裝置麥克風的語音即時轉譯為文字。 應用程式是利用[語音 SDK NuGet 套件](https://aka.ms/csspeech/nuget)與 Microsoft Visual Studio 2017 (任何版本) 所建置的。

> [!NOTE]
> 通用 Windows 平台可讓您開發應用程式，以在任何支援 Windows 10 的裝置 (包括 PC、Xbox、Surface Hub 與其他裝置) 上執行。

## <a name="prerequisites"></a>必要條件

您需要語音服務訂用帳戶金鑰，才能完成本快速入門。 您可以免費取得一個金鑰。 如需詳細資訊，請參閱[免費試用語音服務](get-started.md)。

## <a name="create-a-visual-studio-project"></a>建立 Visual Studio 專案

1. 啟動 Visual Studio 2017。

1. 確定可以使用**通用 Windows 平台開發**工作負載。 從 Visual Studio 功能表列選擇 [工具] > [取得工具和功能]，以開啟 Visual Studio 安裝程式。 如果已啟用此工作負載，請關閉對話方塊。 

    ![Visual Studio 安裝程式的螢幕擷取畫面，其中 [工作負載] 索引標籤已醒目提示](media/sdk/vs-enable-uwp-workload.png)

    否則，請選取 [.NET 跨平台開發] 旁邊的方塊，然後選取對話方塊右下角的 [修改]。 安裝新功能需要一些時間。

1. 建立空的 Visual C# 通用 Windows 應用程式。 首先，從功能表中選擇 [檔案] > [新增] > [專案]。 在 [新增專案] 對話方塊中，展開左窗格中的 [已安裝] > [Visual C#] > [Windows Universal]。 然後，選取 [空白應用程式 (通用 Windows)]。 針對專案名稱，請輸入 *helloworld*。

    ![[新增專案] 對話方塊的螢幕擷取畫面](media/sdk/qs-csharp-uwp-01-new-blank-app.png)

1. 要使用 Speed SDK，您必須為 Windows 10 Fall Creators Update 或更新版本建置您的應用程式。 在彈出的 [新增通用 Windows 平台專案] 視窗中，選擇 [Windows 10 Fall Creators Update (10.0；組建 16299)] 作為 [最低版本]。 在 [目標版本] 方塊中選取此版本或任何更新版本，然後按一下 [確定]。

    ![新增通用 Windows 平台專案視窗的螢幕擷取畫面](media/sdk/qs-csharp-uwp-02-new-uwp-project.png)

1. 如果您執行 64 位元的 Windows，您可以使用 Visual Studio 工具列中的下拉式功能表，將您的建置平台切換為 `x64`。 (64 位元 Windows 可執行 32 位元應用程式，因此您可以依需求將其保留為 `x86`。)

   ![此螢幕擷取畫面顯示將 x64 醒目提示的 Visual Studio 工具列](media/sdk/qs-csharp-uwp-03-switch-to-x64.png)

   > [!NOTE]
   > 語音 SDK 僅支援 Intel 相容處理器。 目前不支援 ARM。

1. 安裝並參考 [Speech SDK NuGet 封裝](https://aka.ms/csspeech/nuget)。 在 [方案總管] 中，以滑鼠右鍵按一下解決方案，然後選取 [管理解決方案的 NuGet 套件]。

    ![方案總管的螢幕擷取畫面，其中 [管理解決方案的 NuGet 套件] 選項已醒目提示](media/sdk/qs-csharp-uwp-04-manage-nuget-packages.png)

1. 在右上角的 [套件來源] 欄位中，選取 [nuget.org]。搜尋 `Microsoft.CognitiveServices.Speech` 套件，然後將它安裝到 **helloworld** 專案。

    ![管理解決方案套件對話方塊的螢幕擷取畫面](media/sdk/qs-csharp-uwp-05-nuget-install-1.0.0.png "安裝 NuGet 套件")

1. 接受顯示的授權，才會開始安裝 NuGet 套件。

    ![接受授權對話方塊的螢幕擷取畫面](media/sdk/qs-csharp-uwp-06-nuget-license.png "接受授權")

1. 下列輸出行會出現在套件管理器主控台中。

   ```text
   Successfully installed 'Microsoft.CognitiveServices.Speech 1.0.1' to helloworld
   ```

1. 由於應用程式使用麥克風進行語音輸入，請將 [麥克風] 功能新增至專案。 在 [方案總管] 中，按兩下 [Package.appxmanifest] 以編輯您的應用程式資訊清單。 接著，切換至 [功能] 索引標籤，選取 [麥克風] 功能的方塊，然後儲存您的變更。

   ![此螢幕擷取畫面顯示將功能和麥克風醒目提示的 Visual Studio 應用程式資訊清單](media/sdk/qs-csharp-uwp-07-capabilities.png)


## <a name="add-sample-code"></a>新增範例程式碼

1. 應用程式的使用者介面可使用 XAML 來定義。 在方案總管中開啟 `MainPage.xaml`。 在設計工具的 XAML 檢視中，將下列 XAML 程式碼片段插入格線標記 (在 `<Grid>` 和 `</Grid>` 之間)。

   [!code-xml[UI elements](~/samples-cognitive-services-speech-sdk/quickstart/csharp-uwp/helloworld/MainPage.xaml#StackPanel)]

1. 開啟程式碼後置原始程式檔 `MainPage.xaml.cs` (它會分組於 `MainPage.xaml` 下)。 將其中的所有程式碼取代為下列內容。

   [!code-csharp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/csharp-uwp/helloworld/MainPage.xaml.cs#code)]

1. 在此檔案的 `SpeechRecognitionFromMicrophone_ButtonClicked` 處理常式中，以您的訂用帳戶金鑰取代 `YourSubscriptionKey` 字串。

1. 在 `SpeechRecognitionFromMicrophone_ButtonClicked` 處理常式中，以和您的訂用帳戶 相關聯的[區域](regions.md) (例如，免費試用訂用帳戶的 `westus`) 取代 `YourServiceRegion` 字串。

1. 儲存專案的所有變更。

## <a name="build-and-run-the-app"></a>建置並執行應用程式

1. 建置應用程式。 從功能表列中，選取 [建置]  >  [建置解決方案]。 現在會編譯程式碼，而且不會出現任何錯誤。

    ![Visual Studio 應用程式的螢幕擷取畫面，其中 [建置解決方案] 選項已醒目提示](media/sdk/qs-csharp-uwp-08-build.png "成功建置")

1. 啟動應用程式。 從功能表列中，選取 [偵錯]  >  [開始偵錯]，或按 **F5**。

    ![Visual Studio 應用程式的螢幕擷取畫面，其中 [開始偵錯] 選項已醒目提示](media/sdk/qs-csharp-uwp-09-start-debugging.png "啟動應用程式並進行偵錯")

1. 此時會出現快顯視窗。 選取 [啟用麥克風]，並確認彈出的權限要求。

    ![權限要求的螢幕擷取畫面](media/sdk/qs-csharp-uwp-10-access-prompt.png "啟動應用程式進入偵錯模式")

1. 選取 [使用麥克風輸入的語音辨識]，然後對裝置的麥克風說出英文片語或句子。 您的語音會傳送到語音服務，並且轉譯為文字，出現在視窗中。

    ![語音辨識使用者介面的螢幕擷取畫面](media/sdk/qs-csharp-uwp-11-ui-result.png)

[!INCLUDE [Download this sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]
在 `quickstart/csharp-uwp` 資料夾中尋找此範例。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用適用於 C# 的語音 SDK 從語音辨識意圖](how-to-recognize-intents-from-speech-csharp.md)

## <a name="see-also"></a>另請參閱

- [轉譯語音](how-to-translate-speech-csharp.md)
- [自訂原音模型](how-to-customize-acoustic-models.md)
- [自訂語言模型](how-to-customize-language-model.md)
