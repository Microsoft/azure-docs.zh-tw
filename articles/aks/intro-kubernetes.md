---
title: "Azure Container Service for Kubernetes 簡介 | Microsoft Docs"
description: "Azure Container Service for Kubernetes 可讓您輕鬆地部署和管理 Azure 上的容器型應用程式。"
services: container-service
documentationcenter: 
author: gabrtv
manager: timlt
editor: 
tags: aks, azure-container-service
keywords: "Kubernetes, Docker, 容器, 微服務, Azure"
ms.assetid: 
ms.service: container-service
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/24/2017
ms.author: gamonroy
ms.custom: mvc
ms.openlocfilehash: a8ac18464d0efcc0db96e1667f18f2f853208573
ms.sourcegitcommit: f8437edf5de144b40aed00af5c52a20e35d10ba1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/03/2017
---
# <a name="introduction-to-azure-container-service-aks"></a>Azure Container Service (AKS) 簡介

Azure Container Service (AKS) 可讓您輕鬆建立、設定及管理虛擬機器的叢集，這些虛擬機器預先設定為執行容器化應用程式。 這樣可讓您使用現有技能，或運用大量且不斷成長的社群專業知識，在 Microsoft Azure 上部署及管理容器應用程式。

藉由使用 AKS，您可以充分利用 Azure 的企業級功能，同時仍可保有應用程式在 Kubernetes 內的可攜性和 Docker 映像格式。

## <a name="managed-kubernetes-in-azure"></a> Azure 內的受管理 Kubernetes

AKS 利用將責任轉移給 Azure 的方式減少了複雜度與管理 Kubernetes 叢集營運的負擔，作為一個主控的 Kebernetes 服務，Azure 處理了如健康監測與維護的關鍵性工作，你只需要支付你的叢集內的代理節點 (agent node) 的使用，而不需支付主控節點 (master)，作為一個受管理的 Kubernetes 服務，AKS 提供了：

> [!div class="checklist"]
> * 自動化的 Kubernetes 版本升級與修補。
> * 簡易的叢集擴展。
> * 自我修復的主控制台 (masters)。
> * 節省成本-只需要支付您執行的代理池節點。

Azure 會在您的 AKS 叢集內處理管理工作，您不需要手動執行許多工作，像叢集升級。因為 Azure 為您處理這些關鍵維護工作，AKS 不提供對叢集的直接存取 (像是SSH)。

## <a name="using-azure-container-service-aks"></a>使用 Azure Container Service (AKS)
我們對於 AKS 的目標，是要使用現今頗受客戶歡迎的開放原始碼工具和技術，提供容器主控環境。 為了這個目的，我們已公開標準 Kubernetes API 端點。 透過這些標準端點，您可以利用任何能夠與 Kubernetes 叢集通訊的軟體。 例如，您可能會選擇 [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/)、[helm](https://helm.sh/) 或 [draft](https://github.com/Azure/draft)。

## <a name="creating-a-kubernetes-cluster-using-azure-container-service-aks"></a>使用 Azure Container Service (AKS) 建立 Kubernetes 叢集
若要開始使用 AKS，請使用 [Azure CLI](./kubernetes-walkthrough.md) 或透過入口網站 (在 Marketplace 內搜尋 **Azure Container Service**) 來部署 AKS 叢集。 如果您是需要更充分控制 Azure Resource Manager 範本的進階使用者，您可以使用開放原始碼 [acs-engine](https://github.com/Azure/acs-engine) 專案來建立您自己的自訂 Kubernetes 叢集，並透過 `az` CLI 來部署它。

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

## <a name="videos"></a>影片

Azure Container Service (AKS) - Azure Friday，2017 年 10 月：

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Container-Orchestration-Simplified-with-Managed-Kubernetes-in-Azure-Container-Service-AKS/player]
>
>

用於在 Kubernetes 開發及部署應用程式的工具 - Azure OpenDev，2017 年 6 月：

> [!VIDEO https://channel9.msdn.com/Events/AzureOpenDev/June2017/Tools-for-Developing-and-Deploying-Applications-on-Kubernetes/player]
>
>

使用 AKS 快速入門深入了解部署和管理 AKS。

> [!div class="nextstepaction"]
> [AKS 教學課程](./kubernetes-walkthrough.md)
