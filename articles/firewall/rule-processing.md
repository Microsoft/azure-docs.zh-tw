---
title: Azure 防火牆規則處理邏輯
description: 了解 Azure 防火牆規則處理邏輯
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 9/27/2018
ms.author: victorh
ms.openlocfilehash: 542682037a932a2e3b08c71b38b64b2694ad40f3
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47228063"
---
# <a name="azure-firewall-rule-processing-logic"></a>Azure 防火牆規則處理邏輯
Azure 防火牆具有 NAT 規則、網路規則和應用程式規則。 規則的處理依據是規則類型。


## <a name="network-rules-and-applications-rules"></a>網路規則和應用程式規則 
首先會套用網路規則，然後套用應用程式規則。 之後規則就會終止。 因此，如果在網路規則中找到相符項目，就不會處理應用程式規則。  如果沒有符合的網路規則，而且如果封包通訊協定是 HTTP/HTTPS，則封包會由應用程式規則評估。 如果仍然找不到符合的規則，那麼封包會根據基礎結構規則集合進行評估。 如果仍然沒有相符項目，則封包預設會遭到拒絕。

## <a name="nat-rules"></a>NAT 規則
若要啟用輸入連線，請設定目的地網路位址轉譯 (DNAT)，如[教學課程：使用 Azure 入口網站以 Azure 防火牆 DNAT 篩選輸入流量](tutorial-firewall-dnat.md)所述。 DNAT 規則會先套用。 如果找到相符項目，就會新增隱含的對應網路規則，以允許已轉譯的流量。 若要覆寫這個行為，您可以明確地使用符合已轉譯流量的拒絕規則來新增網路規則集合。 這些連線不會套用任何應用程式規則。


## <a name="next-steps"></a>後續步驟

- 深入了解如何[部署和設定 Azure 防火牆](tutorial-firewall-deploy-portal.md)。