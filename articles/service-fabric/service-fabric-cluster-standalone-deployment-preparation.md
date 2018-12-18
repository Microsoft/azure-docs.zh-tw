---
title: Azure Service Fabric 獨立叢集部署準備 | Microsoft Docs
description: 文件說明關於在部署用來處理生產工作負載的叢集之前，需要考慮準備的環境和建立的叢集組態。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 9/11/2018
ms.author: dekapur
ms.openlocfilehash: 7357577ea7aff467288578673110b1a6dda7ccef
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391042"
---
<a id="preparemachines"></a>

# <a name="plan-and-prepare-your-service-fabric-standalone-cluster-deployment"></a>規劃和準備您的 Service Fabric 獨立叢集部署
建立叢集之前，請先執行下列步驟。

## <a name="plan-your-cluster-infrastructure"></a>規劃叢集基礎結構
您即將「自行」在您所擁有的電腦上建立 Service Fabric 叢集，因此您可以決定您希望叢集不受何種失敗的影響。 例如，您是否需要提供給這些電腦的個別電源線或網際網路連線？ 此外，請考慮這些電腦的實體安全性。 電腦位於何處？哪些人需要存取這些電腦？ 您做出這些決定之後，可依據邏輯將電腦對應到多個容錯網域 (請參閱下一個步驟)。 生產叢集的基礎結構規劃比起測試叢集更為複雜。

## <a name="determine-the-number-of-fault-domains-and-upgrade-domains"></a>決定容錯網域和升級網域的數目
[*容錯網域* (FD)](service-fabric-cluster-resource-manager-cluster-description.md) 是故障的實體單元，而且與資料中心內的實體基礎結構直接相關。 容錯網域是由共用單一失敗點的硬體元件 (電腦、交換器、網路等) 所組成。 雖然容錯網域和機架之間沒有 1:1 對應，但是大致上來說，可以將每個機架視為一個容錯網域。

當您在 ClusterConfig.json 中指定 FD 時，可以選擇每個 FD 的名稱。 Service Fabric 支援階層式 FD，因此，您可以在 FD 中反映您的基礎結構拓撲。  例如，下列 FD 有效：

* "faultDomain": "fd:/Room1/Rack1/Machine1"
* "faultDomain": "fd:/FD1"
* "faultDomain": "fd:/Room1/Rack1/PDU1/M1"

*升級網域* (UD) 是節點的邏輯單元。 在 Service Fabric 協調式升級 (應用程式升級或叢集升級) 期間，會關閉 UD 中的所有節點來執行升級，而其他 UD 中的節點則仍然可用來提供要求。 您對電腦所進行的韌體升級不會接受 UD，因此您必須一次在一部電腦上進行。

思考這些概念最簡單的方式就是將 FD 視為非計劃性故障的單元，並將 UD 視為計劃性維護的單元。

當您在 ClusterConfig.json 中指定 UD 時，可以選擇每個 UD 的名稱。 例如，下列名稱有效：

* "upgradeDomain": "UD0"
* "upgradeDomain": "UD1A"
* "upgradeDomain": "DomainRed"
* "upgradeDomain": "Blue"

如需詳細 FD 和 UD 的詳細資訊，請參閱[描述 Service Fabric 叢集](service-fabric-cluster-resource-manager-cluster-description.md)。

如果您對節點的維修和管理有完整控制權，也就是您負責更新和更換機器，生產環境中的叢集應該跨越至少 3 個 FD，以便在生產環境中獲得支援。 對於在您不需對機器具有完整控制權的環境中執行的叢集 (也就是 Amazon Web Services VM 執行個體)，在您的叢集中應該至少有 5 個 FD。 每個 FD 可以有一或多個節點。 這是為了防止機器升級和更新所造成的問題，因為根據執行的時機，可能會干擾叢集中執行的應用程式和服務。

## <a name="determine-the-initial-cluster-size"></a>決定初始叢集大小

一般而言，叢集中的節點數目是根據您的業務需求 (也就是說，叢集上將執行的服務和容器的數目)，以及要維持您的工作負載所需的資源數目而決定。 針對生產環境叢集，我們建議在您的叢集中至少有五個節點，跨越 5 個 FD。 不過，如上所述，如果您可以完整控制您的節點，並且可以跨越 3 個 FD，那個，3 個節點應該也可行。

執行可設定狀態工作負載的測試叢集應該有三個節點，而在執行無狀態工作負載的測試叢集則只需要一個節點。 您應該注意，基於開發目的，在一部指定的電腦上可以有多個節點。 不過，在生產環境中，對於每個實體或虛擬機器，Service Fabric 只支援一個節點。

## <a name="prepare-the-machines-that-will-serve-as-nodes"></a>準備將做為節點的機器

以下是對您想要新增到叢集的每部機器的一些建議規格：

* 至少 16 GB 的 RAM
* 至少 40 GB 的可用磁碟空間
* 4 核心或更高的 CPU
* 所有電腦的安全網路連線
* 已安裝 Windows Server 作業系統 (有效版本：2012 R2、2016、1709 或 1803)
* [.NET Framework 4.5.1 或更高版本](https://www.microsoft.com/download/details.aspx?id=40773)，完整安裝
* [Windows PowerShell 3.0](https://msdn.microsoft.com/powershell/scripting/setup/installing-windows-powershell)
* [RemoteRegistry 服務](https://technet.microsoft.com/library/cc754820) 應該在所有電腦上執行

部署和設定叢集的叢集系統管理員必須擁有每部電腦的 [系統管理員權限](https://social.technet.microsoft.com/wiki/contents/articles/13436.windows-server-2012-how-to-add-an-account-to-a-local-administrator-group.aspx) 。 您無法在網域控制站上安裝 Service Fabric。

## <a name="download-the-service-fabric-standalone-package-for-windows-server"></a>下載適用於 Windows Server 的 Service Fabric 獨立封裝
[下載連結 - Service Fabric 獨立封裝 - Windows Server](http://go.microsoft.com/fwlink/?LinkId=730690)，並將封裝解壓縮至不屬於叢集一部分的部署電腦，或解壓縮至將屬於叢集的其中一部電腦。

## <a name="modify-cluster-configuration"></a>修改叢集組態
若要建立獨立叢集，您必須建立獨立叢集組態 ClusterConfig.json 檔案，其中描述叢集的規格。 您可以根據可在下面連結找到的範本來建立組態檔。 <br>
[獨立叢集組態](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)

如需此檔案中各個區段的詳細資訊，請參閱[獨立 Windows 叢集的組態設定](service-fabric-cluster-manifest.md)。

從您下載的封裝中開啟其中一個 ClusterConfig.json 檔案，然後修改下列設定︰
| **組態設定** | **說明** |
| --- | --- |
| **NodeTypes** |節點類型可讓您將叢集節點分成不同的群組。 一個叢集至少必須有一個節點類型。 群組中的所有節點都有下列共同的特性： <br> **Name** - 這是節點類型名稱。 <br>**Endpoint Ports** - 這些是與這個節點類型相關聯的各種具名端點 (連接埠)。 您可以使用任何您想要的連接埠號碼，只要該號碼未與此資訊清單中的其他任何號碼衝突，而且目前沒有任何其他在電腦/VM 上執行的應用程式在使用該號碼即可。 <br> **Placement Properties** - 此節點類型的這些屬性是用來做為系統服務或您的服務的放置條件約束。 這些屬性是使用者定義的索引鍵/值組，可針對指定節點提供額外的中繼資料。 節點屬性的範例包括節點是否有硬碟機或圖形卡、其硬碟機的磁針數、核心，以及其他實體屬性。 <br> **Capacities** - 節點容量會定義特定節點可以使用的特定資源名稱和數量。 例如，節點可能會定義它具有名為 "MemoryInMb" 的度量容量，而且預設有 2048 MB 的可用記憶體。 這些容量會在執行階段使用，以確保需要特定資源數量的服務會放在需要的數量中有這些資源的節點上。<br>**IsPrimary** - 如果有一個以上已定義的節點類型，請確定只有一個設為主要 (且值為 *true*)，這是系統服務執行的位置。 其他所有節點類型應該設定為值 *false* |
| **Nodes** |這些是屬於叢集一部分的每個節點的詳細資料 (節點類型、節點名稱、IP 位址、節點的容錯網域和升級網域)。 您想要建立叢集所在的電腦必須與其 IP 位址一起列在這裡。 <br> 如果您為所有節點使用相同的 IP 位址，則會建立一整體叢集，您可以將此叢集用於測試之用。 不要使用一整體叢集部署生產工作負載。 |

完成設定叢集組態的所有設定之後，即可針對叢集環境 (步驟 7) 進行測試。

<a id="environmentsetup"></a>

## <a name="environment-setup"></a>環境設定

當叢集系統管理員設定 Service Fabric 獨立叢集時，必須根據下列準則設定環境︰ <br>
1. 對叢集組態檔中列為節點的所有電腦，建立叢集的使用者應擁有系統管理員層級的安全性權限。
2. 所建立叢集中的機器以及每個叢集節點電腦都必須︰
* 已將 Service Fabric SDK 解除安裝
* 已將 Service Fabric 執行階段解除安裝 
* 已啟用 Windows 防火牆服務 (mpssvc)
* 已啟用遠端登錄服務 (遠端登錄)
* 已啟用檔案共用 (SMB)
* 已根據叢集組態的連接埠，開啟必要的連接埠
* 已經開啟 Windows SMB 和遠端登錄服務的必要通訊埠︰135、137、138、139 和 445
* 網路已彼此連線
3. 叢集節點電腦應無一是網域控制站。
4. 如果要部署的叢集是安全叢集，驗證安全性必要條件已就緒，並已正確設定組態。
5. 如果叢集電腦不可存取網際網路，在叢集組態中設定下列各項：
* 停用遙測：在 *properties* 下設定 *"enableTelemetry": false*
* 停用自動的 Fabric 版本下載，以及目前叢集版本接近終止支援的通知：在 [屬性] 底下，設定 *"fabricClusterAutoupgradeEnabled": false*
* 或者，如果網路的網際網路存取僅限於允許清單上的網域，則自動升級需要下列網域：   go.microsoft.com   download.microsoft.com

6. 設定適當的 Service Fabric 防毒排除項目︰

| **防毒排除目錄** |
| --- |
| Program Files\Microsoft Service Fabric |
| FabricDataRoot (從叢集組態) |
| FabricLogRoot (從叢集組態) |

| **防毒排除處理序** |
| --- |
| Fabric.exe |
| FabricHost.exe |
| FabricInstallerService.exe |
| FabricSetup.exe |
| FabricDeployer.exe |
| ImageBuilder.exe |
| FabricGateway.exe |
| FabricDCA.exe |
| FabricFAS.exe |
| FabricUOS.exe |
| FabricRM.exe |
| FileStoreService.exe |

## <a name="validate-environment-using-testconfiguration-script"></a>使用 TestConfiguration 指令碼驗證環境
您可以在獨立封裝中找到 TestConfiguration.ps1 指令碼。 它可做為「最佳做法分析程式」用來驗證上述的一些準則，並應做為例行性檢查用來驗證是否能在指定的環境上部署叢集。 如果發生任何錯誤，請參閱[環境設定](service-fabric-cluster-standalone-deployment-preparation.md)下的清單進行疑難排解。 

此指令碼可以在以系統管理員身分存取叢集組態檔中列為節點的所有電腦的任何電腦上執行。 執行此指令碼所在的電腦不一定是叢集的一部分。

```powershell
PS C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer> .\TestConfiguration.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json
Trace folder already exists. Traces will be written to existing trace folder: C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer\DeploymentTraces
Running Best Practices Analyzer...
Best Practices Analyzer completed successfully.


LocalAdminPrivilege        : True
IsJsonValid                : True
IsCabValid                 : True
RequiredPortsOpen          : True
RemoteRegistryAvailable    : True
FirewallAvailable          : True
RpcCheckPassed             : True
NoConflictingInstallations : True
FabricInstallable          : True
Passed                     : True
```

目前此組態測試模組並不會驗證安全性設定，因此這必須獨立完成。  

> [!NOTE]
> 我們會持續進行改進，以讓此模組更穩健，因此如果您認為目前有 TestConfiguration 攔截不到的錯誤或有疏漏之處，請透過我們的[支援通道](https://docs.microsoft.com/azure/service-fabric/service-fabric-support)來告訴我們。   
> 
> 

## <a name="next-steps"></a>後續步驟
* [建立在 Windows Server 上執行的獨立叢集](service-fabric-cluster-creation-for-windows-server.md)
