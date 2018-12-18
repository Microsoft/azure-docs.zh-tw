---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: jonbeck7
ms.service: virtual-machines
ms.topic: include
ms.date: 03/09/2018
ms.author: azcspmt;jonbeck;cynthn
ms.custom: include file
ms.openlocfilehash: eaf490979167ea5cbfe5846e62444526310452f8
ms.sourcegitcommit: 301855e018cfa1984198e045872539f04ce0e707
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36269539"
---
部分資料庫工作負載 (例如 SQL Server 或 Oracle) 需要大量記憶體、儲存體和 I/O 頻寬，而不需要大量核心計數。 許多資料庫工作負載不需要大量 CPU。 Azure 提供特定的 VM 大小，您可以在其中限制 VM vCPU 計數以降低軟體授權的成本，同時維持相同的記憶體、儲存體和 I/O 頻寬。

vCPU 計數可以限制為原始 VM 大小的一半或四分之一。 這些新的 VM 大小有尾碼，該尾碼會指定使用中 Vcpu 數目，讓您更容易識別。

例如，目前 VM 大小 Standard_GS5 隨附 32 個 vCPU、448 GB RAM、64 個磁碟 (高達 256 TB)，以及 80000 IOP 或 2 GB/s 的 I/O 頻寬。 新的 VM 大小 Standard_GS5-16 和 Standard_GS5-8 分別隨附 16 個和 8 個使用中 vCPU，同時維持著與 Standard_GS5 的記憶體、儲存體和 I/O 頻寬相同的規格。

針對 SQL Server 或 Oracle 計費的授權費用限制為新的 vCPU 計數，其他產品應該根據新的 vCPU 計數計費。 這樣會讓使用中 (可計費) vCPU 的 VM 規格增加 50% 到 75%。 僅適用於 Azure 的這些新的 VM 大小，可以讓工作負載以 (每個核心) 授權成本為準，推送更高的 CPU 使用率。 目前，計算成本 (包含 OS 授權) 維持與原始大小相同。 如需詳細資訊，請參閱[針對更多符合成本效益的資料庫工作負載推出新的 Azure VM 大小](https://azure.microsoft.com/blog/announcing-new-azure-vm-sizes-for-more-cost-effective-database-workloads/)。


| Name                | vCPU | 規格           |
|---------------------|------|-----------------|
| Standard_M8-2ms     | 2    | 與 M8ms 相同    |
| Standard_M8-4ms     | 4    | 與 M8ms 相同    |
| Standard_M16-4ms    | 4    | 與 M16ms 相同   |
| Standard_M16-8ms    | 8    | 與 M16ms 相同   |
| Standard_M32-8ms    | 8    | 與 M32ms 相同   |
| Standard_M32-16ms   | 16   | 與 M32ms 相同   |
| Standard_M64-32ms   | 32   | 與 M64ms 相同   |
| Standard_M64-16ms   | 16   | 與 M64ms 相同   |
| Standard_M128-64ms  | 64   | 與 M128ms 相同  |
| Standard_M128-32ms  | 32   | 與 M128ms 相同  |
| Standard_E4-2s_v3   | 2    | 與 E4s_v3 相同  |
| Standard_E8-4s_v3   | 4    | 與 E8s_v3 相同  |
| Standard_E8-2s_v3   | 2    | 與 E8s_v3 相同  |
| Standard_E16-8s_v3  | 8    | 與 E16s_v3 相同 |
| Standard_E16-4s_v3  | 4    | 與 E16s_v3 相同 |
| Standard_E32-16_v3  | 16   | 與 E32s_v3 相同 |
| Standard_E32-8s_v3  | 8    | 與 E32s_v3 相同 |
| Standard_E64-32s_v3 | 32   | 與 E64s_v3 相同 |
| Standard_E64-16s_v3 | 16   | 與 E64s_v3 相同 |
| Standard_GS4-8      | 8    | 與 GS4 相同     |
| Standard_GS4-4      | 4    | 與 GS4 相同     |
| Standard_GS5-16     | 16   | 與 GS5 相同     |
| Standard_GS5-8      | 8    | 與 GS5 相同     |
| Standard_DS11-1_v2  | 1    | 與 DS11_v2 相同 |
| Standard_DS12-2_v2  | 2    | 與 DS12_v2 相同 |
| Standard_DS12-1_v2  | 1    | 與 DS12_v2 相同 |
| Standard_DS13-4_v2  | 4    | 與 DS13_v2 相同 |
| Standard_DS13-2_v2  | 2    | 與 DS13_v2 相同 |
| Standard_DS14-8_v2  | 8    | 與 DS14_v2 相同 |
| Standard_DS14-4_v2  | 4    | 與 DS14_v2 相同 |
