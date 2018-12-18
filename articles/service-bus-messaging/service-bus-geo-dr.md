---
title: Azure 服務匯流排地理災害復原 | Microsoft Docs
description: 如何使用地理區域，在 Azure 服務匯流排中進行容錯移轉並執行災害復原
services: service-bus-messaging
author: spelluru
manager: timlt
ms.service: service-bus-messaging
ms.topic: article
ms.date: 06/14/2018
ms.author: spelluru
ms.openlocfilehash: 42960222efb1ade1b48a1d0db56ae3fb0349d174
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43695386"
---
# <a name="azure-service-bus-geo-disaster-recovery"></a>Azure 服務匯流排地理災害復原

當整個 Azure 區域或資料中心 (如果未使用[可用性區域](../availability-zones/az-overview.md)) 遇到停機時，最重要的是資料處理作業能夠繼續在不同的區域或資料中心運作。 因此，「地理災害復原」和「異地複寫」對於任何企業而言都是重要的功能。 Azure 服務匯流排支援命名空間層級的地理災害復原和異地複寫。 

地理災害復原功能適用於全球的服務匯流排進階 SKU。 

## <a name="outages-and-disasters"></a>中斷與災害

請務必注意「中斷」和「災害」之間的差異。 「中斷」是暫時無法使用 Azure 服務匯流排，而且會影響服務的某些元件，例如訊息存放區，或甚至整個資料中心。 不過，修正問題之後，服務匯流排就可再次使用。 中斷通常不會導致訊息或其他資料遺失。 這類中斷的範例可能是資料中心停電。 某些中斷只是因為暫時性或網路問題而造成的短暫連線中斷。 

「災害」定義為永久或較長期遺失服務匯流排叢集、Azure 區域或資料中心。 區域或資料中心不一定能再次使用，也可能會關閉數小時或數天。 這類災害的範例包括火災、水災或地震。 會變成永久的災害可能會導致某些訊息、事件或其他資料遺失。 不過，在大部分情況下，應該不會遺失資料，而且在備份資料中心之後，就可以復原訊息。

Azure 服務匯流排的地理災害復原功能就是一個災害復原解決方案。 本文中所述的概念和工作流程適用於災害案例，不適用暫時性或暫時中斷。 如需 Microsoft Azure 中災害復原的詳細討論，請參閱[本文](/azure/architecture/resiliency/disaster-recovery-azure-applications)。   

## <a name="basic-concepts-and-terms"></a>基本概念與術語

災害復原功能會實作中繼資料災害復原，並依賴主要和次要災害復原命名空間。 請注意，地理災害復原功能僅適用於[進階 SKU](service-bus-premium-messaging.md)。 您不需要進行任何連接字串變更，因為連接是透過別名建立的。

本文中使用下列術語：

-  別名：所設定之災害復原設定的名稱。 別名提供單一穩定完整網域名稱 (FQDN) 連接字串。 應用程式會使用這個別名連接字串連接到命名空間。 

-  主要/次要命名空間：對應到別名的命名空間。 主要命名空間是「主動」並且會接收訊息 (這可以是現有或新的命名空間)。 次要命名空間是「被動」，並不會收到訊息。 這兩者間的中繼資料會進行同步處理，因此，這兩者均能順暢地接受訊息，而不需進行任何應用程式程式碼或連接字串變更。 若要確保只有主動命名空間會接收訊息，您必須使用別名。 

-  中繼資料：佇列、主題和訂用帳戶之類的實體，及其與命名空間關聯之服務的屬性。 請注意，只有實體及其設定會自動複寫。 不會複寫訊息。 

-  容錯移轉：啟用次要命名空間的程序。

## <a name="setup-and-failover-flow"></a>設定和容錯移轉流程

下一節是容錯移轉程序的概觀，並說明如何設定初始容錯移轉。 

![1][]

### <a name="setup"></a>設定

首先建立或使用現有的主要命名空間，以及新的次要命名空間，然後將這兩個命名空間配對。 此配對會為您提供可用來連接的別名。 由於您使用別名，因此不需變更連接字串。 只需將新的命名空間新增至您的容錯移轉配對。 最後，您應該新增某些監視來偵測是否需要容錯移轉。 在大部分情況下，服務是大型生態系統的一部分，因此不太可能自動容錯移轉，容錯移轉通常都必須與其餘子系統或基礎結構同步執行。

### <a name="example"></a>範例

在此案例的一個範例中，請考慮銷售點 (POS) 解決方案，它會發出訊息或事件。 服務匯流排會將這些事件傳遞至某些對應或重新格式化解決方案，然後將對應資料轉送到另一個系統以進一步處理。 屆時，這些系統都會裝載於相同的 Azure 區域。 關於何時容錯移轉及容錯移轉哪個部分的決策，取決於基礎結構中的資料流量。 

您可以使用監視系統或使用自訂監視解決方案來自動容錯移轉。 不過，這類自動化會採用額外的計劃和工作，這部分不在本文的範圍內。

### <a name="failover-flow"></a>容錯移轉流程

如果您初始化容錯移轉，需要兩個步驟：

1. 如果發生另一個中斷，您會希望能夠再次進行容錯移轉。 因此，請設定另一個被動命名空間，並更新配對。 

2. 一旦主要命名空間再次可供使用之後，從先前的主要命名空間提取訊息。 然後，針對地理復原設定以外的一般訊息使用該命名空間，或者刪除舊的主要命名空間。

> [!NOTE]
> 僅支援失敗轉接語意。 在此案例中，您容錯移轉然後與新的命名空間重新配對。 不支援容錯復原；例如，在 SQL 叢集。 

![2][]

## <a name="management"></a>管理性

如果您發生錯誤；例如，您在初始化安裝期間配對錯誤的區域，您可以隨時中斷兩個命名空間的配對。 如果您想要使用配對的命名空間作為一般命名空間，請刪除別名。

## <a name="use-existing-namespace-as-alias"></a>使用現有命名空間作為別名

如果在您的案例中無法變更生產者與消費者的連線，則您可重複使用命名空間名稱作為別名名稱。 請參閱[這裡的 GitHub 範例程式碼](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/SBGeoDR2/SBGeoDR_existing_namespace_name) \(英文\)。

## <a name="samples"></a>範例

[GitHub 上的範例](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/SBGeoDR2/)示範如何設定及初始化容錯移轉。 這些範例示範下列概念：

- 在 Azure Active Directory 中使用 Azure Resource Manager 搭配服務匯流排，以設定和啟用異地災害復原所需的 .NET 範例和設定。
- 執行範例程式碼所需的步驟。
- 如何使用現有命名空間作為別名。
- 另外透過 PowerShell 或 CLI 啟用地理災害復原的步驟。
- 使用別名從目前的主要或次要命名空間[傳送和接收](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/TestGeoDR/ConsoleApp1)。

## <a name="considerations"></a>考量

請注意此版本的下列考量：

1. 在您的容錯移轉規劃中，您也應該考慮時間因素。 例如，如果您中斷連線時間超過 15 到 20 分鐘，您可能會決定初始化容錯移轉。 
 
2. 未複寫任何資料這個事實表示未複寫目前作用中工作階段。 此外，重複的偵測和排程訊息可能無法運作。 新的工作階段、新的排程訊息及新的重複項目將能夠運作。 

3. 容錯移轉複雜分散式基礎結構應該至少[演練](/azure/architecture/resiliency/disaster-recovery-azure-applications#disaster-simulation)一次。 

4. 同步處理實體可能需要一些時間，大約每分鐘 50-100 個實體。 訂用帳戶和規則也算是實體。 

## <a name="availability-zones-preview"></a>可用性區域 (預覽)

服務匯流排進階 SKU 也支援[可用性區域](../availability-zones/az-overview.md)，在 Azure 區域內提供錯誤隔離位置。 

> [!NOTE]
> 只有在**美國中部**、**美國東部 2** 和**法國中部**區域才支援可用性區域預覽。

使用 Azure 入口網站時，只能在新的命名空間上啟用可用性區域。 服務匯流排不支援移轉現有的命名空間。 在命名空間上啟用區域備援之後，便無法停用。

![3][]

## <a name="next-steps"></a>後續步驟

- 請參閱地理災害復原在[這裡的 REST API 參考](/rest/api/servicebus/disasterrecoveryconfigs)。
- 執行地理災害復原在 [GitHub 上的範例](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/SBGeoDR2/SBGeoDR2) \(英文\)。
- 請參閱地理災害復原[範例以將訊息傳送至別名](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/TestGeoDR/ConsoleApp1) \(英文\)。

若要深入了解服務匯流排傳訊，請參閱下列文章：

* [服務匯流排基本概念](service-bus-fundamentals-hybrid-solutions.md)
* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)
* [Rest API](/rest/api/servicebus/) 

[1]: ./media/service-bus-geo-dr/geo1.png
[2]: ./media/service-bus-geo-dr/geo2.png
[3]: ./media/service-bus-geo-dr/az.png