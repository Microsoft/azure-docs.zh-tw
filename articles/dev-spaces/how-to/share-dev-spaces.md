---
title: 如何共用 Azure 開發人員空間 | Microsoft Docs
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
ms.openlocfilehash: 1d7d665c20baddb3a94bfe53568ab56a5a961630
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "42141279"
---
# <a name="share-azure-dev-spaces"></a>分享 Azure 開發人員空間

透過 Azure 開發人員空間，您可以與小組中其他人共用您的開發人員空間。 每位開發人員可在自己的空間工作，而不必擔心中斷其他人的工作。 此外，在同一個空間工作可以讓您端對端地測試程式碼，而不需要建立模擬或模擬相依性。 請參閱[了解小組開發](../team-development-nodejs.md)指南以取得詳細資訊。

## <a name="set-up-a-dev-space-for-multiple-developers"></a>為多個開發人員設定開發人員空間

1. 在 Azure 中建立開發人員空間。 選擇 [.NET Core 和 VS Code](../get-started-netcore.md)、[.NET Core 和 Visual Studio](../get-started-netcore-visualstudio.md) 或 [Node.js 和 VS Code](../get-started-nodejs.md)。 您必須具有所選 Azure 訂用帳戶的擁有者或參與者存取權。
1. 針對每個小組成員，將 Azure 開發人員空間的**資源群組**設定為[授與參與者存取權](/azure/active-directory/role-based-access-control-configure)。 您可以執行此命令來檢查開發人員空間的資源群組：`azds list-up`
1. 要求小組成員**選取開發人員空間**，以在其中進行開發。
     * **命令列或 VS Code**：若要查看您可以存取的現有 Azure 開發人員空間：`azds space list`。 若要選取開發人員空間：`azds space select`。
     * **Visual Studio IDE**：在 Visual Studio 中開啟專案，然後從啟動設定的下拉式清單中選取 [Azure 開發人員空間]。 在開啟的對話方塊中，選取現有的叢集。

    ![Visual Studio 啟動設定的下拉式清單](../media/get-started-netcore-visualstudio/LaunchSettings.png)

## <a name="next-steps"></a>後續步驟

請參閱[了解小組開發](../team-development-nodejs.md)以取得詳細資訊。