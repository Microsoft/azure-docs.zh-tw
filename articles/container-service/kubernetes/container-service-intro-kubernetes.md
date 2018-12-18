---
title: Azure Container Service for Kubernetes 簡介
description: Azure Container Service for Kubernetes 可讓您輕鬆地部署和管理 Azure 上的容器型應用程式。
services: container-service
author: gabrtv
manager: jeconnoc
ms.service: container-service
ms.topic: overview
ms.date: 07/21/2017
ms.author: gamonroy
ms.custom: mvc
ms.openlocfilehash: 3e019f02d13ade6a42f4739eacae55f5bf6ac2af
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973587"
---
# <a name="introduction-to-azure-container-service-for-kubernetes"></a>Azure Container Service for Kubernetes 簡介

[!INCLUDE [aks-preview-redirect.md](../../../includes/aks-preview-redirect.md)]

Azure Container Service for Kubernetes 可讓您輕鬆建立、設定及管理虛擬機器的叢集，這些虛擬機器預先設定為執行容器化應用程式。 這樣可讓您使用現有技能，或運用大量且不斷成長的社群專業知識，在 Microsoft Azure 上部署及管理容器應用程式。

藉由使用 Azure Container Service，您可以充分利用 Azure 的企業級功能，同時仍可保有應用程式在 Kubernetes 內的可攜性和 Docker 映像格式。

## <a name="using-azure-container-service-for-kubernetes"></a>使用 Azure Container Service for Kubernetes
我們對於 Azure Container Service 的目標，是要使用現今頗受客戶歡迎的開放原始碼工具和技術，提供容器主控環境。 為了這個目的，我們已公開標準 Kubernetes API 端點。 透過這些標準端點，您可以利用任何能夠與 Kubernetes 叢集通訊的軟體。 例如，您可能會選擇 [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/)、[helm](https://helm.sh/) 或 [draft](https://github.com/Azure/draft)。

## <a name="creating-a-kubernetes-cluster-using-azure-container-service"></a>使用 Azure Container Service 建立 Kubernetes 叢集
若要開始使用 Azure Container Service，請使用 [Azure CLI](container-service-kubernetes-walkthrough.md) 或透過入口網站 (在 Marketplace 內搜尋 **Azure Container Service**) 來部署 Azure Container Service 叢集。 如果您是需要更充分控制 Azure Resource Manager 範本的進階使用者，您可以使用開放原始碼 [acs-engine](https://github.com/Azure/acs-engine) 專案來建立您自己的自訂 Kubernetes 叢集，並透過 `az` CLI 來部署它。

### <a name="using-kubernetes"></a>使用 Kubernetes
Kubernetes 能自動化部署、調整和管理容器化應用程式。 它包含一組豐富的功能，包括︰
* 自動 binpacking
* 自我修復
* 水平調整
* 服務探索和負載平衡
* 自動化推出和復原
* 祕密和組態管理
* 儲存體協調流程
* 批次執行

透過 Container Service 部署的 Kubernetes 架構圖：

![Azure Container Service 設定為使用 Kubernetes。](media/acs-intro/kubernetes.png)

## <a name="videos"></a>影片

Azure Container Service 中的 Kubernetes 支援 (Azure Friday，2017 年 1 月)：

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Kubernetes-Support-in-Azure-Container-Services/player]
>
>

用於在 Kubernetes 開發及部署應用程式的工具 (Azure OpenDev，2017 年 6 月)：

> [!VIDEO https://channel9.msdn.com/Events/AzureOpenDev/June2017/Tools-for-Developing-and-Deploying-Applications-on-Kubernetes/player]
>
>

## <a name="next-steps"></a>後續步驟

瀏覽 [Kubernetes 快速入門](container-service-kubernetes-walkthrough.md)，立即開始探索 Azure Container Service。
