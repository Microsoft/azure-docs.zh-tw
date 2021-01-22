---
title: Azure 服務匯流排地理災害復原 | Microsoft Docs
description: 如何使用地理區域，在 Azure 服務匯流排中進行容錯移轉並執行災害復原
ms.topic: article
ms.date: 01/04/2021
ms.openlocfilehash: b25fd1befded253c79267b1b016cef979005d01e
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98676450"
---
# <a name="azure-service-bus-geo-disaster-recovery"></a>Azure 服務匯流排地理災害復原

針對許多企業，以及在某些情況下，即使產業法規需要，仍有彈性地因應資料處理資源的災難中斷。 

Azure 服務匯流排已針對橫跨資料中心內多個失敗網域的叢集中的個別機器或甚至是整個機架的風險，將其分散到不同的風險，並且能實行透明的失敗偵測和容錯移轉機制，讓服務能夠繼續在保證的服務層級內運作，而且通常不會在發生這類失敗時明顯中斷。 如果已使用 [可用性區域](../availability-zones/az-overview.md)的啟用選項來建立服務匯流排命名空間，則風險是中斷風險會進一步分散到三個實體分隔的設備，而且服務會有足夠的容量保留，以立即處理整個設施的完整、重大損失。 

使用可用性區域支援的所有主動 Azure 服務匯流排叢集模型，都優於任何內部部署訊息代理程式產品的復原能力，以因應硬體故障和整個資料中心設施的災難遺失。 同樣地，可能會有以大範圍實體終結的抑音符號情況，甚至這些量值無法充分抵禦。 

「服務匯流排地理災難復原」功能的設計目的，是為了讓您更輕鬆地從這種程度的嚴重損壞中復原，並放棄失敗的 Azure 區域，而不需要變更您的應用程式設定。 放棄 Azure 區域通常會牽涉到數個服務，而這項功能主要是為了協助保留複合應用程式設定的完整性。 這項功能適用于服務匯流排 Premium SKU 的全球功能。 

Geo-Disaster 復原功能可確保將命名空間 (佇列、主題、訂用帳戶、篩選) 的完整設定，會在配對時持續從主要命名空間複寫至次要命名空間，而且可讓您在任何時間起始僅限一次的容錯移轉從主要命名空間移至次要命名空間。 容錯移轉移動會將命名空間的所選擇別名重新指向次要命名空間，然後中斷配對。 啟動時，容錯移轉幾乎是瞬間的。 

> [!IMPORTANT]
> 這項功能可讓您使用相同的設定來進行作業的即時持續性，但不 **會複寫保留在佇列或主題訂閱或寄不出信件佇列中的訊息**。 為了保留佇列語義，這類複寫不僅需要複寫訊息資料，而是在訊息代理程式中的每個狀態變更時。 針對大部分的服務匯流排命名空間，所需的複寫流量遠超過應用程式流量，以及高輸送量的佇列，大部分的訊息仍會複寫至次要資料庫，而這些訊息已從主要複本中刪除，而導致流量過高。 針對高延遲的複寫路由（適用于您選擇用於異地災難復原的多個配對），複寫流量可能也無法持續與應用程式流量，因為延遲引發的節流效果。
 
> [!TIP]
> 若要複寫佇列和主題訂用帳戶的內容，以及在主動/主動設定中操作對應的命名空間，以應付中斷和嚴重損壞狀況，請勿在此地理損毀修復功能集上取得，但請遵循複寫 [指引](service-bus-federation-overview.md)。  

## <a name="outages-and-disasters"></a>中斷與災害

請務必注意「中斷」和「災害」之間的差異。 

「中斷」是暫時無法使用 Azure 服務匯流排，而且會影響服務的某些元件，例如訊息存放區，或甚至整個資料中心。 不過，修正問題之後，服務匯流排就可再次使用。 中斷通常不會導致訊息或其他資料遺失。 這類中斷的範例可能是資料中心停電。 某些中斷只是因為暫時性或網路問題而造成的短暫連線中斷。 

「災害」定義為永久或較長期遺失服務匯流排叢集、Azure 區域或資料中心。 區域或資料中心不一定能再次使用，也可能會關閉數小時或數天。 這類災害的範例包括火災、水災或地震。 會變成永久的災害可能會導致某些訊息、事件或其他資料遺失。 不過，在大部分情況下，應該不會遺失資料，而且在備份資料中心之後，就可以復原訊息。

Azure 服務匯流排的地理災害復原功能就是一個災害復原解決方案。 本文中所述的概念和工作流程適用於災害案例，不適用暫時性或暫時中斷。 如需 Microsoft Azure 中災害復原的詳細討論，請參閱[本文](/azure/architecture/resiliency/disaster-recovery-azure-applications)。   

## <a name="basic-concepts-and-terms"></a>基本概念與術語

災害復原功能會實作中繼資料災害復原，並依賴主要和次要災害復原命名空間。 請注意，地理災害復原功能僅適用於[進階 SKU](service-bus-premium-messaging.md)。 您不需要進行任何連接字串變更，因為連接是透過別名建立的。

本文中使用下列術語：

-  *別名*：您所設定的災害復原設定的名稱。 別名提供單一穩定完整網域名稱 (FQDN) 連接字串。 應用程式會使用這個別名連接字串連接到命名空間。 使用別名可確保在觸發容錯移轉時，連接字串不會變更。

-  *主要/次要命名空間*：對應到別名的命名空間。 主要命名空間是「主動」並且會接收訊息 (這可以是現有或新的命名空間)。 次要命名空間是「被動」，並不會收到訊息。 這兩者間的中繼資料會進行同步處理，因此，這兩者均能順暢地接受訊息，而不需進行任何應用程式程式碼或連接字串變更。 若要確保只有主動命名空間會接收訊息，您必須使用別名。 

    > [!IMPORTANT]
    > 異地災難復原功能需要主要和次要命名空間的訂用帳戶和資源群組相同。
-  *中繼資料*：佇列、主題和訂用帳戶之類的實體，及其與命名空間關聯之服務的屬性。 請注意，只有實體及其設定會自動複寫。 不會複寫訊息。

-  *容錯移轉*：啟用次要命名空間的程序。

## <a name="setup"></a>安裝程式

下一節是設定命名空間之間配對的概觀。

![1][]

設定程序如下 -

1. 布建 ***主要** _ 服務匯流排 Premium 命名空間。

2. 在區域中布建 _*_次要_*_ 服務匯流排 Premium 命名空間，_different 從布建主要命名空間 *。 這將有助於允許隔離各個不同資料中心區域的錯誤。

3. 在主要命名空間和次要命名空間之間建立配對以取得 ***別名** _。

    >[!NOTE] 
    > 如果您已將 [Azure 服務匯流排標準命名空間遷移至 Azure 服務匯流排 Premium](service-bus-migrate-standard-premium.md)，則必須使用預先存在的別名 (也就是您的服務匯流排標準命名空間連接字串) 透過 _ *PS/CLI** 或 **REST API** 建立嚴重損壞修復設定。
    >
    >
    > 這是因為在遷移期間，Azure 服務匯流排標準命名空間連接字串/DNS 名稱本身會變成 Azure 服務匯流排進階命名空間的別名。
    >
    > 您的用戶端應用程式必須利用此別名 (亦即 Azure 服務匯流排標準命名空間連接字串)，來連線到已設定災害復原配對的進階命名空間。
    >
    > 如果您使用入口網站來設定災害復原組態，則入口網站將會從您那裡摘錄此警告。


4. 使用在步驟3中取得的 **_別名_* _，將用戶端應用程式連線至已啟用異地 DR 的主要命名空間。 一開始，此別名會指向主要命名空間。

5. [選擇性] 新增一些監視來偵測是否需要容錯移轉。

## <a name="failover-flow"></a>容錯移轉流程

容錯移轉會由客戶手動觸發 (明確地透過命令來觸發，或透過用戶端所擁有會觸發該命令的商務邏輯來觸發)，而一律不會由 Azure 觸發。 這會賦予客戶完整的擁有權和可見性，以解決 Azure 骨幹上發生的中斷情況。

![4][]

觸發容錯移轉之後 -

1. _*_別名_*_ 連接字串會更新以指向次要 Premium 命名空間。

2. 用戶端 (傳送者和接收者) 會自動連線到「次要」命名空間。

3. 「主要」與「次要」進階命名空間之間的現有配對已中斷。

一旦起始容錯移轉 -

1. 如果發生另一個中斷，您會希望能夠再次進行容錯移轉。 因此，請設定另一個被動命名空間，並更新配對。 

2. 一旦主要命名空間再次可供使用之後，從先前的主要命名空間提取訊息。 然後，針對地理復原設定以外的一般訊息使用該命名空間，或者刪除舊的主要命名空間。

> [!NOTE]
> 僅支援失敗轉接語意。 在此案例中，您容錯移轉然後與新的命名空間重新配對。 不支援容錯復原；例如，在 SQL 叢集。 

您可以使用監視系統或使用自訂監視解決方案來自動容錯移轉。 不過，這類自動化會採用額外的計劃和工作，這部分不在本文的範圍內。

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

3. 容錯移轉複雜分散式基礎結構應該至少[演練](/azure/architecture/reliability/disaster-recovery#disaster-recovery-plan)一次。

4. 同步處理實體可能需要一些時間，大約每分鐘 50-100 個實體。 訂用帳戶和規則也算是實體。

## <a name="availability-zones"></a>可用性區域

服務匯流排進階 SKU 也支援[可用性區域](../availability-zones/az-overview.md)，在 Azure 區域內提供錯誤隔離位置。

> [!NOTE]
> 「Azure 服務匯流排進階層」的「可用性區域」支援僅適用於有可用性區域存在的 [Azure 區域](../availability-zones/az-region.md)。

使用 Azure 入口網站時，只能在新的命名空間上啟用可用性區域。 服務匯流排不支援移轉現有的命名空間。 在命名空間上啟用區域備援之後，便無法停用。

![3][]

## <a name="private-endpoints"></a>私人端點
本節提供將異地災害復原與使用私人端點的命名空間搭配使用時的其他考慮。 若要了解通常如何使用私人端點搭配服務匯流排，請參閱[整合 Azure 服務匯流排與 Azure 私人連結](private-link-service.md)。

### <a name="new-pairings"></a>新配對
如果您嘗試在具有私人端點的主要命名空間與沒有私人端點的次要命名空間之間建立配對，配對將會失敗。 只有當主要和次要命名空間都具有私人端點時，配對才會成功。 建議您在主要和次要命名空間，以及在建立私人端點的所在虛擬網路上使用相同的設定。 

> [!NOTE]
> 當您嘗試將具有私人端點的主要命名空間和次要命名空間配對時，驗證程序只會檢查次要命名空間上是否存在私人端點。 不會檢查端點是否正常運作，或在容錯移轉之後是否可運作。 您必須負責確保具有私人端點的次要命名空間在容錯移轉之後能如預期般運作。
>
> 若要測試私人端點組態是否相同，請從虛擬網路外部將[取得佇列](/rest/api/servicebus/stable/queues/get)要求傳送至次要命名空間，並確認您收到來自服務的錯誤訊息。

### <a name="existing-pairings"></a>現有配對
如果主要和次要命名空間之間的配對已存在，在主要命名空間上的私人端點建立將會失敗。 若要解決此問題，請先在次要命名空間上建立私人端點，然後再為主要命名空間建立私人端點。

> [!NOTE]
> 我們允許對次要命名空間唯讀存取，也允許更新私人端點設定。 

### <a name="recommended-configuration"></a>建議組態
建立應用程式和服務匯流排的災害復原組態時，您必須針對裝載您應用程式主要和次要執行個體的虛擬網路，建立主要和次要服務匯流排命名空間的私人端點。

假設您有兩個虛擬網路：VNET-1、VNET 2，以及下列主要和次要命名空間：ServiceBus-Namespace1-Primary、ServiceBus-Namespace2-Secondary。 您需要執行下列步驟： 

- 在 ServiceBus-Namespace1-Primary 上，建立兩個私人端點，且這兩個端點使用 VNET-1 和 VNET-2 中的子網路
- 在 ServiceBus-Namespace2-Secondary 上，建立兩個私人端點，且這兩個端點使用 VNET-1 和 VNET-2 中的相同子網路 

![私人端點和虛擬網路](./media/service-bus-geo-dr/private-endpoints-virtual-networks.png)


這種方法的優點是，容錯移轉可能會發生在與服務匯流排命名空間無關的應用程式層。 請考慮下列案例： 

_ *僅限應用程式的容錯移轉：** 這裡，應用程式不會存在於 vnet 1 中，但會移至 vnet-2。 由於在主要和次要命名空間的 VNET-1 和 VNET-2 上都設定了這兩個私人端點，應用程式會正常執行。 

**僅服務匯流排命名空間的容錯移轉**：同樣的，此案例中由於在主要和次要命名空間的虛擬網路上都設定了這兩個私人端點，應用程式會正常執行。 

> [!NOTE]
> 如需虛擬網路的異地災害復原指引，請參閱[虛擬網路 - 商務持續性](../virtual-network/virtual-network-disaster-recovery-guidance.md)。

## <a name="next-steps"></a>後續步驟

- 請參閱地理災害復原在[這裡的 REST API 參考](/rest/api/servicebus/stable/disasterrecoveryconfigs)。
- 執行地理災害復原在 [GitHub 上的範例](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/SBGeoDR2/SBGeoDR2) \(英文\)。
- 請參閱地理災害復原[範例以將訊息傳送至別名](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/TestGeoDR/ConsoleApp1) \(英文\)。

若要深入了解服務匯流排傳訊，請參閱下列文章：

* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)
* [Rest API](/rest/api/servicebus/) 

[1]: ./media/service-bus-geo-dr/geodr_setup_pairing.png
[2]: ./media/service-bus-geo-dr/geo2.png
[3]: ./media/service-bus-geo-dr/az.png
[4]: ./media/service-bus-geo-dr/geodr_failover_alias_update.png
