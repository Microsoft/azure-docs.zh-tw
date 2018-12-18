---
title: Service Fabric 應用程式升級教學課程 | Microsoft Docs
description: 本文章會逐步解說使用 Visual Studio 部署 Service Fabric 應用程式、變更程式碼及執行升級的體驗。
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: a3181a7a-9ab1-4216-b07a-05b79bd826a4
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: subramar
ms.openlocfilehash: 0a94dc91308f21d9e2b440f9638829ee03a8b903
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34205190"
---
# <a name="service-fabric-application-upgrade-tutorial-using-visual-studio"></a>使用 Visual Studio 進行 Service Fabric 應用程式升級的教學課程
> [!div class="op_single_selector"]
> * [PowerShell](service-fabric-application-upgrade-tutorial-powershell.md)
> * [Visual Studio](service-fabric-application-upgrade-tutorial.md)
> 
> 

<br/>

Azure Service Fabric 藉由確保只升級已變更的服務，並且在整個升級程序中監視應用程式健康狀態，簡化雲端應用程式的升級程序。 它也會在應用程式發生問題時自動回復到舊版。 Service Fabric 應用程式升級並「不需要停機」 ，因為可以在不停機的情況下升級應用程式。 本教學課程涵蓋如何從 Visual Studio 完成輪流升級。

## <a name="step-1-build-and-publish-the-visual-objects-sample"></a>步驟 1：建置和發佈視覺物件範例
首先從 GitHub 下載 [Visual Objects](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/classic/Actors/VisualObjects) 應用程式。 然後在應用程式專案 **VisualObjects** 上按一下滑鼠右鍵，選取 Service Fabric 功能表項目中的 [發佈] 命令來建置和發佈應用程式。

![Service Fabric 應用程式的操作功能表][image1]

選取 [發佈] 會顯示另一個快顯視窗，您可以將 [目標設定檔] 設定成 [PublishProfiles\Local.xml]。 在您按一下 [發佈] 之前，視窗應該看起來如下。

![發佈 Service Fabric 應用程式][image2]

現在，您可以按一下對話方塊中的 [發佈]  。 您可以使用 [Service Fabric 總管來檢視叢集與應用程式](service-fabric-visualizing-your-cluster.md)。 Visual Objects 應用程式有一個 Web 服務，您可在瀏覽器的網址列中輸入 [http://localhost:8081/visualobjects/](http://localhost:8081/visualobjects/)，以移至該服務。  您應該會看到 10 個浮動的視覺物件在畫面中四處移動。

**注意︰** 如果部署至 `Cloud.xml` 設定檔 (Azure Service Fabric)，則應該可在 **http://{ServiceFabricName}.{Region}.cloudapp.azure.com:8081/visualobjects/** 上取得應用程式。 請確定您確實已在負載平衡器 (在與 Service Fabric 執行個體相同的資源群組中尋找負載平衡器) 中設定 `8081/TCP`。

## <a name="step-2-update-the-visual-objects-sample"></a>步驟 2：更新視覺物件範例
您可能會注意到使用步驟 1 中部署的版本，視覺物件不會旋轉。 讓我們將這個應用程式升級到其中的視覺物件也會旋轉的版本。

選取 VisualObjects 方案內的 VisualObjects.ActorService 專案，然後開啟 **VisualObjectActor.cs** 檔案。 在該檔案內，移至 `MoveObject` 方法，將 `visualObject.Move(false)` 標記為註解，並將 `visualObject.Move(true)` 取消註解。 這項程式碼變更會在服務升級後旋轉物件。  **現在您可以建置 (而非重建) 方案**，這會建置修改過的專案。 如果您選取 [全部重建] ，則必須更新所有專案的版本。

我們也需要設定應用程式的版本。 若要進行版本變更，請在 [VisualObjects] 專案上按一下滑鼠右鍵，使用 Visual Studio 的 [編輯資訊清單版本] 選項。 選取此選項會帶出對話方塊供您編輯版本，如以下所示：

![版本設定對話方塊][image3]

將已修改之專案及其程式碼封裝連同應用程式的版本一起更新成 2.0.0 版。 進行變更之後，資訊清單應該會看起來如下 (粗體部分即為所做的變更)：

![更新版本][image4]

選取 [自動更新應用程式與服務版本] 之後，Visual Studio 工具便可自動彙總版本。 如果您使用 [SemVer](http://www.semver.org)，當您選取以上選項時，則必須單獨更新程式碼和 (或) 組態封裝版本。

儲存變更，然後立即核取 [升級應用程式]  方塊。

## <a name="step-3--upgrade-your-application"></a>步驟 3：升級應用程式
請您熟悉[應用程式升級參數](service-fabric-application-upgrade-parameters.md)和[升級程序](service-fabric-application-upgrade.md)，以了解可以套用的各種升級參數、逾時和健康狀態準則。 針對此逐步解說，服務健康狀態評估準則會設定為預設值 (未受監視的模式)。 您可以選取 [設定升級設定]  ，然後視需要修改參數，來設定這些設定。

現在，我們已經準備好選取 [發佈] 來啟動應用程式升級。 此選項會將您的應用程式升級到其中物件會旋轉的版本 2.0.0。 Service Fabric 會一次升級一個更新網域 (某些物件會先更新，其他物件再接著更新)，在升級期間仍可存取服務。 透過您的用戶端 (瀏覽器) 可檢查服務的存取權。  

現在，隨著應用程式繼續進行升級，您可以利用 Service Fabric Explorer (使用應用程式底下的 [Upgrades in Progress (正在進行升級)]  索引標籤) 來監視應用程式。

幾分鐘後，應該就已升級 (完成) 所有更新網域，而 Visual Studio 的輸出視窗應該也會指出升級已完成。 而且，您應該會發現瀏覽器視窗中的「所有」  視覺物件現在都在旋轉！

您可能會想要嘗試變更版本，從版本 2.0.0 移到版本 3.0.0 來做為練習，或甚至是從版本 2.0.0 變回版本 1.0.0。 練習逾時和健康狀態原則，讓自己更熟練。 在部署至 Azure 叢集而不是本機叢集時，所使用的參數可能不同。 我們建議您保守地設定逾時。

## <a name="next-steps"></a>後續步驟
[使用 PowerShell 升級您的應用程式](service-fabric-application-upgrade-tutorial-powershell.md) 將引導您完成使用 PowerShell 進行應用程式升級的步驟。

使用 [升級參數](service-fabric-application-upgrade-parameters.md)來控制您應用程式的升級方式。

了解如何使用 [資料序列化](service-fabric-application-upgrade-data-serialization.md)，以讓您的應用程式升級相容。

參考 [進階主題](service-fabric-application-upgrade-advanced.md)，以了解如何在升級您的應用程式時使用進階功能。

參考 [疑難排解應用程式升級](service-fabric-application-upgrade-troubleshooting.md)中的步驟，以修正應用程式升級中常見的問題。

[image1]: media/service-fabric-application-upgrade-tutorial/upgrade7.png
[image2]: media/service-fabric-application-upgrade-tutorial/upgrade1.png
[image3]: media/service-fabric-application-upgrade-tutorial/upgrade5.png
[image4]: media/service-fabric-application-upgrade-tutorial/upgrade6.png
