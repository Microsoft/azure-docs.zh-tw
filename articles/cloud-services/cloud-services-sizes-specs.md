---
title: Azure 雲端服務的虛擬機器大小 | Microsoft Docs
description: 列出 Azure 雲端服務 Web 和背景工作角色的各種虛擬機器大小 (和識別碼)。
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 1127c23e-106a-47c1-a2e9-40e6dda640f6
ms.service: cloud-services
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 07/18/2017
ms.author: jeconnoc
ms.openlocfilehash: 60ee7c2be29791db91db78472cc002e15390d36b
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39004783"
---
# <a name="sizes-for-cloud-services"></a>雲端服務的大小
本主題描述雲端服務角色執行個體 (Web 角色和背景工作角色)的可用大小和選項。 同時也提供計劃使用這些資源時，需注意的部署考量。 每種大小都有一個識別碼，可讓您放入[服務定義檔](cloud-services-model-and-package.md#csdef)。 每種大小的價格可以在[雲端服務價格](https://azure.microsoft.com/pricing/details/cloud-services/)頁面上取得。

> [!NOTE]
> 若要查看相關的 Azure 限制，請參閱 ＜ [Azure 訂用帳戶和服務限制、配額及條件約束](../azure-subscription-service-limits.md)
>
>

## <a name="sizes-for-web-and-worker-role-instances"></a>用於 Web 和背景工作角色執行個體的大小
在 Azure 上有多個標準大小可選擇。 這些大小的一些考量事項包括：

* D 系列 VM 是為了執行要求更高計算能力和暫存磁碟效能的應用程式所設計。 D 系列 VM 提供更快的處理器、較高的記憶體與核心比率，以及適用於暫存磁碟的固態硬碟 (SSD)。 如需詳細資訊，請參閱 Azure 部落格的公告， [新 D 系列的虛擬機器大小](https://azure.microsoft.com/blog/2014/09/22/new-d-series-virtual-machine-sizes/)。
* Dv3 系列、Dv2 系列是原始 D 系列的延續，擁有更強大的 CPU。 Dv2 系列 CPU 比 D 系列 CPU 快約 35%。 它以最新一代的 2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) 處理器為基礎，搭配 Intel Turbo Boost Technology 2.0，最高可達 3.1 GHz。 Dv2 系列的記憶體和磁碟組態和 D 系列一樣。
* G 系列 VM 提供最大的記憶體，並且是在具有 Intel Xeon E5 V3 系列處理器的主機上執行。
* A 系列 VM 可以部署在各種不同的硬體類型和處理器上。 根據硬體節流大小，為執行中的執行個體提供一致的處理器效能，不論硬體部署的位置。 若要判斷此大小部署所在的實體硬體，請從虛擬機器內查詢虛擬硬體。
* A0 大小是在實體硬體上過度訂閱。 僅針對這個特定大小，其他客戶部署可能會影響您正在執行的工作負載的效能。 以下概述的相對效能為預期的基準，受限於近似變化性的 15%。

虛擬機器的大小會影響定價。 大小也會影響虛擬機器的處理、記憶體和儲存體容量。 儲存體成本會分別根據儲存體帳戶中使用的頁面來計算。 如需詳細資訊，請參閱[雲端服務價格詳細資料](https://azure.microsoft.com/pricing/details/cloud-services/)和 [Azure 儲存體價格](https://azure.microsoft.com/pricing/details/storage/)。

下列考量可協助您決定大小：

* A8-A11 和 H 系列大小也稱為 *計算密集型執行個體*。 執行這些大小的硬體是針對計算密集型和網路密集型應用程式 (包括高效能運算 (HPC) 叢集應用程式)、模型化及模擬而設計及最佳化的。 A8-A11 系列使用 Intel Xeon E5-2670 @ 2.6 GHZ，而 H 系列使用 Intel Xeon E5-2667 v3 @ 3.2 GHz。 如需有關使用這些大小的詳細資訊與考量，請參閱[高效能計算 VM 大小](../virtual-machines/windows/sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
* Dv3 系列、Dv2 系列、D 系列和 G 系列是要求更快速的 CPU、更好的本機磁碟效能，或有更高記憶體需求之應用程式的最佳選擇。 它們為許多企業級應用程式提供了強大的組合。
* Azure 資料中心的某些實體主機可能不支援較大的虛擬機器大小，例如 A5-A11。 因此，您可能會在將現有的虛擬機器調整為新的大小、在 2013 年 4 月 16 日之前建立的虛擬網路中建立新的虛擬機器，或將新的虛擬機器新增至現有雲端服務時，看到錯誤訊息：**無法設定虛擬機器 {machine name}** 或**無法建立虛擬機器 {machine name}**。 請參閱支援論壇上 [錯誤：「無法設定虛擬機器」](https://social.msdn.microsoft.com/Forums/9693f56c-fcd3-4d42-850e-5e3b56c7d6be/error-failed-to-configure-virtual-machine-with-a5-a6-or-a7-vm-size?forum=WAVirtualMachinesforWindows) ，以查看每個部署案例的因應措施。
* 您的訂用帳戶也可能會限制您可以在特定大小系列中部署的核心數目。 若要增加配額，請連絡 Azure 支援服務。

## <a name="performance-considerations"></a>效能考量
我們已經建立「Azure 運算單位」(ACU) 概念來提供一種比較各個 Azure SKU 之運算 (CPU) 效能的方法，以及識別哪個 SKU 最能滿足您的效能需求。  ACU 目前是以「小型 (Standard_A1)」VM 為標準 (數值為 100)，而所有其他 SKU 則大致上代表該 SKU 在執行標準基準測試上可以快多少。

> [!IMPORTANT]
> ACU 只是一個指導方針。 您工作負載的結果可能會有所不同。
>
>

<br>

| SKU 系列 | ACU/核心 |
| --- | --- |
| [ExtraSmall](#a-series) |50 |
| [Small-ExtraLarge](#a-series) |100 |
| [A5-7](#a-series) |100 |
| [A8-A11](#a-series) |225* |
| [A v2](#av2-series) |100 |
| [D](#d-series) |160 |
| [D v2](#dv2-series) |160 - 190* |
| [D v3](#dv3-series) |160 - 190* |
| [E v3](#ev3-series) |160 - 190* |
| [G](#g-series) |180 - 240* |
| [H](#h-series) |290 - 300* |

以 * 標示的 ACU 使用了「Intel® 渦輪」技術來增加 CPU 頻率及提升效能。 提升的程度會依 VM 大小、工作負載及在相同主機上執行的其他工作負載而有所不同。

## <a name="size-tables"></a>大小資料表
下表顯示其所提供的大小和容量。

* 儲存容量會以 GiB 或是 1024^3 位元組為單位顯示。 當比較使用 GB (1000^3 位元組) 為度量單位的磁碟與使用 GiB (1024^3) 為度量單位的磁碟時，請記住以 GiB 為單位提供的容量數字可能較小。 例如，1023 GiB = 1098.4 GB
* 磁碟輸送量是以每秒輸入/輸出作業 (IOPS) 和 MBps 進行測量，其中 MBps = 10^6 位元組/每秒。
* 資料磁碟可以在快取模式或取消快取模式下運作。 針對快取的資料磁碟作業，主機快取模式必須設定為 **ReadOnly** 或 **ReadWrite**。 針對取消快取的資料磁碟作業，主機快取模式必須設定為 **None**。
* 最大網路頻寬是依據各 VM 類型而配置與指派的最大彙總頻寬。 最大頻寬能指導我們選取正確的 VM 類型，以確保有適當的網路容量可用。 在低、中度、高和極高之間移動時，輸送量將隨之增加。 實際的網路效能將取決於許多因素，包括網路和應用程式負載，以及應用程式的網路設定。

## <a name="a-series"></a>A 系列
| 大小            | CPU 核心 | 記憶體：GiB  | 暫存位置：GiB       | 最大 NIC / 網路頻寬 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| 特小型      | 1         | 0.768        | 20                   | 1 / 低 |
| 小型           | 1         | 1.75         | 225                  | 1 / 中 |
| 中          | 2         | 3.5          | 490                  | 1 / 中 |
| 大型           | 4         | 7            | 1000                 | 2 / 高 |
| 特大型      | 8         | 14           | 2040                 | 4 / 高 |
| A5              | 2         | 14           | 490                  | 1 / 中 |
| A6              | 4         | 28           | 1000                 | 2 / 高 |
| A7              | 8         | 56           | 2040                 | 4 / 高 |

## <a name="a-series---compute-intensive-instances"></a>A 系列 - 大量計算執行個體
如需有關使用這些大小的資訊與考量，請參閱[高效能計算 VM 大小](../virtual-machines/windows/sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

| 大小            | CPU 核心 | 記憶體：GiB  | 暫存位置：GiB       | 最大 NIC / 網路頻寬 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| A8*             |8          | 56           | 1817                 | 2 / 高 |
| A9*             |16         | 112          | 1817                 | 4 / 非常高 |
| A10             |8          | 56           | 1817                 | 2 / 高 |
| A11             |16         | 112          | 1817                 | 4 / 非常高 |

\*支援 RDMA

## <a name="av2-series"></a>Av2 系列

| 大小            | CPU 核心 | 記憶體：GiB  | 暫存位置 (SSD)：GiB       | 最大 NIC / 網路頻寬 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Standard_A1_v2  | 1         | 2            | 10                   | 1 / 中                 |
| Standard_A2_v2  | 2         | 4            | 20                   | 2 / 中                 |
| Standard_A4_v2  | 4         | 8            | 40                   | 4 / 高                     |
| Standard_A8_v2  | 8         | 16           | 80                   | 8 / 高                     |
| Standard_A2m_v2 | 2         | 16           | 20                   | 2 / 中                 |
| Standard_A4m_v2 | 4         | 32           | 40                   | 4 / 高                     |
| Standard_A8m_v2 | 8         | 64           | 80                   | 8 / 高                     |


## <a name="d-series"></a>D 系列
| 大小            | CPU 核心 | 記憶體：GiB  | 暫存位置 (SSD)：GiB       | 最大 NIC / 網路頻寬 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| 標準_D1     | 1         | 3.5          | 50                   | 1 / 中 |
| 標準_D2     | 2         | 7            | 100                  | 2 / 高 |
| 標準_D3     | 4         | 14           | 200                  | 4 / 高 |
| 標準_D4     | 8         | 28           | 400                  | 8 / 高 |
| 標準_D11    | 2         | 14           | 100                  | 2 / 高 |
| 標準_D12    | 4         | 28           | 200                  | 4 / 高 |
| 標準_D13    | 8         | 56           | 400                  | 8 / 高 |
| 標準_D14    | 16        | 112          | 800                  | 8 / 非常高 |

## <a name="dv2-series"></a>Dv2 系列
| 大小            | CPU 核心 | 記憶體：GiB  | 暫存位置 (SSD)：GiB       | 最大 NIC / 網路頻寬 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Standard_D1_v2  | 1         | 3.5          | 50                   | 1 / 中 |
| Standard_D2_v2  | 2         | 7            | 100                  | 2 / 高 |
| Standard_D3_v2  | 4         | 14           | 200                  | 4 / 高 |
| Standard_D4_v2  | 8         | 28           | 400                  | 8 / 高 |
| Standard_D5_v2  | 16        | 56           | 800                  | 8 / 極高 |
| Standard_D11_v2 | 2         | 14           | 100                  | 2 / 高 |
| Standard_D12_v2 | 4         | 28           | 200                  | 4 / 高 |
| Standard_D13_v2 | 8         | 56           | 400                  | 8 / 高 |
| Standard_D14_v2 | 16        | 112          | 800                  | 8 / 極高 |
| Standard_D15_v2 | 20        | 140          | 1,000                | 8 / 極高 |

## <a name="dv3-series"></a>Dv3 系列

| 大小            | CPU 核心 | 記憶體：GiB   | 暫存位置 (SSD)：GiB       | 最大 NIC / 網路頻寬 |
|---------------- | --------- | ------------- | -------------------- | ---------------------------- |
| Standard_D2_v3  | 2         | 8             | 50                   | 2 / 中 |
| Standard_D4_v3  | 4         | 16            | 100                  | 2 / 高 |
| Standard_D8_v3  | 8         | 32            | 200                  | 4 / 高 |
| Standard_D16_v3 | 16        | 64            | 400                  | 8 / 極高 |
| Standard_D32_v3 | 32        | 128           | 800                  | 8 / 極高 |
| Standard_D64_v3 | 64        | 256           | 1600                 | 8 / 極高 |

## <a name="ev3-series"></a>Ev3 系列

| 大小            | CPU 核心 | 記憶體：GiB   | 暫存位置 (SSD)：GiB       | 最大 NIC / 網路頻寬 |
|---------------- | --------- | ------------- | -------------------- | ---------------------------- |
| Standard_E2_v3  | 2         | 16            | 50                   | 2 / 中 |
| Standard_E4_v3  | 4         | 32            | 100                  | 2 / 高 |
| Standard_E8_v3  | 8         | 64            | 200                  | 4 / 高 |
| Standard_E16_v3 | 16        | 128           | 400                  | 8 / 極高 |
| Standard_E32_v3 | 32        | 256           | 800                  | 8 / 極高 |
| Standard_E64_v3 | 64        | 432           | 1600                 | 8 / 極高 |


## <a name="g-series"></a>G 系列
| 大小            | CPU 核心 | 記憶體：GiB  | 暫存位置 (SSD)：GiB       | 最大 NIC / 網路頻寬 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Standard_G1     | 2         | 28           | 384                  |1 / 高 |
| Standard_G2     | 4         | 56           | 768                  |2 / 高 |
| Standard_G3     | 8         | 112          | 1,536                |4 / 非常高 |
| Standard_G4     | 16        | 224          | 3,072                |8 / 極高 |
| Standard_G5     | 32        | 448          | 6,144                |8 / 極高 |

## <a name="h-series"></a>H 系列
Azure H 系列虛擬機器是下一代高效能運算 VM，以高端運算需求為目標，例如分子建模以及運算流體力學。 這些 8 與 16 核心 VM 是以 Intel Haswell E5-2667 V3 處理器技術，搭載 DDR4 記憶體與本機 SSD 型儲存體為基礎建置。

除了大量的 CPU 能力，H 系列使用 FDR InfiniBand 與數個記憶體組態，針對低延遲 RDMA 網路提供不同的選項，以支援記憶體大量運算需求。

| 大小            | CPU 核心 | 記憶體：GiB  | 暫存位置 (SSD)：GiB       | 最大 NIC / 網路頻寬 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Standard_H8     | 8         | 56           | 1000                 | 8 / 高 |
| Standard_H16    | 16        | 112          | 2000                 | 8 / 非常高 |
| Standard_H8m    | 8         | 112          | 1000                 | 8 / 高 |
| Standard_H16m   | 16        | 224          | 2000                 | 8 / 非常高 |
| Standard_H16r*  | 16        | 112          | 2000                 | 8 / 非常高 |
| Standard_H16mr* | 16        | 224          | 2000                 | 8 / 非常高 |

\*支援 RDMA

## <a name="configure-sizes-for-cloud-services"></a>設定雲端服務大小
您可以指定角色執行個體的虛擬機器大小，作為 [服務定義檔](cloud-services-model-and-package.md#csdef)所描述之服務模型的一部分。 角色大小決定 CPU 核心數目、記憶體容量，以及配置給執行中執行個體的本機檔案系統大小。 根據您應用程式的資源需求選擇角色大小。

以下是將 Web 角色執行個體的角色大小設定為 [Standard_D2](#general-purpose-d) 的範例：

```xml
<WorkerRole name="Worker1" vmsize="Standard_D2">
...
</WorkerRole>
```

## <a name="changing-the-size-of-an-existing-role"></a>變更現有角色的大小

因為您的工作負載的本質變更或新的 VM 大小變得可用，您可能想要變更您的角色大小。 若要這樣做，您必須變更您的服務定義檔中的 VM 大小 (如上所示)、重新封裝您的雲端服務，然後部署。 您不可以直接從入口網站或 PowerShell 變更 VM 大小。

>[!TIP]
> 您可能想要在不同環境中對您的角色使用不同 VM 大小 (例如， 測試與生產環境)。 其中一種方式是在專案中建立多個服務定義 (.csdef) 檔案，然後在使用 CSPack 工具自動化建置期間於每個環境建立不同的雲端服務套件。 若要深入了解雲端服務套件的元素，以及如何建立這些元素，請參閱[什麼是雲端服務模型，以及如何封裝？](cloud-services-model-and-package.md)
>
>

## <a name="get-a-list-of-sizes"></a>取得大小清單
您可以使用 PowerShell 或 REST API 來取得大小清單。 如需 REST API 的相關記載，請參閱[這裡](https://msdn.microsoft.com/library/azure/dn469422.aspx)。 以下程式碼是一個 PowerShell 命令，此命令會列出「雲端服務」可用的所有大小。 

```powershell
Get-AzureRoleSize | where SupportedByWebWorkerRoles -eq $true | select InstanceSize, RoleSizeLabel
```

## <a name="next-steps"></a>後續步驟
* 了解 [Azure 訂用帳戶和服務限制、配額與限制](../azure-subscription-service-limits.md)。
* 進一步了解 HPC 工作負載的[有關高效能計算 VM 大小](../virtual-machines/windows/sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
