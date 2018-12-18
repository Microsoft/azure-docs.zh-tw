---
title: 管理 Azure DevTest Labs 中的公式來建立 VM | Microsoft Docs
description: 了解如何更新和移除 Azure DevTest Labs 公式
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 841dd95a-657f-4d80-ba26-59a9b5104fe4
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: b7a68f545f60829e5da83f0734c57a4d210cb843
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39001476"
---
# <a name="manage-azure-devtest-labs-formulas"></a>管理 Azure DevTest Labs 公式

[!INCLUDE [devtest-lab-formula-definition](../../includes/devtest-lab-formula-definition.md)]

本文說明如何從基礎 (自訂映像、Marketplace 映像或另一個公式) 或現有的 VM 建立公式。 本文也會引導您管理現有公式。

## <a name="create-a-formula"></a>建立公式
具有 DevTest Labs「使用者」  權限的所有使用者，都可以使用公式作為基底來建立 VM。 有兩種方式可以建立公式： 

* 從基底開始 - 在您想要定義公式的所有特性時使用。
* 從現有實驗室 VM - 在您想要根據現有 VM 的設定建立公式時使用。

如需有關新增使用者和權限的詳細資訊，請參閱[在 Azure DevTest Labs 中新增擁有者和使用者](./devtest-lab-add-devtest-user.md)。

### <a name="create-a-formula-from-a-base"></a>從基底建立公式
下列步驟會引導您完成從自訂映像、Marketplace 映像或其他公式建立公式的程序。

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。

2. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。

3. 從實驗室清單中，選取所需的實驗室。  

4. 在實驗室的刀鋒視窗上，選取 [公式 (可重複使用的基底)] 。
   
    ![公式功能表](./media/devtest-lab-create-formulas/lab-settings-formulas.png)

5. 在 [公式] 刀鋒視窗上，選取 [+新增]。
   
    ![加入公式](./media/devtest-lab-create-formulas/add-formula.png)

6. 在 [選擇基底]  刀鋒視窗上，選取您想要用來建立公式的基底 (自訂映像、Marketplace 映像或公式)。
   
    ![基本清單](./media/devtest-lab-create-formulas/base-list.png)

7. 在 [建立公式]  刀鋒視窗上，指定下列值：
   
    * **公式名稱** - 輸入公式的名稱。 當您建立 VM 時，這個值會在顯示在基本映像清單中。 輸入名稱時會立即驗證，如果無效，則會出現訊息指出有效名稱的需求。
    * **描述** - 輸入公式的有意義描述。 當您建立 VM 時，可以從公式的操作功能表使用這個值。
    * **使用者名稱** - 輸入被授與系統管理員權限的使用者名稱。
    * **密碼** - 從下拉式清單中輸入或選取一個值，該值與您想用於指定使用者的密碼相關聯。 若要深入了解如何在金鑰保存庫中儲存祕密以及在建立實驗室資源時使用它們，請參閱[在 Azure Key Vault 中儲存祕密](devtest-lab-store-secrets-in-key-vault.md)。
    * **虛擬機器磁碟類型** - 指定 HDD (硬碟) 或 SSD (固態硬碟)，以表示使用此基本映像佈建的虛擬機器可使用哪種儲存磁碟類型。
    * **虛擬機器大小** - 選取其中一個預先定義的項目，以指定要建立之 VM 的處理器核心、RAM 大小和硬碟大小。 
    * **構件** - 選取以開啟 [新增構件] 刀鋒視窗，以選取並設定您想要新增至基本映像的構件。 如需構件的詳細資訊，請參閱[建立 Azure DevTest Labs 虛擬機器的自訂構件](devtest-lab-artifact-author.md)。
    * **進階設定** - 選取以開啟 [進階] 刀鋒視窗，以進行下列設定︰
        * **虛擬網路** - 指定所需的虛擬網路。
        * **子網路** - 指定所需的子網路。    
        * **IP 位址組態** - 指定您想要公用、私人或共用 IP 位址。 如需共用 IP 位址的詳細資訊，請參閱[了解 Azure DevTest Labs 中的共用 IP 位址](./devtest-lab-shared-ip.md)。
        * **允許宣告此機器** -「允許宣告」機器表示建立機器時不會指派擁有權。 實驗室使用者將能夠在實驗室的刀鋒視窗中，取得 (「宣告」) 機器的擁有權。     
    * **映像** - 這個欄位會顯示您在前一個刀鋒視窗上選取的基本映像名稱。 
     
       ![建立公式](./media/devtest-lab-create-formulas/create-formula.png)

8. 選取 [建立]  以建立公式。

9. 公式建立之後會顯示在 [公式] 刀鋒視窗的清單中。

### <a name="create-a-formula-from-a-vm"></a>從 VM 建立公式
下列步驟會引導您完成根據現有 VM 建立公式的程序。 

> [!NOTE]
> VM 必須是在 2016 年 3 月 30 日以後建立的，才能從該 VM 建立公式。 
> 
> 

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
2. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。
3. 從實驗室清單中，選取所需的實驗室。  
4. 在實驗室的 [概觀]  刀鋒視窗中，選取要從中建立公式的 VM。
   
    ![實驗室 VM](./media/devtest-lab-create-formulas/my-vms.png)
5. 在 VM 的刀鋒視窗上，選取 [建立公式 (可重複使用的基底)] 。
   
    ![建立公式](./media/devtest-lab-create-formulas/create-formula-menu.png)
6. 在 [建立公式] 刀鋒視窗上，輸入新公式的 [名稱] 和 [描述]。
   
    ![建立公式刀鋒視窗](./media/devtest-lab-create-formulas/create-formula-blade.png)
7. 選取 [確定]  以建立公式。

## <a name="modify-a-formula"></a>修改公式
若要修改公式，請遵循下列步驟︰

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
2. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。
3. 從實驗室清單中，選取所需的實驗室。  
4. 在實驗室的刀鋒視窗上，選取 [公式 (可重複使用的基底)] 。
   
    ![公式功能表](./media/devtest-lab-manage-formulas/lab-settings-formulas.png)
5. 在 [Lab formulas] \(實驗室公式)  刀鋒視窗上，選取您想要修改的公式。
6. 在 [更新公式] 刀鋒視窗上，進行所需的編輯，然後選取 [更新]。

## <a name="delete-a-formula"></a>刪除公式
若要刪除公式，請遵循下列步驟︰

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
2. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。
3. 從實驗室清單中，選取所需的實驗室。  
4. 在實驗室的 [設定] 刀鋒視窗上，選取 [公式]。
   
    ![公式功能表](./media/devtest-lab-manage-formulas/lab-settings-formulas.png)
5. 在 [Lab formulas] \(實驗室公式)  刀鋒視窗上，選取您想要刪除之公式右邊的省略符號。
   
    ![公式功能表](./media/devtest-lab-manage-formulas/lab-formulas-blade.png)
6. 在公式的操作功能表上，選取 [刪除] 。
   
    ![公式操作功能表](./media/devtest-lab-manage-formulas/formula-delete-context-menu.png)
7. 在刪除確認對話方塊上，選取 [是]  。

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="related-blog-posts"></a>相關部落格文章
* [自訂映像或公式？](https://blogs.msdn.microsoft.com/devtestlab/2016/04/06/custom-images-or-formulas/)

## <a name="next-steps"></a>後續步驟
建立要在建立 VM 時使用的公式之後，下一個步驟就是[將 VM 新增實驗室](devtest-lab-add-vm.md)。

