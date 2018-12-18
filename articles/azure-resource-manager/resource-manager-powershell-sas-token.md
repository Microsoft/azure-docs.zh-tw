---
title: 使用 SAS 權杖與 PowerShell 部署 Azure 範本 | Microsoft Docs
description: 使用 Azure Resource Manager 和 Azure PowerShell，將受 SAS 權杖保護之範本中的資源部署到 Azure。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: ''
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/19/2017
ms.author: tomfitz
ms.openlocfilehash: f138cceb88cb9a43efdd3f11b24203378a288286
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34602974"
---
# <a name="deploy-private-resource-manager-template-with-sas-token-and-azure-powershell"></a>使用 SAS 權杖和 Azure PowerShell 部署私用 Resource Manager 範本

當您的範本位於儲存體帳戶中時，您可以限制範本的存取權，並在部署期間提供共用存取簽章 (SAS) Token 本主題說明如何在部署期間使用 Azure PowerShell 與 Resource Manager 範本提供 SAS 權杖。 

## <a name="add-private-template-to-storage-account"></a>將私人範本新增至儲存體帳戶

您可以將範本加入儲存體帳戶，並在部署期間使用 SAS Token 連結它們。

> [!IMPORTANT]
> 遵循下列步驟，則僅有帳戶擁有者可以存取包含範本的 Blob。 不過，當您建立 Blob 的 SAS Token 時，具備該 URI 的任何人都可以存取該 Blob。 如果另一位使用者攔截了 URI，該使用者也能存取範本。 使用 SAS Token 是限制存取您的範本的好方法，但您不應該將機密資料 (如密碼) 直接包含在範本中。
> 
> 

下列範例會設定私用儲存體帳戶容器並上傳範本︰
   
```powershell
# create a storage account for templates
New-AzureRmResourceGroup -Name ManageGroup -Location "South Central US"
New-AzureRmStorageAccount -ResourceGroupName ManageGroup -Name {your-unique-name} -Type Standard_LRS -Location "West US"
Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name {your-unique-name}

# create a container and upload template
New-AzureStorageContainer -Name templates -Permission Off
Set-AzureStorageBlobContent -Container templates -File c:\MyTemplates\storage.json
```

## <a name="provide-sas-token-during-deployment"></a>在部署期間提供 SAS Token
若要在儲存體帳戶中部署私人範本，請產生 SAS Token 並將它包含在範本的 URI 中。 設定到期時間，以允許足夠的時間來完成部署。
   
```powershell
Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name {your-unique-name}

# get the URI with the SAS token
$templateuri = New-AzureStorageBlobSASToken -Container templates -Blob storage.json -Permission r `
  -ExpiryTime (Get-Date).AddHours(2.0) -FullUri

# provide URI with SAS token during deployment
New-AzureRmResourceGroup -Name ExampleGroup -Location "South Central US"
New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateUri $templateuri
```

如需使用包含已連結範本的 SAS Token 範例，請參閱 [透過 Azure Resource Manager 使用連結的範本](resource-group-linked-templates.md)。


## <a name="next-steps"></a>後續步驟
* 如需部署範本的簡介，請參閱[使用 Resource Manager 範本與 Azure PowerShell 來部署資源](resource-group-template-deploy.md)。
* 如需部署範本的完整範例指令碼，請參閱[部署 Resource Manager 範本指令碼](resource-manager-samples-powershell-deploy.md)。
* 若要在範本中定義參數，請參閱 [編寫範本](resource-group-authoring-templates.md#parameters)。
