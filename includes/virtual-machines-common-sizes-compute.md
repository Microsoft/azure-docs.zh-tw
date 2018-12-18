---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: jonbeck7
ms.service: virtual-machines
ms.topic: include
ms.date: 07/06/2018
ms.author: azcspmt;jonbeck;cynthn
ms.custom: include file
ms.openlocfilehash: 033ae1de25fbaada0c2bce715e6bdd71818b341a
ms.sourcegitcommit: d551ddf8d6c0fd3a884c9852bc4443c1a1485899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2018
ms.locfileid: "37906797"
---
<!-- F-series, Fs-series* -->

計算最佳化的 VM 大小，具有高 CPU 對記憶體比率，並且適用於中流量網頁伺服器、網路設備、批次處理及應用程式伺服器。 本文提供 vCPU 數量、資料磁碟和 NIC 的相關資訊，以及此群組中各種大小之儲存體輸送量和網路頻寬的相關資訊。

Fsv2 系列是根據 Intel® Xeon® Platinum 8168 處理器，具備 2.7 GHz 基底核心頻率，以及 3.7 GHz 的最大單一核心加速頻率。 Intel® AVX-512 指示，在 Intel 可擴充處理器上是新功能，為單精確度浮點數和雙精確度浮點數作業上的向量處理工作負載皆提供最多 2X 效能加速。 也就是說，它們確實比任何計算工作負載還要快。 

Fsv2 系列的每小時訂價較低，在 Azure 產品組合中，就每一 vCPU 的「Azure 計算單位」(ACU) 而言，具有最佳的價格/性能比表現。 

F 系列是以 2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) 處理器為基礎，在 Intel Turbo Boost Technology 2.0 的搭配下，時脈速度最高可達 3.1 GHz。 這是與 Dv2 系列 VM 同等級的 CPU 效能。  

對於需要較快的 CPU 但每一 vCPU 不需要太多記憶體或暫存儲存體的工作負載來說，F 系列 VM 是一個絕佳選擇。  分析、遊戲伺服器、Web 伺服器及批次處理之類的工作負載都將因 F 系列的實用性而受益。

Fs 系列可提供 F 系列的所有優點 (進階儲存體除外)。

## <a name="fsv2-series-sup1sup"></a>Fsv2 系列 <sup>1</sup>

ACU：195 - 210

進階儲存體：支援

進階儲存體快取：支援

| 大小             | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大資料磁碟 | 最大快取和暫存儲存體輸送量︰IOPS / MBps (以 GiB 為單位的快取大小) | 最大 NIC/預期的網路頻寬 (Mbps) |
|------------------|--------|-------------|----------------|----------------|-----------------------------------------------------------------------|------------------------------------------------|
| Standard_F2s_v2  | 2      | 4           | 16             | 4              | 4000 (32)                                                             | 2 / 875                                        |
| Standard_F4s_v2  | 4      | 8           | 32             | 8              | 8000 (64)                                                             | 2 / 1,750                                      |
| Standard_F8s_v2   | 8      | 16          | 64             | 16             | 16000 (128)                                                           | 4 / 3,500                                      |
| Standard_F16s_v2 | 16     | 32          | 128            | 32             | 32000 (256)                                                           | 4 / 7,000                                      |
| Standard_F32s_v2 | 32     | 64          | 256            | 32             | 64000 (512)                                                           | 8 / 14,000                                     |
| Standard_F64s_v2 | 64     | 128         | 512            | 32             | 128000 (1024)                                                         | 8 / 28,000                                     |
| Standard_F72s_v2<sup>2, 3</sup> | 72     | 144         | 576            | 32             | 144000 (1520)                                                         | 8 / 30,000                                     |

<sup>1</sup> Fsv2 系列 VM 的功能 Intel® 超執行緒技術

<sup>2</sup> 超過 64 個 vCPU 需要下列其中一個支援的客體 OS：Windows Server 2016、Ubuntu 16.04 LTS、SLES 12 SP2 和 Red Hat Enterprise Linux、CentOS 7.3 或 Oracle Linux 7.3 (含 LIS 4.2.1)

<sup>3</sup> 執行個體會隔離至單一客戶專用的硬體。

## <a name="fs-series-sup1sup"></a>Fs 系列 <sup>1</sup>

ACU：210 - 250

進階儲存體：支援

進階儲存體快取：支援

| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大資料磁碟 | 最大快取和暫存儲存體輸送量︰IOPS / MBps (以 GiB 為單位的快取大小) | 最大取消快取的磁碟輸送量︰IOPS / MBps | 最大 NIC/預期的網路頻寬 (Mbps) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_F1s |1 |2 |4 |4 |4,000 / 32 (12) |3,200 / 48 |2 / 750 |
| Standard_F2s |2 |4 |8 |8 |8,000 / 64 (24) |6,400 / 96 |2 / 1500 |
| Standard_F4s |4 |8 |16 |16 |16,000 / 128 (48) |12,800 / 192 |4 / 3000 |
| Standard_F8s |8 |16 |32 |32 |32,000 / 256 (96) |25,600 / 384 |8 / 6000 |
| Standard_F16s |16 |32 |64 |64 |64,000 / 512 (192) |51,200 / 768 |8 / 12000 |

MBps = 每秒 10^6 位元組，而 GiB = 1024^3 位元組。

<sup>1</sup> Fs 系列 VM 的最大磁碟輸送量 (IOPS 或 MBps)，可能會受到所連接磁碟的數量、大小和串接所限制。  如需詳細資訊，請參閱 [進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](../articles/virtual-machines/windows/premium-storage.md)。


<br>

## <a name="f-series"></a>F 系列

ACU：210 - 250

進階儲存體：不支援

進階儲存體快取：不支援

| 大小         | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大暫存儲存體輸送量：IOPS / 讀取 MBps / 寫入 MBps | 最大資料磁碟 / 輸送量︰IOPS | 最大 NIC/預期的網路頻寬 (Mbps) |
|--------------|-----------|-------------|----------------|----------------------------------------------------------|-----------------------------------|------------------------------|
| Standard_F1  | 1         | 2           | 16             | 3000 / 46 / 23                                           | 4 / 4x500                         | 2 / 750                 |
| Standard_F2  | 2         | 4           | 32             | 6000 / 93 / 46                                           | 8 / 8x500                         | 2 / 1500                     |
| Standard_F4  | 4         | 8           | 64             | 12000 / 187 / 93                                         | 16 / 16x500                         | 4 / 3000                     |
| Standard_F8  | 8         | 16          | 128            | 24000 / 375 / 187                                        | 32 / 32x500                       | 8 / 6000                     |
| Standard_F16 | 16        | 32          | 256            | 48000 / 750 / 375                                        | 64 / 64x500                       | 8 / 12000           |


<br>


