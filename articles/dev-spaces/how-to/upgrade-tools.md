---
title: 如何升級 Azure Dev Spaces 工具 | Microsoft Docs
titleSuffix: Azure Dev Spaces
services: azure-dev-spaces
ms.service: azure-dev-spaces
ms.component: azds-kubernetes
author: ghogen
ms.author: ghogen
ms.date: 07/03/2018
ms.topic: article
ms.technology: azds-kubernetes
description: 在 Azure 上使用容器和微服務快速進行 Kubernetes 開發
keywords: Docker、Kubernetes、Azure、AKS、Azure Container Service、容器
manager: douge
ms.openlocfilehash: 10838528d79caa49abc541b4fcd38fea3c24d68f
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "42141277"
---
# <a name="how-to-upgrade-azure-dev-spaces-tools"></a>如何升級 Azure Dev Spaces 工具

如果有提供新的版本，而您目前已在使用 Azure Dev Spaces，則可能需要升級您的 Azure Dev Spaces 用戶端工具。

## <a name="update-the-azure-cli"></a>更新 Azure CLI

當您更新為最新的 Azure CLI 時，也會取得最新版本的 Dev Spaces CLI 擴充功能。

您不需要解除安裝舊版本，只需在 [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest) 找到適合的下載項目。


## <a name="update-the-dev-spaces-cli-extension-and-command-line-tools"></a>更新 Dev Spaces CLI 擴充功能和命令列工具

執行以下命令：

```cmd
az aks use-dev-spaces -n <your-aks-cluster> -g <your-aks-cluster-resource-group> --update
```

## <a name="update-the-vs-code-extension"></a>更新 VS Code 擴充功能

安裝之後，會自動更新擴充功能。 您可能需要重新載入擴充功能以使用新功能。 在 VS Code 中，開啟 [擴充功能] 窗格，選擇 [Azure Dev Spaces] 擴充功能，然後選擇 [重新載入]。

## <a name="update-the-visual-studio-extension"></a>更新 Visual Studio 擴充功能

正如其他擴充功能和更新一樣，當適用於 Kubernetes 的 Visual Studio Tools 有可用的更新時 (包括 Azure Dev Spaces)，Visual Studio 會通知您。 在畫面右上角尋找旗標圖示。

若要更新 Visual Studio 中的工具，請選擇 [工具] > [擴充功能和更新] 功能表項目，然後在左側選擇 [更新]。 尋找 [適用於 Kubernetes 的 Visual Studio Tools] ，然後選擇 [更新] 按鈕。

## <a name="next-steps"></a>後續步驟

藉由建立新叢集以測試新工具。 嘗試 [Azure Dev Spaces](/azure/dev-spaces) 的快速入門與教學課程。

> [!WARNING]
> 現有叢集上的 Azure Dev Spaces 不會立即修補，因此若要確保在所有 Azure 部署中使用最新版本，請在升級工具後建立新叢集。
