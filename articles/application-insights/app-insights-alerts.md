---
title: 在 Azure Application Insights 中設定警示 | Microsoft Docs
description: 獲知回應時間緩慢、例外狀況，以及 Web 應用程式中的其他效能或使用量變更。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.reviewer: lagayhar
ms.assetid: f8ebde72-f819-4ba5-afa2-31dbd49509a5
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 03/14/2017
ms.author: mbullwin
ms.openlocfilehash: 1061f356d75037bae440a5289413b2b5d17af1c9
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43246947"
---
# <a name="set-alerts-in-application-insights"></a>在 Application Insights 中設定警示
[Azure Application Insights][start] 可在 Web 應用程式中發生效能或使用量計量變更時對您發出警示。 

Application Insights 會在[多種平台][platforms]上監視即時應用程式，協助您診斷效能問題，以及了解使用模式。

共有三種警示︰

* **計量警示**會在計量超出某些期間的臨界值 (例如回應時間、例外狀況計數、CPU 使用量或頁面檢視) 的時候通知您。 
* [**Web 測試**][availability]會在您的網站無法在網際網路上使用或回應速度很慢時通知您。 [深入了解][availability]。
* [**主動診斷**](app-insights-proactive-diagnostics.md)會自動設定成通知您異常的效能模式。

在本文中，我們著重於計量警示。

## <a name="set-a-metric-alert"></a>設定計量警示
開啟 [警示規則] 刀鋒視窗，然後使用 [新增] 按鈕。 

![在 [警示規則] 刀鋒視窗中，按一下 [新增警示]。 將您的應用程式設定為要測量的資源，提供警示的名稱，然後選擇計量。](./media/app-insights-alerts/01-set-metric.png)

* 設定其他屬性之前的資源。 **選擇 "(元件)" 資源** 。
* 您提供的警示名稱必須為資源群組 (不只是您的應用程式) 中的唯一名稱。
* 請小心注意系統要求您輸入臨界值時所使用的單位。
* 如果您勾選 [電子郵件擁有者] 方塊，系統會透過電子郵件，將警示傳給每個可以存取此資源群組的人員。 若要展開這一組人員，請將他們新增至 [資源群組或訂用帳戶](app-insights-resources-roles-access-control.md) (而非資源)。
* 如果您指定 [其他電子郵件]，系統會將警示傳送給這些人員或群組 (無論您是否核取 [電子郵件擁有者] 方塊)。 
* 如果您已設定回應通知的 Web 應用程式，請設定 [Webhook 位址](../monitoring-and-diagnostics/insights-webhooks-alerts.md)。 系統會在警示啟動和解決時加以呼叫。 (不過請注意，查詢參數目前不會當作 Webhook 屬性傳遞)。
* 您可以停用或啟用警示：請參閱位於刀鋒視窗頂端的按鈕。

<bpt id="p1">*</bpt>I don't see the Add Alert button.<ept id="p1">*</ept> 

* 您是否使用組織帳戶？ 如果您有這個應用程式資源的擁有者或參與者存取權，您可以設定警示。 請看一下 [存取控制] 刀鋒視窗。 [深入了解存取控制][roles]。

> [!NOTE]
> 在 [警示] 刀鋒視窗中，您會看到已經設定警示︰[主動診斷](app-insights-proactive-failure-diagnostics.md)。 自動警示會監視要求失敗率這一個特定度量。 除非您決定要停用主動警示，否則不需要設定自己的要求失敗率警示。 
> 
> 

## <a name="see-your-alerts"></a>查看您的警示
當警示在非作用中與作用中之間變更狀態時，您會收到電子郵件。 

每個警示目前的狀態都顯示在警示規則刀鋒視窗中。

警示下拉式清單中有最近活動的摘要：

![警示下拉式清單](./media/app-insights-alerts/010-alert-drop.png)

狀態變更的歷程記錄位於活動記錄檔中：

![在 [概觀] 刀鋒視窗中，按一下 [設定]、[稽核記錄檔]](./media/app-insights-alerts/09-alerts.png)

## <a name="how-alerts-work"></a>警示的運作方式
* 警示有三種狀態：「永遠不啟動」、「已啟動」和「已解決」。 「已啟動」表示您指定的條件在上次評估時為 true。
* 警示狀態變更時，會產生通知。 (如果警示條件在建立警示時已為 true，您可能在條件變為 false 之前都不會得到通知。)
* 如果您已勾選電子郵件方塊或提供電子郵件地址，則每個通知都會產生一封電子郵件。 您也可以於 [通知] 查看下拉式清單。
* 每次度量抵達時都會評估警示，除此之外則無。
* 評估會彙總前一個期間的度量，然後將其與臨界值進行比較，以決定新的狀態。
* 您選擇的期間會指定彙總度量的間隔。 它並不會影響評估警示的頻率：這會根據度量抵達的頻率而定。
* 如果經過一段時間後沒有特定度量的資料到達，此間距會在警示評估上，以及在度量總管的圖表上有不同的效果。 在計量總管中，如果沒看到資料的時間超過圖表的取樣間隔時間，則圖表會顯示 0 值。 但是以相同度量為基礎的警示不會重新接受評估，而且警示的狀態會維持不變。 
  
    當資料終於抵達時，圖表會跳回非零的值。 警示的評估作業會依您指定的期間，並根據可用的資料來進行。 如果新的資料點是期間內唯一可用的資料點，則彙總只會以該資料點為基礎。
* 即使您設定的期間較長，警示也可能會在警示和良好狀態之間經常變動。 如果度量值徘徊在臨界值附近，就會發生這種情形。 臨界值中沒有任何磁滯：轉換為警示時的值會與轉換為良好時的值相同。

## <a name="what-are-good-alerts-to-set"></a>哪些是好的設定警示？
這取決於您的應用程式。 開始時，最好不要設定太多度量。 花點時間查看您的應用程式執行時的度量圖表，以了解正常運作時的情形。 這個做法可協助您找出改善其效能的方式。 然後設定警示，在度量偏離正常區域時通知您。 

熱門的警示包括：

* [瀏覽器計量][client]，適合用於 Web 應用程式，尤其是瀏覽器**頁面載入時間**。 如果您的分頁有許多指令碼，您應該尋找**瀏覽器例外狀況**。 若要取得這些計量和警示，您必須設定[網頁監視][client]。
* Web 應用程式伺服器端的**伺服器回應時間**。 以及設定警示，注意這些計量，以查看高要求率時的差異是否不成比例：差異可能表示您的應用程式資源不足。 
* **伺服器例外狀況** - 若要查看它們，您只需要進行一些 [額外設定](app-insights-asp-net-exceptions.md)。

別忘了，[主動失敗率診斷](app-insights-proactive-failure-diagnostics.md)會自動監視應用程式以失敗碼回應要求的速率。 

## <a name="automation"></a>自動化
* [使用 PowerShell 自動設定警示](app-insights-powershell-alerts.md)
* [使用 Webhook 自動回應警示](../monitoring-and-diagnostics/insights-webhooks-alerts.md)

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="see-also"></a>另請參閱
* [可用性 Web 測試](app-insights-monitor-web-app-availability.md)
* [自動化設定警示](app-insights-powershell-alerts.md)
* [主動診斷](app-insights-proactive-diagnostics.md) 

<!--Link references-->

[availability]: app-insights-monitor-web-app-availability.md
[client]: app-insights-javascript.md
[platforms]: app-insights-platforms.md
[roles]: app-insights-resources-roles-access-control.md
[start]: app-insights-overview.md

