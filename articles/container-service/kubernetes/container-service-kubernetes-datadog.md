---
title: 使用 Datadog 監視 Azure Kubernetes 叢集
description: 使用 Datadog 監視 Azure Container Service 中的 Kubernetes 叢集
services: container-service
author: bburns
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 12/09/2016
ms.author: bburns
ms.custom: mvc
ms.openlocfilehash: 0a3f0baa4998dbc594023935575d659f7d45bbb9
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38629064"
---
# <a name="monitor-an-azure-container-service-cluster-with-datadog"></a>使用 DataDog 監視 Azure Container Service 叢集

[!INCLUDE [aks-preview-redirect.md](../../../includes/aks-preview-redirect.md)]

## <a name="prerequisites"></a>先決條件
本逐步解說假設您已[使用 Azure Container Service 建立 Kubernetes 叢集](container-service-kubernetes-walkthrough.md)。

同時也假設您已經安裝 `az` Azure cli 和 `kubectl` 工具。

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

## <a name="datadog"></a>DataDog
Datadog 是一項監視服務，會從 Azure 容器服務叢集內的容器收集監視資料。 Datadog 有 Docker 整合儀表板，可供您查看容器內的特定度量。 從容器收集到的度量會依 CPU、記憶體、網路和 I/O 來加以整理。 Datadog 會將度量分割成容器和映像。

您必須先[建立帳戶 (英文)](https://www.datadoghq.com/lpg/)

## <a name="installing-the-datadog-agent-with-a-daemonset"></a>使用 DaemonSet 安裝 Datadog Agent
DaemonSet 是 Kubernetes 用來在叢集中每個主機上執行容器的單一執行個體。
它們非常適合用來執行監視代理程式。

當您登入 Datadog 之後，您可以依照 [Datadog 指示 (英文)](https://app.datadoghq.com/account/settings#agent/kubernetes) 使用 DaemonSet 在您的叢集上安裝 Datadog Agent。

## <a name="conclusion"></a>結論
就這麼簡單！ 當代理程式啟動並執行之後，幾分鐘之內您應該會在主控台中看到資料。 您可以造訪這些整合式 [kubernetes 儀表板 (英文)](https://app.datadoghq.com/screen/integration/kubernetes) 以查看您的叢集摘要。
