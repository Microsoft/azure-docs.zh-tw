---
title: Azure 監視器中的 Azure 轉送計量 (預覽) | Microsoft Docs
description: 使用 Azure 監視器監視 Azure 轉送
services: service-bus-relay
documentationcenter: .NET
author: spelluru
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-bus-relay
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/23/2018
ms.author: spelluru
ms.openlocfilehash: cffc84cdb86c01f5ca892b987a23dc3ded611dd9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986945"
---
# <a name="azure-relay-metrics-in-azure-monitor-preview"></a>Azure 監視器中的 Azure 轉送計量 (預覽)

Azure 轉送計量會提供您 Azure 訂用帳戶中的資源狀態。 您可以使用豐富的計量資料集，評估轉送資源的整體健康狀態，不只在命名空間層級，還有實體層級。 這些統計資料相當重要，因為它們可協助您監視 Azure 轉送的狀態。 計量也可協助針對問題的根本原因進行疑難排解，而不需要連絡 Azure 支援。

「Azure 監視器」提供統一的使用者介面，可供您監視各個不同的 Azure 服務。 如需詳細資訊，請參閱 [Microsoft Azure 中的監視](../monitoring-and-diagnostics/monitoring-overview.md)和 GitHub 上的 [Retrieve Azure Monitor metrics with .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) (使用 .NET 擷取 Azure 監視計量) 範例。

## <a name="access-metrics"></a>存取計量

Azure 監視器提供了多種方法供您存取計量。 您可以透過 [Azure 入口網站](https://portal.azure.com)來存取計量，或使用「Azure 監視器」API (REST 和 .NET) 和分析解決方案 (例如 Operation Management Suite 和「事件中樞」) 來存取計量。 如需詳細資訊，請參閱[監視 Azure 監視器所收集的資料](../monitoring/monitoring-data-collection.md)。

計量是預設啟用的功能，您可以存取最近 30 天的資料。 如果您需要延長這些資料的保留時間，您可以將計量資料封存到 Azure 儲存體帳戶。 此功能可於 Azure 監視器的[診斷設定](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md#diagnostic-settings)中進行設定。

## <a name="access-metrics-in-the-portal"></a>在入口網站中存取計量

您可以在 [Azure 入口網站](https://portal.azure.com)中監視一段時間的計量。 下列範例示範如何檢視帳戶層級的成功要求及連入要求：

![][1]

您也可以直接透過命名空間來存取計量。 若要這樣做，請選取您的命名空間，然後按一下 [Metrics (Peview)] \(計量 (預覽))。 

如需支援維度的計量，您必須使用所需的維度值來進行篩選。

## <a name="billing"></a>計費

在預覽版「Azure 監視器」中使用計量目前是免費的。 不過，如果您使用內嵌計量資料的額外解決方案，可能就需支付這些解決方案的使用費。 例如，如果您將計量資料封存到 Azure 儲存體帳戶，就要支付 Azure 儲存體的使用費。 如果您將計量資料串流到 Log Analytics 進行進階分析，則也需支付 Log Analytics 的費用。

下列計量會提供您服務健康狀態的概觀。 

> [!NOTE]
> 我們正在取代許多計量，因為它們移至不同的名稱下。 這可能需要您更新參考。 此後不再支援標記「已取代」關鍵字的計量。

所有計量值都會每分鐘傳送至「Azure 監視器」。 時間細微性會定義呈現計量值的時間間隔。 所有的 Azure 轉送計量都支援 1 分鐘的時間間隔。

## <a name="connection-metrics"></a>連接計量

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
| ListenerConnections-Success (預覽) | 指定時段內，接聽程式成功連接到 Azure 轉送的數目。 <br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|ListenerConnections-ClientError (預覽)|指定時段內，接聽程式連線發生的用戶端錯誤數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|ListenerConnections-ServerError (預覽)|指定時段內，接聽程式連線發生的伺服器錯誤數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|SenderConnections-Success (預覽)|指定時段內，寄件者成功連線的數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|SenderConnections-ClientError (預覽)|指定時段內，寄件者連線發生的用戶端錯誤數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|SenderConnections-ServerError (預覽)|指定時段內，寄件者連線發生的伺服器錯誤數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|ListenerConnections-TotalRequests (預覽)|指定時段內，接聽程式連線的總數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|SenderConnections-TotalRequests (預覽)|指定時段內，寄件者發出的連線要求。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|ActiveConnections (預覽)|指定時段內，作用中的連線數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|ActiveListeners (預覽)|指定時段內，作用中的接聽程式數目。<br/><br/> 單位：計數 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|ListenerDisconnects (預覽)|指定時段內，中斷連線的接聽程式數目。<br/><br/> 單位：位元組 <br/> 彙總類型：總計 <br/> 維度：EntityName|
|SenderDisconnects (預覽)|指定時段內，中斷連線的寄件者數目。<br/><br/> 單位：位元組 <br/> 彙總類型：總計 <br/> 維度：EntityName|

## <a name="memory-usage-metrics"></a>記憶體使用量計量

| 度量名稱 | 說明 |
| ------------------- | ----------------- |
|BytesTransferred (預覽)|在指定時段內，傳輸的位元組數目。<br/><br/> 單位：位元組 <br/> 彙總類型：總計 <br/> 維度：EntityName|

## <a name="metrics-dimensions"></a>計量維度

Azure 轉送支援下列的 Azure 監視器計量維度。 將維度新增至計量為選擇性。 如果不新增維度，會在命名空間層級指定計量。 

|維度名稱|說明|
| ------------------- | ----------------- |
|EntityName| Azure 轉送支援命名空間下的訊息實體。|

## <a name="next-steps"></a>後續步驟

請參閱 [Azure 監視概觀](../monitoring-and-diagnostics/monitoring-overview.md)。

[1]: ./media/relay-metrics-azure-monitor/relay-monitor1.png




