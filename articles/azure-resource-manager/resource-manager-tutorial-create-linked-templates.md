---
title: 建立連結的 Azure Resource Manager 範本 | Microsoft Docs
description: 了解如何建立連結的 Azure Resource Manager 範本以用於建立虛擬機器
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 09/07/2018
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: d25ca14b78465a6c4fec7e90bc20436e3ca553fc
ms.sourcegitcommit: 3150596c9d4a53d3650cc9254c107871ae0aab88
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47419622"
---
# <a name="tutorial-create-linked-azure-resource-manager-templates"></a>教學課程：建立連結的 Azure Resource Manager 範本

了解如何建立連結的 Azure Resource Manager 範本。 使用連結的範本，您可以讓一個範本呼叫另一個範本。 此功能非常適合用來將範本模組化時。 在此教學課程中，您會使用與在[教學課程：使用 Resource Manager 範本建立多個資源執行個體](./resource-manager-tutorial-create-multiple-instances.md)中使用之範本相同的範本，此範本會建立虛擬機器、虛擬網路與其他相依資源，包括儲存體帳戶。 您會將儲存體帳戶資源分散到連結的範本中。

此教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 開啟快速入門範本
> * 建立連結的範本
> * 上傳連結的範本
> * 連結到連結的範本
> * 設定相依性
> * 從連結的範本取得值
> * 部署範本

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件

若要完成此文章，您需要：

* [Visual Studio Code](https://code.visualstudio.com/)。
* Resource Manager Tools 擴充功能。  請參閱[安裝延伸模組](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)。
* 完成[教學課程：使用 Resource Manager 範本建立多個資源執行個體](./resource-manager-tutorial-create-multiple-instances.md)。

## <a name="open-a-quickstart-template"></a>開啟快速入門範本

Azure 快速入門範本是 Resource Manager 範本的存放庫。 您可以尋找範例範本並加以自訂，而不要從頭建立範本。 此教學課程中使用的範本名為[部署簡單的 Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/)。 這與在[教學課程：使用 Resource Manager 範本建立多個資源執行個體](./resource-manager-tutorial-create-multiple-instances.md)中使用的範本相同。 您儲存相同範本的兩個複本以做為：

- **主範本**：建立所有資源，儲存體帳戶除外。
- **連結的範本**：建立儲存體帳戶。

1. 在 Visual Studio Code 中，選取 [檔案]>[開啟檔案]。
2. 在 [檔案名稱] 中，貼上下列 URL：

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```
3. 選取 [開啟] 以開啟檔案。
4. 選取 [檔案]>[另存新檔]，以名稱 **azuredeploy.json** 將檔案的複本儲存至您的本機電腦。
5. 選取 [檔案]>[另存新檔] 以使用名稱 **linkedTemplate.json** 建立該檔案的複本。

## <a name="create-the-linked-template"></a>建立連結的範本

連結的範本會建立儲存體帳戶。 連結的範本幾乎與建立儲存體帳戶的獨立範本一模一樣。 在此教學課程中，連結的範本必須將一個值傳遞回主範本。 此值是在 `outputs` 元素中所定義。

1. 在 Visual Studio Code 中開啟 linkedTemplate.json (若尚未開啟)。
2. 進行下列變更：

    - 移除所有資源，儲存體帳戶除外。 您必須移除總共四個資源。
    - 更新 **outputs** 元素，讓它看起來像這樣：

        ```json
        "outputs": {
            "storageUri": {
                "type": "string",
                "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
              }
        }
        ```
        主範本中的虛擬機器資源定義需要 **storageUri**。  您必須將該值傳遞回主範本做為輸出值。
    - 移除從未使用的參數。 這些參數下方有綠色波浪線。 您應該只保留一個稱為 **location** 的參數。
    - 移除 **variables** 元素。 它們在此教學課程中是不必要的。
    - 新增稱為 **storageAccountName** 的參數。 儲存體帳戶名稱會以參數形式從主範本傳遞到連結的範本。

    當您完成時，範本看起來應該像這樣：

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "storageAccountName":{
            "type": "string",
            "metadata": {
              "description": "Azure Storage account name."
            }
          },
          "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
              "description": "Location for all resources."
            }
          }
        },
        "resources": [
          {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[parameters('storageAccountName')]",
            "apiVersion": "2016-01-01",
            "location": "[parameters('location')]",
            "sku": {
              "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
          }
        ],
        "outputs": {
            "storageUri": {
                "type": "string",
                "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
              }
        }
    }
    ```
3. 儲存變更。

## <a name="upload-the-linked-template"></a>上傳連結的範本

範本必須可從您執行部署的位置存取。 此位置可以是 Azure 儲存體帳戶、Github 或 Dropbox。 若您的範本包含機密資訊，請務必保護其存取。 在此教學課程中，您使用如同您在[教學課程：使用 Resource Manager 範本建立多個資源執行個體](./resource-manager-tutorial-create-multiple-instances.md)中使用的 Cloud Shell 部署方法。 主範本 (azuredeploy.json) 已上傳到殼層。 連結的範本 (linkedTemplate.json) 必須在某處共用。  為減少此教學課程中的工作量，在上一節中定義的連結的範本已上傳到 [Azure 儲存體帳戶](https://armtutorials.blob.core.windows.net/linkedtemplates/linkedStorageAccount.json)。

## <a name="call-the-linked-template"></a>呼叫連結的範本

主範本的名稱是 azuredeploy.json。

1. 在 Visual Studio Code 中開啟 azuredeploy.json (若尚未開啟)。
2. 從範本刪除儲存體帳戶資源定義。
3. 新增下列 json 程式碼片段到您儲存體帳戶定義存放所在之處：

    ```json
    {
      "apiVersion": "2017-05-10",
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "properties": {
          "mode": "Incremental",
          "templateLink": {
              "uri":"https://armtutorials.blob.core.windows.net/linkedtemplates/linkedStorageAccount.json"
          },
          "parameters": {
              "storageAccountName":{"value": "[variables('storageAccountName')]"},
              "location":{"value": "[parameters('location')]"}
          }
      }
    },
    ```

    請注意這些詳細資料：

    - 主範本中的 `Microsoft.Resources/deployments` 資源會用來連結到另一個範本。
    - `deployments` 資源的名稱是 `linkedTemplate`。 此名稱會用於[設定相依性](#configure-dependency)。  
    - 呼叫連結的範本時，您只能使用[增量](./deployment-modes.md)部署模式。
    - `templateLink/uri` 包含連結的範本 URI。 連結的範本已上傳到共用儲存體帳戶。 若您將範本上傳到網際網路上的另一個位置，您可以更新 URI。
    - 使用 `parameters` 將值從主範本傳遞到連結的範本。
4. 儲存變更。

## <a name="configure-dependency"></a>設定相依性

回想一下[教學課程：使用 Resource Manager 範本建立多個資源執行個體](./resource-manager-tutorial-create-multiple-instances.md)，虛擬機器資源相依於儲存體帳戶：

![Azure Resource Manager 範本相依性圖表](./media/resource-manager-tutorial-create-linked-templates/resource-manager-template-visual-studio-code-dependency-diagram.png)

因為現在儲存體帳戶是在連結的範本中所定義，您必須更新下列 `Microsoft.Compute/virtualMachines` 資源的兩個元素。

- 重新設定 `dependOn` 元素。 儲存體帳戶定義已從連結的範本移除。
- 重新設定 `properties/diagnosticsProfile/bootDiagnostics/storageUri` 元素。 在 [建立連結的範本](#create-the-linked-template) 中，您已新增輸出值：

    ```json
    "outputs": {
        "storageUri": {
            "type": "string",
            "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
            }
    }
    ```
    主範本需要此值。

1. 在 Visual Studio Code 中開啟 azuredeploy.json (若尚未開啟)。
2. 展開虛擬機器資源定義並更新**dependsOn**，如下列螢幕擷取化驗所示：

    ![Azure Resource Manager 連結的範本設定相依性 ](./media/resource-manager-tutorial-create-linked-templates/resource-manager-template-linked-templates-configure-dependency.png)
    
    「連結的範本」是部署資源的名稱。  
3. 更新 **properties/diagnosticsProfile/bootDiagnostics/storageUri**，如上一個螢幕擷取畫面所示。

如需詳細資訊，請參閱[在部署 Azure 資源時使用連結與巢狀的範本](./resource-group-linked-templates.md)。

## <a name="deploy-the-template"></a>部署範本

請參閱[部署範本](./resource-manager-tutorial-create-multiple-instances.md#deploy-the-template)一節，以了解部署程序。

## <a name="clean-up-resources"></a>清除資源

不再需要 Azure 資源時，可透過刪除資源群組來清除您所部署的資源。

1. 在 Azure 入口網站中，選取左側功能表中的 [資源群組]。
2. 在 [依名稱篩選] 欄位中輸入資源群組名稱。
3. 選取資源群組名稱。  您在資源群組中應該會看到共計六個資源。
4. 從頂端功能表中選取 [刪除資源群組]。

## <a name="next-steps"></a>後續步驟

在此教學課程中，您會開發並部署連結的範本。 若要了解如何跨多個區域部署 Azure 資源，以及如何使用安全的部署實務，請參閱


> [!div class="nextstepaction"]
> [使用 Azure 部署管理員](./deployment-manager-tutorial.md)

