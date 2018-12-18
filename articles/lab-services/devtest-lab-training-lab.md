---
title: 使用 Azure DevTest Labs 進行訓練 | Microsoft Docs
description: 了解如何針對訓練案例使用 Azure DevTest Labs。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
ms.assetid: 57ff4e30-7e33-453f-9867-e19b3fdb9fe2
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: 85eddaaf101c3e85eca7514b04660163d23c1c80
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38666789"
---
# <a name="use-azure-devtest-labs-for-training"></a>使用 Azure DevTest Labs 進行訓練
除了進行開發/測試，Azure DevTest Labs 還可用來實作許多重要案例。 這些案例的其中之一便是設置訓練實驗室。 Azure DevTest Labs 可讓您建立實驗室，在其中提供自訂範本供每位受訓者建立相同且隔離的訓練環境。 您可以新增原則，以確保訓練環境只會在受訓者需要時才提供給他們使用，且包含足夠的訓練所需資源，例如虛擬機器。 最後，您可以輕易地與受訓者共用實驗室，讓他們只要按一下就能存取。

![使用 DevTest Labs 進行訓練](./media/devtest-lab-training-lab/devtest-lab-training.png)

Azure DevTest Labs 符合下列在任何虛擬環境進行訓練所必須具備的需求︰ 

* 受訓者無法看到其他受訓者所建立的 VM
* 每台訓練用機器應該相同
* 受訓者可以快速佈建其訓練環境
* 確保受訓者無法取得超過訓練所需數量的 VM 以控制成本，並在受訓者不使用 VM 時將其關閉
* 輕易地與每位受訓者共用訓練實驗室
* 一再重複使用訓練實驗室

在本文中，您會了解可用來符合先前所述訓練需求的各種 Azure DevTest Labs 功能，以及可供遵循的訓練實驗室詳細設定步驟。  

## <a name="implementing-training-with-azure-devtest-labs"></a>使用 Azure DevTest Labs 實作訓練
1. **建立實驗室** 
   
    實驗室是 Azure DevTest Labs 的起點。 建立實驗室之後就可以執行各種工作，例如新增使用者 (受訓者) 至實驗室、設定原則來控制成本，以及定義可以快速建立的 VM 映像等。   
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [在 Azure 研測實驗室中建立實驗室](devtest-lab-create-lab.md) |了解如何在 Azure 入口網站的 Azure DevTest Labs 中建立實驗室。 |
2. **使用現成的 Marketplace 映像和自訂映像在幾分鐘內建立訓練 VM** 
   
    您可以從 Azure Marketplace 的各種映像中挑選現成的映像，並在實驗室中提供給受訓者使用。 如果現成映像不符合您的需求，您可以透過使用 Azure Marketplace 中的現成映像建立實驗室 VM、安裝訓練所需的所有軟體，並將該 VM 儲存為實驗室中的自訂映像，以建立自訂映像。 
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [設定 Azure Marketplace 映像](devtest-lab-configure-marketplace-images.md) |了解如何將 Azure Marketplace 映像加入白名單；只讓想要用於進行訓練的映像可供選取。 |
   | [建立自訂映像](devtest-lab-create-template.md) |預先安裝訓練所需軟體以建立自訂映像，讓受訓者可以使用自訂映像快速建立 VM。 |
3. **建立可重複用於訓練機器的範本** 
   
    Azure DevTest Labs 中的公式是用來建立 VM 的預設屬性值清單。 您可以在實驗室中挑選映像、VM 大小 (CPU 和 RAM 的組合) 和虛擬網路以建立公式。 每位受訓者都能查看實驗室中的公式，並使用它來建立 VM。 
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [管理研發/測試實驗室公式來建立 VM](devtest-lab-manage-formulas.md) |了解如何挑選映像、VM 大小 (CPU 和 RAM 的組合) 和虛擬網路以建立公式。 |
4. **控制成本**
   
    Azure DevTest Labs 可讓您在實驗室中設定原則，以指定受訓者可在實驗室中建立的 VM 數目上限。 
   
    如果您在進行為期數天的訓練，並且想要在一天當中的特定時間停止所有 VM，然後在隔天自動重新啟動，您可以藉由設定實驗室中的自動關閉和自動啟動原則來輕鬆達到該目的。 
   
    最後，當訓練完成時，您可以執行單一 PowerShell 指令碼一次刪除所有 VM。 
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [定義實驗室原則](devtest-lab-set-lab-policy.md) |在實驗室中設定原則來控制成本。 |
   | [使用 PowerShell 指令碼刪除所有實驗室 VM](devtest-lab-faq.md#how-do-i-automate-the-process-of-deleting-all-the-vms-in-my-lab) |當訓練完成時，在一次作業中刪除所有實驗室。 |
5. **與每位受訓者共用實驗室**
   
    使用您分享給受訓者的連結即可直接存取實驗室。 受訓者甚至不必有 Azure 帳戶，只要他們擁有 [Microsoft 帳戶](devtest-lab-faq.md#what-is-a-microsoft-account)。 受訓者無法看到其他受訓者所建立的 VM。  
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [在 Azure DevTest Labs 中新增受訓者到實驗室](devtest-lab-add-devtest-user.md) |使用 Azure 入口網站將受訓者新增到訓練實驗室。 |
   | [使用 PowerShell 指令碼新增受訓者到實驗室](devtest-lab-add-devtest-user.md#add-an-external-user-to-a-lab-using-powershell) |使用 PowerShell 自動將受訓者新增到訓練實驗室。 |
   | [取得實驗室連結](devtest-lab-faq.md#how-do-i-share-a-direct-link-to-my-lab) |了解如何透過超連結直接存取實驗室。 |
6. **一再重複使用實驗室** 
   
    您可以建立 Resource Manager 範本並一再使用它來建立相同的實驗室，以自動建立實驗室，包括自訂設定。 
   
    按一下下表中的連結以便深入了解︰
   
   | Task | 您學到什麼 |
   | --- | --- |
   | [使用 Resource Manager 範本建立實驗室](devtest-lab-faq.md#how-do-i-create-a-lab-from-a-resource-manager-template) |在 Azure DevTest Labs 中使用 Resource Manager 範本建立實驗室。 |

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

