---
title: 在 Azure DevTest Labs 中從另一個實驗室匯入 VM | Microsoft Docs
description: 了解如何在 Azure DevTest Labs 中將虛擬機器從另一個實驗室匯入到目前的實驗室
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
ms.openlocfilehash: 4585d151e286917c67586a02539a10ade32bdd4c
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47094512"
---
# <a name="import-vms-from-another-lab-in-azure-devtest-labs"></a>在 Azure DevTest Labs 中從另一個實驗室匯入 VM
Azure DevTest Labs 服務大幅改善開發和測試活動的虛擬機器 (VM) 管理。 它可讓您隨著小組或基礎結構需求的變更，將 VM 從一個實驗室移至另一個實驗室。 以下是一些您可能需要這麼做的常見案例： 

- 小組中的某個人員移至企業內的另一個群組，而想要將開發 VM 帶到新小組的實驗室。
- 群組已觸達訂用帳戶層級配額，而想要將小組分割成多個訂用帳戶。
- 公司打算移轉至 Express Route (或某個其他新的網路技術)，而小組想要將 VM 移轉成使用此新基礎結構。

## <a name="solution-and-constraints"></a>解決方案和條件約束
Azure DevTest Labs 可讓實驗室擁有者將來源實驗室中的 VM 匯入至目的地實驗室。 實驗室擁有者可以在此程序中，視需要為目的地 VM 提供新名稱。 此匯入程序會包含所有相依性，例如磁碟、排程、網路設定等。此程序需要花費一些時間，並且會受到一些因素影響，例如連結至來源機器的磁碟數目/大小及與目的地的距離 (例如「美國東部」區域與「東南亞」區域)。 匯入程序完成之後，來源 VM 會保持關閉，而新 VM 則會在目的地實驗室中處於執行中狀態。

打算將 VM 匯入至另一個實驗室時，有兩個需要注意的重要條件約束：

- 支援跨訂用帳戶和跨區域匯入 VM，但訂用帳戶必須與相同的 Azure Active Directory 租用戶相關。
- VM 在來源實驗室中不得處於可認領狀態。

此外，若要能夠將 VM 從一個實驗室匯入至另一個實驗室，您必須在來源實驗室中是該 VM 的擁有者，並且在目的地實驗室中是該實驗室的擁有者。

## <a name="steps-to-import-a-vm-from-another-lab"></a>從另一個實驗室匯入 VM 的步驟
目前，您只能使用 Azure PowerShell 和 REST API 將 VM 從一個實驗室匯入至另一個實驗室。

### <a name="use-powershell"></a>使用 PowerShell
請從 [Azure DevTest Lab Git 存放庫](https://github.com/Azure/azure-devtestlab/tree/master/Scripts/ImportVirtualMachines) 將 PowerShell 指令檔 ImportVirtualMachines.ps1 下載到您的本機磁碟機。 

#### <a name="import-a-single-vm"></a>匯入單一 VM
請執行 ImportVirtualMachines.ps1 指令碼以將單一 VM 從來源實驗室匯入至目的地實驗室。 您可以使用 DestinationVirtualMachineName 參數為所要複製的 VM 指定新名稱。 

```powershell
./ImportVirtualMachines.ps1 -SourceSubscriptionId "<ID of the subscription that contains the source VM>" `
                            -SourceDevTestLabName "<Name of the lab that contains the source VM>" `
                            -SourceVirtualMachineName "<Name of the machine. Optional. If not specified, all VMs are copied>" `
                            -DestinationSubscriptionId "<ID of the target/destination subscription>" `
                            -DestinationDevTestLabName "<Name of the lab to which the VM is copied>" `
                            -DestinationVirtualMachineName "<New name for the VM. Optional>"
```


#### <a name="importing-all-vms"></a>匯入所有 VM
執行 ImportVirtualMachines.ps1 指令碼時，如果您未指定來源實驗室中的 VM，指令碼就會將來源實驗室中的所有 VM 都匯入至目的地實驗室。 

```powershell
./ImportVirtualMachines.ps1 -SourceSubscriptionId "<ID of the subscription that contains the source VM>" `
                            -SourceDevTestLabName "<Name of the lab that contains the source VM>" `
                            -DestinationSubscriptionId "<ID of the target/destination subscription>" `
                            -DestinationDevTestLabName "<Name of the lab to which the VMs are copied>"
```

### <a name="use-rest-api"></a>使用 REST API
請針對目標/目的地實驗室叫用 REST API，然後以參數形式傳入來源實驗室、訂用帳戶及 VM 資訊，如以下範例所示： 

```json
POST https://management.azure.com/subscriptions/<ID of the target/destination subscription>/resourceGroups/<Name of the resource group that contains the destination lab>/providers/Microsoft.DevTestLab/labs/<Name of the lab to which the VMs are copied>/ImportVirtualMachine?api-version=2017-04-26-preview
{
   sourceVirtualMachineResourceId: "/subscriptions/<ID of the subscription that contains the source VM>/resourcegroups/<Name of the resource group that contains the source lab>/providers/microsoft.devtestlab/labs/<Name of the lab that contains the source VM>/virtualmachines/MyVm",
   destinationVirtualMachineName: "MyVmNew"
}
```

## <a name="next-steps"></a>後續步驟

- 如需有關調整 VM 大小的資訊，請參閱[調整 VM 的大小](devtest-lab-resize-vm.md)。
- 如需有關重新部署 VM 的資訊，請參閱[重新部署 VM](devtest-lab-redeploy-vm.md)。


