---
title: Microsoft Azure Service Fabric 的相關常見問題 | Microsoft Docs
description: 和 Service Fabric 有關的常見問題集和解答
services: service-fabric
documentationcenter: .net
author: chackdan
manager: timlt
editor: ''
ms.assetid: 5a179703-ff0c-4b8e-98cd-377253295d12
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/18/2017
ms.author: chackdan
ms.openlocfilehash: 0b731e94675992e59f79b61a2f3a15fa20bdf8a7
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42146389"
---
# <a name="commonly-asked-service-fabric-questions"></a>Service Fabric 的常見問題

和 Service Fabric 的功能及使用方式有關的常見問題很多。 本文件涵蓋許多這類問題和解答。

## <a name="cluster-setup-and-management"></a>叢集設定和管理

### <a name="how-do-i-roll-back-my-service-fabric-cluster-certificate"></a>如何復原 Service Fabric 叢集憑證？

若要復原應用程式的任何升級，唯有發生健康情況失敗偵測，Service Fabric 叢集仲裁才會認可變更；認可後的變更只能向前復原。 如果您引進未受監視的中斷性憑證變更，若要復原叢集，您可能需要透過客戶支援服務提報工程師。  [Service Fabric 的應用程式升級](https://review.docs.microsoft.com/azure/service-fabric/service-fabric-application-upgrade?branch=master)會套用[應用程式升級參數](https://review.docs.microsoft.com/azure/service-fabric/service-fabric-application-upgrade-parameters?branch=master)，以及履行不需要停機升級承諾。  完成我們建議的應用程式升級監視模式後，是否能自動進展到更新網域，端視應用程式是否通過健康情況檢查，如果預設服務更新失敗，系統將進行自動復原。
 
如果叢集的 Resource Manager 範本還在使用傳統憑證指紋屬性，建議您[將叢集從憑證指紋變更為通用名稱](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-change-cert-thumbprint-to-cn)，這樣才能運用現代化的秘密管理功能。

### <a name="can-i-create-a-cluster-that-spans-multiple-azure-regions-or-my-own-datacenters"></a>我能否建立跨多個 Azure 區域或自有資料中心的叢集？

是。 

核心的 Service Fabric 叢集技術可用來結合在世界各地執行的機器，只要它們彼此間有網路連線即可。 不過，建置和執行這類叢集的程序很複雜。

如果您對於這個案例有興趣，建議您透過 [Service Fabric Github 問題清單](https://github.com/azure/service-fabric-issues)或透過您的支援代表來進行聯繫，以取得其他指引。 Service Fabric 小組正在針對這個案例著手提供其他清楚說明、指引和建議。 

需考量的事項： 

1. 目前在 Azure 中的 Service Fabric 叢集資源是地區性的，據以建置叢集的虛擬機器擴展集也是。 這表示在發生區域失敗時，您可能就無法透過 Azure Resource Manager 或 Azure 入口網站來管理叢集。 即使叢集仍在執行，而且您可以直接與其進行互動，還是可能發生此情況。 此外，Azure 目前並無法讓您擁有可跨區域使用的單一虛擬網路。 這表示如果您想在 Azure 中擁有多區域叢集，就需要 [VM 擴展集中每個 VM 的公用 IP 位址](../virtual-machine-scale-sets/virtual-machine-scale-sets-networking.md#public-ipv4-per-virtual-machine)或 [Azure VPN 閘道](../vpn-gateway/vpn-gateway-about-vpngateways.md)。 這些網路選擇對於成本、效能以及某種程度的應用程式設計會有不同的影響，因此請先謹慎地分析和規劃，再建立這樣的環境。
2. 維護、管理和監視這些機器的程序可能會變得複雜，特別是在跨_數種類型_的環境時，例如在不同雲端提供者之間或在內部部署資源和 Azure 之間。 請務必謹慎，確保您已了解叢集和應用程式的升級、監視、管理和診斷方式，再於這類環境中執行生產工作負載。 如果您對於在 Azure 中或自己的資料中心內解決這些問題已有豐富經驗，則在建置或執行 Service Fabric 叢集時或許也可以套用同樣的一套解決方案。 

### <a name="do-service-fabric-nodes-automatically-receive-os-updates"></a>Service Fabric 節點是否會自動接收作業系統更新？

目前不行，但這是常見的要求，Azure 有打算提供這項功能。

在過渡期間，我們[提供了一個應用程式](service-fabric-patch-orchestration-application.md)，您的 Service Fabric 節點下的作業系統會持續修補並保持最新狀態。

作業系統更新的挑戰是它們通常需要重新啟動機器，這會造成暫時失去可用性。 就其本身來說並不是問題，因為 Service Fabric 會自動將那些服務的流量重新導向至其他節點。 不過，如果作業系統更新不會跨叢集協調，則有可能讓多個節點同時停機。 這類同時重新啟動會造成服務，或至少造成某個特定分割 (如果為具狀態服務) 完全失去可用性。

未來，我們計劃支援在所有更新網域中完全自動化且協調化的作業系統更新原則，確保在發生重新開機或其他非預期失敗的情況下，都能夠維持可用性。

### <a name="can-i-use-large-virtual-machine-scale-sets-in-my-sf-cluster"></a>我可以在 SF 叢集中使用大型虛擬機器擴展集嗎？ 

**簡短回答** - 否。 

**完整回答** - 雖然大型虛擬機器擴展集最多可讓您將虛擬機器擴展集調整到 1000 個 VM 執行個體，不過這是藉由使用放置群組 (PG) 來達成的。 容錯網域 (FD) 和升級網域 (UD) 只會在使用 FD 和 UD 以決定服務複本/服務執行個體放置位置的放置群組 Service Fabric 內保持一致。 由於 FD 和 UD 只能在放置群組內比較，所以 SF 無法使用它。 例如，如果 PG1 中的 VM1 具有 FD=0 的拓撲，且 PG2 中的 VM9 具有 FD=4 的拓撲，此情況並不代表 VM1 和 VM2 位於兩個不同的硬體機架上，因此在此情況下，SF 無法使用 FD 值來決定放置位置。

大型虛擬機器擴展集目前有其他問題，像是缺乏層級 4 的負載平衡支援。 如需詳細資訊，請參閱[大型擴展集詳細資料](../virtual-machine-scale-sets/virtual-machine-scale-sets-placement-groups.md)



### <a name="what-is-the-minimum-size-of-a-service-fabric-cluster-why-cant-it-be-smaller"></a>Service Fabric 叢集的大小下限為何？ 為什麼無法更小？

執行生產工作負載的 Service Fabric 叢集所支援的大小下限為五個節點。 針對開發/測試案例，我們支援三個節點的叢集。

之所以會有這些最低限制，是因為 Service Fabric 叢集會執行一組可設定狀態的系統服務，包括命名服務及容錯移轉管理員。 這些服務會根據強式一致性，追蹤哪些服務已經部署至叢集，以及它們目前的裝載位置。 該強式一致性接著會根據取得「仲裁」的能力，提供任何指定的更新給那些服務的狀態，其中仲裁代表指定之服務的特定多數複本 (N/2 + 1)。

了解該背景後，讓我們來檢查一些可能的叢集組態：

**一個節點**：這個選項不提供高可用性，因為任何原因造成的單一節點遺失都會導致整個叢集遺失。

**兩個節點**︰跨兩個節點 (N = 2) 部署的服務，仲裁為 2 (2/2 + 1 = 2)。 當單一複本遺失時，就無法建立仲裁。 由於執行服務升級時需要暫時關閉複本，因此這不是有用的組態。

**三個節點**︰使用三個節點 (N=3)，建立仲裁的需求仍然是兩個節點 (3/2 + 1 = 2)。 這表示您可以遺失一個單一節點，並且仍維持仲裁。

開發/測試可支援三節點叢集組態，因為您可以安全地執行更新，而且只要那些節點不要同時發生失敗，就能在個別節點失敗時維持叢集。 針對生產工作負載，您必須能夠從這類同時失敗復原，因此必須有五個節點。

### <a name="can-i-turn-off-my-cluster-at-nightweekends-to-save-costs"></a>我能否在晚上/週末關閉叢集以節省成本？

一般而言不行。 Service Fabric 會將狀態儲存在本機的暫時磁碟上，這表示如果虛擬機器移至不同的主機，資料將不會隨之移動。 在一般作業中，因為新節點會透過其他節點保持在最新狀態，所以這不會是問題。 不過，如果您停止所有節點並在稍後重新啟動它們，則很有可能讓大部分的節點在新的主機上啟動，而使系統無法復原。

如果您要建立叢集以在部署應用程式之前測試應用程式，建議您在[持續整合/持續部署管線](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)中動態建立那些叢集。


### <a name="how-do-i-upgrade-my-operating-system-for-example-from-windows-server-2012-to-windows-server-2016"></a>如何升級我的作業系統 (例如從 Windows Server 2012 升級到 Windows Server 2016)？

在我們努力改善體驗時，您的責任是升級。 您必須升級叢集的虛擬機器上的作業系統映像，一次一部 VM。 

### <a name="can-i-encrypt-attached-data-disks-in-a-cluster-node-type-virtual-machine-scale-set"></a>如何將叢集節點類型 (虛擬機器擴展集) 中已連結的資料磁碟加密？
是。  如需詳細資訊，請參閱[使用已連結的資料磁碟建立叢集](../virtual-machine-scale-sets/virtual-machine-scale-sets-attached-disks.md#create-a-service-fabric-cluster-with-attached-data-disks)、[加密磁碟 (PowerShell)](../virtual-machine-scale-sets/virtual-machine-scale-sets-encrypt-disks-ps.md)，以及[加密磁碟 (CLI)](../virtual-machine-scale-sets/virtual-machine-scale-sets-encrypt-disks-cli.md)。

### <a name="what-are-the-directories-and-processes-that-i-need-to-exclude-when-running-an-anti-virus-program-in-my-cluster"></a>當我在叢集中執行防毒程式時需要排除哪些目錄和處理序？

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
 
### <a name="how-can-my-application-authenticate-to-keyvault-to-get-secrets"></a>我的應用程式要如何向 KeyVault 進行驗證來取得祕密？
您的應用程式可使用下列方法取得向 KeyVault 驗證所需的認證：

A. 在您的應用程式建置/封裝作業期間，您可以將憑證提取到 SF 應用程式的資料套件中，並以此憑證向 KeyVault 驗證。
B. 針對已啟用虛擬機器擴展集 MSI 的主機，您可以針對 SF 應用程式開發簡單的 PowerShell SetupEntryPoint，以取得[來自 MSI 端點的存取權杖](https://docs.microsoft.com/en-us/azure/active-directory/managed-service-identity/how-to-use-vm-token)，然後[從 KeyVault 擷取您的祕密](https://docs.microsoft.com/en-us/powershell/module/azurerm.keyvault/Get-AzureKeyVaultSecret?view=azurermps-6.5.0)

## <a name="application-design"></a>應用程式設計

### <a name="whats-the-best-way-to-query-data-across-partitions-of-a-reliable-collection"></a>在可靠集合的所有分割中查詢資料的最佳方式為何？

可靠的集合通常[經過分割](service-fabric-concepts-partitioning.md)以使用相應放大來提供更好的效能及輸送量。 這表示指定服務的狀態可能跨越數十部或數百部機器。 若要在完整的資料集上執行作業，您有幾個選項：

- 建立會查詢另一個服務所有分割的服務，以提取所需資料。
- 建立可接收來自另一個服務所有分割之資料的服務。
- 定期從每個服務將資料推送至外部存放區。 只有在您要執行的查詢不是核心商業邏輯的一部分時，才適用此方法。


### <a name="whats-the-best-way-to-query-data-across-my-actors"></a>在所有動作項目查詢資料的最佳方式為何？

動作項目是設計為獨立的狀態與計算單位，因此不建議在執行階段執行動作項目狀態的各種查詢。 如果您有必要查詢完整的動作項目狀態集合，您也應該考慮：

- 以可設定狀態的可靠服務取代動作項目服務，以便收集從動作項目數目到服務中分割數目之所有資料的網路要求數目。
- 設計您的動作項目，定期將其狀態推送至外部存放區以便能夠更容易地查詢。 如上所述，只有在您執行的查詢並非執行階段行為的必要項目時，才適用此方法。

### <a name="how-much-data-can-i-store-in-a-reliable-collection"></a>可靠的集合中能夠儲存多少資料？

可靠的服務通常經過分割，因此您可以儲存的資料量僅受限於叢集中您擁有的機器數量，以及那些機器提供的記憶體數量。

例如，假設您在服務中擁有的可靠集合含有 100 個分割及 3 個複本，儲存的物件大小平均為 1 kb。 現在，假設您有 10 部機器的叢集，每部機器均含有 16GB 記憶體。 為了簡單與謹慎起見，假設作業系統及系統服務 (Service Fabric 執行階段以及您的服務) 會使用 6GB 的記憶體，並在每部機器上保留 10GB 以供使用，或是為該叢集保留 100GB 記憶體以供使用。

請記住，每個物件必須儲存三次 (一次主要，兩次複本)，在以完整容量運作的情況下，您會有足夠的記憶體提供您集合中大約 3500 萬個物件使用。 不過，我們建議您要能夠從失敗網域以及升級網域同時遺失情況下還原，這代表約 1/3 的容量，而物件數目會減至約 2300 萬。

請注意，這項計算也假設：

- 資料在所有分割上的分佈大致上是平均的，或您會對叢集資源管理員報告負載計量。 根據預設，Service Fabric 會根據複本數量進行負載平衡。 在上述範例中，叢集中的每個節點上會放置 10 個主要複本以及 20 個次要複本。 這適用於平均分佈於所有分割的負載。 如果負載不平均，您必須報告負載，以便資源管理員可以將較小的複本封裝在一起，讓較大的複本可以使用個別節點上較多的記憶體。

- 發生問題的可靠服務為叢集中唯一的儲存狀態。 由於您可以部署多個服務到叢集，因此您必須留意每個服務在執行及管理其狀態時所需的資源。

- 叢集本身不會擴大或縮小。 如果您新增更多機器，Service Fabric 將會重新平衡您的複本以利用額外的容量，直到機器數量超出服務中的分割數量為止，因為個別複本無法跨越機器。 反之，如果您移除機器以縮小叢集大小，您的複本會更緊密地封裝並縮小整體容量。

### <a name="how-much-data-can-i-store-in-an-actor"></a>動作項目中可儲存多少資料？

正如同可靠的服務，您可以在動作項目服務中儲存的資料量僅受限於叢集中所有節點可用的整體磁碟空間以及記憶體。 不過，個別的動作項目在用來封裝少量的狀態以及相關聯商務邏輯時最為有效。 一般而言，個別的動作項目應該會有以 kb 為單位所測量的狀態。

## <a name="other-questions"></a>其他問題

### <a name="how-does-service-fabric-relate-to-containers"></a>Service Fabric 如何與容器相關聯？

容器會提供簡單的方式來封裝服務及其相依性，使它們可以在所有環境中一致地執行，並且可在單一機器上以隔離的方式運作。 Service Fabric 提供部署與管理服務的方式，包括[已經包裝於容器中的服務](service-fabric-containers-overview.md)。

### <a name="are-you-planning-to-open-source-service-fabric"></a>您打算開放 Service Fabric 原始碼嗎？

我們在 GitHub 上有開放部分 Service Fabric 原始碼 ([可靠的服務架構](https://github.com/Azure/service-fabric-services-and-actors-dotnet)、[可靠的執行者架構](https://github.com/Azure/service-fabric-services-and-actors-dotnet)、[ASP.NET Core 整合程式庫](https://github.com/Azure/service-fabric-aspnetcore)、[Service Fabric Explorer](https://github.com/Azure/service-fabric-explorer)，以及 [Service Fabric CLI](https://github.com/Azure/service-fabric-cli))，並接受社群對這些專案的貢獻。 

我們[最近宣佈](https://blogs.msdn.microsoft.com/azureservicefabric/2018/03/14/service-fabric-is-going-open-source/)計劃開放 Service Fabric 執行階段原始碼。 此時，我們透過 Linux 組建和測試工具在 GitHub 上安裝 [Service Fabric 存放庫](https://github.com/Microsoft/service-fabric/)，這表示您可以複製存放庫、組建適用於 Linux 的 Service Fabric、執行基本測試、開啟問題，並提交提取要求。 我們正努力讓 Windows 組建環境以及完整的 CI 環境進行移轉。

請關注 [Service Fabric 部落格](https://blogs.msdn.microsoft.com/azureservicefabric/)，以便在公告更多詳細資料時進行了解。

## <a name="next-steps"></a>後續步驟

- [深入了解核心 Service Fabric 概念和最佳做法 (英文)](https://mva.microsoft.com/en-us/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tbuZM46yC_5206218965)
