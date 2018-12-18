---
title: 如何使用 Kubectl 搭配 Azure Dev Spaces | Microsoft Docs
titleSuffix: Azure Dev Spaces
services: azure-dev-spaces
ms.service: azure-dev-spaces
ms.component: azds-kubernetes
author: ghogen
ms.author: ghogen
ms.date: 05/11/2018
ms.topic: article
description: 在 Azure 上使用容器和微服務快速進行 Kubernetes 開發
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, 容器
manager: douge
ms.openlocfilehash: 3dc0dd4b571f716bcabb67c4cbef1ea6d762eb94
ms.sourcegitcommit: 50f82f7682447245bebb229494591eb822a62038
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/08/2018
ms.locfileid: "35248653"
---
# <a name="use-kubectl-with-an-azure-dev-space"></a>使用 Kubectl 搭配 Azure Dev Spaces

您可以存取 Azure Dev Spaces 內的 Kubernetes 叢集，並且使用像是 `kubectl` 的現有 Kubernetes 工具。

執行 `az aks use-dev-spaces` 命令會自動為您新增 `kubectl` 組態內容，因此 kubectl 應該已連線至您的 Azure Dev Spaces Kubernetes 叢集。 範例：
- 確認目前的內容：`kubectl config current-context`
- 列出所有可用的內容：`kubectl config get-contexts`。 
- 變更內容：`kubectl config use-context <context-name>`
- 檢視 Kubernetes 儀表板：執行 `kubectl proxy`，然後開啟瀏覽器前往此命令發出的位址 (將 `/ui` 附加至 URL 以瀏覽到 Kubernetes 儀表板)。
- 列出在名為「預設」的預設 Azure Dev Spaces 空間中執行的服務：`kubectl get services --namespace=default`

