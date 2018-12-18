---
title: Azure Application Insights | Microsoft Docs
description: ''
services: application-insights
documentationcenter: ''
author: eternovsky
manager: carmonm
ms.assetid: ea2a28ed-4cd9-4006-bd5a-d4c76f4ec20b
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 08/08/2018
ms.reviewer: mbullwin
ms.author: Evgeny.Ternovsky
ms.openlocfilehash: 6d19c64c74a01a012bac3d867e2d65393ae21cd3
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45577766"
---
# <a name="correlating-application-insights-data-with-custom-data-sources"></a>使自訂資料來源與 Application Insights 資料相互關聯

Application Insights 會收集數個不同的資料類型：例外狀況、追蹤、網頁檢視和其他項目。 雖然這通常足以用來檢查您的應用程式效能、可靠性和使用量，但在某些情況下，讓儲存在 Application Insights 中的資料與其他完全自訂的資料集相互關聯會相當實用。

您需要自訂資料的情況可能包括：

- 資料擴充或查閱資料表：例如，在伺服器名稱中增添伺服器擁有者，以及能在其中找到該伺服器名稱的實驗室位置 
- 與非 Application Insights 資料來源相互關聯：例如，讓網路商店上的購買資料與您的購買履約服務相互關聯，以判斷寄送時間的預估準確度 
- 完全自訂的資料：我們有許多客戶喜愛 Log Analytics 資料平台 (用來支援 Application Insights) 的查詢語言和效能，而且想要使用它來查詢與 Application Insights 完全不相關的資料。 例如，追蹤屬於智慧型家電安裝的太陽能面板效能，如[此處]( http://blogs.catapultsystems.com/cfuller/archive/2017/10/04/using-log-analytics-and-a-special-guest-to-forecast-electricity-generation/)所述。

## <a name="how-to-correlate-custom-data-with-application-insights-data"></a>如何使自訂資料與 Application Insights 資料相互關聯 

由於 Application Insights 受到功能強大的 Log Analytics 資料平台所支援，因此我們能夠使用 Log Analytics 的完整功能來內嵌資料。 然後，我們會使用 “join” 運算子撰寫查詢，此運算子會使此自訂資料與 Log Analytics 中的可用資料相互關聯。 

## <a name="ingesting-data"></a>內嵌資料

在本節中，我們將檢閱如何將您的資料放入 Log Analytics。

請一一遵循[這些指示]( https://docs.microsoft.com/azure/log-analytics/log-analytics-quick-collect-azurevm)及「建立工作區」步驟，以佈建新的 Log Analytics 工作區 (如果您沒有的話)。

開始將資料傳送至 Log Analytics。 有幾種選項可用：

- 若要使用同步機制，您可以直接呼叫[資料收集器 API](https://docs.microsoft.com/azure/log-analytics/log-analytics-data-collector-api)，或使用我們的邏輯應用程式連接器 – 只要尋找 [Azure Log Analytics]，然後選擇 [傳送資料] 選項即可：

 ![選擇和動作的螢幕擷取畫面](./media/app-insights-custom-data-correlation/01-logic-app-connector.png)  

- 若要使用非同步選項，請使用「資料收集器 API」來建置處理管線。 如需詳細資訊，請參閱[本文](https://docs.microsoft.com/azure/log-analytics/log-analytics-create-pipeline-datacollector-api)。

## <a name="correlating-data"></a>使資料相互關聯

Application Insights 會以 Log Analytics 資料平台作為基礎。 因此我們可以使用[跨資源聯結](https://docs.microsoft.com/azure/log-analytics/log-analytics-cross-workspace-search)，讓內嵌至 Log Analytics 的所有資料與 Application Insights 資料相互關聯。

例如，我們可以將實驗室詳細目錄和位置內嵌至名為 “LabLocations_CL” 的資料表 (位於名為 “myLA” 的 Log Analytics 工作區中)。 如果我們接著要檢閱在 Application Insights 應用程式 "myAI" 中追蹤的要求，並使處理要求的機器名稱與先前所述自訂資料表中儲存的這些機器位置相互關聯，我們可以從Application Insights 或 Log Analytics 環境中執行下列查詢：

```
app('myAI').requests
| join kind= leftouter (
    workspace('myLA').LabLocations_CL
    | project Computer_S, Owner_S, Lab_S
) on $left.cloud_RoleInstance == $right.Computer
```

## <a name="next-steps"></a>後續步驟

- 請參閱[資料收集器 API](https://docs.microsoft.com/azure/log-analytics/log-analytics-data-collector-api) 參考。
- 取得[跨資源聯結](https://docs.microsoft.com/azure/log-analytics/log-analytics-cross-workspace-search)的詳細資訊。
