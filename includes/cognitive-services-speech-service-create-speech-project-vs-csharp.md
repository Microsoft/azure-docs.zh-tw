---
author: wolfma61
ms.service: cognitive-services
ms.topic: include
ms.date: 09/13/2018
ms.author: wolfma
ms.openlocfilehash: 8ff8e8341b6f39f66c2cc8014d41d3d3a2918d2b
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47454498"
---
1. 啟動 Visual Studio 2017。
 
1. 請確定 **.NET 桌面環境**工作負載可用。 從 Visual Studio 功能表列選擇 [工具] \> [取得工具和功能]，以開啟 Visual Studio 安裝程式。 如果已啟用此工作負載，請關閉對話方塊。 

    否則，勾選 [.NET 桌面開發] 旁的核取方塊，然後按一下對話方塊右下角的 [修改] 按鈕。 安裝新功能需要一些時間。

    ![啟用 .NET 桌面開發](~/articles/cognitive-services/speech-service/media/sdk/vs-enable-net-desktop-workload.png)

1. 建立新的 Visual C# 主控台應用程式。 在 [新增專案] 對話方塊中，從左窗格展開 [已安裝] \> [Visual C#] \> [Windows 桌面]，然後選擇 [主控台應用程式 (.NET Framework)]。 針對專案名稱，請輸入 *helloworld*。

    ![建立 Visual C# 主控台應用程式 (.NET Framework)](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-dotnet-windows-01-new-console-app.png "建立 Visual C# 主控台應用程式 (.NET Framework)")

1. 安裝並參考 [Speech SDK NuGet 封裝](https://aka.ms/csspeech/nuget)。 在 [方案總管] 中，以滑鼠右鍵按一下解決方案，然後選取 [管理解決方案的 NuGet 套件]。

    ![以滑鼠右鍵按一下解決方案的管理 NuGet 封裝](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-dotnet-windows-02-manage-nuget-packages.png "管理解決方案的 NuGet 封裝")

1. 在右上角的 [套件來源] 欄位中，選取 [Nuget.org]。搜尋 `Microsoft.CognitiveServices.Speech` 套件，然後將它安裝到 **helloworld** 專案。

    ![安裝 Microsoft.CognitiveServices.Speech NuGet 套件](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-dotnet-windows-03-nuget-install-1.0.0.png "安裝 NuGet 套件")

1. 接受顯示的授權，才會開始安裝 NuGet 套件。

    ![接受授權](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-dotnet-windows-04-nuget-license.png "接受授權")

    安裝套件之後，[套件管理員] 主控台上會出現確認訊息。

1. 透過 [組態管理員] 建立符合您電腦架構的平台組態。 選取 [建置] >  [組態管理員]。

    ![啟動組態管理員](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-dotnet-windows-05-cfg-manager-click.png "啟動組態管理員")

1. 在 [組態管理員] 對話方塊中，新增平台。 從 [使用中的方案平台] 下拉式清單中，選取 [新增]。

    ![在 [組態管理員] 視窗下方新增平台](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-dotnet-windows-06-cfg-manager-new.png "在 [組態管理員] 視窗下方新增平台")

1. 如果您執行的是 64 位元 Windows，請建立一個名為 `x64` 的新平台設定。 如果您執行的是32 位元 Windows，請建立一個名為 `x86` 的新平台設定。

    ![在 64 位元 Windows 上新增名為「x64」的平台](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-dotnet-windows-07-cfg-manager-add-x64.png "新增 x64 平台")


