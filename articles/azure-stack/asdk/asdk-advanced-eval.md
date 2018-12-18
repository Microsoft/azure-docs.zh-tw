---
title: 進階 Azure Stack 評估工作 | Microsoft Docs
description: 本文會說明進階 Azure Stack 評估工作。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/16/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: 0b65f37310b911184c05fd8fb8554eeb824814b8
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49364807"
---
# <a name="advanced-azure-stack-development-kit-evaluation-tasks"></a>進階 Azure Stack 開發套件評估工作
在熟悉基本 Azure Stack 開發套件 (ASDK) 服務功能與能力後，您可以藉由測試更進階的案例來進一步加深您對 Azure Stack 的了解。 這些更進階的評估工作會完整記錄在 Azure Stack 操作員文件中。

> [!NOTE]
> 雖然許多操作員工作都同時支援 ASDK 和多節點 Azure Stack 生產部署，但並非所有使用案例皆支援 ASDK 部署。 請參閱 [ASDK 與多節點 Azure Stack 差異](asdk-what-is.md#asdk-and-multi-node-azure-stack-differences)以了解詳細資訊。

## <a name="delegate-offers-in-azure-stack"></a>在 Azure Stack 中委派供應項目
身為 Azure Stack 操作員，您往往會想要了解如何讓其他人負責建立供應項目和註冊使用者。 例如，如果您是服務提供者，或許需要請轉銷商註冊客戶並代表您管理客戶。 或者，如果您是公司內中央 IT 群組的一員，或許您會想要讓子公司在不需要您介入的情況下註冊使用者。

[在 Azure Stack 中委派供應項目](.\.\azure-stack-delegated-provider.md)會透過讓您得以接觸和管理超過您可以直接接觸和管理的使用者數量，來協助您進行這些工作。 

## <a name="make-sql-databases-available-to-your-azure-stack-users"></a>將 SQL 資料庫提供給您的 Azure Stack 使用者
身為 Azure Stack 操作員，您可以建立供應項目以讓您的使用者 (租用戶) 建立 SQL 資料庫，以搭配其雲端原生應用程式、網站與工作負載使用。 透過將對這些自訂隨選雲端式資料庫的存取權提供給您的使用者，您可以節省其時間與資源。 

使用 SQL Server 資源提供者配接器，[讓 SQL 資料庫可提供給 Azure Stack 使用者](.\.\azure-stack-tutorial-sql-server.md)作為 Azure Stack 的服務。 安裝資源提供者後，將它連接至一或多個 SQL Server 執行個體。

## <a name="make-web-and-api-apps-available-to-your-azure-stack-users"></a>將 Web 與 API 應用程式提供給您的 Azure Stack 使用者
身為 Azure Stack 操作員，您可以建立供應項目，以讓您的使用者 (租用戶) 建立 Azure Functions 和 Web，還有 API 應用程式。 透過將對這些隨選雲端式應用程式的存取權提供給您的使用者，您可以節省其時間與資源。

部署 App Service 資源提供者以[將 Web 與 API 應用程式提供給您的 Azure Stack 使用者](.\.\azure-stack-tutorial-app-service.md)

## <a name="next-steps"></a>後續步驟
[深入了解 Azure Stack 整合系統的供應項目服務](.\.\azure-stack-offer-services-overview.md)