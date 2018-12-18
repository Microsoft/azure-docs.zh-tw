---
title: 包含檔案
description: 包含檔案
services: azure-policy
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 09/18/2018
ms.author: dacoulte
ms.custom: include file
ms.openlocfilehash: df9c291136fc6a48effb08cd59eeb66486eb641c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47003614"
---
### <a name="virtual-networks"></a>虛擬網路

|  |  |
|---------|---------|
| [允許的應用程式閘道 SKU](../articles/governance/policy/samples/allowed-app-gate-sku.md) | 要求應用程式閘道使用已核准的 SKU。 您需指定已核准的 SKU 陣列。 |
| [禁止對 ER 網路進行網路對等互連](../articles/governance/policy/samples/no-peering-er-net.md) | 禁止將網路對等互連與指定資源群組中的網路建立關聯。 用來防止與中央受控網路基礎結構建立連線。 您需指定要防止建立關聯的資源群組名稱。 |
| [禁止使用者定義的路由表](../articles/governance/policy/samples/no-user-def-route-table.md)  |禁止使用使用者定義的路由表來部署虛擬網路。 |
| [對每個子網路使用 NSG X](../articles/governance/policy/samples/nsg-on-subnet.md) | 要求搭配每個虛擬子網路使用特定的網路安全性群組。 您需指定要使用之網路安全性群組的識別碼。 |
| [針對 VM 網路介面使用已核准的子網路](../articles/governance/policy/samples/use-approved-subnet-vm-nics.md) | 要求網路介面使用已核准的子網路。 您需指定已核准之子網路的識別碼。 |
| [針對 VM 網路介面使用已核准的 vNet](../articles/governance/policy/samples/use-approved-vnet-vm-nics.md) | 要求網路介面使用已核准的虛擬網路。 您需指定已核准之虛擬網路的識別碼。 |