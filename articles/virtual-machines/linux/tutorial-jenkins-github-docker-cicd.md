---
title: 教學課程 - 在 Azure 中使用 Jenkins 建立開發管線 | Microsoft Docs
description: 教學課程 - 在本教學課程中，您會了解如何在 Azure 中建立 Jenkins 虛擬機器，用於在每次程式碼認可時從 GitHub 提取資料，並建立新的 Docker 容器來執行應用程式。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/27/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: d1d760c2642e0c986e50570630d31dda11d17478
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46964732"
---
# <a name="tutorial-create-a-development-infrastructure-on-a-linux-vm-in-azure-with-jenkins-github-and-docker"></a>教學課程：在 Azure 中的 Linux VM 上以 Jenkins、GitHub 及 Docker 建立開發基礎結構

若要將應用程式開發的組建和測試階段自動化，可以使用持續整合和部署 (CI/CD) 管線。 在本教學課程中，您會在 Azure VM 上建立 CI/CD 管線，包括如何︰

> [!div class="checklist"]
> * 建立 Jenkins VM
> * 安裝並設定 Jenkins
> * 建立 GitHub 與 Jenkins 之間的 webhook 整合
> * 從 GitHub 認可建立並觸發 Jenkins 組建作業
> * 建立應用程式的 Docker 映像
> * 確認 GitHub 已認可組建的新 Docker 映像，並更新了執行中的應用程式

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-jenkins-instance"></a>建立 Jenkins 執行個體
在[如何在首次開機時自訂 Linux 虛擬機器](tutorial-automate-vm-deployment.md)的先前教學課程中，您已了解如何使用 cloud-init 自動進行 VM 自訂。 本教學課程使用 cloud-init 檔案在 VM 上安裝 Jenkins 和 Docker。 Jenkins 是熱門的開放原始碼 Automation 伺服程式，可順暢地與 Azure 整合，以進行持續整合 (CI) 及持續傳遞 (CD)。 如需如何使用 Jenkins 的更多教學課程，請參閱 [Azure 中樞中的 Jenkins](https://docs.microsoft.com/azure/jenkins/)。

在您目前的殼層中，建立名為 *cloud-init-jenkins.txt* 的檔案，並貼上下列設定。 例如，在 Cloud Shell 中建立不在本機電腦上的檔案。 輸入 `sensible-editor cloud-init-jenkins.txt` 可建立檔案，並查看可用的編輯器清單。 請確定已正確複製整個 cloud-init 檔案，特別是第一行：

```yaml
#cloud-config
package_upgrade: true
write_files:
  - path: /etc/systemd/system/docker.service.d/docker.conf
    content: |
      [Service]
        ExecStart=
        ExecStart=/usr/bin/dockerd
  - path: /etc/docker/daemon.json
    content: |
      {
        "hosts": ["fd://","tcp://127.0.0.1:2375"]
      }
runcmd:
  - apt install default-jre -y
  - wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
  - sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
  - apt-get update && apt-get install jenkins -y
  - curl -sSL https://get.docker.com/ | sh
  - usermod -aG docker azureuser
  - usermod -aG docker jenkins
  - service jenkins restart
```

建立 VM 之前，請先使用 [az group create](/cli/azure/group#az_group_create) 來建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroupJenkins 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroupJenkins --location eastus
```

現在，使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 使用 `--custom-data` 參數以傳入 cloud-init 組態檔。 如果您將檔案儲存於目前工作目錄之外的位置，請提供 cloud-init-jenkins.txt 的完整路徑。

```azurecli-interactive 
az vm create --resource-group myResourceGroupJenkins \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init-jenkins.txt
```

系統需要花幾分鐘的時間來建立及設定 VM。

為了允許網路流量連線到您的 VM，使用 [az vm open-port](/cli/azure/vm#az_vm_open_port) 開啟連接埠 8080 (用於 Jenkins 流量) 和連接埠 1337 (用於 Node.js 應用程式，此應用程式用來執行範例應用程式)︰

```azurecli-interactive 
az vm open-port --resource-group myResourceGroupJenkins --name myVM --port 8080 --priority 1001
az vm open-port --resource-group myResourceGroupJenkins --name myVM --port 1337 --priority 1002
```


## <a name="configure-jenkins"></a>設定 Jenkins
若要存取您的 Jenkins 執行個體，取得您的 VM 的公用 IP 位址︰

```azurecli-interactive 
az vm show --resource-group myResourceGroupJenkins --name myVM -d --query [publicIps] --o tsv
```

基於安全考量，您必須輸入初始的系統管理員密碼 (儲存在您的 VM 上的文字檔案中)，才能啟動 Jenkins 安裝。 使用上一個步驟取得的公用 IP 位址，透過 SSH 連線至您的 VM：

```bash
ssh azureuser@<publicIps>
```

檢視 Jenkins 安裝的 `initialAdminPassword`，並複製它︰

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

如果檔案尚無法使用，請等待數分鐘，讓 cloud-init 完成 Jenkins 和 Docker 安裝。

現在開啟瀏覽器，前往 `http://<publicIps>:8080`。 完成初始 Jenkins 設定，如下所示︰

- 選擇 [選取要安裝的外掛程式]
- 在頂端的文字方塊中搜尋 *GitHub*。 核取 [GitHub] 方塊，然後選取 [安裝]
- 建立第一位管理使用者。 輸入使用者名稱，例如 **admin**，然後提供您自己的安全密碼。 最後，輸入完整名稱和電子郵件地址。
- 選取 [儲存並結束]
- Jenkins 準備就緒之後，選取 [開始使用 Jenkins]
  - 如果您開始使用 Jenkins 時，網頁瀏覽器顯示空白頁面，請重新啟動 Jenkins 服務。 從您的 SSH 工作階段輸入 `sudo service jenkins restart`，然後重新整理 Web 瀏覽器。
- 使用您建立的使用者名稱和密碼登入 Jenkins。


## <a name="create-github-webhook"></a>建立 GitHub webhook
若要設定與 GitHub 整合，開啟 Azure 範例存放庫中的 [Node.js Hello World 範例應用程式](https://github.com/Azure-Samples/nodejs-docs-hello-world)。 若要將存放庫分支至您自己的 GitHub 帳戶，請選取右上角的 [分支] 按鈕。

在您建立的分支內建立 webhook︰

- 選取 [設定]，然後選取左側的 [整合與服務]。
- 選擇 [新增服務]，然後在篩選方塊中輸入 *Jenkins*。
- 選取 [Jenkins (GitHub 外掛程式)]
- 在 [Jenkins 勾點 URL] 輸入 `http://<publicIps>:8080/github-webhook/`。 別漏掉最後的斜線 (/)。
- 選取 [新增服務]

![將 GitHub webhook 新增至您的分支存放庫](media/tutorial-jenkins-github-docker-cicd/github_webhook.png)


## <a name="create-jenkins-job"></a>建立 Jenkins 作業
為了讓 Jenkins 回應 GitHub 中的事件，例如認可程式碼，請建立 Jenkins 作業。 使用您自己的 GitHub 分支 URL。

在您的 Jenkins 網站中，選取首頁中的 [建立新作業]︰

- 輸入 HelloWorld 作為作業名稱。 選擇 [自由樣式專案]，然後選取 [確定]。
- 在 [一般] 區段下，選取 [GitHub 專案]，然後輸入您的分支存放庫 URL，例如 *https://github.com/cynthn/nodejs-docs-hello-world*
- 在 [原始碼管理] 區段中，選取 [Git]，輸入您的分支存放庫 *.git* URL，例如 *https://github.com/cynthn/nodejs-docs-hello-world.git*
- 在 [組建觸發程序] 下，選取 [GITScm 輪詢的 GitHub 勾點觸發程序]。
- 在 [組建] 區段中，選擇 [新增組建步驟]。 選取 [執行殼層]，然後在命令視窗中輸入 `echo "Test"`。
- 選取作業視窗底部的 [儲存]。


## <a name="test-github-integration"></a>測試 GitHub 整合
若要測試 GitHub 與 Jenkins 的整合，請認可您分支中的變更。 

回到 GitHub 的 Web UI，選取您的分支存放庫，然後選取 **index.js** 檔案。 選取鉛筆圖示以編輯此檔案，並將第 6 行改為︰

```nodejs
response.end("Hello World!");
```

若要認可變更，請選取位於底部的 [認可變更] 按鈕。

在 Jenkins 在作業頁面左下角的 [組建歷程記錄] 區段中會啟動新的組建。 選擇組建編號的連結，然後選取左側的 [主控台輸出]。 在從 GitHub 提取您的程式碼時，您可以檢視 Jenkins 進行的步驟，組建動作會將 `Testing` 訊息輸出到主控台。 每次在 GitHub 中認可，webhook 就會連線到 Jenkins，以這種方式觸發新的組建。


## <a name="define-docker-build-image"></a>定義 Docker 組建映像
為了查看因應您的 GitHub 認可而執行的 Node.js 應用程式，可以組建一個 Docker 映像來執行應用程式。 映像是從 Dockerfile 建立，此檔案定義如何設定執行應用程式的容器。 

在您的虛擬機器的 SSH 連線中，切換至以上一個步驟建立之作業為名的 Jenkins 工作區目錄。 在此範例中命名為 *HelloWorld*。

```bash
cd /var/lib/jenkins/workspace/HelloWorld
```

使用 `sudo sensible-editor Dockerfile` 在此工作區目錄中建立檔案，並貼上下列內容。 請確定已正確複製整個 Docker 檔案，特別是第一行：

```yaml
FROM node:alpine

EXPOSE 1337

WORKDIR /var/www
COPY package.json /var/www/
RUN npm install
COPY index.js /var/www/
```

這個 Dockerfile 會使用基本 Node.js 映像 (此映像使用 Alpine Linux)，公開Hello World 應用程式執行的連接埠 1337，然後複製應用程式檔案並將它初始化。


## <a name="create-jenkins-build-rules"></a>建立 Jenkins 組建規則
您已在上一個步驟中建立基本 Jenkins 組建規則，將訊息輸出至主控台。 現在要建立組建步驟來使用我們的 Dockerfile 並執行應用程式。

回到您的 Jenkins 執行個體，選取在上一個步驟建立的作業。 選取左側的 [設定]，然後向下捲動至 [建置] 區段︰

- 移除現有的 `echo "Test"` 組建步驟。 選取現有建置步驟方塊右上角的紅色叉號。
- 選擇 [新增建置步驟]，然後選取 [執行殼層]
- 在 [命令] 方塊中輸入下列 Docker 命令，然後選取 [儲存]：

  ```bash
  docker build --tag helloworld:$BUILD_NUMBER .
  docker stop helloworld && docker rm helloworld
  docker run --name helloworld -p 1337:1337 helloworld:$BUILD_NUMBER node /var/www/index.js &
  ```

Docker 組建步驟會建立映像，並標記 Jenkins 組建編號，讓您可以維護映像的歷程記錄。 系統會停止任何執行應用程式的現有容器，並加以移除。 然後會使用映像啟動新的容器將，並根據 GitHub 中最新的認可執行您的 Node.js 應用程式。


## <a name="test-your-pipeline"></a>測試您的管線
若要查看整個管線的作用情況，請再次編輯您分支 GitHub 存放庫中的 *index.js* 檔案，然後選取 [認可變更]。 在 Jenkins 中會依據 GitHub 的 webhook啟動新的作業。 系統約需要幾秒鐘來建立 Docker 映像並在新容器中啟動應用程式。

如有需要，再次取得您的 VM 公用 IP 位址：

```azurecli-interactive 
az vm show --resource-group myResourceGroupJenkins --name myVM -d --query [publicIps] --o tsv
```

開啟網路瀏覽器，輸入 `http://<publicIps>:1337`。 您的 Node.js 應用程式會顯示，而且會反映您的 GitHub 分支中最新的認可，如下所示︰

![執行 Node.js 應用程式](media/tutorial-jenkins-github-docker-cicd/running_nodejs_app.png)

現在，對 GitHub 中的 index.js 檔案進行另一次編輯，並認可變更。 稍等幾秒鐘讓 Jenkins 中的作業完成，然後重新整理網路瀏覽器以查看在新容器中執行之應用程式的更新版本，如下所示︰

![在另一次 GitHub 認可後執行 Node.js 應用程式](media/tutorial-jenkins-github-docker-cicd/another_running_nodejs_app.png)


## <a name="next-steps"></a>後續步驟
在本教學課程中，您設定 GitHub 在每次程式碼認可時執行 Jenkins 組建作業，然後部署 Docker 容器來測試您的應用程式。 您已了解如何︰

> [!div class="checklist"]
> * 建立 Jenkins VM
> * 安裝並設定 Jenkins
> * 建立 GitHub 與 Jenkins 之間的 webhook 整合
> * 從 GitHub 認可建立並觸發 Jenkins 組建作業
> * 建立應用程式的 Docker 映像
> * 確認 GitHub 已認可組建的新 Docker 映像，並更新了執行中的應用程式

前進到下一個教學課程，以深入了解如何整合 Jenkins 與 Azure DevOps Services。

> [!div class="nextstepaction"]
> [使用 Jenkins 和 Azure DevOps Services 部署應用程式](tutorial-build-deploy-jenkins.md)
