---
title: 使用 SAS 權杖和 Azure CLI 部署 Azure 範本 | Microsoft Docs
description: 使用 Azure Resource Manager 和 Azure CLI，將受 SAS 權杖保護之範本中的資源部署到 Azure。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: ''
ms.service: azure-resource-manager
ms.devlang: azurecli
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/31/2017
ms.author: tomfitz
ms.openlocfilehash: d09aa46eabe705082edafe12da8442a79d1080b3
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34603191"
---
# <a name="deploy-private-resource-manager-template-with-sas-token-and-azure-cli"></a>使用 SAS 權杖和 Azure CLI 部署私用 Resource Manager 範本

當您的範本位於儲存體帳戶中時，您可以限制範本的存取權，並在部署期間提供共用存取簽章 (SAS) Token 本主題說明如何在部署期間使用 Azure PowerShell 與 Resource Manager 範本提供 SAS 權杖。 

## <a name="add-private-template-to-storage-account"></a>將私人範本新增至儲存體帳戶

您可以將範本加入儲存體帳戶，並在部署期間使用 SAS Token 連結它們。

> [!IMPORTANT]
> 遵循下列步驟，則僅有帳戶擁有者可以存取包含範本的 Blob。 不過，當您建立 Blob 的 SAS Token 時，具備該 URI 的任何人都可以存取該 Blob。 如果另一位使用者攔截了 URI，該使用者也能存取範本。 使用 SAS Token 是限制存取您的範本的好方法，但您不應該將機密資料 (如密碼) 直接包含在範本中。
> 
> 

下列範例會設定私用儲存體帳戶容器並上傳範本︰
   
```azurecli
az group create --name "ManageGroup" --location "South Central US"
az storage account create \
    --resource-group ManageGroup \
    --location "South Central US" \
    --sku Standard_LRS \
    --kind Storage \
    --name {your-unique-name}
connection=$(az storage account show-connection-string \
    --resource-group ManageGroup \
    --name {your-unique-name} \
    --query connectionString)
az storage container create \
    --name templates \
    --public-access Off \
    --connection-string $connection
az storage blob upload \
    --container-name templates \
    --file vmlinux.json \
    --name vmlinux.json \
    --connection-string $connection
```

### <a name="provide-sas-token-during-deployment"></a>在部署期間提供 SAS Token
若要在儲存體帳戶中部署私人範本，請產生 SAS Token 並將它包含在範本的 URI 中。 設定到期時間，以允許足夠的時間來完成部署。
   
```azurecli
expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
    --resource-group ManageGroup \
    --name {your-unique-name} \
    --query connectionString)
token=$(az storage blob generate-sas \
    --container-name templates \
    --name vmlinux.json \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
url=$(az storage blob url \
    --container-name templates \
    --name vmlinux.json \
    --output tsv \
    --connection-string $connection)
az group deployment create --resource-group ExampleGroup --template-uri $url?$token
```

如需使用包含已連結範本的 SAS Token 範例，請參閱 [透過 Azure Resource Manager 使用連結的範本](resource-group-linked-templates.md)。

## <a name="next-steps"></a>後續步驟
* 如需部署範本的簡介，請參閱[使用 Resource Manager 範本與 Azure PowerShell 來部署資源](resource-group-template-deploy-cli.md)。
* 如需部署範本的完整範例指令碼，請參閱[部署 Resource Manager 範本指令碼](resource-manager-samples-cli-deploy.md)。
* 若要在範本中定義參數，請參閱 [編寫範本](resource-group-authoring-templates.md#parameters)。
