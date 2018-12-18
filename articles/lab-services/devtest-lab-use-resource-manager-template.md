---
title: 檢視及使用虛擬機器的 Azure Resource Manager 範本 | Microsoft Docs
description: 了解如何從虛擬機器使用 Azure Resource Manager 範本來建立其他 VM
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: a759d9ce-655c-4ac6-8834-cb29dd7d30dd
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: 1b96c801e3c5a43f23d4c63c13a611c24dd6705b
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783437"
---
# <a name="create-virtual-machines-using-an-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本建立虛擬機器 

透過 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)在 DevTest Labs 中建立虛擬機器 (VM) 時，您可以在儲存 VM 之前檢視 Azure Resource Manager 範本。 然後可以使用範本作為基礎來建立具有相同設定的多個實驗室 VM。

本文說明多部 VM 與單一 VM Resource Manager 範本，並示範如何在建立 VM 時檢視和儲存範本。

## <a name="multi-vm-vs-single-vm-resource-manager-templates"></a>多部 VM 與單一 VM Resource Manager 範本
有兩種方式可以在使用Resource Manager 範本，在 DevTest Labs 中建立 VM：佈建 Microsoft.DevTestLab/labs/virtualmachines 資源或佈建 Microsoft.Commpute/virtualmachines 資源。 每種方式都在不同案例中使用，而且需要不同的權限。

- 使用 Microsoft.DevTestLab/labs/virtualmachines 資源類型 (在範本的 “resource” 屬性中宣告) 的 Resource Manager 範本可以佈建個別實驗室 VM。 然後每個 VM 會顯示為 DevTest Labs 虛擬機器清單中的單一項目：

   ![DevTest Labs 虛擬機器清單中單一項目之 VM 的清單](./media/devtest-lab-use-arm-template/devtestlab-lab-vm-single-item.png)

   這個類型的 Resource Manager 範本可以透過 Azure PowerShell 命令 **New-AzureRmResourceGroupDeployment**，或透過 Azure CLI 命令 **az group deployment create** 進行佈建。 它需要系統管理員權限，所以獲得 DevTest Labs 使用者角色指派的使用者無法執行部署。 

- 使用 Microsoft.Compute/virtualmachines 資源類型的 Resource Manager 範本可以將多個 VM 佈建為 DevTest Labs 虛擬機器清單中的單一環境：

   ![DevTest Labs 虛擬機器清單中單一項目之 VM 的清單](./media/devtest-lab-use-arm-template/devtestlab-lab-vm-single-environment.png)

   在相同環境中的 VM 可以一起管理，並且共用相同生命週期。 只要系統管理員已對實驗室進行相關設定，獲得 DevTest Labs 使用者角色指派的使用者就可以使用這些範本來建立環境。

本文的其餘部分討論使用 Mirosoft.DevTestLab/labs/virtualmachines 的 Resource Manager 範本。 這些範本是由實驗室管理員用來自動化實驗室 VM 建立 (例如，可宣告 VM) 或黃金映像產生 (例如，映像工廠)。

[建立 Azure Resource Manager 範本的最佳做法](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-template-best-practices)提供許多指導方針和建議，可以協助您建立可靠又容易使用的 Azure Resource Manager 範本。

## <a name="view-and-save-a-virtual-machines-resource-manager-template"></a>檢視及儲存虛擬機器的 Resource Manager 範本
1. 請依照[在實驗室中建立您的第一個 VM](devtest-lab-create-first-vm.md) 中的步驟，開始建立虛擬機器。
1. 輸入虛擬機器的必要資訊，並且新增您想要用於此 VM 的任何成品。
1. 在 [設定] 設定視窗底部，選擇 [檢視 ARM 範本]。

   ![檢視 ARM 範本按鈕](./media/devtest-lab-use-arm-template/devtestlab-lab-view-rm-template.png)
1. 複製及儲存 Resource Manager 範本，以便稍後用來建立另一個虛擬機器。

   ![儲存以供稍後使用的 Resource Manager 範本](./media/devtest-lab-use-arm-template/devtestlab-lab-copy-rm-template.png)

儲存 Resource Manager 範本之後，您必須更新範本的參數區段，才能使用它。 您可以建立 parameter.json，它只會自訂實際 Resource Manager 範本外部的參數。 

![使用 JSON 檔案自訂參數](./media/devtest-lab-use-arm-template/devtestlab-lab-custom-params.png)

Resource Manager 範本現在可用來[建立 VM](devtest-lab-create-environment-from-arm.md)。

### <a name="next-steps"></a>後續步驟
* 了解如何[使用 Resource Manager 範本建立多個 VM 環境](devtest-lab-create-environment-from-arm.md)。
* [部署 Resource Manager 範本以建立 VM](devtest-lab-create-environment-from-arm.md#deploy-a-resource-manager-template-to-create-a-vm)
* 從[公用 DevTest Labs GitHub 存放庫](https://github.com/Azure/azure-quickstart-templates) \(英文\) 探索更多 DevTest Labs 自動化的快速入門 Resource Manager 範本。
