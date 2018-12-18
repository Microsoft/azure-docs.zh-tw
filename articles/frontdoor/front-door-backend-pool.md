---
title: Azure Front Door Service - 後端和後端集區 | Microsoft Docs
description: 本文可協助您了解後端和後端集區在 Front Door 設定中為何。
services: front-door
documentationcenter: ''
author: sharad4u
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: sharadag
ms.openlocfilehash: 64d96d54b323d634703301e48cdaa28fa875fbbc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958793"
---
# <a name="backends-and-backend-pools-in-azure-front-door-service"></a>Azure Front Door Service 中的後端和後端集區
本文說明有關如何對應應用程式部署與 Front Door 的不同概念。 我們也會說明應用程式後端之 Front Door 設定中不同詞彙的意義。

## <a name="backend-pool"></a>後端集區
Front Door 中的後端集區指的是一組對等後端，可接收其應用程式的相同類型流量。 換句話說，它是您在世界各地的應用程式執行個體邏輯群組，可以接收相同的流量，而且可以回應預期行為。 這些後端通常會部署至不同的區域或相同的區域內。 此外，這些後端全部可以處於主動-主動部署模式，否則無法定義為主動/被動設定。

後端集區也會定義應該如何透過健康狀態探查評估不同後端的健康狀態，以及相應地定義後端之間應該如何進行負載平衡。

### <a name="health-probes"></a>健康狀態探查
Front Door 會將定期 HTTP/HTTPS 探查要求傳送至您的每個已設定後端，以判斷每個後端的鄰近性和健康狀態來負載平衡終端使用者要求。 後端集區的健康狀態探查設定定義如何輪詢您應用程式後端的健康狀態。 下列設定適用於設定和負載平衡：

1. **路徑**：將探查要求傳送至後端集區中所有後端的 URL 路徑。 例如，如果您的其中一個後端是 `contoso-westus.azurewebsites.net`，並且路徑設定為 `/probe/test.aspx`，則 Front Door 環境會在假設通訊協定設定為 HTTP 的情況下將健康狀態探查要求傳送至 http://contoso-westus.azurewebsites.net/probe/test.aspx。 
2. **通訊協定**：定義透過 HTTP 還是 HTTPS 通訊協定將健康狀態探查要求從 Front Door 傳送至後端。
3. **間隔 (秒)**：此欄位定義對後端提出健康狀態探查的頻率，即每個 Front Door 環境傳送探查的間隔。 注意：如果您要尋找更快速的容錯移轉，則請將此欄位設定為較低值。 不過，值越低，您後端將收到的健康狀態探查數量就越多。 若要了解 Front Door 對您的後端產生的探查數量，請查看範例。 假設，間隔設定為 30 秒，而且全域大約有 90 個 Front Door 環境或 POP。 因此，您的每個後端大約每秒會收到 3-5 個探查要求。

如需詳細資料，請參閱[健康狀態探查](front-door-health-probes.md)。

### <a name="load-balancing-settings"></a>負載平衡設定
後端集區的負載平衡設定定義如何評估健康狀態探查以決定後端的狀況為良好或不良，同時定義需要如何負載平衡後端集區中不同後端之間的流量。 下列設定適用於設定和負載平衡：

1. **樣本大小**：此屬性識別後端健康狀態評估所需考量的健康狀態探查樣本數目。
2. **成功樣本大小**：此屬性定義上面所述的「樣本大小」值，即需要多少樣本才能確認狀況良好後端呼叫成功。 
</br>例如，假設針對 Front Door，您已將健康狀態探查「間隔」設定為 30 秒、將「樣本大小」設定為 '5'，並將「成功樣本大小」設定為 '3'。 然後，此設定的意義在於每次評估後端的健康狀態探查時，我們將查看最後五個樣本，其跨最後 150 秒 (=5*30 秒)，除非其中有 3 個以上的探查成功將後端宣告為狀況不良。 假設只有兩個成功探查，因此我們會將後端標示為狀況不良。 下次我們執行評估時，如果我們發現最後五個探查中有三個成功，則會再次將後端標示為狀況良好。
3. **延遲敏感度 (額外延遲)**：延遲敏感度欄位定義要 Front Door 根據延遲測量將要求傳送至敏感度範圍內的後端，還是將要求轉送至最接近的後端。 若要深入了解，請參閱 Front Door 的[最小延遲路由方法](front-door-routing-methods.md#latency)。

## <a name="backend"></a>後端
後端等同於區域中應用程式的部署執行個體。 Front Door 同時支援 Azure 和非 Azure 後端，因此該區域不是只能使用 Azure 區域，也可以是內部部署資料中心或是某個其他雲端中的應用程式執行個體。

在 Front Doors 的內容中，後端指的是應用程式的主機名稱或公用 IP，以提供用戶端要求。 因此，不應該將後端與您的資料庫層或儲存層等混淆，而是應該當成您應用程式後端的公用端點予以檢視。

當您在 Front Door 的後端集區中新增後端時，必須填寫下列詳細資料：

1. **後端主機類型**：您想要新增的資源類型。 Front Door 支援從應用程式服務、雲端服務或儲存體自動探索您的應用程式後端。 如果您想要 Azure 或甚至非 Azure 後端中的不同資源，則請選取 [自訂主機]。 注意：在設定期間，API 不會驗證是否可以從 Front Door 環境存取後端，而是您需要確定 Front Door 可以連線您的後端。 
2. **訂用帳戶和後端主機名稱**：如果您尚未選取 [自訂主機] 作為後端主機類型，則需要縮小範圍，並從使用者介面中選擇適當的訂用帳戶和對應後端主機名稱來選取後端。
3. **後端主機標頭**：傳送至每個要求後端的主機標頭值。 如需詳細資料，請參閱[後端主機標頭](#hostheader)。
4. **優先順序**：如果您想要針對所有流量使用某個主要服務端點，則可以將優先順序指派給不同的後端，並提供備用方案，以防萬一發生主要或備份後端無法使用的情況。 深入了解[優先順序](front-door-routing-methods.md#priority)。
5. **權數**：當您想要平均還是根據權數係數將流量分散到一組後端時，可以將權數指派給不同的後端。 深入了解[權數](front-door-routing-methods.md#weighted)。


### <a name = "hostheader"></a>後端主機標頭

Front Door 轉送至後端的要求具有主機標頭欄位，後端會使用它來擷取目標資源。 此欄位的值通常來自後端 URI，並且具有主機和連接埠。 例如，針對 `www.contoso.com` 提出的要求將會有主機標頭 `www.contoso.com`。 如果您要使用 Azure 入口網站設定後端，則填入此欄位的預設值就是後端的主機名稱。 例如，如果您的後端是 `contoso-westus.azurewebsites.net`，則在 Azure 入口網站中，後端主機標頭的自動填入值會是 `contoso-westus.azurewebsites.net`。 
</br>不過，如果您使用 Resource Manager 範本或其他機制，而且未明確地設定此欄位，則 Front Door 會傳送連入主機名稱作為主機標頭的值。 例如，如果針對 `www.contoso.com` 提出要求，而且您的後端為後端主機標頭欄位空白的 `contoso-westus.azurewebsites.net`，則 Front Door 會將主機標頭設定為 `www.contoso.com`。

大部分的應用程式後端 (例如 Web Apps、Blob 儲存體和雲端服務) 都需要主機標頭符合後端的網域。 不過，路由傳送至您後端的前端主機將會有不同主機名稱，例如 www.contoso.azurefd.net。 如果您要設定的後端需要主機標頭符合後端主機名稱，則應該確定「後端主機標頭」也有後端的主機名稱。

#### <a name="configuring-the-backend-host-header-for-the-backend"></a>設定後端的後端主機標頭
可以在後端集區區段中設定後端的「後端主機標頭」欄位。

1. 開啟 Front Door 資源，然後按一下要設定後端的後端集區。

2. 如果您尚未新增任何後端，或編輯現有後端，則請新增後端。 「後端主機標頭」欄位可以設定為自訂值或空白，表示連入要求的主機名稱將會當成主機標頭值使用。



## <a name="next-steps"></a>後續步驟

- 了解如何[建立 Front Door](quickstart-create-front-door.md)。
- 了解 [Front Door 的運作方式](front-door-routing-architecture.md)。