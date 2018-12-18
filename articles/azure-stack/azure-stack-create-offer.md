---
title: 在 Azure Stack 中建立供應項目 | Microsoft Docs
description: 身為雲端系統管理員，您應該了解如何在 Azure Stack 中為使用者建立供應項目。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/12/2018
ms.author: sethm
ms.reviewer: efemmano
ms.openlocfilehash: 4ccff997c7e9f29aafc6966730ab36dfcf72ca9f
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49077335"
---
# <a name="create-an-offer-in-azure-stack"></a>在 Azure Stack 中建立優惠

[供應項目](azure-stack-key-features.md)是一組提供者供使用者購買或訂閱的一或多個方案。 此文件將說明如何建立一個包含[所建立方案](azure-stack-create-plan.md)的供應項目。 此供應項目可讓訂閱者設定虛擬機器。

1. 登入 Azure Stack 系統管理員入口網站 (https://adminportal.local.azurestack.external)，然後選取 [+建立資源] > [租用戶供應項目 + 方案] > [供應項目]。

   ![建立供應項目](media/azure-stack-create-offer/image01.png)
  
2. 在 [新增供應項目] 下，輸入 [顯示名稱] 與 [資源名稱]，然後在 [資源群組] 下選取 [新建] 或 [使用現有的]。 [顯示名稱] 是易記的供應項目名稱。 此易記名稱是使用者訂閱供應項目時唯一會看到的供應項目相關資訊。 請使用可協助使用者了解該供應項目附帶內容的直覺式名稱。 只有系統管理員可以看到「資源名稱」。 它是系統管理員用來處理其他供應項目 (以 Azure Resource Manager 資源方式) 的名稱。

   ![新增供應項目](media/azure-stack-create-offer/image01a.png)
  
3. 選取 [基本方案] 來開啟 [方案]。 選取要包含在供應項目中的方案，然後選擇 [選取]。 若要建立供應項目，請選取 [建立]。

   ![選取方案](media/azure-stack-create-offer/image02.png)
  
4. 建立供應項目之後，您可以變更其狀態。 供應項目必須設定為「公用」，使用者才能在訂閱時取得完整的檢視。 供應項目可以是：

   - **公用**：使用者可以看到。
   - **私用**：只有雲端系統管理員可以看到。 在草擬方案或供應項目時，或雲端系統管理員想要[為使用者建立訂用帳戶](azure-stack-subscribe-plan-provision-vm.md#create-a-subscription-as-a-cloud-operator)時，此設定會相當實用。
   - **已解除**：不開放給新的訂閱者。 雲端系統管理員可以使用「已解除」來避免之後的訂用帳戶使用，但不會影響目前的訂閱者。

   > [!TIP]  
   > 對供應項目所做的變更不會立即對使用者顯示。 若要查看變更，使用者可能必須先登出並再次登入使用者入口網站，才能看見新的供應項目。

   在供應項目的 [概觀] 上，選取 [可存取性狀態]。 選擇您想要使用的狀態 (例如，公用)，然後選取 [儲存]。
 
     ![選擇狀態](media/azure-stack-create-offer/change-stage-1807.png)

     或者，您可以選取 [變更狀態]，然後選擇狀態。

    ![選取可存取性狀態](media/azure-stack-create-offer/change-stage-select-1807.png)

   > [!NOTE]
   > 您也可以使用 PowerShell 來建立預設供應項目、方案和配額。 如需詳細資訊，請參閱 [Azure Stack PowerShell Module 1.4.0](https://docs.microsoft.com/powershell/azure/azure-stack/overview?view=azurestackps-1.4.0)。

## <a name="next-steps"></a>後續步驟

- [建立訂用帳戶](azure-stack-subscribe-plan-provision-vm.md)
- [佈建虛擬機器](azure-stack-provision-vm.md)
