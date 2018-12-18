---
title: Azure Batch 的服務配額和限制 | Microsoft Docs
description: 了解預設的 Azure Batch 配額、限制和條件約束，以及如何要求增加配額
services: batch
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: 28998df4-8693-431d-b6ad-974c2f8db5fb
ms.service: batch
ms.workload: big-compute
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/24/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b3f4907d99b25df31ac7f081282cebe700f55b62
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39423737"
---
# <a name="batch-service-quotas-and-limits"></a>Batch 服務配額和限制

如同使用其他 Azure 服務，對於與 Batch 服務相關聯的特定資源有一些限制。 這其中有許多限制是 Azure 在訂用帳戶或帳戶層級上所套用的預設配額。 本文將討論這些預設值，以及如何申請加大配額。

當您設計和相應增加您的 Batch 工作負載時，請記住這些配額。 比方說，如果您的集區未達到您所指定的計算節點目標數目，您可能已達到 Batch 帳戶的核心配額限制。

您可以在單一 Batch 帳戶中執行多個 Batch 工作負載，或在相同訂用帳戶 (但不同 Azure 區域) 的 Batch 帳戶之間散佈工作負載。

如果您計劃在 Batch 中執行生產工作負載，您可能需要增加一或多個高於預設值的配額。 若要加大配額，您可以開啟線上 [客戶支援要求](#increase-a-quota) ，不另外加收費用。

> [!NOTE]
> 配額是一種信用限制，不是容量保證。 如果您有大規模的容量需求，請連絡 Azure 支援。
> 
> 

## <a name="resource-quotas"></a>資源配額
[!INCLUDE [azure-batch-limits](../../includes/azure-batch-limits.md)]


### <a name="cores-quotas-in-user-subscription-mode"></a>使用者訂用帳戶模式中的核心配額

如果您建立 Batch 帳戶時所用的集區配置模式是設定為**使用者訂用帳戶**，則配額會以不同的方式套用。 在這個模式中，建立集區時，Batch VM 和其他資源會直接建立在您的訂用帳戶中。 Azure Batch 核心配額不適用於在此模式中建立的帳戶。 相反地，會套用您在地區計算核心和其他資源之訂用帳戶中的配額。 深入了解 [Azure 訂用帳戶和服務限制、配額與限制](../azure-subscription-service-limits.md)中的配額。

## <a name="other-limits"></a>其他限制

| **Resource** | **上限** |
| --- | --- |
| [並行工作](batch-parallel-node-tasks.md)  |4 x 節點的核心數目 |
| [應用程式](batch-application-packages.md)  |20 |
| 每個應用程式的應用程式封裝 |40 |
| 工作存留期上限 | 7 天<sup>1</sup> |
| 計算節點間通訊已啟用集區中的節點 | 100 |

<sup>1</sup> 工作的最長存留期 (從它新增至作業到完成時) 為 7 天。 已完成的工作會無限期保留；無法存取未在最長存留期內完成之工作的資料。

## <a name="view-batch-quotas"></a>檢視 Batch 配額

在 [Azure 入口網站][portal]中檢視您的 Batch 帳戶配額。

1. 在入口網站中選取 [Batch 帳戶]  ，然後選取您感興趣的 Batch 帳戶。
1. 在 Batch 帳戶的功能表上選取 [配額]。
1. 檢視目前套用至 Batch 帳戶的配額
   
    ![Batch 帳戶配額][account_quotas]



## <a name="increase-a-quota"></a>增加配額

使用 [Azure 入口網站][portal]，遵循下列步驟來要求增加您 Batch 帳戶或訂用帳戶的配額。 增加配額的類型取決於您 Batch 帳戶的集區配置模式。

### <a name="increase-a-batch-cores-quota"></a>增加 Batch 的核心配額 

1. 選取入口網站儀表板上的 [說明 + 支援] 圖格或入口網站右上角的問號 (**？**)。
1. 選取 [新增支援要求] > [基本]。
1. 在 [基本] 中：
   
    a. **問題類型** > **配額**
   
    b. 選取您的訂用帳戶。
   
    c. **配額類型** > **批次**
   
    d. **支援方案** > **配額支援 - 已包含**
   
    按 [下一步] 。
1. 在 [問題]：
   
    a. 根據[商業影響][support_sev]選取 [嚴重性]。
   
    b. 在 [詳細資料] 中，指定每個您想要變更的配額、Batch 帳戶名稱和新限制。
   
    按 [下一步] 。
1. 在 [合約資訊] 中：
   
    a. 選取 [偏好的連絡方式]。
   
    b. 確認並輸入必要的連絡人詳細資料。
   
    按一下 [建立]  提交支援要求。

一旦您上傳支援要求，Azure 支援會與您連絡。 請注意，完成要求最多需要花費 2 個工作天。

## <a name="related-quotas-for-vm-pools"></a>適用於 VM 集區的相關配額

部署於 Azure 虛擬網路之虛擬機器設定中的 Batch 集區會自動配置額外的 Azure 網路資源。 針對虛擬網路中每 50 個集區節點，將會需要下列資源：

* 1 個[網路安全性群組](../virtual-network/security-overview.md#network-security-groups)
* 1 個[公用 IP 位址](../virtual-network/virtual-network-ip-addresses-overview-arm.md)
* 1 個[負載平衡器](../load-balancer/load-balancer-overview.md)

這些資源會配置於包含在建立 Batch 集區時所提供之虛擬網路的訂用帳戶中。 這些資源會被訂用帳戶的[資源配額](../azure-subscription-service-limits.md)所限制。 如果您計畫在虛擬網路中進行大型的集區部署，請檢查這些資源的訂用帳戶配額。 若有需要，請在 Azure 入口網站中選取 [説明 + 支援] 來要求提升配額。


## <a name="related-topics"></a>相關主題
* [使用 Azure 入口網站建立 Azure Batch 帳戶](batch-account-create-portal.md)
* [Azure Batch 功能概觀](batch-api-basics.md)
* [Azure 訂用帳戶和服務限制、配額與限制](../azure-subscription-service-limits.md)

[portal]: https://portal.azure.com
[portal_classic_increase]: https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/
[support_sev]: http://aka.ms/supportseverity

[account_quotas]: ./media/batch-quota-limit/accountquota_portal.png
