---
title: 我使用行動服務，App Service 有何幫助？
description: 了解 App Service 為您現有的行動服務專案帶來哪些優勢。
services: app-service\mobile
documentationcenter: ios
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 26b68a11-8352-4f78-acd2-e4e0ec177781
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: 365f00ced38a1ddc20df211121fba43efff8ea87
ms.sourcegitcommit: d1f35f71e6b1cbeee79b06bfc3a7d0914ac57275
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/22/2018
ms.locfileid: "29460407"
---
# <a name="getting-started"> </a>我使用行動服務，App Service 有何幫助？
## <a name="overview"></a>概觀
您現有的行動服務很安全並將繼續提供支援。 不過，Azure App Service 平台為您的行動應用程式提供了行動服務目前所沒有的優點：

* 針對包含 Web 和行動用戶端的應用程式，提供更簡單、更容易且更符合成本效益的功能
* 新的主機功能包括 Web 工作、自訂 CName、更完善的監視
* 與流量管理員整合
* 除了混合式連線以外，還會使用 VNet 連線至內部部署資源和 VPN
* 使用 AppInsights 針對您的應用程式進行監視、警示和疑難排解
* 更廣泛的基礎計算資源與定價
* 內建自動調整、負載平衡，以及效能監視。
* 內建預備、備份、回復和實際環境測試功能

## <a name="new-hosting-features"></a>新的裝載功能
在 AzureApp Service 中，「行動應用程式」後端程式碼會在與 Web 應用程式和 API 應用程式相同的容器中執行。 您可以利用此容器中的所有功能，包括行動服務中目前不存在的功能：

* 透過 Web 工作加入連續執行的後端邏輯
* 確保您的後端程式碼持續不斷執行
* 使用自訂 CName，為您的行動裝置後端端點提供易記且穩定的名稱
* 使用流量管理員，異地擴充您的 app
* 包含任何您想要的程式庫和封裝。
* (適用於 .NET) 使用 ASP.NET 的任何功能，包括 MVC
* (適用於 Node.js) 使用 Node 生態系統的任何純 JavaScript 程式庫，包含一般 MVC 程式庫。

## <a name="access-on-premises-data-using-vnet"></a>使用 VNet 存取內部部署資料
現今有了行動服務，您已可使用混合式連線來存取內部部署資源。 不過，有些情況還是慣用 VPN 解決方案。 透過 Azure App Service ，您可以將 Azure VNet 用於您的行動應用程式後端程式碼。

## <a name="use-your-favorite-backend-language"></a>使用您最愛的後端語言
*Azure App Service* 提供對 ASP.NET 與 Node.js 平台更廣泛、更豐富的支援，包含對最新執行階段的存取。

## <a name="set-up-automatic-scale"></a>設定自動調整
有了行動服務，後端程式碼的所有執行個體都已在小型 VM 上執行。 *Azure App Service* 可讓您從更豐富的選項中選取 VM 的大小。 您可以根據各種效能計量，快速相應增加或相應放大，以處理任何傳入的客戶負載。

## <a name="be-in-the-know"></a>處於「知情」狀態
利用監視及警示即時對問題做出反應，以自動通知您和您的團隊。 整合 AppInsights 的進階應用程式分析和監視功能，更深入了解如何執行您的行動應用程式。 使用 Azure App Service ，您現在可以根據多種效能計量 (以程式設計方式或透過 Azure 入口網站) 設定警示。

## <a name="keep-your-assets-safe"></a>讓您的資產安全無虞
自動備份您的後端和資料庫。 您的程式碼和資料可免於遭受災害並可輕鬆還原，讓您有信心地執行您的業務。

## <a name="ready-stage-go"></a>準備上場 ！
利用 Azure App Service ，您現在可以為行動應用程式建立多個私人測試和預備環境。 您可以在部署之前，使用這些環境來執行測試。 不需停機即可切換至生產環境。 Web 應用程式已預先載入，可確保最佳的客戶體驗。

您可以藉由遵循此[教學課程](app-service-mobile-migrating-from-mobile-services.md)，開始將 App Service 運用於您現有的行動服務。
