---
title: 在 Resource Manager 範本部署中整合 Azure Key Vault | Microsoft Docs
description: 了解如何使用 Azure Key Vault 在 Resource Manager 範本部署期間傳遞安全的參數值
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 10/10/2018
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 46291fe425e5d0fd1c142960a250c5601bd8ad29
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49077996"
---
# <a name="tutorial-integrate-azure-key-vault-in-resource-manager-template-deployment"></a>教學課程：在 Resource Manager 範本部署中整合 Azure Key Vault

了解如何從 Azure Key Vault 擷取祕密值，並且在 Resource Manager 部署期間傳遞祕密值作為參數。 您只參考其 Key Vault 識別碼，因此該值絕不會公開。 如需詳細資訊，請參閱[在部署期間使用 Azure Key Vault 以傳遞安全的參數值](./resource-manager-keyvault-parameter.md)。

在本教學課程中，您會使用[教學課程：建立具有相依資源的 Azure Resource Manager 範本](./resource-manager-tutorial-create-templates-with-dependent-resources.md)中使用的相同範本，建立虛擬機器以及一些相依資源。 從 Azure Key Vault 擷取虛擬機器的系統管理員密碼。

本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 準備 Key Vault
> * 開啟快速入門範本
> * 編輯參數檔案
> * 部署範本
> * 驗證部署
> * 清除資源

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

若要完成本文，您需要：

* [Visual Studio Code](https://code.visualstudio.com/) 搭配 [Resource Manager Tools 擴充功能](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)

## <a name="prepare-the-key-vault"></a>準備 Key Vault

在本節中，您會使用 Resource Manager 範本建立 Key Vault 和祕密。 此範本會：

* 建立已啟用 **enabledForTemplateDeployment** 屬性的 Key Vault。 此屬性必須為 true，範本部署程序才能存取此 Key Vault 中定義的祕密。
* 將秘密新增至 Key Vault。  此祕密會儲存虛擬機器的系統管理員密碼。

如果您 (作為要部署虛擬機器範本的使用者) 不是 Key Vault 的擁有者或參與者，則 Key Vault 的擁有者或參與者必須准許您存取 Key Vault 的 Microsoft.KeyVault/vaults/deploy/action 權限。 如需詳細資訊，請參閱[在部署期間使用 Azure Key Vault 以傳遞安全的參數值](./resource-manager-keyvault-parameter.md)。

範本需要您的 Azure AD 使用者物件識別碼來設定權限。 下列程序會取得物件識別碼 (GUID)，也會產生系統管理員密碼。 若要避免密碼噴濺攻擊，建議使用所產生的密碼。

1. 執行下列 Azure PowerShell 或 Azure CLI 命令。  

    ```azurecli-interactive
    az ad user show --upn-or-object-id "<Your User Principle Name>" --query "objectId"
    openssl rand -base64 32
    ```
    ```azurepowershell-interactive
    $upn = Read-Host -Prompt "Input your user principal name (email address) used to sign in to Azure"
    (Get-AzureADUser -ObjectId $upn).ObjectId
    openssl rand -base64 32
    ```
2. 記下物件識別碼和所產生的密碼。 您稍後需要這些資訊。
3. 請確認所產生的密碼符合虛擬機器的密碼需求。 每個 Azure 服務都有特定的密碼需求。 如需 VM 密碼需求，請參閱[建立 VM 時的密碼需求為何？](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm)。

若要建立 Key Vault：

1. 選取以下影像來登入 Azure 並開啟範本。 此範本會建立 Key Vault 和 Key Vault 祕密。

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Farmtutorials.blob.core.windows.net%2Fcreatekeyvault%2FCreateKeyVault.json"><img src="./media/resource-manager-tutorial-use-key-vault/deploy-to-azure.png" /></a>

2. 選取或輸入下列值。  請勿在輸入值之後選取 [購買]。

    ![Resource Manager 範本 Key Vault 整合部署入口網站](./media/resource-manager-tutorial-use-key-vault/resource-manager-tutorial-create-key-vault-portal.png)

    * **訂用帳戶**：選取 Azure 訂用帳戶。
    * **資源群組**：指派唯一的名稱。 請記下此名稱，您會在下一節中使用相同的資源群組來部署虛擬機器。 將 Key Vault 和虛擬機器放在相同的資源群組中，可讓您在本教學課程結束時更輕鬆地清除資源。
    * **位置**：選取位置。  預設位置為 [美國中部]。
    * **Key Vault 名稱**：指派唯一的名稱。 
    * **租用戶識別碼**：範本功能會自動擷取您的租用戶識別碼。請勿變更預設值
    * **AD 使用者識別碼**：輸入您從上一個程序中擷取的 Azure AD 使用者物件識別碼。
    * **祕密名稱**：預設名稱是 **mAdminPassword**。 如果您在此變更祕密名稱，則需要在部署虛擬機器時更新祕密名稱。
    * **祕密值**：輸入您的祕密。  祕密是用來登入虛擬機器的密碼。 建議使用您在上一個程序中產生的密碼。
    * **我同意上方所述的條款及條件**：選取。
3. 選取頂端的 [編輯參數] 來看一下範本。
4. 瀏覽至範本 JSON 檔案的第 28 行。 這是 Key Vault 資源定義。
5. 瀏覽至第 35 行：

    ```json
    "enabledForTemplateDeployment": true,
    ```
    `enabledForTemplateDeployment` 是 Key Vault 屬性。 此屬性必須為 true，才可以在部署期間從此 Key Vault 擷取祕密。 
6. 瀏覽至第 89 行。 這是 Key Vault 祕密定義。
7. 選取頁面底部的 [捨棄]。 您並未進行任何變更。
8. 確認您已提供如前一個螢幕擷取畫面顯示的所有值，然後按一下頁面底部的 [購買]。
9. 選取頁面頂端的鈴鐺圖示 (通知)，以開啟 [通知] 窗格。 等到資源部署成功為止。
8. 在 [通知] 窗格中選取 [移至資源群組]。 
9. 輸入 Key Vault 名稱，加以開啟。
10. 從左窗格中選取 [存取原則]。 您的名稱 (Active Directory) 應會列出，否則您就沒有存取金鑰保存庫的權限。
11. 選取 [按一下以顯示進階存取原則]。 請注意，[為範本部署啟用對 Azure Resource Manager 的存取] 已選取。 這是讓 Key Vault 整合得以運作的另一個條件。

    ![Resource Manager 範本 Key Vault 整合存取原則](./media/resource-manager-tutorial-use-key-vault/resource-manager-tutorial-key-vault-access-policies.png)    
12. 選取左窗格中的 [屬性]。
13. 複製 [資源識別碼] 。 當您部署虛擬機器時，您會需要此識別碼。  資源識別碼格式為：

    ```
    /subscriptions/<SubscriptionID>/resourceGroups/mykeyvaultdeploymentrg/providers/Microsoft.KeyVault/vaults/<KeyVaultName>
    ```

## <a name="open-a-quickstart-template"></a>開啟快速入門範本

Azure 快速入門範本是 Resource Manager 範本的存放庫。 您可以尋找範例範本並加以自訂，而不要從頭建立範本。 本教學課程中使用的範本名為[部署簡單的 Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/)。

1. 在 Visual Studio Code 中，選取 [檔案]>[開啟檔案]。
2. 在 [檔案名稱] 中，貼上下列 URL：

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```
3. 選取 [開啟] 以開啟檔案。 這與[教學課程：建立具有相依資源的 Azure Resource Manager 範本](./resource-manager-tutorial-create-templates-with-dependent-resources.md)中使用的案例相同。
4. 選取 [檔案]>[另存新檔]，以名稱 **azuredeploy.json** 將檔案的複本儲存至您的本機電腦。
5. 重複步驟 1-4 以開啟下列 URL，然後將檔案儲存為 **azuredeploy.parameters.json**。

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.parameters.json
    ```

## <a name="edit-the-parameters-file"></a>編輯參數檔案

您完全不必對範本檔案進行任何變更。

1. 在 Visual Studio Code 中開啟 **azuredeploy.parameters.json** (如果尚未開啟)。
2. 將 **adminPassword** 參數更新為：

    ```json
    "adminPassword": {
        "reference": {
            "keyVault": {
            "id": "/subscriptions/<SubscriptionID>/resourceGroups/mykeyvaultdeploymentrg/providers/Microsoft.KeyVault/vaults/<KeyVaultName>"
            },
            "secretName": "vmAdminPassword"
        }
    },
    ```
    使用在上一個程序所建立 Key Vault 的資源識別碼取代 **id**。  

    ![整合金鑰保存庫與 Resource Manager 範本虛擬機器部署參數檔案](./media/resource-manager-tutorial-use-key-vault/resource-manager-tutorial-create-vm-parameters-file.png)
3. 提供下列值：

    * **adminUsername**：虛擬機器系統管理員帳戶的名稱。
    * **dnsLabelPrefix**：dnsLablePrefix 的名稱。
4. 儲存變更。

## <a name="deploy-the-template"></a>部署範本

請依照[部署範本](./resource-manager-tutorial-create-templates-with-dependent-resources.md#deploy-the-template)中的指示來部署範本。 您必須將 **azuredeploy.json** 與 **azuredeploy.parameters.json** 上傳到 Cloud Shell，然後使用下列 PowerShell 指令碼來部署範本：

```azurepowershell
$resourceGroupName = Read-Host -Prompt "Enter the resource group name of the Key Vault"
$deploymentName = Read-Host -Prompt "Enter the name for this deployment"
New-AzureRmResourceGroupDeployment -Name $deploymentName -ResourceGroupName $resourceGroupName `
    -TemplateFile azuredeploy.json -TemplateParameterFile azuredeploy.parameters.json
```

當您部署範本時，請使用相同的資源群組作為 Key Vault。 這可讓您更為輕鬆地清除資源。 您只需要刪除一個資源群組，而非兩個資源群組。

## <a name="valid-the-deployment"></a>讓部署生效

在您成功部署虛擬機器之後，請使用在 Key Vault 中儲存的密碼來測試登入。

1. 開啟 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [資源群組識別碼]/**YourResourceGroupName >**/**simpleWinVM**
3. 選取頂端的 [連線]。
4. 選取 [下載 RDP 檔案]，然後依照指示，使用 Key Vault 中儲存的密碼來登入虛擬機器。

## <a name="clean-up-resources"></a>清除資源

不再需要 Azure 資源時，可藉由刪除資源群組來清除您所部署的資源。

1. 在 Azure 入口網站中，選取左側功能表中的 [資源群組]。
2. 在 [依名稱篩選] 欄位中輸入資源群組名稱。
3. 選取資源群組名稱。  您在資源群組中應該會看到共計六個資源。
4. 從頂端功能表中選取 [刪除資源群組]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您從 Azure Key Vault 擷取了祕密，並且在範本部署中使用此祕密。  若要了解如何建立連結的範本，請參閱：

> [!div class="nextstepaction"]
> [建立連結的範本](./resource-manager-tutorial-create-linked-templates.md)
