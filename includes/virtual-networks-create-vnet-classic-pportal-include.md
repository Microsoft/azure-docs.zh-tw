---
title: 包含檔案
description: 包含檔案
services: virtual-network
author: genlin
ms.service: virtual-network
ms.topic: include
ms.date: 04/13/2018
ms.author: genli
ms.custom: include file
ms.openlocfilehash: 815a217526117325ff80759701141b2b4d148514
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31805296"
---
## <a name="how-to-create-a-classic-vnet-in-the-azure-portal"></a>如何在 Azure 入口網站中建立傳統 VNet
若要以上述案例為基礎建立傳統 VNet，請遵循下列步驟執行。

1. 透過瀏覽器瀏覽至 http://portal.azure.com，並視需要使用您的 Azure 帳戶登入。
2. 按一下 [建立資源] > [網路] > [虛擬網路]。 請注意，[選取部署模型] 清單已顯示 [傳統]。 3. 按一下 [建立]，如下圖所示。
   
    ![在 Azure 入口網站中建立 VNet](./media/virtual-networks-create-vnet-classic-pportal-include/vnet-create-pportal-figure1.gif)
4. 在 [虛擬網路] 窗格上，輸入 VNet 的 [名稱]，然後再按一下 [位址空間]。 設定 VNet 和其第一個子網路的位址空間設定，然後按一下 [ **確定**]。 下圖顯示我們案例的 CIDR 區塊設定。
   
    ![位址空間窗格](./media/virtual-networks-create-vnet-classic-pportal-include/vnet-create-pportal-figure2.png)
5. 按一下 [資源群組]，選取要新增 VNet 的資源群組，或按一下 [建立新的資源群組]，將 VNet 新增到新的資源群組。 下圖顯示名為 **TestRG** 的新資源群組之資源群組設定。 如需資源群組的詳細資訊，請造訪 [Azure 資源管理員概觀](../articles/azure-resource-manager/resource-group-overview.md#resource-groups)。
   
    ![建立資源群組窗格](./media/virtual-networks-create-vnet-classic-pportal-include/vnet-create-pportal-figure3.png)
6. 如有必要，變更 VNet 的**訂用帳戶**和**位置**設定。 
7. 如果您不想在 **「開始面板」** 上看到 VNet 圖格，請停用 [釘選到「開始面板」]。 
8. 按一下 [建立]，並注意名為 [建立虛擬網路] 的圖格，如下圖所示。
   
    ![在入口網站中建立 VNet](./media/virtual-networks-create-vnet-classic-pportal-include/vnet-create-pportal-figure4.png)
9. 等候建立 VNet，然後在看到圖格之後，按一下以新增更多子網路。
   
    ![在入口網站中建立 VNet](./media/virtual-networks-create-vnet-classic-pportal-include/vnet-create-pportal-figure5.png)
10. 您應該會看到 VNet 的 [設定]，如下所示。 
   
    ![在入口網站中建立 VNet](./media/virtual-networks-create-vnet-classic-pportal-include/vnet-create-pportal-figure6.png)
11. 依序按一下 [子網路]  >  [新增]，然後鍵入 [名稱]，並指定您子網路的 [位址範圍 (CIDR 區塊)]，然後按一下 [確定]。 下圖顯示我們目前案例的設定。
    
    ![在 Azure 入口網站中建立 VNet](./media/virtual-networks-create-vnet-classic-pportal-include/vnet-create-pportal-figure7.gif)

