---
title: 在虛擬網路中佈建 Azure Batch 集區 | Microsoft Docs
description: 您可以在虛擬網路中建立 Batch 集區，以便計算節點可以與網路中的其他 VM (例如，檔案伺服器) 安全地通訊。
services: batch
author: dlepow
manager: jeconnoc
ms.service: batch
ms.topic: article
ms.date: 08/15/2018
ms.author: danlep
ms.openlocfilehash: 9e8bd6819601cc4436e3432ee390f2ae82a0d94a
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42144836"
---
# <a name="create-an-azure-batch-pool-in-a-virtual-network"></a>在虛擬網路中建立 Azure Batch 集區


當您建立 Azure Batch 集區時，您可以在您指定的 [Azure 虛擬網路](../virtual-network/virtual-networks-overview.md) (VNet) 子網路中佈建集區。 本文會說明如何在 VNet 中設定 Batch 集區。 



## <a name="why-use-a-vnet"></a>為何要使用 VNet？


Azure Batch 集區有設定可讓計算節點彼此通訊，以便 (舉例來說) 執行多執行個體工作。 這些設定不需要個別的 VNet。 但根據預設，節點無法與不屬於 Batch 集區的虛擬機器 (例如，授權伺服器或檔案伺服器) 通訊。 若要讓集區的計算節點能與其他虛擬機器或內部部署網路安全地通訊，您可以在 Azure VNet 的子網路中佈建集區。 



## <a name="prerequisites"></a>必要條件

* **驗證**。 若要使用 Azure VNet，Batch 用戶端 API 必須使用 Azure Active Directory (AD) 驗證。 Azure Batch 對於 Azure AD 的支援記載於[使用 Active Directory 驗證 Batch 服務解決方案](batch-aad-auth.md)中。 

* **Azure VNet**。 若要事先準備具有一個或多個子網路的 VNet，您可以使用 Azure 入口網站、Azure PowerShell、Azure 命令列介面 (CLI) 或其他方法。 若要建立以 Azure Resource Manager 為基礎的 VNet，請參閱[建立虛擬網路](../virtual-network/manage-virtual-network.md#create-a-virtual-network)。 若要建立傳統 VNet，請參閱[建立有多個子網路的虛擬網路 (傳統)](../virtual-network/create-virtual-network-classic.md)。

### <a name="vnet-requirements"></a>VNet 需求
[!INCLUDE [batch-virtual-network-ports](../../includes/batch-virtual-network-ports.md)]
    
## <a name="create-a-pool-with-a-vnet-in-the-portal"></a>在入口網站中建立具有 VNet 的集區

在建立了 VNet 並為其指派子網路後，您可以建立具有該 VNet 的 Batch 集區。 請遵循下列步驟，從 Azure 入口網站建立集區： 



1. 在 Azure 入口網站中瀏覽至您的 Batch 帳戶。 此帳戶必須位於與您要使用之 VNet 所在的資源群組相同的訂用帳戶和區域中。 
2. 在左側的 [設定] 視窗中，選取 [集區] 功能表項目。
3. 在 [集區] 視窗中，選取 [新增] 命令。
4. 在 [新增集區] 視窗上，從 [映像類型] 下拉式清單選取您要使用的選項。 
5. 依據您的自訂映像選取正確的**發行者/提供項目/SKU**。
6. 指定其餘的必要設定，包括**節點大小**、**目標專用的節點**和**低優先權的節點**，以及所需的任何選擇性設定。
7. 在 [虛擬網路] 中，選取您想要使用的虛擬網路和子網路。
  
  ![新增具有虛擬網路的集區](./media/batch-virtual-network/add-vnet-pool.png)

## <a name="user-defined-routes-for-forced-tunneling"></a>強制通道的使用者定義路由

您的組織可能需要從子網路將網際網路繫結流量重新導向 (強制) 回到內部部署位置，以便進行檢查和記錄。 您可能已在 VNet 中啟用子網路的強制通道。 

若要確保 Azure Batch 集區的計算節點會在已啟用強制通道的 VNet 中運作，您必須為該子網路新增下列[使用者定義的路由](../virtual-network/virtual-networks-udr-overview.md)：

* Batch 服務需要與集區的計算節點通訊，以便安排工作。 若要啟用此通訊，請在 Batch 帳戶所在的區域中，為 Batch 服務所使用的每個 IP 位址新增使用者定義的路由。 若要取得 Batch 服務的 IP 位址清單，請連絡 Azure 支援。

* 確定您並未透過內部部署網路應用裝置禁止輸出到 Azure 儲存體的流量 (具體地說，就是 `<account>.table.core.windows.net`、`<account>.queue.core.windows.net` 和 `<account>.blob.core.windows.net` 表單的 URL)。

當您新增使用者定義的路由時，請為每個相關的「批次 IP」位址首碼定義路由，然後將 [下一個躍點類型] 設定為 [網際網路]。 請參閱下列範例：

![使用者定義路由](./media/batch-virtual-network/user-defined-route.png)

## <a name="next-steps"></a>後續步驟

- 如需 Batch 的深入概觀，請參閱[使用 Batch 開發大規模的平行計算解決方案](batch-api-basics.md)。
- 如需有關建立使用者定義路由的詳細資訊，請參閱[建立使用者定義的路由 - Azure 入口網站](../virtual-network/tutorial-create-route-table-portal.md)。
