---
title: 建立及管理 Azure Machine Learning 工作區
description: 了解如何在 Azure 入口網站中建立、檢視及刪除 Azure Machine Learning 工作區。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: shipatel
author: shivp950
ms.date: 09/24/2018
ms.openlocfilehash: 7d01a2e3ebd46315966c82a43a17ffc5b329b829
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46954342"
---
# <a name="create-and-manage-azure-machine-learning-workspaces"></a>建立及管理 Azure Machine Learning 工作區

在本文中，您將在 Azure 入口網站中為 [Azure Machine Learning Serivce](overview-what-is-azure-ml.md) 建立、檢視和刪除 [**Azure Machine Learning 工作區**](concept-azure-machine-learning-architecture.md#workspace)。  您也可以[使用 CLI](reference-azure-machine-learning-cli.md) 或[透過 Python 程式碼](http://aka.ms/aml-sdk)建立和刪除工作區。

若要建立工作區，您將需要 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="create-a-workspace"></a>建立工作區 

[!INCLUDE [aml-create-portal](../../../includes/aml-create-in-portal.md)]

## <a name="view-a-workspace"></a>檢視工作區

1. 在入口網站的左上角，選取 [所有服務]。 

1. 在 [所有服務] 篩選欄位中，鍵入 **Machine Learning 工作區**。  

   ![搜尋 Azure Machine Learning 工作區](media/how-to-manage-workspace/allservices-search1.png)

1. 在篩選結果中，選取 [Machine Learning 工作區] 以顯示您的工作區清單。 

   ![搜尋 Azure Machine Learning 工作區](media/how-to-manage-workspace/allservices-search.PNG)

1. 查看找到的工作區清單。 您可以根據訂用帳戶、資源群組和位置來篩選。  

   ![Azure Machine Learning 工作區清單](media/how-to-manage-workspace/allservices_view_workspace.PNG)

1. 選取您剛才建立的工作區來顯示其屬性。

   ![png](media/how-to-manage-workspace/allservices_view_workspace_full.PNG)

## <a name="delete-a-workspace"></a>刪除工作區

在您要刪除的工作區頂端，使用 [刪除] 按鈕。

  ![png](media/how-to-manage-workspace/delete-workspace.png)


## <a name="clean-up-resources"></a>清除資源 

[!INCLUDE [aml-delete-resource-group](../../../includes/aml-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

遵循完整的教學課程，了解如何使用工作區來建立、訓練和部署使用 Azure Machine Learning 服務的模型。

> [!div class="nextstepaction"]
> [教學課程：訓練模型](tutorial-train-models-with-aml.md)
