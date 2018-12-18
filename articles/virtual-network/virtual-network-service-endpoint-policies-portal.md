---
title: 建立服務端點原則並為其建立關聯 - Azure 入口網站 | Microsoft Docs
description: 在本文中，了解如何使用 Azure 入口網站設定服務端點原則並為其建立關聯。
services: virtual-network
documentationcenter: virtual-network
author: anithaa
manager: narayan
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 09/18/2018
ms.author: anithaa
ms.openlocfilehash: 8c750937cd45e104e754335f7da51d813b8dc7ba
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46999642"
---
# <a name="create-change-or-delete-service-endpoint-policy-using-the-azure-portal"></a>使用 Azure 入口網站建立、變更或刪除服務端點原則

服務端點原則可讓您透過服務端點來篩選特定 Azure 資源的虛擬網路流量。 如果您不熟悉服務端點原則，請參閱[服務端點原則概觀](virtual-network-service-endpoint-policies-overview.md)以深入了解。

 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立服務端點原則
> * 建立服務端點原則定義
> * 建立具有子網路的虛擬網路
> * 將服務端點原則建立關聯至子網路

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="sign-in-to-azure"></a>登入 Azure 

在 http://portal.azure.com 登入 Azure 入口網站。

## <a name="create-a-service-endpoint-policy"></a>建立服務端點原則

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 在搜尋窗格中，輸入「服務端點原則」，然後選取 [服務端點原則 (預覽)]，然後選取 [建立]。
3. 從 [基本] 中，輸入或選取以下資訊 

   - 訂用帳戶：為原則選取訂用帳戶。    
   - 資源群組：選取 [新建]，並輸入 *myResourceGroup*。     
   - 名稱：myEndpointPolicy
   - 位置：美國中西部     
 
   ![建立服務端點原則基本事項](./media/virtual-network-service-endpoint-policies-portal/virtual-network-endpoint-policies-create-startpane.PNG)
   
4. 從 [原則定義] 中，輸入或選取以下資訊

   - 按一下 [+新增資源]，輸入或選取下列資訊、接受其餘設定的預設值，然後按一下 [新增]。  
   - 範圍︰選取 [單一帳戶] 或 [訂用帳戶中的所有帳戶] 或 [資源群組中的所有帳戶]。    
   - 訂用帳戶：為儲存體帳戶選取訂用帳戶。 原則和儲存體帳戶可以位於不同的訂用帳戶中。   
   - 資源群組：選取您的資源群組。 如果範圍設為「資源群組中的所有帳戶」或「單一帳戶」，則為必要項目。  
   - 資源：mystorageaccountportal    
   - 按一下 [+ 新增資源]，繼續新增更多資源。
   
   ![建立服務端點原則定義](./media/virtual-network-service-endpoint-policies-portal/virtual-network-endpoint-policies-create-policydefinitionspane.PNG)
   
5. 選用：在 [標記] 中，輸入或選取以下資訊
   
   - 索引鍵：為您的原則選取索引鍵。 例如：部門     
   - 值：為索引建輸入值組。 例如：財務

6. 選取 [檢閱 + 建立]。 驗證資訊，然後按一下 [建立]。 若要進一步編輯，請按一下 [上一步]。 

   ![建立服務端點原則定義最終驗證](./media/virtual-network-service-endpoint-policies-portal/virtual-network-endpoint-policies-create-finalcreatereview.PNG)
  
 
## <a name="view-endpoint-policies"></a>檢視端點原則 

1. 在入口網站的 [所有服務] 方塊中，開始輸入*服務端點原則*。 選取 [服務端點原則 (預覽)]。
2. 在 [訂用帳戶] 下，選取您的訂用帳戶與資源群組，如下圖所示

   ![顯示原則](./media/virtual-network-service-endpoint-policies-portal/virtual-network-endpoint-policies-viewpolicies.PNG)
       
3. 選取原則，然後按一下 [原則定義]，以檢視或新增更多原則定義。

   ![顯示原則定義](./media/virtual-network-service-endpoint-policies-portal/virtual-network-endpoint-policies-viewpolicy-adddefinitions.PNG)

4. 選取 [相關聯的子網路] 以檢視與原則相關聯的子網路。 若要將原則與子網路建立關聯，請按一下 [導覽至相同區域中的虛擬網路]。

   ![顯示相關聯的子網路](./media/virtual-network-service-endpoint-policies-portal/virtual-network-endpoint-policies-view-associatedsubnets.PNG)
 
## <a name="associate-a-policy-to-a-subnet"></a>將原則建立關聯至子網路

>[!WARNING] 
> 請先針對選取的服務，確定從子網路存取的所有資源皆已新增至原則，才可將原則建立關聯。 將原則建立關聯後，只能針對服務的端點區域，存取原則中列出的資源。 

在您可以讓原則與子網路產生關聯之前，您必須先建立虛擬網路和子網路，然後才可以讓原則與子網路產生關聯：

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [網絡]，然後選取 [虛擬網路]。
3. 在 [建立虛擬網路] 底下，輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：
   - 名稱：myVirtualNetwork      
   - 位址空間：10.0.0.0/16      
   - 訂用帳戶︰選取您的訂用帳戶。 原則應該位於與 VNet 相同的訂用帳戶     
   - 資源群組：選取 [使用現有的]，然後選取 [myResourceGroup]     
   - 位置：美國中西部     
   - 子網路名稱：私用     
   - 位址範圍：10.0.0.0/24
     
4. 在入口網站頂端的 [搜尋資源、服務和文件] 方塊中，開始輸入 myVirtualNetwork。 當搜尋結果中出現 **myVirtualNetwork** 時加以選取。
5. 在 [設定] 底下選取 [子網路]，然後選取 [私用]。
6. 如下圖所示，選取 [服務端點]，選取 [Microsoft.Storage]，選取 [服務端點原則]，選取 [myEndpointPolicy]，然後選取 [儲存]：

   ![將原則建立關聯](./media/virtual-network-service-endpoint-policies-portal/virtual-network-endpoint-policies-associatepolicies.PNG)

>[!WARNING] 
>以網路安全性群組 (NSG) 為基礎，將允許從該子網路存取其他區域中的服務資源。 若要限制僅存取端點區域，請將 NSG 限制為僅為端點區域中的流量提供服務。 如需有關如何為每個區域建立帶有服務標記之 NSG 的詳細資訊，請參閱 [NSG Azure 服務標籤](manage-network-security-group.md?toc=%2fcreate-a-security-rule%2f.json)。

在下列範例中，NSG 限制為僅存取 WestCentralUS 和 WestUS2 中的 Azure 儲存體資源，並將「全部拒絕」規則作為優先順序較低的規則。

![拒絕所有 NSG](./media/virtual-network-service-endpoint-policies-portal/virtual-network-endpoint-policies-nsg-rules.PNG)


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已建立服務端點原則，並將其與子網路產生關聯。 若要深入了解服務端點原則，請參閱[服務端點原則概觀](virtual-network-service-endpoint-policies-overview.md)。

