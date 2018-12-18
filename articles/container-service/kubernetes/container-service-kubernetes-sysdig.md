---
title: 監視 Azure Kubernetes 叢集 - Sysdig
description: 使用 Sysdig 監視 Azure Container Service 中的 Kubernetes 叢集
services: container-service
author: bburns
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 12/09/2016
ms.author: bburns
ms.custom: mvc
ms.openlocfilehash: bbf59a35f420b5bbf292fbdaa5a8bbc173e4ee24
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35633324"
---
# <a name="monitor-an-azure-container-service-kubernetes-cluster-using-sysdig"></a>使用 Sysdig 監視 Azure Container Service Kubernetes 叢集

[!INCLUDE [aks-preview-redirect.md](../../../includes/aks-preview-redirect.md)]

## <a name="prerequisites"></a>必要條件
本逐步解說假設您已[使用 Azure Container Service 建立 Kubernetes 叢集](container-service-kubernetes-walkthrough.md)。

同時也假設您已經安裝 azure cli 和 kubectl 工具。

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

## <a name="sysdig"></a>Sysdig
Sysdig 是一家外部監視即服務公司，可監視您在 Azure 中執行的 Kubernetes 叢集中的容器。 使用 Sysdig 需要有效的 Sysdig 帳戶。
您可以在他們的[網站](https://app.sysdigcloud.com)上註冊帳戶。

登入至 Sysdig 雲端網站後，按一下使用者名稱，在頁面上應該會看到「存取金鑰」。 

![Sysdig API 金鑰](./media/container-service-kubernetes-sysdig/sysdig2.png)

## <a name="installing-the-sysdig-agents-to-kubernetes"></a>將 Sysdig 代理程式安裝至 Kubernetes
為了監視您的容器，Sysdig 要在每部使用 Kubernetes `DaemonSet` 的機器上執行處理序。
DaemonSets 是每部機器執行單一容器執行個體的 Kubernetes API 物件。
它們非常適合用來安裝 Sysdig 監視代理程式之類的工具。

若要安裝 Sysdig daemonset，您應該先從 sysdig 下載[範本](https://github.com/draios/sysdig-cloud-scripts/tree/master/agent_deploy/kubernetes)。 將該檔案儲存為 `sysdig-daemonset.yaml`。

在 Linux 與 OS X 上，您可以執行︰

```console
$ curl -O https://raw.githubusercontent.com/draios/sysdig-cloud-scripts/master/agent_deploy/kubernetes/sysdig-daemonset.yaml
```

在 PowerShell 中：

```console
$ Invoke-WebRequest -Uri https://raw.githubusercontent.com/draios/sysdig-cloud-scripts/master/agent_deploy/kubernetes/sysdig-daemonset.yaml | Select-Object -ExpandProperty Content > sysdig-daemonset.yaml
```

然後編輯該檔案，以插入您從 Sysdig 帳戶取得的存取金鑰。

最後，建立 DaemonSet：

```console
$ kubectl create -f sysdig-daemonset.yaml
```

## <a name="view-your-monitoring"></a>檢視您的監視
完成安裝並在執行時，代理程式會將資料送回 Sysdig。  返回 [sysdig 儀表板](https://app.sysdigcloud.com)，您應該會看到您容器的相關資訊。

您也可以透過[新的儀表板精靈](https://app.sysdigcloud.com/#/dashboards/new)來安裝 Kubernetes 專用儀表板。
