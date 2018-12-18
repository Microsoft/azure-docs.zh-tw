---
title: 什麼是 Azure 網路存取控制清單？
description: 了解 Azure 中的存取控制清單
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: ''
tags: azure-service-management
ms.assetid: 83d66c84-8f6b-4388-8767-cd2de3e72d76
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/15/2016
ms.author: genli
ms.openlocfilehash: 6265a7f9f60b60d7b79e15a04feb0ec76dc59f6d
ms.sourcegitcommit: 1438b7549c2d9bc2ace6a0a3e460ad4206bad423
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36293113"
---
# <a name="what-is-an-endpoint-access-control-list"></a>什麼是端點存取控制清單？

> [!IMPORTANT]
> Azure 有兩種不同的[部署模型](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)來建立和使用資源：Resource Manager 和傳統。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署都使用 Resource Manager 部署模型。 

端點存取控制清單 (ACL) 是可供您的 Azure 部署使用的安全性增強功能。 ACL 提供針對虛擬機器端點選擇性允許或拒絕流量的功能。 此封包篩選功能提供了一層額外的安全性。 您可以僅針對端點指定網路 ACL。 您無法針對虛擬網路或是包含在虛擬網路中的特定子網路指定 ACL。 建議您儘可能使用網路安全性群組 (NSG) 來取代 ACL。 使用 NSG 時，端點存取控制清單會遭到取代，且不必再強制使用。 若要深入了解 NSG，請參閱[網路安全性群組概觀](security-overview.md)

您可以使用 PowerShell 或 Azure 入口網站來設定 ACL。 若要使用 PowerShell 來設定網路 ACL，請參閱[使用 PowerShell 來管理端點的存取控制清單](virtual-networks-acl-powershell.md)。 若要使用 Azure 入口網站來設定網路 ACL，請參閱[如何設定虛擬機器的端點](../virtual-machines/windows/classic/setup-endpoints.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。

您可以使用網路 ACL 執行下列作業：

* 根據遠端子網路與虛擬機器輸入端點的 IPv4 位址範圍，選擇性允許或拒絕傳入流量。
* 將 IP 位址加入黑名單
* 為每個虛擬機器端點建立多個規則
* 使用規則順序以確保正確的規則集套用於指定的虛擬機器端點 (從最低到最高)
* 為特定遠端子網路 IPv4 位址指定 ACL。

如需了解 ACL 限制，請參閱 [Azure 限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits)一文。

## <a name="how-acls-work"></a>ACL 的運作方式
ACL 是包含規則清單的物件。 當您建立 ACL 並將它套用到虛擬機器端點時，封包篩選會在您 VM 的主機節點上進行。 這表示來自遠端 IP 位址的流量會透過符合 ACL 規則的主機節點進行篩選，而非在您的 VM 上。 這可讓您的 VM 在封包篩選時免於耗費珍貴的 CPU 週期。

建立虛擬機器後，預設的 ACL 將準備好封鎖所有傳入流量。 不過，如果所建立的端點使用 (連接埠 3389)，則預設 ACL 會修改成允許該端點的所有輸入流量。 接著會允許來自任何遠端子網路的輸入流量傳輸至該端點，且不需要佈建任何防火牆。 除非建立那些連接埠的端點，否則會封鎖所有其他連接埠則的輸入流量。 預設為允許輸出流量。

**範例預設 ACL 資料表**

| **規則編號** | **遠端子網路** | **端點** | **允許/拒絕** |
| --- | --- | --- | --- |
| 100 |0.0.0.0/0 |3389 |允許 |

## <a name="permit-and-deny"></a>允許和拒絕
您可以透過建立指定「允許」或「拒絕」的規則，選擇性允許或拒絕虛擬機器輸入端點的網路流量。 請務必注意建立端點時，預設為允許端點的所有流量。 基於這個原因，如果您想要更精確地控制允許連接到虛擬機器端點的網路流量，請務必了解如何建立允許/拒絕規則，並將它們放在適當的優先順序。

考慮事項：

1. **無 ACL –** 建立端點後，我們預設允許所有端點的流量。
2. **允許 -** 當您新增一個或多個「允許」範圍時，將預設為拒絕所有其他範圍。 僅來自允許 IP 範圍的封包能夠與虛擬機器端點進行通訊。
3. **拒絕 -** 當您新增一個或多個「拒絕」範圍時，將預設為允許所有其他範圍。
4. **允許和拒絕的組合 -** 當您想要規劃一個允許或拒絕的特定 IP 範圍時，您可以使用「允許」和「拒絕」的組合。

## <a name="rules-and-rule-precedence"></a>規則和規則優先順序
網路 ACL 可以在特定的虛擬機器端點上進行設定。 例如，您可以針對在虛擬機器上建立的 RDP 端點指定網路 ACL，以便鎖定特定 IP 位址的存取權。 下方資料表顯示授與特定範圍公用虛擬 IP (VIP) 的存取權來允許存取 RDP 的方式。 所有其他遠端 IP 會遭到拒絕。 我們遵循 *最低優先* 的規則順序。

### <a name="multiple-rules"></a>多個規則
在下方範例中，如果您想要允許僅能從兩個公用 IPv4 位址範圍 (65.0.0.0/8，以及 159.0.0.0/8) 來存取 RDP 端點，您可以指定兩個 *允許* 規則來達到這個目的。 在此案例中，因為預設會建立虛擬機器的 RDP，您可能要根據遠端子網路鎖定 RDP 連接埠的存取權。 下方範例顯示授與特定範圍公用虛擬 IP (VIP) 的存取權來允許存取 RDP 的方式。 所有其他遠端 IP 會遭到拒絕。 這是因為網路 ACL 可以針對特定虛擬機器端點進行設定，且預設為拒絕存取。

**範例 – 多個規則**

| **規則編號** | **遠端子網路** | **端點** | **允許/拒絕** |
| --- | --- | --- | --- |
| 100 |65.0.0.0/8 |3389 |允許 |
| 200 |159.0.0.0/8 |3389 |允許 |

### <a name="rule-order"></a>規則順序
因為端點可以指定多個規則，故必須依靠一種方式組織規則，以判斷規則的優先順序。 規則順序會指定優先順序。 網路 ACL 遵循 *最低優先* 的規則順序。 在下方範例中，連接埠 80 上的端點會選擇性僅針對某些特定 IP 位址範圍授與存取權。 若要進行此設定，我們可以針對 175.1.0.1/24 空間中的位址使用拒絕規則 (規則\# 100)。 接著指定第二個規則的優先順序為 200，以便允許 175.0.0.0/8 下所有其他位址的存取。

**範例 – 規則優先順序**

| **規則編號** | **遠端子網路** | **端點** | **允許/拒絕** |
| --- | --- | --- | --- |
| 100 |175.1.0.1/24 |80 |拒絕 |
| 200 |175.0.0.0/8 |80 |允許 |

## <a name="network-acls-and-load-balanced-sets"></a>網路 ACL 和負載平衡集合
您可以在負載平衡集端點上指定網路 ACL。 如果已為負載平衡集指定 ACL，系統就會將網路 ACL 套用至該負載平衡集內的所有虛擬機器。 例如，如果建立一個使用「連接埠 80」的負載平衡集，而該負載平衡集包含 3 個 VM，則在其中一個 VM 的端點「連接埠 80」上建立的網路 ACL 會自動套用至其他 VM。

![網路 ACL 和負載平衡集合](./media/virtual-networks-acl/IC674733.png)

## <a name="next-steps"></a>後續步驟
[使用 PowerShell 來管理端點的存取控制清單](virtual-networks-acl-powershell.md)

