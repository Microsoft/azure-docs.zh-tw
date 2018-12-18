---
title: 建立用來部署加密儲存體帳戶的 Azure Resource Manager 範本 | Microsoft Docs
description: 您可以使用 Visual Studio Code 建立用來部署加密儲存體帳戶的範本。
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
ms.openlocfilehash: d48374d7919be3d141ea199e8238a220dbfe0332
ms.sourcegitcommit: 3150596c9d4a53d3650cc9254c107871ae0aab88
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47419520"
---
# <a name="tutorial-create-an-azure-resource-manager-template-for-deploying-an-encrypted-storage-account"></a>教學課程：建立用來部署加密儲存體帳戶的 Azure Resource Manager 範本

了解如何尋找完成 Azure Resource Manager 範本所需的資訊。

在此教學課程中，您會使用 Azure 快速入門範本中的基底範本，來建立 Azure 儲存體帳戶。  您可以使用範本參考文件來自訂基底範本，用以建立加密儲存體帳戶。

此教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 開啟快速入門範本
> * 了解範本
> * 編輯範本
> * 部署範本

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件

若要完成此文章，您需要：

* [Visual Studio Code](https://code.visualstudio.com/)。
* Resource Manager Tools 擴充功能。 若要安裝，請參閱[安裝 Resource Manager Tools 擴充功能](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)。

## <a name="open-a-quickstart-template"></a>開啟快速入門範本

此快速入門中使用的範本名為[建立標準儲存體帳戶](https://azure.microsoft.com/resources/templates/101-storage-account-create/)。 此範本會定義 Azure 儲存體帳戶資源。

1. 在 Visual Studio Code 中，選取 [檔案]>[開啟檔案]。
2. 在 [檔案名稱] 中，貼上下列 URL：

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
    ```
3. 選取 [開啟] 以開啟檔案。
4. 選取 [檔案]>[另存新檔]，在您的本機電腦上將檔案另存為 **azuredeploy.json**。

## <a name="understand-the-format"></a>了解格式

在 VS Code 中，將範本摺疊至根層級。 此時會有最簡單的結構，內含下列元素：

![最簡單的 Resource Manager 範本結構](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-simplest-structure.png)

* **$schema**：指定說明範本語言版本的 JSON 結構描述檔案所在的位置。
* **contentVersion**：為此元素指定任何值，用以記錄範本中的重大變更。
* **parameters**：指定執行部署以自訂資源部署時所提供的值。
* **variables**：指定範本中作為 JSON 片段以簡化範本語言運算式的值。
* **resources**：指定在資源群組中部署或更新的資源類型。
* **outputs**：指定部署後傳回的值。

## <a name="use-parameters-in-template"></a>使用範本中的參數

參數可提供針對特定環境量身訂做的值，讓您自訂部署。 您可以在設定儲存體帳戶的值時使用範本中定義的參數。

![Resource Manager 範本參數](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-parameters.png)

此範本中會定義兩個參數。 請注意，在 location.defaultValue 中使用了範本函式：

```json
"defaultValue": "[resourceGroup().location]",
```

resourceGroup() 函式會傳回代表目前資源群組的物件。 如需範本函式清單，請參閱 [Azure 資源管理員範本函式](./resource-group-template-functions.md)。

若要使用範本中定義的參數：

```json
"location": "[parameters('location')]",
"name": "[parameters('storageAccountType')]"
```

## <a name="use-variables-in-template"></a>使用範本中的變數

變數可讓您建構可用於整個範本的值。 變數有助於降低範本的複雜度。

![Resource Manager 範本變數](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-variables.png)

此範本會定義一個變數 *storageAccountName*。 在定義中，會使用兩個範本函式：

- **concat()**：串連字串。 如需詳細資訊，請參閱 [concat](./resource-group-template-functions-string.md#concat)。
- **uniqueString()**：根據當作參數提供的值，建立具決定性的雜湊字串。 每個 Azure 儲存體帳戶的名稱在整個 Azure 中都必須是唯一的。 此函式會提供唯一字串。 如需更多字串函式，請參閱[字串函式](./resource-group-template-functions-string.md)。

若要使用範本中定義的變數：

```json
"name": "[variables('storageAccountName')]"
```

## <a name="edit-the-template"></a>編輯範本

此教學課程的目標是要定義用來建立加密儲存體帳戶的範本。  範例範本只會建立基本的未加密儲存體帳戶。 若要尋找加密相關組態，您可以使用 Azure 儲存體帳戶的範本參考。

1. 瀏覽至 [Azure 範本](https://docs.microsoft.com/azure/templates/)。
2. 在 [依標題篩選] 中，輸入**儲存體帳戶**。
3. 選取 [參考/範本參考/儲存體/儲存體帳戶]，如下列螢幕擷取畫面所示：

    ![Resource Manager 範本參考儲存體帳戶](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-resources-reference-storage-accounts.png)

    resource-manager-template-resources-reference-storage-accounts
1. 瀏覽加密相關資訊。  
1. 在儲存體帳戶資源定義的屬性元素內，新增下列 JSON：

    ```json
    "encryption": {
        "keySource": "Microsoft.Storage",
        "services": {
            "blob": {
                "enabled": true
            }
        }
    }
    ```
    此組件可啟用 Blob 儲存體服務的加密功能。

從 Visual Studio Code 修改範本，使最終的資源元素顯示如下：

![Resource Manager 範本加密的儲存體帳戶資源](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-encrypted-storage-resources.png)

## <a name="deploy-the-template"></a>部署範本

請參閱 Visual Studio Code 快速入門中的[部署範本](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#deploy-the-template)一節，以取得部署程序。

下列螢幕擷取畫面顯示 CLI 命令列出了新建的儲存體帳戶，這表示已為 Blob 儲存體啟用加密。

![Azure Resource Manager 加密的儲存體帳戶](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-encrypted-storage-account.png)

## <a name="clean-up-resources"></a>清除資源

不再需要 Azure 資源時，可透過刪除資源群組來清除您所部署的資源。

1. 在 Azure 入口網站中，選取左側功能表中的 [資源群組]。
2. 在 [依名稱篩選] 欄位中輸入資源群組名稱。
3. 選取資源群組名稱。  您在資源群組中應該會看到共計六個資源。
4. 從頂端功能表中選取 [刪除資源群組]。

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何使用範本參考來自訂現有的範本。 若要了解如何建立多個儲存體帳戶執行個體，請參閱：

> [!div class="nextstepaction"]
> [建立多個執行個體](./resource-manager-tutorial-create-multiple-instances.md)
