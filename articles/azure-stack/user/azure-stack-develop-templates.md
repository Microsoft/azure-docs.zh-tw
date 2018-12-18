---
title: 開發適用於 Azure Stack 的範本 | Microsoft Docs
description: 了解 Azure Stack 範本最佳做法
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: 8a5bc713-6f51-49c8-aeed-6ced0145e07b
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2018
ms.author: sethm
ms.reviewer: jeffgo
ms.openlocfilehash: d09dec2f327d8b5911a4e55832ba106838c7ebc3
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "41946549"
---
# <a name="azure-resource-manager-template-considerations"></a>Azure Resource Manager 範本考量

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

當您開發應用程式時，請務必確保 Azure 與 Azure Stack 之間的範本可攜性。 本文提供開發 Azure Resource Manager [範本](http://download.microsoft.com/download/E/A/4/EA4017B5-F2ED-449A-897E-BD92E42479CE/Getting_Started_With_Azure_Resource_Manager_white_paper_EN_US.pdf)時必須考量的事項，以便您可以建立原型應用程式，並在 Azure 中測試部署，而不需要存取 Azure Stack 環境。

## <a name="resource-provider-availability"></a>資源提供者可用性

您計畫部署的範本必須僅使用已可用或在 Azure Stack 中為預覽版的 Microsoft Azure 服務。

## <a name="public-namespaces"></a>公用命名空間

因為 Azure Stack 是裝載在您的資料中心，它的服務端點命名空間與 Azure 公用雲端不同。 因此，當您嘗試將 Resource Manager 範本部署到 Azure Stack 時，以硬式編碼方式寫在 Azure Resource Manager 範本中的公用端點將會失敗。 您可以使用 reference 和 concatenate 函式動態建置服務端點，以在部署期間從資源提供者擷取值。 例如，您不需要以硬式編碼方式在您的範本中寫入 *blob.core.windows.net*，而是改為擷取 [primaryEndpoints.blob](https://github.com/Azure/AzureStack-QuickStart-Templates/blob/master/101-simple-windows-vm/azuredeploy.json#L201) 以動態設定 *osDisk.URI* 端點：

     "osDisk": {"name": "osdisk","vhd": {"uri":
     "[concat(reference(concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2015-06-15').primaryEndpoints.blob, variables('vmStorageAccountContainerName'),
      '/',variables('OSDiskName'),'.vhd')]"}}

## <a name="api-versioning"></a>API 版本控制

Azure 服務版本在 Azure 與 Azure Stack 之間可能不同。 每個資源都需要 **apiVersion** 屬性，此屬性會定義提供的功能。 若要確保 Azure Stack 中的 API 版本相容性，下列 API 版本適用於每個資源提供者：

| 資源提供者 | apiVersion |
| --- | --- |
| 計算 |`'2015-06-15'` |
| 網路 |`'2015-06-15'`、`'2015-05-01-preview'` |
| 儲存體 |`'2016-01-01'`、`'2015-06-15'`, `'2015-05-01-preview'` |
| KeyVault | `'2015-06-01'` |
| App Service 方案 |`'2015-08-01'` |

## <a name="template-functions"></a>範本函式

Azure Resource Manager [函式](../../azure-resource-manager/resource-group-template-functions.md)提供建置動態範本所需的功能。 做為範例，您可以針對如下工作使用函式：

* 串連或修剪字串。
* 參考來自其他資源的值。
* 在資源上進行反覆處理以部署多個執行個體。

這些函式在 Azure Stack 中無法使用：

* Skip
* Take

## <a name="resource-location"></a>資源位置

Azure Resource Manager 範本使用位置屬性在部署期間放置資源。 在 Azure 中，位置指的是如美國西部或南美等地區。 在 Azure Stack 中，位置則不同，因為 Azure Stack 位於您的資料中心。 若要確保能在 Azure 與 Azure Stack 之間移轉範本，您應該在部署個別資源時參考資源群組位置。 您可以使用 `[resourceGroup().Location]` 來執行此動作，以確保所有資源皆會繼承資源群組位置。 下列摘要是在部署儲存體帳戶時使用此功能的範例：

    "resources": [
    {
      "name": "[variables('storageAccountName')]",
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "[variables('apiVersionStorage')]",
      "location": "[resourceGroup().location]",
      "comments": "This storage account is used to store the VM disks",
      "properties": {
      "accountType": "Standard_GRS"
      }
    }
    ]

## <a name="next-steps"></a>後續步驟

* [使用 PowerShell 部署範本](azure-stack-deploy-template-powershell.md)
* [使用 Azure CLI 部署範本](azure-stack-deploy-template-command-line.md)
* [使用 Visual Studio 部署範本](azure-stack-deploy-template-visual-studio.md)
