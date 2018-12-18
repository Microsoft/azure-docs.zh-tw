---
title: 使用 Jenkins 和藍/綠部署模式來部署至 Azure Kubernetes Service (AKS)
description: 了解如何使用 Jenkins 和藍/綠部署模式來部署至 Azure Kubernetes Service (AKS)。
ms.service: jenkins
keywords: jenkins, azure, devops, kubernetes, k8s, aks, 藍/綠部署, 持續傳遞, cd
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 10/11/2018
ms.openlocfilehash: 6cd3938844d7f6977c7b0912acffbfb1679dc42e
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387375"
---
# <a name="deploy-to-azure-kubernetes-service-aks-by-using-jenkins-and-the-bluegreen-deployment-pattern"></a>使用 Jenkins 和藍/綠部署模式來部署至 Azure Kubernetes Service (AKS)

Azure Kubernetes Service (AKS) 可管理裝載 Kubernetes 的環境，以便快速、輕鬆地部署及管理容器化應用程式。 您不需具備容器協調流程專業知識。 AKS 也可透過佈建、升級與依需求調整資源，消除進行中作業及維護之間的界線。 您不需要讓應用程式離線。 如需有關 AKS 的詳細資訊，請參閱 [AKS 文件](/azure/aks/)。

藍/綠部署是 Azure DevOps 的持續傳遞模式，此模式仰賴在部署新版本 (綠色) 時，仍讓現有版本 (藍色) 持續運作的方式。 一般而言，此模式會運用負載平衡將增加的流量導向綠色部署。 當監視功能發現事件時，流量就會路由到仍在執行中的藍色部署。 如需有關持續傳遞的詳細資訊，請參閱[什麼是持續傳遞](/azure/devops/what-is-continuous-delivery)。

在本教學課程中，您會了解如何執行下列工作：

> [!div class="checklist"]
> * 了解藍/綠部署模式
> * 建立受控 Kubernetes 叢集
> * 執行指令碼範例來設定 Kubernetes 叢集
> * 手動設定 Kubernetes 叢集
> * 建立及執行 Jenkins 作業

## <a name="prerequisites"></a>必要條件
- [GitHub 帳戶](https://github.com)：您需要 GitHub 帳戶來複製範例存放庫。
- [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)：您可使用 Azure CLI 2.0 來建立 Kubernetes 叢集。
- [Chocolatey](https://chocolatey.org) - 用來安裝 kubectl 的套件管理員。
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)：用來對 Kubernetes 叢集執行命令的命令列介面。
- [jq](https://stedolan.github.io/jq/download/)：輕量級的命令列 JSON 處理器。

## <a name="clone-the-sample-app-from-github"></a>從 GitHub 複製範例應用程式

在 GitHub 中的 Microsoft 存放庫上，您可以找到一個範例應用程式，說明如何使用 Jenkins 和藍/綠模式來部署到 AKS。 在本節中，您會在您的 GitHub 中建立該存放庫的分支，並將應用程式複製到本機系統。

1. 瀏覽至 GitHub 存放庫以找出 [todo-app-java-on-azure](https://github.com/microsoft/todo-app-java-on-azure.git) 範例應用程式。

    ![Microsoft GitHub 存放庫上範例應用程式的螢幕擷取畫面](./media/jenkins-aks-blue-green-deployment/github-sample-msft.png)

1. 若要建立此存放庫的分支，請選取頁面右上角的 [派生]，並遵循指示來將存放庫派生至您的 GitHub 帳戶中。

    ![可分支處理的 GitHub 選項螢幕擷取畫面](./media/jenkins-aks-blue-green-deployment/github-sample-msft-fork.png)

1. 建立好存放庫的分支後，您會看到帳戶名稱已變更為您的帳戶名稱，並有標示此存放庫從何處派生的附註 (Microsoft)。

    ![GitHub 帳戶名稱和附註的螢幕擷取畫面](./media/jenkins-aks-blue-green-deployment/github-sample-msft-forked.png)

1. 選取 [複製或下載] 。

    ![可複製或下載存放庫的 GitHub 選項螢幕擷取畫面](./media/jenkins-aks-blue-green-deployment/github-sample-clone.png)

1. 在 [使用 HTTPS 複製] 視窗中，選取**複製**圖示。

    ![可將複製 URL 複製到剪貼簿的 GitHub 選項螢幕擷取畫面](./media/jenkins-aks-blue-green-deployment/github-sample-copy.png)

1. 開啟終端機或 Git Bash 視窗。

1. 將目錄變更為您想要用來儲存存放庫本機複本 (複製項目) 的位置。

1. 使用 `git clone` 命令來複製先前的 URL 副本。

    ![Git Bash git 複製命令的螢幕擷取畫面](./media/jenkins-aks-blue-green-deployment/git-clone-command.png)

1. 按 Enter 鍵以啟動複製程序。

    ![Git Bash git 複製命令結果的螢幕擷取畫面](./media/jenkins-aks-blue-green-deployment/git-clone-results.png)

1. 將目錄變更為新建立的目錄，其中包含應用程式來源的複本。

## <a name="create-and-configure-a-managed-kubernetes-cluster"></a>建立和設定受控 Kubernetes 叢集

在本節中，您將執行下列步驟：

- 使用 Azure CLI 2.0 建立受控 Kubernetes 叢集。
- 了解如何使用安裝指令碼或手動設定叢集。
- 建立 Azure Container Registry 服務的執行個體。

### <a name="use-the-azure-cli-20-to-create-a-managed-kubernetes-cluster"></a>使用 Azure CLI 2.0 建立受控 Kubernetes 叢集
若要使用 [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 建立受控 Kubernetes 叢集，請確定您是使用 Azure CLI 2.0.25 版或更新版本。

1. 登入您的 Azure 帳戶。 輸入下列命令後，您會收到說明如何完成登入的指示。 
    
    ```bash
    az login
    ```

1. 當您在上一個步驟中執行 `az login` 時，包含您所有 Azure 訂用帳戶的清單會隨即出現 (連同它們的訂用帳戶識別碼)。 在此步驟中，您可以設定預設 Azure 訂用帳戶。 以所需的 Azure 訂用帳戶識別碼取代 &lt;your-subscription-id> 預留位置。 

    ```bash
    az account set -s <your-subscription-id>
    ```

1. 建立資源群組。 以新資源群組的名稱取代 &lt;your-resource-group-name> 預留位置，並以您的位置取代 &lt;your-location> 預留位置。 此 `az account list-locations` 命令會顯示所有 Azure 位置。 在 AKS 還是預覽版的期間，並非所有位置都可使用。 如果您目前輸入的位置無效，則錯誤訊息會列出可用的位置。

    ```bash
    az group create -n <your-resource-group-name> -l <your-location>
    ```

1. 建立 Kubernetes 叢集。 以上一個步驟中建立的資源群組名稱取代 &lt;your-resource-group-name>，並以新叢集的名稱取代 &lt;your-kubernetes-cluster-name>。 (此程序可能需要數分鐘的時間才能完成。)

    ```bash
    az aks create -g <your-resource-group-name> -n <your-kubernetes-cluster-name> --generate-ssh-keys --node-count 2
    ```

### <a name="set-up-the-kubernetes-cluster"></a>設定 Kubernetes 叢集

您可以在 AKS 中手動設定藍/綠部署，或者使用稍早所複製範例中提供的安裝指令碼。 在本節中，您將了解如何使用這兩個方式。

#### <a name="set-up-the-kubernetes-cluster-via-the-sample-setup-script"></a>透過安裝指令碼範例設定 Kubernetes 叢集
1. 編輯 **deploy/aks/setup/setup.sh** 檔案，以適用您環境的值取代下列預留位置： 

    - **&lt;your-resource-group-name>**
    - **&lt;your-kubernetes-cluster-name>**
    - **&lt;your-location>**
    - **&lt;your-dns-name-suffix>**

    ![bash 中已醒目提示數個預留位置的 setup.sh 指令碼螢幕擷取畫面](./media/jenkins-aks-blue-green-deployment/edit-setup-script.png)

1. 執行安裝指令碼。

    ```bash
    sh setup.sh
    ```

#### <a name="set-up-a-kubernetes-cluster-manually"></a>手動設定 Kubernetes 叢集 
1. 將 Kubernetes 組態下載到您的設定檔資料夾。

    ```bash
    az aks get-credentials -g <your-resource-group-name> -n <your-kubernetes-cluster-name> --admin
    ```

1. 將目錄變更為 **deploy/aks/setup** 目錄。 

1. 執行下列 **kubectl** 命令，為公用端點和兩個測試端點設定服務。

    ```bash
    kubectl apply -f  service-green.yml
    kubectl apply -f  test-endpoint-blue.yml
    kubectl apply -f  test-endpoint-green.yml
    ```

1. 更新公用和測試端點的 DNS 名稱。 當您建立 Kubernetes 叢集時，您也會建立[額外的資源群組](https://github.com/Azure/AKS/issues/3)，其命名模式為 **MC_&lt;your-resource-group-name>_&lt;your-kubernetes-cluster-name>_&lt;your-location>**。

    找出資源群組中的公用 IP。

    ![資源群組中公用 IP 的螢幕擷取畫面](./media/jenkins-aks-blue-green-deployment/publicip.png)

    執行下列命令可找出每個服務的外部 IP 位址︰
    
    ```bash
    kubectl get service todoapp-service
    ```
    
    使用下列命令更新對應 IP 位址的 DNS 名稱：

    ```bash
    az network public-ip update --dns-name aks-todoapp --ids /subscriptions/<your-subscription-id>/resourceGroups/MC_<resourcegroup>_<aks>_<location>/providers/Microsoft.Network/publicIPAddresses/kubernetes-<ip-address>
    ```

    重複 `todoapp-test-blue` 和 `todoapp-test-green` 的呼叫：

    ```bash
    az network public-ip update --dns-name todoapp-blue --ids /subscriptions/<your-subscription-id>/resourceGroups/MC_<resourcegroup>_<aks>_<location>/providers/Microsoft.Network/publicIPAddresses/kubernetes-<ip-address>

    az network public-ip update --dns-name todoapp-green --ids /subscriptions/<your-subscription-id>/resourceGroups/MC_<resourcegroup>_<aks>_<location>/providers/Microsoft.Network/publicIPAddresses/kubernetes-<ip-address>
    ```

    訂用帳戶中的 DNS 名稱必須是唯一的。 若要確保唯一性，您可以使用 `<your-dns-name-suffix>`。

### <a name="create-an-instance-of-container-registry"></a>建立 Container Registry 的執行個體

1. 執行 `az acr create` 命令以建立 Container Registry 的執行個體。 在下一節中，您可以接著使用 `login server` 作為 Docker 登錄 URL。

    ```bash
    az acr create -n <your-registry-name> -g <your-resource-group-name>
    ```

1. 執行 `az acr credential` 命令可顯示您的 Container Registry 認證。 請記下 Docker 登錄使用者名稱和密碼，因為您會在下一節中用到。

    ```bash
    az acr credential show -n <your-registry-name>
    ```

## <a name="prepare-the-jenkins-server"></a>準備 Jenkins 伺服器

在本節中，您會了解如何準備 Jenkins 伺服器來執行適合測試的組建。 不過，您應該使用 [Azure VM 代理程式](https://plugins.jenkins.io/azure-vm-agents)或 [Azure 容器代理程式](https://plugins.jenkins.io/azure-container-agents)，加速 Azure 中的代理程式執行您的組建。 如需詳細資訊，請參閱有關[在主要節點上建置的安全性含意](https://wiki.jenkins.io/display/JENKINS/Security+implication+of+building+on+master)的 Jenkins 文章。

1. [在 Azure 上部署 Jenkins 主要伺服器](https://aka.ms/jenkins-on-azure)。

1. 透過 SSH 連線到伺服器，並在執行您組建的伺服器上安裝組建工具。
   
   ```bash
   sudo apt-get install git maven 
   ```
   
1. [安裝 Docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce)。 確定使用者 `jenkins` 有權執行 `docker` 命令。

1. [安裝 kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)。

1. [下載 jq](https://stedolan.github.io/jq/download/)。

1. 使用下列命令安裝 jq：

   ```bash
   sudo apt-get install jq
   ```
   
1. 在 Jenkins 儀表板中執行下列步驟，即可在 Jenkins 中安裝外掛程式：

    1. 選取 [管理 Jenkins] > [管理外掛程式] > [可用項目]。
    1. 搜尋和安裝 Azure Container Service 外掛程式。

1. 新增認證以在 Azure 中管理資源。 如果您還沒有此外掛程式，請安裝 **Azure 認證**外掛程式。

1. 將您的 Azure 服務主體認證新增為 **Microsoft Azure 服務主體**類型。

1. 新增您的 Azure Docker Registry 使用者名稱和密碼 (已在「建立 Container Registry 的執行個體」一節中取得) 作為 [使用者名稱與密碼] 類型。

## <a name="edit-the-jenkinsfile"></a>編輯 Jenkinsfile

1. 在您自己的存放庫中，移至 `/deploy/aks/` 並開啟 `Jenkinsfile`。

2. 更新檔案，如下所示：

    ```groovy
    def servicePrincipalId = '<your-service-principal>'
    def resourceGroup = '<your-resource-group-name>'
    def aks = '<your-kubernetes-cluster-name>'

    def cosmosResourceGroup = '<your-cosmodb-resource-group>'
    def cosmosDbName = '<your-cosmodb-name>'
    def dbName = '<your-dbname>'

    def dockerRegistry = '<your-acr-name>.azurecr.io'
    ```
    
    更新容器登錄認證識別碼：
    
    ```groovy
    def dockerCredentialId = '<your-acr-credential-id>'
    ```

## <a name="create-the-job"></a>建立工作
1. 在**管線**類型中新增作業。

1. 選取 [管線] > [定義] > [來自 SCM 的管線指令碼]。

1. 輸入您的 &lt;your-forked-repo> 作為 SCM 存放庫 URL。

1. 輸入指令碼路徑 `deploy/aks/Jenkinsfile`。

## <a name="run-the-job"></a>執行工作

1. 請確認您可以在本機環境中成功執行專案。 方式如下：[在本機電腦上執行專案](https://github.com/Microsoft/todo-app-java-on-azure/blob/master/README.md#run-it)。

1. 執行 Jenkins 作業。 第一次執行作業時，Jenkins 會將 Todo 應用程式部署到藍色環境，也就是預設非使用中環境。 

1. 若要確認作業是否已執行，請移到下列 URL：
    - 公用端點：`http://aks-todoapp<your-dns-name-suffix>.<your-location>.cloudapp.azure.com`
    - 藍色端點 - `http://aks-todoapp-blue<your-dns-name-suffix>.<your-location>.cloudapp.azure.com`
    - 綠色端點 - `http://aks-todoapp-green<your-dns-name-suffix>.<your-location>.cloudapp.azure.com`

當綠色端點顯示預設的 Tomcat 影像時，公用和藍色測試端點會有相同的更新。 

如果您會多次執行組建，則該組建會在藍色與綠色部署中循環。 換句話說，如果目前的環境是藍色，作業會部署到綠色環境並加以測試。 然後，如果測試狀況良好，則作業會更新應用程式公用端點，以將流量路由傳送至綠色環境。

## <a name="additional-information"></a>其他資訊

如需零停機部署的詳細資訊，請參閱此[快速入門範本](https://github.com/Azure/azure-quickstart-templates/tree/master/301-jenkins-aks-zero-downtime-deployment)。 

## <a name="clean-up-resources"></a>清除資源

當您不再需要您在本教學課程中建立的資源時，您可加以刪除。

```bash
az group delete -y --no-wait -n <your-resource-group-name>
```

## <a name="troubleshooting"></a>疑難排解

如果您遇到任何有關 Jenkins 外掛程式的錯誤，請在 [Jenkins JIRA](https://issues.jenkins-ci.org/) 的特定元件中提交問題。

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何使用 Jenkins 和藍/綠部署模式來部署到 AKS。 若要深入了解 Azure Jenkins 提供者，請參閱 Azure 網站上的 Jenkins。

> [!div class="nextstepaction"]
> [Azure 上的 Jenkins](/azure/jenkins/)