---
title: Azure PowerShell 範例 - App Service | Microsoft Docs
description: Azure PowerShell 範例 - App Service
services: app-service
documentationcenter: app-service
author: syntaxc4
manager: erikre
editor: ggailey777
tags: azure-service-management
ms.assetid: b48d1137-8c04-46e0-b430-101e07d7e470
ms.service: app-service
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: app-service
ms.date: 03/08/2017
ms.author: cfowler
ms.custom: mvc
ms.openlocfilehash: ba2bd2b185c395e54f2f085317a424a2aa1b4421
ms.sourcegitcommit: 176c575aea7602682afd6214880aad0be6167c52
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/09/2018
ms.locfileid: "27713411"
---
# <a name="azure-powershell-samples"></a>Azure PowerShell 範例

下表包含使用 Azure PowerShell 所建置的 PowerShell 指令碼連結。

| | |
|-|-|
|**建立應用程式**||
| [建立可從 GitHub 部署的 Web 應用程式](./scripts/app-service-powershell-deploy-github.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 建立會從 GitHub 提取程式碼的 Azure Web 應用程式。 |
| [建立可從 GitHub 連續部署的 Web 應用程式](./scripts/app-service-powershell-continuous-deployment-github.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 建立可從 GitHub 連續部署程式碼的 Azure Web 應用程式。 |
| [建立 Web 應用程式並使用 FTP 部署程式碼](./scripts/app-service-powershell-deploy-ftp.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 建立 Azure Web 應用程式，並使用 FTP 從本機目錄上傳檔案。 |
| [建立 Web 應用程式並從本機 Git 存放庫部署程式碼](./scripts/app-service-powershell-deploy-local-git.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 建立 Azure Web 應用程式並設定從本機 Git 存放庫推送程式碼的作業。 |
| [建立 Web 應用程式並將程式碼部署至預備環境](./scripts/app-service-powershell-deploy-staging-environment.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 建立具有部署位置以供放置預備程式碼變更的 Azure Web 應用程式。 |
|**設定應用程式**||
| [將自訂網域對應至 Web 應用程式](./scripts/app-service-powershell-configure-custom-domain.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 建立 Azure Web 應用程式，並向它對應自訂網域名稱。 |
| [將自訂 SSL 憑證繫結至 Web 應用程式](./scripts/app-service-powershell-configure-ssl-certificate.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 建立 Azure Web 應用程式，並將自訂網域名稱的 SSL 憑證加以繫結。 |
|**調整應用程式**||
| [手動調整 Web 應用程式](./scripts/app-service-powershell-scale-manual.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 建立 Azure Web 應用程式，並將它調整到 2 個執行個體中。 |
| [透過高可用性架構將 Web 應用程式調整為全球可用](./scripts/app-service-powershell-scale-high-availability.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 在兩個不同的地理區域中建立兩個 Azure Web 應用程式，並使用 Azure 流量管理員讓它們可透過單一端點來使用。 |
|**將應用程式連線至資源**||
| [將 Web 應用程式連線至 SQL Database](./scripts/app-service-powershell-connect-to-sql.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 建立 Azure Web 應用程式和 SQL Database，然後將資料庫連接字串新增至應用程式設定。 |
| [將 Web 應用程式連線至儲存體帳戶](./scripts/app-service-powershell-connect-to-storage.md?toc=%2fpowershell%2fmodule%2ftoc.json)| 建立 Azure Web 應用程式和儲存體帳戶，然後將儲存體連接字串新增至應用程式設定。 |
|**備份與還原應用程式**||
| [備份 Web 應用程式](./scripts/app-service-powershell-backup-onetime.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 建立 Azure Web 應用程式並為其建立一次性的備份。 |
| [為 Web 應用程式建立排程備份](./scripts/app-service-powershell-backup-scheduled.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 建立 Azure Web 應用程式並為其建立排程備份。 |
| [刪除 Web 應用程式的備份](./scripts/app-service-powershell-backup-delete.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 刪除 Web 應用程式的現有備份。 |
|**監視應用程式**||
| [使用 Web 伺服器記錄監視 Web 應用程式](./scripts/app-service-powershell-monitor.md?toc=%2fpowershell%2fmodule%2ftoc.json) | 建立 Azure Web 應用程式、為其啟用記錄，並將記錄下載到本機電腦。 |
| | |
