---
title: Azure Application Insights 交易診斷 | Microsoft Docs
description: Application Insights 端對端交易診斷
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: TBD
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 01/19/2018
ms.reviewer: sdash
ms.author: mbullwin
ms.openlocfilehash: df88e9025da305701dc7168f663cad2e8f5ac738
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42819380"
---
# <a name="unified-cross-component-transaction-diagnostics"></a>整合跨元件交易診斷

整合診斷體驗會自動地將所有 Application Insights 監視元件的伺服器端遙測資料相互關聯為單一檢視。 多個資源是否各自擁有檢測金鑰並不重要。 Application Insights 能偵測基礎關聯性，讓您輕鬆診斷出導致交易緩慢或失敗的應用程式元件、相依性或例外狀況。

## <a name="what-is-a-component"></a>什麼是元件？

元件是分散式/微服務應用程式中可獨立部署的組件。 開發人員和作業小組能在程式碼層級檢視或存取這些應用程式元件所產生的遙測資料。

* 元件不同於 SQL、EventHub 等小組/組織可能無法存取的「可觀察」外部相依性 (程式碼或遙測資料)。
* 元件能在任何數量的伺服器/角色/容器執行個體上執行。
* 元件可以是不同的 Application Insights 檢測金鑰 (即使訂用帳戶不同)，也可以是回報給單一 Application Insights 檢測金鑰的不同角色。 新體驗能顯示所有元件的詳細資料，不論設定方式為何。

> [!NOTE]
> * **遺失相關項目連結？** 所有相關遙測都位於左側的[頂端](#cross-component-transaction-chart)和[底部](#all-telemetry-with-this-Operation-Id)區段中。 

## <a name="transaction-diagnostics-experience"></a>交易診斷體驗
此檢視有四大重要組件：結果清單、跨元件交易圖表、此作業所有相關遙測的時間序列清單、左側任何選定的遙測項目的詳細資料窗格。

![重要組件](media/app-insights-e2eTxn-diagnostics/4partsCrossComponent.png)

## <a name="cross-component-transaction-chart"></a>跨元件交易圖表

該圖表提供時間軸，以水平橫條呈現要求的持續時間和元件之間的相依性。 所有收集到的例外狀況也會標示在時間軸上。

* 這個圖表的上方資料列代表進入點，也就是此交易中呼叫之第一個元件的連入要求。 持續時間是交易完成所需的總時間。
* 所有外部相依性的呼叫都是不可折疊的簡單資料列，並以圖示代表相依性類別。
* 其他元件的呼叫是可折疊的資料列。 每個資料列都會對應到在元件叫用的特定作業。
* 按照預設，您選取的要求、相依性或例外狀況會顯示在右側。
* 選取任何資料列即可[在右側查看詳細資料](#details-of-the-selected-telemetry)。 

> [!NOTE]
其他元件的呼叫有兩個資料列：一個資料列代表來自呼叫端元件的輸出呼叫 (相依性)，另一個資料列則對應至被呼叫元件的輸入要求。 主要圖示和樣式分明的持續時間橫條有助於區分兩者。

## <a name="all-telemetry-with-this-operation-id"></a>此作業識別碼的所有遙測

此區段會以此交易所有相關遙測的時間序列顯示一般清單檢視。 此外，也會顯示交易圖表中未顯示的自訂事件和追蹤。 您可以篩選這份清單，以顯示由特定元件/呼叫產生的遙測。 您可以選取清單中的任何遙測項目，並在[右側查看對應的詳細資料](#details-of-the-selected-telemetry)。

![所有遙測資料的時間序列](media/app-insights-e2eTxn-diagnostics/allTelemetryDrawerOpened.png)

## <a name="details-of-the-selected-telemetry"></a>所選遙測的詳細資料

這個可摺疊的窗格會顯示交易圖表或清單中任何選取項目的詳細資料。 [全部顯示] 能列出所有收集到的標準屬性。 任何自訂屬性都會分別列在標準集合下方。 按一下堆疊追蹤視窗下面的 [...] 可取得複製追蹤的選項。 [開啟分析工具追蹤] 或 [開啟偵錯快照集] 會在對應的詳細資料窗格中顯示程式碼層級的診斷資料。

![例外狀況詳細資料](media/app-insights-e2eTxn-diagnostics/exceptiondetail.png)

## <a name="search-results"></a>搜尋結果

這個可摺疊的窗格會顯示符合篩選準則的其他結果。 按一下任何結果，即可更新上面所列 3 個區段的個別詳細資料。 我們試著找出最可能具有所有元件可用詳細資料的範例 (即使任何一個已有生效的取樣功能)。 這些項目會顯示為「建議」範例。

![搜尋結果](media/app-insights-e2eTxn-diagnostics/searchResults.png)

## <a name="profiler-and-snapshot-debugger"></a>分析工具和快照集偵錯工具

[Application Insights 分析工具](app-insights-profiler.md)或[快照集偵錯工具](app-insights-snapshot-debugger.md)能協助您從程式碼層級診斷效能和失敗問題。 透過這項體驗，只要按一下按鈕，您就可以查看任何元件的分析工具追蹤或快照集。

如果無法使用分析工具，請連絡 **serviceprofilerhelp@microsoft.com**

如果無法使用快照偵錯工具，請連絡 **snapshothelp@microsoft.com**

![分析工具整合](media/app-insights-e2eTxn-diagnostics/profilerTraces.png)

## <a name="faq"></a>常見問題集

*我在圖表中看到一個元件，其他元件只顯示為外部相依性，無法確切得知這些元件的內部狀況。*

可能的原因：

* 您是否曾使用 Application Insights 檢測其他元件？
* 使用的是否為最新的穩定版 Application Insights SDK？
* 如果這些元件是個別的 Application Insights 資源，您是否擁有存取其遙測資料的必要權限？

如果您擁有存取權限，而且元件是由最新的 Application Insights SDK 進行檢測，請透過右上角的意見反應管道告訴我們。

*我看到重複的相依性資料列。這是預期行為嗎？*

現階段我們會分別顯示輸出相依性呼叫和輸入要求。 這兩項呼叫看起來通常會一樣，唯一的差異在於持續時間值會因網路來回時間而有所不同。 主要圖示和樣式分明的持續時間橫條有助於區分兩者。 用這樣的方法呈現資料是否會混淆？ 請提供意見給我們！

*不同的元件執行個體之間有時鐘誤差嗎？*

交易圖表中的時間軸已調整過時鐘誤差。 您可以在詳細資料中或使用 Analytics 查看確切的時間戳記。

*為什麼新體驗遺漏大部分的相關項目查詢？*

原先的設計就是如此。 左側 (上方和下方區段) 已有全部元件的所有相關項目。 新體驗的左側未涵蓋兩項相關項目：此事件前後五分鐘的所有遙測資料和使用者時間軸。
