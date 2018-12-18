---
title: Azure Stack 中的 Azure App Service 伺服器角色容量規劃 | Microsoft Docs
description: Azure Stack 中的 Azure App Service 伺服器角色容量規劃
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/28/2018
ms.author: sethm
ms.reviewer: anwestg
ms.openlocfilehash: 7cdcd8b7e9814c206255077fae0af2029fab6583
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078108"
---
# <a name="capacity-planning-for-azure-app-service-server-roles-in-azure-stack"></a>Azure Stack 中的 Azure App Service 伺服器角色容量規劃

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

若要在 Azure Stack 上設定已可在生產環境中使用的 Azure App Service 部署，您必須規劃您希望系統支援的容量。  

本文針對應用於任何生產環境部署的計算執行個體和計算 SKU 最小數目，提供指引。

您可以使用這些指導方針規劃您的 App Service 容量策略。 Azure Stack 的未來版本會為 App Service 提供高可用性選項。

| App Service 伺服器角色 | 建議的執行個體最小數目 | 建議的計算 SKU|
| --- | --- | --- |
| Controller | 2 | A1 |
| 前端 | 2 | A1 |
| 管理性 | 2 | A3 |
| 發行者 | 2 | A1 |
| Web 背景工作 - 共用 | 2 | A1 |
| Web 背景工作 - 專用 | 每層 2 個 | A1 |

## <a name="controller-role"></a>控制器角色

**建議的最小數目**：兩個 A1 標準的執行個體

Azure App Service 控制器通常會有 CPU、記憶體和網路資源取用量不高的問題。 不過，為確保高可用性，您必須擁有兩個控制器。 兩個控制器也是允許的控制器最大數目。 您可以在部署期間直接從安裝程式建立第二個網站控制器。

## <a name="front-end-role"></a>前端角色

**建議的最小數目**：兩個 A1 標準的執行個體

前端會根據 Web 背景工作的可用性將要求路由到 Web 背景工作。 為確保高可用性，您應該要擁有多個前端，而且可以擁有不只兩個。 若要進行容量規劃，請考慮每個核心每秒可以處理大約 100 個要求。

## <a name="management-role"></a>管理角色

**建議的最小數目**：兩個 A3 標準的執行個體

Azure App Service 管理角色負責 App Service Azure Resource Manager 和 API 端點、入口網站擴充功能 (管理員、租用戶、Functions 入口網站)，以及資料服務。 在生產環境中，管理伺服器角色通常只需要約 4 GB 的 RAM。 不過，在執行許多管理工作 (例如建立網站) 時，它可能會遇到較高的 CPU 使用量。 為確保高可用性，您應該對這個角色指派多個伺服器，而且每一部伺服器至少要有兩個核心。

## <a name="publisher-role"></a>發佈者角色

**建議的最小數目**：兩個 A1 標準的執行個體

如果同時發佈多個使用者，發佈者角色可能會遇到極高的 CPU 使用量。 為確保高可用性，請準備多個可用的發佈者角色。  發佈者只會處理 FTP/FTPS 流量。

## <a name="web-worker-role"></a>Web 背景工作角色

**建議的最小數目**：兩個 A1 標準的執行個體

為確保高可用性，您應該擁有至少四個 Web 背景工作角色，兩個用於共用網站模式，兩個用於您計劃要提供的每個專用背景工作層。 共用和專用計算模式會提供不同層級的服務給租用戶。 如果您的客戶有下列情形，您可能需要更多 Web 背景工作角色：

- 使用專用的計算模式背景工作層 (也就是需要大量資源)。
- 在共用計算模式下執行。

在使用者為專用的計算模式 SKU 建立 App Service 方案之後，該 App Service 方案中指定的 Web 背景工作數目將無法再供使用者使用。

若要在取用方案模型提供 Azure Functions 給使用者，您必須部署共用 Web 背景工作。

在決定要使用的共用 Web 背景工作角色數目時，請檢閱下列考量：

- **記憶體**：記憶體是 Web 背景工作角色最重要的資源。 從磁碟交換虛擬記憶體時若記憶體不足，將會影響網站的效能。 每個伺服器都需要將大約 1.2 GB 的 RAM 用於作業系統。 高於此閾值的 RAM 則可用於執行網站。
- **使用中網站的百分比**：一般而言，Azure Stack 部署上的 Azure App Service 中，大約會有 5% 的應用程式是使用中狀態。 不過，任何給定時刻的使用中應用程式百分比則有高有低。 在使用中應用程式比率為 5% 時，要在 Azure Stack 部署的 Azure App Service 中放置的應用程式最大數目應小於：
  - 20 乘上使用中網站數目 (5 x 20 = 100)。
- **平均記憶體使用量**：在生產環境中觀察到的應用程式記憶體使用量平均約為 70 MB。 利用此使用量，配置給所有 Web 背景工作角色電腦或 VM 的記憶體可依下列方式計算：

    已佈建的應用程式數目 * 70 MB * 5% - (Web 背景工作角色數目 * 1044 MB)

   例如，如果環境中有 5,000 個執行 10 個 Web 背景工作角色的應用程式，則每個 Web 背景工作角色 VM 都應該有 7060 MB 的 RAM：

   5,000 * 70 * 0.05 – (10 * 1044) = 7060 (= 大約 7 GB)

   如需有關新增更多背景工作執行個體的資訊，請參閱[新增更多背景工作角色](azure-stack-app-service-add-worker-roles.md)。

## <a name="file-server-role"></a>檔案伺服器角色

針對檔案伺服器角色，您可以使用獨立檔案伺服器進行開發和測試，例如在 Azure Stack 開發套件上部署 Azure App Service 時，您可以使用此範本 - <https://aka.ms/appsvconmasdkfstemplate>。 對於生產環境用途，則應該使用預先設定的 Windows 檔案伺服器或預先設定的非 Windows 檔案伺服器。

在生產環境中，檔案伺服器角色會遇到大量的磁碟 I/O。 該角色存放了使用者網站的所有內容與應用程式檔案，因此您應該為該角色預先設定下列其中一項：

- Windows 檔案伺服器
- Windows 檔案伺服器叢集
- 非 Windows 檔案伺服器
- 非 Windows 檔案伺服器叢集
- NAS (網路連接儲存裝置) 裝置。

如需詳細資訊，請參閱[佈建檔案伺服器](azure-stack-app-service-before-you-get-started.md#prepare-the-file-server)。

## <a name="next-steps"></a>後續步驟

[開始使用 Azure Stack 上的 App Service 之前](azure-stack-app-service-before-you-get-started.md)
