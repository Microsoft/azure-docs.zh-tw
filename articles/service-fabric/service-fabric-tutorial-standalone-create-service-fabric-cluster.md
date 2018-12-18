---
title: 教學課程：安裝 Service Fabric 獨立用戶端 - Azure Service Fabric | Microsoft Docs
description: 在本教學課程中，您將了解如何在您於上一篇教學課程文章中建立的叢集上安裝 Service Fabric 獨立用戶端。
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
ms.openlocfilehash: 36d65abb26435581f3e6c9c4a9fc46bb3dd538ce
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39362767"
---
# <a name="tutorial-install-and-create-service-fabric-cluster"></a>教學課程：安裝和建立 Service Fabric 叢集

Service Fabric 獨立叢集讓您能夠選擇自己的環境，並且在 Service Fabric 所採用的「任何 OS、任何雲端」方法中建立叢集。 在此教學課程系列中，您會建立裝載於 AWS 的獨立叢集，並在其中安裝應用程式。

本教學課程是一個系列的第二部分。 本教學課程將逐步引導您完成建立 Service Fabric 獨立叢集的步驟。

在本系列的第二部分中，您將瞭解如何：

> [!div class="checklist"]
> * 下載並安裝 Service Fabric 獨立套件
> * 建立 Service Fabric 叢集
> * 連線至 Service Fabric 叢集

## <a name="download-the-service-fabric-for-windows-server-package"></a>下載 Windows Server 套件的 Service Fabric

Service Fabric 會提供一個安裝套件以建立 Service Fabric 獨立叢集。  在您的本機電腦上[下載安裝套件](http://go.microsoft.com/fwlink/?LinkId=730690)。  成功下載後，請透過 RDP 連線將其複製到您的 EC2 執行個體，並貼在桌面上。

選取 zip 檔案並開啟內容功能表，然後選取 [全部解壓縮] > [解壓縮]。  在解壓縮檔案時，您將會在桌面上產生與 zip 檔案名稱相同的資料夾。

如有需要，請在[安裝套件的內容](service-fabric-cluster-standalone-package-contents.md)取得詳細資料。

## <a name="set-up-your-configuration-file"></a>設定組態檔

您將建置含有三個節點的 Windows 叢集，因此需要修改 `ClusterConfig.Unsecure.MultiMachine.json` 檔案。

接著，請將出現在檔案中第 8、15 和 22 行上的三行 ipAddress 更新為每個執行個體的 IP 位址。

節點在更新之後，將如下所示：

```json
        {
            "nodeName": "vm0",
            "ipAddress": "172.31.27.1",
            "nodeTypeRef": "NodeType0",
            "faultDomain": "fd:/dc1/r0",
            "upgradeDomain": "UD0"
        }
```

然後，您必須更新若干屬性。  在第 34 行上，您必須修改診斷存放區的連接字串，其看起來會像下面這樣：`"connectionstring": "C:\\ProgramData\\SF\\DiagnosticsStore"`

最後，在組態的 `nodeTypes` 區段中新增一個區段，以對應 Windows 所將使用的暫時連接埠。  組態檔應如下所示：

```json
"applicationPorts": {
    "startPort": "20001",
    "endPort": "20031"
},
"ephemeralPorts": {
    "startPort": "20606",
    "endPort": "20861"
},
"isPrimary": true
```

## <a name="validate-the-environment"></a>驗證環境

系統會使用獨立套件中的 *TestConfiguration.ps1* 指令碼作為最佳做法分析程式，以驗證是否可以在指定的環境上部署叢集。 [部署準備](service-fabric-cluster-standalone-deployment-preparation.md)會列出必要條件和環境需求。 執行指令碼來確認您是否可以建立開發叢集︰

```powershell
cd .\Desktop\Microsoft.Azure.ServiceFabric.WindowsServer.6.2.274.9494\
.\TestConfiguration.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json
```

您應該會看到如下的輸出： 如果底層欄位 "Passed" 傳回為 `True`，表示已通過例行性檢查，並可根據輸入組態來部署該叢集。

```powershell
Trace folder already exists. Traces will be written to existing trace folder: C:\Users\Administrator\Desktop\Microsoft.Azure.ServiceFabric.WindowsServer.6.2.274.9494\DeploymentTraces
Running Best Practices Analyzer...
Best Practices Analyzer completed successfully.


LocalAdminPrivilege        : True
IsJsonValid                : True
IsCabValid                 :
RequiredPortsOpen          : True
RemoteRegistryAvailable    : True
FirewallAvailable          : True
RpcCheckPassed             : True
NoConflictingInstallations : True
FabricInstallable          : True
DataDrivesAvailable        : True
NoDomainController         : True
Passed                     : True
```

## <a name="create-the-cluster"></a>建立叢集

在您成功驗證叢集組態後，請執行 *CreateServiceFabricCluster.ps1* 指令碼，以將 Service Fabric 叢集部署至組態檔中的虛擬機器。

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json -AcceptEULA
```

如果執行成功，將會產生如下的輸出：

```powershell
Your cluster is successfully created! You can connect and manage your cluster using Microsoft Azure Service Fabric Explorer or PowerShell. To connect through PowerShell, run 'Connect-ServiceFabricCluster [ClusterConnectionEndpoint]'.
```

> [!NOTE]
> 部署追蹤會寫入您可以執行 CreateServiceFabricCluster.ps1 PowerShell 指令碼的 VM/電腦。 這些可以根據指令碼執行的目錄，在其子資料夾 DeploymentTraces 中找到。 若要查看是否已正確將 Service Fabric 部署到電腦，請在 FabricDataRoot 目錄中找到安裝的檔案，如叢集組態檔的 FabricSettings 區段 (預設為 c:\ProgramData\SF) 中所述。 同時，也要能在 [工作管理員] 看到 FabricHost.exe 和 Fabric.exe 處理序正在執行中。
>
>

### <a name="bring-up-service-fabric-explorer"></a>啟動 Service Fabric Explorer

現在，您可以使用 Service Fabric Explorer 直接從其中一個機器透過 http://localhost:19080/Explorer/index.html 連線至叢集，或透過 http://<*IPAddressofaMachine*>:19080/Explorer/index.html 從遠端連線。

## <a name="add-and-remove-nodes"></a>新增和移除節點

當您的商務需求改變時，您可以在獨立 Service Fabric 叢集中新增或移除節點。 如需詳細步驟，請參閱[在 Service Fabric 獨立叢集中新增或移除節點](service-fabric-cluster-windows-server-add-remove-nodes.md)。

## <a name="next-steps"></a>後續步驟

在此系列的第二個部分中，您已學到如何將大量隨機資料平行上傳至儲存體帳戶，例如如何：

> [!div class="checklist"]
> * 設定連接字串
> * 建置應用程式
> * 執行應用程式
> * 驗證連線數目

繼續進行此系列的第三個部分，將應用程式安裝到您所建立的叢集中。

> [!div class="nextstepaction"]
> [將應用程式安裝到 Service Fabric 叢集中](service-fabric-tutorial-standalone-install-an-application.md)

<!--Image references-->
[Trusted Zone]: ./media/service-fabric-cluster-creation-for-windows-server/TrustedZone.png
