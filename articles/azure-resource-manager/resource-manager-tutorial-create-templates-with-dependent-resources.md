---
title: 使用相依資源建立 Azure Resource Manager 範本 | Microsoft Docs
description: 了解如何使用多項資源建立 Azure Resource Manager 範本，以及如何使用 Azure 入口網站加以部署
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 10/09/2018
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 50f1c81f08787181de2fe3a9f6fb97a96a2bd882
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49114307"
---
# <a name="tutorial-create-azure-resource-manager-templates-with-dependent-resources"></a>教學課程：使用相依資源建立 Azure Resource Manager 範本

了解如何建立 Azure Resource Manager 範本以部署多項資源。  在建立範本之後，您可以從 Azure 入口網站使用 Cloud Shell 來部署範本。

在本教學課程中，您會建立儲存體帳戶、虛擬機器、虛擬網路和其他相依資源。 某些資源必須在另一項資源已存在時才能部署。 例如，在虛擬機器的儲存體帳戶和網路介面存在之前，您無法建立虛擬機器。 您可以讓一項資源相依於其他資源，以定義此關聯性。 資源管理員會評估資源之間的相依性，並依其相依順序進行部署。 如果資源並未彼此相依，Resource Manager 就會平行部署資源。 如需詳細資訊，請參閱[定義 Azure Resource Manager 範本中部署資源的順序](./resource-group-define-dependencies.md)。

本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 準備 Key Vault
> * 開啟快速入門範本
> * 瀏覽範本
> * 編輯參數檔案
> * 部署範本

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

若要完成本文，您需要：

* [Visual Studio Code](https://code.visualstudio.com/) 搭配 Resource Manager Tools 擴充功能。  請參閱[安裝擴充功能](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)

## <a name="prepare-key-vault"></a>準備 Key Vault

為了防止密碼噴濺攻擊，建議將自動產生的密碼用於虛擬機器系統管理員帳戶密碼，並使用 Key Vault 來儲存密碼。 下列程序會建立 Key Vault 和秘密，以儲存密碼。 此外也會設定範本部署對儲存在 Key Vault 中的祕密進行存取所需的權限。 如果 Key Vault 位於不同的 Azure 訂用帳戶下，則需要額外的存取原則。 如需詳細資訊，請參閱[在部署期間使用 Azure Key Vault 以傳遞安全的參數值](./resource-manager-keyvault-parameter.md)。

1. 登入 [Azure Cloud Shell](https://shell.azure.com)。
2. 從左上角的 **PowerShell** 或 **Bash**，切換至您慣用的環境。
3. 執行下列 Azure PowerShell 或 Azure CLI 命令。  

    ```azurecli-interactive
    keyVaultName='<your-unique-vault-name>'
    resourceGroupName='<your-resource-group-name>'
    location='Central US'
    userPrincipalName='<your-email-address-associated-with-your-subscription>'
    
    # Create a resource group
    az group create --name $resourceGroupName --location $location
    
    # Create a Key Vault
    keyVault=$(az keyvault create \
      --name $keyVaultName \
      --resource-group $resourceGroupName \
      --location $location \
      --enabled-for-template-deployment true)
    keyVaultId=$(echo $keyVault | jq -r '.id')
    az keyvault set-policy --upn $userPrincipalName --name $keyVaultName --secret-permissions set delete get list

    # Create a secret
    password=$(openssl rand -base64 32)
    az keyvault secret set --vault-name $keyVaultName --name 'vmAdminPassword' --value $password
    
    # Print the useful property values
    echo "You need the following values for the virtual machine deployment:"
    echo "Resource group name is: $resourceGroupName."
    echo "The admin password is: $password."
    echo "The Key Vault resource ID is: $keyVaultId."
    ```

    ```azurepowershell-interactive
    $keyVaultName = "<your-unique-vault-name>"
    $resourceGroupName="<your-resource-group-name>"
    $location='Central US'
    $userPrincipalName="<your-email-address-associated-with-your-subscription>"
    
    # Create a resource group
    New-AzureRmResourceGroup -Name $resourceGroupName -Location $location
        
    # Create a Key Vault
    $keyVault = New-AzureRmKeyVault `
      -VaultName $keyVaultName `
      -resourceGroupName $resourceGroupName `
      -Location $location `
      -EnabledForTemplateDeployment
    Set-AzureRmKeyVaultAccessPolicy -VaultName $keyVaultName -UserPrincipalName $userPrincipalName -PermissionsToSecrets set,delete,get,list
      
    # Create a secret
    $password = openssl rand -base64 32
    
    $secretValue = ConvertTo-SecureString $password -AsPlainText -Force
    Set-AzureKeyVaultSecret -VaultName $keyVaultName -Name "vmAdminPassword" -SecretValue $secretValue
    
    # Print the useful property values
    echo "You need the following values for the virtual machine deployment:"
    echo "Resource group name is: $resourceGroupName."
    echo "The admin password is: $password."
    echo "The Key Vault resource ID is: " $keyVault.ResourceID
    ```
4. 記下輸出值。 稍後在教學課程中將需要這些資訊

> [!NOTE]
> 每個 Azure 服務都有特定的密碼需求。 例如，Azure 虛擬機器的需求可在「建立 VM 時的密碼需求為何？」上找到

## <a name="open-a-quickstart-template"></a>開啟快速入門範本

Azure 快速入門範本是 Resource Manager 範本的存放庫。 您可以尋找範例範本並加以自訂，而不要從頭建立範本。 本教學課程中使用的範本名為[部署簡單的 Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/)。

1. 在 Visual Studio Code 中，選取 [檔案]>[開啟檔案]。
2. 在 [檔案名稱] 中，貼上下列 URL：

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```
3. 選取 [開啟] 以開啟檔案。
4. 選取 [檔案]>[另存新檔]，以名稱 **azuredeploy.json** 將檔案的複本儲存至您的本機電腦。
5. 重複步驟 1-4 以開啟 **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.parameters.json**，然後將檔案儲存為 **azuredeploy.parameters.json**。

## <a name="explore-the-template"></a>瀏覽範本

當您探索這一節中的範本時，請試著回答下列問題：

- 此範本中定義了多少個 Azure 資源？
- 其中一項資源是 Azure 儲存體帳戶。  定義是否與最後一個教學課程中使用的定義相仿？
- 您是否可找到此範本中定義的資源所適用的範本參考？
- 您是否可找到資源的相依性？

1. 在 Visual Studio Code 中摺疊元素，直到您只看到**資源**內的第一層元素和第二層元素：

    ![Visual Studio Code 的 Azure Resource Manager 範本](./media/resource-manager-tutorial-create-templates-with-dependent-resources/resource-manager-template-visual-studio-code.png)

    範本中定義了五項資源。
2. 展開第一項資源。 這是儲存體帳戶。 其定義應與最後一個教學課程開頭所用的定義相同。

    ![Visual Studio Code 的 Azure Resource Manager 範本儲存體帳戶定義](./media/resource-manager-tutorial-create-templates-with-dependent-resources/resource-manager-template-storage-account-definition.png)

3. 展開第二項資源。 其資源類型為 **Microsoft.Network/publicIPAddresses**。 若要尋找範本參考，請瀏覽至[範本參考](https://docs.microsoft.com/azure/templates/)，然後在 [依標題篩選] 欄位中輸入**公用 IP 位址**或**多個公用 IP 位址**。 將資源定義與範本參考相比較。

    ![Visual Studio Code 的 Azure Resource Manager 範本公用 IP 位址定義](./media/resource-manager-tutorial-create-templates-with-dependent-resources/resource-manager-template-public-ip-address-definition.png)
4. 重複上一個步驟，以尋找此範本中定義的其他資源所適用的範本參考。  將資源定義與參考相比較。
5. 展開第四項資源：

    ![Visual Studio Code 的 Azure Resource Manager 範本 dependson](./media/resource-manager-tutorial-create-templates-with-dependent-resources/resource-manager-template-visual-studio-code-dependson.png)

    dependsOn 元素可讓您定義一項資源，作為一或多項資源的相依項目。 在此範例中，此資源是 networkInterface。  它依存於其他兩項資源：

    * publicIPAddress
    * virtualNetwork

6. 展開第五項資源。 此資源是虛擬機器。 它依存於其他兩項資源：

    * storageAccount
    * networkInterface

下圖說明此範本的資源和相依性資訊：

![Visual Studio Code 的 Azure Resource Manager 範本相依性圖表](./media/resource-manager-tutorial-create-templates-with-dependent-resources/resource-manager-template-visual-studio-code-dependency-diagram.png)

藉由指定相依性，Resource Manager 將可有效部署解決方案。 它會以平行方式部署儲存體帳戶、公用 IP 位址和虛擬網路，因為它們沒有相依性。 在公用 IP 位址和虛擬網路部署之後，會建立網路介面。 當所有其他資源皆部署後，Resource Manager 會部署虛擬機器。

## <a name="edit-the-parameters-file"></a>編輯參數檔案

您完全不必對範本檔案進行任何變更。 但是，您必須修改參數檔案，才能從 Key Vault 中擷取系統管理員密碼。

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
    使用在上一個程序所建立 Key Vault 的資源識別碼取代 **id**。 這是其中一個輸出。 

    ![整合金鑰保存庫與 Resource Manager 範本虛擬機器部署參數檔案](./media/resource-manager-tutorial-use-key-vault/resource-manager-tutorial-create-vm-parameters-file.png)
3. 提供下列值：

    - **adminUsername**：虛擬機器系統管理員帳戶的名稱。
    - **dnsLabelPrefix**：dnsLablePrefix 的名稱。
4. 儲存變更。

## <a name="deploy-the-template"></a>部署範本

有許多方法可用來部署範本。  在本教學課程中，您會從 Azure 入口網站使用 Cloud Shell。

1. 登入 [Cloud Shell](https://shell.azure.com)。 您也可以登入 [Azure 入口網站](https://portal.azure.com)，然後選取位於右上角的 [Cloud Shell]，如下圖所示：

    ![Azure 入口網站的 Cloud Shell](./media/resource-manager-tutorial-create-templates-with-dependent-resources/azure-portal-cloud-shell.png)
2. 從 Cloud Shell 的左上角選取 [PowerShell]，然後選取 [確認]。  在本教學課程中您會使用 PowerShell。
3. 從 Cloud Shell 中選取 [上傳檔案]：

    ![Azure 入口網站的 Cloud Shell 上傳檔案](./media/resource-manager-tutorial-create-templates-with-dependent-resources/azure-portal-cloud-shell-upload-file.png)
4. 選取您先前在本教學課程中儲存的檔案。 預設名稱為 **azuredeploy.json** 和 **azuredeploy.paraemters.json**。  如果有檔案具有相同的檔案名稱，將會直接覆寫舊檔案而不另行通知。
5. 從 Cloud Shell 執行下列命令，以確認已成功上傳檔案。 

    ```bash
    ls
    ```

    ![Azure 入口網站的 Cloud Shell 列出檔案](./media/resource-manager-tutorial-create-templates-with-dependent-resources/azure-portal-cloud-shell-list-file.png)

    顯示在螢幕擷取畫面上的檔案名稱是 azuredeploy.json。

6. 從 Cloud Shell 執行下列命令，以驗證 JSON 檔案的內容：

    ```bash
    cat azuredeploy.json
    cat azuredeploy.parameters.json
    ```
7. 從 Cloud Shell 執行下列 PowerShell 命令。 範例指令碼會使用為 Key Vault 建立的相同資源群組。 使用相同的資源群組可讓您更輕鬆地清除資源。

    ```powershell
    $resourceGroupName = "<Enter the resource group name>"
    $deploymentName = "<Enter a deployment name>"

    New-AzureRmResourceGroupDeployment -Name $deploymentName `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile azuredeploy.json `
        -TemplateparameterFile azuredeploy.parameters.json
    ```
8. 執行下列 PowerShell 命令，以列出新建立的虛擬機器：

    ```powershell
    Get-AzureRmVM -Name SimpleWinVM -ResourceGroupName $resourceGroupName
    ```

    在範本內，虛擬機器名稱會硬式編碼為 **SimpleWinVM**。

9. 登入虛擬機器，以測試系統管理員的認證。 

## <a name="clean-up-resources"></a>清除資源

不再需要 Azure 資源時，可藉由刪除資源群組來清除您所部署的資源。

1. 在 Azure 入口網站中，選取左側功能表中的 [資源群組]。
2. 在 [依名稱篩選] 欄位中輸入資源群組名稱。
3. 選取資源群組名稱。  您在資源群組中應該會看到共計六個資源。
4. 從頂端功能表中選取 [刪除資源群組]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您會開發和部署用來建立虛擬機器、虛擬網路和相依資源的範本。 若要深入了解如何根據條件部署 Azure 資源，請參閱：


> [!div class="nextstepaction"]
> [使用條件](./resource-manager-tutorial-use-conditions.md)

