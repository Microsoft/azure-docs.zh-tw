---
title: 在 Azure DevTest Labs 中調整實驗室中 VM 的大小 | Microsoft Docs
description: 了解如何在 Azure DevTest Labs 中調整虛擬機器的大小
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 8460f09e-482f-48ba-a57a-c95fe8afa001
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/11/2018
ms.author: spelluru
ms.openlocfilehash: 9b9a1839bf4b028aec13b764b4de66385de4189e
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47090746"
---
# <a name="resize-a-vm-in-a-lab-in-azure-devtest-labs"></a>在 Azure DevTest Labs 中調整實驗室中 VM 的大小
Azure 虛擬機器的其中一個重要功能就是可讓您根據您的 CPU、網路或磁碟效能需求，變更虛擬機器 (VM) 的大小。 Azure DevTest Labs 現在可針對實驗室中的 VM 支援此功能。 調整大小功能會遵守適用於實驗室所允許 VM 大小的實驗室原則。 也就是說，您只能在實驗室所允許的大小範圍內變更 VM 的大小。 


## <a name="steps-to-resize-a-vm-in-a-lab"></a>調整實驗室中 VM 大小的步驟 
若要在 Azure DevTest Labs 中調整實驗室中 VM 的大小，請進行下列步驟： 

> [!NOTE]
> 如果您是透過遠端桌面工作階段 (RDP) 連線至 VM，請儲存您的工作，然後先與 VM 中斷連線，再調整 VM 大小。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。
3. 從實驗室清單中，選取包含您想要調整大小之 VM 的實驗室。  
4. 在左面板中，選取 [我的虛擬機器]。 
5. 從 VM 清單中，選取 VM。
6. 如果 VM 正在執行，請選取工具列上的 [停止]。 查看 [通知] 視窗中作業的狀態。 等候直到 VM 停止，然後關閉 [通知] 視窗。 

    ![停止 VM](media/devtest-lab-resize-vm/stop-vm.png)
1. 在您 VM 的 [虛擬機器] 頁面中，選取左側功能表中 [設定] 底下的 [大小]。

    ![[大小] 功能表](media/devtest-lab-resize-vm/size-menu.png)
1. 在 [選擇大小] 視窗中，為您的 VM 瀏覽並選取大小，然後按一下 [選取]。     
1. 查看 [通知] 視窗中調整大小作業的狀態。

    ![調整大小狀態](media/devtest-lab-resize-vm/resize-status.png)
10. 在調整大小作業成功之後，關閉 [通知] 視窗。 
11. 選取左側功能表中的 [概觀]，然後選取工具列上的 [重新啟動] 以重新啟動 VM。 

## <a name="next-steps"></a>後續步驟
如需有關 Azure 虛擬機器所支援調整大小功能的詳細資訊，請參閱[調整虛擬機器的大小](https://azure.microsoft.com/blog/resize-virtual-machines/) \(英文\)。


