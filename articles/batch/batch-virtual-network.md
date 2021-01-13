---
title: 在虛擬網路中佈建集區
description: 如何在 Azure 虛擬網路中建立 Batch 集區，以便計算節點可以與網路中的其他 VM (例如，檔案伺服器) 安全地通訊。
ms.topic: how-to
ms.date: 01/13/2021
ms.custom: seodec18
ms.openlocfilehash: bedb0cbd826e2891560320ed11c0ba28e07f8e88
ms.sourcegitcommit: 16887168729120399e6ffb6f53a92fde17889451
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/13/2021
ms.locfileid: "98165751"
---
# <a name="create-an-azure-batch-pool-in-a-virtual-network"></a>在虛擬網路中建立 Azure Batch 集區

當您建立 Azure Batch 集區時，您可以在您指定的 [Azure 虛擬網路](../virtual-network/virtual-networks-overview.md) (VNet) 子網路中佈建集區。 本文會說明如何在 VNet 中設定 Batch 集區。

## <a name="why-use-a-vnet"></a>為何要使用 VNet？

集區中的計算節點可以彼此通訊，例如執行多重實例工作，而不需要個別的 VNet。 不過，根據預設，集區中的節點無法與集區外部的虛擬機器進行通訊，例如授權伺服器或檔案伺服器。

若要允許計算節點與其他虛擬機器或內部部署網路安全地通訊，您可以在 Azure VNet 的子網中布建集區。

## <a name="prerequisites"></a>Prerequisites

- **驗證**。 若要使用 Azure VNet，Batch 用戶端 API 必須使用 Azure Active Directory (AD) 驗證。 Azure Batch 對於 Azure AD 的支援記載於[使用 Active Directory 驗證 Batch 服務解決方案](batch-aad-auth.md)中。

- **Azure VNet**。 請參閱下一節以取得 VNet 的需求和組態。 若要事先準備具有一個或多個子網路的 VNet，您可以使用 Azure 入口網站、Azure PowerShell、Azure 命令列介面 (CLI) 或其他方法。
  - 若要建立以 Azure Resource Manager 為基礎的 VNet，請參閱[建立虛擬網路](../virtual-network/manage-virtual-network.md#create-a-virtual-network)。 建議針對新的部署使用以 Resource Manager 為基礎的 VNet，而且只有使用虛擬機器設定的集區才支援。
  - 若要建立傳統 VNet，請參閱[建立有多個子網路的虛擬網路 (傳統)](/previous-versions/azure/virtual-network/create-virtual-network-classic)。 只有使用雲端服務設定的集區才支援傳統 VNet。

## <a name="vnet-requirements"></a>VNet 需求

[!INCLUDE [batch-virtual-network-ports](../../includes/batch-virtual-network-ports.md)]

## <a name="create-a-pool-with-a-vnet-in-the-azure-portal"></a>在 Azure 入口網站中建立具有 VNet 的集區

在建立了 VNet 並為其指派子網路後，您可以建立具有該 VNet 的 Batch 集區。 請遵循下列步驟，從 Azure 入口網站建立集區： 

1. 在 Azure 入口網站中瀏覽至您的 Batch 帳戶。 此帳戶必須位於與您要使用之 VNet 所在的資源群組相同的訂用帳戶和區域中。
2. 在左側的 [設定] 視窗中，選取 [集區] 功能表項目。
3. **在 [集** 區] 視窗中，選取 [**新增**]。
4. 在 [新增集區] 視窗上，從 [映像類型] 下拉式清單選取您要使用的選項。
5. 依據您的自訂映像選取正確的 **發行者/提供項目/SKU**。
6. 指定其餘的必要設定，包括 **節點大小**、**目標專用的節點** 和 **低優先權的節點**，以及所需的任何選擇性設定。
7. 在 [虛擬網路] 中，選取您想要使用的虛擬網路和子網路。

   ![新增具有虛擬網路的集區](./media/batch-virtual-network/add-vnet-pool.png)

## <a name="user-defined-routes-for-forced-tunneling"></a>強制通道的使用者定義路由

您的組織可能需要重新導向 (強制) 從子網傳回到內部部署位置的網際網路系結流量，以進行檢查和記錄。 此外，您可能已針對 VNet 中的子網啟用強制通道。

若要確保集區中的節點在已啟用強制通道的 VNet 中運作，您必須將下列 [使用者定義的路由](../virtual-network/virtual-networks-udr-overview.md) 新增 (子網的 UDR) ：

- Batch 服務需要與節點進行通訊，以排程工作。 若要啟用此通訊，請在 Batch 帳戶所在的區域中，為 Batch 服務所使用的每個 IP 位址新增 UDR。 若要取得 Batch 服務的 IP 位址清單，請參閱 [內部部署服務標記](../virtual-network/service-tags-overview.md)。

- 確定 `<account>.table.core.windows.net` `<account>.queue.core.windows.net` 您的 `<account>.blob.core.windows.net` 內部部署網路不會封鎖表單、和) 的 url，以明確 Azure 儲存體 (的輸出流量。

當您新增 UDR 時，請為每個相關的 Batch IP 位址首碼定義路由，然後將 [下一個躍點類型] 設定為 [網際網路]。

![使用者定義路由](./media/batch-virtual-network/user-defined-route.png)

> [!WARNING]
> Batch 服務 IP 位址可能會隨著時間變更。 若要防止由於 IP 位址變更而中斷，請建立一個程式來自動重新整理 Batch 服務 IP 位址，並將其保持在路由表中的最新狀態。

## <a name="next-steps"></a>後續步驟

- 深入了解 [Batch 服務工作流程和主要資源](batch-service-workflow-features.md)，例如集區、節點、作業和工作。
- 瞭解如何 [在 Azure 入口網站中建立使用者定義的路由](../virtual-network/tutorial-create-route-table-portal.md)。
