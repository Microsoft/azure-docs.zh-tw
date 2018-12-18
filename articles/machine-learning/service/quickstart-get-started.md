---
title: 快速入門：在 Azure 入口網站中建立機器學習服務工作區 - Azure Machine Learning
description: 使用 Azure 入口網站來建立 Azure Machine Learning 工作區。 此工作區是雲端中使用 Azure Machine Learning 來實驗、訓練及部署機器學習模型的基礎區塊。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: quickstart
ms.reviewer: sgilley
author: rastala
ms.author: roastala
ms.date: 09/24/2018
ms.openlocfilehash: 14bd85a23e2630a1cf2a8b5621d669c4c6748168
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376613"
---
# <a name="quickstart-use-the-azure-portal-to-get-started-with-azure-machine-learning"></a>快速入門：利用 Azure 入口網站開始使用 Azure Machine Learning

在本快速入門中，您會使用 Azure 入口網站來建立 Azure Machine Learning 工作區。 此工作區是雲端中使用 Machine Learning 來實驗、訓練及部署機器學習模型的基礎區塊。 

在本教學課程中，您：

* 在您的 Azure 訂用帳戶中建立工作區。
* 試著在 Azure Notebook 中使用 Python 並記錄多個反覆項目中的值。
* 在工作區中檢視記錄的值。

下列 Azure 資源可在區域內取得時，就會自動新增至您的工作區：

  - [Azure Container Registry](https://azure.microsoft.com/services/container-registry/)
  - [Azure 儲存體](https://azure.microsoft.com/services/storage/)
  - [Azure Application Insights](https://azure.microsoft.com/services/application-insights/) 
  - [Azure 金鑰保存庫](https://azure.microsoft.com/services/key-vault/)

您所建立的資源可用來作為其他 Machine Learning 服務教學課程和操作說明文章的必要條件。 如同使用其他 Azure 服務，對於與 Machine Learning 相關聯的特定資源有一些限制。 例如，Azure Batch AI 叢集大小。 如需預設限制以及如何增加配額的資訊，請參閱[這篇文章](how-to-manage-quotas.md)。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。


## <a name="create-a-workspace"></a>建立工作區 

[!INCLUDE [aml-create-portal](../../../includes/aml-create-in-portal.md)]

在工作區頁面上，選取 `Explore your Azure Machine Learning service workspace`。

 ![探索工作區](./media/quickstart-get-started/explore_aml.png)


## <a name="use-the-workspace"></a>使用工作區

現在會看到工作區如何協助您管理機器學習服務指令碼。 在本節中，您可：

* 在 Azure Notebooks 中開啟 Notebook。
* 執行程式碼來建立一些記錄的值。
* 在工作區中檢視記錄的值。

這個範例示範工作區如何協助您追蹤指令碼中所產生的資訊。 

### <a name="open-a-notebook"></a>開啟 Notebook 

Azure Notebooks 會為 Jupyter Notebook 提供免費的雲端平台，其會利用您執行 Machine Learning 所需的一切來預先設定。  

選取 `Open Azure Notebooks` 嘗試您的第一個實驗。

 ![開啟 Azure Notebooks](./media/quickstart-get-started/explore_ws.png)

您的組織可能需要[系統管理員的同意](https://notebooks.azure.com/help/signing-up/work-or-school-account/admin-consent)，您才能登入。

登入之後，即會開啟新索引標籤並出現 `Clone Library` 提示。 選取 `Clone`


### <a name="run-the-notebook"></a>執行 Notebook

除了兩個 Notebook，您還會看見 `config.json` 檔案。 此組態檔包含您建立的工作區相關資訊。  

選取 `01.run-experiment.ipynb` 來開啟 Notebook。

若要一次執行一個資料格，請使用 `Shift`+`Enter`。 或者選取 `Cells` > `Run All` 來執行整個 Notebook。 當您在資料格旁邊看到 [*] 時，表示它正在執行中。 該資料格的程式碼完成後，就會出現一個數字。

系統會提示您登入。 複製訊息中的程式碼。 然後選取連結，並將程式碼貼到新視窗中。 請確定不會複製程式碼前後的空格。 請使用您在 Azure 入口網站中使用的相同帳戶登入。

 ![登入](./media/quickstart-get-started/login.png)

在 Notebook 中，第二個資料格會從 `config.json` 讀取以連線到您的工作區。
```
ws = Workspace.from_config()
```

程式碼的第三個資料格會開始進行名為 "my-first-experiment" 的實驗。 使用此名稱來查看要返回您工作區之執行的相關資訊。

```
experiment = Experiment(workspace_object=ws, name = "my-first-experiment")
```

在 Notebook 的最後一個資料格中，請注意要寫入到記錄檔的值。

```
# Log final results
run.log("Final estimate: ",pi_estimate)
run.log("Final error: ",math.pi-pi_estimate)
```

執行程式碼之後，您可以在工作區中檢視這些值。

## <a name="view-logged-values"></a>檢視記錄的值

在 Notebook 中執行所有資料格之後，請回到入口網站頁面。  

選取 `View Experiments`。

![檢視實驗](./media/quickstart-get-started/view_exp.png)

關閉 `Reports` 快顯。

選取 `my-first-experiment`。

查看您剛執行的執行相關資訊。 向下捲動頁面以找出執行表格。 選取執行號碼連結。

 ![執行歷程記錄連結](./media/quickstart-get-started/report.png)

您會看到已自動建立所記錄值的繪圖。  

   ![檢視歷程記錄](./media/quickstart-get-started/plots.png)

用於大致估計 pi 的程式碼會使用隨機值，因此您的圖會顯示不同的值。

## <a name="clean-up-resources"></a>清除資源 

[!INCLUDE [aml-delete-resource-group](../../../includes/aml-delete-resource-group.md)]

您也可以保留資源群組，但刪除單一工作區。 顯示工作區屬性，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

您已建立了進行實驗和部署模型所需的資源。 您也在 Notebook 中執行了一些程式碼。 而且在雲端的工作區中探索了該程式碼的執行歷程記錄。

如需深入的工作流程體驗，請按照 Machine Learning 教學課程來定型和部署模型。  

> [!div class="nextstepaction"]
> [教學課程：將影像分類模型定型](tutorial-train-models-with-aml.md)
