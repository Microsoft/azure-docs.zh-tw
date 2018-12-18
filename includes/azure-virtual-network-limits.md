---
title: 包含檔案
description: 包含檔案
services: networking
author: jimdial
ms.service: networking
ms.topic: include
ms.date: 08/16/2018
ms.author: jdial
ms.custom: include file
ms.openlocfilehash: 673faff51c3d06e50941a6982a9f654c28598d86
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47006468"
---
<a name="virtual-networking-limits-classic"></a>下列限制僅適用於透過每個訂用帳戶的傳統部署模型所管理的網路資源。 深入了解如何[根據您的訂用帳戶限制檢視目前資源使用量](../articles/networking/check-usage-against-limits.md)。

| 資源 | 預設限制 | 上限 |
| --- | --- | --- |
| 虛擬網路 |50 |100 |
| 區域網路網站 |20 |連絡支援人員 |
| 每一虛擬網路的 DNS 伺服器 |20 |100 |
| 每個虛擬網路的私人 IP 位址 |4096 |4096 |
| 虛擬機器或角色執行個體之每個 NIC 的並行 TCP 或 UDP 流程 |500K |500K |
| 網路安全性群組 (NSG) |100 |200 |
| 每一 NSG 的 NSG 規則 |200 |1000 |
| 使用者定義的路由表 |100 |200 |
| 每個路由表的使用者定義路由 |100 |400 |
| 公用 IP 位址 (動態) |5 |連絡支援人員 |
| 保留的公用 IP 位址 |20 |連絡支援人員 |
| 每個部署的公用 VIP |5 |連絡支援人員 |
| 每個部署的私人 VIP (ILB) |1 |1 |
| 端點存取控制清單 (ACL) |50 |50 |

#### <a name="azure-resource-manager-virtual-networking-limits"></a>網路限制 - Azure Resource Manager
下列限制僅適用於透過每個訂用帳戶每一區域的 Azure Resource Manager 所管理的網路資源。 深入了解如何[根據您的訂用帳戶限制檢視目前資源使用量](../articles/networking/check-usage-against-limits.md)。

> [!NOTE]
> 我們最近已將所有預設限制提升至其最大限制。 如果沒有**上限**欄，資源即沒有可調整的限制。 如果您先前已透過支援增加這些限制，卻未看到如下所示的更新後限制，則可以[免費提出線上客戶支援要求](../articles/azure-resource-manager/resource-manager-quota-errors.md)

| 資源 | 預設限制 | 
| --- | --- |
| 虛擬網路 |1000 |
| 每一虛擬網路的子網路 |3000 |
| 每個虛擬網路的虛擬網路對等互連 |100 |
| 每一虛擬網路的 DNS 伺服器 |25 |
| 每個虛擬網路的私人 IP 位址 |65536 |
| 每個網路介面的私人 IP 位址 |256 |
| 虛擬機器或角色執行個體之每個 NIC 的並行 TCP 或 UDP 流程 |500K |
| 網路介面 (NIC) |24000 |
| 網路安全性群組 (NSG) |5000 |
| 每一 NSG 的 NSG 規則 |1000 |
| 針對安全性群組中的來源或目的地所指定的 IP 位址和範圍 |4000 |
| 應用程式安全性群組 |3000 |
| 每個 NIC、每個 IP 組態的應用程式安全性群組 |20 |
| 每個應用程式安全性群組的 IP 組態 |4000 |
| 可在網路安全性群組的所有安全性規則內指定的應用程式安全性群組 |100 |
| 使用者定義的路由表 |200 |
| 每個路由表的使用者定義路由 |400 |
| 每個 VPN 閘道的點對站根憑證 |20 |
| 虛擬網路 TAP |100 |
| 每個虛擬網路 TAP 的網路介面 TAP 設定 |100 |

#### <a name="publicip-address"></a>公用 IP 位址限制

| 資源 | 預設限制 | 上限 |
| --- | --- | --- |
| 公用 IP 位址 - 動態 |(基本) 200 |連絡支援人員 |
| 公用 IP 位址 - 靜態 |(基本) 200 |連絡支援人員 |
| 公用 IP 位址 - 靜態 |(標準) 200 |連絡支援人員 |

#### <a name="load-balancer"></a>負載平衡器限制
下列限制僅適用於透過每個訂用帳戶每一區域的 Azure Resource Manager 所管理的網路資源。 深入了解如何[根據您的訂用帳戶限制檢視目前資源使用量](../articles/networking/check-usage-against-limits.md)

| 資源 | 預設限制 | 上限 |
| --- | --- | --- |
| 負載平衡器 | 100 | 1000 |
| 每個資源的規則，基本 | 250 | 250 |
| 每個資源的規則，標準 | 1500 | 1500 |
| 每個 IP 設定的規則 | 299 |299 |
| 前端 IP 設定，基本 | 10 | 200 |
| 前端 IP 設定，標準 | 10 | 600 |
| 後端集區，基本 | 100，單一可用性設定組 | 100，單一可用性設定組 |
| 後端集區，標準 | 1000，單一 VNet | 1000，單一 VNet |
| 每個 Load Balancer 的後端資源，標準 * | 150 | 150 |
| HA 連接埠，標準 | 每個內部前端 1 個 | 每個內部前端 1 個 |

** 最多 150 個資源，獨立虛擬機器、可用性設定組與虛擬機器擴展集的任意組合。

如果您需要增加預設值以外的限制，請[連絡支援服務](../articles/azure-supportability/resource-manager-core-quotas-request.md )。

