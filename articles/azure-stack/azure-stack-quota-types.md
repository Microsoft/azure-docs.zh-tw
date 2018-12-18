---
title: Azure Stack 中的配額類型 | Microsoft Docs
description: 檢閱在 Azure Stack 中可用於服務和資源的不同配額類型。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 07/30/2018
ms.author: sethm
ms.reviewer: xiaofmao
ms.openlocfilehash: 3c0ab236dd6fce10be0a50c435f04517e14c1387
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49077590"
---
# <a name="quota-types-in-azure-stack"></a>Azure Stack 中的配額類型

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

[配額](azure-stack-plan-offer-quota-overview.md#plans)會定義使用者訂用帳戶可以佈建或取用的資源限制。 例如，配額可能會允許使用者建立最多五個 VM。 每個資源都可以有自己的配額類型。

## <a name="compute-quota-types"></a>計算配額類型 
| **類型** | **預設值** | **說明** |
| --- | --- | --- |
| 虛擬機器的數目上限 | 20 | 訂用帳戶可以在這個位置建立的虛擬機器數目上限。 |
| 虛擬機器核心的數目上限 | 50 | 訂用帳戶可以在這個位置建立的核心數目上限 (例如，A3 VM 有四個核心)。 |
| 可用性設定組的數目上限 | 10 | 可以在這個位置建立的可用性設定組數目上限。 |
| 虛擬機器擴展集的數目上限 | 20 | 可以在這個位置建立的虛擬機器擴展集數目上限。 |

## <a name="storage-quota-types"></a>儲存體配額類型 
| **Item** | **預設值** | **說明** |
| --- | --- | --- |
| 最大容量 (GB) |500 |訂用帳戶可以在這個位置取用的總儲存體容量。 |
| 儲存體帳戶的總數 |20 |訂用帳戶可以在這個位置建立的儲存體帳戶數目上限。 |

> [!NOTE]  
> 最多可能需要兩個小時，才會強制使用儲存體配額。


## <a name="network-quota-types"></a>網路配額類型
| **Item** | **預設值** | **說明** |
| --- | --- | --- |
| 公用 IP 的數目上限 |50 |訂用帳戶可以在這個位置建立的公用 IP 數目上限。 |
| 虛擬網路的數目上限 |50 |訂用帳戶可以在這個位置建立的虛擬網路數目上限。 |
| 虛擬網路閘道的數目上限 |1 |訂用帳戶可以在這個位置建立的虛擬網路閘道 (VPN 閘道) 數目上限。 |
| 網路連線的數目上限 |2 |訂用帳戶可以在這個位置所有虛擬網路閘道上建立的網路連線 (點對點或站台對站台) 數目上限。 |
| 負載平衡器的數目上限 |50 |訂用帳戶可以在這個位置建立的負載平衡器數目上限。 |
| 最大 NIC |100 |訂用帳戶可以在這個位置建立的網路介面數目上限。 |
| 網路安全性群組的數目上限 |50 |訂用帳戶可以在這個位置建立的網路安全性群組數目上限。 |

## <a name="view-an-existing-quota"></a>檢視現有的配額
1. 在系統管理員入口網站的預設儀表板上，尋找 [資源提供者] 圖格。
2. 選取您要檢視其配額的服務，像是 [計算] 或 [儲存體]。
3. 選取 [配額]，然後選取要檢視的配額。


## <a name="edit-a-quota"></a>編輯配額  
您可以選擇編輯配額的原始組態，而不是[使用附加方案](create-add-on-plan.md)。 當您編輯配額時，新的組態會自動全面套用至使用該配額的所有方案，以及使用這些方案的所有現有訂用帳戶。 配額的編輯方式不同於當您使用附加方案來提供修改過的配額時，這是使用者選擇要訂閱的方式。 

### <a name="to-edit-a-quota"></a>編輯配額  
1. 在系統管理員入口網站的預設儀表板上，尋找 [資源提供者] 圖格。
2. 選取您要修改其配額的服務，像是 [計算]、[網路] 或 [儲存體]。
3. 接下來，選取 [配額]，然後選取您要變更的配額。
4. 在 [設定配額] 窗格中，編輯其值，然後選取 [儲存]。 

此配額的新值會全面套用至使用已修改配額的所有方案，以及使用這些方案的所有現有訂用帳戶。 



## <a name="next-steps"></a>後續步驟

- [深入了解方案、供應項目與配額。](azure-stack-plan-offer-quota-overview.md)
- [建立方案時建立配額。](azure-stack-create-plan.md)
