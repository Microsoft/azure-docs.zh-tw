---
title: Azure 監視器中的 Azure 服務匯流排計量 (預覽) | Microsoft Docs
description: 使用 Azure 監視器監視服務匯流實體
services: service-bus-messaging
documentationcenter: .NET
author: spelluru
manager: timlt
ms.service: service-bus-messaging
ms.topic: article
ms.date: 09/24/2018
ms.author: spelluru
ms.openlocfilehash: 293cde00e53171e848263df8564ec85f273c1a40
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166328"
---
# <a name="azure-service-bus-metrics-in-azure-monitor-preview"></a>Azure 監視器中的 Azure 服務匯流排計量 (預覽)

服務匯流排計量會提供您 Azure 訂用帳戶中的資源狀態。 您可以使用豐富的計量資料集，評估服務匯流排資源的整體健康狀態，不只在命名空間層級，還有實體層級。 這些統計資料相當重要，因為它們可協助您監視服務匯流排的狀態。 計量也可協助針對問題的根本原因進行疑難排解，而不需要連絡 Azure 支援。

「Azure 監視器」提供統一的使用者介面，可供您監視各個不同的 Azure 服務。 如需詳細資訊，請參閱 [Microsoft Azure 中的監視](../monitoring-and-diagnostics/monitoring-overview.md)和 GitHub 上的 [Retrieve Azure Monitor metrics with .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) (使用 .NET 擷取 Azure 監視計量) 範例。

> [!IMPORTANT]
> 在未與實體互動的時間達 2 小時之後，計量將會開始顯示 "0" 作為值，直到該實體不再閒置為止。

## <a name="access-metrics"></a>存取計量

Azure 監視器提供了多種方法供您存取計量。 您可以透過 [Azure 入口網站](https://portal.azure.com)來存取計量，或使用 Azure 監視器 API (REST 和 .NET) 和分析解決方案 (例如 Log Analytics 和事件中樞) 來存取計量。 如需詳細資訊，請參閱[監視 Azure 監視器所收集的資料](../monitoring/monitoring-data-collection.md)。

計量是預設啟用的功能，您可以存取最近 30 天的資料。 如果您需要延長這些資料的保留時間，您可以將計量資料封存到 Azure 儲存體帳戶。 此功能可於 Azure 監視器的[診斷設定](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md#diagnostic-settings)中進行設定。

## <a name="access-metrics-in-the-portal"></a>在入口網站中存取計量

您可以在 [Azure 入口網站](https://portal.azure.com)中監視一段時間的計量。 下列範例示範如何檢視帳戶層級的成功要求及連入要求：

![][1]

您也可以直接透過命名空間來存取計量。 若要這樣做，請選取您的命名空間，然後按一下 [Metrics (Peview)] \(計量 (預覽))。 若要顯示篩選至實體範圍的計量，請選取實體，然後按一下[Metrics (Peview)] \(計量 (預覽))。

![][2]

如需支援維度的計量，您必須使用所需的維度值來進行篩選。

## <a name="billing"></a>計費

在預覽版「Azure 監視器」中使用計量是免費的。 不過，如果您使用內嵌計量資料的額外解決方案，可能就需支付這些解決方案的使用費。 例如，如果您將計量資料封存到 Azure 儲存體帳戶，就要支付 Azure 儲存體的使用費。 如果您將計量資料串流到 Log Analytics 進行進階分析，則也需支付 Log Analytics 的費用。

下列計量會提供您服務健康狀態的概觀。 

> [!NOTE]
> 我們正在取代許多計量，因為它們移至不同的名稱下。 這可能需要您更新參考。 此後不再支援標記「已取代」關鍵字的計量。

所有計量值都會每分鐘傳送至「Azure 監視器」。 時間細微性會定義用來呈現計量值的時間間隔。 所有的服務匯流排計量都支援 1 分鐘的時間間隔。

## <a name="request-metrics"></a>要求的計量

計算資料和管理作業要求的數目。

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
| 傳入的要求 (預覽) | 在指定時段內，向服務匯流排服務提出的要求數目。 <br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|成功的要求 (預覽)|在指定時段內，向服務匯流排服務提出的成功要求數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|伺服器錯誤 (預覽)|在指定時段內，服務匯流排服務中因錯誤而未處理的要求數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|使用者錯誤 (預覽 - 請參閱以下子區段)|在指定的期間內，因使用者錯誤而未處理的要求數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|節流的要求 (預覽)|因為超過使用量而節流的要求數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|

### <a name="user-errors"></a>使用者錯誤

下列兩種錯誤類型會分類為使用者錯誤：

1. 用戶端錯誤 (在 HTTP 中是 400 錯誤)。
2. 處理訊息時發生的錯誤，例如 [MessageLockLostException](/dotnet/api/microsoft.azure.servicebus.messagelocklostexception)。


## <a name="message-metrics"></a>訊息計量

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
|傳入訊息 (預覽)|在指定時段內，傳送至服務匯流排的事件或訊息數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|外送訊息 (預覽)|在指定時段內，從服務匯流排接收的事件或訊息數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|

## <a name="connection-metrics"></a>連接計量

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
|ActiveConnections (預覽)|命名空間及實體上的作用中連線數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|已開啟的連線 (預覽)|已開啟的連線數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|已關閉的連線 (預覽)|已關閉的連線數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName |

## <a name="resource-usage-metrics"></a>資源使用量計量

> [!NOTE] 
> 下列計量僅適用於**進階**層。 

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
|每個命名空間的 CPU 使用量 (預覽)|命名空間的 CPU 使用量百分比。<br/><br/> 單位：百分比 <br/> 彙總類型：最大 <br/> 維度：EntityName|
|每個命名空間的記憶體大小使用量 (預覽)|命名空間的記憶體使用量百分比。<br/><br/> 單位：百分比 <br/> 彙總類型：最大 <br/> 維度：EntityName|

## <a name="metrics-dimensions"></a>計量維度

Azure 服務匯流排支援下列的 Azure 監視器計量維度。 將維度新增至計量為選擇性。 如果不新增維度，會在命名空間層級指定計量。 

|維度名稱|說明|
| ------------------- | ----------------- |
|EntityName| 服務匯流排支援命名空間下的訊息實體。|

## <a name="next-steps"></a>後續步驟

請參閱 [Azure 監視概觀](../monitoring-and-diagnostics/monitoring-overview.md)。

[1]: ./media/service-bus-metrics-azure-monitor/service-bus-monitor1.png
[2]: ./media/service-bus-metrics-azure-monitor/service-bus-monitor2.png


