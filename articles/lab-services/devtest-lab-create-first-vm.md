---
title: 在 Azure DevTest Labs 的實驗室中建立您的第一個 VM | Microsoft Docs
description: 了解如何在 Azure DevTest Labs 的實驗室中建立您的第一個虛擬機器
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: fbc5a438-6e02-4952-b654-b8fa7322ae5f
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: 0240bbdf2055ac44f8836adfaad3bf6c44893b77
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33781817"
---
# <a name="create-your-first-vm-in-a-lab-in-azure-devtest-labs"></a>在 Azure DevTest Labs 的實驗室中建立您的第一個 VM

當您首次存取 DevTest Labs 並想要建立您的第一個 VM 時，您很有可能會透過預先載入的[基底 Marketplace 映像](devtest-lab-configure-marketplace-images.md)來完成。 您在稍後建立更多 VM 時，也可以在[自訂映像和公式](devtest-lab-add-vm.md)之間做出選擇。 

本教學課程會逐步引導您使用 Azure 入口網站，在 DevTest Labs 中將您的第一個 VM 新增至實驗室。

## <a name="steps-to-add-your-first-vm-to-a-lab-in-azure-devtest-labs"></a>在 Azure DevTest Labs 中將您的第一個 VM 新增至實驗室的步驟
1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
1. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。
1. 從實驗室清單中，選取您想要在其中建立 VM 的實驗室。  
1. 在實驗室的 [概觀] 刀鋒視窗中，選取 [+ 新增]。  

    ![加入 VM 按鈕](./media/devtest-lab-add-vm/devtestlab-home-blade-add-vm.png)

1. 在 [選擇基底] 刀鋒視窗上，選取 VM 的 Marketplace 映像。
1. 在 [虛擬機器] 刀鋒視窗的 [虛擬機器名稱] 文字方塊中，輸入新虛擬機器的名稱。

    ![實驗室 VM 刀鋒視窗](./media/devtest-lab-add-vm/devtestlab-lab-add-first-vm.png)

1. 輸入**使用者名稱**，此名稱會被授與虛擬機器上的系統管理員權限。  
1. 在標示為 [輸入值] 的文字欄位中輸入密碼。
1. [虛擬機器磁碟類型] 會決定實驗室中的虛擬機器所允許的儲存磁碟類型。
1. 選取 [虛擬機器大小]  ，然後選取其中一個預先定義的項目，這些項目可以指定處理器核心、RAM 大小，以及要建立的 VM 的硬碟大小。
1. 選取 [構件]，然後從構件清單中，選取並設定您想要新增到基本映像中的構件。
    **附註：** 如果您對 DevTest Labs 或設定構件並不熟悉，請參閱[將現有的構件加入至 VM](./devtest-lab-add-vm.md#add-an-existing-artifact-to-a-vm) 一節，完成該節之後再返回此處。
1. 選取 [建立]  ，將指定的 VM 加入實驗室。

   實驗室刀鋒視窗會顯示 VM 的建立狀態，其狀態會先是 [正在建立]，在啟動該 VM 之後才會變成 [正在執行]。

## <a name="next-steps"></a>後續步驟
* 一旦建立 VM 之後，您可以選取 VM 刀鋒視窗上的 [連接]  來連接至 VM。
* 查看[對實驗室新增 VM](devtest-lab-add-vm.md) 以取得在實驗室中新增後續 VM 的完整資訊。
* 瀏覽 [DevTest Labs Azure Resource Manager 快速入門範本資源庫 (英文)](https://github.com/Azure/azure-devtestlab/tree/master/ARMTemplates)。
