---
title: 教學課程：在獨立 Service Fabric 叢集上安裝應用程式 - Azure Service Fabric | Microsoft Docs
description: 在本教學課程中，您將了解如何將應用程式安裝到獨立 Service Fabric 叢集中。
services: service-fabric
documentationcenter: .net
author: david-stanford
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/11/2018
ms.author: dastanfo
ms.custom: mvc
ms.openlocfilehash: 65c0ca98393fbb4f54eee2f4864218f231765904
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34208568"
---
# <a name="tutorial-deploy-an-application-on-your-service-fabric-standalone-cluster"></a>教學課程：將應用程式部署在 Service Fabric 獨立叢集上

Service Fabric 獨立叢集讓您能夠選擇自己的環境，並且在 Service Fabric 所採用的「任何 OS、任何雲端」方法中建立叢集。 在此教學課程系列中，您會建立裝載於 AWS 的獨立叢集，並在其中部署應用程式。

本教學課程是一個系列課程的第三部分。  Service Fabric 獨立叢集讓您能夠選擇自己的環境，並且在用於 Service Fabric 的「任何 OS、任何雲端」方法中建立叢集。 本教學課程將說明如何建立裝載此獨立叢集所需的 AWS 基礎結構。

在系列的第三部分中，您將了解如何：

> [!div class="checklist"]
> * 下載範例應用程式
> * 部署至叢集

## <a name="prerequisites"></a>先決條件

開始進行本教學課程之前：

* [安裝 Visual Studio 2017](https://www.visualstudio.com/) 並安裝 **Azure 開發**以及 **ASP.NET 和 Web 開發**工作負載。
* [安裝 Service Fabric SDK](service-fabric-get-started.md)

## <a name="download-the-voting-sample-application"></a>下載投票應用程式範例

在命令視窗中執行下列命令，將範例應用程式存放庫複製到本機電腦。

```
git clone https://github.com/Azure-Samples/service-fabric-dotnet-quickstart
```

### <a name="deploy-the-app-to-the-service-fabric-cluster"></a>將應用程式部署到 Service Fabric 叢集

應用程式下載後，您可以直接從 Visual Studio 將其部署至叢集。

1. 開啟 Visual Studio

2. 選取 [檔案] > [開啟]

3. 瀏覽至您複製 git 存放庫的資料夾，並選取 Voting.sln

4. 以滑鼠右鍵按一下 [方案總管] 中的 `Voting` 應用程式專案，然後選擇 [發佈]。

5. 選取 [連線端點] 的下拉式清單，並輸入叢集中一個節點的公開 DNS 名稱。  例如， `ec2-34-215-183-77.us-west-2.compute.amazonaws.com:19000`

6. 開啟您偏好的瀏覽器，並輸入叢集位址 (連線端點，此應用程式部署於連接埠 8080 - 例如 ec2-34-215-183-77.us-west-2.compute.amazonaws.com:8080)。

    ![叢集的 API 回應](./media/service-fabric-tutorial-standalone-cluster/deployed-app.png)

## <a name="next-steps"></a>後續步驟

在此系列的第三個單元中，您已了解如何將應用程式部署至您的叢集：

> [!div class="checklist"]
> * 下載範例應用程式
> * 部署至叢集

請繼續進行此系列的第四個單元，以清除您的叢集。

> [!div class="nextstepaction"]
> [清除資源](service-fabric-tutorial-standalone-clean-up.md)