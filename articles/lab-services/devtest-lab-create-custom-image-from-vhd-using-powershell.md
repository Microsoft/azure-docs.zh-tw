---
title: 使用 PowerShell 從 VHD 檔案建立 Azure DevTest Labs 自訂映像 | Microsoft Docs
description: 使用 PowerShell 在 Azure DevTest Labs 中從 VHD 檔案自動建立自訂映像
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/05/2018
ms.author: spelluru
ms.openlocfilehash: 17679ee3a5cb50f78cad0f66cb3fffcc6d556087
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33781757"
---
# <a name="create-a-custom-image-from-a-vhd-file-using-powershell"></a>使用 PowerShell 從 VHD 檔案建立自訂映像

[!INCLUDE [devtest-lab-create-custom-image-from-vhd-selector](../../includes/devtest-lab-create-custom-image-from-vhd-selector.md)]

[!INCLUDE [devtest-lab-custom-image-definition](../../includes/devtest-lab-custom-image-definition.md)]

[!INCLUDE [devtest-lab-upload-vhd-options](../../includes/devtest-lab-upload-vhd-options.md)]

## <a name="step-by-step-instructions"></a>逐步指示

下列步驟將逐步引導您使用 PowerShell 從 VHD 檔案建立自訂映像：

1. 在 PowerShell 提示字元中，使用下列對 **Connect-AzureRmAccount** Cmdlet 的呼叫來登入您的 Azure 帳戶。  
    
    ```PowerShell
    Connect-AzureRmAccount
    ```

1.  呼叫 **Select-AzureRmSubscription** Cmdlet 來選取想要的 Azure 訂用帳戶。 以有效的 Azure 訂用帳戶 ID 取代 **$subscriptionId** 變數的下列預留位置。 

    ```PowerShell
    $subscriptionId = '<Specify your subscription ID here>'
    Select-AzureRmSubscription -SubscriptionId $subscriptionId
    ```

1.  呼叫 **Get-AzureRmResource** Cmdlet 來取得實驗室物件。 以您環境的適當值取代 **$labRg** 和 **$labName** 變數的下列預留位置。 

    ```PowerShell
    $labRg = '<Specify your lab resource group name here>'
    $labName = '<Specify your lab name here>'
    $lab = Get-AzureRmResource -ResourceId ('/subscriptions/' + $subscriptionId + '/resourceGroups/' + $labRg + '/providers/Microsoft.DevTestLab/labs/' + $labName)
    ```
 
1.  從實驗室物件取得實驗室儲存體帳戶和實驗室儲存體帳戶金鑰值。 

    ```PowerShell
    $labStorageAccount = Get-AzureRmResource -ResourceId $lab.Properties.defaultStorageAccount 
    $labStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $labStorageAccount.ResourceGroupName -Name $labStorageAccount.ResourceName)[0].Value
    ```

1.  以您所上傳 VHD 檔案的 URI 取代 **$vhdUri** 變數的下列預留位置。 您可以從 Azure 入口網站中儲存體帳戶的 Blob 刀鋒視窗取得 VHD 檔案的 URI。

    ```PowerShell
    $vhdUri = '<Specify the VHD URI here>'
    ```

1.  使用 **New-AzureRmResourceGroupDeployment** Cmdlet 來建立自訂映像。 將 **$customImageName** 和 **$customImageDescription** 變數的下列預留位置取代為對您環境有意義的名稱。

    ```PowerShell
    $customImageName = '<Specify the custom image name>'
    $customImageDescription = '<Specify the custom image description>'

    $parameters = @{existingLabName="$($lab.Name)"; existingVhdUri=$vhdUri; imageOsType='windows'; isVhdSysPrepped=$false; imageName=$customImageName; imageDescription=$customImageDescription}

    New-AzureRmResourceGroupDeployment -ResourceGroupName $lab.ResourceGroupName -Name CreateCustomImage -TemplateUri 'https://raw.githubusercontent.com/Azure/azure-devtestlab/master/Samples/201-dtl-create-customimage-from-vhd/azuredeploy.json' -TemplateParameterObject $parameters
    ```

## <a name="powershell-script-to-create-a-custom-image-from-a-vhd-file"></a>可從 VHD 檔案建立自訂映像的 PowerShell 指令碼

您可以使用下列 PowerShell 指令碼從 VHD 檔案建立自訂映像。 以符合您需求的適當值取代預留位置 (開頭和結尾是角括弧)。 

```PowerShell
# Log in to your Azure account.  
Connect-AzureRmAccount

# Select the desired Azure subscription. 
$subscriptionId = '<Specify your subscription ID here>'
Select-AzureRmSubscription -SubscriptionId $subscriptionId

# Get the lab object.
$labRg = '<Specify your lab resource group name here>'
$labName = '<Specify your lab name here>'
$lab = Get-AzureRmResource -ResourceId ('/subscriptions/' + $subscriptionId + '/resourceGroups/' + $labRg + '/providers/Microsoft.DevTestLab/labs/' + $labName)

# Get the lab storage account and lab storage account key values.
$labStorageAccount = Get-AzureRmResource -ResourceId $lab.Properties.defaultStorageAccount 
$labStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $labStorageAccount.ResourceGroupName -Name $labStorageAccount.ResourceName)[0].Value

# Set the URI of the VHD file.  
$vhdUri = '<Specify the VHD URI here>'

# Set the custom image name and description values.
$customImageName = '<Specify the custom image name>'
$customImageDescription = '<Specify the custom image description>'

# Set up the parameters object.
$parameters = @{existingLabName="$($lab.Name)"; existingVhdUri=$vhdUri; imageOsType='windows'; isVhdSysPrepped=$false; imageName=$customImageName; imageDescription=$customImageDescription}

# Create the custom image. 
New-AzureRmResourceGroupDeployment -ResourceGroupName $lab.ResourceGroupName -Name CreateCustomImage -TemplateUri 'https://raw.githubusercontent.com/Azure/azure-devtestlab/master/Samples/201-dtl-create-customimage-from-vhd/azuredeploy.json' -TemplateParameterObject $parameters
```

## <a name="related-blog-posts"></a>相關部落格文章

- [自訂映像或公式？](https://blogs.msdn.microsoft.com/devtestlab/2016/04/06/custom-images-or-formulas/)
- [在 Azure DevTest Labs 之間複製自訂映像](http://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs#copying-custom-images-between-azure-devtest-labs)

## <a name="next-steps"></a>後續步驟

- [將 VM 新增到實驗室](devtest-lab-add-vm.md)
