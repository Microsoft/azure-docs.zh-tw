---
title: 在 Azure DevTest Labs 中存取實驗室 | Microsoft Docs
description: 在本教學課程中，您會存取使用 Azure DevTest Labs 建立的實驗室、認領虛擬機器並加以使用，以及取消認領虛擬機器。
services: devtest-lab, lab-services, virtual-machines
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 05/17/2018
ms.author: spelluru
ms.openlocfilehash: cd623767c9627810afb64ca9185c991c5c9f3858
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34638019"
---
# <a name="tutorial-access-a-lab-in-azure-devtest-labs"></a>教學課程：在 Azure DevTest Labs 中存取實驗室
在本教學課程中，您會使用[教學課程：在 Azure DevTest Labs 中建立實驗室](tutorial-create-custom-lab.md)中建立的實驗室。

在本教學課程中，您會執行下列動作：

> [!div class="checklist"]
> * 宣告實驗室中的虛擬機器 (VM)
> * 連接至 VM
> * 取消認領 VM

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/) 。

## <a name="access-the-lab"></a>存取實驗室

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在左側功能表上，選取 [所有資源]。 
3. 選取 [DevTest Labs] 作為資源類型。 
4. 選取實驗室。 

    ![選取實驗室](./media/tutorial-use-custom-lab/search-for-select-custom-lab.png)

## <a name="claim-a-vm"></a>認領 VM

1. 在 [可認領虛擬機器] 清單中，選取 [...] (省略符號)，然後選取 [認領機器]。

    ![認領虛擬機器](./media/tutorial-use-custom-lab/claim-virtual-machine.png)
1. 確認您有在 [我的虛擬機器] 清單中看到虛擬機器。

    ![我的虛擬機器](./media/tutorial-use-custom-lab/my-virtual-machines.png)

## <a name="connect-to-the-vm"></a>連接至 VM

1. 在清單中選取您的 VM。 您會看到 VM 的**虛擬機器頁面**。 選取工具列上的 [連線]。

    ![連線至虛擬機器](./media/tutorial-use-custom-lab/connect-button.png)
2. 將下載的 **RDP** 檔案儲存在您的硬碟，並用它來連線到虛擬機器。 指定在上一節中建立 VM 時提及的使用者名稱和密碼。 

## <a name="unclaim-the-vm"></a>取消認領 VM
當您使用完 VM 之後，請依照下列步驟取消認領 VM： 

1. 在 [虛擬機器] 頁面的工具列上，選取 [取消認領]。 

    ![取消認領 VM](./media/tutorial-use-custom-lab/unclaim-vm-menu.png)
1. VM 會先關機然後再取消認領。 

    ![取消認領狀態](./media/tutorial-use-custom-lab/unclaim-status.png) 
1. 取消認領作業完成之後，您會看到 VM 出現底部的 [可認領虛擬機器] 清單中。 
    
## <a name="next-steps"></a>後續步驟
本教學課程會示範如何存取和使用透過 Azure DevTest Labs 建立的實驗室。 若要深入了解如何存取和使用實驗室中的 VM，請參閱 

> [!div class="nextstepaction"]
> [作法：在實驗室中使用 VM](devtest-lab-add-vm.md)

