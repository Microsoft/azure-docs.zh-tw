---
title: App Service 概觀：Azure Stack | Microsoft Docs
description: Azure Stack 上的 App Service 概觀
services: azure-stack
documentationcenter: ''
author: brenduns
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/28/2018
ms.author: brenduns
ms.reviewer: anwestg
ms.openlocfilehash: 42258a352b05d97341f20c13bf837e1398dd8979
ms.sourcegitcommit: d7725f1f20c534c102021aa4feaea7fc0d257609
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37099957"
---
# <a name="app-service-on-azure-stack-overview"></a>Azure Stack 概觀上的 App Service

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

Azure App Service on Azure Stack 是 Microsoft Azure 的平台即服務 (PaaS) 供應項目，提供給 Azure Stack 使用。 服務可讓您的客戶 (內部或外部) 為任何平台或裝置建立 Web、API 和 Azure Functions 應用程式。 他們可以將您的應用程式與內部部署應用程式整合，並將其商務程序自動化。 Azure Stack 雲端操作員可以使用所選的共用 VM 資源或專用的 VM，在完全受控的虛擬機器 (VM) 上執行客戶應用程式。

Azure App Service 可讓您自動執行商務程序及裝載雲端 API。 Azure App Service 是單一的整合式服務，可讓您將各種元件 (網站、RESTful API 和商務程序) 組合成單一解決方案。

## <a name="why-offer-azure-app-service-on-azure-stack"></a>為什麼提供 Azure App Service on Azure Stack？

以下是 App Service 的一些重要功能和能力︰

- **多種語言和架構**：App Service 有 ASP.NET、Node.js、Java、PHP 和 Python 的頂級支援。 您也可以在 App Service VM 上執行 Windows PowerShell 和其他指令碼或可執行檔。
- **DevOps 最佳化**：使用 GitHub、本機 Git 或 BitBucket 設定連續整合和部署。 透過測試和預備環境升級更新。 使用 Azure PowerShell 或跨平台命令列介面 (CLI)，在 App Service 中管理您的應用程式。
- **Visual Studio 整合**：Visual Studio 中的專用工具可簡化建立和部署應用程式。

## <a name="app-types-in-app-service"></a>App Service 中的應用程式類型

App Service 提供數個「應用程式類型」，而每個類型主要裝載特定工作負載：

- [Web Apps](https://docs.microsoft.com/azure/app-service-web/app-service-web-overview) 用於裝載網站和 Web 應用程式。
- [API Apps](https://docs.microsoft.com/azure/app-service-api/app-service-api-apps-why-best-platform) 用於裝載 RESTful API。
- Azure Functions 用於託管活動導向的無伺服器工作負載。

應用程式一詞在這裡是指專門用來執行工作負載的裝載資源。 以「Web 應用程式」為例，您可能已習慣將 Web 應用程式視為可一起提供瀏覽器功能的計算資源和應用程式程式碼。 但是在 App Service 中，Web 應用程式是 Azure Stack 提供用來裝載應用程式程式碼的計算資源。

您的應用程式可能是由多個各種不同的 App Service 應用程式所組成。 例如，如果您的應用程式包含 Web 前端和 RESTful API 後端，您可以：

- 將兩者 (前端和 API) 部署至單一 Web 應用程式
- 將前端程式碼部署至 Web 應用程式，而將後端程式碼部暑至 API 應用程式。

   ![使用 App Service 監視資料的概觀](media/azure-stack-app-service-overview/image01.png)

## <a name="what-is-an-app-service-plan"></a>什麼是 App Service 方案？

App Service 資源提供者所使用的程式碼與 Azure App Service 使用的一樣。 因此，有些通用概念值得一提。 在 App Service 中，應用程式的定價容器稱為 App Service 方案。 它代表用來保存您應用程式的專用虛擬機器組。 在指定訂用帳戶內，您可以有多個 App Service 方案。

在 Azure 中，有共用和專用背景工作。 共用背景工作支援裝載高密度的多租用戶應用程式，而且只有一組共用背景工作。 專用伺服器只可供一個租用戶使用且分成三種大小：小型、中型和大型。 內部部署客戶的需求永遠無法使用這些詞彙來描述。 在 Azure Stack 上的 App Service 中，資源提供者系統管理員可以定義他們想要提供的背景工作層。 您可以根據您的獨特的裝載需求，定義多組共用背景工作或不同的專用背景工作組。 透過使用這些背景工作層定義，他們接著就能定義自己的定價 SKU。

## <a name="portal-features"></a>入口網站功能

Azure Stack 上的 App Service 所使用的 UI 與 Azure App Service 使用的相同，後端也是如此。 不過，某些功能已停用且無法在 Azure Stack 中運作。 Azure Stack 中目前未提供那些功能所需的 Azure 特有預期或服務。

## <a name="next-steps"></a>後續步驟

- [開始使用 Azure Stack 上的 App Service 之前](azure-stack-app-service-before-you-get-started.md)
- [安裝 App Service 資源提供者](azure-stack-app-service-deploy.md)

您也可以嘗試其他[平台即服務 (PaaS) 服務](azure-stack-tools-paas-services.md)，例如 [SQL Server 資源提供者](azure-stack-sql-resource-provider-deploy.md)和 [MySQL 資源提供者](azure-stack-mysql-resource-provider-deploy.md)。
