---
title: 如何使用範本在 Azure VM 上設定 Azure 資源的受控識別
description: 使用 Azure Resource Manager 範本在 Azure VM 上設定 Azure 資源的受控識別相關的逐步指示。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/14/2017
ms.author: daveba
ms.openlocfilehash: bb62f892ec3d171958764d10f4b069bbd536d2ea
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223421"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-a-templates"></a>使用範本在 Azure VM 上設定 Azure 資源的受控識別

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Azure 資源受控識別會在 Azure Active Directory 中為 Azure 服務提供自動受控識別。 您可以使用此身分識別來向任何支援 Azure AD 驗證的服務進行驗證，不需要任何您程式碼中的認證。 

在本文中，藉由使用 Azure Resource Manager 部署範本，您將了解如何在 Azure VM 上執行 Azure 資源作業的下列受控識別：

## <a name="prerequisites"></a>必要條件

- 如果您不熟悉如何使用 Azure Resource Manager 部署範本，請參閱[概觀一節](overview.md)。 **請務必檢閱[系統指派和使用者指派受控識別之間的差異](overview.md#how-does-it-work)**。
- 如果您還沒有 Azure 帳戶，請先[註冊免費帳戶](https://azure.microsoft.com/free/)，再繼續進行。
- 若要執行本文中的管理作業，您的帳戶需要下列 Azure 角色型存取控制指派：

    > [!NOTE]
    > 不需要其他 Azure AD 目錄角色指派。

    - [虛擬機器參與者](/azure/role-based-access-control/built-in-roles#virtual-machine-contributor)，可建立 VM，並從 Azure VM 啟用和移除系統和/或使用者指派的受控識別。
    - [受控識別參與者](/azure/role-based-access-control/built-in-roles#managed-identity-contributor)角色，可建立使用者指派的受控識別。
    - [受控識別操作員](/azure/role-based-access-control/built-in-roles#managed-identity-operator)角色，可為虛擬機器指派和移除使用者指派的受控識別。

## <a name="azure-resource-manager-templates"></a>Azure 資源管理員範本

如同 Azure 入口網站和指令碼，[Azure Resource Manager](../../azure-resource-manager/resource-group-overview.md) 範本可讓您部署由 Azure 資源群組所定義的最新或已修改資源。 範本編輯和部署有幾個選項可用 (在本機和入口網站)，包括：

   - 使用 [Azure Marketplace 所提供的自訂範本](../../azure-resource-manager/resource-group-template-deploy-portal.md#deploy-resources-from-custom-template)，可讓您從頭開始建立範本，或根據現有的常用範本或[快速入門範本](https://azure.microsoft.com/documentation/templates/)來建立範本。
   - 衍生自現有的資源群組，方法是從[原始部署](../../azure-resource-manager/resource-manager-export-template.md#view-template-from-deployment-history)，或從[部署的目前狀態](../../azure-resource-manager/resource-manager-export-template.md#export-the-template-from-resource-group)匯出範本。
   - 使用本機 [JSON 編輯器 (例如 VS Code)](../../azure-resource-manager/resource-manager-create-first-template.md)，然後使用 PowerShell 或 CLI 上傳和部署。
   - 使用 Visual Studio 的 [Azure 資源群組專案](../../azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy.md)來建立和部署範本。  

不論您選擇的選項，在初始部署和重新部署期間，範本的語法都相同。 在新的或現有虛擬機器上啟用系統或使用者指派的受控識別，是以相同的方式來進行。 此外，根據預設，Azure Resource Manager 會對部署採取[累加式更新](../../azure-resource-manager/deployment-modes.md)。

## <a name="system-assigned-managed-identity"></a>系統指派的受控識別

在本節中，您將使用 Azure Resource Manager 範本以啟用和停用系統指派的受控識別。

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm-or-on-an-existing-vm"></a>在建立 Azure VM 時，或在現有虛擬機器上啟用系統指派的受控識別

1. 無論您是在本機登入 Azure 或透過 Azure 入口網站登入，都請使用與包含虛擬機器的 Azure 訂用帳戶相關聯的帳戶。

2. 若要啟用系統指派的受控識別，請將範本載入到編輯器、在 `resources` 區段中找出感興趣的 `Microsoft.Compute/virtualMachines` 資源，然後在與 `"type": "Microsoft.Compute/virtualMachines"` 屬性相同的層級上新增 `"identity"` 屬性。 使用下列語法：

   ```JSON
   "identity": { 
       "type": "SystemAssigned"
   },
   ```

3. (選擇性) 新增 Azure 資源擴充的虛擬機器受控識別做為 `resources` 項目。 此步驟是選擇性的，因為您也可以使用 Azure Instance Metadata Service (IMDS) 識別端點以擷取權杖。  使用下列語法：

   >[!NOTE] 
   > 下列範例假設正在部署 Windows 虛擬機器擴充 (`ManagedIdentityExtensionForWindows`)。 您也可以改用 `ManagedIdentityExtensionForLinux`來為 Linux 設定 `"name"` 和 `"type"` 元素。 VM 擴充已計劃在 2019 年 1 月淘汰。
   >

   ```JSON
   { 
       "type": "Microsoft.Compute/virtualMachines/extensions",
       "name": "[concat(variables('vmName'),'/ManagedIdentityExtensionForWindows')]",
       "apiVersion": "2018-06-01",
       "location": "[resourceGroup().location]",
       "dependsOn": [
           "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
       ],
       "properties": {
           "publisher": "Microsoft.ManagedIdentity",
           "type": "ManagedIdentityExtensionForWindows",
           "typeHandlerVersion": "1.0",
           "autoUpgradeMinorVersion": true,
           "settings": {
               "port": 50342
           },
           "protectedSettings": {}
       }
   }
   ```

4. 當您完成時，應該將下列區段新增至範本的 `resource` 區段，而且應該類似下列內容：

   ```JSON
   "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2018-06-01",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "[variables('vmName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "SystemAssigned",
                },
            },
            {
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "name": "[concat(variables('vmName'),'/ManagedIdentityExtensionForLinux')]",
            "apiVersion": "2018-06-01",
            "location": "[resourceGroup().location]",
            "dependsOn": [
                "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
            ],
            "properties": {
                "publisher": "Microsoft.ManagedIdentity",
                "type": "ManagedIdentityExtensionForWindows",
                "typeHandlerVersion": "1.0",
                "autoUpgradeMinorVersion": true,
                "settings": {
                    "port": 50342
                }
            }
        }
    ]
   ```

### <a name="assign-a-role-the-vms-system-assigned-managed-identity"></a>指派角色給虛擬機器的系統指派受控識別

在虛擬機器上啟用系統指派的受控識別後，建議您授予虛擬機器在其中建立的資源群組**讀者**存取這類角色。

1. 無論您是在本機登入 Azure 或透過 Azure 入口網站登入，都請使用與包含虛擬機器的 Azure 訂用帳戶相關聯的帳戶。
 
2. 在[編輯器](#azure-resource-manager-templates)中載入範本，然後新增下列資訊，以將 VM 在其中建立的資源群組**讀者**存取授與 VM。  您的範本結構可能會因您選擇的編輯器與部署模型而有所不同。
   
   在 `parameters` 區段下新增下列內容：

    ```JSON
    "builtInRoleType": {
          "type": "string",
          "defaultValue": "Reader"
        },
        "rbacGuid": {
          "type": "string"
        }
    ```

    在 `variables` 區段下新增下列內容：

    ```JSON
    "Reader": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')]"
    ```

    在 `resources` 區段下新增下列內容：

    ```JSON
    {
        "apiVersion": "2017-09-01",
         "type": "Microsoft.Authorization/roleAssignments",
         "name": "[parameters('rbacGuid')]",
         "properties": {
                "roleDefinitionId": "[variables(parameters('builtInRoleType'))]",
                "principalId": "[reference(variables('vmResourceId'), '2017-12-01', 'Full').identity.principalId]",
                "scope": "[resourceGroup().id]"
          },
          "dependsOn": [
                "[concat('Microsoft.Compute/virtualMachines/', parameters('vmName'))]"
            ]
    }
    ```

### <a name="disable-a-system-assigned-managed-identity-from-an-azure-vm"></a>停用來自 Azure VM 的系統指派受控識別

如果您的虛擬機器不再需要系統指派的受控識別：

1. 無論您是在本機登入 Azure 或透過 Azure 入口網站登入，都請使用與包含虛擬機器的 Azure 訂用帳戶相關聯的帳戶。

2. 在[編輯器](#azure-resource-manager-templates)中載入範本，然後在 `resources` 區段找出想要的 `Microsoft.Compute/virtualMachines` 資源。 若您的虛擬機器只有系統指派的受控識別，您可以將識別類型變更為 `None` 予以停用。  
   
   **Microsoft.Compute/virtualMachines API 版本 2018-06-01**

   如果虛擬機器同時具有系統與使用者指派的身分識別，請從受控識別類型中移除 `SystemAssigned`，並保留 `UserAssigned` 以及 `userAssignedIdentities` 字典值。

   **Microsoft.Compute/virtualMachines API 版本 2018-06-01 和先前版本**
   
   若 `apiVersion` 為 `2017-12-01` 且虛擬機器同時具有系統與使用者指派的受控識別，請從身分識別類型中移除 `SystemAssigned`，並保留 `UserAssigned` 以及使用者指派受控識別的 `identityIds` 陣列。  
   
下列範例示範如何從沒有使用者指派受控識別的虛擬機器中移除系統指派的受控識別：

```JSON
{
    "apiVersion": "2018-06-01",
    "type": "Microsoft.Compute/virtualMachines",
    "name": "[parameters('vmName')]",
    "location": "[resourceGroup().location]",
    "identity": { 
        "type": "None"
}
```

## <a name="user-assigned-managed-identity"></a>使用者指派的受控識別

在本節中，您會使用 Azure Resource Manager 範本，將使用者指派的受控識別指派至 Azure VM。

> [!Note]
> 若要使用 Azure Resource Manager 範本建立使用者指派的受控識別，請參閱[建立使用者指派的受控識別](how-to-manage-ua-identity-arm.md#create-a-user-assigned-managed-identity)。

 ### <a name="assign-a-user-assigned-managed-identity-to-an-azure-vm"></a>將使用者指派的受控識別指派給 Azure VM

1. 在 `resources` 元素之下，新增下列項目，以將使用者指派的受控識別指派至您的虛擬機器。  請務必將 `<USERASSIGNEDIDENTITY>` 取代為您建立的使用者指派受控識別名稱。

   **Microsoft.Compute/virtualMachines API 版本 2018-06-01**

   如果 `apiVersion` 為 `2018-06-01`，則使用者指派的受控識別會以 `userAssignedIdentities` 字典格式儲存，而 `<USERASSIGNEDIDENTITYNAME>` 值必須儲存在您範本 `variables` 區段內所定義的變數中。

   ```json
   {
       "apiVersion": "2018-06-01",
       "type": "Microsoft.Compute/virtualMachines",
       "name": "[variables('vmName')]",
       "location": "[resourceGroup().location]",
       "identity": {
           "type": "userAssigned",
           "userAssignedIdentities": {
               "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]": {}
           }
        }
   }
   ```
   
   **Microsoft.Compute/virtualMachines API 版本 2017-12-01 和先前版本**
    
   如果 `apiVersion` 為 `2017-12-01`，則使用者指派的受控識別會儲存在 `identityIds` 陣列中，而 `<USERASSIGNEDIDENTITYNAME>` 值必須儲存在您範本 `variables` 區段內所定義的變數中。
    
   ```json
   {
       "apiVersion": "2017-12-01",
       "type": "Microsoft.Compute/virtualMachines",
       "name": "[variables('vmName')]",
       "location": "[resourceGroup().location]",
       "identity": {
           "type": "userAssigned",
           "identityIds": [
               "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]"
           ]
       }
   }
   ```
       

2. (選用) 接著，在 `resources` 元素下新增下列項目，以將受控身分識別擴充指派至您的虛擬機器 (已計劃在 2019 年 1 月淘汰)。 此步驟是選擇性的，因為您也可以使用 Azure Instance Metadata Service (IMDS) 識別端點以擷取權杖。 使用下列語法：
    ```json
    {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "[concat(variables('vmName'),'/ManagedIdentityExtensionForWindows')]",
        "apiVersion": "2018-06-01",
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
        ],
        "properties": {
            "publisher": "Microsoft.ManagedIdentity",
            "type": "ManagedIdentityExtensionForWindows",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "port": 50342
            }
        }
    }
    ```
    
3. 當您完成時，應該將下列區段新增至範本的 `resource` 區段，而且應該類似下列內容：
   
   **Microsoft.Compute/virtualMachines API 版本 2018-06-01**    

   ```JSON
   "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2018-06-01",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "[variables('vmName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "userAssigned",
                "userAssignedIdentities": {
                   "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]": {}
                }
            }
        },
        {
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "name": "[concat(variables('vmName'),'/ManagedIdentityExtensionForLinux')]",
            "apiVersion": "2018-06-01-preview",
            "location": "[resourceGroup().location]",
            "dependsOn": [
                "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
            ],
            "properties": {
                "publisher": "Microsoft.ManagedIdentity",
                "type": "ManagedIdentityExtensionForWindows",
                "typeHandlerVersion": "1.0",
                "autoUpgradeMinorVersion": true,
                "settings": {
                    "port": 50342
            }
        }
       }
    ]
   ```
   **Microsoft.Compute/virtualMachines API 版本 2017-12-01 和先前版本**
   
   ```JSON
   "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2017-12-01",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "[variables('vmName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "userAssigned",
                "identityIds": [
                   "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]"
                ]
            }
        },
        {
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "name": "[concat(variables('vmName'),'/ManagedIdentityExtensionForLinux')]",
            "apiVersion": "2015-05-01-preview",
            "location": "[resourceGroup().location]",
            "dependsOn": [
                "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
            ],
            "properties": {
                "publisher": "Microsoft.ManagedIdentity",
                "type": "ManagedIdentityExtensionForWindows",
                "typeHandlerVersion": "1.0",
                "autoUpgradeMinorVersion": true,
                "settings": {
                    "port": 50342
            }
        }
       }
    ]
   ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>從 Azure VM 移除使用者指派的受控識別

如果您的虛擬機器不再需要使用者指派的受控識別：

1. 無論您是在本機登入 Azure 或透過 Azure 入口網站登入，都請使用與包含虛擬機器的 Azure 訂用帳戶相關聯的帳戶。

2. 在[編輯器](#azure-resource-manager-templates)中載入範本，然後在 `resources` 區段找出想要的 `Microsoft.Compute/virtualMachines` 資源。 若您的虛擬機器只有使用者指派的受控識別，您可以將身分識別類型變更為 `None` 予以停用。
 
   下列範例示範如何從沒有系統指派受控識別的虛擬機器中移除所有使用者指派的受控識別：
   
   ```json
    {
      "apiVersion": "2018-06-01",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "identity": { 
          "type": "None"
    }
   ```
   
   **Microsoft.Compute/virtualMachines API 版本 2018-06-01 和先前版本**
    
   若要從虛擬機器中移除單一使用者指派的受控識別，請從 `useraAssignedIdentities` 字典中移除它。

   如果您有系統指派的受控識別，則將它保存在 `identity` 值下方的 `type` 值中。
 
   **Microsoft.Compute/virtualMachines API 版本 2017-12-01**

   若要從虛擬機器中移除單一使用者指派的受控識別，請從 `identityIds` 陣列中移除它。

   如果您有系統指派的受控識別，則將它保存在 `identity` 值下方的 `type` 值中。
   
## <a name="next-steps"></a>後續步驟

- [適用於 Azure 資源的受控識別概觀](overview.md)。

