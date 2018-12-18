---
title: 在 Azure Resource Manager 範本中使用條件 | Microsoft Docs
description: 深入了解如何根據條件部署 Azure 資源。
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 10/02/2018
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 216e474f519e57352b017dc3e6bcdd74d48b03de
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48238641"
---
# <a name="tutorial-use-condition-in-azure-resource-manager-templates"></a>教學課程：在 Azure Resource Manager 範本中使用條件

深入了解如何根據條件部署 Azure 資源。 

本教學課程中所使用的案例與[教學課程：建立具有相依資源的 Azure Resource Manager 範本](./resource-manager-tutorial-create-templates-with-dependent-resources.md)中使用的案例類似。 在該教學課程中，您會建立儲存體帳戶、虛擬機器、虛擬網路和其他相依資源。 您不是建立新的儲存體帳戶，而是讓使用者在建立新的儲存體帳戶與使用現有的儲存體帳戶之間做選擇。 為了達成此目標，您會定義額外的參數。 如果參數的值是 "new"，則會建立新的儲存體帳戶。

本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 開啟快速入門範本
> * 修改範本
> * 部署範本
> * 清除資源

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

若要完成本文，您需要：

* [Visual Studio Code](https://code.visualstudio.com/) 搭配 [Resource Manager Tools 擴充功能](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)

## <a name="open-a-quickstart-template"></a>開啟快速入門範本

Azure 快速入門範本是 Resource Manager 範本的存放庫。 您可以尋找範例範本並加以自訂，而不要從頭建立範本。 本教學課程中使用的範本名為[部署簡單的 Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/)。

1. 在 Visual Studio Code 中，選取 [檔案]>[開啟檔案]。
2. 在 [檔案名稱] 中，貼上下列 URL：

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```
3. 選取 [開啟] 以開啟檔案。
4. 選取 [檔案]>[另存新檔]，以名稱 **azuredeploy.json** 將檔案的複本儲存至您的本機電腦。

## <a name="modify-the-template"></a>修改範本

對現有範本進行兩個變更：

* 新增參數，用來提供儲存體帳戶名稱。 此參數可讓使用者選擇指定現有儲存體帳戶名稱。 也可以作為新儲存體帳戶名稱。
* 新增名為 **newOrExisting** 的參數。 部署會使用這個參數來決定要建立新的儲存體帳戶或使用現有儲存體帳戶。

1. 在 Visual Studio Code 中，開啟 **azuredeploy.json**。
2. 在整個範本中以 **parameters('storageAccountName')** 來取代 **variables('storageAccountName')**。  **variables('storageAccountName')** 會出現三次。
3. 移除下列變數定義：

    ```json
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'sawinvm')]",
    ```
4. 將以下兩個參數新增至範本：

    ```json
    "newOrExisting": {
      "type": "string"
    },
    "storageAccountName": {
      "type": "string"
    },
    ```
    已更新的參數定義看起來如下：

    ![Resource Manager 使用條件](./media/resource-manager-tutorial-use-conditions/resource-manager-tutorial-use-condition-template-parameters.png)

5. 將下列行新增至儲存體帳戶定義的開頭。

    ```json
    "condition": "[equals(parameters('newOrExisting'),'yes')]",
    ```

    條件會檢查名為 **newOrExisting** 的參數值。 如果參數值是 **new**，則部署會建立儲存體帳戶。

    已更新的儲存體帳戶定義看起來如下：

    ![Resource Manager 使用條件](./media/resource-manager-tutorial-use-conditions/resource-manager-tutorial-use-condition-template.png)

6. 儲存變更。

## <a name="deploy-the-template"></a>部署範本

請依照[部署範本](./resource-manager-tutorial-create-templates-with-dependent-resources.md#deploy-the-template)中的指示來部署範本。

當您使用 Azure PowerShell 部署範本時，需要指定一個額外參數：

```powershell
$resourceGroupName = "<Enter the resource group name>"
$storageAccountName = "Enter the storage account name>"
$location = "<Enter the Azure location>"
$vmAdmin = "<Enter the admin username>"
$vmPassword = "<Enter the password>"
$dnsLabelPrefix = "<Enter the prefix>"

New-AzureRmResourceGroup -Name $resourceGroupName -Location $location
$vmPW = ConvertTo-SecureString -String $vmPassword -AsPlainText -Force
New-AzureRmResourceGroupDeployment -Name mydeployment0710 -ResourceGroupName $resourceGroupName `
    -TemplateFile azuredeploy.json -adminUsername $vmAdmin -adminPassword $vmPW `
    -dnsLabelPrefix $dnsLabelPrefix -storageAccountName $storageAccountName -newOrExisting "new"
```

> [!NOTE]
> 如果 **newOrExisting** 是 **new**，則部署會失敗，但是具有已指定儲存體帳戶名稱的儲存體帳戶已存在。

請嘗試將 **newOrExisting** 設為 "existing" 並且指定現有儲存體帳戶，來進行另一個部署。 若要事先建立儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)。

## <a name="clean-up-resources"></a>清除資源

不再需要 Azure 資源時，可藉由刪除資源群組來清除您所部署的資源。

1. 在 Azure 入口網站中，選取左側功能表中的 [資源群組]。
2. 在 [依名稱篩選] 欄位中輸入資源群組名稱。
3. 選取資源群組名稱。  您在資源群組中應該會看到共計六個資源。
4. 從頂端功能表中選取 [刪除資源群組]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您會開發範本，讓使用者在建立新儲存體帳戶與使用現有儲存體帳戶之間做選擇。 在本教學課程中建立的虛擬機器需要系統管理員使用者名稱和密碼。 您並非在部署期間傳遞密碼，而是使用 Azure Key Vault 預先儲存密碼，然後在部署期間擷取密碼。 若要深入了解如何從 Azure Key Vault 擷取祕密，並且在範本部署中使用祕密，請參閱：

> [!div class="nextstepaction"]
> [在範本部署中整合 Key Vault](./resource-manager-tutorial-use-key-vault.md)
