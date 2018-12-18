---
title: 使用 C# 建立 Azure Service Fabric 可靠服務
description: 使用 Visual Studio，建立、部署和偵錯以 Azure Service Fabric 為建置基礎的 Reliable Services 應用程式。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: vturecek
ms.assetid: c3655b7b-de78-4eac-99eb-012f8e042109
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/21/2018
ms.author: ryanwi
ms.openlocfilehash: 7dadaadd0e6a6e6d71685356568076ad26305cc2
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34642218"
---
# <a name="create-your-first-c-service-fabric-stateful-reliable-services-application"></a>建立第一個 C# Service Fabric 具狀態 Reliable Services 應用程式

了解如何在短短幾分鐘內，在 Windows 上部署第一個適用於 .NET 的 Azure Service Fabric 應用程式。 當您完成時，將會有以 Reliable Services 應用程式執行的本機叢集。

## <a name="prerequisites"></a>先決條件

開始之前，請確定您已[設定開發環境](service-fabric-get-started.md)。 此程序包括安裝 Service Fabric SDK 及 Visual Studio 2017 或 2015。

## <a name="create-the-application"></a>建立應用程式

1. 以系統管理員身分啟動 Visual Studio。

2. 選取 Ctrl+Shift+N 以建立專案。

3. 在 [新增專案]  對話方塊中，選取 [雲端] >  [Service Fabric 應用程式]。

4. 將應用程式命名為 **MyApplication**。 然後選取 [確定]。

   ![Visual Studio 中的 [新增專案] 對話方塊][1]

5. 您可以從下一個對話方塊建立任何類型的 Service Fabric 應用程式。 在本快速入門中，選擇 [.Net Core 2.0] > [具狀態服務]。

6. 將服務命名為 **MyStatefulService**。 然後選取 [確定]。

    ![Visual Studio 中的 [新增服務] 對話方塊][2]

    Visual Studio 會建立應用程式專案和具狀態服務專案。 然後在 [方案總管] 中加以顯示。

    ![方案總管接著會建立具狀態服務的應用程式。][3]

    應用程式專案 (**MyApplication**) 沒有任何程式碼。 它反而會參考一組服務專案。 它還包含三種其他類型的內容：

    * **發佈設定檔**  
    可供部署至不同環境的設定檔。

    * **指令碼**  
    用來部署或升級應用程式的 PowerShell 指令碼。

    * **應用程式定義**  
包含 ApplicationPackageRoot 之下的 ApplicationManifest.xml 檔案，其描述您的應用程式組合。 相關聯的應用程式參數檔案位於 ApplicationParameters 之下，其可用來指定指定環境特有參數。 Visual Studio 會選取在相關聯的發行設定檔中指定的應用程式參數檔案。
    
如需服務專案的內容概觀，請參閱 [開始使用 Reliable Services](service-fabric-reliable-services-quick-start.md)。

## <a name="deploy-and-debug-the-application"></a>部署和偵錯應用程式

您現在有一個應用程式，請採取下列步驟來執行、部署它，以及進行偵錯。

1. 在 Visual Studio 中，選取 **F5** 部署應用程式以供偵錯。  如果出現訊息方塊，詢問您是否要將 Visual Studio 專案目錄的讀取和執行權限授與 'ServiceFabricAllowedUsers' 群組，請按一下 [是]。

    >[!NOTE]
    >您第一次在本機執行及部署應用程式時，Visual Studio 會建立本機叢集以供偵錯。 這可能需要一些時間。 叢集建立狀態會顯示在 Visual Studio 輸出視窗中。
    
     備妥叢集時，您會收到來自 SDK 隨附的本機叢集系統匣管理員應用程式通知。
     
    >[!NOTE]
    >此練習需要 5 個節點 (vs.1 個節點) 的叢集。 您可以如下確認這點：以滑鼠右鍵按一下 [Service Fabric 本機叢集管理員] 系統匣應用程式，然後按一下 [切換叢集模式]，以啟動 Service Fabric Explorer 工具。 如果目前選取 [1 個節點]，則按一下 [5 個節點]。
    
    ![本機叢集系統匣通知][4]

    應用程式啟動之後，Visual Studio 會自動顯示 [診斷事件檢視器]，以便查看服務的追蹤輸出。
    
    ![診斷事件檢視器][5]

    >[!NOTE]
    >事件應在 [診斷事件檢視器] 中自動啟動追蹤。 如果您需要手動設定 [診斷事件檢視器]，請先開啟位於 **MyStatefulService** 專案中的 `ServiceEventSource.cs` 檔案。 在 `ServiceEventSource` 類別頂端，複製 `EventSource` 屬性的值。 在下面範例中，事件來源名為 `"MyCompany-MyApplication-MyStatefulService"`，這在您的情況中可能會有所不同。
>
    >![尋找服務事件來源名稱][service-event-source-name]


2. 接著，開啟 [ETW 提供者] 對話方塊。 然後選取位於 [診斷事件] 索引標籤上的齒輪圖示。將您複製的事件來源名稱貼到 [ETW 提供者] 輸入方塊中。 然後，選取 [套件] 按鈕。 這會自動啟動追蹤事件。

    ![設定診斷事件來源名稱][setting-event-source-name]

    您現在應會看到事件顯示在 [診斷事件] 視窗中。

    具狀態服務範本會顯示一個計數器數值，此值會依 **MyStatefulService.cs** 的 `RunAsync` 方法遞增。

3. 展開其中一個事件以查看更多詳細資料，包括程式碼執行所在的節點。 在此情況下是 **\_Node\_0**，但在您的電腦上可能會是其他節點。
   
    ![診斷事件檢視器詳細資訊][6]

4. 本機叢集包含五個節點，均裝載於單一電腦上。 在生產環境中，每個節點會裝載於不同的實體或虛擬機器上。 若要在您運用 Visual Studio 偵錯工具時模擬遺失機器的情況，請關閉本機叢集上的其中一個節點。

5. 在 [方案總管] 視窗中，開啟 **MyStatefulService.cs**。 

6. 尋找 `RunAsync` 方法，然後在方法的第一行上設定中斷點。

    ![具狀態服務 RunAsync 方法的中斷點][7]

7. 以滑鼠右鍵按一下 [Service Fabric 本機叢集管理員] 系統匣應用程式，然後選取 [管理本機叢集]，以啟動 Service Fabric Explorer 工具。

    ![從本機叢集管理員啟動 Service Fabric Explorer][systray-launch-sfx]

    
  [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 會提供叢集的視覺表示法。 其中包括部署至叢集的應用程式集，以及構成叢集的實體節點集合。

8. 在左窗格中展開 [叢集] > [節點]，並尋找您的程式碼執行所在的節點。 然後選取 [動作] > [停用 (重新啟動)] 來模擬電腦重新啟動。

    ![在 Service Fabric Explorer 中停止節點][sfx-stop-node]

    您應該會在 Visual Studio 中短暫看見達到您的中斷點，因為您一個節點上所做的計算完美地容錯移轉至另一個節點。 按 **F5** 繼續。

9. 接下來，返回 [診斷事件檢視器] 並觀察訊息。 計數器已繼續遞增，即使事件實際上來自不同的節點。

    ![容錯移轉之後的診斷事件檢視器][diagnostic-events-viewer-detail-post-failover]

## <a name="clean-up-the-local-cluster-optional"></a>清除本機叢集 (選用)

請記住，此本機叢集是真實的叢集。 停止偵錯工具將會移除應用程式執行個體，並取消註冊應用程式類型。 不過，叢集會繼續在背景中執行。 當您準備要停止本機叢集時，有幾個選項可用。

### <a name="keep-application-and-trace-data"></a>保留應用程式和追蹤資料

以滑鼠右鍵按一下 [本機叢集管理員] 系統匣應用程式，然後選取 [停止本機叢集]，以關閉叢集。

### <a name="delete-the-cluster-and-all-data"></a>刪除叢集和所有資料

以滑鼠右鍵按一下 [本機叢集管理員] 系統匣應用程式，即可移除叢集。 然後選擇 [移除本機叢集]。 

如果您選擇此選項，Visual Studio 會在您下一次執行應用程式時重新部署叢集。 如果您打算停用本機叢集一陣子或需要回收資源，請選擇此選項。

## <a name="next-steps"></a>後續步驟
深入了解 [Reliable Services](service-fabric-reliable-services-introduction.md)。
<!-- Image References -->

[1]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog.png
[2]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog-2.png
[3]: ./media/service-fabric-create-your-first-application-in-visual-studio/solution-explorer-stateful-service-template.png
[4]: ./media/service-fabric-create-your-first-application-in-visual-studio/local-cluster-manager-notification.png
[5]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer.png
[6]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer-detail.png
[7]: ./media/service-fabric-create-your-first-application-in-visual-studio/runasync-breakpoint.png
[sfx-stop-node]: ./media/service-fabric-create-your-first-application-in-visual-studio/sfe-deactivate-node.png
[systray-launch-sfx]: ./media/service-fabric-create-your-first-application-in-visual-studio/launch-sfx.png
[diagnostic-events-viewer-detail-post-failover]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer-detail-post-failover.png
[sfe-delete-application]: ./media/service-fabric-create-your-first-application-in-visual-studio/sfe-delete-application.png
[switch-cluster-mode]: ./media/service-fabric-create-your-first-application-in-visual-studio/switch-cluster-mode.png
[cluster-setup-success-1-node]: ./media/service-fabric-get-started-with-a-local-cluster/cluster-setup-success-1-node.png
[service-event-source-name]: ./media/service-fabric-create-your-first-application-in-visual-studio/event-source-attribute-value.png
[setting-event-source-name]: ./media/service-fabric-create-your-first-application-in-visual-studio/setting-event-source-name.png
[switch-cluster-mode]: ./media/service-fabric-create-your-first-application-in-visual-studio/switch-cluster-mode.png
[cluster-setup-success-1-node]: ./media/service-fabric-get-started-with-a-local-cluster/cluster-setup-success-1-node.png
[service-event-source-name]: ./media/service-fabric-create-your-first-application-in-visual-studio/event-source-attribute-value.png
[setting-event-source-name]: ./media/service-fabric-create-your-first-application-in-visual-studio/setting-event-source-name.png
