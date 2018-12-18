---
title: Azure 轉送常見問題集 | Microsoft Docs
description: 獲得一些有關 Azure 轉送常見問題的答案。
services: service-bus-relay
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: 886d2c7f-838f-4938-bd23-466662fb1c8e
ms.service: service-bus-relay
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/21/2018
ms.author: spelluru
ms.openlocfilehash: e30e8c94547ac0f9106a69f1e99cf9a7c03abea5
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43695892"
---
# <a name="azure-relay-faqs"></a>Azure 轉送常見問題集

本文提供 [Azure 轉送](https://azure.microsoft.com/services/service-bus/)的一些常見問題集 (FAQ) 答案。 如需一般的 Azure 定價和支援資訊，請參閱 [Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。

## <a name="general-questions"></a>一般問題
### <a name="what-is-azure-relay"></a>什麼是 Azure 轉送？
[Azure 轉送服務](relay-what-is-it.md)有助於您的混合式應用程式，方法是協助您更安全地將位於公司企業網路內的服務公開至公用雲端。 您可以在不開啟防火牆連線的情況下公開服務，也不需要對公司網路基礎結構做侵入式變更。

### <a name="what-is-a-relay-namespace"></a>什麼是轉送命名空間？
[命名空間](relay-create-namespace-portal.md)是範圍容器，您可以用來在應用程式內定址轉送資源。 您必須建立命名空間，才能使用轉送。 這是開始使用的第一個步驟其中之一。

### <a name="what-happened-to-service-bus-relay-service"></a>服務匯流排轉送服務發生什麼事？
先前稱為「服務匯流排轉送」的服務現在稱為 [WCF 轉送](relay-wcf-dotnet-get-started.md)。 您可以繼續如常使用這項服務。 混合式連線功能是服務的更新版本，從 Azure BizTalk 服務移植。 會繼續支援「WCF 轉送」和「混合式連線」。

## <a name="pricing"></a>價格
本節提供轉送價格結構的一些常見問題解答。 如需一般的 Azure 定價資訊，也可以參閱 [Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。 如需轉送價格的完整資訊，請參閱[服務匯流排價格詳細資料][Pricing overview]。

### <a name="how-do-you-charge-for-hybrid-connections-and-wcf-relay"></a>混合式連接和 WCF 轉送如何收費？
如需有關轉送價格的完整資訊，請參閱服務匯流排價格詳細資料頁面上的[混合式連線與 WCF 轉送][Pricing overview]資料表。 除了頁面上註明的價格，您還需支付您的應用程式佈建所在資料中心外部的輸出相關資料傳輸費用。

### <a name="how-am-i-billed-for-hybrid-connections"></a>混合式連接如何計費？
以下是混合式連線計費案例的三個範例︰

*   案例 1：
    *   您有單一接聽程式，例如已安裝且整個月份持續執行的混合式連線管理員執行個體。
    *   您在該月份的連線中傳送 3 GB 的資料。 
    *   您的總費用為 $5。
*   案例 2：
    *   您有單一接聽程式，例如已安裝且整個月份持續執行的混合式連線管理員執行個體。
    *   您在該月份的連線中傳送 10 GB 的資料。
    *   您的總費用為 $7.5。 其中 $5 是連線和前面 5 GB 的費用，$2.50 是其他 5 GB 資料的費用。
*   案例 3：
    *   您有兩個執行個體，A 和 B，屬於已安裝且整個月份持續執行的混合式連線管理員。
    *   您在該月份的連線 A 中傳送 3 GB 的資料。
    *   您在該月份的連線 B 中傳送 6 GB 的資料。
    *   您的總費用為 $10.5。 其中 $5 是連線 A 的費用、$5 是連線 B 的費用，加上 $0.50 (連線 B 上的第 6 個 GB)。

請注意，範例中使用的價格僅適用於混合式連線預覽期間。 價格可能會在混合式連線正式運作之後有所變更。

### <a name="how-are-hours-calculated-for-relay"></a>如何計算轉送時數？

只能在標準層命名空間中使用 WCF 轉送。 否則轉送的價格和[連線配額](../service-bus-messaging/service-bus-quotas.md)會保持不變。 這表示轉送將繼續根據訊息數目 (而非作業數) 和轉送時數計費。 如需詳細資訊，請參閱價格詳細資料頁面上的[混合式連線與 WCF 轉送](https://azure.microsoft.com/pricing/details/service-bus/)資料表。

### <a name="what-if-i-have-more-than-one-listener-connected-to-a-specific-relay"></a>如果我有一個以上的接聽程式連線到指定的轉送，該怎麼辦？
在某些情況下，單一轉送可能有多個已連線的接聽程式。 至少有一個轉送接聽程式連線到轉送時，即會被視為開放式轉送。 將接聽程式新增至開放式轉送會導致額外的轉送時數。 連線到轉送的轉送傳送者 (叫用或傳送訊息至轉送的用戶端) 數目不會影響轉送時數的計算。

### <a name="how-is-the-messages-meter-calculated-for-wcf-relays"></a>如何針對 WCF 轉送計算訊息計量？
(**這只適用於 WCF 轉送。訊息並不是混合式連線的成本。**)

一般而言，對代理實體 (佇列、主題和訂用帳戶) 使用先前所述相同方法的轉送會計算計費訊息。 但是，請注意以下幾個差異。

將訊息傳送至服務匯流排轉送會被視為「完整通過」傳送至接收訊息的轉送接聽程式。 它不會被視為送至服務匯流排轉送的傳送作業，其後面接著傳遞至轉送接聽程式。 對於轉送接聽程式的要求-回覆模式服務叫用 (最多 64 KB) 將會產生兩則計費訊息︰一則是要求的計費訊息，一則是回應的計費訊息 (假設回應也是 64 KB 或更小)。 這與使用佇列在用戶端與服務之間居中協調不同。 如果您使用佇列在用戶端與服務之間居中協調，相同要求-回覆模式需要將要求傳送至佇列，後面接著將來自佇列的清除佇列/傳遞傳送至服務。 後面接著將回應傳送至另一個佇列，以及將來自該佇列的清除佇列/傳遞傳送至用戶端。 使用相同的大小假設輸送量 (最多 64 KB)，居中協調的佇列模式會產生 4 則計費訊息。 您要支付兩倍的訊息數目，才能實作使用轉送完成的相同模式。 當然，使用佇列來達成此模式有許多優點，例如，持久性和負載調節。 這些優點可合理解釋額外的費用。

使用 **netTCPRelay** WCF 繫結開啟的轉送不會將訊息視為個別的訊息，但是會視為通過系統的資料流。 當您使用此繫結時，只有傳送者和接聽程式能夠看見傳送和接收之個別訊息的框架。 對於使用 **netTCPRelay** 繫結的轉送，所有資料都會被視為資料流，以便計算計費訊息。 在此情況下，服務匯流排會以 5 分鐘為基礎，計算透過每個個別轉送傳送或接收的資料總量。 然後，它會依據 64 KB 來分割資料總量，以判斷該期間該轉送的計費訊息數目。

## <a name="quotas"></a>配額
| 配額名稱 | 影響範圍 |  注意 | 值 |
| --- | --- | --- | --- |
| 轉送上的並行接聽程式 |實體 |後續對更多連線的要求將會遭到拒絕，而且呼叫端程式碼將會收到例外狀況。 |25 |
| 服務命名空間中所有轉送端點的並行轉送連線 |命名空間 |- |5,000 |
| 每個服務命名空間的轉送端點 |命名空間 |- |10,000 |
| [NetOnewayRelayBinding](/dotnet/api/microsoft.servicebus.netonewayrelaybinding) 和 [NetEventRelayBinding](/dotnet/api/microsoft.servicebus.neteventrelaybinding) 轉送的訊息大小 |命名空間 |超出這些配額的內送訊息將會遭到拒絕，而且呼叫端程式碼將會收到例外狀況。 |64 KB |
| [HttpRelayTransportBindingElement](/dotnet/api/microsoft.servicebus.httprelaytransportbindingelement) 和 [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding) 轉送的訊息大小 |命名空間 |訊息大小沒有限制。 |無限 |

### <a name="does-relay-have-any-usage-quotas"></a>轉送是否有任何使用量配額？
根據預設，對於所有雲端服務，Microsoft 會設定針對所有客戶的訂用帳戶計算的彙總每月使用量配額。 我們了解有時候您的需求可能會超過這些限制。 您可以隨時連絡客戶服務部門，讓我們知道您的需求並適當地調整這些限制。 服務匯流排的彙總使用量配額如下：

* 50 億則訊息
* 2 百萬個轉送小時

雖然我們有權停用超出其每月使用量配額的帳戶，但我們將提供電子郵件通知並且在採取任何動作之前多次嘗試連絡客戶。 超出這些配額的客戶仍需負責支付額外費用。

### <a name="naming-restrictions"></a>命名限制
轉送命名空間名稱的長度必須介於 6 到 50 個字元之間。

## <a name="subscription-and-namespace-management"></a>訂用帳戶和命名空間管理
### <a name="how-do-i-migrate-a-namespace-to-another-azure-subscription"></a>如何將命名空間移轉到另一個 Azure 訂用帳戶？

若要將命名空間從某個 Azure 訂用帳戶移至另一個訂用帳戶，您可以使用 [Azure 入口網站](https://portal.azure.com)或使用 PowerShell 命令。 若要將命名空間移至另一個訂用帳戶，命名空間必須已經是作用中。 執行命令的使用者必須是來源和目標訂用帳戶的系統管理員使用者。

#### <a name="azure-portal"></a>Azure 入口網站

若要使用 Azure 入口網站，將 Azure 轉送命名空間從某個訂用帳戶移轉到另一個訂用帳戶，請參閱[將資源移至新的資源群組或訂用帳戶](../azure-resource-manager/resource-group-move-resources.md#use-portal)。 

#### <a name="powershell"></a>PowerShell

若要使用 PowerShell 將命名空間從某個 Azure 訂用帳戶移至另一個訂用帳戶，請使用下列的命令順序。 若要執行這項作業，命名空間必須已經是作用中，而且執行 PowerShell 命令的使用者必須是來源與目標訂用帳戶的系統管理員使用者。

```azurepowershell-interactive
# Create a new resource group in the target subscription.
Select-AzureRmSubscription -SubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff'
New-AzureRmResourceGroup -Name 'targetRG' -Location 'East US'

# Move the namespace from the source subscription to the target subscription.
Select-AzureRmSubscription -SubscriptionId 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa'
$res = Find-AzureRmResource -ResourceNameContains mynamespace -ResourceType 'Microsoft.ServiceBus/namespaces'
Move-AzureRmResource -DestinationResourceGroupName 'targetRG' -DestinationSubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff' -ResourceId $res.ResourceId
```

## <a name="troubleshooting"></a>疑難排解
### <a name="what-are-some-of-the-exceptions-generated-by-azure-relay-apis-and-suggested-actions-you-can-take"></a>Azure 轉送 API 所產生的例外狀況有哪些，您可以採取的建議動作為何？
如需常見例外狀況的描述以及您可以採取的建議動作，請參閱[轉送例外狀況][Relay exceptions]。

### <a name="what-is-a-shared-access-signature-and-which-languages-can-i-use-to-generate-a-signature"></a>什麼是共用存取簽章，我可以使用何種語言來產生簽章？
共用存取簽章 (SAS) 是以 SHA-256 安全雜湊或 URI 為基礎的驗證機制。 如需如何以 Node、PHP、Java、C 和 C# 產生自有簽章的相關資訊，請參閱[具有共用存取簽章的服務匯流排驗證][Shared Access Signatures]。

### <a name="is-it-possible-to-whitelist-relay-endpoints"></a>可以將轉送端點列入白名單嗎？
是。 轉送用戶端會使用完整的網域名稱連線至 Azure 轉送服務。 客戶可以在防火牆上新增 `*.servicebus.windows.net` 項目以支援 DNS 白名單。

## <a name="next-steps"></a>後續步驟
* [建立命名空間](relay-create-namespace-portal.md)
* [開始使用 .NET](relay-hybrid-connections-dotnet-get-started.md)
* [開始使用 Node](relay-hybrid-connections-node-get-started.md)

[Pricing overview]: https://azure.microsoft.com/pricing/details/service-bus/
[Relay exceptions]: relay-exceptions.md
[Shared Access Signatures]: ../service-bus-messaging/service-bus-sas.md
