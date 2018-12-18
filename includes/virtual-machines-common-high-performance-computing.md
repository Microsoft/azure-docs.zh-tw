---
title: 包含檔案
description: 包含檔案
services: virtual-machines-linux, virtual-machines-windows
author: dlepow
ms.service: multiple
ms.topic: include
ms.date: 07/02/2018
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: 77da95ce0d37e8d0b35125f0cc8dc33376a52259
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47401527"
---
組織有大規模的運算需求。 這些 Big Compute 工作負載包括工程設計和分析、財務風險計算、影像轉譯、複雜模型、Monte Carlo 模擬等等。 

使用 Azure 雲端能有效率地執行需要大量運算的 Linux 和 Windows 工作負載，從平行批次作業到傳統 HPC 模擬。 透過您所選擇的運算服務、格線管理員、Marketplace 解決方案和廠商裝載 (SaaS) 應用程式，在 Azure 基礎結構上執行 HPC 和批次工作負載。 Azure 提供彈性的解決方案可分配工作並擴充至數千個的 VM 或核心，然後在您不需要這麼多資源時相應減少。 



## <a name="solution-options"></a>解決方案選項



* **自己動手做 (DIY) 解決方案**
    * 在 Azure 虛擬機器或[虛擬機器擴展集](../articles/virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)中設定自己的叢集環境。 
    * 原形移轉內部部署叢集，或在 Azure 中部署新的叢集以取得額外容量。 
    * 使用 Azure Resource Manager 範本來部署前置[工作負載管理員](#workload-managers)基礎結構和[應用程式](#hpc-applications)。 
    * 選擇 [HPC 和 GPU VM 大小](#hpc-and-gpu-sizes)，包括 MPI 或 GPU 工作負載的特製化硬體和網路連線。 
    * 為 I/O 密集的工作負載新增[高效能儲存體](#hpc-storage)。
* **混合式方案**
    * 擴充內部部署解決方案，以便將尖峰 (「激增」) 工作負載卸載至 Azure 基礎結構
    * 使用隨選雲端運算搭配現有的[工作負載管理員](#workload-manager)。
    * 利用 MPI 或 GPU 工作負載的 [HPC 和 GPU VM 大小](#hpc-and-gpu-sizes)。
* **Big Compute 解決方案即服務**
    * 使用 [Azure Batch](#azure-batch) 和相關 [Azure 服務](#related-azure-services)來開發自訂 Big Compute 解決方案和工作流程。
    * 執行來自 [Altair](http://www.altair.com/)、[Rescale](https://www.rescale.com/azure/) 和[Cycle Computing](https://cyclecomputing.com/) (現在[與 Microsoft 聯手](https://blogs.microsoft.com/blog/2017/08/15/microsoft-acquires-cycle-computing-accelerate-big-computing-cloud/)) 等廠商之已啟用 Azure 的工程和模擬解決方案。
    * 使用 [Cray 超級電腦](https://www.cray.com/solutions/supercomputing-as-a-service/cray-in-azure)作為 Azure 中裝載的服務。
* **Marketplace 解決方案**
    * 使用 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/) 中提供的 [HPC 應用程式](#hpc-applications)和[解決方案](#marketplace-solutions)。 
    


下列各節提供關於支援技術和指引連結的詳細資訊。



## <a name="marketplace-solutions"></a>Marketplace 解決方案

請瀏覽 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/) 以取得針對 HPC 設計的 Linux 和 Windows VM 映像和解決方案。 範例包括：

* [RogueWave CentOS 型 HPC](https://azuremarketplace.microsoft.com/marketplace/apps/RogueWave.CentOSbased73HPC?tab=Overview)
* [適用於 HPC 的 SUSE Linux Enterprise Server](https://azure.microsoft.com/marketplace/partners/suse/suselinuxenterpriseserver12optimizedforhighperformancecompute/)
*  [TIBCO Grid Server Engine](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/tibco-software.gridserverlinuxengine?tab=Overview)
* [適用於 Windows 和 Linux 的 Azure 資料科學 VM](../articles/machine-learning/machine-learning-data-science-virtual-machine-overview.md)
* [D3View](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/xfinityinc.d3view-v5?tab=Overview)
* [UberCloud](https://azure.microsoft.com/search/marketplace/?q=ubercloud)
* [Intel Cloud Edition for Lustre](https://azuremarketplace.microsoft.com/marketplace/apps/intel.intel-cloud-edition-gs)


 
## <a name="hpc-applications"></a>HPC 應用程式

在 Azure 中執行自訂或商業 HPC 應用程式。 本節中的數個範例會經過基準測試，以使用其他 VM 或運算核心有效地進行擴充。 請瀏覽 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace) 以取得可立即部署的解決方案。

> [!NOTE]
> 請向廠商確認任何商業應用程式在雲端中的執行授權或其他限制。 並非所有廠商都提供隨用隨付授權。 您可能需要視您的解決方案，在雲端中授權伺服器，或連線至內部部署授權伺服器。

### <a name="engineering-applications"></a>工程應用程式


* [Altair RADIOSS](https://azure.microsoft.com/blog/availability-of-altair-radioss-rdma-on-microsoft-azure/)
* [ANSYS CFD](https://azure.microsoft.com/blog/ansys-cfd-and-microsoft-azure-perform-the-best-hpc-scalability-in-the-cloud/)
* [MATLAB Distributed Computing Server](../articles/virtual-machines/windows/matlab-mdcs-cluster.md)
* [StarCCM+](https://blogs.msdn.microsoft.com/azurecat/2017/07/07/run-star-ccm-in-an-azure-hpc-cluster/)
* [OpenFOAM](https://simulation.azure.com/casestudies/Team-182-ABB-UC-Final.pdf)



### <a name="graphics-and-rendering"></a>圖形和轉譯器

* Azure Batch 上的 [Autodesk Maya、3ds Max 和 Arnold](../articles/batch/batch-rendering-service.md) 

### <a name="ai-and-deep-learning"></a>AI 和深入學習

* 深入學習模型的 [Batch AI](../articles/batch-ai/overview.md) 訓練
* [Microsoft 辨識工具組](https://docs.microsoft.com/cognitive-toolkit/cntk-on-azure)
* [深入學習 VM](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning)
* [深入學習的 Batch Shipyard 訣竅](https://github.com/Azure/batch-shipyard/tree/master/recipes#deeplearning)






## <a name="hpc-and-gpu-vm-sizes"></a>HPC 和 GPU VM 大小
Azure 提供一系列適用於 [Linux](../articles/virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 和 [Windows](../articles/virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) VM 的大小，包括專為需要大量運算的工作負載所設計的大小。 例如，H16r 和 H16mr VM 可以連線到高輸送量後端 RDMA 網路。 此雲端網路可以改善 [Microsoft MPI](https://msdn.microsoft.com/library/bb524831.aspx) 或 Intel MPI 下執行的緊密結合平行應用程式效能。 

N 系列 VM 功能 NVIDIA GPU 是專為需要大量運算或需要大量圖形的應用程式所設計，包括人工地智慧 (AI) 學習和視覺效果。 

深入了解：

* [Linux](../articles/virtual-machines/linux/sizes-hpc.md) 和 [Windows](../articles/virtual-machines/windows/sizes-hpc.md) VM 的高效能運算大小 
* [Linux](../articles/virtual-machines/linux/sizes-gpu.md) 和 [Windows](../articles/virtual-machines/windows/sizes-gpu.md) VM 的已啟用 GPU 功能之大小 

了解如何：

* [設定 Linux RDMA 叢集以執行 MPI 應用程式](../articles/virtual-machines/linux/classic/rdma-cluster.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)
* [使用 Microsoft HPC Pack 設定 Windows RDMA 叢集以執行 MPI 應用程式](../articles/virtual-machines/windows/classic/hpcpack-rdma-cluster.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)
* [在 Batch 集區中使用需要大量運算的 VM](../articles/batch/batch-pool-compute-intensive-sizes.md)



## <a name="azure-batch"></a>Azure Batch
[Batch](../articles/batch/batch-technical-overview.md) 是一項平台服務，可用於在雲端有效地執行大規模的平行和高效能運算 (HPC) 應用程式。 Azure Batch 可排程要在受控集區虛擬機器上執行的需要大量運算工作，而且可以調整運算資源以符合工作的需求。 

SaaS 提供者或開發人員可使用 Batch SDK 和工具，將 HPC 應用程式或容器工作負載與 Azure 進行整合、將資料暫存至 Azure，並建置作業執行管線。 

了解如何：

* [開始使用 Batch 進行開發](../articles/batch/quick-run-dotnet.md)
* [使用 Azure Batch 程式碼範例](https://github.com/Azure/azure-batch-samples)
* [使用低優先順序的 VM 搭配 Batch](../articles/batch/batch-low-pri-vms.md)
* [使用 Batch Shipyard 執行容器化的 HPC 工作負載](https://github.com/Azure/batch-shipyard)
* [在 Batch 上執行平行的 R 工作負載](https://github.com/Azure/doAzureParallel)
* [在 Batch 上執行隨選 Spark 作業](https://github.com/Azure/aztk)

## <a name="workload-managers"></a>工作負載管理員

下列是可以在 Azure 基礎結構中執行的叢集和工作負載管理員範例。 在 Azure VM 中建立獨立叢集，或從內部部署叢集高載至 Azure VM。 
* [Alces Flight Compute](https://azuremarketplace.microsoft.com/marketplace/apps/alces-flight-limited.alces-flight-compute-solo?tab=Overview)
* [TIBCO DataSynapse GridServer](https://azure.microsoft.com/blog/tibco-datasynapse-comes-to-the-azure-marketplace/) 
* [Bright Cluster Manager](http://www.brightcomputing.com/technology-partners/microsoft)
* [IBM Spectrum Symphony 和 Symphony LSF](https://azure.microsoft.com/blog/ibm-and-microsoft-azure-support-spectrum-symphony-and-spectrum-lsf/)
* [PBS Pro](http://pbspro.org)
* [Microsoft HPC Pack](https://technet.microsoft.com/library/mt744885.aspx) - 請參閱在 [Windows](../articles/virtual-machines/windows/hpcpack-cluster-options.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 和 [Linux](../articles/virtual-machines/linux/hpcpack-cluster-options.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) VM 中執行的選項 



## <a name="hpc-storage"></a>HPC 儲存體

大規模 Batch 和 HPC 工作負載所需要的資料儲存和存取會超過傳統雲端檔案系統的容量。 在 Azure 中實作平行的檔案系統方案，例如 [Lustre](http://lustre.org/) 和 [BeeGFS](http://www.beegfs.com/content/)。

深入了解：

* [Azure 上的平行虛擬檔案系統](https://azure.microsoft.com/resources/parallel-virtual-file-systems-on-microsoft-azure/)
* 來自 [Avere](http://www.averesystems.com/about-us/about-avere) (現在[已與 Microsoft 合併](https://blogs.microsoft.com/blog/2018/01/03/microsoft-to-acquire-avere-systems-accelerating-high-performance-computing-innovation-for-media-and-entertainment-industry-and-beyond/)) 的高效能雲端儲存體解決方案


## <a name="related-azure-services"></a>相關的 Azure 服務

Azure 虛擬機器、虛擬機器擴展集、Batch 和相關的運算服務是大部分 Azure HPC 解決方案的基礎。 不過，您的解決方案可以利用許多相關的 Azure 服務。 部份清單如下：

### <a name="storage"></a>儲存體

* [Blob、資料表和佇列儲存體](../articles/storage/storage-introduction.md)
* [檔案儲存體](../articles/storage/storage-files-introduction.md)

### <a name="data-and-analytics"></a>資料與分析
* [HDInsight](../articles/hdinsight/hadoop/apache-hadoop-introduction.md)
* [Data Factory](../articles/data-factory/introduction.md)
* [Data Lake Storage Gen1](../articles/data-lake-store/data-lake-store-overview.md)
* [Databricks](../articles/azure-databricks/what-is-azure-databricks.md)
* [SQL Database](../articles/sql-database/sql-database-technical-overview.md)

### <a name="ai-and-machine-learning"></a>AI 和機器學習
* [機器學習服務](../articles/machine-learning/service/overview-what-is-azure-ml.md)
* [Batch AI](../articles/batch-ai/overview.md)
* [Genomics](../articles/genomics/overview-what-is-genomics.md)

### <a name="networking"></a>網路功能
* [虛擬網路](../articles/virtual-network/virtual-networks-overview.md)
* [ExpressRoute](../articles/expressroute/expressroute-introduction.md)

### <a name="containers"></a>容器
* [容器服務](../articles/container-service/dcos-swarm/container-service-intro.md)
* [Azure Kubernetes Service (AKS)](../articles/aks/intro-kubernetes.md)
* [容器登錄](../articles/container-registry/container-registry-intro.md)



## <a name="customer-stories"></a>客戶案例

已使用 Azure HPC 解決方案來解決商務問題的客戶範例：

* [ANEO](https://customers.microsoft.com/story/it-provider-finds-highly-scalable-cloud-based-hpc-redu) 
* [AXA Global P&C](https://customers.microsoft.com/story/axa-global-p-and-c)
* [Axioma](https://customers.microsoft.com/story/axioma-delivers-fintechs-first-born-in-the-cloud-multi-asset-class-enterprise-risk-solution)
* [d3View](https://customers.microsoft.com/story/big-data-solution-provider-adopts-new-cloud-gains-thou)
* [EFS](https://customers.microsoft.com/story/efs-professionalservices-azure)
* [Hymans Robertson](https://customers.microsoft.com/story/hymans-robertson)
* [MetLife](https://enterprise.microsoft.com/en-us/customer-story/industries/insurance/metlife/)
* [Microsoft Research](https://customers.microsoft.com/doclink/fast-lmm-and-windows-azure-put-genetics-research-on-fa)
* [明德精算顧問有限公司](https://customers.microsoft.com/story/actuarial-firm-works-to-transform-insurance-industry-w)
* [Mitsubishi UFJ Securities International](https://customers.microsoft.com/story/powering-risk-compute-grids-in-the-cloud)
* [NeuroInitiative](https://customers.microsoft.com/en-us/story/neuroinitiative-health-provider-azure)
* [Schlumberger](http://azure.microsoft.com/blog/big-compute-for-large-engineering-simulations)
* [Towers Watson](https://customers.microsoft.com/story/insurance-tech-provider-delivers-disruptive-solutions)


## <a name="next-steps"></a>後續步驟
* 深入了解適用於[工程模擬](https://simulation.azure.com/)、[轉譯](https://azure.microsoft.com/solutions/big-compute/rendering/)、[銀行與資本市場](https://finance.azure.com/)和[基因體學](https://enterprise.microsoft.com/en-us/industries/health/genomics/)的 Big Compute 解決方案。
* 如需最新公告，請參閱 [Microsoft HPC 和 Batch 小組部落格](http://blogs.technet.com/b/windowshpc/)以及[Azure 部落格](https://azure.microsoft.com/blog/tag/hpc/)。

* 使用受控和可擴充的 Azure [Batch](https://azure.microsoft.com/services/batch/) 服務來執行需要大量運算的工作負載，而不需要管理基礎結構[進一步了解](https://azure.microsoft.com/solutions/architecture/hpc-big-compute-saas/)



