---
title: Azure Resource Manager vCPU 配額增加要求 | Microsoft Docs
description: Azure Resource Manager vCPU 配額增加要求
author: ganganarayanan
ms.author: gangan
ms.date: 6/13/2018
ms.topic: article
ms.service: microsoft-docs
ms.assetid: ce37c848-ddd9-46ab-978e-6a1445728a3b
ms.openlocfilehash: 7456785815dbefb2436713814965d90ba0e789ee
ms.sourcegitcommit: 04fc1781fe897ed1c21765865b73f941287e222f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39037232"
---
# <a name="resource-manager-vcpu-quota-increase-requests"></a>Resource Manager vCPU 配額增加要求

區域層級和 SKU 系列層級強制執行 Resource Manager vCPU 配額。
您可以在 [Azure 訂用帳戶和服務限制](http://aka.ms/quotalimits)頁面深入了解如何強制執行配額。
若要深入了解 SKU 系列，您可以在[虛擬機器價格](http://aka.ms/pricingcompute)頁面上比較成本和效能。

如需要求增加，請依照下列指示，該指示適用於透過 Azure 入口網站中可用的 Azure [使用量 + 配額] 刀鋒視窗，來建立支援要求。 

## <a name="request-quota-increase-at-subscription-level"></a>在訂用帳戶層級要求增加配額

1. 從 https://portal.azure.com 選取 [訂用帳戶]。

   ![訂用帳戶](./media/resource-manager-core-quotas-request/subscriptions.png)

2. 選取需要增加配額的訂用帳戶。

   ![選取訂用帳戶](./media/resource-manager-core-quotas-request/select-subscription.png)

3. 選取 [使用量 + 配額]

   ![選取使用量和配額](./media/resource-manager-core-quotas-request/select-usage-quotas.png)

4. 在右上角，選取 [要求增加配額]。

   ![要求增加配額](./media/resource-manager-core-quotas-request/request-increase.png)

5. 步驟：1 - 選取 [核心] 為報價類型。 

   ![填寫表單](./media/resource-manager-core-quotas-request/forms.png)
   
6. 步驟：2 - 在 [部署模型] 選取 [Resource Manager]，並選取位置。

    ![配額問題刀鋒視窗](./media/resource-manager-core-quotas-request/Problem-step.png)

3. 選取需要增加的 SKU 系列。

    ![選取的 SKU 系列](./media/resource-manager-core-quotas-request/SKU-selected.png)

4. 輸入您想要對訂用帳戶採取的新限制。

    ![SKU 新配額要求](./media/resource-manager-core-quotas-request/SKU-new-quota.png)

- 若要移除一行，請從 SKU 系列下拉式清單中取消核取 SKU，或按一下捨棄 [x] 圖示。
輸入每個 SKU 系列所需的配額後，請在 [問題步驟] 頁面上按 [下一步] 繼續建立支援要求。

