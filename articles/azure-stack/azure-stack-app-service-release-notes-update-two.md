---
title: App Service on Azure Stack 更新 2 版本資訊 | Microsoft Docs
description: 了解適用於 App Service on Azure Stack 更新 2 的新功能、已知問題，以及可下載更新的位置。
services: azure-stack
documentationcenter: ''
author: apwestgarth
manager: stefsch
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/18/2018
ms.author: anwestg
ms.reviewer: brenduns
ms.openlocfilehash: be09773a1ce3e80547d9e5f0e9de2a2d9e093c60
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38970912"
---
# <a name="app-service-on-azure-stack-update-2-release-notes"></a>App Service on Azure Stack 更新 2 版本資訊

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

這些版本資訊說明 Azure App Service on Azure Stack 更新 2 的增強功能和修正，以及任何已知問題。 已知問題分為直接與部署、更新程序相關的問題，以及組建 (安裝後) 的相關問題。

> [!IMPORTANT]
> 在部署 Azure App Service 1.2 之前，請先將 1804 更新套用到您的 Azure Stack 整合式系統，或部署最新的 Azure Stack 開發套件。
>
>

## <a name="build-reference"></a>建置參考

App Service on Azure Stack 更新 2 組建編號是 **72.0.13698.10**

### <a name="prerequisites"></a>先決條件

> [!IMPORTANT]
> 新的 Azure App Service on Azure Stack 部署現在需要[三主體萬用字元憑證](azure-stack-app-service-before-you-get-started.md#get-certificates)，因為現在已改進在 Azure App Service 中處理 Kudu 之 SSO 的方式。 新主體為 **\*.sso.appservice.\<region\>.\<domainname\>.\<extension\>**
>
>

開始部署之前，請參閱[「在您開始之前」文件](azure-stack-app-service-before-you-get-started.md)。

### <a name="new-features-and-fixes"></a>新功能和修正

Azure App Service on Azure Stack 更新 2 包含下列改良功能和修正：

- **App Service 租用戶、系統管理員、Functions 入口網站和 Kudu 工具**的更新。 與 Azure Stack 入口網站 SDK 版本保持一致。

- 用來改善可靠性和錯誤訊息以利進行常見問題診斷的核心服務更新。

- **下列應用程式架構和工具的更新**：
  - 已新增 .Net Framework 4.7.1
  - 已新增 **Node.JS** 版本：
    - NodeJS 6.12.3
    - NodeJS 8.9.4
    - NodeJS 8.10.0
    - NodeJS 8.11.1
  - 已新增 **NPM** 版本：
    - 5.6.0
  - 已將 .Net Core 元件更新為與公用雲端中的 Azure App Service 一致。
  - 已更新 Kudu

- 已啟用部署位置的自動交換功能 - [設定自動交換](https://docs.microsoft.com/azure/app-service/web-sites-staged-publishing#configure-auto-swap)

- 已啟用在生產環境中測試的功能 - [在生產環境中測試的簡介](https://azure.microsoft.com/resources/videos/introduction-to-azure-websites-testing-in-production-with-galin-iliev/)

- 已啟用 Azure Functions Proxy - [使用 Azure Functions Proxy](https://docs.microsoft.com/azure/azure-functions/functions-proxies)

- 已新增下列功能的 App Service 管理員擴充功能 UX 支援：
  - 密碼輪替
  - 憑證輪替
  - 系統認證輪替
  - 連接字串輪替

### <a name="known-issues-post-installation"></a>已知問題 (安裝後)

- 將應用程式服務部署在現有的虛擬網路中且只能在私人網路上使用檔案伺服器時，背景工作角色無法連線到檔案伺服器。

如果您選擇要部署到現有的虛擬網路並以內部 IP 位址連線到檔案伺服器，便必須新增輸出安全性規則，以啟用背景工作角色子網路與檔案伺服器之間的 SMB 流量。 若要這樣做，請移至管理入口網站中的 WorkersNsg，然後使用下列屬性新增輸出安全性規則：
 * 來源：任何
 * 來源連接埠範圍：*
 * 目的地：IP 位址
 * 目的地 IP 位址範圍：檔案伺服器的 IP 範圍
 * 目的地連接埠範圍：445
 * 通訊協定：TCP
 * 動作：允許
 * 優先順序：700
 * 名稱：Outbound_Allow_SMB445

### <a name="known-issues-for-cloud-admins-operating-azure-app-service-on-azure-stack"></a>雲端管理員操作 Azure App Service on Azure Stack 時的已知問題

請參閱 [Azure Stack 1804 版本資訊](azure-stack-update-1804.md)中的文件

## <a name="next-steps"></a>後續步驟

- 如需 Azure App Service 的概觀，請參閱 [Azure App Service on Azure Stack 概觀](azure-stack-app-service-overview.md)。
- 如需如何準備部署 App Service on Azure Stack 的詳細資訊，請參閱[開始使用 App Service on Azure Stack 之前](azure-stack-app-service-before-you-get-started.md)。
