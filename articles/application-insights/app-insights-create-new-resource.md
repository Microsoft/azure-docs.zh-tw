---
title: 建立新的 Azure Application Insights 資源 | Microsoft Docs
description: 針對新的即時應用程式手動設定 Application Insights 監視。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 878b007e-161c-4e36-8ab2-3d7047d8a92d
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 12/02/2016
ms.author: mbullwin
ms.openlocfilehash: 59bb8564613e9a0cebda00c2c847283ff218b882
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35294712"
---
# <a name="create-an-application-insights-resource"></a>建立 Application Insights 資源
Azure Application Insights 會在 Microsoft Azure「資源」中顯示您應用程式的相關資料。 因此，建立新的資源是屬於[設定 Application Insights 以監視新應用程式][start]的一環。 在許多情況下，建立資源可以由 IDE 自動完成。 但在某些情況下，您需要手動建立資源 - 例如，讓應用程式的開發和生產組建有各自可用的資源。

建立資源後，您會取得其檢測金鑰，並將該金鑰用來設定應用程式中的 SDK。 資源索引鍵會將遙測連結到資源。

## <a name="sign-up-to-microsoft-azure"></a>註冊 Microsoft Azure
如果您還沒有 [Microsoft 帳戶，請立即申請](http://live.com)。 (如果您使用 Outlook.com、OneDrive、Windows Phone 或 XBox Live 等服務，就會有 Microsoft 帳戶)。

此外您也需要 [Microsoft Azure](http://azure.com) 訂用帳戶。 如果您的小組或組織擁有 Azure 訂用帳戶，則擁有者就可以使用您的 Windows Live ID 將您加入該訂用帳戶。 您只需針對使用的項目付費。 預設的基本方案有一定的免費數量可作為實驗用途。

當您可以存取訂用帳戶時，請在 [http://portal.azure.com](https://portal.azure.com) 中使用您的 Live ID 登入 Application Insights。

## <a name="create-an-application-insights-resource"></a>建立 Application Insights 資源
在 [portal.azure.com](https://portal.azure.com)中新增 Application Insights 資源：

![按一下 [新增]，然後按一下 [Application Insights]](./media/app-insights-create-new-resource/01-new.png)

* **應用程式類型**會影響您在 [概觀] 刀鋒視窗中看到的內容，以及[計量瀏覽器][metrics]中提供的屬性。 如果沒有看到您的應用程式類型，請選擇 [一般]。
* **訂用帳戶** 是您在 Azure 中的付款帳戶。
* **資源群組** 可讓您輕鬆管理屬性，例如存取控制。 如果您已經建立其他 Azure 資源，可以選擇將這個新的資源放到同一個群組。
* **位置** 是我們保留您資料的地方。
* **釘選到儀表板板** 可在 Azure 首頁上放置資源的快速存取圖格。 建議使用。

建立您的應用程式後，會開啟新的刀鋒視窗。 此刀鋒視窗是您會在其中看到應用程式的效能和使用情況資料的位置。 

若要在下次登入 Azure 時返回該應用程式，請在開始面板 (主畫面) 上尋找應用程式的快速啟動圖格。 或按一下 [瀏覽] 以尋找它。

## <a name="copy-the-instrumentation-key"></a>複製檢測金鑰
檢測金鑰會識別您所建立的資源。 您需要它以提供給 SDK。

![按一下 [基本功能]，按一下 [檢測金鑰]，CTRL+C](./media/app-insights-create-new-resource/02-props.png)

## <a name="install-the-sdk-in-your-app"></a>在應用程式中安裝 SDK
在應用程式中安裝 Application Insights SDK 核心。 此步驟高度仰賴於應用程式的類型。 

使用檢測金鑰來設定[您在應用程式中安裝的 SDK][start]。

SDK 包含不需撰寫任何程式碼，即可傳送遙測資料的標準模組。 若要更詳細追蹤使用者動作或診斷問題，請[使用 API][api] 來傳送您自己的遙測。

## <a name="monitor"></a>查看遙測資料
關閉 [快速入門] 刀鋒視窗，返回 Azure 入口網站中的應用程式刀鋒視窗。

按一下 [搜尋] 圖格以查看[診斷搜尋][diagnostic]，其中會顯示前幾個事件。 

如果您預期有更多資料，請在幾秒之後按一下 [重新整理]。

## <a name="creating-a-resource-automatically"></a>自動建立資源
您可以撰寫 [PowerShell 指令碼](app-insights-powershell.md) 來自動建立資源。

## <a name="next-steps"></a>後續步驟
* [建立儀表板](app-insights-dashboards.md)
* [診斷搜尋](app-insights-diagnostic-search.md)
* [探索度量](app-insights-metrics-explorer.md)
* [撰寫分析查詢](app-insights-analytics.md)

<!--Link references-->

[api]: app-insights-api-custom-events-metrics.md
[diagnostic]: app-insights-diagnostic-search.md
[metrics]: app-insights-metrics-explorer.md
[start]: app-insights-overview.md

