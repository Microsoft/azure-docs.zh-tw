---
title: 負載平衡器閒置 TCP 重設 | Microsoft Docs
description: 具有閒置逾時雙向 TCP RST 封包的 Load Balancer
services: load-balancer
documentationcenter: na
author: KumudD
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: 46b152c5-6a27-4bfc-bea3-05de9ce06a57
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/08/2018
ms.author: kumud
ms.openlocfilehash: 9aa3811eb03d38a4c6ab8203512f3e6699098122
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48883630"
---
# <a name="load-balancer-with-tcp-reset-on-idle-public-preview"></a>具有閒置 TCP 重設的負載平衡器 (公開預覽)

您可以使用 [Standard Load Balancer](load-balancer-standard-overview.md)，以在具有每個可設定閒置逾時的雙向 TCP 重設 (TCP RST 封包) 時，建立更容易預測的應用程式行為。  Load Balancer 的預設行為是在達到流程的閒置逾時時，以無訊息模式卸除流程。

![負載平衡器 TCP 重設](media/load-balancer-tcp-reset/load-balancer-tcp-reset.png)

>[!NOTE] 
>具有閒置逾時 TCP 重設的 Load Balancer 功能目前為公開預覽版，而且只有有限的一組[區域](#regions)時才提供。 此預覽版是在沒有服務等級協定的情況下提供，不建議用於生產工作負載。 可能不支援特定功能，或可能已經限制功能。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。
 
您變更此預設行為，以及對輸入 NAT 規則、負載平衡規則和[輸出規則](https://aka.ms/lboutboundrules)啟用傳送閒置逾時 TCP 重設。  根據規則啟用時，Load Balancer 會在達到所有相符流程的閒置逾時時，將雙向 TCP 重設 (TCP RST 封包) 傳送至用戶端和伺服器端點。

收到 TCP RST 封包的端點會立即關閉對應的通訊端。 這會立即通知端點有關已經發生連線的版本，而且相同 TCP 連線上的任何未來通訊都會失敗。  如果通訊端視需要關閉並重新建立連線，而不需要等到 TCP 連線最後逾時，則應用程式可以清除連線。

在許多情況下，這可能不需要傳送 TCP (或應用程式層) 保持運作，即可重新整理流程的閒置逾時。 

如果閒置持續時間超過設定所允許的值，或您的應用程式顯示已啟用 TCP 重設的非預期行為，您可能仍然需要使用 TCP Keepalive (或應用程式層 Keepalive) 來監視 TCP 連線的作用性。  此外，Keepalive 仍然適用於在路徑的某處 Proxy 處理連線時，特別是應用程式層 Keepalive。  

請仔細檢查整個完整情節，決定您是否受益於啟用 TCP 重設、調整閒置逾時，以及可能需要額外的步驟，以確保所需的應用程式行為。

## <a name="enabling-tcp-reset-on-idle-timeout"></a>啟用閒置逾時的 TCP 重設

使用 API 版本 2018-07-01，您可以啟用傳送每個規則的閒置逾時雙向 TCP 重設：

```json
      "loadBalancingRules": [
        {
          "enableTcpReset": true | false,
        }
      ]
```

```json
      "inboundNatRules": [
        {
          "enableTcpReset": true | false,
        }
      ]
```

```json
      "outboundRules": [
        {
          "enableTcpReset": true | false,
        }
      ]
```

## <a name="regions"></a> 區域可用性

此參數目前作用於下列區域。  在此處未列出的區域中，參數不會有任何作用。

| 區域 |
|---|
| 東南亞 |
| 西歐 |
| 美國東部 2 |
| 英國北部 |
| 美國西部 |

將預覽展開至其他區域時，將會更新此資料表。  

## <a name="limitations"></a>限制

- 有限的[區域可用性](#regions)。
- 入口網站無法用來設定或檢視 TCP 重設。  請改為使用範本、REST API、Az CLI 2.0 或 PowerShell。

## <a name="next-steps"></a>後續步驟

- 深入了解 [Standard Load Balancer](load-balancer-standard-overview.md)。
- 深入了解[輸出規則](load-balancer-outbound-rules-overview.md)。
