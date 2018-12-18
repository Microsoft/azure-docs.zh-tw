---
title: Azure Terraform VS Code 模組產生器
description: 了解如何使用 Yeoman 來建立 Terraform 基底範本。
services: terraform
ms.service: terraform
keywords: terraform, devops, virtual machine, azure, yeoman, 虛擬機器
author: v-mavick
manager: jeconnoc
ms.author: v-mavick
ms.topic: tutorial
ms.date: 09/12/2018
ms.openlocfilehash: 513b123c44cf2cd37cf81a91e0d2da9599eb1fcd
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47396199"
---
# <a name="create-a-terraform-base-template-using-yeoman"></a>使用 Yeoman 建立 Terraform 基底範本

[Terraform](https://docs.microsoft.com/azure/terraform/
) 能提供在 Azure 上輕鬆建立基礎結構的方法。 [Yeoman](http://yeoman.io/) 能大幅簡化模組開發人員建立 Terraform 模組的作業，並提供絕佳的*最佳做法*架構。

在此文章中，您會了解如何使用 Yeoman 模組產生器來建立基底 Terraform 範本。

## <a name="prerequisites"></a>先決條件

- 執行 Windows 10、Linux 或 macOS 10.10+ 的電腦。
- **Azure 訂用帳戶**：如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。
- **Visual Studio Code**：我們將會使用 [Visual Studio Code](https://www.bing.com/search?q=visual+studio+code+download&form=EDGSPH&mkt=en-us&httpsmsn=1&refig=dffc817cbc4f4cb4b132a8e702cc19a3&sp=3&ghc=1&qs=LS&pq=visual+studio+code&sk=LS1&sc=8-18&cvid=dffc817cbc4f4cb4b132a8e702cc19a3&cc=US&setlang=en-US) 來檢視由 Yeoman 產生器所建立的檔案。 不過，您也可以使用自己偏好的程式碼編輯器。
- **Terraform**：您需要安裝 [Terraform](https://docs.microsoft.com/azure/virtual-machines/linux/terraform-install-configure ) 以執行由 Yeoman 所建立的模組。
- **Docker**：我們將會使用 [Docker](https://www.docker.com/get-started) 以執行由 Yeoman 產生器所建立的模組。 (如果想要的話，您也可以使用 Ruby 來代替 Docker 執行範例模組)。
- **Go 程式設計語言**：您需要安裝 [Go](https://golang.org/)，因為由 Yeoman 所產生的測試案例將會以 Go 撰寫。

>[!NOTE]
>此教學課程中大部分的程序都牽涉到命令列輸入。 這裡所描述的步驟皆適用於所有作業系統與命令列工具。 在下列範例中，我們選擇使用 PowerShell。 不過，您可以選擇使用數個其他替代方案，例如 Git Bash、Windows 命令提示字元，或是 Linux 或 macOS 命令列命令。

## <a name="prepare-your-environment"></a>準備您的環境

### <a name="install-nodejs"></a>安裝 Node.js

若要在 Cloud Shell 中使用 Terraform，您必須[安裝 Node.js](https://nodejs.org/en/download/) 6.0+。

>[!NOTE]
>若要確認是否已安裝 Node.js，請開啟終端機視窗並輸入 `node --version`。

### <a name="install-yeoman"></a>安裝 Yeoman

從命令提示字元，輸入 `npm install -g yo`。

![安裝 Yeoman](media/terraform-vscode-module-generator/ymg-npm-install-yo.png)

### <a name="install-the-yeoman-template-for-terraform-module"></a>安裝適用於 Terraform 模組的 Yeoman 範本

從命令提示字元，輸入 `npm install -g generator-az-terra-module`。

![安裝 generator-az-terra-module](media/terraform-vscode-module-generator/ymg-pm-install-generator-module.png)

>[!NOTE]
>若要確認是否已安裝 Yeoman，請從終端機視窗輸入 `yo --version`。

### <a name="create-an-empty-folder-to-hold-the-yeoman-generated-module"></a>建立空白資料夾以保存 Yeoman 所產生的模組

Yeoman 範本會在**目前的目錄**中產生檔案。 因此，您必須建立目錄。

>[!Note]
>您必須將這個空白目錄放在 $GOPATH/src 底下。您可以在[這裡](https://github.com/golang/go/wiki/SettingGOPATH)找到完成此作業的指示。

從命令提示字元：

1. 瀏覽到您要新增空白目錄的父目錄。
1. 輸入 `mkdir <new-directory-name>`。

    >[!NOTE]
    >將 <new-directory-name> 取代為新目錄的名稱。 在此範例中，我們將新目錄命名為 `GeneratorDocSample`。

    ![mkdir](media/terraform-vscode-module-generator/ymg-mkdir-GeneratorDocSample.png)

1. 輸入 `cd <new directory's name>` 以瀏覽至新目錄，然後按 **Enter**。

    ![瀏覽到您的新目錄](media/terraform-vscode-module-generator/ymg-cd-GeneratorDocSample.png)

    >[!NOTE]
    >若要確認此目錄是空的，請輸入 `ls`。 這個命令所輸出的結果不應該列出任何檔案。

## <a name="create-a-base-module-template"></a>建立基底模組範本

從命令提示字元：

1. 輸入 `yo az-terra-module`。

1. 依照螢幕上的指示提供下列資訊：

    - *Terraform module project Name* \(Terraform 模組專案名稱\)

        ![專案名稱](media/terraform-vscode-module-generator/ymg-project-name.png)       

        >[!NOTE]
        >在此範例中，我們輸入的是 `doc-sample-module`。

    - *Would you like to include the Docker image file?* \(是否要包含 Docker 映像檔案？\)

        ![包含 Docker 映像檔案？](media/terraform-vscode-module-generator/ymg-include-docker-image-file.png) 

        >[!NOTE]
        >輸入 `y`。 如果您輸入 **n**，產生的模組程式碼將僅支援以原生模式執行。

3. 輸入 `ls` 以檢視所建立的檔案。

    ![列出所建立檔案](media/terraform-vscode-module-generator/ymg-ls-GeneratorDocSample-files.png)

## <a name="review-the-generated-module-code"></a>檢閱所產生的模組程式碼

1. 啟動 Visual Studio Code

1. 從功能表列，選取 [檔案] > [開啟資料夾]，然後選取您所建立的資料夾。

    ![Visual Studio Code](media/terraform-vscode-module-generator/ymg-open-in-vscode.png)

讓我們看看由 Yeoman 模組產生器所建立的一些檔案。

>[!Note]
>在此文章中，我們將會使用由 Yeoman 模組產生器所建立的 main.tf、variables.tf 與 outputs.tf 檔案。 不過，當您自行建立模組時，您將會編輯這些檔案以配合您 Terraform 模組的功能。 如需這些檔案及其使用方式的更深入討論，請參閱 [Terraform 模組中的 Terratest](https://mseng.visualstudio.com/VSJava/_git/Terraform?path=%2FTerratest%20Introduction.md&version=GBmaster) \(英文\)。

### <a name="maintf"></a>main.tf

定義名為 *random-shuffle* 的模組。 輸入是 *string_list*。 輸出是排列的計數。

### <a name="variablestf"></a>variables.tf

定義模組所使用的輸入與輸出變數。

### <a name="outputstf"></a>outputs.tf

定義模組的輸出內容。 在這裡，該內容為由內建的 Terraform 模組 **random_shuffle** 所傳回的值。

### <a name="rakefile"></a>Rakefile

定義建置步驟。 這些步驟包括：

- **build**：驗證 main.tf 檔案的格式。
- **unit**：產生的模組基本架構不會包含單元測試的程式碼。 如果您想要指定單元測試案例，便必須在此新增該程式碼。
- **e2e**：執行模組的端對端測試。

### <a name="test"></a>test

- 測試案例是以 Go 撰寫的。
- test 中的所有程式碼都是端對端測試。
- 端對端測試會嘗試使用 Terraform 來佈建定義於 **fixture** 底下的所有項目，然後將 **template_output.go** 程式碼中的輸出與預先定義的預期值比較。
- **Gopkg.lock** 與 **Gopkg.toml**：定義您的相依性。 

## <a name="test-the-module-using-docker"></a>使用 Docker 來測試模組

>[!NOTE]
>在我們的範例中，我們是以本機模組的形式執行模組，而沒有實際碰觸到 Azure。

### <a name="confirm-docker-is-installed-and-running"></a>確認 Docker 已安裝且正在執行

從命令提示字元，輸入 `docker version`。

![Docker 版本](media/terraform-vscode-module-generator/ymg-docker-version.png)

輸出結果確認 Docker 已安裝。

若要確認 Docker 是否正在執行，請輸入 `docker info`。

![Docker 資訊](media/terraform-vscode-module-generator/ymg-docker-info.png)

### <a name="set-up-a-docker-container"></a>設定 Docker 容器

1. 從命令提示字元，輸入

    `docker build --build-arg BUILD_ARM_SUBSCRIPTION_ID= --build-arg BUILD_ARM_CLIENT_ID= --build-arg BUILD_ARM_CLIENT_SECRET= --build-arg BUILD_ARM_TENANT_ID= -t terra-mod-example .` 。

    系統將會顯示訊息 **Successfully built** (成功建置)。

    ![成功建置](media/terraform-vscode-module-generator/ymg-successfully-built.png)

1. 從命令提示字元，輸入 `docker image ls`。

    螢幕將會列出您新建立的模組 *terra-mod-example*。

    ![存放庫結果](media/terraform-vscode-module-generator/ymg-repository-results.png)

    >[!NOTE]
    >模組的名稱 *terra-mod-example* 已於您在先前步驟 1 中所輸入的命令中指定。

1. 輸入 `docker run -it terra-mod-example /bin/sh`。

    您現已在 Docker 中執行，並可以輸入 `ls` 來列出檔案。

    ![列出 Docker 檔案](media/terraform-vscode-module-generator/ymg-list-docker-file.png)

1. 輸入 `bundle install`。

    等候 **Bundle complete** (組合完成) 的訊息，然後繼續進行下個步驟。

1. 輸入 `rake build`。

    ![Rake build](media/terraform-vscode-module-generator/ymg-rake-build.png)

### <a name="perform-the-end-to-end-test"></a>執行端對端測試

1. 輸入 `rake e2e`。

1. 過幾分鐘之後，將會出現 **PASS** (通過) 訊息。

    ![PASS](media/terraform-vscode-module-generator/ymg-pass.png)

1. 輸入 `exit` 以完成端對端測試。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [安裝及使用 Azure Terraform Visual Studio Code 擴充功能](https://docs.microsoft.com/azure/terraform/terraform-vscode-extension)。
