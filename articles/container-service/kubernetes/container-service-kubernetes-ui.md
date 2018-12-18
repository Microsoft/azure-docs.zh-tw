---
title: 使用 Web UI 管理 Azure Kubernetes 叢集
description: 在 Azure Container Service 中使用 Kubernetes Web UI
services: container-service
author: bburns
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 02/21/2017
ms.author: bburns
ms.custom: mvc
ms.openlocfilehash: b5079ddb63f468f924577f61adda6a3056ec8c6c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998520"
---
# <a name="using-the-kubernetes-web-ui-with-azure-container-service"></a>搭配 Azure Container Service 使用 Kubernetes Web UI

[!INCLUDE [aks-preview-redirect.md](../../../includes/aks-preview-redirect.md)]

## <a name="prerequisites"></a>必要條件
本逐步解說假設您已[使用 Azure Container Service 建立 Kubernetes 叢集](container-service-kubernetes-walkthrough.md)。


同時也假設您已經安裝 Azure CLI 和 `kubectl` 工具。

您可以藉由執行下列操作來測試是否已安裝 `az` 工具：

```console
$ az --version
```

如果您尚未安裝 `az` 工具，[這裡](https://github.com/azure/azure-cli#installation)有指示。

您可以藉由執行下列操作來測試是否已安裝 `kubectl` 工具：

```console
$ kubectl version
```

如果您尚未安裝 `kubectl`，可以執行︰

```console
$ az acs kubernetes install-cli
```

## <a name="overview"></a>概觀

### <a name="connect-to-the-web-ui"></a>連接至 Web UI
您可以藉由執行下列操作啟動 Kubernetes Web UI：

```console
$ az acs kubernetes browse -g [Resource Group] -n [Container service instance name]
```

這應該會開啟網頁瀏覽器，設定與安全 Proxy (將您的本機電腦連接至 Kubernetes Web UI) 通訊。

### <a name="create-and-expose-a-service"></a>建立和公開服務
1. 在 Kubernetes Web UI 中，按一下右上方視窗中的 [建立] 按鈕。

    ![Kubernetes Create UI](./media/container-service-kubernetes-ui/create.png)

    隨即會開啟對話方塊，您可以在其中開始建立應用程式。

2. 請為其指定 `hello-nginx` 名稱。 使用 [`nginx`Docker 容器](https://hub.docker.com/_/nginx/)及部署此 Web 服務的三個複本。

    ![Kubernetes Pod 建立對話方塊](./media/container-service-kubernetes-ui/nginx.png)

3. 在 [服務] 之下，選取 [外部] 並輸入連接埠 80。

    這項設定會將流量負載平衡至三個複本。

    ![Kubernetes 服務建立對話方塊](./media/container-service-kubernetes-ui/service.png)

4. 按一下 [部署] 來部署這些容器和服務。

    ![Kubernetes 部署](./media/container-service-kubernetes-ui/deploy.png)

### <a name="view-your-containers"></a>檢視您的容器
按一下 [部署] 之後，UI 會在部署時顯示服務的檢視︰

![Kubernetes 狀態](./media/container-service-kubernetes-ui/status.png)

您可以在 [Pod] 之下 UI 左手邊的圓圈中，看到每個 Kubernetes 物件的狀態。 如果是部分完整的圓圈，則物件仍然在部署中。 完整部署物件時，它會顯示綠色的核取記號︰

![Kubernetes 已部署](./media/container-service-kubernetes-ui/deployed.png)

一旦所有項目都在執行，按一下其中一個 Pod，便可查看執行中相關的 Web 服務詳細資料。

![Kubernetes Pods](./media/container-service-kubernetes-ui/pods.png)

在 [Pod] 檢視中，您可以看到 Pod 中容器的相關資訊，以及這些容器所使用的 CPU 和記憶體資源︰

![Kubernetes 資源](./media/container-service-kubernetes-ui/resources.png)

如果看不到資源，您可能需要等候幾分鐘讓監視資料傳播。

若要查看您容器的記錄檔，請按一下 [檢視記錄檔]。

![Kubernetes 記錄檔](./media/container-service-kubernetes-ui/logs.png)

### <a name="viewing-your-service"></a>檢視您的服務
除了執行您的容器，Kubernetes UI 建立外部 `Service`，佈建負載平衡器將流量帶向叢集中的容器。

在左側導覽窗格中，按一下 [服務] 以檢視所有的服務 (應該只有一個)。

![Kubernetes 服務](./media/container-service-kubernetes-ui/service-deployed.png)

在該檢視中，您應該會看到已經配置給服務的外部端點 (IP 位址)。
如果您按一下該 IP 位址，應該會看到在負載平衡器後方執行的 Nginx 容器。

![nginx 檢視](./media/container-service-kubernetes-ui/nginx-page.png)

### <a name="resizing-your-service"></a>調整您的服務大小
除了在 UI 中檢視您的物件，您可以編輯和更新 Kubernetes API 物件。

首先，按一下左側導覽窗格中的 [部署]，以查看您服務的部署。

一旦您在該檢視中，請按一下複本集，然後按一下上方導覽列中的 [編輯]：

![Kubernetes 編輯](./media/container-service-kubernetes-ui/edit.png)

將 `spec.replicas` 欄位編輯為 `2`，然後按一下 [更新]。

這會刪除您的其中一個 Pod，使得複本數目降為兩個。

 

