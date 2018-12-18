---
title: 包含檔案
description: 包含檔案
services: billing
author: rothja
ms.service: billing
ms.topic: include
ms.date: 07/02/2018
ms.author: jroth
ms.custom: include file
ms.openlocfilehash: f47654d056a26f5a58b1ccc2c200f774bee1f19f
ms.sourcegitcommit: 756f866be058a8223332d91c86139eb7edea80cc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37348196"
---
| 資源 | 預設限制 | 上限 |
| --- | --- | --- |
| 每一[訂用帳戶](../articles/billing-buy-sign-up-azure-subscription.md) VM |每個區域 10,000 個 <sup>1</sup> |每個區域 10,000 個 |
| 每一[訂用帳戶](../articles/billing-buy-sign-up-azure-subscription.md)的 VM 總計核心 |每個區域 20<sup>1</sup> 個 | 請連絡支援人員 |
| 每一[訂用帳戶](../articles/billing-buy-sign-up-azure-subscription.md)的 VM 每個系列 (Dv2、F 等等) 核心 |每個區域 20<sup>1</sup> 個 | 請連絡支援人員 |
| [共同管理員](../articles/billing-add-change-azure-subscription-administrator.md)  |無限 |無限 |
| 每一訂用帳戶每一區域的[儲存體帳戶](../articles/storage/common/storage-create-storage-account.md) |200 |200<sup>2</sup> |
| 每一訂用帳戶[資源群組](../articles/azure-resource-manager/resource-group-overview.md) |980 |980 |
| 每一訂用帳戶[可用性設定組](../articles/virtual-machines/windows/manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy) |每個區域 2,000 個 |每個區域 2,000 個 |
| 資源管理員 API 讀取 |每小時 15,000 個 |每小時 15,000 個 |
| 資源管理員 API 寫入 |每小時 1,200 個 |每小時 1,200 個 |
| 資源管理員 API 要求的大小 |4,194,304 個位元組 |4,194,304 個位元組 |
| 每個訂用帳戶的標籤<sup>3</sup> |無限制 |無限制 |
| 每個訂用帳戶的唯一標記計算<sup>3</sup> | 10,000 | 10,000 |
| 每一訂用帳戶[雲端服務](../articles/cloud-services/cloud-services-choose-me.md) |不適用<sup>4</sup> |不適用<sup>4</sup> |
| 每一訂用帳戶[同質群組](../articles/virtual-network/virtual-networks-migrate-to-regional-vnet.md) |不適用<sup>4</sup> |不適用<sup>4</sup> |


  <sup>1</sup>預設限制會因供應項目類別類型 (如免費試用、隨收隨付和系列，例如 Dv2、F、G 等等) 而有所差異。

<sup>2</sup>這包括標準和進階儲存體帳戶。 如果您需要超過 200 個儲存體帳戶，請透過 [Azure 支援](https://azure.microsoft.com/support/faq/)提出要求。 Azure 儲存體團隊將會檢閱您的商務案例，而且可以核准多達 250 個儲存體帳戶。

<sup>3</sup>您可以套用每個訂用帳戶的標記數目不受限。 每個資源或資源群組的標記數目受限於 15。 當標記數目為 10,000 或更少時，Resource Manager 只會傳回訂用帳戶中[唯一標記名稱和值的清單](/rest/api/resources/tags#Tags_List)。 不過，當數目超出 10,000 時，您仍然可以依照標記尋找資源。  

<sup>4</sup>Azure 資源群組和 Azure 資源管理員已不再需要這些功能。

> [!NOTE]
> 請務必注意虛擬機器核心具有個別強制執行的區域總數限制，以及區域每個大小系列 (Dv2、F 等等) 的限制。  例如，請考慮美國東部訂用帳戶的總計 VM 核心限制為 30、A 系列核心限制為 30，和 D 系列核心限制為 30。  此訂用帳戶會允許部署 30 個 A1 VM、30 個 D1 VM，或是兩個的組合，總計不超過 30 個核心 (例如 10 個 A1 VM 和 20 個 D1 VM)。  
> <!-- -->
> 
> 

