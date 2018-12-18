---
title: Azure Resource Manager 個人資料 | Microsoft Docs
description: 了解如何管理與 Azure 資源管理員作業相關聯的個人資料。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/14/2018
ms.author: tomfitz
ms.openlocfilehash: 71928be07080ed14fdcb93f33ea64d2572955b53
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38466019"
---
# <a name="manage-personal-data-associated-with-azure-resource-manager"></a>管理與 Azure 資源管理員相關聯的個人資料

若要避免洩漏敏感資訊，請刪除任何您可能已提供在部署、資源群組或標籤中的個人資訊。 Azure Resource Manager 提供的作業可讓您管理可能已提供在部署、資源群組或標籤中的個人資料。

[!INCLUDE [Handle personal data](../../includes/gdpr-intro-sentence.md)]

## <a name="delete-personal-data-in-deployment-history"></a>刪除部署記錄中的個人資料

針對部署，資源管理員會在部署記錄中保留參數值和狀態訊息。 這些值會保存到您從記錄中刪除部署為止。 若要查看您是否在這些值中提供了個人資料，請列出部署。 如果您發現個人資料，請從記錄中刪除部署。

若要在記錄中列出**部署**，請使用：

* [依資源群組列示](/rest/api/resources/deployments/listbyresourcegroup)
* [Get-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/Get-AzureRmResourceGroupDeployment)
* [az group deployment list](/cli/azure/group/deployment#az-group-deployment-list)

若要從記錄中刪除**部署**，請使用：

* [刪除](/rest/api/resources/deployments/delete)
* [Remove-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/Remove-AzureRmResourceGroupDeployment)
* [az group deployment delete](/cli/azure/group/deployment#az-group-deployment-delete)

## <a name="delete-personal-data-in-resource-group-names"></a>刪除資源群組名稱中的個人資料

資源群組的名稱會保存到您刪除資源群組為止。 若要查看您是否在名稱中提供了個人資料，請列出資源群組。 如果您發現個人資料，請[將資源移到](resource-group-move-resources.md)新的資源群組，並刪除名稱中含有個人資料的資源群組。

若要列出**資源群組**，請使用：

* [清單](/rest/api/resources/resourcegroups/list)
* [Get-AzureRmResourceGroup](/powershell/module/azurerm.resources/Get-AzureRmResourceGroup)
* [az group list](/cli/azure/group#az-group-list)

若要刪除**資源群組**，請使用：

* [刪除](/rest/api/resources/resourcegroups/delete)
* [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/Remove-AzureRmResourceGroup)
* [az group delete](/cli/azure/group#az-group-delete)

## <a name="delete-personal-data-in-tags"></a>在標籤中刪除個人資料

標籤名稱和值會保存到您刪除或修改標籤為止。 若要查看您是否在標籤中提供了個人資料，請列出標籤。 如果您發現個人資料，請刪除標籤。

若要列出**標籤**，請使用：

* [清單](/rest/api/resources/tags/list)
* [Get-AzureRmTag](/powershell/module/azurerm.tags/get-azurermtag)
* [az tag list](/cli/azure/tag#az-tag-list)

若要刪除**標籤**，請使用：

* [刪除](/rest/api/resources/tags/delete)
* [Remove-AzureRmTag](/powershell/module/azurerm.tags/remove-azurermtag)
* [az tag delete](/cli/azure/tag#az-tag-delete)

## <a name="next-steps"></a>後續步驟
* 如需 Azure Resource Manager 的概觀，請參閱[什麼是資源管理員？](resource-group-overview.md)