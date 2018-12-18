---
title: 建立已啟用 Apache Kafka 的 Azure 事件中樞 | Microsoft Docs
description: 使用 Azure 入口網站來建立「已啟用 Kafka 的 Azure 事件中樞」命名空間
services: event-hubs
documentationcenter: .net
author: basilhariri
manager: timlt
ms.service: event-hubs
ms.devlang: dotnet
ms.topic: article
ms.date: 08/16/2018
ms.author: bahariri
ms.openlocfilehash: 7ce12f9dcaa15ade95274419f99c13d5915dbaaa
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42145552"
---
# <a name="create-apache-kafka-enabled-event-hubs"></a>建立已啟用 Apache kafka 的事件中樞

Azure 事件中樞是一種巨量資料串流平台即服務 (PaaS)，每秒可內嵌數百萬個事件，並為即時分析和視覺效果提供低延遲性和高輸送量。

Azure 事件中樞為您提供 Kafka 端點。 此端點可讓您的事件中樞命名空間在原生狀態下了解 [Apache Kafka](https://kafka.apache.org/intro) 訊息通訊協定和 API。 您可使用這項功能，像使用 Kafka 主題一樣與您的事件中樞通訊，而無須變更通訊協定用戶端或執行自己的叢集。 事件中樞支援 [Apache Kafka 1.0 版](https://kafka.apache.org/10/documentation.html)和更新版本。

本文說明如何建立事件中樞命名空間，以及取得將 Kafka 應用程式連線至已啟用 Kafka 之事件中樞所需的連接字串。

## <a name="prerequisites"></a>必要條件

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

## <a name="create-a-kafka-enabled-event-hubs-namespace"></a>建立已啟用 Kafka 的事件中樞命名空間

1. 登入 [Azure 入口網站][Azure portal]，然後按一下畫面左上方的 [建立資源]。

2. 搜尋事件中樞，並選取選項，如下所示：
    
    ![在入口網站中搜尋事件中樞](./media/event-hubs-create-kafka-enabled/event-hubs-create-event-hubs.png)
 
3. 提供唯一名稱並在命名空間上啟用 Kafka。 按一下頁面底部的 [新增] 。
    
    ![建立命名空間](./media/event-hubs-create-kafka-enabled/create-kafka-namespace.png)
 
4. 建立命名空間後，在 [設定] 索引標籤上，按一下 [共用存取原則] 以取得連接字串。

    ![按一下 [共用存取原則]](./media/event-hubs-create/create-event-hub7.png)

5. 您可以選擇預設的 **RootManageSharedAccessKey**，或新增新的原則。 按一下原則名稱並複製連接字串。 
    
    ![選取原則](./media/event-hubs-create/create-event-hub8.png)
 
6. 將連接字串新增到應用程式組態。

您現在可以將事件從使用 Kafka 通訊協定的應用程式串流到事件中樞。

## <a name="next-steps"></a>後續步驟

若要深入了解事件中樞，請造訪下列連結：

* [從您的 Kafka 應用程式串流到事件中樞](event-hubs-quickstart-kafka-enabled-event-hubs.md)
* [了解適用於 Kafka 的事件中樞](event-hubs-for-kafka-ecosystem-overview.md)
* [了解事件中樞](event-hubs-what-is-event-hubs.md)


[Azure portal]: https://portal.azure.com/
