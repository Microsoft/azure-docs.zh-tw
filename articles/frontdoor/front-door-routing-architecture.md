---
title: Azure Front Door Service - 路由架構 | Microsoft Docs
description: 此文章可協助您了解 Front Door 架構的全域檢視觀點。
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
ms.openlocfilehash: 42a9c4f9983a68459351fc98fb8b757759e585bf
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46964874"
---
# <a name="routing-architecture-overview"></a>路由架構概觀

Azure Front Door Service 在接收到您的客戶要求後便會回應它們 (若快取已啟用)，或是將它們轉送到適當的應用程式後端 (以反向 Proxy 的形式)。

</br>在路由傳送到 Azure Front Door 與後端的期間，有著許多對流量進行最佳化的機會。

## <a name = "anycast"></a>選取適用於流量路由傳送的 Front Door 環境 (任一傳播)

路由傳送到 Azure Front Door 環境會針對 DNS (網域名稱系統) 與 HTTP (超文字傳輸通訊協定) 運用[任一傳播](https://en.wikipedia.org/wiki/Anycast)，因此使用者流量將會移至根據網路拓撲最接近 (最少躍點) 的環境。 此架構通常能為使用者提供較佳的來回行程時間 (將分割 TCP 的優點最大化)。 Front Door 會將其環境分類為主要及後援「圈」。  外圈具有較接近使用者的環境，並能提供較低的延遲。  內圈則具有能在發生問題時處理外圈環境之容錯移轉的環境。 對於所有流量而言，外圈都是偏好的目標，但仍需要內圈來處理來自外圈的流量溢位。 根據 VIP (虛擬網際網路通訊協定位址)，每部前端主機或由 Front Door 所提供的網域都會被指派主要 VIP (同時由內圈與外圈中的環境宣佈)，以及後援 VIP (僅由內圈中的環境宣佈)。 

</br>這個整體策略能確保來自使用者的要求一律會抵達最接近的 Front Door 環境，而在偏好的 Front Door 環境健康情況不良的情況下，流量將會自動移至下一個最接近的環境。

## <a name = "splittcp"></a>連線到 Front Door 環境 (分割 TCP)

[分割 TCP](https://en.wikipedia.org/wiki/Performance-enhancing_proxy) \(英文\) 是一個可降低延遲與 TCP 問題的技術，能將會產生較高來回時間的連線分割成較小的片段。  透過使 Front Door 環境更接近使用者，並終止 Front Door 環境中的 TCP 連線，針對應用程式後端具有較高來回時間 (RTT) 的單一 TCP 連線，將會被分割成兩個 TCP 連線。 透過縮短使用者與 Front Door 環境之間的連線，可使連線透過三個較短 (而非三個較長) 的來回行程便能完成，進而降低延遲。  您可以預先建立 Front Door 環境與後端之間的長連線，並將其重複用於多個使用者呼叫上，進而節省 TCP 連線時間。  在建立 SSL/TLS (傳輸層安全性) 連線的情況下，此效果將會倍增，因為會進行更多來回行程以保護連線。

## <a name="processing-request-to-match-a-routing-rule"></a>處理要求以比對路由規則
在建立連線並進行 SSL 交握之後，當要求登陸至 Front Door 環境時，第一個步驟便是比對路由規則。 基本上，此比對是要從 Front Door 的所有設定中，決定要將哪個特定的路由規則與該要求比對。 若要深入了解，請參閱 Front Door 執行[路由比對](front-door-route-matching.md)的方式。

## <a name="identifying-available-backends-in-the-backend-pool-for-the-routing-rule"></a>針對路由規則識別後端集區中的可用後端
當 Front Door 根據傳入要求比對路由規則之後，且沒有快取的情況下，下一步便是提取和已比對路由相關聯之後端集區的健康情況探查狀態。 若要深入了解，請參閱 Front Door 使用[健康情況探查](front-door-health-probes.md)來監視後端健康情況的方式。

## <a name="forwarding-the-request-to-your-application-backend"></a>將要求轉送到應用程式後端
最後，假設沒有設定快取，系統會根據您的 [Front Door 路由傳送方法](front-door-routing-methods.md)設定，將使用者要求轉送到「最佳」後端。

## <a name="next-steps"></a>後續步驟

- 了解如何[建立 Front Door](quickstart-create-front-door.md)。
