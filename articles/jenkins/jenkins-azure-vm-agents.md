---
title: 使用 Azure VM 代理程式調整 Jenkins 部署。
description: 使用有 Jenkins Azure VM 代理程式外掛程式的 Azure 虛擬機器，增加 Jenkins 管線的容量。
ms.service: jenkins
keywords: jenkins, azure, devops, 虛擬機器, 代理程式
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 07/31/2018
ms.openlocfilehash: 46710b3a669b6a00dc1826c55e8d35fe700f312f
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43106218"
---
# <a name="scale-your-jenkins-deployments-to-meet-demand-with-azure-vm-agents"></a>使用 Azure VM 代理程式調整 Jenkins 部署以滿足需求

本教學課程示範如何使用 Jenkins [Azure VM 代理程式外掛程式](https://plugins.jenkins.io/azure-vm-agents)對於 Azure 中執行的 Linux 虛擬機器新增隨需容量。

在本教學課程中，您將：

> [!div class="checklist"]
> * 安裝 Azure VM 代理程式外掛程式
> * 設定外掛程式在您的 Azure 訂用帳戶中建立資源
> * 設定每個代理程式可用的計算資源
> * 設定每個代理程式上安裝的作業系統和工具
> * 建立新的 Jenkins 自由樣式專案
> * 在 Azure VM 代理程式上執行作業

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Continuous-Integration-with-Jenkins-Using-Azure-VM-Agents/player]

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶
* Jenkins 主要伺服器。 如果沒有，請參閱[快速入門](install-jenkins-solution-template.md)，在 Azure 中設定一個。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="install-azure-vm-agents-plugin"></a>安裝 Azure VM 代理程式外掛程式

> [!TIP]
> 如果您已使用[方案範本](install-jenkins-solution-template.md)在 Azure 上部署 Jenkins，則 Azure VM 代理程式外掛程式已安裝。

1. 從 Jenkins 儀表板中，選取 [管理 Jenkins]，然後選取[管理外掛程式]。
1. 選取 [可用] 索引標籤，然後搜尋 [Azure VM 代理程式]。 選取外掛程式項目旁邊的核取方塊，然後選取儀表板底部的 [安裝但不重新啟動]。

## <a name="configure-the-azure-vm-agents-plugin"></a>設定 Azure VM 代理程式外掛程式

1. 從 Jenkins 儀表板中，選取 [管理 Jenkins]，然後選取 [設定系統]。
1. 捲動到頁面底端，然後於 [新增新雲端] 下拉式功能表中尋找 [雲端] 區段，再選擇 [Microsoft Azure VM 代理程式]。
1. 從 [Azure 認證] 區段的 [新增] 下拉式功能表中選取現有的服務主體。 如果未列出任何項目，請執行下列步驟為您的 Azure 帳戶[建立服務主體](/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager)，然後將它新增至您的 Jenkins 組態：   

    a. 選取 [Azure 認證] 旁的 [新增]，然後選擇 [Jenkins]。   
    b. 在 [新增認證] 對話方塊中，從 [種類] 下拉式清單中選取 [Microsoft Azure 服務主體]。   
    c. 從 Azure CLI 或 [Cloud Shell](/azure/cloud-shell/overview) 建立 Active Directory 服務主體。
    
    ```azurecli-interactive
    az ad sp create-for-rbac --name jenkins_sp --password secure_password
    ```

    ```json
    {
        "appId": "BBBBBBBB-BBBB-BBBB-BBBB-BBBBBBBBBBB",
        "displayName": "jenkins_sp",
        "name": "http://jenkins_sp",
        "password": "secure_password",
        "tenant": "CCCCCCCC-CCCC-CCCC-CCCCCCCCCCC"
    }
    ```
    d. 將服務主體的認證輸入 [新增認證] 對話方塊。 如果您不知道您的 Azure 訂用帳戶識別碼，您可以從 CLI 查詢：
     
     ```azurecli-interactive
     az account list
     ```

     ```json
        {
            "cloudName": "AzureCloud",
            "id": "AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA",
            "isDefault": true,
            "name": "Visual Studio Enterprise",
            "state": "Enabled",
            "tenantId": "CCCCCCCC-CCCC-CCCC-CCCC-CCCCCCCCCCC",
            "user": {
            "name": "raisa@fabrikam.com",
            "type": "user"
            }
     ```

    完成的服務主體應將 `id` 欄位用於 [訂用帳戶識別碼]、`appId` 值用於 [用戶端識別碼]、`password` 用於 [用戶端密碼] 以及 `tenant` 用於 [租用戶識別碼]。 選取 [新增] 新增服務主體，然後設定外掛程式以使用新建立的認證。

    ![設定 Azure 服務主體](./media/jenkins-azure-vm-agents/new-service-principal.png)

    

1. 在 [資源群組名稱] 區段中，保留 [建立新項目] 為已選取，並輸入 `myJenkinsAgentGroup`。
1. 選取 [驗證設定] 連線至 Azure 以測試設定檔設定。
1. 選取 [套用] 更新外掛程式設定。

## <a name="configure-agent-resources"></a>設定代理程式資源

設定用來定義 Azure VM 代理程式的範本。 此範本在建立後會定義每個代理程式的計算資源。

1. 選取 [新增 Azure 虛擬機器範本] 旁的 [新增]。
1. 於 [名稱] 輸入 `defaulttemplate`
1. 於 [標籤] 輸入 `ubuntu`
1. 從下拉式方塊中選取所需的 [Azure 區域](https://azure.microsoft.com/regions/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。
1. 從 **[虛擬機器大小]** 下方的下拉式清單中選取 [虛擬機器大小](/azure/virtual-machines/linux/sizes)。 一般用途的 `Standard_DS1_v2` 大小適用於教學課程。   
1. 將 [保留時間] 保留為 `60`。 此設定會定義 Jenkins 解除配置閒置的代理程式前等待的分鐘數。 如果您不想要自動移除閒置的代理程式，請指定 0。

   ![一般虛擬機器組態](./media/jenkins-azure-vm-agents/general-config.png)

## <a name="configure-agent-operating-system-and-tools"></a>選擇代理程式作業系統和工具

在外掛程式設定的 [影像設定] 區段中，選取 [Ubuntu 16.04 LTS]。 勾選 [安裝 Git (最新版)]、[安裝 Maven (V3.5.0)] 與 [安裝 Docker] 旁的方塊，藉此在新建立的代理程式上安裝這些工具。

![設定虛擬機器作業系統與工具](./media/jenkins-azure-vm-agents/jenkins-os-config.png)

選取 [管理員認證] 旁的 [新增]，再選取 [Jenkins]。 輸入用來登入代理程式的使用者名稱與密碼，並確定這些資料符合 Azure VM 系統管理帳戶的[使用者名稱與密碼原則](/azure/virtual-machines/linux/faq#what-are-the-username-requirements-when-creating-a-vm)。

選取 [驗證範本] 驗證設定，然後選取 [儲存] 儲存您的變更並返回 Jenkins 儀表板。

## <a name="create-a-job-in-jenkins"></a>在 Jenkins 中建立作業

1. 在 Jenkins 儀表板中，按一下 [新增項目] 。 
1. 輸入 `demoproject1` 作為名稱，然後選取 [自由式專案]，再選取 [確定]。
1. 在 [一般] 索引標籤中，選擇 [限制可執行專案的位置]，並在 [標籤運算式] 中輸入 `ubuntu`。 您會看到一則訊息確認標籤是由上個步驟中建立的雲端設定所提供。 
   ![設定工作](./media/jenkins-azure-vm-agents/job-config.png)
1. 在 [原始程式碼管理] 索引標籤中，選取 [Git]，然後在 [存放庫 URL] 欄位中新增以下 URL：`https://github.com/spring-projects/spring-petclinic.git`
1. 在 [組建] 索引標籤中，選取 [新增組建步驟]，然後選取 [叫用最上層 Maven 目標]。 在 [目標] 欄位中輸入 `package`。
1. 選取 [儲存] 以儲存作業定義。

## <a name="build-the-new-job-on-an-azure-vm-agent"></a>在 Azure VM 代理程式上建置新作業

1. 返回 Jenkins 儀表板。
1. 選取您在前一個步驟中建立的作業，然後按一下 [立即建置]。 新組建隨即排入佇列，但會一直等到代理程式虛擬機器在您的 Azure 訂用帳戶中建立後才會啟動。
1. 當建置完成之後，請移至 [主控台輸出]。 您會看到該組建在 Azure 代理程式上遠端執行。

![主控台輸出](./media/jenkins-azure-vm-agents/console-output.png)

## <a name="troubleshooting-the-jenkins-plugin"></a>對 Jenkins 外掛程式進行疑難排解

如果您遇到任何有關 Jenkins 外掛程式的錯誤，請在 [Jenkins JIRA](https://issues.jenkins-ci.org/) 的特定元件中提交問題。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [CI/CD 至 Azure App Service](java-deploy-webapp-tutorial.md)