---
title: 在 Azure DevTest Labs 中設定 Azure Marketplace 映像設定 | Microsoft Docs
description: 設定在 Azure DevTest Labs 中建立 VM 時可以使用哪些 Azure Marketplace 映像
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 804c6af2-17e9-4320-af3a-f454bd398379
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/05/2018
ms.author: spelluru
ms.openlocfilehash: 90df853eeca89e61fa69e6967218c4c18274ac1b
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38633031"
---
# <a name="configure-azure-marketplace-image-settings-in-azure-devtest-labs"></a>在 Azure DevTest Labs 中設定 Azure Marketplace 映像設定
DevTest Labs 支援根據 Azure Marketplace 映像來建立 VM，而這取決於您設定 Azure Marketplace 映像使用於實驗室的方式。 本文將說明在實驗室中建立 VM 時，如何指定可以使用哪些 Azure Marketplace 映像 (如果有的話)。 這可確保您的小組只能存取他們所需的 Marketplace 映像。 

## <a name="select-which-azure-marketplace-images-are-allowed-when-creating-a-vm"></a>選取在建立 VM 時允許使用哪些 Azure Marketplace 映像
1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
2. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。
3. 從實驗室清單中，選取所需的實驗室。 
4. 在實驗室的刀鋒視窗上，選取 [組態和原則]。
5. 在實驗室的 [組態和原則] 刀鋒視窗的 [Virtual Machine Bases] \(虛擬機器基礎) 下，選取 [Marketplace 映像]。
6. 指定是否要讓所有合格的 Azure Marketplace 映像可用來做為新 VM 的基底。 如果您選取 [是] ，則實驗室中允許所有符合下列準則的 Azure Marketplace 映像︰
   
   * 映像會建立單一 VM， **而且**
   * 映像會使用 Azure Resource Manager 來佈建 VM， **而且**
   * 映像不需要購買額外的授權方案
     
    如果您不想允許任何映像，或者想要指定可以使用哪些映像，請選取 [否] 。
     
     ![允許使用所有 Marketplace 映像做為 VM 基底映像的選項](./media/devtest-lab-configure-marketplace-images/allow-all-marketplace-images.png)
7. 如果您在上一個步驟中選取 [否]，將會啟用 [允許的映像/全選] 核取方塊。 
   您可以將此選項與搜尋方塊一起使用，快速選取或取消選取清單中顯示的所有項目。
   * 藉由核取每個映像的對應核取方塊，個別選取您想要允許來建立 VM 的 Azure Marketplace 映像。
   * 如果您不想在實驗室中允許使用任何 Azure Marketplace 映像，請不要從清單中選取任何項目。
   
    ![您可以指定可使用哪些 Marketplace 映像做為 VM 的基底映像](./media/devtest-lab-configure-marketplace-images/select-marketplace-images.png)

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>後續步驟
一旦您設定在建立 VM 時允許使用 Azure Marketplace 映像的方式之後，下一個步驟就是 [將 VM 新增至您的實驗室](devtest-lab-add-vm.md)。

