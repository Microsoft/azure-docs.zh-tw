---
title: 將高載節點新增至 HPC Pack 叢集 | Microsoft Docs
description: 了解如何新增在雲端服務中執行的背景工作角色執行個體，以隨選擴充 Azure 中的 HPC Pack 叢集
services: virtual-machines-windows
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-service-management,hpc-pack
ms.assetid: 24b79a8a-24ad-4002-ae76-75abc9b28c83
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-multiple
ms.workload: big-compute
ms.date: 10/14/2016
ms.author: danlep
ms.openlocfilehash: eee9183321f21676271c8a9c7e023c80c4daf554
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30915101"
---
# <a name="add-on-demand-burst-nodes-to-an-hpc-pack-cluster-in-azure"></a>將隨選「高載」節點新增至 Azure 中的 HPC Pack 叢集
如果您在 Azure 中設定 [Microsoft HPC Pack](https://technet.microsoft.com/library/cc514029) 叢集，您可能想迅速相應增加或相應減少叢集容量，而不用維護一組預先設定的計算節點 VM。 本文說明如何將隨選「高載」節點 (在雲端服務中執行的背景工作角色執行個體) 新增至 Azure 中的前端節點作為運算資源。 

> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

![高載節點][burst]

本文中的步驟可協助您將 Azure 節點快速新增至雲端型 HPC Pack 前端節點 VM，以進行測試或概念驗證部署。 高階步驟與將雲端計算能力新增至內部部署 HPC Pack 叢集的「將暴增的工作負載移至 Azure」步驟相同。 如需教學課程，請參閱 [使用 Microsoft HPC Pack 設定混合式計算叢集](../../../cloud-services/cloud-services-setup-hybrid-hpcpack-cluster.md)。 如需生產部署的詳細指引和考量，請參閱 [使用 Microsoft HPC Pack 將暴增的工作負載移至 Azure](https://technet.microsoft.com/library/gg481749.aspx)。

## <a name="prerequisites"></a>先決條件
* **部署在 Azure VM 中的 HPC Pack 前端節點** -您可以使用獨立的前端節點 VM 或屬於較大叢集的節點。 若要建立獨立的前端節點，請參閱 [在 Azure VM 中部署 HPC Pack 前端節點](../../virtual-machines-windows-hpcpack-cluster-headnode.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 如需自動化的 HPC Pack 叢集部署選項，請參閱 [使用 Microsoft HPC Pack 在 Azure 中建立及管理 Windows HPC 叢集的選項](../../virtual-machines-windows-hpcpack-cluster-options.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
  
  > [!TIP]
  > 如果您使用 [HPC Pack IaaS 部署指令碼](hpcpack-cluster-powershell-script.md) 在 Azure 中建立叢集，則可以在自動化部署中包含 Azure 高載節點。 請參閱該文件中的範例。
  > 
  > 
* **Azure 訂用帳戶** - 若要新增 Azure 節點，您可以選擇與用來部署前端節點 VM 相同的訂用帳戶，或選擇一或多個不同的訂用帳戶。
* **核心配額** - 您可能需要增加核心的配額，特別是當您選擇部署數個具有多核心大小的 Azure 節點時。 若要增加配額，請 [開立線上客戶支援要求](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/) (免費)。

## <a name="step-1-create-a-cloud-service-and-a-storage-account-for-the-azure-nodes"></a>步驟 1：為 Azure 節點建立雲端服務和儲存體帳戶
使用 Azure 入口網站或對等工具，以設定下列在部署 Azure 節點時所需的資源：

* 新的 Azure 雲端服務 (傳統)
* 新的 Azure 儲存體帳戶 (傳統)

> [!NOTE]
> 請勿重複使用您訂用帳戶中現有的雲端服務。 
> 
> 

**考量**

* 您打算建立的每個 Azure 節點範本設定個別的雲端服務。 不過，您可以將同一個儲存體帳戶用於多個節點範本。
* 建議您將用於部署的雲端服務和儲存體帳戶放置在相同的 Azure 區域中。

## <a name="step-2-configure-an-azure-management-certificate"></a>步驟 2：設定 Azure 管理憑證
若要將 Azure 節點新增為運算資源，您在前端節點上必須要有管理憑證，且必須將對應的憑證上傳至用於部署的 Azure 訂用帳戶。

針對這個案例，您可以選擇 HPC Pack 在前端節點上自動安裝及設定的 **預設 HPC Azure 管理憑證** 。 此憑證可用於測試及概念證明部署。 若要使用此憑證，請將檔案 C:\Program Files\Microsoft HPC Pack 2012\Bin\hpccert.cer 從前端節點 VM 上傳至訂用帳戶。 在 [Azure 入口網站](https://portal.azure.com)中上傳憑證：

1. 按一下 [訂用帳戶] > 您的訂用帳戶名稱。

2. 按一下 [管理憑證] > [上傳]。

如需設定管理憑證的其他選項，請參閱 [為 Azure 高載部署設定 Azure 管理憑證的案例](http://technet.microsoft.com/library/gg481759.aspx)。

## <a name="step-3-deploy-azure-nodes-to-the-cluster"></a>步驟 3：將 Azure 節點部署至叢集
在此案例中新增及啟動 Azure 節點的步驟，與用於內部部署前端節點的步驟大致相同。 如需詳細資訊，請參閱 [使用 Microsoft HPC Pack 部署 Azure 節點的步驟](https://technet.microsoft.com/library/gg481758.aspx)中的下列小節：

* 建立 Azure 節點範本
* 將 Azure 節點新增至 Windows HPC 叢集
* 啟動 (佈建) Azure 節點

節點新增並啟動之後，即可供您用來執行叢集工作。

如果您在部署 Azure 節點時遇到問題，請參閱 [使用 Microsoft HPC Pack 部署 Azure 節點時的疑難排解](http://technet.microsoft.com/library/jj159097.aspx)。

## <a name="next-steps"></a>後續步驟
* 若要對高載節點使用密集計算的執行個體大小，請參閱[高效能運算 VM 大小](../sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中的考量。
* 如果您想要根據叢集工作負載自動擴增或縮減 Azure 計算資源，請參閱 [在 HPC Pack 叢集中自動擴增和縮減 Azure 運算資源](hpcpack-cluster-node-autogrowshrink.md)。

<!--Image references-->
[burst]: ./media/hpcpack-cluster-node-burst/burst.png
