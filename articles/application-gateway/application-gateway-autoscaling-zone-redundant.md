---
title: Azure 中的自動調整規模和區域備援應用程式閘道 (公開預覽) | Microsoft Docs
description: 本文說明如何使用 Azure 入口網站來設定「應用程式閘道」中的 Web 應用程式防火牆要求大小限制和排除清單。
documentationcenter: na
services: application-gateway
author: vhorne
manager: jpconnock
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.custom: ''
ms.workload: infrastructure-services
ms.date: 09/26/2018
ms.author: victorh
ms.openlocfilehash: ab1c9405042de02183b8742fa940a3a5a482923a
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47165224"
---
# <a name="autoscaling-and-zone-redundant-application-gateway-public-preview"></a>自動調整規模和區域備援應用程式閘道 (公開預覽)

在提供效能增強功能並新增重要新功能 (例如自動調整規模、區域備援及靜態 VIP 支援) 支援的新 SKU 下，現以「公開預覽版」方式提供了「應用程式閘道」和「Web 應用程式防火牆」(WAF)。 除了已知限制一節中所列的幾個例外項目之外，新 SKU 會繼續支援已正式推出之 SKU 下的現有功能。 SKU 包含下列增強功能：

- **自動調整規模**：自動調整規模 SKU 下的「應用程式閘道」或 WAF 部署可以根據變動的流量負載模式來調升或調降規模。 自動調整規模也可讓您在佈建時，無須選擇部署大小或執行個體計數。 因此，SKU 可提供真正的靈活彈性。 在新 SKU 中，「應用程式閘道」除了可在啟用自動調整規模功能模式下運作之外，也可在固定容量 (停用自動調整規模功能) 模式下運作。 固定容量模式適用於工作負載一致且可預測的案例。 自動調整規模模式則有利於在應用程式流量中可看到許多變異的應用程式。
   
   > [!NOTE]
   > 目前針對 WAF SKU 並未提供自動調整規模功能。 請為 WAF 設定固定容量模式，而不是自動調整規模模式。
- **區域備援**：「應用程式閘道」或 WAF 部署可以跨多個「可用性區域」，讓您無須使用「流量管理員」在每個區域中佈建及運轉個別的「應用程式閘道」執行個體。 您可以選擇已部署「應用程式閘道」執行個體的單一區域或多個區域，藉此確保區域在失敗後能夠復原。 應用程式的後端集區可以均等分散在可用性區域間。
- **效能增強功能**：與已正式推出的 SKU 相比，自動調整規模 SKU 最多可提升 5 倍的 SSL 卸載效能。
- **更快的佈建和更新速度**：與已正式推出的 SKU 相比，自動調整規模 SKU 提供更快的佈建和更新速度。
- **靜態 VIP**：應用程式閘道 VIP 現在獨家支援靜態 VIP 類型。 這可確保與應用程式閘道相關的 VIP 即使在重新啟動後也不會變更。

> [!IMPORTANT]
> 自動調整規模和區域備援應用程式閘道 SKU 目前為公開預覽版。 此預覽版是在沒有服務等級協定的情況下提供，不建議用於生產工作負載。 可能不支援特定功能，或可能已經限制功能。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

![](./media/application-gateway-autoscaling-zone-redundant/application-gateway-autoscaling-zone-redundant.png)

## <a name="supported-regions"></a>支援區域
自動調整規模 SKU 適用區域包括美國東部 2、美國中部、美國西部 2、法國中部、西歐及東南亞。

## <a name="pricing"></a>價格
預覽期間無須付費。 將向您收取的是應用程式閘道以外資源 (例如 Key Vault、虛擬機器等) 的費用。 

## <a name="known-issues-and-limitations"></a>已知問題與限制

|問題|詳細資料|
|--|--|
|計費|目前不收費。|
|FIPS 模式、WebSocket|目前不支援。|
|純 ILB 模式|目前不支援。 支援將公用與 ILB 模式搭配使用。|
|Web 應用程式防火牆自動調整規模|WAF 不支援自動調整規模模式。 支援固定容量模式。|

## <a name="next-steps"></a>後續步驟
- [使用 Azure PowerShell 建立具有保留虛擬 IP 位址的自動調整規模、區域備援應用程式閘道](tutorial-autoscale-ps.md)
- 深入了解[應用程式閘道](overview.md)。
- 深入了解 [Azure 防火牆](../firewall/overview.md)。 

