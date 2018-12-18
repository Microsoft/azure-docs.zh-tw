---
title: 適用於開發人員的 Azure Batch 概觀 | Microsoft Docs
description: 從開發觀點了解 Batch 服務的功能及其 API。
services: batch
documentationcenter: .net
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: 416b95f8-2d7b-4111-8012-679b0f60d204
ms.service: batch
ms.devlang: multiple
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: big-compute
ms.date: 08/22/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8b6e543a4835410368e752e70e7e8cb6d8805c0e
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45735574"
---
# <a name="develop-large-scale-parallel-compute-solutions-with-batch"></a>使用 Batch 開發大規模的平行運算解決方案

在 Azure Batch 服務的核心元件概觀中，我們會討論 Batch 開發人員可用來建置大規模平行計算解決方案的主要服務功能和資源。

不論您正在開發可發出直接 [REST API][batch_rest_api] 呼叫的分散式計算應用程式或服務，或是正在使用其中一個 [Batch SDK](batch-apis-tools.md#azure-accounts-for-batch-development)，您都會使用此文章所討論的眾多資源和功能。

> [!TIP]
> 如需更高層級的 Batch 服務簡介，請參閱 [Azure Batch 的基本概念](batch-technical-overview.md)。 另請參閱最新的 [Batch 服務更新](https://azure.microsoft.com/updates/?product=batch)。
>
>

## <a name="batch-service-workflow"></a>Batch 服務工作流程
下列高階工作流程是幾乎所有使用 Batch 服務處理平行工作負載的應用程式和服務典型︰

1. 將您要處理的**資料檔**上傳至 [Azure 儲存體][azure_storage]帳戶。 Batch 包含可供存取 Azure Blob 儲存體的內建支援，而在執行工作時，您的工作可以將這些檔案下載至 [計算節點](#compute-node) 。
2. 上傳您的工作將要執行的 **應用程式檔案** 。 這些檔案可以是二進位檔或指令碼及其相依項目，並由您作業中的工作執行。 您的工作可以從儲存體帳戶下載這些檔案，或者您可以使用 Batch 的 [應用程式套件](#application-packages) 功能來管理和部署應用程式。
3. 建立計算節點的 [集區](#pool) 。 當您建立集區時，會指定集區的計算節點數目、其大小和作業系統。 當您作業中的每個工作執行時，會指派其在您集區的其中一個節點上執行。
4. 建立 [作業](#job)。 作業可管理一群工作。 您可以將每項作業關聯至將執行該作業之工作的特定集區。
5. 將 [工作](#task) 加入至作業。 每個工作會執行您上傳的應用程式或指令碼，以處理它從您的儲存體帳戶下載的資料檔案。 當每個工作完成時，即可將其輸出上傳至 Azure 儲存體。
6. 監視作業進度並從 Azure 儲存體擷取工作輸出。

下列各節討論可達成分散式計算案例的 Batch 資源。

> [!NOTE]
> 您需要有 [Batch 帳戶](#account)才能使用 Batch 服務。 大部分 Batch 解決方案也會使用相關聯的 [Azure 儲存體][azure_storage]帳戶來儲存和擷取檔案。 
>
>

## <a name="batch-service-resources"></a>Batch 服務資源
使用 Batch 服務的所有解決方案需要下列某些資源：帳戶、計算節點、集區、作業和工作。 其他資源 (如作業排程和應用程式套件) 都很實用，但為選用功能。

* [帳戶](#account)
* [計算節點](#compute-node)
* [集區](#pool)
* [作業](#job)
  * [作業排程](#scheduled-jobs)
* [Task](#task)
  * [啟動工作](#start-task)
  * [作業管理員工作](#job-manager-task)
  * [作業準備和作業釋放工作](#job-preparation-and-release-tasks)
  * [多重執行個體工作 (MPI)](#multi-instance-tasks)
  * [作業相依性](#task-dependencies)
* 套件[應用程式套件](#application-packages)

## <a name="account"></a>帳戶
批次帳戶是批次服務內唯一識別的實體。 所有處理都與 Batch 帳戶相關聯。

您可以使用 [Azure 入口網站](batch-account-create-portal.md)或以程式設計的方式建立 Azure Batch 帳戶，例如使用 [Batch管理 .NET 程式庫](batch-management-dotnet.md)。 建立帳戶時，您可以將 Azure 儲存體帳戶產生關聯，以儲存作業相關的輸入和輸出資料或應用程式。

您可以在單一 Batch 帳戶中執行多個 Batch 工作負載，或在相同訂用帳戶 (但不同 Azure 區域) 的 Batch 帳戶之間散佈工作負載。

[!INCLUDE [batch-account-mode-include](../../includes/batch-account-mode-include.md)]

## <a name="azure-storage-account"></a>Azure 儲存體帳戶

大部分 Batch 解決方案都使用 Azure 儲存體來儲存資源檔和輸出檔。 例如，您的 Batch 工作 (包括標準工作、啟動工作、作業準備工作和作業發行工作) 通常會指定位於儲存體帳戶中的資源檔案。

Batch 支援下列類型的「Azure 儲存體」帳戶：

* 一般用途 v2 (GPv2) 帳戶 
* 一般用途 v1 (GPv1) 帳戶
* Blob 儲存體帳戶 (目前支援虛擬機器組態中的集區)

如需有關儲存體帳戶的詳細資訊，請參閱 [Azure 儲存體帳戶概觀](../storage/common/storage-account-overview.md)。

您可以在建立 Batch 帳戶時或在稍後，建立儲存體帳戶與 Batch 帳戶的關聯。 在選擇儲存體帳戶時，請考慮您的成本和效能需求。 例如，相較於 GPv1，GPv2 和 Blob 儲存體帳戶選項支援更大的[容量和延展性限制](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/)。 (請連絡 Azure 支援以要求增加儲存體限制。)對於包含讀取自或寫入至儲存體帳戶之大量平行工作的 Batch 解決方案，這些帳戶選項可以改善其效能。

## <a name="compute-node"></a>計算節點
計算節點是 Azure 虛擬機器 (VM) 或雲端服務 VM，專門用來處理您應用程式的部分工作負載。 節點大小決定配置給節點的 CPU 核心數目、記憶體容量，以及本機檔案系統大小。 您可以使用 Azure 雲端服務、[Azure 虛擬機器 Marketplace][vm_marketplace] 中的映像，或您準備的自訂映像來建立 Windows 或 Linux 節點的集區。 如需這些選項的詳細資訊，請參閱下列 [集區](#pool) 。

節點可以執行節點作業系統環境所支援的任何可執行檔或指令碼。 這包括適用於 Windows 的 \*.exe、\*.cmd、\*.bat 和 PowerShell 指令碼，以及適用於 Linux 的二進位檔、Shell 和 Python 指令碼。

Batch 中的所有計算節點也包括︰

* 工作可參考的標準[資料夾結構](#files-and-directories)與相關聯的[環境變數](#environment-settings-for-tasks)。
* **防火牆** 設定。
* [遠端存取](#connecting-to-compute-nodes) Windows (遠端桌面通訊協定 (RDP)) 和 Linux (安全殼層 (SSH)) 節點。

## <a name="pool"></a>集區
集區是應用程式執行所在的一群節點。 集區可以由您手動建立，或當您指定要完成的工作時，由 Batch 服務自動建立。 您可以建立和管理符合應用程式資源需求的集區。 集區只能由建立它的 Batch 帳戶使用。 批次帳戶可以有多個集區。

Azure Batch 集區的建置基礎為核心 Azure 計算平台。 這些集區可提供大規模的配置、應用程式安裝、資料散發、健全狀況監視，以及在集區內彈性調整計算節點數目 ([調整規模](#scaling-compute-resources))。

系統會指派唯一的名稱及 IP 位址給新增至集區的每個節點。 當節點從集區中移除時，就會失去對作業系統或檔案所做的任何變更，且其名稱及 IP 位址都會釋出供未來使用。 當節點離開集區時，其存留期就結束。

當您建立集區時，您可以指定下列屬性：

- 計算節點作業系統和版本
- 計算節點類型和目標節點數目
- 計算節點的大小
- 調整原則
- 工作排程原則
- 計算節點的通訊狀態
- 計算節點的啟動工作
- 應用程式套件
- 網路組態

這些設定每一個都會在下列各節詳細說明。

> [!IMPORTANT]
> Batch 帳戶具有預設配額，以限制 Batch 帳戶中的核心數目。 對應到計算節點數目的核心數目。 在 [Azure Batch 服務的配額和限制](batch-quota-limit.md)中，您可以找到預設配額及如何[增加配額](batch-quota-limit.md#increase-a-quota)的說明。 如果您的集區未達其目標節點數目，可能是因為核心配額的原因。
>


### <a name="compute-node-operating-system-and-version"></a>計算節點作業系統和版本

當您建立 Batch 集區時，可以指定 Azure 虛擬機器設定，以及您想要在集區中的每個計算節點上執行之作業系統類型。 Batch 中可用的兩個設定類型為：

- **虛擬機器設定**，會指定集區是由 Azure 虛擬機器所組成。 這些 VM 可能會從 Linux 或 Windows 映像建立。 

    以虛擬機器設定作為基礎建立集區時，您不僅需要指定節點的大小和用來建立這些節點的映像來源，也必須在節點上安裝**虛擬機器映像參考**和 Batch **節點代理程式 SKU**。 如需指定這些集區屬性的詳細資訊，請參閱 [在 Azure Batch 集區中佈建 Linux 計算節點](batch-linux-nodes.md)。 您可以選擇性地將一或多個空的資料磁碟連結至從 Marketplace 映像建立的集區 VM，或是在自訂映像中包含資料磁碟，以建立 VM。

- **雲端服務設定**，會指定集區是由 Azure 雲端服務節點所組成。 雲端服務組態「只」提供 Windows 計算節點。

    雲端服務組態集區可用的作業系統列於 [Azure 客體 OS 版次與 SDK 相容性矩陣](../cloud-services/cloud-services-guestos-update-matrix.md)。 建立包含雲端服務節點的集區時，您需要指定節點大小及其「作業系統系列」。 雲端服務部署至 Azure 的速度比執行 Windows 的虛擬機器還快。 如果您需要 Windows 計算節點的集區，可能會發現雲端服務提供了部署時間方面的效能優勢。

    * 「OS 系列」  也會決定哪些版本的.NET 會與作業系統一起安裝。
    * 如同雲端服務內的背景工作角色，您可以指定 [OS 版本] (如需背景工作角色的詳細資訊，請參閱[雲端服務概觀](../cloud-services/cloud-services-choose-me.md))。
    * 如同背景工作角色，建議為 [OS 版本]指定 `*`，以便自動升級節點，而且不需為了因應新發行的版本而執行工作。 選取特定 OS 版本的主要使用案例是為了確保應用程式相容性，以允許在更新版本之前執行回溯相容性測試。 通過驗證之後，即可更新集區的 [OS 版本] 並安裝新的 OS 映像，如此會中斷任何執行中的工作並重新排入佇列。

當您建立一個集區時，必須選取適當的 **nodeAgentSkuId**，視您 VHD 的基礎映像 OS 而定。 您可以取得對應至其 OS 映像參考的可用節點代理程式 SKU 識別碼，方法是呼叫[列出受支援節點代理程式 SKU 的清單](https://docs.microsoft.com/rest/api/batchservice/list-supported-node-agent-skus)作業。


#### <a name="custom-images-for-virtual-machine-pools"></a>適用於虛擬機器集區的自訂映像

若要使用自訂映像，您必須將映像一般化來備妥映像。 如需有關從 Azure VM 準備自訂 Linux 映像的資訊，請參閱[如何建立虛擬機器或 VHD 的映像](../virtual-machines/linux/capture-image.md)。 如需有關從 Azure VM 準備自訂 Windows 映像的資訊，請參閱[在 Azure 中建立一般化 VM 的受控映像](../virtual-machines/windows/capture-image-resource.md)。 

如需詳細的需求和步驟，請參閱[使用自訂映像來建立虛擬機器的集區](batch-custom-images.md)。

#### <a name="container-support-in-virtual-machine-pools"></a>虛擬機器集區中的容器支援

使用 Batch API 建立虛擬機器設定集區時，您可以設定集區執行設定集區執行 Docker 容器中的工作。 目前，您必須使用支援 Docker 容器的映像來建立集區。 使用 Windows Server 2016 Datacenter 與 Azure Marketplace 中的容器映像，或者提供自訂 VM 映像，其中包含 Docker Community Edition 或 Enterprise Edition 和任何所需的驅動程式。 集區設定必須包含[容器設定](/rest/api/batchservice/pool/add#definitions_containerconfiguration)，該設定會在集區建立時將容器映像複製到 VM。 然後，在集區上執行的工作就可以參考容器映像和容器執行選項。

如需詳細資訊，請參閱[在 Azure Batch 上執行 Docker 容器應用程式](batch-docker-container-workloads.md)。

## <a name="compute-node-type-and-target-number-of-nodes"></a>計算節點類型和目標節點數目

當您建立集區時，您可以指定您想要的計算節點類型及每個類型的目標數目。 計算節點有兩個類型︰

- **專用計算節點。** 專用計算節點會保留給您的工作負載使用。 它們比低優先順序的節點昂貴，但保證永遠不會遭到佔用。

- **低優先順序的計算節點。** 低優先順序的節點會利用 Azure 中剩餘的容量來執行 Batch 工作負載。 低優先順序的節點每小時比專用節點更便宜，並可啟用需要大量計算能力的工作負載。 如需詳細資訊，請參閱[使用低優先順序的 VM 搭配 Batch](batch-low-pri-vms.md)。

    Azure 多餘的容量不足時，可能會佔用低優先順序的計算節點。 如果節點在執行工作時遭到佔用，工作會重新排入佇列並於計算節點再次可用時再次執行。 低優先順序的節點很適合用於作業完成時間有彈性且工作分散於許多節點的工作負載。 在決定使用適合您案例的低優先順序節點之前，請確定會盡可能降低因優先佔用而遺失的工作數，且可輕鬆重新建立。

    
您可以在相同的集區中同時擁有低優先順序和專用的計算節點。 每種節點類型 &mdash; 低優先順序和專用&mdash; 有它自己的目標設定，您可以對其指定所需的節點數目。 
    
計算節點數目稱為「目標」，因為在某些情況下，您的集區可能無法達到所需的節點數目。 例如，如果集區先達到 Batch 帳戶的[核心配額](batch-quota-limit.md)，它就可能無法達成目標。 或者，如果您已將可限制節點數目上限的自動調整公式套用至集區，集區可能無法達成目標。

如需低優先順序和專用計算節點的價格資訊，請參閱 [Batch 價格](https://azure.microsoft.com/pricing/details/batch/)。

### <a name="size-of-the-compute-nodes"></a>計算節點的大小

當您建立 Azure Batch 集區時，您可以從幾乎所有的 VM 系列及 Azure 中可用的大小進行選擇。 Azure 針對不同的工作負載提供各種 VM 大小，包括特製化的 [HPC](../virtual-machines/linux/sizes-hpc.md)或[已啟用的 GPU](../virtual-machines/linux/sizes-gpu.md) VM 大小。 

如需詳細資訊，請參閱[為 Azure Batch 集區中的運算節點選擇 VM 大小](batch-pool-vm-sizes.md)。

### <a name="scaling-policy"></a>調整原則

針對動態工作負載，您可以撰寫並將[自動調整公式](#scaling-compute-resources)套用至集區。 Batch 服務將會定期評估您的公式，並根據可以指定的各種集區、作業、和工作參數，調整集區中的節點數目。

### <a name="task-scheduling-policy"></a>工作排程原則

[每個節點的工作數上限](batch-parallel-node-tasks.md) 組態選項會決定可在集區內的每個計算節點上平行執行的工作數目上限。

預設組態會指定在節點上一次執行一個工作，但在某些情況下，在一個節點上同時執行兩項以上工作較為有利。 請參閱[並行節點工作](batch-parallel-node-tasks.md)一文中的[範例案例](batch-parallel-node-tasks.md#example-scenario)，以了解如何從每個節點的多個工作受惠。

您也可以指定「填滿類型」  ，以決定 Batch 是要將工作平均分散到集區中的所有節點，還是將最大數目的工作分配給一個節點後，再將工作指派給另一個節點。

### <a name="communication-status-for-compute-nodes"></a>計算節點的通訊狀態

大部分情況下，工作會獨立運作，並不需要彼此通訊。 不過，有一些工作必須進行通訊的應用程式，例如 [MPI 案例](batch-mpi.md)。

您可以設定集區來允許**節點間通訊**，以便集區內的節點可以在執行階段進行通訊。 啟用節點間通訊時，[雲端服務組態] 集區中的節點可以在超過 1100 個連接埠上彼此通訊，而且 [虛擬機器組態] 集區並不會限制任何連接埠的流量。

請注意，啟用節點間通訊也會影響叢集內的節點位置，而且由於部署限制，可能會限制集區中的節點數目上限。 如果您的應用程式不需要節點之間的通訊，Batch 服務可以將許多不同叢集和資料中心的大量節點配置給集區，以發揮更強大的平行處理能力。

### <a name="start-tasks-for-compute-nodes"></a>計算節點的啟動工作

選用的「啟動工作」  將在每個節點加入集區以及每次重新啟動節點或重新安裝其映像時，於該節點上執行。 啟動工作特別適合用於準備計算節點，以便執行工作，例如在計算節點上安裝工作要執行的應用程式。

### <a name="application-packages"></a>應用程式套件

您可以指定要部署至集區中計算節點的 [應用程式套件](#application-packages) 。 應用程式套件會提供您的工作執行之應用程式的簡化部署和版本控制。 您針對集區指定的應用程式套件會安裝於加入該集區的每個節點，以及在節點重新啟動或重新安裝映像時安裝。

> [!NOTE]
> 在 2017 年 7 月 5 日之後建立的所有 Batch 集區都支援應用程式套件。 只有在使用雲端服務設定建立集區時，在 2016 年 3 月 10 日與 2017 年 7 月 5 日之間所建立的 Batch 集區上才支援應用程式套件。 在 2016 年 3 月 10 日之前建立的 Batch 集區不支援應用程式套件。 如需使用應用程式套件將應用程式部署至 Batch 節點的詳細資訊，請參閱[使用 Batch 應用程式套件將應用程式部署至計算節點](batch-application-packages.md)。
>
>

### <a name="network-configuration"></a>網路組態

您可以指定 Azure [虛擬網路 (VNet)](../virtual-network/virtual-networks-overview.md) 的子網路，可在其中建立集區的計算節點。 如需詳細資訊，請參閱[集區網路組態](#pool-network-configuration)區段。


## <a name="job"></a>工作 (Job)
作業是工作的集合。 作業可管理其工作在集區中的計算節點上執行計算的方式。

* 作業會指定工作執行所在的**集區**。 您可以為每個作業建立新的集區，或將集區使用於許多工作。 您可以針對與作業排程相關聯的每項作業建立集區，或針對與作業排程相關聯的所有作業建立集區。
* 您可以指定選擇性的 **作業優先順序**。 使用高於其他進行中作業的優先順序提交作業時，較高優先順序的作業工作會插入在佇列中較低優先順序的作業工作之前。 已在執行中的較低優先順序作業中的工作不會被優先佔用。
* 您可以使用作業 **條件約束** 來為作業指定特定的限制：

    您可以設定 **最大時鐘時間**，因此如果作業的執行時間超過指定的最大時鐘時間，則會終止此作業和所有相關聯的工作。

    Batch 可以偵測並重試失敗的工作。 您可以指定**工作重試次數上限**作為條件約束，包括要「一律」重試工作還是「決不」重試工作。 重試工作表示工作會重新排入佇列，以再次執行。
* 用戶端應用程式可以在作業中新增工作，或者您可以指定 [作業管理員工作](#job-manager-task)。 作業管理員工作包含為作業建立必要工作所需的資訊，而作業管理員工作會在集區內的其中一個計算節點上執行。 Batch 會特別處理作業管理員工作，此工作會在作業建立後立即排入佇列，且如果失敗，則會重新啟動。 由[作業排程](#scheduled-jobs)建立的作業「需要」有作業管理員工作，因為它是在作業具現化之前唯一可定義工作的方法。
* 根據預設，作業內的所有工作都完成時，作業仍會保持作用中狀態。 您可以變更此行為，讓作業在其中的所有工作完成時自動終止。 將作業的 **onAllTasksComplete** 屬性 (在 Batch .NET 中為 [OnAllTasksComplete][net_onalltaskscomplete]) 設定為 *terminatejob*，以在作業的所有工作處於已完成狀態時，自動終止該作業。

    請注意，Batch 服務會將「沒有」  工作的作業視為其所有工作都已完成。 因此，這個選項最常搭配 [作業管理員工作](#job-manager-task)使用。 如果您想要使用自動作業終止，而不透過作業管理員，您一開始就應該將新作業的 **onAllTasksComplete** 屬性設定為 noaction，而只在您完成將工作新增至作業之後，將它設定為 terminatejob。

### <a name="job-priority"></a>作業優先順序
您可以指派優先順序給在 Batch 中建立的作業。 Batch 服務會使用作業的優先順序值，以決定帳戶內的作業排程順序 (這不會與 [排程的作業](#scheduled-jobs)混淆)。 優先順序值可以介於 -1000 到 1000，-1000 表示最低優先順序，1000 表示最高優先順序。 若要更新作業的屬性，呼叫[更新作業的屬性][rest_update_job]作業 (Batch REST) 或藉由修改 [CloudJob.Priority][net_cloudjob_priority] 屬性 (Batch .NET)。

在相同的帳戶內，較高優先順序的作業具有比低優先順序作業更高的排程優先順序。 一個帳戶中具有較高優先順序值的作業，其排程優先順序並不高於不同帳戶中較低優先順序值的另一個作業。

不同集區的作業排程是獨立的。 在不同的集區之間，即使作業的優先順序較高，如果其相關聯的集區缺少閒置的節點，並不保證此作業會優先排程。 在相同的集區上，相同優先順序等級的作業有相同的排程機會。

### <a name="scheduled-jobs"></a>Scheduled jobs
[作業排程][rest_job_schedules]可讓您在 Batch 服務內建立週期性作業。 作業排程會指定何時要執行作業，並且包含要執行之作業的規格。 您可以指定排程的持續時間 (排程的有效時間和生效時間)，以及在排程期間建立作業的頻率。

## <a name="task"></a>Task
工作是與作業相關聯的計算單位。 工作是在節點上執行。 工作會指派給節點以便執行，或排入佇列直到節點變成可用為止。 簡言之，工作會在計算節點上執行一或多個程式或指令碼，以執行您需要完成的工作。

建立工作時，您可以指定︰

* 工作的 **命令列**。 這是可在計算節點上執行應用程式或指令碼的命令列。

    請務必注意，命令列實際上不是在 Shell 底下執行。 因此，它無法以原生方式利用 Shell 功能，例如[環境變數](#environment-settings-for-tasks)擴充功能 (這包括 `PATH`)。 若要利用這類功能，您必須在命令列中叫用此 Shell，例如藉由在 Windows 節點上啟動 `cmd.exe` 或在 Linux 上啟動 `/bin/sh`︰

    `cmd /c MyTaskApplication.exe %MY_ENV_VAR%`

    `/bin/sh -c MyTaskApplication $MY_ENV_VAR`

    如果您的工作需要執行不在節點的 `PATH` 中的應用程式或指令碼，或參考環境變數，請在工作命令列中明確地叫用 Shell。
* **資源檔** 。 在工作的命令列執行之前，這些檔案會自動從 Azure 儲存體帳戶中的 Blob 儲存體複製到節點。 如需詳細資訊，請參閱[啟動工作](#start-task)和[檔案和目錄](#files-and-directories)章節。
* 應用程式所需的 **環境變數** 。 如需詳細資訊，請參閱 [工作的環境設定](#environment-settings-for-tasks) 一節。
* 執行工作所應根據的 **條件約束** 。 例如，條件約束包括允許執行工作的時間上限、失敗的工作應該重試的次數上限，以及檔案保留在工作的工作目錄中的時間上限。
* **Application packages** 。 [應用程式套件](#application-packages) 會提供您的工作執行之應用程式的簡化部署和版本控制。 在共用集區的環境中，工作層級的應用程式套件特別有用：不同的作業會在一個集區上執行，而某項作業完成時並不會刪除該集區。 如果您的作業擁有的工作少於集區中的節點，工作應用程式套件可以減少資料傳輸，因為您的應用程式只會部署至執行工作的節點。
* Docker 中樞或私人登錄中的**容器映像**參考和其他設定，用來建立 Docker 容器，工作會在其中的節點上執行。 如果集區是使用容器設定進行設定，您只能指定此資訊。

> [!NOTE]
> 工作的最長存留期 (從它新增至作業到完成時) 為 7 天。 已完成的工作會無限期保留；無法存取未在最長存留期內完成之工作的資料。

除了您定義在節點上執行計算的工作以外，Batch 服務還提供下列特殊工作：

* [啟動工作](#start-task)
* [作業管理員工作](#job-manager-task)
* [作業準備和作業釋放工作](#job-preparation-and-release-tasks)
* [多重執行個體工作 (MPI)](#multi-instance-tasks)
* [作業相依性](#task-dependencies)

### <a name="start-task"></a>啟動工作
藉由建立 **啟動工作** 與集區的關聯，您可以準備其節點的作業環境。 例如，您可以執行下列動作：安裝您的工作執行的應用程式或啟動背景程序。 啟動工作會在節點每次啟動時執行，且只要留在集區中就會持續執行，包括在節點第一次新增至集區時，以及重新啟動或重新安裝映像時。

啟動工作的主要優點，是其中可包含設定計算節點和安裝工作執行所需的應用程式時必須用到的所有資訊。 因此，增加集區中的節點數目如同指定新目標節點計數一樣簡單。 啟動工作會為 Batch 服務提供設定新節點並讓它們準備好接受工作所需的資訊。

如同任何 Azure Batch 工作，除了要執行的**命令列**以外，您還可以指定 [Azure 儲存體][azure_storage]中的**資源檔**清單。 Batch 服務會先從 Azure 儲存體將資源檔案複製到節點，然後再執行命令列。 對於集區啟動工作，檔案清單通常包含工作應用程式與其相依項目。

但啟動工作也可以包含在計算節點上執行的所有工作將會使用的參考資料。 例如，啟動工作的命令列可執行 `robocopy` 作業，將應用程式檔案 (已指定為資源檔案並下載至節點) 從啟動工作的[工作目錄](#files-and-directories)複製到[共用資料夾](#files-and-directories)，然後執行 MSI 或 `setup.exe`。

通常 Batch 服務最好能夠等待開始工作完成，然後再考慮將工作指派給節點，但您可以設定此行為。

如果計算節點上的啟動工作失敗，則會更新節點的狀態以反映失敗，且不會指派任何工作給該節點。 如果從儲存體複製啟動工作的資源檔案時發生問題，或由其命令列執行的程序傳回非零的結束程式碼，啟動工作可能會失敗。

如果您新增或更新現有集區的啟動工作，您必須重新啟動其計算節點，啟動工作才會套用至節點。

>[!NOTE]
> Batch 會限制啟動工作的總大小，包括資源檔案和環境變數。 如果您需要減少啟動工作的大小，您可以使用下列其中一個方法：
>
> 1. 您可以使用應用程式套件，將應用程式或資料分散於 Batch 集區中的每個節點。 如需應用程式套件的詳細資訊，請參閱[使用 Batch 應用程式套件將應用程式部署至計算節點](batch-application-packages.md)。
> 2. 您可以手動建立已壓縮的封存檔，其中包含您的應用程式檔案。 將已壓縮的封存檔以 blob 形式上傳至 Azure 儲存體。 指定已壓縮的封存檔作為您啟動工作的資源檔。 執行啟動工作的命令列之前，請從命令列將封存檔解壓縮。 
>
>    若要將封存檔解壓縮，您可以使用您選擇的封存工具。 您必須包含用來將封存檔解壓縮為啟動工作之資源檔的工具。
>
>

### <a name="job-manager-task"></a>作業管理員工作
您通常會使用 **作業管理員工作** 來控制和/或監視作業執行，例如，建立和提交作業的工作、決定其他要執行的工作，以及判斷工作何時完成。 但作業管理員的工作並不限定於這些活動。 它是功能完備的工作，可執行作業所需的任何動作。 比方說，作業管理員工作可以下載指定為參數的檔案、分析該檔案的內容，並根據這些內容提交其他工作。

作業管理員工作會在所有其他工作之前啟動。 它可提供下列功能：

* 建立作業時由 Batch 服務自動提交為工作。
* 排程在作業中的其他工作之前執行。
* 縮小集區時，相關聯的節點最後才會從集區中移除。
* 此終止可能完全取決於作業中的所有工作終止。
* 需要重新啟動時，作業管理員工作有最高的優先順序。 如果找不到閒置的節點，Batch 服務可能會終止集區中正在執行的其中另一個工作，以便挪出空間供作業管理員工作執行。
* 一個作業中的作業管理員工作的優先順序不會高於其他作業的工作。 不同作業之間，只注重作業層級優先順序。

### <a name="job-preparation-and-release-tasks"></a>作業準備和作業釋放工作
Batch 會提供作業前執行設定的作業準備工作。 作業釋放工作則用於作業後維護或清理。

* **作業準備工作**：在執行任何其他作業工作之前，會在排定要執行工作的所有計算節點上執行作業準備工作。 舉例來說，您可以使用作業準備工作可以複製所有工作所共用、但對作業是唯一的資料。
* **作業釋放工作**：作業完成後，系統會對集區中至少執行了一個工作的每個節點上執行作業釋放工作。 舉例來說，您可以使用作業釋放工作來刪除作業準備工作所複製的資料，或壓縮並上傳診斷記錄資料。

作業準備和釋放工作可讓您指定要在叫用工作時執行的命令列。 它們會提供許多功能，例如檔案下載、提升權限的執行、自訂環境變數、最大執行持續時間、重試計數、檔案保留時間。

如需關於作業準備和釋放工作的詳細資訊，請參閱 [在 Azure Batch 計算節點上執行準備和完成的工作](batch-job-prep-release.md)。

### <a name="multi-instance-task"></a>多重執行個體工作
[多重執行個體工作](batch-mpi.md) 是設定為同時在多個計算節點上執行的工作。 利用多重執行個體工作，您可以啟用高效能計算案例，例如需要一組配置在一起以處理單一工作負載 (像是訊息傳遞介面 (MPI)) 的計算節點。

如需在 Batch 中使用 Batch .NET 程式庫執行 MPI 作業的詳細討論，請參閱 [在 Azure Batch 中使用多個執行個體的工作執行訊息傳遞介面 (MPI) 應用程式](batch-mpi.md)。

### <a name="task-dependencies"></a>作業相依性
[工作相依性](batch-task-dependencies.md)正如其名，可讓您在執行某個工作之前，指定該工作相依於其他工作。 此功能提供下列情況的支援：「下游」工作取用「上游」工作的輸出，或當上游工作執行下游工作所需的某種初始化時。 若要使用此功能，您必須先在 Batch 作業上啟用工作相依性。 然後，針對每個相依於另一個工作 (或其他許多工作) 的工作，指定該工作相依的工作。

利用工作相依性，您可以設定如下所示的案例︰

* taskB 相依於 taskA (直到 taskA 完成，才會開始執行 taskB)。
* taskC 同時相依於 taskA 和 taskB。
* taskD 在執行前相依於某個範圍的工作，例如工作 1 至 10。

如需此功能的其他詳細資訊，請查看 [Azure Batch 中的工作相依性](batch-task-dependencies.md)和 [azure-batch-samples][github_samples] GitHub 存放庫中的 [TaskDependencies][github_sample_taskdeps] 程式碼範例。

## <a name="environment-settings-for-tasks"></a>工作的環境設定
Batch 服務所執行的每個工作都可以存取在計算節點上設定的環境變數。 這包括 Batch 服務所定義 ([服務定義的][msdn_env_vars]) 的環境變數，以及您可以為工作定義的自訂環境變數。 工作所執行的應用程式和指令碼，可以在執行期間存取這些環境變數。

您可以填入這些實體的「環境設定」  屬性，以在工作或作業層級設定自訂環境變數。 例如，請參閱[將工作新增至作業][rest_add_task]作業 (Batch REST API) 或 Batch .NET 中的 [CloudTask.EnvironmentSettings][net_cloudtask_env] 和 [CloudJob.CommonEnvironmentSettings][net_job_env] 屬性。

您的用戶端應用程式或服務可藉由使用[取得工作的相關資訊][rest_get_task_info]作業 (Batch REST) 或存取 [CloudTask.EnvironmentSettings][net_cloudtask_env] 屬性 (Batch .NET)，取得工作的環境變數 (服務定義和自訂)。 在計算節點上執行的程序可以在節點上存取這些和其他環境變數，例如，藉由使用熟悉的 `%VARIABLE_NAME%` (Windows) 或 `$VARIABLE_NAME` (Linux) 語法。

您可以在[計算節點環境變數][msdn_env_vars]中找到所有服務定義的環境變數完整清單。

## <a name="files-and-directories"></a>檔案和目錄
每個工作會在其「工作目錄」  下建立零個或多個檔案和目錄。 此工作目錄可用來儲存工作所執行的程式、工作所處理的資料，以及工作所執行之處理的輸出。 工作使用者擁有工作的所有檔案和目錄。

Batch 服務會在節點上公開檔案系統的一部分作為「根目錄」 。 工作可藉由參考 `AZ_BATCH_NODE_ROOT_DIR` 環境變數來存取根目錄。 如需有關如何使用環境變數的詳細資訊，請參閱 [工作的環境設定](#environment-settings-for-tasks)。

根目錄包含下列目錄結構：

![計算節點目錄結構][1]

* **共用**：此目錄允許對「所有」在節點上執行的工作進行讀取/寫入存取。 任何在節點上執行的工作都可以建立、讀取、更新和刪除此目錄中的檔案。 工作可藉由參考 `AZ_BATCH_NODE_SHARED_DIR` 環境變數來存取這個目錄。
* **啟動**：啟動工作使用這個目錄做為它的工作目錄。 由啟動工作下載到的節點所有檔案都會儲存在這裡。 啟動工作可以建立、讀取、更新和刪除此目錄下的檔案。 工作可藉由參考 `AZ_BATCH_NODE_STARTUP_DIR` 環境變數來存取這個目錄。
* **工作**：系統會為每個在節點上執行的工作建立一個目錄。 它可透過參考 `AZ_BATCH_TASK_DIR` 環境變數來存取。

    在每個工作目錄中，Batch 服務會建立由 `AZ_BATCH_TASK_WORKING_DIR` 環境變數指定唯一路徑的工作目錄 (`wd`)。 這個目錄可供讀取/寫入工作。 工作可以建立、讀取、更新和刪除此目錄下的檔案。 此目錄會根據工作指定的「RetentionTime」  條件約束而保留。

    `stdout.txt` 和 `stderr.txt`：這些檔案會在工作執行期間寫入至工作資料夾。

> [!IMPORTANT]
> 當節點從集區移除時，也會移除「所有」儲存在節點上的檔案。
>
>

## <a name="application-packages"></a>應用程式套件
[應用程式套件](batch-application-packages.md) 功能可為集區中的計算節點提供簡單的應用程式管理和部署能力。 您可以輕鬆上傳及管理您的工作所執行的多個應用程式版本，包括其二進位檔和支援檔案。 接著，您可以將一或多個這種類型的應用程式自動部署到集區中的計算節點。

您可以在集區和工作層級指定應用程式套件。 套件當您指定集區應用程式套件時，應用程式會部署到集區中的每個節點。 當您指定工作應用程式套件時，應用程式只會在執行工作的命令列之前，部署至排定要執行作業的至少一個工作的節點。

Batch 可處理使用 Azure 儲存體將應用程式套件儲存及部署到計算節點的詳細資料，因此可以簡化程式碼和管理額外負荷。

若要深入了解應用程式套件功能，請參閱[使用 Batch 應用程式套件將應用程式部署至計算節點](batch-application-packages.md)。

> [!NOTE]
> 如果您將集區應用程式套件新增至「現有」集區，您必須重新啟動其計算節點，應用程式套件才會套用至節點。
>
>

## <a name="pool-and-compute-node-lifetime"></a>集區和計算節點存留期
在設計 Azure Batch 解決方案時，必須制定關於如何及何時建立集區，以及這些集區中的計算節點可用性要保持多久的設計決策。

在極端情況下，您可以針對您所提交的每項作業建立集區，並在其工作執行完成時立即移除此集區。 這樣只有在必要時才會配置節點，而且節點會在變成閒置時立即關閉，因此可達到最高使用率。 雖然這表示作業必須等候節點進行配置，但請務必注意，工作會在節點個別可用、配置且啟動工作完成時立即排程執行。 Batch「不會」等到集區中的所有節點都可用，才將工作指派到節點。 這可確保所有可用節點的最大使用率。

就另一個極端而言，如果讓作業立即啟動是最高的優先順序，則可以預先建立集區，並使其節點在作業提交之前成為可用。 在此情況下，工作可以立即啟動，但節點可能會閒置以等候指派。

有一個通常用來處理可變但持續負載的組合方法。 您可以設定有多個作業提交至該處，但根據作業負載向上或向下調整節點數目的集區 (請參閱下一節 [調整計算資源](#scaling-compute-resources) )。 您可以根據目前的負載被動完成，或在負載可預測時主動完成。

## <a name="virtual-network-vnet-and-firewall-configuration"></a>虛擬網路 (VNet) 和防火牆設定 

當您在 Batch 中佈建計算節點集區時，可以將此集區與 Azure [虛擬網路 (VNet)](../virtual-network/virtual-networks-overview.md) 的子網路產生關聯。 若要使用 Azure VNet，Batch 用戶端 API 必須使用 Azure Active Directory (AD) 驗證。 Azure Batch 對於 Azure AD 的支援記載於[使用 Active Directory 驗證 Batch 服務解決方案](batch-aad-auth.md)中。  

### <a name="vnet-requirements"></a>VNet 需求
[!INCLUDE [batch-virtual-network-ports](../../includes/batch-virtual-network-ports.md)]

如需在 VNet 中設定 Batch 集區的詳細資訊，請參閱[使用虛擬網路建立虛擬機器的集區](batch-virtual-network.md)。

## <a name="scaling-compute-resources"></a>調整計算資源
透過 [自動調整](batch-automatic-scaling.md)功能，您可以讓 Batch 服務根據計算案例的目前工作負載和資源使用狀況，動態調整集區中的計算節點數目。 這樣一來，您只會使用所需資源並可釋放不需要的資源，因而能夠降低應用程式的整體執行成本。

撰寫 [自動調整公式](batch-automatic-scaling.md#automatic-scaling-formulas) 並將該公式與集區建立關聯，以啟用自動調整。 Batch 服務使用此公式來決定集區中下一個調整間隔 (您可以設定的間隔) 的目標節點數目。 您可以在建立集區時指定集區的自動調整設定，或稍後在集區上啟用調整。 您也可以更新已啟用調整的集區上的調整設定。

例如，或許作業需要您提交非常大量要執行的工作。 您可以指派調整公式給集區，以根據目前已排入佇列的工作數目和作業中工作的完成率來調整集區中的節點數目。 Batch 服務會定期評估公式，並根據工作負載和其他公式設定來調整集區大小。 服務會在有大量排入佇列的工作時視需要新增節點，並在沒有排入佇列或正在執行時移除節點。

調整公式可以根據下列度量：

* **時間度量** 是以指定時數內每隔五分鐘收集的統計資料為基礎。
* **資源度量** 是以 CPU 使用量、頻寬使用量、記憶體使用量和節點的數目為基礎。
* **工作計量**是以工作狀態為基礎，例如 [作用中] \(已排入佇列)、[執行中] 或 [已完成]。

當自動調整減少集區中的計算節點數目時，您必須考量如何處理在減少作業時正在執行的工作。 為了配合這一點，Batch 會提供您可以納入公式中的「節點解除配置選項」  。 例如，您可以指定執行中的工作立即停止，然後重新排入佇列以便在另一個節點上執行，或允許先完成再從集區中移除節點。

如需關於自動調整應用程式的詳細資訊，請參閱 [自動調整 Azure Batch 集區中的計算節點](batch-automatic-scaling.md)。

> [!TIP]
> 若要獲得最大的計算資源使用率，請將節點的目標數目設定成在作業結束時降為零，但允許執行中的工作完成。
>
>

## <a name="security-with-certificates"></a>憑證的安全性
在加密或解密工作的敏感資訊 (例如 [Azure 儲存體帳戶的金鑰][azure_storage]) 時，您通常需要使用憑證。 若要支援此功能，您可以在節點上安裝憑證。 加密的機密資料會透過命令列參數或內嵌在其中一個工作資源中而傳遞至工作，已安裝的憑證可用來解密這些資料。

您可以使用[新增憑證][rest_add_cert]作業 (Batch REST) 或 [CertificateOperations.CreateCertificate][net_create_cert] 方法 (Batch .NET)，將憑證新增至 Batch 帳戶。 然後，您可以將憑證與新的或現有的集區產生關聯。 當憑證與集區相關聯時，Batch 服務會在集區中的每個節點上安裝憑證。 當節點啟動時，在啟動任何工作之前 (包括開始工作和作業管理員工作)，Batch 服務會安裝適當的憑證。

如果您將憑證加入至「現有」  集區，您必須重新啟動其計算節點，憑證才會套用至節點。

## <a name="error-handling"></a>錯誤處理
有時候您可能需要處理 Batch 解決方案中的工作和應用程式失敗。

### <a name="task-failure-handling"></a>工作失敗處理
工作失敗可分成下列幾類：

* **前置處理失敗**

    如果工作無法啟動，系統便會對該工作設定前置處理錯誤。  

    前置處理錯誤的發生原因可能是工作的資源檔案已移動、儲存體帳戶已無法使用，或發生其他使檔案無法成功複製到節點的問題。

* **檔案上傳失敗**

    如果在上傳為工作指定的檔案時因故失敗，系統會對該工作設定檔案上傳錯誤。

    檔案上傳錯誤的發生原因可能是所提供來存取 Azure 儲存體的 SAS 無效或未提供寫入權限、儲存體帳戶已無法再使用，或發生其他讓系統無法成功從節點複製檔案的問題。    

* **應用程式失敗**

    工作的命令列所指定的程序也可能會失敗。 工作所執行的程序傳回非零的結束碼時，此程序會被視為失敗 (請參閱下一節的＜工作結束代碼＞  )。

    針對應用程式失敗，您可以將 Batch 設定成自動重試工作直到指定的次數為止。

* **條件約束失敗**

    您可以設定條件約束來指定作業或工作的最大執行持續期間「maxWallClockTime」 。 這很適合用於終止無法進行的工作。

    超過時間量上限時，則會將工作標示為「已完成」，但結束代碼會設為 `0xC000013A`，[schedulingError] 欄位會標示為 `{ category:"ServerError", code="TaskEnded"}`。

### <a name="debugging-application-failures"></a>應用程式失敗偵錯
* `stderr`和`stdout`

    在執行期間，應用程式可能會產生診斷輸出，以便用來排解疑難問題。 如前面的[檔案和目錄](#files-and-directories)一節所述，Batch 服務會將標準輸出和標準錯誤輸出寫入至計算節點上工作目錄中的 `stdout.txt` 和 `stderr.txt` 檔案。 您可以使用 Azure 入口網站或其中一個 Batch SDK 來下載這些檔案。 例如，您可以使用 Batch .NET 程式庫中的 [ComputeNode.GetNodeFile][net_getfile_node] 和 [CloudTask.GetNodeFile][net_getfile_task]，擷取這些和其他檔案來進行疑難排解。

* **工作結束代碼**

    如前文所述，如果工作所執行的程序傳回非零的結束代碼，則 Batch 服務會將此工作標示為失敗。 當工作執行一個程序時，Batch 會使用「程序的傳回代碼」 填入工作的結束代碼屬性。 請務必注意，工作的結束代碼**不會**取決於 Batch 服務。 工作的結束代碼取決於程序本身，或程序執行所在的作業系統。

### <a name="accounting-for-task-failures-or-interruptions"></a>處理工作失敗或中斷
工作可能偶爾會失敗或中斷。 工作應用程式本身可能失敗、執行工作的節點可能重新開機，或節點在調整大小作業期間可能因為集區的取消配置原則設為不需等待工作完成便立即移除節點而從集區中移除。 在所有的情況下，此工作均可由 Batch 自動重新排入佇列，並且在另一個節點上執行。

也可能是間歇性問題導致工作懸置或花太長時間執行。 您可以設定工作的執行間隔上限。 如果超過最大執行間隔，Batch 服務會中斷工作應用程式。

### <a name="connecting-to-compute-nodes"></a>連接到計算節點
您可以從遠端登入計算節點，以執行額外的偵錯和疑難排解。 您可以使用 Azure 入口網站下載 Windows 節點的遠端桌面通訊協定 (RDP) 檔案，並取得 Linux 節點的安全殼層 (SSH) 連線資訊。 您也可以使用 Batch API (例如透過 [Batch .NET][net_rdpfile] 或 [Batch Python](batch-linux-nodes.md#connect-to-linux-nodes-using-ssh)) 進行此項作業。

> [!IMPORTANT]
> 若要透過 RDP 或 SSH 連接到節點，您必須先在節點上建立使用者。 若要這樣做，您可以使用 Azure 入口網站，透過 Batch REST API [將使用者帳戶新增至節點][rest_create_user]、在 Batch .NET 中呼叫 [ComputeNode.CreateComputeNodeUser][net_create_user] 方法，或在 Batch Python 模組中呼叫 [add_user][py_add_user] 方法。
>
>

如果您需要限制或停用對計算節點的 RDP 或 SSH 存取，請參閱[設定或停用對 Azure Batch 集區中的計算節點的遠端存取](pool-endpoint-configuration.md)。

### <a name="troubleshooting-problematic-compute-nodes"></a>疑難排解有問題的計算節點
在部分工作失敗的情況下，Batch 用戶端應用程式或服務可以檢查失敗工作的中繼資料來找出行為異常的節點。 集區中的每個節點都有唯一的 ID，而執行工作的節點會包含在工作中繼資料中。 在找出問題節點之後，您即可對其採取數個行動︰

* **重新啟動節點** ([REST][rest_reboot] | [.NET][net_reboot])

    重新啟動節點有時可以清除潛在的問題，例如停滯或損毀的程序。 請注意，如果集區使用啟動工作或作業使用作業準備工作，則會在節點重新啟動時執行這些工作。
* **重新安裝節點映像** ([REST][rest_reimage] | [.NET][net_reimage])

    這會在節點上重新安裝作業系統。 和重新啟動節點一樣，在重新安裝映像節點後，便會重新執行啟動工作和作業準備工作。
* **從集區中移除節點** ([REST][rest_remove] | [.NET][net_remove])

    有時候您必須從集區中完整移除節點。
* **停用節點上的工作排程** ([REST][rest_offline] | [.NET][net_offline])

    這實際上會讓節點離線，以便不再指派任何工作給它，但允許該節點繼續執行並留在集區中。 這可讓您執行進一步的調查以了解失敗原因，卻又不會遺失失敗工作的資料，而且不會讓節點造成額外的工作失敗。 例如，您可以停用節點上的工作排程，然後 [從遠端登入](#connecting-to-compute-nodes) 以檢查節點的事件記錄檔，或執行其他疑難排解動作。 在完成調查之後，您就可以啟用工作排程 ([REST][rest_online] | [.NET][net_online]) 讓節點重新上線，或是執行稍早所討論的另一個動作。

> [!IMPORTANT]
> 您可以使用本節說明的各個動作 (重新啟動、重新安裝映像、移除和停用工作排程)，指定當您執行動作時要如何處理節點上目前執行的工作。 例如，當您使用 Batch .NET 用戶端程式庫停用節點上的工作排程時，可以指定 [DisableComputeNodeSchedulingOption][net_offline_option] 列舉值，以指定是要**終止**執行中的工作、將工作**重新放入佇列**以在其他節點上排程，還是允許執行中的工作先完成再執行動作 (**TaskCompletion**)。
>
>

## <a name="next-steps"></a>後續步驟
* 了解可用來建置 Batch 解決方案的 [Batch API 和工具](batch-apis-tools.md)。
* 了解使用 [Batch .NET 用戶端程式庫](quick-run-dotnet.md)或 [Python](quick-run-python.md) 開發啟用 Batch 之應用程式的基本概念。 這些快速入門會介紹使用 Batch 服務在多個計算節點上執行工作負載的範例應用程式，並說明如何使用 Azure 儲存體進行工作負載檔案的預備和擷取。
* 下載並安裝 [Batch Explorer][batch_labs]，以在開發 Batch 解決方案時使用。 使用 Batch Explorer 有助於建立、偵錯和監視 Azure Batch 應用程式。 
* 請參閱社群資源，包括[Stack Overflow](http://stackoverflow.com/questions/tagged/azure-batch)、[Batch 社群存放庫](https://github.com/Azure/Batch)以及 MSDN 上的 [Azure Batch 論壇][batch_forum]。 

[1]: ./media/batch-api-basics/node-folder-structure.png

[azure_storage]: https://azure.microsoft.com/services/storage/
[batch_forum]: https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurebatch
[cloud_service_sizes]: ../cloud-services/cloud-services-sizes-specs.md
[msmpi]: https://msdn.microsoft.com/library/bb524831.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_sample_taskdeps]:  https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/TaskDependencies
[batch_labs]: https://azure.github.io/BatchExplorer/
[batch_net_api]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[msdn_env_vars]: https://msdn.microsoft.com/library/azure/mt743623.aspx
[net_cloudjob_jobmanagertask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobmanagertask.aspx
[net_cloudjob_priority]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.priority.aspx
[net_cloudpool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx
[net_cloudtask_env]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.environmentsettings.aspx
[net_create_cert]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.certificateoperations.createcertificate.aspx
[net_create_user]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.createcomputenodeuser.aspx
[net_getfile_node]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.getnodefile.aspx
[net_getfile_task]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.getnodefile.aspx
[net_job_env]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.commonenvironmentsettings.aspx
[net_multiinstancesettings]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.aspx
[net_onalltaskscomplete]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.onalltaskscomplete.aspx
[net_rdp]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.getrdpfile.aspx
[net_reboot]: https://msdn.microsoft.com/library/azure/mt631495.aspx
[net_reimage]: https://msdn.microsoft.com/library/azure/mt631496.aspx
[net_remove]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.removefrompoolasync.aspx
[net_offline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.disableschedulingasync.aspx
[net_online]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.enableschedulingasync.aspx
[net_offline_option]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.common.disablecomputenodeschedulingoption.aspx
[net_rdpfile]: https://msdn.microsoft.com/library/azure/Mt272127.aspx
[vnet]: https://msdn.microsoft.com/library/azure/dn820174.aspx#bk_netconf

[py_add_user]: https://docs.microsoft.com/python/azure/?view=azure-python

[batch_rest_api]: https://msdn.microsoft.com/library/azure/Dn820158.aspx
[rest_add_job]: https://msdn.microsoft.com/library/azure/mt282178.aspx
[rest_add_pool]: https://msdn.microsoft.com/library/azure/dn820174.aspx
[rest_add_cert]: https://msdn.microsoft.com/library/azure/dn820169.aspx
[rest_add_task]: https://msdn.microsoft.com/library/azure/dn820105.aspx
[rest_create_user]: https://msdn.microsoft.com/library/azure/dn820137.aspx
[rest_get_task_info]: https://msdn.microsoft.com/library/azure/dn820133.aspx
[rest_job_schedules]: https://msdn.microsoft.com/library/azure/mt282179.aspx
[rest_multiinstance]: https://msdn.microsoft.com/library/azure/mt637905.aspx
[rest_multiinstancesettings]: https://msdn.microsoft.com/library/azure/dn820105.aspx#multiInstanceSettings
[rest_update_job]: https://msdn.microsoft.com/library/azure/dn820162.aspx
[rest_rdp]: https://msdn.microsoft.com/library/azure/dn820120.aspx
[rest_reboot]: https://msdn.microsoft.com/library/azure/dn820171.aspx
[rest_reimage]: https://msdn.microsoft.com/library/azure/dn820157.aspx
[rest_remove]: https://msdn.microsoft.com/library/azure/dn820194.aspx
[rest_offline]: https://msdn.microsoft.com/library/azure/mt637904.aspx
[rest_online]: https://msdn.microsoft.com/library/azure/mt637907.aspx

[vm_marketplace]: https://azure.microsoft.com/marketplace/virtual-machines/
