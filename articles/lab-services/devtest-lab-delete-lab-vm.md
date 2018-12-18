---
title: 在 Azure DevTest Labs 的實驗室中刪除實驗室或 VM | Microsoft Docs
description: 本文章說明如何在實驗室中刪除實驗室或 VM。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/30/2018
ms.author: spelluru
ms.openlocfilehash: 99caf04698226de8daa9cfb8f60662e5cb0f8b49
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39450388"
---
# <a name="delete-a-lab-or-vm-in-a-lab-in-azure-devtest-labs"></a>在 Azure DevTest Labs 的實驗室中刪除實驗室或 VM
本文章說明如何在實驗室中刪除實驗室或 VM。

## <a name="delete-a-lab"></a>刪除實驗室
當您從資源群組中刪除 DevTest Labs 執行個體時，DevTest Labs 服務就會執行下列動作： 

- 系統會自動刪除在建立實驗室時自動建立的所有資源。 不會刪除資源群組本身。 如果您手動建立此資源群組的任何資源，則服務不會刪除它們。 
- 系統會自動刪除實驗室中的所有 VM 以及與這些 VM 相關聯的資源群組。 當您在實驗室中建立 VM 時，服務會在個別資源群組中為此 VM 建立資源 (磁碟、網路介面、公用 IP 位址等)。 不過，如果您在這些資源群組中手動建立任何額外的資源，則 DevTest Labs 服務不會刪除這些資源和資源群組。 

若要刪除實驗室，請執行下列動作： 

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 從左側功能表中選取 [所有資源]，選取 [DevTest Labs] 作為類型服務，然後選取實驗室。

    ![選取您的實驗室](media\devtest-lab-delete-lab-vm\select-lab.png)
3. 在 [DevTest Lab] 頁面上，按一下工具列上的 [刪除]。 

    ![刪除按鈕](media\devtest-lab-delete-lab-vm\delete-button.png)
4. 在 [確認] 頁面上，輸入您實驗室的**名稱**，然後選取 [刪除]。 

    ![確認](media\devtest-lab-delete-lab-vm\confirm-delete.png)
5. 若要查看作業狀態，請選取 [通知] 圖示 (鈴鐺)。 

    ![通知](media\devtest-lab-delete-lab-vm\delete-status.png)

 
## <a name="delete-a-vm-in-a-lab"></a>在實驗室中刪除 VM
如果刪除實驗室中的 VM，則會刪除在建立實驗室時所建立的一些 (並非全部) 資源。 不會刪除下列資源： 

-   主要資源群組中的金鑰保存庫
-   VM 資源群組中的可用性設定組、負載平衡器、公用 IP 位址。 這些資源會由資源群組中的多部 VM 共用。 

與 VM 相關聯的虛擬機器、網路介面和磁碟都會遭到刪除。 

若要刪除實驗室中的 VM，請執行下列動作： 

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 從左側功能表中選取 [所有資源]，選取 [DevTest Labs] 作為類型服務，然後選取實驗室。

    ![選取您的實驗室](media\devtest-lab-delete-lab-vm\select-lab.png)
3. 在 VM 清單中選取 VM 的 **... (省略符號)**，然後選取 [刪除]。 

    ![在功能表中刪除 VM](media\devtest-lab-delete-lab-vm\delete-vm-menu-in-list.png)
4. 在**確認**對話方塊上，選取 [確定]。 
5. 若要查看作業狀態，請選取 [通知] 圖示 (鈴鐺)。 

若要從 [虛擬機器] 頁面中刪除 VM，請選取工具列中的 [刪除]，如下圖所示：

![從 VM 頁面刪除 VM](media\devtest-lab-delete-lab-vm\delete-from-vm-page.png) 


## <a name="next-steps"></a>後續步驟
如果您想建立實驗室，請參閱下列文章︰ 

- [建立實驗室](devtest-lab-create-lab.md)
- [將 VM 新增至實驗室](devtest-lab-add-vm.md)