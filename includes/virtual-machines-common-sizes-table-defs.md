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
ms.openlocfilehash: 30f6feb920d78c9c325c5556f5530ac4c8d9459b
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "29955657"
---
<!-- Not used for Ls-series -->

## <a name="size-table-definitions"></a>資料表大小定義

- 儲存容量會以 GiB 或是 1024^3 位元組為單位顯示。 當比較使用 GB (1000^3 位元組) 為度量單位的磁碟與使用 GiB (1024^3) 為度量單位的磁碟時，請記住以 GiB 為單位提供的容量數字可能較小。 例如，1023 GiB = 1098.4 GB
- 磁碟輸送量是以每秒輸入/輸出作業 (IOPS) 和 MBps 進行測量，其中 MBps = 10^6 位元組/每秒。
- 資料磁碟可以在快取模式或取消快取模式下運作。 針對快取的資料磁碟作業，主機快取模式必須設定為 **ReadOnly** 或 **ReadWrite**。  針對取消快取的資料磁碟作業，主機快取模式必須設定為 **None**。
-   如果您想要讓虛擬機器獲得最佳效能，應將資料磁碟數目限制為每個 vCPU 有 2 個磁碟。
- **預期的網路頻寬**是根據跨所有目的地之所有 NIC 的 VM 類型所[配置的最大彙總頻寬](../articles/virtual-network/virtual-machine-network-throughput.md)。 不保證上限，但會提供選取想要之應用程式的正確 VM 類型的指引。 實際網路效能取決於各種因素，包括網路壅塞、應用程式負載和網路設定。 如需最佳化網路輸送量的資訊，請參閱[最佳化 Windows 和 Linux 的網路輸送量](../articles/virtual-network/virtual-network-optimize-network-bandwidth.md)。 若要達到 Linux 或 Windows 上的預期網路效能，可能需要選取特定版本，或最佳化 VM。 如需詳細資訊，請參閱[如何可靠地測試虛擬機器輸送量](../articles/virtual-network/virtual-network-bandwidth-testing.md)。



