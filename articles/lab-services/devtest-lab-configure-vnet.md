---
title: 設定 Azure DevTest Labs 中的虛擬網路 | Microsoft Docs
description: 了解如何設定現有的虛擬網路和子網路，並在具備 Azure DevTest Labs 的 VM 中使用它們
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 6cda99c2-b87e-4047-90a0-5df10d8e9e14
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/05/2018
ms.author: spelluru
ms.openlocfilehash: 0141ea8a88c0322e6f56cbea56d3a43c923769af
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38687783"
---
# <a name="configure-a-virtual-network-in-azure-devtest-labs"></a>設定 Azure DevTest Labs 中的虛擬網路
如[將 VM 加入實驗室](devtest-lab-add-vm.md)一文中所述，當您在實驗室中建立 VM 時，可以指定已設定的虛擬網路。 例如，您可能需要使用以 ExpressRoute 或站台對站台 VPN 設定的虛擬網路從您的 VM 存取公司資源。

本文說明如何將現有的虛擬網路加入至實驗室的虛擬網路設定，以在建立 VM 時選擇它。

## <a name="configure-a-virtual-network-for-a-lab-using-the-azure-portal"></a>使用 Azure 入口網站設定適用於實驗室的虛擬網路
下列步驟會逐步引導您將現有虛擬網路 (和子網路) 加入至實驗室，以便在同一個實驗室中建立 VM 時加以使用。 

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
1. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。
1. 從實驗室清單中，選取所需的實驗室。 
1. 在實驗室的主窗格上，選取 [設定與原則]。

    ![存取實驗室的設定與原則](./media/devtest-lab-configure-vnet/policies-menu.png)
1. 在 [外部資源] 區段中，選取 [虛擬網路]。 隨即顯示針對目前實驗室設定的虛擬網路清單，以及為實驗室所建立的預設虛擬網路。 
1. 選取 [+ 新增] 。
   
    ![將現有的虛擬網路加入至您的實驗室](./media/devtest-lab-configure-vnet/lab-settings-vnet-add.png)
1. 在 [虛擬網路] 窗格上，選取 [選取虛擬網路]。
   
    ![選取現有的虛擬網路](./media/devtest-lab-configure-vnet/lab-settings-vnets-vnet1.png)
1. 在 [選擇虛擬網路] 窗格上，選取所需的虛擬網路。 隨即顯示訂用帳戶中與實驗室位於相同區域下的所有虛擬網路。
1. 選取虛擬網路之後，您會回到 [虛擬網路] 窗格。 選取底端清單中的子網路。

    ![子網路清單](./media/devtest-lab-configure-vnet/lab-settings-vnets-vnet2.png)
    
    隨即顯示 [實驗室的子網路] 窗格。

    ![[實驗室的子網路] 窗格](./media/devtest-lab-configure-vnet/lab-subnet.png)
     
   - 指定 [實驗室子網路名稱]。
   - 若要允許在實驗室 VM 建立期間使用子網路，請選取 [在虛擬機器建立時使用]。
   - 若要啟用[共用公用 IP 位址](devtest-lab-shared-ip.md)，請選取 [啟用共用公用 IP]。
   - 若要允許子網路中使用公用 IP 位址，請選取 [允許建立公用 IP] 。
   - 在 [每位使用者的最大虛擬機器數] 欄位中，針對每個子網路指定每位使用者的最大 VM 數。 如果您想要不限數目的 VM 數，請將此欄位保留空白。
1. 選取 [確定] 以關閉 [實驗室的子網路] 窗格。
1. 選取 [儲存] 以關閉 [虛擬網路] 窗格。

現在已設定虛擬網路，在建立 VM 時就能選取它。 若要查看如何建立虛擬機器並指定虛擬網路，請參閱[將虛擬機器加入實驗室](devtest-lab-add-vm.md)一文。 

Azure 的[虛擬網路文件](https://docs.microsoft.com/azure/virtual-network)會提供更多有關 VNet 使用方法的資訊，包括如何設定及管理 VNet，以及如何將它連線到您的內部部署網路。

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>後續步驟
一旦您在實驗室中加入所需的虛擬網路之後，下一個步驟就是 [將 VM 加入至實驗室](devtest-lab-add-vm.md)。

