---
title: Azure DevOps Project 概觀 | Microsoft Docs
description: 瞭解 Azure DevOps Project 的價值
services: devops-project
documentationcenter: ''
author: mlearned
manager: douge
editor: mlearned
ms.assetid: ''
ms.service: devops-project
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: ''
ms.date: 05/03/2018
ms.author: mlearned
ms.openlocfilehash: 39dffad597b8382dea4df6fa1b0726d9582d67d1
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44293621"
---
# <a name="overview-of-azure-devops-project"></a>Azure DevOps Project 概觀

Azure DevOps Project 可讓您輕鬆地開始使用 Azure。 DevOps Project 資源可協助您只透過幾個快速步驟，就能從 Azure 入口網站中，在所選擇的 Azure 服務上啟動您最喜愛的 App 類型。 DevOps Project 會設定您在開發、部署及監控應用程式時所需的一切。
DevOps Project 儀表板可讓您監控程式碼認可、建置和部署，全部都只要在 Azure 入口網站的單一檢視中就能完成。

## <a name="why-should-i-use-the-azure-devops-project"></a>為何應使用 Azure DevOps Project？

Azure DevOps Project 會自動設定對 Azure 的整個持續整合 (CI) 與持續傳遞 (CD) 管線。  您可從現有的程式碼開始，或使用其中一個提供的範例應用程式，然後將該應用程式快速部署到各種 Azure 服務，例如虛擬機器、App Service、Azure Container Service、Azure SQL Database 和 Azure Service Fabric。  

Azure DevOps Project 會執行 DevOps 管線初始組態設定的所有工作，包括從設定初始 Git 儲存機制、設定 CI/CD 管線、建立用於監控的 Application Insights 資源、以及提供整個解決方案的單一檢視，包括在 Azure 入口網站中建立 Azure DevOps Project 儀表板的所有工作。

您可以使用 Azure DevOps Project 進行以下作業：

* 將應用程式快速部署至 Azure
* 將 Azure CI/CD 管線的設定自動化
* 使用 DevOps Project 作為範本，以檢視及了解如何使用 Azure DevOps 將 CI/CD 正確設定至 Azure
* 開始對 Azure 使用 CI/CD 管線，然後進一步依據您的特定情況自訂發行管線

## <a name="how-do-i-use-the-azure-devops-project"></a>如何使用 Azure DevOps Project？

Azure DevOps Project 可從 Azure 入口網站取得。  您可以從入口網站建立 Azure DevOps Project 資源，就像建立任何其他 Azure 資源一樣。  DevOps Project 針對各種組態設定選項提供逐步執行精靈型體驗。  

您可以在初始設定中選擇幾個組態設定選項。  這些選項包含︰

* 使用提供的範例 App，或使用自己的程式碼
* 選取應用程式語言
* 依據語言選擇應用程式架構
* 選取 Azure 服務 (部署目標)
* Azure DevOps 組織 (新的或現有的)
* 選擇 Azure 訂用帳戶
* 挑選 Azure 服務的位置
* 從 Azure 服務的多種定價層中選擇

使用 Azure DevOps Project 之後，您也可以在 Azure 入口網站上，從單一位置將所有資源從 Azure DevOps Project 儀表板中刪除。

## <a name="azure-devops-project-and-azure-devops-integration"></a>Azure DevOps Project 與 Azure DevOps 整合

DevOps Project 是由 Azure DevOps 提供技術。  DevOps Project 可將 Azure DevOps 中將 CI/CD 設定至 Azure 的所有所需工作自動化。  Git 儲存機制會以新的或現有的 Azure DevOps 組織建立。  DevOps Project 可將範例應用程式，或您現有的程式碼認可至新的 Git 儲存機制。  自動化功能也會為組建建立一個 CI 觸發程序，讓每個新的程式碼認可都能起始組建。  DevOps Project 也會建立 CD 觸發程序，且將會把每個成功建立的新組建部署到您選擇的 Azure 服務中。  您可以自訂組建和發行管線以用於其他案例。  您也可以複製組建和發行管線以在其他專案中使用。

建立 DevOps Project 之後，您可以：

* 自訂組建和發行管線
* 使用提取要求管理程式碼流程，並維持高品質
* 在合併程式碼以提高品質之前，先測試及建置每次認可
* 追蹤您應用程式的 Project 待處理項目與問題

## <a name="how-do-i-start-using-the-azure-devops-project"></a>如何開始使用 Azure DevOps Project？

* [開始使用 Azure DevOps Project](https://docs.microsoft.com/azure/devops-project/azure-devops-project-github)

## <a name="azure-devops-project-videos"></a>Azure DevOps Project 影片

* [使用 Azure DevOps Project 建立 CI/CD](https://channel9.msdn.com/Events/Connect/2017/T174/player/)
