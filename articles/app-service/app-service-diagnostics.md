---
title: Azure App Service 診斷概觀 | Microsoft Docs
description: 了解如何使用 App Service 診斷對 Web 應用程式問題進行疑難排解。
keywords: App Service, Azure App Service, 診斷, 支援, Web 應用程式, 疑難排解, 自助
services: app-service
documentationcenter: ''
author: jen7714
manager: cfowler
editor: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/10/2017
ms.author: jennile
ms.openlocfilehash: 7ad205c75a02b496abe2cb910c7eb459cdb16c97
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38969233"
---
# <a name="azure-app-service-diagnostics-overview"></a>Azure App Service 診斷概觀 

當您執行 Web 應用程式時，必須為使用者因 500 錯誤向您反映網站已關閉的所有可能問題作好準備。 App Service 診斷是一種智慧型和互動式的體驗，可協助您對 Web 應用程式進行疑難排解，而且不需設定。 當您的 Web 應用程式發生問題時，App Service 診斷將點出問題，並以正確的資訊引導您更輕鬆且快速地對問題進行疑難排解並解決。 
 
這項體驗對於過去 24 小時內發生問題的 Web 應用程式最有幫助，但所有診斷圖形隨時都可供您分析。 其他的疑難排解工具及有用的文件和論壇的連結則位於右邊資料行。

App Service 診斷不僅適用於 Windows 上您的應用程式，也適用於 [Linux/容器](https://docs.microsoft.com/azure/app-service/containers/app-service-linux-intro)、[App Service Environment](https://docs.microsoft.com/azure/app-service/environment/intro) 和 [Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-overview) 上的應用程式。 

## <a name="open-app-service-diagnostics"></a>開啟 App Service 診斷

若要存取 App Service 診斷，請在 [Azure 入口網站](https://portal.azure.com)中瀏覽至您的 App Service 應用程式或 App Service Environment。 在左方導覽列中，按一下 [診斷並解決問題]。 

在 Azure Functions 中，瀏覽至您的函式應用程式，然後在頂端導覽列中按一下 [平台功能] 並從 [監視] 區段中選取 [診斷並解決問題]。 

![首頁](./media/app-service-diagnostics/Homepage1.png)

## <a name="health-checkup"></a>健康狀態檢查

如果您不知道您的 Web 應用程式發生什麼問題，或不知道從何處開始對問題進行疑難排解，可以從健康狀態檢查開始。 健康狀態檢查會分析您的 Web 應用程式，為您提供快速的互動式概觀將點出狀況良好或問題的癥結，告訴您應從何處調查問題。 其智慧型及互動式的介面可提供您進行疑難排解程序的指導方針。  

![健康狀態檢查](./media/app-service-diagnostics/HealthCheckup2.png)

如果偵測到的問題是過去 24 小時內偵測到的特定問題類別，您可以檢視完整的診斷報告，且 App Service 診斷可能會提示您檢視更多的疑難排解建議和後續步驟，以提供更具引導性質的體驗。

![疑難排解和後續步驟](./media/app-service-diagnostics/Troubleshooting3.png)

## <a name="tile-shortcuts"></a>圖格快速鍵

如果您清楚知道要尋找的是何種疑難排解資訊，圖格快速鍵將帶您直接前往您感興趣之問題類別的完整診斷報告。 相較於健康狀態檢查，以圖格快速鍵存取診斷計量的方式更直接，但較不具引導性質。 在圖格快速鍵當中，這也是您將找到 [診斷工具] 的位置，這些進階工具可協助您調查應用程式程式碼問題、速度緩慢、連接字串等相關問題。 

![圖格快速鍵](./media/app-service-diagnostics/TileShortcuts4.png)

## <a name="diagnostic-report"></a>診斷報告

執行[健康狀態檢查](#health-checkup)後是否需要詳細資訊，或是您已按一下其中一個[圖格快速鍵](#tile-shortcuts)，完整診斷報告將顯示過去 24 小時內相關圖形的計量。 如果您的應用程式曾經有停機狀況，時間表下方會顯示一個橙色列。 您可以選取其中一個橘色列來選取停機狀況，以查看關於該停機狀況的觀察及建議的疑難排解步驟。 

![診斷報告](./media/app-service-diagnostics/DiagnosticReport5.png)


## <a name="investigating-application-code-issues"></a>調查應用程式程式碼問題

因為許多應用程式問題與您應用程式程式碼中的問題相關，App Service 診斷與 [Application Insights](https://azure.microsoft.com/services/application-insights/) 整合，以反白顯示要與所選停機狀況相互關聯的例外狀況和相依性問題。 Application Insights 不必個別啟用。 

若要檢視 Application Insights 例外狀況和相依性，請選取 [關閉的 Web 應用程式] 或 [緩慢的 Web 應用程式] 圖格捷徑。 

![Application insights](./media/app-service-diagnostics/AppInsights6.png)

