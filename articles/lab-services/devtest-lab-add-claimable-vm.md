---
title: 於 Azure DevTest Labs 在實驗室中建立和管理可認領 VM | Microsoft Docs
description: 了解如何在 Azure DevTest Labs 中對實驗室新增可宣告的虛擬機器
services: devtest-lab,virtual-machines
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.assetid: f671e66e-9630-4e30-a131-a6bad9ed9c11
ms.service: devtest-lab
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/01/2018
ms.author: spelluru
ms.openlocfilehash: 669dfab75f34a0d1f997dc34f600402d3c10669b
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33781747"
---
# <a name="create-and-manage-claimable-vms-in-azure-devtest-labs"></a>在 Azure DevTest Labs 中建立和管理可認領 VM
您可以從[自訂映像](devtest-lab-create-template.md)、[公式](devtest-lab-manage-formulas.md)或 [Marketplace 映像](devtest-lab-configure-marketplace-images.md)等「基底」，使用和[新增標準 VM](devtest-lab-add-vm.md) 相似的方法將可宣告 VM 新增至實驗室。 本教學課程會逐步引導您使用 Azure 入口網站，新增可認領 VM 至 DevTest Labs 中的實驗室，並為使用者示範認領與取消認領 VM 的流程。

## <a name="steps-to-add-a-claimable-vm-to-a-lab-in-azure-devtest-labs"></a>將可宣告 VM 新增至 Azure DevTest Labs 中實驗室的步驟
1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
1. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。
1. 從實驗室清單中，選取您想要在其中建立可宣告 VM 的實驗室。  
1. 在實驗室的 [概觀] 窗格中，選取 [+ 新增]。  

    ![加入 VM 按鈕](./media/devtest-lab-add-vm/devtestlab-home-blade-add-vm.png)

1. 在 [選擇基底]  區域中，選取 VM 的基底。
1. 在 [虛擬機器] 窗格的 [虛擬機器名稱] 文字方塊中，輸入新虛擬機器的名稱。

    ![[實驗室 VM] 窗格](./media/devtest-lab-add-vm/devtestlab-lab-vm-blade.png)

1. 輸入**使用者名稱**，此名稱會被授與虛擬機器上的系統管理員權限。  
1. 如果您想使用儲存在[密碼存放區](https://azure.microsoft.com/updates/azure-devtest-labs-keep-your-secrets-safe-and-easy-to-use-with-the-new-personal-secret-store)中的密碼，請選取 [使用儲存的密碼]，並指定與密碼對應的金鑰值。 否則，請在標示為 [輸入值] 的文字欄位中輸入密碼。
1. [虛擬機器磁碟類型] 會決定實驗室中的虛擬機器所允許的儲存磁碟類型。
1. 選取 [虛擬機器大小]  ，然後選取其中一個預先定義的項目，這些項目可以指定處理器核心、RAM 大小，以及要建立的 VM 的硬碟大小。
1. 選取 [構件]，然後從構件清單中，選取並設定您想要新增到基本映像中的構件。 如果您對 DevTest Labs 或設定構件並不熟悉，請參閱[將現有的構件加入至 VM](devtest-lab-add-vm.md#add-an-existing-artifact-to-a-vm) 一節，完成該節之後再返回此處。
1. 選取 [進階設定] 以設定 VM 的網路選項和到期日選項。 在 [宣告選項] 底下，選擇 [是] 來允許宣告機器。

  ![選擇以允許宣告 VM。](./media/devtest-lab-add-vm/devtestlab-claim-VM-option.png)

1. 如果您想要檢視或複製 Azure Resource Manager 範本，請參閱[儲存 Azure Resource Manager 範本](devtest-lab-add-vm.md#save-azure-resource-manager-template)一節，然後在完成時回到這裡。
1. 選取 [建立]  ，將指定的 VM 加入實驗室。

   此時會顯示 VM 的建立狀態，其狀態會先是 [正在建立]，在啟動該 VM 之後才會變為 [正在執行]。

> [!NOTE]
> 如果您是透過 [Azure Resource Manager 範本](devtest-lab-create-environment-from-arm.md)部署實驗室 VM，您可以在 [屬性] 區段中將 **allowClaim** 屬性設定為 [true]，來建立可宣告 VM。
>
>

## <a name="using-a-claimable-vm"></a>使用可宣告 VM

使用者可以透過執行下列其中一項步驟，來宣告 [可宣告的虛擬機器] 清單上的任何 VM：

* 在實驗室 [概觀] 窗格底部的 [可認領的虛擬機器] 清單中，以滑鼠右鍵按一下清單中的其中一個 VM，並選擇 [認領機器]。

 ![要求特定可宣告 VM。](./media/devtest-lab-add-vm/devtestlab-claim-VM.png)


* 在 [概觀] 窗格的頂端，選擇 [認領任何項目]。 系統將會從可宣告 VM 清單中隨機指派虛擬機器。

 ![要求任何可宣告 VM。](./media/devtest-lab-add-vm/devtestlab-claim-any.png)


在使用者宣告 VM 之後，該 VM 將會移至該使用者的 [我的虛擬機器] 清單，且其他使用者將無法宣告該 VM。

## <a name="unclaim-a-vm"></a>取消認領 VM

使用者使用完已認領 VM，想將 VM 提供給其他人使用時，可執行以下其中一個步驟，將已認領 VM 傳回至可認領虛擬機器的清單：

- 從 [我的虛擬機器] 清單中，以滑鼠右鍵按一下清單中的其中一個 VM，或是選取其省略符號 (...)，然後選擇 [取消認領]。

  ![在 VM 清單上取消認領 VM。](./media/devtest-lab-add-vm/devtestlab-unclaim-VM2.png)

- 在 [我的虛擬機器] 清單中，選取 VM 以開啟其管理窗格，然後從頂端功能表列中選取 [取消認領]。

  ![在 VM 的管理窗格上取消認領 VM。](./media/devtest-lab-add-vm/devtestlab-unclaim-VM.png)

使用者若取消認領 VM，即不再擁有該特定實驗室 VM 的任何權限。

### <a name="transferring-the-data-disk"></a>傳送資料磁碟
若可認領 VM 有附加資料磁碟，而使用者取消認領該 VM，資料磁碟會與該 VM 一同保留。 若有另一位使用者認領該 VM，則該名新使用者會連同 VM 一併認領資料磁碟。

此稱為「傳送資料磁碟」。 隨後，資料磁碟即可在新使用者的 [我的資料磁碟] 清單中供其進行管理。

![取消認領資料磁碟。](./media/devtest-lab-add-vm/devtestlab-unclaim-datadisks.png)



## <a name="next-steps"></a>後續步驟
* 建立完成後，您可選取其管理窗格上的 [連線] 來重新連線至 VM。
* 瀏覽 [DevTest Labs Azure Resource Manager 快速入門範本資源庫](https://github.com/Azure/azure-devtestlab/tree/master/Samples)\(英文\)。
