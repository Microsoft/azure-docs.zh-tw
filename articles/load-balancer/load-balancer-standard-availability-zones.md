---
title: Azure 標準 Load Balancer 和可用性區域 | Microsoft Docs
description: 標準 Load Balancer 和可用性區域
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/29/2018
ms.author: kumud
ms.openlocfilehash: f5d46fda6bdb32c1a5000883c6aedb2da15e796a
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
ms.locfileid: "30322789"
---
# <a name="standard-load-balancer-and-availability-zones"></a>標準 Load Balancer 和可用性區域

Azure Load Balancer 的標準 SKU 支援[可用性區域](../availability-zones/az-overview.md)案例。 標準 Load Balancer 有數個新的概念，可讓您藉由對應資源與區域並將其分散到區域，將您端對端案例中的可用性最佳化。  請檢閱[可用性區域](../availability-zones/az-overview.md)的指引，以了解可用性區域是什麼，目前有哪些區域支援可用性區域，以及其他相關概念與產品。 可用性區域與標準 Load Balancer 可組合成可擴充且具有彈性的功能集，用以建立許多不同的案例。  請檢閱本文以了解這些[概念](#concepts)，以及基本案例的[設計指引](#design)。

>[!NOTE]
>如需其他相關主題，請檢閱[可用性區域](https://aka.ms/availabilityzones)。 

## <a name="concepts"></a>適用於 Load Balancer 的可用性區域概念

Load Balancer 資源與實際的基礎結構之間沒有直接關聯性；建立 Load Balancer 並不會建立執行個體。 Load Balancer 資源是一種物件，而您可以在其中表示 Azure 應如何對其預先建置的多租用戶基礎結構進行設計程式，以達到您想要建立的案例。  這在可用性區域的內容中是十分重要的，因為單一 Load Balancer 資源可控制多個可用性區域中的基礎結構程式設計，而從客戶觀點來看，區域備援服務會呈現為一個資源。

Load Balancer 資源的運作方式可用前端、規則、健康情況探查和後端集區定義來表示。

在可用性區域的內容中，Load Balancer 資源的行為和屬性可解釋為區域備援或區域性。  區域備援和區域性說明屬性的區域性。  在 Load Balancer 的內容中，區域備援一律表示*所有區域*，而區域性則表示確保服務屬於*單一區域*。

公用和內部的 Load Balancer 均支援區域備援和區域性案例，且兩者都可視需要在區域之間進行流量導向 (*跨區域負載平衡*)。

Load Balancer 資源本身是區域的，且絕不具區域性。  VNet 和子網路則一律是區域的，且絕不具區域性。

### <a name="frontend"></a>前端

Load Balancer 前端是對虛擬網路資源之子網路內的公用 IP 位址資源或私人 IP 位址進行參考的前端 IP 組態。  它會構成服務公開所在的負載平衡端點。

Load Balancer 資源可以同時包含區域性和區域備援前端。 

保證公用 IP 資源屬於某區域時，區域性 (或無區域性) 是不可變動的。  如果您想要變更或省略公用 IP 前端的區域性，您必須在適當的區域中重新建立公用 IP。  

您可以藉由移除並重新建立前端、變更或省略區域性，來變更內部 Load Balancer 前端的區域性。

使用多個前端時，請檢閱[多個 Load Balancer 前端](load-balancer-multivip-overview.md)，以考量重要事項。

#### <a name="zone-redundant-by-default"></a>預設具有區域備援功能

在具有可用性區域的區域中，標準 Load Balancer 前端依預設具有區域備援功能。  單一前端 IP 位址在區域失敗時仍可存留，可用來連線至所有的後端集區成員，而不考量區域。 這不表示資料路徑不會中斷，但任何重試或重新建立作業都會成功。 您不需要進行 DNS 備援配置。 每個可用性區域中獨立的基礎結構部署會同時為前端的單一 IP 位址提供服務。  區域備援表示區域中的所有可用性區域會同時使用單一 IP 位址為所有的輸入或輸出流量提供服務。

雖然有一或多個可用性區域可能會失敗，但區域中只要有一個區域維持良好狀況，資料路徑就得以存留。 區域備援組態是預設值，且不需要其他動作。  當一個區域有能力支援可用性區域時，現有的前端就會自動具有區域備援功能。

請使用下列指令碼，為您的內部標準 Load Balancer 建立區域備援公用 IP 位址。 如果您在組態中使用現有的 Resource Manager 範本，請將 **sku** 區段新增到這些範本。

```json
            "apiVersion": "2017-08-01",
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "public_ip_standard",
            "location": "region",
            "sku":
            {
                "name": "Standard"
            },
```

請使用下列指令碼，為您的內部標準 Load Balancer 建立區域備援前端 IP 位址。 如果您在組態中使用現有的 Resource Manager 範本，請將 **sku** 區段新增到這些範本。

```json
            "apiVersion": "2017-08-01",
            "type": "Microsoft.Network/loadBalancers",
            "name": "load_balancer_standard",
            "location": "region",
            "sku":
            {
                "name": "Standard"
            },
            "properties": {
                "frontendIPConfigurations": [
                    {
                        "name": "zone_redundant_frontend",
                        "properties": {
                            "subnet": {
                                "Id": "[variables('subnetRef')]"
                            },
                            "privateIPAddress": "10.0.0.6",
                            "privateIPAllocationMethod": "Static"
                        }
                    },
                ],
```

#### <a name="optional-zone-guarantee"></a>選用的區域保證

您可以選擇讓某個前端保證屬於單一區域，我們稱之為*區域性前端*。  這表示任何輸入或輸出流量都會由區域中的單一區域提供服務。  您的前端會具有與區域相同的健康情況。  非保證區域中的失敗皆不會對資料路徑造成影響。 您可以使用區域性前端來公開每個可用性區域的 IP 位址。  此外，您可以直接取用區域性前端，或是在前端包含公用 IP 位址時將其與 DNS 負載平衡產品 (如 [Traffic Manager](../traffic-manager/traffic-manager-overview.md)) 整合，並使用單一 DNS 名稱，讓用戶端解析為多個區域性 IP 位址。  您也可以使用此前端公開每個區域的負載平衡端點，以個別監視每個區域。  如果您想要綜合這些概念 (相同後端的區域備援和區域性)，請檢閱 [Azure Load Balancer 的多個前端](/load-balancer-multivip-overview.md)。

對於公用 Load Balancer 前端，您可以將*區域*參數新增至前端 IP 組態所參考的公用 IP。  

對於內部 Load Balancer 前端，請將*區域*參數新增至內部 Load Balancer 前端 IP 組態。 區域性前端會使 Load Balancer 保證子網路中的某個 IP 位址屬於特定區域。

請使用下列指令碼，在「可用性區域 1」中建立標準公用 IP 位址。 如果您在組態中使用現有的 Resource Manager 範本，請將 **sku** 區段新增到這些範本。

```json
            "apiVersion": "2017-08-01",
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "public_ip_standard",
            "location": "region",
            "zones": [ "1" ],
            "sku":
            {
                "name": "Standard"
            },
```

請使用下列指令碼，在「可用性區域 1」中建立內部標準 Load Balancer 前端。

如果您在組態中使用現有的 Resource Manager 範本，請將 **sku** 區段新增到這些範本。 此外，也請定義子資源之前端 IP 組態中的 **zones** 屬性。

```json
            "apiVersion": "2017-08-01",
            "type": "Microsoft.Network/loadBalancers",
            "name": "load_balancer_standard",
            "location": "region",
            "sku":
            {
                "name": "Standard"
            },
            "properties": {
                "frontendIPConfigurations": [
                    {
                        "name": "zonal_frontend_in_az1",
                        "zones": [ "1" ],
                        "properties": {
                            "subnet": {
                                "Id": "[variables('subnetRef')]"
                            },
                            "privateIPAddress": "10.0.0.6",
                            "privateIPAllocationMethod": "Static"
                        }
                    },
                ],
```

### <a name="cross-zone-load-balancing"></a>跨區域負載平衡

跨區域負載平衡是 Load Balancer 可及於任何區域中的後端端點，並獨立於前端及其區域性以外的能力。

如果您想要讓部署對應單一區域並保證位於該區域內，請將區域性前端和區域性後端資源對應至相同區域。 不需要採取任何動作。

### <a name="backend"></a>後端

Load Balancer 與虛擬機器搭配運作。  單一 VNet 中的任何 VM 都可以是後端集區的一部分，無論該 VM 是否保證屬於某區域，或保證屬於哪個區域。

如果您想要讓前端和後端對應於並保證屬於單一區域，只要將相同區域內的 VM 放入各自的後端集區中即可。

如果您想要跨多個區域處理 VM，只要將多個區域中的 VM 放入相同的後端集區中即可。  使用虛擬機器擴展集時，您可以將一或多個虛擬機器擴展集放入相同的後端集區中。  而且，每個虛擬機器擴展集可位於單一或多個區域中。

### <a name="outbound-connections"></a>輸出連線

[輸出連線](load-balancer-outbound-connections.md)會由所有區域提供服務，且在虛擬機器與公用 Load Balancer 和區域備援前端相關聯時，將會透過可用性區域在某區域中自動獲得區域備援功能。  輸出連線 SNAT 連接埠配置在區域失敗時仍可存留。  

接著，如果 VM 與公用 Load Balancer 和區域性前端相關聯，則會保證輸出連線將由單一區域提供服務。  輸出連線會具有與個別區域相同的健康情況。

無論是否有區域，SNAT 連接埠預先配置和演算法皆相同。

### <a name="health-probes"></a>健康狀態探查

您現有的健康情況探查定義仍會與沒有可用性區域時相同。  但是，我們已在基礎結構層級擴充健康情況模型。 

使用區域備援前端時，Load Balancer 會擴充其內部健康情況模型，以個別探查從每個可用性區域存取 VM 的能力，並關閉區域間可能已失敗的路徑，而不需要客戶手動操作。  如果從一個區域的 Load Balancer 基礎結構到另一個區域中的 VM 的指定路徑無法使用，Load Balancer 可以偵測出來，並避免此失敗。 其他可存取此 VM 的區域將可繼續從其各自的前端為 VM 提供服務。  因此，在發生失敗事件期間，每個區域可以有些微不同的流量分配，同時保護您端對端服務的整體健康情況。

## <a name="design"></a>設計考量

Load Balancer 特意設計成在可用性區域的內容中具有彈性。 您可以選擇對應至區域，或選擇具有區域備援能力。  增加可用性可能會導致複雜性隨之提高，因此您的可用性設計必須顧及最佳效能。  以下我們將討論一些重要的設計考量。

### <a name="automatic-zone-redundancy"></a>自動區域備援

Load Balancer 可簡化以單一 IP 作為區域備援前端的作業。 區域備援 IP 位址可安全地為任何區域中的區域性資源提供服務，且在一或多個區域失敗時仍可存留，只要區域內有一個區域維持良好狀況即可。 相反地，區域性前端則是將服務縮減為單一區域，且個別的區域將會有共同的結果。

區域備援不代表無中斷的資料路徑或控制平面；它實際上是資料平面。 區域備援流量可以使用任何區域，且客戶的流量將會使用區域中所有狀況良好的區域。 如果區域失敗，在該時間點使用良好區域的流量並不會受到影響。  流量使用的若是發生區域失敗時的區域，則可能受到影響，但在 Azure 收斂區域失敗後，應用程式即可復原，且這些流量也可在重新傳輸或重新建立後，在區域內其餘狀態良好的區域中繼續執行。

### <a name="xzonedesign"></a>跨區域界限

請務必了解，無論何時，使用跨區域的端對端服務都不會產生來自單一區域的結果，而是可能來自多個區域的結果。  因此，您的端對端服務可能無法透過非區域性部署獲得任何可用性。

使用可用性區域時請避免導入非預期的跨區域相依性而得不到任何可用性。  如果您的應用程式包含多個元件，而您想要有發生區域失敗後的復原能力，您必須謹慎確保有足夠的重要元件能夠在發生區域失敗時存留。  例如，如果應用程式的單一重要元件僅存在於存留區域以外的一個區域中，就可能對整個應用程式造成影響。  此外，也請考慮區域還原以及應用程式的收斂方式。 接下來我們將回顧一些要點，藉以激發您在思考特定案例所應提出的問題。

- 如果您的應用程式有兩個元件，例如 IP 位址和具有受控磁碟的 VM，且兩者分別保證位於區域 1 和區域 2 中，這時如果區域 1 失敗，您的端對端服務時將無法存留。  除非您已完全了解您將建立潛在危害性失敗模式，否則請勿跨區域。

- 如果您的應用程式有兩個元件，例如 IP 位址和具有受控磁碟的 VM，且兩者分別保證位於區域備援和區域 1 中，這時，只要區域 1 未失敗，您的端對端服務在區域 2 和區域 3 失敗時仍將存留。  不過，如果您所關注的都只是前端的可存取性，您研判服務健康情況的能力就會削弱。  請考慮開發更大範圍的健康情況和容量模型。  您可以搭配運用區域備援和區域性概念來擴充洞察性與管理能力。

- 如果您的應用程式在三個區域中有兩個元件 (例如區域備援 Load Balancer 前端和跨區域虛擬機器擴展集)，在區域失敗期間，不受失敗影響之區域中的資源仍將可供使用，但您的端對端服務在容量方面可能會降低。 從基礎結構的觀點而言，您的部署在歷經一或多個區域失敗後仍可存留，而這就產生了下列問題：
  - 您是否了解應用程式如何判讀這類失敗和降低的容量？
  - 您的服務中是否需要有保護措施，以在必要時強制容錯移轉至某個區域組？
  - 您將如何監視、偵測並消弭這種案例？ 您可以使用標準 Load Balancer 診斷，以加強您對於端對端服務效能的監視。 請考量可用項目以及可能需要增強的層面，以勾勒完整藍圖。

- 區域有助於您了解及抑制失敗。  不過，就逾時、重試和輪詢演算法等概念而言，區域失敗與其他失敗並無不同。 即使 Azure Load Balancer 提供區域備援路徑，而且會嘗試在封包層級即時快速復原，但在失敗發生時，仍可能會執行重新傳輸或重新建立，因此請務必了解您的應用程式處理失敗的方式。 您的負載平衡配置也將繼續存留，但您必須進行下列規劃：
  - 當區域失敗時，您的端對端服務是否了解此狀況？且在狀態遺失時，您將如何復原？
  - 區域回復時，您的應用程式了解如何安全地收斂嗎？

### <a name="zonalityguidance"></a>區域備援與區域性的比較

區域備援可透過服務的單一 IP 位址提供無從驗證的區域，同時提供恢復功能選項。  複雜性也可因而降低。  區域備援也具有跨區域的行動性，並可以安全地用於任何區域中的資源。  此外，它是不具可用性區域的區域未來的模型，能夠限制區域取得可用性區域之後所需的變更。  區域備援 IP 位址或前端的設定語法將可在任何區域中成功執行，包括不具可用性區域的區域在內。

區域性可提供屬於某區域的明確保證，而具有與該區域相同的健康情況。 建立區域性 IP 位址或區域性 Load Balancer 前端的關聯性，可能是理想或合理的表徵，，特別是當您連結的資源是位於相同區域中的區域性 VM 時。  或者，您的應用程式可能需要明確認知資源所在的區域，而您想要明確研判個別區域中的可用性。  您可以選擇將端對端服務的多個區域性前端分散公開於各區域間 (也就是，每個區域各有多個區域性虛擬機器擴展集的區域性前端)。  如果您的區域性前端是公用 IP 位址，您可以透過 [Traffic Manager](../traffic-manager/traffic-manager-overview.md) 使用這幾個區域性前端來公開您的服務。  或者，您可以使用多個區域性前端透過第三方監視解決方案取得個別區域的健康情況和效能深入資訊，並使用區域備援前端公開整體的服務。 您僅應為區域性資源提供與相同區域對應的區域性前端，並避免將可能有害的跨區域案例用於區域性資源。  區域性資源僅存在於有可用性區域的區域中。

在未了解服務架構的情況下，並沒有通則可說明哪個選擇較為理想。

## <a name="limitations"></a>限制

- 雖然資料平面具有完整的區域備援能力 (除非已指定區域性保證)，但控制平面作業則沒有完整的區域備援能力。

## <a name="next-steps"></a>後續步驟
- 深入了解[可用性區域](../availability-zones/az-overview.md)
- 深入了解[標準 Load Balancer](load-balancer-standard-overview.md)
- 了解如何[使用具有區域前端的標準 Load Balancer 來進行區域內的 VM 負載平衡](load-balancer-standard-public-zonal-cli.md)
- 了解如何[使用具有區域備援前端的標準 Load Balancer 來進行跨區域的 VM 負載平衡](load-balancer-standard-public-zone-redundant-cli.md)
