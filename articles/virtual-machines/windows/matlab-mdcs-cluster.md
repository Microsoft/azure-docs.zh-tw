---
title: 虛擬機器上的 MATLAB 叢集 | Microsoft Docs
description: 使用 Microsoft Azure 虛擬機器建立 MATLAB Distributed Computing Server 叢集，以執行需密集計算的平行 MATLAB 工作負載。
services: virtual-machines-windows
documentationcenter: ''
author: mscurrell
manager: jeconnoc
editor: ''
ms.assetid: e9980ce9-124a-41f1-b9ec-f444c8ea5c72
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: Windows
ms.workload: infrastructure-services
ms.date: 05/09/2016
ms.author: markscu
ms.openlocfilehash: 695833fb12c0c7a130e98fe9b3bdfa502672ab29
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30914913"
---
# <a name="create-matlab-distributed-computing-server-clusters-on-azure-vms"></a>在 Azure VM 上建立 MATLAB Distributed Computing Server 叢集
使用 Microsoft Azure 虛擬機器建立一或多個 MATLAB Distributed Computing Server 叢集，以執行需密集計算的平行 MATLAB 工作負載。 在 VM 上安裝 MATLAB Distributed Computing Server 軟體來作為基礎映像，並使用 Azure 快速入門範本或 Azure PowerShell 指令碼 (可在 [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/matlab-cluster)上取得) 來部署和管理叢集。 在部署之後，連接到叢集以執行工作負載。

## <a name="about-matlab-and-matlab-distributed-computing-server"></a>關於 MATLAB 和 MATLAB Distributed Computing Server
[MATLAB](http://www.mathworks.com/products/matlab/) 平台最適合解決工程及科學問題。 MATLAB 使用者如需要進行大規模的模擬和資料處理工作，可以使用 MathWorks 平行計算產品，利用計算叢集與計算格線服務來加快其密集計算工作負載的執行速度。 [Parallel Computing Toolbox](http://www.mathworks.com/products/parallel-computing/) 可讓 MATLAB 使用者平行處理應用程式，並利用多核心處理器、GPU 和計算叢集。 [MATLAB Distributed Computing Server](http://www.mathworks.com/products/distriben/) 則可讓 MATLAB 使用者利用計算叢集內的許多電腦。

藉由使用 Azure 虛擬機器，您可以建立 MATLAB Distributed Computing Server 叢集，讓其擁有所有可供提交平行工作的相同機制以作為內部部署叢集，例如互動式作業、批次作業、獨立的工作和通訊工作。 相較於佈建和使用傳統內部部署硬體，搭配使用 Azure 與 MATLAB 平台有許多好處︰能因應各種虛擬機器大小、可隨選建立叢集以便只針對使用的計算資源付費，以及能夠大規模測試模型。  

## <a name="prerequisites"></a>先決條件
* **用戶端電腦** - 您將需要以 Windows 為基礎的用戶端電腦，以便在部署之後與 Azure 和 MATLAB Distributed Computing Server 叢集通訊。
* **Azure PowerShell** - 請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 以將其安裝在您的用戶端電腦上。
* **Azure 訂用帳戶** - 如果您沒有訂用帳戶，只需要幾分鐘就可以建立 [免費帳戶](https://azure.microsoft.com/free/) 。 針對較大的叢集，請考慮隨用隨付訂用帳戶或其他購買選項。
* **vCPU 配額** -您可能需要增加 vCPU 配額，以部署大型叢集或多個 MATLAB Distributed Computing Server 叢集。 若要增加配額，請 [開立線上客戶支援要求](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/) (免費)。
* **MATLAB、Parallel Computing Toolbox 和 MATLAB Distributed Computing Server 授權** - 指令碼假設所有授權皆使用 [MathWorks Hosted License Manager](http://www.mathworks.com/products/parallel-computing/mathworks-hosted-license-manager/) 。  
* **MATLAB Distributed Computing Server 軟體** - 將會安裝在作為叢集 VM 之基礎 VM 映像的 VM 上。

## <a name="high-level-steps"></a>高階步驟
若要對 MATLAB Distributed Computing Server 叢集使用 Azure 虛擬機器，必須執行下列高階步驟。 [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/matlab-cluster)上的快速入門範本和指令碼所隨附的文件會有詳細指示。

1. **建立基礎 VM 映像**  

   * 在此 VM 上下載並安裝 MATLAB Distributed Computing Server 軟體。

     > [!NOTE]
     > 此程序可能需要幾個小時，但您所使用的每個 MATLAB 版本只需執行此作業一次。   
     >
     >
2. **建立一或多個叢集**  

   * 使用提供的 PowerShell 指令碼或使用快速入門範本，透過基礎 VM 映像建立叢集。   
   * 使用提供的 PowerShell 指令碼管理叢集，此指令碼可讓您列出、暫停、繼續和刪除叢集。

## <a name="cluster-configurations"></a>叢集組態
目前，叢集建立指令碼和範本可讓您建立單一的 MATLAB Distributed Computing Server 拓撲。 如果您想要的話，請另外建立一或多個叢集，讓每個叢集擁有不同數量的背景工作 VM、使用不同的 VM 大小等等。

### <a name="matlab-client-and-cluster-in-azure"></a>Azure 中的 MATLAB 用戶端和叢集
MATLAB 用戶端節點、MATLAB 作業排程器節點和 MATLAB Distributed Computing Server「背景工作」節點全都會設定為虛擬網路中的 Azure VM，如下圖所示。


* 若要使用叢集，請透過遠端桌面連接到用戶端節點。 用戶端節點會執行 MATLAB 用戶端。
* 用戶端節點具有可供所有背景工作角色存取的檔案共用。
* 所有 MATLAB 軟體的授權檢查都會使用 MathWorks Hosted License Manager。
* 在背景工作角色 VM 上，預設會為每個 vCPU 建立一個 MATLAB Distributed Computing Server 背景工作角色，但您可以指定任意數目。

## <a name="use-an-azure-based-cluster"></a>使用以 Azure 為基礎的叢集
和其他類型的 MATLAB Distributed Computing Server 叢集一樣，您必須在 MATLAB 用戶端 (位於用戶端 VM 上) 中使用 Cluster Profile Manager，以建立 MATLAB 作業排程器叢集設定檔。

![Cluster Profile Manager](./media/matlab-mdcs-cluster/cluster_profile_manager.png)

## <a name="next-steps"></a>後續步驟
* 如需在 Azure 中部署和管理 MATLAB Distributed Computing Server 叢集的詳細指示，請參閱包含範本和指令碼的 [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/matlab-cluster) 儲存機制。
* 如需 MATLAB 和 MATLAB Distributed Computing Server 的詳細文件，請移至 [MathWorks 網站](http://www.mathworks.com/) 。
