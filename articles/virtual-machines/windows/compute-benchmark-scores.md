---
title: Azure Windows VM 的計算基準測試分數 | Microsoft Docs
description: 比較執行 Windows Server 之 Azure VM 的 SPECint 計算基準測試分數。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: 69ae72ec-e8be-4e46-a8f0-e744aebb5cc2
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 04/09/2018
ms.author: cynthn;davberg
ms.openlocfilehash: a8d071544462361e9750d3fa622467cd0000a040
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39056778"
---
# <a name="compute-benchmark-scores-for-windows-vms"></a>Windows VM 的計算基準測試分數
下列 SPECInt 基準測試分數顯示執行 Windows Server 的 Azure 高效能 VM 產品陣容的計算效能。 也有 [Linux VM](../linux/compute-benchmark-scores.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)的計算基準測試分數。 

> [!NOTE]
> Linux 數字最近已更新，並包含一組更完整的 VM。

## <a name="a-series---compute-intensive"></a>A 系列 - 計算密集型
| 大小 | vCPU | NUMA 節點 | CPU | 執行 | 平均基本費率 | 標準差 |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_A8 |8 |1 |Intel Xeon CPU E5-2670 0 \@ 2.6 GHz |10 |236.1 |1.1 |
| Standard_A9 |16 |2 |Intel Xeon CPU E5-2670 0 \@ 2.6 GHz |10 |450.3 |7.0 |
| Standard_A10 |8 |1 |Intel Xeon CPU E5-2670 0 \@ 2.6 GHz |5 |235.6 |0.9 |
| Standard_A11 |16 |2 |Intel Xeon CPU E5-2670 0 \@ 2.6 GHz |7 |454.7 |4.8 |

## <a name="dv2-series"></a>Dv2 系列
| 大小 | vCPU | NUMA 節點 | CPU | 執行 | 平均基本費率 | 標準差 |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_D1_v2 |1 |1 |Intel Xeon E5-2673 v3 \@ 2.4 GHz |83 |36.6 |2.6 |
| Standard_D2_v2 |2 |1 |Intel Xeon E5-2673 v3 \@ 2.4 GHz |27 |70.0 |3.7 |
| Standard_D3_v2 |4 |1 |Intel Xeon E5-2673 v3 \@ 2.4 GHz |19 |130.5 |4.4 |
| Standard_D4_v2 |8 |1 |Intel Xeon E5-2673 v3 \@ 2.4 GHz |19 |238.1 |5.2 |
| Standard_D5_v2 |16 |2 |Intel Xeon E5-2673 v3 \@ 2.4 GHz |14 |460.9 |15.4 |
| Standard_D11_v2 |2 |1 |Intel Xeon E5-2673 v3 \@ 2.4 GHz |19 |70.1 |3.7 |
| Standard_D12_v2 |4 |1 |Intel Xeon E5-2673 v3 \@ 2.4 GHz |2 |132.0 |1.4 |
| Standard_D13_v2 |8 |1 |Intel Xeon E5-2673 v3 \@ 2.4 GHz |17 |235.8 |3.8 |
| Standard_D14_v2 |16 |2 |Intel Xeon E5-2673 v3 \@ 2.4 GHz |15 |460.8 |6.5 |

## <a name="g-series-gs-series"></a>G 系列、GS 系列
| 大小 | vCPU | NUMA 節點 | CPU | 執行 | 平均基本費率 | 標準差 |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_G1、Standard_GS1 |2 |1 |Intel Xeon E5-2698B v3 \@ 2 GHz |31 |71.8 |6.5 |
| Standard_G2、Standard_GS2 |4 |1 |Intel Xeon E5-2698B v3 \@ 2 GHz |5 |133.4 |13.0 |
| Standard_G3、Standard_GS3 |8 |1 |Intel Xeon E5-2698B v3 \@ 2 GHz |6 |242.3 |6.0 |
| Standard_G4、Standard_GS4 |16 |1 |Intel Xeon E5-2698B v3 \@ 2 GHz |15 |398.9 |6.0 |
| Standard_G5、Standard_GS5 |32 |2 |Intel Xeon E5-2698B v3 \@ 2 GHz |22 |762.8 |3.7 |

## <a name="h-series"></a>H 系列
| 大小 | vCPU | NUMA 節點 | CPU | 執行 | 平均基本費率  | 標準差 |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_H8 |8 |1 |Intel Xeon E5-2667 v3 \@ 3.2 GHz |5 |297.4 |0.9 |
| Standard_H16 |16 |2 |Intel Xeon E5-2667 v3 \@ 3.2 GHz |5 |575.8 |6.8 |
| Standard_H8m |8 |1 |Intel Xeon E5-2667 v3 \@ 3.2 GHz |5 |297.0 |1.2 |
| Standard_H16m |16 |2 |Intel Xeon E5-2667 v3 \@ 3.2 GHz |5 |572.2 |3.9 |
| Standard_H16r |16 |2 |Intel Xeon E5-2667 v3 \@ 3.2 GHz |5 |573.2 |2.9 |
| Standard_H16mr |16 |2 |Intel Xeon E5-2667 v3 \@ 3.2 GHz |7 |569.6 |2.8 |

## <a name="about-specint"></a>關於 SPECint
Windows 數字是藉由在 Windows Server 上執行 [SPECint 2006](https://www.spec.org/cpu2006/results/rint2006.html) 來計算出。 執行 SPECint 時是使用基本費率選項 (SPECint_rate2006)，其中每個 vCPU 有一個複本。 SPECint 是由 12 個個別的測試組成，每個測試都執行三次，系統會從每個測試挑出中間值，然後將它們加權來形成複合分數。 這些測試會接著在多個 VM 上執行以提供所顯示的平均分數。

## <a name="next-steps"></a>後續步驟
* 如需了解儲存體容量、磁碟詳細資料及其他選擇 VM 大小的考量，請參閱 [虛擬機器的大小](sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

