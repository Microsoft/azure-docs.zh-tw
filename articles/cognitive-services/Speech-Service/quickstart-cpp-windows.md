---
title: 快速入門：使用認知服務語音 SDK 在 Windows 上以 C++ 辨識語音
titleSuffix: Microsoft Cognitive Services
description: 了解如何使用認知服務語音 SDK 在 Windows 桌面上以 C++ 辨識語音
services: cognitive-services
author: wolfma61
ms.service: cognitive-services
ms.component: Speech
ms.topic: quickstart
ms.date: 10/12/2018
ms.author: wolfma
ms.openlocfilehash: 2adad78760c1d12da688106c45e86c91a8b13f91
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339350"
---
# <a name="quickstart-recognize-speech-in-c-on-windows-by-using-the-speech-sdk"></a>快速入門：使用語音 SDK 在 Windows 上以 C++ 辨識語音

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

在本文中，您會建立適用於 Windows 的 C++ 主控台應用程式。 您將使用認知服務[語音 SDK](speech-sdk.md)，即時從電腦的麥克風將語音轉譯為文字。 應用程式是利用[語音 SDK NuGet 套件](https://aka.ms/csspeech/nuget)與 Microsoft Visual Studio 2017 (任何版本) 所建置的。

## <a name="prerequisites"></a>必要條件

您需要語音服務訂用帳戶金鑰，才能完成本快速入門。 您可以免費取得一個金鑰。 如需詳細資訊，請參閱[免費試用語音服務](get-started.md)。

## <a name="create-a-visual-studio-project"></a>建立 Visual Studio 專案

1. 啟動 Visual Studio 2017。

1. 請確定**使用 C++ 進行桌面開發**工作負載可供使用。 從 Visual Studio 功能表列選擇 [工具]  >  [取得工具和功能]，以開啟 Visual Studio 安裝程式。 如果已啟用此工作負載，請跳至下一個步驟。 

    ![Visual Studio 工作負載索引標籤的螢幕擷取畫面](media/sdk/vs-enable-cpp-workload.png)

    否則，請核取 [使用 C++ 進行桌面開發] 旁邊的方塊。 

1. 請確定 **NuGet 套件管理員**元件可用。 切換至 Visual Studio 安裝程式對話方塊的 [個別元件] 索引標籤，如果 [NuGet 套件管理員] 尚未啟用，則加以選取。

      ![Visual Studio 個別元件索引標籤的螢幕擷取畫面](media/sdk/vs-enable-nuget-package-manager.png)

1. 如果您需要啟用 C++ 工作負載或 NuGet，請選取 [修改] (在對話方塊的右下角)。 安裝新功能需要一些時間。 如果這兩項功能都已經啟用，則關閉對話方塊。

1. 建立新 Visual C++ Windows 桌面的 Windows 主控台應用程式。 首先，從功能表選擇 [檔案]  >  [新增]  >  [專案]。 在 [新增專案] 對話方塊中，展開左窗格中的 [已安裝]  >  [Visual C++]  >  [Windows 桌面]。 然後選取 [ Windows 主控台應用程式]。 針對專案名稱，請輸入 *helloworld*。

    ![[新增專案] 對話方塊的螢幕擷取畫面](media/sdk/qs-cpp-windows-01-new-console-app.png)

1. 如果您執行 64 位元的 Windows，可以使用 Visual Studio 工具列中的下拉式功能表，將您的建置平台切換為 `x64`。 (64 位元版本的 Windows 可以執行 32 位元應用程式，因此您不一定需要進行此動作。)

    ![Visual Studio 工具列的螢幕擷取畫面，其中 x64 選項已醒目提示](media/sdk/qs-cpp-windows-02-switch-to-x64.png)

1. 在 [方案總管] 中，以滑鼠右鍵按一下解決方案，然後選擇 [管理解決方案的 NuGet 套件]。

    ![方案總管的螢幕擷取畫面，其中 [管理解決方案的 NuGet 套件] 選項已醒目提示](media/sdk/qs-cpp-windows-03-manage-nuget-packages.png)

1. 在右上角的 [套件來源] 欄位中，選取 [nuget.org]。搜尋 `Microsoft.CognitiveServices.Speech` 套件，然後將它安裝到 **helloworld** 專案。

    ![管理解決方案套件對話方塊的螢幕擷取畫面](media/sdk/qs-cpp-windows-04-nuget-install-1.0.0.png)

    > [!NOTE]
    > 認知服務語音 SDK 目前的版本為 `1.0.1`。

1. 接受顯示的授權，才會開始安裝 NuGet 套件。

    ![接受授權對話方塊的螢幕擷取畫面](media/sdk/qs-cpp-windows-05-nuget-license.png)

安裝套件之後，[套件管理員] 主控台上會出現確認訊息。

## <a name="add-sample-code"></a>新增範例程式碼

1. 開啟來源檔案 helloworld.cpp。 使用下列項目取代開頭 include 陳述式 (`#include "stdafx.h"` 或 `#include "pch.h"`) 下方的所有程式碼：

   [!code-cpp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/cpp-windows/helloworld/helloworld.cpp#code)]

1. 在相同檔案中，以您的訂用帳戶金鑰取代 `YourSubscriptionKey` 字串。

1. 以與您的訂用帳戶 (例如，免費試用訂用帳戶的 `westus`) 相關聯的[區域](regions.md)取代 `YourServiceRegion` 字串。

1. 儲存專案的變更。

## <a name="build-and-run-the-app"></a>建置並執行應用程式

1. 建置應用程式。 從功能表列中，選擇 [建置] > [建置解決方案]。 這應該會編譯程式碼，而且不會出現任何錯誤。

   ![Visual Studio 應用程式的螢幕擷取畫面，其中 [建置解決方案] 選項已醒目提示](media/sdk/qs-cpp-windows-06-build.png)

1. 啟動應用程式。 從功能表列中，選擇 [偵錯] > [開始偵錯]，或按 **F5**。

   ![Visual Studio 應用程式的螢幕擷取畫面，其中 [開始偵錯] 選項已醒目提示](media/sdk/qs-cpp-windows-07-start-debugging.png)

1. 一個主控台視窗隨即顯示，提示您說話。 請說英文片語或句子。 您的語音會傳送到語音服務，並且轉譯為文字，該文字會出現在相同視窗中。

   ![成功辨識後主控台輸出的螢幕擷取畫面](media/sdk/qs-cpp-windows-08-console-output-release.png)

[!INCLUDE [Download this sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]
在 `quickstart/cpp-windows` 資料夾中尋找此範例。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用適用於 C++ 的語音 SDK 從語音辨識意圖](how-to-recognize-intents-from-speech-cpp.md)

## <a name="see-also"></a>另請參閱

- [轉譯語音](how-to-translate-speech-csharp.md)
- [自訂原音模型](how-to-customize-acoustic-models.md)
- [自訂語言模型](how-to-customize-language-model.md)
