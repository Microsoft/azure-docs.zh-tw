---
title: Azure 的 vCPU 配額 | Microsoft Docs
description: 了解 Azure 的 vCPU 配額。
keywords: ''
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 05/31/2018
ms.author: cynthn
ms.openlocfilehash: 3e9a9e3f902439f0fc3f1fa4aa758d4fcd55f5bd
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048939"
---
# <a name="virtual-machine-vcpu-quotas"></a>虛擬機器 vCPU 配額

虛擬機器和虛擬機器擴展集的 vCPU 配額依每個地區的每個訂用帳戶分兩層排列。 第一層是「區域 vCPU 總計」，第二層是不同的 VM 大小系列核心，例如「D 系列 vCPU」。 在每次部署新的 VM 時，VM 的 vCPU 皆不得超過 VM 大小系列的 vCPU 配額或區域 vCPU 總配額。 如果超過這些配額，將不允許 VM 部署。 另外還有區域中虛擬機器整體數目的配額。 在 [Azure 入口網站](https://portal.azure.com)的 [訂用帳戶] 頁面的 [使用量 + 配額] 區段中可以看到這些配額的詳細資料，或者您可以使用 Azure CLI 查詢這些值。


## <a name="check-usage"></a>檢查使用量

您可以使用 [az vm list-usage](/cli/azure/vm#az_vm_list_usage)來檢查您配額使用量。

```azurecli-interactive
az vm list-usage --location "East US" -o table
```

輸出應該會看起來像這樣：


```
Name                                CurrentValue    Limit
--------------------------------  --------------  -------
Availability Sets                              0     2000
Total Regional vCPUs                          29      100
Virtual Machines                               7    10000
Virtual Machine Scale Sets                     0     2000
Standard DSv3 Family vCPUs                     8      100
Standard DSv2 Family vCPUs                     3      100
Standard Dv3 Family vCPUs                      2      100
Standard D Family vCPUs                        8      100
Standard Dv2 Family vCPUs                      8      100
Basic A Family vCPUs                           0      100
Standard A0-A7 Family vCPUs                    0      100
Standard A8-A11 Family vCPUs                   0      100
Standard DS Family vCPUs                       0      100
Standard G Family vCPUs                        0      100
Standard GS Family vCPUs                       0      100
Standard F Family vCPUs                        0      100
Standard FS Family vCPUs                       0      100
Standard Storage Managed Disks                 5    10000
Premium Storage Managed Disks                  5    10000
```

## <a name="reserved-vm-instances"></a>保留的 VM 執行個體
保留的 VM 執行個體 (僅限於單一訂用帳戶、沒有 VM 大小彈性) 將為 vCPU 配額新增新的層面。 這些值描述在訂用帳戶中必須是可部署的指定大小執行個體數目。 它們是配額系統中的預留位置，以確保該配額會保留起來，使 Azure 保留可於訂用帳戶中部署。 例如，如果特定訂用帳戶有 10 個 Standard_D1 保留，則 Standard_D1 保留的使用量限制將為 10。 這可讓 Azure 確保「區域 vCPU 總計」配額中永遠至少有 10 個 vCPU 可用於 Standard_D1 執行個體，「標準 D 系列 vCPU」配額中有至少 10 個 vCPU 可用於 Standard_D1 執行個體。

如果需要購買單一訂用帳戶 RI 來增加配額，您可以在訂用帳戶中[要求增加配額](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request)。

## <a name="next-steps"></a>後續步驟

如需有關帳單和配額的詳細資訊，請參閱 [Azure 訂用帳戶和服務限制、配額與限制](https://docs.microsoft.com/azure/azure-subscription-service-limits?toc=/azure/billing/TOC.json)。
