---
title: 教學課程 - 對在本機開發叢集中執行的 Azure Service Fabric Mesh 應用程式進行偵錯 | Microsoft Docs
description: 在本教學課程中，您可以對在本機叢集上執行的 Azure Service Fabric Mesh 應用程式進行偵錯。
services: service-fabric-mesh
documentationcenter: .net
author: TylerMSFT
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/18/2018
ms.author: twhitney
ms.custom: mvc, devcenter
ms.openlocfilehash: 27e4c8f6ac24d40a6afacf10175413745f5151d9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46997007"
---
# <a name="tutorial-debug-a-service-fabric-mesh-application-running-in-your-local-development-cluster"></a>教學課程：對在本機開發叢集中執行的 Service Fabric Mesh 應用程式進行偵錯

本教學課程是系列中的第二部分，說明如何在您的本機開發叢集上建置 Azure Service Fabric Mesh Web 應用程式，和執行其偵錯。

在本教學課程中，您將了解：

> [!div class="checklist"]
> * 當您建置 Azure Service Fabric Mesh 應用程式時，會有何情況
> * 如何設定中斷點以觀察服務對服務呼叫

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * [在 Visual Studio 中建立 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * 對在本機開發叢集中執行的 Service Fabric Mesh 應用程式進行偵錯
> * [部署 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * [升級 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-upgrade.md)
> * [清除 Service Fabric Mesh 資源](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，您可以在開始前[建立免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

* 確定您已[設定開發環境](service-fabric-mesh-howto-setup-developer-environment-sdk.md)，包括安裝 Service Fabric 執行階段、SDK、Docker 和 Visual Studio 2017。

## <a name="download-the-to-do-sample-application"></a>下載待辦事項範例應用程式

如果您未在[本教學課程系列的第一部分](service-fabric-mesh-tutorial-create-dotnetcore.md)中建立待辦事項範例應用程式，可以在此時下載。 在命令視窗中執行下列命令，將範例應用程式存放庫複製到本機電腦。

```
git clone https://github.com/azure-samples/service-fabric-mesh
```

該應用程式位於 `src\todolistapp` 目錄下。

## <a name="build-and-debug-on-your-local-cluster"></a>在本機叢集上建置和偵測

在您的專案載入後，Docker 映像就會自動建置並部署至本機叢集。 完成此程序可能需要一些時間。 若要在 Visual Studio [輸出] 窗格中監視進度，請將 [輸出] 窗格中的 [顯示輸出來源:] 下拉式清單設為 [Service Fabric 工具]。

按 **F5** 以在本機編譯並執行您的服務。 在本機執行專案並進行偵錯時，Visual Studio 將會：

* 確定適用於 Windows 的 Docker 正在執行，並設定以 Windows 作為容器作業系統。
* 下載任何遺漏的 Docker 基底映像。 完成這部分可能需要一些時間。
* 建置 (或重建) 用來裝載程式碼專案的 Docker 映像。
* 在本機 Service Fabric 開發叢集上部署及執行容器。
* 執行您的服務，並叫用您已設定的任何中斷點。

在本機部署完成，且 Visual Studio 正在執行您的應用程式後，瀏覽器視窗將會開啟並顯示預設範例網頁。

**偵錯秘訣**

目前有問題會導致對 `using (HttpResponseMessage response = client.GetAsync("").GetAwaiter().GetResult())` 的呼叫無法連線至服務。 當您的主機 IP 位址變更時，就可能發生此問題。 若要解決這個問題：

1. 從本機叢集中移除應用程式 (在 Visual Studio 中，選取 [組建] > [清除解決方案])。
2. 在 Service Fabric 本機叢集管理員中選取 [停止本機叢集]，然後選取 [啟動本機叢集]。
3. 重新部署應用程式 (在 Visual Studio 中，按 **F5**)。

如果出現**沒有執行中的 Service Fabric 本機叢集**錯誤，請確定 Service Fabric 本機叢集管理員 (LCM) 正在執行，並以滑鼠右鍵按一下工作列上的 LCM 圖示，然後按一下 [啟動本機叢集]。 在其啟動後，回到 Visual Studio，然後按 **F5**。

如果在應用程式啟動時出現 **404** 錯誤，這可能表示您在 **service.yaml** 中的環境變數不正確。 請確定 `ApiHostPort` 和 `ToDoServiceName` 都已根據[建立環境變數](https://docs.microsoft.com/azure/service-fabric-mesh/service-fabric-mesh-tutorial-create-dotnetcore#create-environment-variables)中的指示正確設定。

如果 **service.yaml** 中出現建置錯誤，請確定用來縮排各行的是空格，而不是定位字元。 此外，目前您必須使用英文地區設定來建置應用程式。

### <a name="debug-in-visual-studio"></a>在 Visual Studio 中偵錯

當您對 Visual Studio 中的 Service Fabric Mesh 應用程式進行偵錯時，您會使用本機 Service Fabric 開發叢集。 若要查看從後端服務擷取待辦事項項目的情形，請對 OnGet() 方法進行偵錯。
1. 在 **WebFrontEnd** 專案中，開啟 [頁面] > [Index.cshtml] > [Index.cshtml.cs]，然後在 **Get** 方法中 (第 17 行) 設定中斷點。
2. 在 **ToDoService** 專案中開啟 **TodoController.cs**，然後在 **OnGet** 方法中 (第 15 行) 設定中斷點。
3. 回到瀏覽器並重新整理頁面。 您會在 Web 前端 `OnGet()` 方法中叫用中斷點。 您可以檢查 `backendUrl` 變數，以查看您在 **service.yaml** 檔案中定義的環境變數如何結合到用來聯繫後端服務的 URL 中。
4. 逐步執行 (F10) `client.GetAsync(backendUrl).GetAwaiter().GetResult())` 呼叫，您將會叫用控制器的 `Get()` 中斷點。 在此方法中，您可以查看從記憶體內部清單擷取待辦事項項目清單的方式。
5. 完成作業後，請按 **Shift+F5**，以停止對 Visual Studio 中的專案進行偵錯。
 
## <a name="next-steps"></a>後續步驟

在教學課程的這個部分中，您已了解：

> [!div class="checklist"]
> * 當您建置 Azure Service Fabric Mesh 應用程式時，會有何情況
> * 如何設定中斷點以觀察服務對服務呼叫

前進到下一個教學課程：
> [!div class="nextstepaction"]
> [部署 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)