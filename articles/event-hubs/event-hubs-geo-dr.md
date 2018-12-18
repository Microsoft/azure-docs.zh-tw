---
title: Azure 事件中樞地理災害復原 | Microsoft Docs
description: 如何使用地理區域，在 Azure 事件中樞中進行容錯移轉並執行災害復原
services: event-hubs
documentationcenter: ''
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/14/2018
ms.author: shvija
ms.openlocfilehash: 89ecfd93c22fe72fee3befd8d4a722eb41d816af
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42747166"
---
# <a name="azure-event-hubs-geo-disaster-recovery"></a>Azure 事件中樞地理災害復原

當整個 Azure 區域或資料中心 (如果未使用[可用性區域](../availability-zones/az-overview.md)) 遇到停機時，最重要的是資料處理作業能夠繼續在不同的區域或資料中心運作。 因此，「地理災害復原」和「異地複寫」對於任何企業而言都是重要的功能。 Azure 事件中樞支援命名空間層級的地理災害復原和異地複寫。 

地理災害復原功能全域適用於事件中樞標準 SKU。

## <a name="outages-and-disasters"></a>中斷與災害

請務必注意「中斷」和「災害」之間的差異。 「中斷」是暫時無法使用 Azure 事件中樞，而且會影響服務的某些元件，例如訊息存放區，或甚至整個資料中心。 不過，修正問題之後，事件中樞就可再次使用。 中斷通常不會導致訊息或其他資料遺失。 這類中斷的範例可能是資料中心停電。 某些中斷只是因為暫時性或網路問題而造成的短暫連線中斷。 

「災害」定義為永久或較長期遺失事件中樞叢集、Azure 區域或資料中心。 區域或資料中心不一定能再次使用，也可能會關閉數小時或數天。 這類災害的範例包括火災、水災或地震。 會變成永久的災害可能會導致某些訊息、事件或其他資料遺失。 不過，在大部分情況下，應該不會遺失資料，而且在備份資料中心之後，就可以復原訊息。

Azure 事件中樞的地理災害復原功能就是一個災害復原解決方案。 本文中所述的概念和工作流程適用於災害案例，不適用暫時性或暫時中斷。 如需 Microsoft Azure 中災害復原的詳細討論，請參閱[本文](/azure/architecture/resiliency/disaster-recovery-azure-applications)。

## <a name="basic-concepts-and-terms"></a>基本概念與術語

災害復原功能會實作中繼資料災害復原，並依賴主要和次要災害復原命名空間。 請注意，地理災害復原功能僅適用於[標準 SKU](https://azure.microsoft.com/pricing/details/event-hubs/)。 您不需要進行任何連接字串變更，因為連接是透過別名建立的。

本文中使用下列術語：

-  別名：所設定之災害復原設定的名稱。 別名提供單一穩定完整網域名稱 (FQDN) 連接字串。 應用程式會使用這個別名連接字串連接到命名空間。 

-  主要/次要命名空間：對應到別名的命名空間。 主要命名空間是「主動」並且會接收訊息 (這可以是現有或新的命名空間)。 次要命名空間是「被動」，並不會收到訊息。 這兩者間的中繼資料會進行同步處理，因此，這兩者均能順暢地接受訊息，而不需進行任何應用程式程式碼或連接字串變更。 若要確保只有主動命名空間會接收訊息，您必須使用別名。 

-  中繼資料：實體 (例如事件中樞和取用者群組)；以及與命名空間相關聯之服務的屬性。 請注意，只有實體及其設定會自動複寫。 不會複寫訊息和事件。 

-  容錯移轉：啟用次要命名空間的程序。

## <a name="setup-and-failover-flow"></a>設定和容錯移轉流程

下一節是容錯移轉程序的概觀，並說明如何設定初始容錯移轉。 

![1][]

### <a name="setup"></a>設定

首先建立或使用現有的主要命名空間，以及新的次要命名空間，然後將這兩個命名空間配對。 此配對會為您提供可用來連接的別名。 由於您使用別名，因此不需變更連接字串。 只需將新的命名空間新增至您的容錯移轉配對。 最後，您應該新增某些監視來偵測是否需要容錯移轉。 在大部分情況下，服務是大型生態系統的一部分，因此不太可能自動容錯移轉，容錯移轉通常都必須與其餘子系統或基礎結構同步執行。

### <a name="example"></a>範例

在此案例的一個範例中，請考慮銷售點 (POS) 解決方案，它會發出訊息或事件。 事件中樞會將這些事件傳遞至某些對應或重新格式化解決方案，接下來將對應資料轉接到另一個系統，以進一步處理。 屆時，這些系統都會裝載於相同的 Azure 區域。 關於何時容錯移轉及容錯移轉哪個部分的決策，取決於基礎結構中的資料流量。 

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

## <a name="samples"></a>範例

[GitHub 上的範例](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/GeoDRClient)示範如何設定及初始化容錯移轉。 此範例示範下列概念︰

- 在 Azure Active Directory 中使用 Azure Resource Manager 與事件中樞所需的設定。 
- 執行範例程式碼所需的步驟。 
- 從目前的主要命名空間傳送和接收。 

## <a name="considerations"></a>考量

請注意此版本的下列考量：

1. 在您的容錯移轉規劃中，您也應該考慮時間因素。 例如，如果您中斷連線時間超過 15 到 20 分鐘，您可能會決定初始化容錯移轉。 
 
2. 未複寫任何資料這個事實表示未複寫目前作用中工作階段。 此外，重複的偵測和排程訊息可能無法運作。 新的工作階段、排程訊息及新的重複項目會運作。 

3. 容錯移轉複雜分散式基礎結構應該至少[演練](/azure/architecture/resiliency/disaster-recovery-azure-applications#disaster-simulation)一次。 

4. 同步處理實體可能需要一些時間，大約每分鐘 50-100 個實體。

## <a name="availability-zones-preview"></a>可用性區域 (預覽)

事件中樞標準 SKU 也支援[可用性區域](../availability-zones/az-overview.md)，在 Azure 區域內提供錯誤隔離位置。 

> [!NOTE]
> 只有在**美國中部**、**美國東部 2** 和**法國中部**區域才支援可用性區域預覽。

使用 Azure 入口網站時，只能在新的命名空間上啟用可用性區域。 事件中樞不支援移轉現有的命名空間。 在命名空間上啟用區域備援之後，便無法停用。

![3][]

## <a name="next-steps"></a>後續步驟

* [GitHub 上的範例](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/GeoDRClient)會逐步解說簡單工作流程，以建立地理配對並在災害復原情況下起始容錯移轉。
* [REST API 參考](/rest/api/eventhub/disasterrecoveryconfigs)描述用來執行地理災害復原設定的 API。

如需事件中樞的詳細資訊，請造訪下列連結：

* 開始使用[事件中樞教學課程](event-hubs-dotnet-standard-getstarted-send.md)
* [事件中樞常見問題集](event-hubs-faq.md)
* [使用事件中樞的完整範例應用程式](https://github.com/Azure/azure-event-hubs/tree/master/samples)

[1]: ./media/event-hubs-geo-dr/geo1.png
[2]: ./media/event-hubs-geo-dr/geo2.png
[3]: ./media/event-hubs-geo-dr/eh-az.png