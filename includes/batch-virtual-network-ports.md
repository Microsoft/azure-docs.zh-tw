---
title: 包含檔案
description: 包含檔案
services: batch
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: batch
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: ''
ms.date: 10/05/2018
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: 9246dea7fa12e5ac9378203e96352e917679525b
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49312558"
---
### <a name="general-requirements"></a>一般需求

* VNet 必須與您用來建立集區的 Batch 帳戶位於相同的訂用帳戶和區域中。

* 使用 VNet 的集區最多可以有 4096 個節點。

* 針對集區指定的子網路必須有足夠的未指派 IP 位址，可容納目標設為集區的 VM 數目；也就是集區之 `targetDedicatedNodes` 和 `targetLowPriorityNodes` 屬性的總和。 如果子網路沒有足夠的未指派 IP 位址，集區會局部配置計算節點，並發生調整大小錯誤。 

* 為 VNet 提供服務的任何自訂 DNS 服務，都必須能解析您的 Azure 儲存體端點。 具體而言，`<account>.table.core.windows.net`、`<account>.queue.core.windows.net` 和 `<account>.blob.core.windows.net` 形式的 URL 應該可解析。 

其他的 VNet 需求不盡相同，這取決於 Batch 集區是位於虛擬機器組態還是雲端服務組態中。 對於部署至 VNet 的新集區，建議使用虛擬機器組態。

### <a name="pools-in-the-virtual-machine-configuration"></a>虛擬機器組態中的集區

**支援的 Vnet** - 僅限以 Azure Resource Manager 為基礎的 Vnet

**子網路識別碼** - 使用 Batch API 指定子網路時，請使用子網路的*資源識別碼*。 子網路識別碼的格式為：

  ```
  /subscriptions/{subscription}/resourceGroups/{group}/providers/Microsoft.Network/virtualNetworks/{network}/subnets/{subnet}
  ```

**權限** - 請確認您對 VNet 的訂用帳戶或資源群組的安全性原則或鎖定是否限制了使用者管理 VNet 的權限。

**其他網路資源** - Batch 會在包含 VNet 的資源群組中自動配置其他網路資源。 對於每 50 個專用節點 (或每 20 個低優先順序節點)，Batch 將會配置：1 個網路安全性群組 (NSG)、1 個公用 IP 位址，和 1 個負載平衡器。 這些資源會被訂用帳戶的[資源配額](../articles/azure-subscription-service-limits.md)所限制。 對於大型集區，您可能必須要求增加其中一或多項資源的配額。

#### <a name="network-security-groups"></a>網路安全性群組

子網路必須允許來自 Batch 服務的輸入通訊，才能在計算節點上排程工作，且必須允許輸出通訊，才能與 Azure 儲存體或其他資源進行通訊。 對於虛擬機器組態中的集區，Batch 會在連結至 VM 的網路介面 (NIC) 層級上新增 NSG。 這些 NSG 會自動設定輸入和輸出規則，以允許下列流量：

* 連接埠 29876 和 29877 上來自 Batch 服務角色 IP 位址的輸入 TCP 流量。 
* 連接埠 22 (Linux 節點) 或連接埠 3389 (Windows 節點) 上的輸入 TCP 流量，以允許遠端存取。
* 任何連接埠上傳至虛擬網路的輸出流量。
* 任何連接埠上傳至網際網路的輸出流量。

> [!IMPORTANT]
> 如果您要在 Batch 設定的 NSG 中修改或新增輸入或輸出規則，請謹慎操作。 如果 NSG 拒絕對所指定子網路中計算節點的通訊，則 Batch 服務會將計算節點的狀態設為 [無法使用]。

您不需要在子網路層級指定 NSG，因為 Batch 會設定其本身的 NSG。 不過，如果指定的子網路有相關聯的網路安全性群組 (NSG) 和 (或) 防火牆，請設定輸入和輸出安全性規則，如下列表格所示。 只有在需要允許對集區 VM 的遠端存取時，才必須設定連接埠 3389 (Windows) 或 22 (Linux) 上的輸入流量。 集區 VM 不需要此設定即可使用。

<bpt id="p1">**</bpt>Inbound security rules<ept id="p1">**</ept>

| 來源 IP 位址 | 來源連接埠 | 目的地 | 目的地連接埠 | 通訊協定 | 動作 |
| --- | --- | --- | --- | --- | --- |
任意 <br /><br />雖然這實際上需要「全部允許」，但 Batch 服務會在每個依據虛擬機器組態建立的 VM 上套用網路介面層級的 NSG，而篩選掉所有非 Batch 服務的 IP 位址。 | * | 任意 | 29876-29877 | TCP | 允許 |
| 使用者電腦 (用於偵錯目的)，用以從遠端存取集區 VM。 | * | 任意 |  3389 (Windows)、22 (Linux) | TCP | 允許 |

**輸出安全性規則**

| 來源 | 來源連接埠 | 目的地 | 目的地服務標記 | 通訊協定 | 動作 |
| --- | --- | --- | --- | --- | --- |
| 任意 | 443 | [服務標記](../articles/virtual-network/security-overview.md#service-tags) | 儲存體 (與您的 Batch 帳戶和 VNet 位於相同的區域中)  | 任意 | 允許 |

### <a name="pools-in-the-cloud-services-configuration"></a>雲端服務組態中的集區

**支援的 VNet** - 僅限傳統 VNet

**子網路識別碼** - 使用 Batch API 指定子網路時，請使用子網路的*資源識別碼*。 子網路識別碼的格式為：

  ```
  /subscriptions/{subscription}/resourceGroups/{group}/providers/Microsoft.ClassicVirtualNetwork /virtualNetworks/{network}/subnets/{subnet}
  ```

**權限** - 針對指定的 VNet，`MicrosoftAzureBatch` 服務主體必須具有 `Classic Virtual Machine Contributor` 角色型存取控制 (RBAC) 角色。

#### <a name="network-security-groups"></a>網路安全性群組

子網路必須允許來自 Batch 服務的輸入通訊，才能在計算節點上排程工作，且必須允許輸出通訊，才能與 Azure 儲存體或其他資源進行通訊。

您不需要指定 NSG，因為 Batch 只會設定從 Batch IP 位址到集區節點的輸入通訊。 不過，如果指定的子網路有相關聯的 NSG 和 (或) 防火牆，請設定輸入和輸出安全性規則，如下列表格所示。 如果 NSG 拒絕對所指定子網路中計算節點的通訊，則 Batch 服務會將計算節點的狀態設為 [無法使用]。

 只有在需要允許對集區節點的遠端存取時，才必須設定連接埠 3389 (Windows) 或 22 (Linux) 上的輸入流量。 集區節點不需要此設定即可使用。

<bpt id="p1">**</bpt>Inbound security rules<ept id="p1">**</ept>

| 來源 IP 位址 | 來源連接埠 | 目的地 | 目的地連接埠 | 通訊協定 | 動作 |
| --- | --- | --- | --- | --- | --- |
任意 <br /><br />雖然這實際上需要「全部允許」，但 Batch 服務會在每個節點的層級上套用 ACL 規則，而篩選掉所有非 Batch 服務的 IP 位址。 | * | 任意 | 10100、20100、30100 | TCP | 允許 |
| 使用者電腦 (用於偵錯目的)，用以從遠端存取集區 VM。 | * | 任意 |  3389 (Windows)、22 (Linux) | TCP | 允許 |

**輸出安全性規則**

| 來源 | 來源連接埠 | 目的地 | 目的地連接埠 | 通訊協定 | 動作 |
| --- | --- | --- | --- | --- | --- |
| 任意 | * | 任意 | 443  | 任意 | 允許 |