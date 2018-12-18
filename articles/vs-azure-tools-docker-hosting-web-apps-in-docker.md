---
title: 將 ASP.NET Docker 容器部署到 Azure Container Registry (ACR) | Microsoft Docs
description: 了解如何使用 Visual Studio Tools for Docker，將 ASP.NET Core Web 應用程式部署到容器登錄
services: azure-container-service
documentationcenter: .net
author: mlearned
manager: douge
editor: ''
ms.assetid: e5e81c5e-dd18-4d5a-a24d-a932036e78b9
ms.service: azure-container-service
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/21/2018
ms.author: mlearned
ms.openlocfilehash: 58df17b17de1d93683875b68dd7c6c087bc6d16d
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38972303"
---
# <a name="deploy-an-aspnet-container-to-a-container-registry-using-visual-studio"></a>使用 Visual Studio 將 ASP.NET 容器部署到容器登錄
## <a name="overview"></a>概觀
Docker 是輕量級容器引擎，與虛擬機器在某些方面類似，您可以用它來裝載應用程式和服務。
本教學課程會引導您使用 Visual Studio，將容器化應用程式發佈至 [Azure Container Registry](https://azure.microsoft.com/services/container-registry)。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/dotnet/?utm_source=acr-publish-doc&utm_medium=docs&utm_campaign=docs)。

## <a name="prerequisites"></a>先決條件
若要完成本教學課程：

* 安裝包含 "ASP.NET 和 Web 開發" 工作負載的 [Visual Studio 2017](https://azure.microsoft.com/downloads/) 最新版本
* 安裝 [Docker for Windows](https://docs.docker.com/docker-for-windows/install/)

## <a name="1-create-an-aspnet-core-web-app"></a>1.建立 ASP.NET Core Web 應用程式
下列步驟會逐步引導您建立將在本教學課程中使用的基本 ASP.NET Core 應用程式。

[!INCLUDE [create-aspnet5-app](../includes/create-aspnet5-app.md)]

## <a name="2-publish-your-container-to-azure-container-registry"></a>2.將容器發佈至 Azure Container Registry
1. 在**方案總管**中以滑鼠右鍵按一下專案，並選擇 [發佈]。
2. 在 [發佈目標] 對話方塊中，選取 [容器登錄] 索引標籤。
3. 選擇 [新 Azure Container Registry] 然後按一下 [發佈]。
4. 在 [建立新的 Azure Container Registry] 中填入您想要的值。

    | 設定      | 建議的值  | 說明                                |
    | ------------ |  ------- | -------------------------------------------------- |
    | **DNS 首碼** | 全域唯一的名稱 | 用以唯一識別容器登錄的名稱。 |
    | **訂用帳戶** | 選擇您的訂用帳戶 | 要使用的 Azure 訂用帳戶。 |
    | **[資源群組](../articles/azure-resource-manager/resource-group-overview.md)** | myResourceGroup |  要在其中建立容器登錄的資源群組名稱。 選擇 [新增] 以建立新的資源群組。|
    | **[SKU](https://docs.microsoft.com/azure/container-registry/container-registry-skus)** | 標準 | 容器登錄的服務層  |
    | **登錄位置** | 接近您的位置 | 在[區域](https://azure.microsoft.com/regions/)中選擇您附近的 [位置]，或選擇將會使用容器登錄的其他服務所接近的位置。 |
    ![Visual Studio 的 [建立 Azure Container Registry] 對話方塊][0]
5. 按一下 [建立] 

您現在可以從登錄中，將容器提取至能夠執行 Docker 映像的任何主機，例如 [Azure 容器執行個體](./container-instances/container-instances-tutorial-deploy-app.md)。

[0]:./media/vs-azure-tools-docker-hosting-web-apps-in-docker/vs-acr-provisioning-dialog.png
