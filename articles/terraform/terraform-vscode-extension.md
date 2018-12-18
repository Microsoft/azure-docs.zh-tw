---
title: 安裝和使用 Azure Terraform Visual Studio Code 擴充功能
description: 了解如何在 Visual Studio Code 中安裝和使用 Azure Terraform 擴充功能。
services: terraform
ms.service: terraform
keywords: terraform, azure, devops, visual studio code, 擴充功能
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 08/31/2018
ms.openlocfilehash: be0140606302335884d62ba60c875a049949b495
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49353045"
---
# <a name="install-and-use-the-azure-terraform-visual-studio-code-extension"></a>安裝和使用 Azure Terraform Visual Studio Code 擴充功能

Microsoft Azure Terraform Visual Studio Code 擴充功能，旨在讓開發人員搭配 Azure 製作、測試及使用 Terraform，以提高生產力。 此擴充功能提供了 Terraform 命令支援、資源圖表視覺效果，並可將 CloudShell 整合到 Visual Studio Code 內。

在本文中，您將了解：
> [!div class="checklist"]
> * 使用 Terraform 來自動化並簡化 Azure 服務的佈建。
> * 安裝和使用適用於 Azure 服務的 Microsoft Terraform Visual Studio Code 擴充功能。
> * 使用 Visual Studio Code 來撰寫、規劃及執行 Terraform 方案。

## <a name="prerequisites"></a>必要條件
- **Azure 訂用帳戶**：如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

- **Terraform**：[安裝及設定 Terraform](/azure/virtual-machines/linux/terraform-install-configure)。

- **Visual Studio Code**：安裝適合您環境的 [Visual Studio Code](https://code.visualstudio.com/download) (英文) 版本。

## <a name="prepare-your-dev-environment"></a>準備您的開發環境

### <a name="install-git"></a>安裝 Git

若要完成此文章中的練習，您必須[安裝 Git](https://git-scm.com/)。

### <a name="install-hashicorp-terraform"></a>安裝 HashiCorp Terraform

依照 the HashiCorp [安裝 Terraform](https://www.terraform.io/intro/getting-started/install.html) 網頁上的指示執行，這些指示涵蓋：

- 下載 Terraform
- 安裝 Terraform
- 確認已正確安裝 Terraform

>[!Tip]
>務必依照有關設定 PATH 系統變數的指示執行。

### <a name="install-nodejs"></a>安裝 Node.js

若要在 Cloud Shell 中使用 Terraform，您必須[安裝 Node.js](https://nodejs.org/) 6.0+。

>[!NOTE]
>若要確認是否已安裝 Node.js，請開啟終端機視窗並輸入 `node -v`。

### <a name="install-graphviz"></a>安裝 GraphViz

若要使用 Terraform 視覺化函式，您必須[安裝 GraphViz](http://graphviz.org/)。

>[!NOTE]
>若要確認是否已安裝 GraphViz，請開啟終端機視窗並輸入 `dot -V`。

### <a name="install-the-azure-terraform-visual-studio-code-extension"></a>安裝 Azure Terraform Visual Studio Code 擴充功能

1. 啟動 Visual Studio Code。

1. 選取 [擴充功能]。

    ![[延伸模組] 按鈕](media/terraform-vscode-extension/tf-vscode-extensions-button.png)

1. 使用 [在 Marketplace 中搜尋擴充功能] 文字方塊來搜尋 Azure Terraform 延伸模組：

    ![在 Marketplace 中搜尋 Visual Studio Code 擴充功能](media/terraform-vscode-extension/tf-search-extensions.png)

1. 選取 [安裝]。

    >[!NOTE]
    >當您針對 Azure Terraform 擴充功能選取 [安裝] 時，Visual Studio Code 將會自動安裝「Azure 帳戶」擴充功能。 Azure Account 是 Azure Terraform 延伸模組的相依性檔案，Azure Terraform 延伸模組使用它來執行 Azure 訂用帳戶驗證與 Azure 相關程式碼延伸模組。

#### <a name="verify-the-terraform-extension-is-installed-in-visual-studio-code"></a>確認是否已正確在 Visual Studio Code 中安裝 Terraform 延伸模組

1. 選取 [擴充功能]。

1. 在搜尋文字方塊中輸入 `@installed`。

    ![已安裝的延伸模組](media/terraform-vscode-extension/tf-installed-extensions.png)

Azure Terraform 延伸模組將會出現在已安裝的延伸模組清單中。

![已安裝的 Terraform 延伸模組](media/terraform-vscode-extension/tf-installed-terraform-extension-button.png)

您現在可以從 Visual Studio Code，在您的 Cloud Shell 環境中執行所有支援的 Terraform 命令。

## <a name="exercise-1-basic-terraform-commands-walk-through"></a>練習 1: 基本 Terraform 命令逐步解說

在此練習中，您會建立及執行基本 Terraform 設定檔，此設定檔會佈建新的 Azure 資源群組。

### <a name="prepare-a-test-plan-file"></a>準備測試方案檔案

1. 在 Visual Studio Code 中，從功能表列選取 [檔案] > [新增檔案]。

1. 在瀏覽器中瀏覽到 [Terraform azurerm_resource_group 頁面](https://www.terraform.io/docs/providers/azurerm/r/resource_group.html#)，並複製 [使用方式範例] 程式碼區塊中的程式碼：

    ![使用方式範例](media/terraform-vscode-extension/tf-azurerm-resource-group-example-usage.png)

1. 將複製的程式碼貼到您在 Visual Studio Code 中所建立的新檔案內。

    ![貼上範例使用方式程式碼](media/terraform-vscode-extension/tf-paste-example-usage-code.png)

    >[!NOTE]
    >您可以變更資源群組的**名稱**值，但它在您的 Azure 訂用帳戶中必須是唯一的。

1. 從功能表列，選取 [檔案] > [另存新檔]。

1. 在 [另存新檔] 對話方塊中，巡覽至您選擇的位置，然後選取 [新增資料夾]。 (將新資料夾的名稱變更為比*新資料夾*更具描述性的名稱)。

    >[!NOTE]
    >在此範例中，我們將資料夾命名為 TERRAFORM-TEST-PLAN。

1. 確定您的新資料夾已醒目提示 (已選取)，然後選取 [開啟]。

1. 在 [另存新檔] 對話方塊中，將檔案的預設名稱變更為 *main.tf*。

    ![另存為 main.tf](media/terraform-vscode-extension/tf-save-as-main.png)

1. 選取 [ **儲存**]。
- 在功能表列中，選取 [檔案] > [開啟資料夾]。 瀏覽並選取您建立的新資料夾。

### <a name="run-terraform-init-command"></a>執行 Terraform *init* 命令

1. 啟動 Visual Studio Code。

1. 從 Visual Studio Code 功能表列，選取 [檔案] > [開啟資料夾...]，然後尋找並選取您的 *main.tf* 檔案。

    ![main.tf 檔案](media/terraform-vscode-extension/tf-main-tf.png)

1. 從功能表列，選取 [檢視] > [命令選擇區...] > [Azure Terraform：初始化]。

1. 要求您確認時，請選取 [確定]。

    ![是否要開啟 Cloud Shell?](media/terraform-vscode-extension/tf-do-you-want-to-open-cloud-shell.png)

1. 第一次從新資料夾啟動 Cloud Shell 時，系統會要求您設定 Web 應用程式。 選取 [開啟] 。

    ![第一次啟動 Cloud Shell](media/terraform-vscode-extension/tf-first-launch-of-cloud-shell.png)

1. [歡迎使用 Azure Cloud Shell] 頁面隨即開啟。 選 [Bash] 或 [PowerShell]。

    ![歡迎使用 Azure Cloud Shell](media/terraform-vscode-extension/tf-welcome-to-azure-cloud-shell.png)

    >[!NOTE]
    >在此範例中，我們選取 [Bash (Linux)]。

1. 若您尚未設定 Azure 儲存體帳戶，會出現下列畫面。 選取 [建立儲存體]。

    ![您未掛接任何儲存體](media/terraform-vscode-extension/tf-you-have-no-storage-mounted.png)

1. Azure Cloud Shell 會在您先前選取的殼層中啟動，並顯示剛為您建立之雲端磁碟機的資訊。

    ![已建立您的雲端磁碟機](media/terraform-vscode-extension/tf-your-cloud-drive-has-been-created-in.png)

1. 您現在可以結束 Cloud Shell

1. 從功能表列，選取 [檢視] > [命令選擇區] > [Azure Terraform：初始化]。

    ![Terraform 已成功初始化](media/terraform-vscode-extension/tf-terraform-has-been-successfully-initialized.png)

### <a name="visualize-the-plan"></a>將方案視覺化

在此教學課程稍早，您已安裝 GraphViz。 Terraform 可以使用 GraphViz 來產生設定或執行計畫的視覺表示。 Azure Terraform Visual Studio Code 擴充功能會透過 visualize 命令來實作此功能。

- 從功能表列，選取 [檢視] > [命令選擇區] > [Azure Terraform：視覺化]。

    ![將方案視覺化](media/terraform-vscode-extension/tf-graph.png)

### <a name="run-terraform-plan-command"></a>執行 Terraform *plan* 命令

Terraform *plan* 命令是用來檢查變更集的執行計畫是否會執行您要執行的動作。

>[!NOTE]
>Terraform *plan* 不會對您的實際 Azure 資源進行任何變更。 若要實際變更您方案中包含的項目，我們會使用 Terraform *apply* 命令。

- 從功能表列，選取 [檢視] > [命令選擇區] > [Azure Terraform：方案]。

    ![Terraform plan](media/terraform-vscode-extension/tf-terraform-plan.png)

### <a name="run-terraform-apply-command"></a>執行 Terraform *apply* 命令

滿意 Terraform *plan*, 的結果之後，您可以執行 *apply* 命令。

1. 從功能表列，選取 [檢視] > [命令選擇區] > [Azure Terraform：套用]。

    ![Terraform apply](media/terraform-vscode-extension/tf-terraform-apply.png)

1. 輸入 `yes`。

    ![Terraform 套用 > 是](media/terraform-vscode-extension/tf-terraform-apply-yes.png)

### <a name="verify-your-terraform-plan-was-executed"></a>確認您的 Terraform 方案已執行

查看是否已成功建立您的新 Azure 資源群組：

1. 開啟 Azure 入口網站。

1. 在左側瀏覽窗格中，選取 [資源群組]。

    ![確認您的新資源群組是否存在](media/terraform-vscode-extension/tf-verify-resource-group-created.png)

您的新資源群組應該會列在 [名稱] 欄。

>[!NOTE]
>現在您可以將您的 Azure 入口網站視窗維持開啟，們將會在下一個步驟中使用它。

### <a name="run-terraform-destroy-command"></a>執行 Terraform *destroy* 命令

1. 從功能表列，選取 [檢視] > [命令選擇區] > [Azure Terraform：終結]。

    ![Terraform 摧毀](media/terraform-vscode-extension/tf-terraform-destroy.png)

1. 輸入 yes 。

    ![Terraform 摧毀 > 是](media/terraform-vscode-extension/tf-terraform-destroy-yes.png)

### <a name="verify-your-resource-group-was-destroyed"></a>確認是否已摧毀您的資源群組

確認 Terraform 是否已成功摧毀您的資源群組：

1. 在 Azure 入口網站的 [資源群組] 頁面上，選取 [重新整理]。

1. 將不再列出您的資源群組。

    ![確認已摧毀資源群組](media/terraform-vscode-extension/tf-refresh-resource-groups-button.png)

## <a name="exercise-2-terraform-compute-module"></a>練習 2：Terraform *compute* 模組

在此練習中，您將了解如何將 Terraform compute 模組載入到 Visual Studio Code 環境。

### <a name="clone-the-terraform-azurerm-compute-module"></a>複製 terraform-azurerm-compute 模組

1. 使用[此連結](https://github.com/Azure/terraform-azurerm-compute)來存取 GitHub 上的 Terraform Azure Rm Compute 模組。

1. 選取 [複製或下載] 。

    ![複製或下載](media/terraform-vscode-extension/tf-clone-with-https.png)

    >[!NOTE]
    >在此範例中，我們的資料夾是命名為 *terraform-azurerm-compute*。

### <a name="open-the-folder-in-visual-studio-code"></a>在 Visual Studio Code 中開啟資料夾

1. 啟動 Visual Studio Code。

1. 從功能表列，選取 [檔案] > [開啟資料夾]，然後巡覽並選取您在上一個步驟中所建立的資料夾。

    ![terraform-azurerm-compute 資料夾](media/terraform-vscode-extension/tf-terraform-azurerm-compute-folder.png)

### <a name="initialize-terraform"></a>初始化 Terraform

您必須先下載兩個 Azure 提供者的外掛程式，才能開始在 Visual Studio Code 中使用 Terraform 命令：random 與 azurerm。

1. 在 Visual Studio Code IDE 的 [終端機] 窗格中，輸入 `terraform init`。

    ![terraform init 命令](media/terraform-vscode-extension/tf-terraform-init-command.png)

1. 輸入 `az login`，按下 **<Enter**，然後依照畫面上的指示進行。

### <a name="module-test-lint"></a>模組清單：*lint*

1. 請從功能表列，選取 [檢視] > [命令選擇區] > [Azure Terraform：執行測試]。

1. 從測試類型選項清單，選取 [lint]。

    ![選取測試類型](media/terraform-vscode-extension/tf-select-type-of-test-lint.png)

1. 要求您確認時，請選取 [確定]，然後依照畫面上的指示進行。

    ![是否要開啟 CloudShell?](media/terraform-vscode-extension/tf-do-you-want-to-open-cloudshell-small.png)

>[!NOTE]
>當您執行 **lint** 或**端對端**測試時，Azure 會使用容器服務來佈建測試機器以執行實際測試。 因此，您的測試結果可能需要數分鐘才能傳回。

不久之後，您會看到 [終端機] 窗格顯示一個清單，如此範例所示：

![Lint 測試結果](media/terraform-vscode-extension/tf-lint-test-results.png)

### <a name="module-test-end-to-end"></a>模組測試：*端對端*

1. 請從功能表列，選取 [檢視] > [命令選擇區] > [Azure Terraform：執行測試]。

1. 從測試類型選項清單，選取 [端對端]。

    ![選取測試類型](media/terraform-vscode-extension/tf-select-type-of-test-end-to-end.png)

1. 要求您確認時，請選取 [確定]，然後依照畫面上的指示進行。

    ![是否要開啟 CloudShell?](media/terraform-vscode-extension/tf-do-you-want-to-open-cloudshell-small.png)

>[!NOTE]
>當您執行 **lint** 或**端對端**測試時，Azure 會使用容器服務來佈建測試機器以執行實際測試。 因此，您的測試結果可能需要數分鐘才能傳回。

不久之後，您會看到 [終端機] 窗格顯示一個清單，如此範例所示：

![端對端測試結果](media/terraform-vscode-extension/tf-end-to-end-test-results.png)

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [Azure (和其他支援的提供者) 可用的 Terraform 模組清單](https://registry.terraform.io/) (英文)