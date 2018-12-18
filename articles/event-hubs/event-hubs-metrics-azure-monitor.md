---
title: Azure 監視器中的 Azure 事件中樞計量 (預覽) | Microsoft Docs
description: 使用 Azure 監視來監視事件中樞
services: event-hubs
documentationcenter: .NET
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: ''
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/16/2018
ms.author: shvija
ms.openlocfilehash: a2b6fbae4a7b49c5596d5f2a6f9305181a0717d3
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46979959"
---
# <a name="azure-event-hubs-metrics-in-azure-monitor-preview"></a>Azure 監視器中的 Azure 事件中樞計量 (預覽)

「事件中樞」計量可提供您 Azure 訂用帳戶中「事件中樞」資源的狀態。 您可以使用一組豐富的計量資料，來評估事件中樞的整體健康情況 (不僅是命名空間層級，還包括實體層級)。 這些統計資料相當重要，因為它們可協助您監視事件中樞的狀態。 計量也可協助您針對根本原因問題進行疑難排解，而不需連絡 Azure 支援服務。

「Azure 監視器」提供統一的使用者介面，可供您監視各個不同的 Azure 服務。 如需詳細資訊，請參閱 [Microsoft Azure 中的監視](../monitoring-and-diagnostics/monitoring-overview.md)和 GitHub 上的 [Retrieve Azure Monitor metrics with .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) (使用 .NET 擷取 Azure 監視計量) 範例。

## <a name="access-metrics"></a>存取計量

Azure 監視器提供了多種方法供您存取計量。 您可以透過 [Azure 入口網站](https://portal.azure.com)來存取計量，或使用「Azure 監視器」API (REST 和 .NET) 和分析解決方案 (例如 Operation Management Suite 和「事件中樞」) 來存取計量。 如需詳細資訊，請參閱[監視 Azure 監視器所收集的資料](../monitoring/monitoring-data-collection.md)。

計量是預設啟用的功能，您可以存取最近 30 天的資料。 如果您需要延長這些資料的保留時間，您可以將計量資料封存到 Azure 儲存體帳戶。 此功能可於 Azure 監視器的[診斷設定](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md#diagnostic-settings)中進行設定。

## <a name="access-metrics-in-the-portal"></a>在入口網站中存取計量

您可以在 [Azure 入口網站](https://portal.azure.com)中監視一段時間的計量。 下列範例示範如何檢視帳戶層級的成功要求及連入要求：

![][1]

您也可以直接透過命名空間來存取計量。 若要這樣做，請選取您的命名空間，然後按一下 [計量 (預覽)]。 若要顯示篩選至事件中樞範圍的計量，請選取事件中樞，然後按一下 [計量 (預覽)]。

針對支援維度的計量，您必須使用所需的維度值來進行篩選，如以下範例所示：

![][2]

## <a name="billing"></a>計費

在預覽版「Azure 監視器」中使用計量目前是免費的。 不過，如果您使用內嵌計量資料的額外解決方案，可能就需支付這些解決方案的使用費。 例如，如果您將計量資料封存到 Azure 儲存體帳戶，就要支付 Azure 儲存體的使用費。 如果您將計量資料串流到 Log Analytics 進行進階分析，則也需支付 Azure 的費用。

下列計量會提供您服務健康狀態的概觀。 

> [!NOTE]
> 我們正在取代許多計量，因為它們移至不同的名稱下。 這可能需要您更新參考。 此後不再支援標記「已取代」關鍵字的計量。

所有計量值都會每分鐘傳送至「Azure 監視器」。 時間細微性會定義呈現計量值的時間間隔。 針對所有「事件中樞」計量，支援的時間間隔為 1 分鐘。

## <a name="request-metrics"></a>要求計量

計算資料和管理作業要求的數目。

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
| 傳入的要求 (預覽) | 在指定的期間內，向「Azure 事件中樞」服務提出的要求數目。 <br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName |
| 成功的要求 (預覽)   | 在指定的期間內，向「Azure 事件中樞」服務提出的成功要求數目。 <br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName |
| 伺服器錯誤 (預覽) | 在指定的期間內，因「Azure 事件中樞」服務發生錯誤而未處理的要求數目。 <br/><br/>單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName |
|使用者錯誤 (預覽)|在指定的期間內，因使用者錯誤而未處理的要求數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|節流的要求 (預覽)|因為超出輸送量單位使用量而節流的要求數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|已超出配額錯誤 (預覽)|超出可用配額的要求數目。 如需有關「事件中樞」配額的詳細資訊，請參閱[這篇文章](event-hubs-quotas.md)。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|

## <a name="throughput-metrics"></a>輸送量計量

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
|節流的要求 (預覽)|因為超出輸送量單位使用量而節流的要求數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|

## <a name="message-metrics"></a>訊息計量

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
|傳入訊息 (預覽)|在指定的期間內，傳送到「事件中樞」的事件或訊息數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|傳出訊息 (預覽)|在指定的期間內，從「事件中樞」擷取的事件或訊息數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|傳入的位元組 (預覽)|在指定的期間內，傳送到「Azure 事件中樞」服務的位元組數目。<br/><br/> 單位：位元組 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|傳出的位元組 (預覽)|在指定的期間內，從「Azure 事件中樞」服務擷取的位元組數目。<br/><br/> 單位：位元組 <br/> 彙總類型：總計 <br/> 維度：EntityName|

## <a name="connection-metrics"></a>連接計量

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
|ActiveConnections (預覽)|命名空間及實體上的作用中連線數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|已開啟的連線 (預覽)|已開啟的連線數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|已關閉的連線 (預覽)|已關閉的連線數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|

## <a name="event-hubs-capture-metrics"></a>事件中樞擷取計量

當您針對事件中樞啟用「擷取」功能時，即可監視「事件中樞」的「擷取」計量。 下列計量說明當您啟用「擷取」時可以監視的項目。

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
|擷取待辦項目 (預覽)|尚未擷取到所選目的地的位元組數目。<br/><br/> 單位：位元組 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|已擷取的訊息 (預覽)|在指定的期間內，擷取到所選目的地的訊息或事件數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|已擷取的位元組 (預覽)|在指定的期間內，擷取到所選目的地的位元組數目。<br/><br/> 單位：位元組 <br/> 彙總類型：總計 <br/> 維度：EntityName|

## <a name="metrics-dimensions"></a>計量維度

「Azure 事件中樞」針對「Azure 監視器」中的計量支援下列維度。 將維度新增到您的計量中是選擇性的。 如果不新增維度，則會在命名空間層級指定計量。 

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
|EntityName| 「事件中樞」可在命名空間底下支援事件中樞實體。|

## <a name="next-steps"></a>後續步驟

* 請參閱 [Azure 監視概觀](../monitoring-and-diagnostics/monitoring-overview.md)。
* GitHub 上的[使用 .NET 擷取 Azure 監視計量](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) \(英文\) 範例。 

如需事件中樞的詳細資訊，請造訪下列連結：

* 開始使用[事件中樞教學課程](event-hubs-dotnet-standard-getstarted-send.md)
* [事件中樞常見問題集](event-hubs-faq.md)
* [使用事件中樞的完整範例應用程式](https://github.com/Azure/azure-event-hubs/tree/master/samples)

[1]: ./media/event-hubs-metrics-azure-monitor/event-hubs-monitor1.png
[2]: ./media/event-hubs-metrics-azure-monitor/event-hubs-monitor2.png



