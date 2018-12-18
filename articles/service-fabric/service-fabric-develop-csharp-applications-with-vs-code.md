---
title: 使用 Visual Studio Code 開發 .NET Core Azure Service Fabric 應用程式 | Microsoft Docs
description: 本文示範如何使用 Visual Studio Code 建置及部署 .NET Core Service Fabric 應用程式，並對其進行偵錯。
services: service-fabric
documentationcenter: .net
author: JimacoMS2
manager: timlt
editor: ''
ms.assetid: 96176149-69bb-4b06-a72e-ebbfea84454b
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/29/2018
ms.author: v-jamebr
ms.openlocfilehash: 27c7c62125f3f559fb1764292729cbbfdc1c4e5f
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37115415"
---
# <a name="develop-c-service-fabric-applications-with-visual-studio-code"></a>使用 Visual Studio Code 開發 C# Service Fabric 應用程式

[適用於 VS Code 的 Service Fabric Reliable Services 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-service-fabric-reliable-services)可以輕易地在 Windows、Linux 和 macOS 作業系統上建置 .NET Core Service Fabric 應用程式。

本文示範如何使用 Visual Studio Code 建置及部署 .NET Core Service Fabric 應用程式，並對其進行偵錯。

## <a name="prerequisites"></a>先決條件

本文假設您已安裝 VS Code、適用於 VS Code 的 Service Fabric Reliable Services 擴充功能，和開發環境所需的任何相依性。 若要深入了解，請參閱[使用者入門](./service-fabric-get-started-vs-code.md#prerequisites)。

## <a name="download-the-sample"></a>下載範例
本文使用的 CounterService 應用程式位於 [Service Fabric .NET Core 入門範例的 GitHub 存放庫](https://github.com/Azure-Samples/service-fabric-dotnet-core-getting-started)中。 

若要將存放庫複製到您的開發機器，請從終端機視窗 (在 Windows 上是命令視窗) 執行下列命令：

```
git clone https://github.com/Azure-Samples/service-fabric-dotnet-core-getting-started.git
```

## <a name="open-the-application-in-vs-code"></a>在 VS Code 中開啟應用程式

### <a name="windows"></a>Windows
以滑鼠右鍵按一下 [開始] 功能表中的 VS Code 圖示，然後選擇 [以系統管理員身分執行]。 若要將偵錯工具附加至您的服務，您需要以系統管理員身分執行 VS Code。

### <a name="linux"></a>Linux
使用終端機，從具有應用程式副本的本機目錄中瀏覽至 /service-fabric-dotnet-core-getting-started/Services/CounterService 路徑。

以根使用者身分執行下列命令來開啟 VS Code，以便將偵錯工具附加到您的服務。
```
sudo code . --user-data-dir='.'
```

應用程式現在應該會出現在 VS Code 工作區中。

![工作區中的計數器服務應用程式](./media/service-fabric-develop-csharp-applications-with-vs-code/counter-service-application-in-workspace.png)

## <a name="build-the-application"></a>建置應用程式
1. 按下 (Ctrl + Shift + p) 以在 VS Code 中開啟 [命令選擇區]。
2. 搜尋並選取 **Service Fabric: Build Application** 命令。 建置輸出會傳送到整合式終端機。

   ![VS Code 中的建置應用程式命令](./media/service-fabric-develop-csharp-applications-with-vs-code/sf-build-application.png)

## <a name="deploy-the-application-to-the-local-cluster"></a>將應用程式部署至本機叢集
建置應用程式後，您可以將其部署到本機叢集。 

1. 從 [命令選擇區] 中選取 **Service Fabric: Deploy Application (Localhost)** 命令。 安裝程序的輸出會傳送到整合式終端機。

   ![VS Code 中的部署應用程式命令](./media/service-fabric-develop-csharp-applications-with-vs-code/sf-deploy-application.png)

4. 部署完成時，請啟動瀏覽器並開啟 Service Fabric Explorer： http://localhost:19080/Explorer 。 您應該會看到應用程式正在執行。 這可能需要一些時間，請耐心等候。 

   ![Service Fabric Explorer 中的計數器服務應用程式](./media/service-fabric-develop-csharp-applications-with-vs-code/sfx-verify-deploy.png)

4. 確認應用程式正在執行之後，請啟動瀏覽器並開啟此頁面： http://localhost:31002 。 這是應用程式的 Web 前端。 由於計數器的值會增加，請重新整理頁面以查看目前的值。

   ![瀏覽器中的計數器服務應用程式](./media/service-fabric-develop-csharp-applications-with-vs-code/counter-service-running.png)

## <a name="debug-the-application"></a>偵錯應用程式
在 VS Code 中針對應用程式進行偵錯時，應用程式必須在本機叢集上執行。 然後您可以將中斷點新增到程式碼中。

若要設定中斷點並進行偵錯，請完成下列步驟：
1. 在 Explorer 中，開啟 /src/CounterServiceApplication/CounterService/CounterService.cs 檔案，並在 `RunAsync` 方法中的第 62 行上設定中斷點。
3. 按一下 [活動列] 中的偵錯圖示，以在 VS Code 中開啟偵錯工具檢視。 按一下偵錯工具檢視頂端的齒輪圖示，然後從下拉式環境功能表中選取 [.NET Core]。 launch.json 檔案隨即開啟。 您可以關閉此檔案。 現在您應該會在 [執行] 按鈕 (綠色箭頭) 旁的 [偵錯組態] 功能表中看到組態選項。

   ![VS Code 工作區中的偵錯圖示](./media/service-fabric-develop-csharp-applications-with-vs-code/debug-icon-workspace.png)

2. 從 [偵錯組態] 功能表中選取 [.NET Core 附加]。

   ![VS Code 工作區中的偵錯圖示](./media/service-fabric-develop-csharp-applications-with-vs-code/debug-start.png)

3. 在瀏覽器中開啟 Service Fabric Explorer： http://localhost:19080/Explorer 。 按一下 [應用程式] 並向下切入，以判斷其中正在執行 CounterService 的主要節點。 在下圖中，CounterService 的主要節點是「節點 0」。

   ![CounterService 的主要節點](./media/service-fabric-develop-csharp-applications-with-vs-code/counter-service-primary-node.png)

4. 在 VS Code 中，按一下 [.NET Core 附加] 偵錯組態旁的 [執行] 圖示 (綠色箭頭)。 在 [程序選項] 對話方塊中，選取在主要節點 (定義於步驟 4) 上執行的 CounterService 程序。

   ![主要程序](./media/service-fabric-develop-csharp-applications-with-vs-code/select-process.png)

5. CounterService.cs 檔案中的中斷點將會非常快速地叫用。 您可以接著探索本機變數的值。 使用 VS Code 頂端的 [偵錯] 工具列繼續執行，越過程式碼行、逐步執行方法，或跳離目前的方法。 

   ![偵錯值](./media/service-fabric-develop-csharp-applications-with-vs-code/breakpoint-hit.png)

6. 若要結束偵錯工作階段，請按一下 VS Code 頂端 [偵錯] 工具列上的插頭圖示。
   
   ![中斷與偵錯工具的連線](./media/service-fabric-develop-csharp-applications-with-vs-code/debug-bar-disconnect.png)
       
7. 當您完成偵錯時，可以使用 **Service Fabric: Remove Application** 命令，從本機叢集內移除 CounterService 應用程式。 

## <a name="next-steps"></a>後續步驟

* 了解如何[使用 VS Code 開發 Java Service Fabric 應用程式及進行偵錯](./service-fabric-develop-java-applications-with-vs-code.md)。



