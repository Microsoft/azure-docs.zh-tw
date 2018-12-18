---
title: 將 Azure Key Vault 用於受控應用程式 | Microsoft Docs
description: 說明在部署受控應用程式時如何使用 Azure Key Vault 中的存取祕密
services: managed-applications
author: tfitzmac
manager: timlt
ms.service: managed-applications
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.date: 07/11/2018
ms.author: tomfitz
ms.openlocfilehash: f091ba44a3170dcc4141829f2f4105d6e7993cdf
ms.sourcegitcommit: 04fc1781fe897ed1c21765865b73f941287e222f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39035284"
---
# <a name="access-key-vault-secret-when-deploying-azure-managed-applications"></a>在部署 Azure 受控應用程式時存取 Key Vault 秘密

當您需要在部署期間，傳送安全值 (例如密碼) 做為參數時，您可以從 [Azure Key Vault](../key-vault/key-vault-whatis.md) 擷取值。 若要在部署受控應用程式時存取 Key Vault，您必須授與**設備資源提供者**服務主體的存取權。 本文說明如何設定與受控應用程式搭配使用的 Key Vault。

## <a name="enable-template-deployment"></a>啟用範本部署

1. 在入口網站中，選取您的 Key Vault。

1. 選取 [存取原則]。   

   ![選取存取原則](./media/key-vault-access/select-access-policies.png)

1. 選取 [按一下以顯示進階存取原則]。

   ![顯示進階存取原則](./media/key-vault-access/advanced.png)

1. 選取 [為範本部署啟用對 Azure Resource Manager 的存取]。 然後選取 [儲存]。

   ![啟用範本部署](./media/key-vault-access/enable-template.png)

## <a name="add-service-as-contributor"></a>將服務新增為參與者

1. 選取 [存取控制 (IAM)]。

   ![選取存取控制](./media/key-vault-access/access-control.png)

1. 選取 [新增] 。

   ![選取 [新增]](./media/key-vault-access/add-access-control.png)

1. 選取 [參與者] 作為角色。 搜尋**設備資源提供者**，並從可用的選項中加以選取。

   ![搜尋提供者](./media/key-vault-access/search-provider.png)

1. 選取 [ **儲存**]。

## <a name="reference-key-vault-secret"></a>參考 Key Vault 祕密

若要將 Key Vault 中的祕密傳至受控應用程式中的範本，您必須使用[連結的範本](../azure-resource-manager/resource-group-linked-templates.md)，並參考連結範本參數中的 Key Vault。 請提供 Key Vault 的資源識別碼和秘密的名稱。

```json
"resources": [{
  "apiVersion": "2015-01-01",
  "name": "linkedTemplate",
  "type": "Microsoft.Resources/deployments",
  "properties": {
    "mode": "incremental",
    "templateLink": {
      "uri": "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/keyvaultparameter/sqlserver.json",
      "contentVersion": "1.0.0.0"
    },
    "parameters": {
      "adminPassword": {
        "reference": {
          "keyVault": {
            "id": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<key-vault-name>"
          },
          "secretName": "<secret-name>"
        }
      },
      "adminLogin": { "value": "[parameters('adminLogin')]" },
      "sqlServerName": {"value": "[parameters('sqlServerName')]"}
    }
  }
}],
```

## <a name="next-steps"></a>後續步驟

您已將 Key Vault 設定為在受控應用程式部署期間可供存取。

* 如需從 Key Vault 傳入值作為範本參數的相關資訊，請參閱[在部署期間使用 Azure Key Vault 傳遞安全的參數值](../azure-resource-manager/resource-manager-keyvault-parameter.md)。
* 如需受控應用程式範例，請參閱[適用於 Azure 受控應用程式之範例專案](sample-projects.md)。
* 若要了解如何建立受控應用程式的 UI 定義檔案，請參閱[開始使用 CreateUiDefinition](create-uidefinition-overview.md)。