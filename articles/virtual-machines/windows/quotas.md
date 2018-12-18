---
title: Azure 的 vCPU 配額 | Microsoft Docs
description: 了解 Azure 的 vCPU 配額。
keywords: ''
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 05/31/2018
ms.author: cynthn
ms.openlocfilehash: c766303920913a3ed6e654b3f9b29a0b99bb012a
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041270"
---
# <a name="virtual-machine-vcpu-quotas"></a>虛擬機器 vCPU 配額

虛擬機器和虛擬機器擴展集的 vCPU 配額依每個地區的每個訂用帳戶分兩層排列。 第一層是「區域 vCPU 總計」，第二層是不同的 VM 大小系列核心，例如「D 系列 vCPU」。 在每次部署新的 VM 時，VM 的 vCPU 皆不得超過 VM 大小系列的 vCPU 配額或區域 vCPU 總配額。 如果超過這些配額，將不允許 VM 部署。 另外還有區域中虛擬機器整體數目的配額。 在 [Azure 入口網站](https://portal.azure.com) [訂用帳戶] 頁面的 [使用量 + 配額] 區段中可以看到這些配額的詳細資料，或者您可以使用 PowerShell 查詢這些值。

 
## <a name="check-usage"></a>檢查使用量

您可以使用 [Get-AzureRmVMUsage](/powershell/module/azurerm.compute/get-azurermvmusage) Cmdlet 來檢查您的配額使用量。

```azurepowershell-interactive
Get-AzureRmVMUsage -Location "East US"
```

輸出類似如下範例：

```
Name                             Current Value Limit  Unit
----                             ------------- -----  ----
Availability Sets                            0  2000 Count
Total Regional vCPUs                         4   260 Count
Virtual Machines                             4 10000 Count
Virtual Machine Scale Sets                   1  2000 Count
Standard B Family vCPUs                      1    10 Count
Standard DSv2 Family vCPUs                   1   100 Count
Standard Dv2 Family vCPUs                    2   100 Count
Basic A Family vCPUs                         0   100 Count
Standard A0-A7 Family vCPUs                  0   250 Count
Standard A8-A11 Family vCPUs                 0   100 Count
Standard D Family vCPUs                      0   100 Count
Standard G Family vCPUs                      0   100 Count
Standard DS Family vCPUs                     0   100 Count
Standard GS Family vCPUs                     0   100 Count
Standard F Family vCPUs                      0   100 Count
Standard FS Family vCPUs                     0   100 Count
Standard NV Family vCPUs                     0    24 Count
Standard NC Family vCPUs                     0    48 Count
Standard H Family vCPUs                      0     8 Count
Standard Av2 Family vCPUs                    0   100 Count
Standard LS Family vCPUs                     0   100 Count
Standard Dv2 Promo Family vCPUs              0   100 Count
Standard DSv2 Promo Family vCPUs             0   100 Count
Standard MS Family vCPUs                     0     0 Count
Standard Dv3 Family vCPUs                    0   100 Count
Standard DSv3 Family vCPUs                   0   100 Count
Standard Ev3 Family vCPUs                    0   100 Count
Standard ESv3 Family vCPUs                   0   100 Count
Standard FSv2 Family vCPUs                   0   100 Count
Standard ND Family vCPUs                     0     0 Count
Standard NCv2 Family vCPUs                   0     0 Count
Standard NCv3 Family vCPUs                   0     0 Count
Standard LSv2 Family vCPUs                   0     0 Count
Standard Storage Managed Disks               2 10000 Count
Premium Storage Managed Disks                1 10000 Count
```


## <a name="reserved-vm-instances"></a>保留的 VM 執行個體
保留的 VM 執行個體 (僅限於單一訂用帳戶、沒有 VM 大小彈性) 將為 vCPU 配額新增新的層面。 這些值描述在訂用帳戶中必須是可部署的指定大小執行個體數目。 它們是配額系統中的預留位置，以確保該配額會保留起來，使保留的 VM 執行個體可於訂用帳戶中部署。 例如，如果特定訂用帳戶有 10 個 Standard_D1 保留的 VM 執行個體，則 Standard_D1 保留的 VM 執行個體使用量限制將為 10。 這可讓 Azure 確保「區域 vCPU 總計」配額中永遠至少有 10 個 vCPU 可用於 Standard_D1 執行個體，「標準 D 系列 vCPU」配額中有至少 10 個 vCPU 可用於 Standard_D1 執行個體。

如果需要購買單一訂用帳戶 RI 來增加配額，您可以在訂用帳戶中[要求增加配額](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request)。

## <a name="next-steps"></a>後續步驟

如需有關帳單和配額的詳細資訊，請參閱 [Azure 訂用帳戶和服務限制、配額與限制](https://docs.microsoft.com/azure/azure-subscription-service-limits?toc=/azure/billing/TOC.json)。
