---
title: 使用 Visual Studio Code 開發 Java Azure Service Fabric 應用程式 | Microsoft Docs
description: 本文示範如何使用 Visual Studio Code 建置及部署 Java Service Fabric 應用程式，並對其進行偵錯。
services: service-fabric
documentationcenter: .net
author: JimacoMS
manager: timlt
editor: ''
ms.assetid: 96176149-69bb-4b06-a72e-ebbfea84454b
ms.service: service-fabric
ms.devlang: java
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/29/2018
ms.author: v-jamebr
ms.openlocfilehash: 54c94c50f6292694e947d97a10fd6976c14e19df
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37115340"
---
# <a name="develop-java-service-fabric-applications-with-visual-studio-code"></a>使用 Visual Studio Code 開發 Java Service Fabric 應用程式

[適用於 VS Code 的 Service Fabric Reliable Services 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-service-fabric-reliable-services)可以輕易地在 Windows、Linux 和 macOS 作業系統上建置 Java Service Fabric 應用程式。

本文示範如何使用 Visual Studio Code 建置及部署 Java Service Fabric 應用程式，並對其進行偵錯。

> [!IMPORTANT]
> Service Fabric Java 應用程式可以在 Windows 機器上開發，但是只能部署到 Azure Linux 叢集上。 不支援在 Windows 上針對 Java 應用程式進行偵錯。

## <a name="prerequisites"></a>先決條件

本文假設您已安裝 VS Code、適用於 VS Code 的 Service Fabric Reliable Services 擴充功能，和開發環境所需的任何相依性。 若要深入了解，請參閱[使用者入門](./service-fabric-get-started-vs-code.md#prerequisites)。

## <a name="download-the-sample"></a>下載範例
本文會使用 [Service Fabric Java 應用程式快速入門範例 GitHub 存放庫](https://github.com/Azure-Samples/service-fabric-java-quickstart)中的投票應用程式。 

若要將存放庫複製到您的開發機器，請從終端機視窗 (在 Windows 上是命令視窗) 執行下列命令：

```sh
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart.git
```

## <a name="open-the-application-in-vs-code"></a>在 VS Code 中開啟應用程式

開啟 VS Code。  按一下 [活動列] 中的 Explorer 圖示，然後按一下 [開啟資料夾]，或按一下 [檔案 -> 開啟資料夾]。 巡覽至您複製存放庫所在資料夾中的 ./service-fabric-java-quickstart/Voting 目錄，然後按一下 [確定]。 工作區應該包含與以下螢幕擷取畫面所示相同的檔案。

![工作區中的 Java 投票應用程式](./media/service-fabric-develop-java-applications-with-vs-code/java-voting-application.png)

## <a name="build-the-application"></a>建置應用程式

1. 按下 (Ctrl + Shift + p) 以在 VS Code 中開啟 [命令選擇區]。
2. 搜尋並選取 **Service Fabric: Build Application** 命令。 建置輸出會傳送到整合式終端機。

   ![VS Code 中的建置應用程式命令](./media/service-fabric-develop-java-applications-with-vs-code/sf-build-application.png)

## <a name="deploy-the-application-to-the-local-cluster"></a>將應用程式部署至本機叢集
建置應用程式後，您可以將其部署到本機叢集。 

> [!IMPORTANT]
> 在 Windows 機器上不支援將 Java 應用程式部署到本機叢集。

1. 從 [命令選擇區] 中選取 **Service Fabric: Deploy Application (Localhost)** 命令。 安裝程序的輸出會傳送到整合式終端機。

   ![VS Code 中的部署應用程式命令](./media/service-fabric-develop-java-applications-with-vs-code/sf-deploy-application.png)

4. 部署完成時，請啟動瀏覽器並開啟 Service Fabric Explorer：http://localhost:19080/Explorer。 您應該會看到應用程式正在執行。 這可能需要一些時間，請耐心等候。 

   ![Service Fabric Explorer 中的投票應用程式](./media/service-fabric-develop-java-applications-with-vs-code/sfx-localhost-java.png)

4. 確認應用程式正在執行之後，請啟動瀏覽器並開啟此頁面：http://localhost:8080。 這是應用程式的 Web 前端。 您可以新增項目，然後按一下以進行投票。

   ![瀏覽器中的投票應用程式](./media/service-fabric-develop-java-applications-with-vs-code/voting-sample-in-browser.png)

5. 若要從叢集內移除應用程式，請從 [命令選擇區] 中選取 **Service Fabric: Remove Application** 命令。 解除安裝程序的輸出會傳送到整合式終端機。 您可以使用 Service Fabric Explorer，來確認應用程式是否已從本機叢集內移除。

## <a name="debug-the-application"></a>偵錯應用程式
在 VS Code 中針對應用程式進行偵錯時，應用程式必須在本機叢集上執行。 然後您可以將中斷點新增到程式碼中。

> [!IMPORTANT]
> 不支援在 Windows 機器上針對 Java 應用程式進行偵錯。

若要準備 VotingDataService 和投票應用程式，以便進行偵錯，請完成下列步驟：

1. 請更新 Voting/VotingApplication/VotingDataServicePkg/Code/entryPoint.sh 檔案。
在第 6 行 (使用 '#') 對命令加上註解，然後將下列命令新增至檔案底部：

   ```
   java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -jar VotingDataService.jar
   ```

2. 請更新 Voting/VotingApplication/ApplicationManifest.xml 檔案。 在 **StatefulService** 元素中將 **MinReplicaSetSize** 和 **TargetReplicaSetSize** 屬性設為 "1"：
   
   ```xml
         <StatefulService MinReplicaSetSize="1" ServiceTypeName="VotingDataServiceType" TargetReplicaSetSize="1">
   ```

3. 按一下 [活動列] 中的偵錯圖示，以在 VS Code 中開啟偵錯工具檢視。 按一下偵錯工具檢視頂端的齒輪圖示，然後從下拉式環境功能表中選取 [Java]。 launch.json 檔案隨即開啟。 

   ![VS Code 工作區中的偵錯圖示](./media/service-fabric-develop-java-applications-with-vs-code/debug-icon-workspace.png)

3. 在 launch.json 檔案中，將組態 (名為**偵錯 (附加)**) 中的連接埠值設為 **8001**。 儲存檔案。

   ![針對 launch.json 的組態進行偵錯](./media/service-fabric-develop-java-applications-with-vs-code/launch-json-java.png)

4. 藉由使用 **Service Fabric: Deploy Application (Localhost)** 命令，將應用程式部署到本機叢集。 確認應用程式正在 Service Fabric Explorer 中執行。 您的應用程式現已完成偵錯準備。

若要設定中斷點，請完成下列步驟：

1. 在 Explorer 中，開啟 /Voting/VotingDataService/src/statefulservice/VotingDataService.java 檔案。 在 `addItem` 方法 (第 80 行) 中 `try` 區塊的程式碼第一行上，設定中斷點。
   
   ![在投票資料服務中設定中斷點](./media/service-fabric-develop-java-applications-with-vs-code/breakpoint-set.png)

   > [!IMPORTANT]
   > 請確定您在可執行程式碼行上設定中斷點。 舉例來說，偵錯工具會遺漏在方法宣告、`try` 陳述式或 `catch` 陳述式上設定的中斷點。
2. 若要開始偵錯，請按一下 [活動列] 中的偵錯圖示，從偵錯功能表選取 [偵錯 (附加)] 組態，然後按一下 [執行] 按鈕 (綠色箭號)。

   ![偵錯 (附加) 組態](./media/service-fabric-develop-java-applications-with-vs-code/debug-attach-java.png)

3. 在網頁瀏覽器中移至 http://localhost:8080。 在文字方塊中輸入新項目，然後按一下 [+ 新增]。 您的中斷點應已遭點擊。 您可以使用 VS Code 頂端的 [偵錯] 工具列繼續執行，越過程式碼行、逐步執行方法，或跳離目前的方法。 
   
   ![點擊中斷點](./media/service-fabric-develop-java-applications-with-vs-code/breakpoint-hit.png)
       
4. 若要結束偵錯工作階段，請按一下 VS Code 頂端 [偵錯] 工具列上的插頭圖示。
   
   ![中斷與偵錯工具的連線](./media/service-fabric-develop-java-applications-with-vs-code/debug-bar-disconnect.png)
       
5. 當您完成偵錯時，您可以使用 **Service Fabric: Remove Application** 命令，從您的本機叢集內移除投票應用程式。 

## <a name="next-steps"></a>後續步驟

* 深入了解如何[使用 VS Code 開發 C# Service Fabric 應用程式及進行偵錯](./service-fabric-develop-csharp-applications-with-vs-code.md)。
