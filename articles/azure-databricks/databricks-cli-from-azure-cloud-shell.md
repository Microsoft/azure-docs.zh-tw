---
title: 從 Azure Cloud Shell 使用 Databricks CLI | Microsoft Docs
description: 了解如何從 Azure Cloud Shell 使用 Databricks CLI。
services: azure-databricks
documentationcenter: ''
author: nitinme
manager: cgronlun
editor: cgronlun
ms.service: azure-databricks
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 3ea4ebbd95237b50054fb0e344f260120d597ab5
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39225229"
---
# <a name="use-databricks-cli-from-azure-cloud-shell"></a>從 Azure Cloud Shell 使用 Databricks CLI

了解如何從 Azure Cloud Shell 使用 Databricks CLI，進而在 Databricks 上執行作業。

## <a name="prerequisites"></a>必要條件

* Azure Databricks 工作區和叢集。 如需指示，請參閱[開始使用 Azure Databricks](quickstart-create-databricks-workspace-portal.md)。 

* 在 Databricks 中設定個人存取權杖。 如需指示，請參閱[權杖管理](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management)。

## <a name="use-the-azure-cloud-shell"></a>使用 Azure Cloud Shell

1. 登入 [Azure 入口網站](https://portal.azure.com)。
 
2. 從右上角，按一下 [Cloud Shell] 圖示。

   ![啟動 Cloud Shell](./media/databricks-cli-from-azure-cloud-shell/launch-azure-cloud-shell.png "啟動 Azure Cloud Shell")

3. 請確定您選取適用於 Cloud Shell 環境的 **Bash**。 您可以從下拉式選項中選取，如下列螢幕擷取畫面所示。

   ![選取適用於 Cloud Shell 環境的 Bash](./media/databricks-cli-from-azure-cloud-shell/select-bash-for-shell.png "選取 Bash") 

4. 建立可以在其中安裝 Databricks CLI 的虛擬環境。 在下列程式碼片段中，您會建立名為 `databrickscli` 的虛擬環境。

       virtualenv -p /usr/bin/python2.7 databrickscli

5. 切換至您所建立的虛擬環境。

       source databrickscli/bin/activate

6. 安裝 Databricks CLI。

       pip install databricks-cli

7. 使用存取權杖設定 Databricks 的驗證，您必須已建立存取權杖，因為這已列為必要條件。 使用下列命令：

       databricks configure --token

    您會收到下列提示：

    * 首先，系統會提示您輸入 Databricks 主機。 請以 `https://eastus2.azuredatabricks.net` 格式輸入值。 在這裡，**美國東部 2** 是您用來建立 Azure Databricks 工作區的 Azure 地區。

    * 接下來，系統會提示您輸入權杖。 輸入您稍早建立的權杖。

完成這些步驟後，您就可以開始從 Azure Cloud Shell 使用 Databricks CLI。

## <a name="use-databricks-cli"></a>使用 Databricks CLI

您現在可以開始使用 Databricks CLI。 例如，執行下列命令，來列出您工作區中的所有 Databricks 叢集。

    databricks clusters list

您也可以使用下列命令來存取 Databricks 檔案系統 (DBFS)。

    databricks fs ls


如需命令的完整參考，請參閱 [Databricks CLI](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html)。


## <a name="next-steps"></a>後續步驟

* 若要深入了解 Azure CLI，請參閱 [Azure CLI 概觀](../cloud-shell/overview.md)
* 若要查看 Azure CLI 命令的清單，請參閱 [Azure CLI 參考](https://docs.microsoft.com/cli/azure/reference-index?view=azure-cli-latest)
* 若要查看 Databricks CLI 命令的清單，請參閱[Databricks CLI](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html)


