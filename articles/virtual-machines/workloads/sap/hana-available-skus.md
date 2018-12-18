---
title: SAP HANA on Azure (大型執行個體) 適用的 SKU | Microsoft Docs
description: SAP HANA on Azure (大型執行個體) 適用的 SKU。
services: virtual-machines-linux
documentationcenter: ''
author: RicksterCDN
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/04/2018
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7ed7291333e7baa764b09e66aa5cfaedc77072fb
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44030233"
---
# <a name="available-skus-for-hli"></a>HLI 可用的 SKU

SAP Hana on Azure (大型執行個體) 服務已可用於美國西部和美國東部、澳大利亞東部、澳大利亞東南部、西歐、北歐、日本東部及日本西部等 Azure 區域中的數個組態。

[HANA 大型執行個體的 SAP Hana 認證 SKU](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) 清單如下：

| SAP 解決方案 | CPU | 記憶體 | 儲存體 | 可用性 |
| --- | --- | --- | --- | --- |
| 已針對 OLAP 最佳化：SAP BW、BW/4HANA<br /> 或 SAP HANA (一般 OLAP 工作負載) | SAP HANA on Azure S72<br /> – 2 x Intel® Xeon® Processor E7-8890 v3<br /> 36 個 CPU 核心和 72 個 CPU 執行緒 |  768 GB |  3 TB | 可用 |
| --- | SAP HANA on Azure S144<br /> – 4 x Intel® Xeon® Processor E7-8890 v3<br /> 72 個 CPU 核心和 144 個 CPU 執行緒 |  1.5 TB |  6 TB | 不再提供 |
| --- | SAP HANA on Azure S192<br /> – 4 x Intel® Xeon® Processor E7-8890 v4<br /> 96 個 CPU 核心和 192 個 CPU 執行緒 |  2.0 TB |  8 TB | 可用 |
| --- | SAP HANA on Azure S384<br /> – 8 x Intel® Xeon® Processor E7-8890 v4<br /> 192 個 CPU 核心和 384 個 CPU 執行緒 |  4.0 TB |  16 TB | 可用 |
| 已針對 OLTP 最佳化：SAP Business Suite<br /> on SAP HANA 或 S/4HANA (OLTP)<br /> (一般 OLTP) | SAP HANA on Azure S72m<br /> – 2 x Intel® Xeon® Processor E7-8890 v3<br /> 36 個 CPU 核心和 72 個 CPU 執行緒 |  1.5 TB |  6 TB | 可用 |
|---| SAP HANA on Azure S144m<br /> – 4 x Intel® Xeon® Processor E7-8890 v3<br /> 72 個 CPU 核心和 144 個 CPU 執行緒 |  3.0 TB |  12 TB | 不再提供 |
|---| SAP HANA on Azure S192m<br /> – 4 x Intel® Xeon® Processor E7-8890 v4<br /> 96 個 CPU 核心和 192 個 CPU 執行緒  |  4.0 TB |  16 TB | 可用 |
|---| SAP HANA on Azure S384m<br /> – 8 x Intel® Xeon® Processor E7-8890 v4<br /> 192 個 CPU 核心和 384 個 CPU 執行緒 |  6.0 TB |  18 TB | 可用 |
|---| SAP HANA on Azure S384xm<br /> – 8 x Intel® Xeon® Processor E7-8890 v4<br /> 192 個 CPU 核心和 384 個 CPU 執行緒 |  8.0 TB |  22 TB |  可用 |
|---| SAP HANA on Azure S576m<br /> – 12 x Intel® Xeon® Processor E7-8890 v4<br /> 288 個 CPU 核心和 576 個 CPU 執行緒 |  12.0 TB |  28 TB | 可用 |
|---| SAP HANA on Azure S768m<br /> – 16 x Intel® Xeon® Processor E7-8890 v4<br /> 384 個 CPU 核心和 768 個 CPU 執行緒 |  16.0 TB |  36 TB | 可用 |
|---| SAP HANA on Azure S960m<br /> – 20 x Intel® Xeon® Processor E7-8890 v4<br /> 480 個 CPU 核心和 960 個 CPU 執行緒 |  20.0 TB |  46 TB | 可用 |


在 SAP Hana TDIv5 下，SAP 允許客戶專屬的大小調整及客戶專屬專案，可能會導致以下位置的伺服器設定未列為已認證：

- [SAP Hana 認證設備](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/appliances.html)
- [SAP Hana 認證 IaaS 平台](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure)

在許多情況下，這些客戶專屬的伺服器設定會比以 SAP 認證的伺服器單位執行更多記憶體。 使用 SAP 時，客戶可能會取得 SAP 支援，並為其客戶專屬大小的伺服器設定進行認證。 下列 HANA 大型執行個體標準 SKU 可於 Azure 中使用，且這類 TDIv5 客戶專屬的大小調整專案可在 Microsoft 價格清單中。


| 原始 SKU 可以 <br /> 在記憶體中擴充 | CPU | 記憶體 | 儲存體 | 可用性 |
| --- | --- | --- | --- | --- |
| S192m 可擴充到 | SAP HANA on Azure S192xm<br /> – 4 x Intel® Xeon® Processor E7-8890 v4<br /> 96 個 CPU 核心和 192 個 CPU 執行緒 |  6.0 TB |  16 TB | 可用 |
| S384xm 可擴充到 | SAP HANA on Azure S384xxm<br /> – 8 x Intel® Xeon® Processor E7-8890 v4<br /> 192 個 CPU 核心和 384 個 CPU 執行緒 |  12.0 TB |  28 TB | 可用 |
| S576m 可擴充到 | SAP HANA on Azure S576xm<br /> – 12 x Intel® Xeon® Processor E7-8890 v4<br /> 288 個 CPU 核心和 576 個 CPU 執行緒 |  18.0 TB |  41 TB | 可用 |
| S768m 可擴充到 | SAP HANA on Azure S768xm<br /> – 16 x Intel® Xeon® Processor E7-8890 v4<br /> 384 個 CPU 核心和 768 個 CPU 執行緒 |  24.0 TB |  56 TB | 可用 |

- CPU 核心 = 伺服器單位處理器總和的非超執行緒 CPU 核心總和。
- CPU 執行緒 = 伺服器單位處理器總和的超執行緒 CPU 核心所提供之計算執行緒的總和。 依預設會將大部分的單位設定為使用超執行緒技術。
- 根據供應商建議，S768m、S768xm 和 S960m 並未設定可使用超執行緒來執行 SAP Hana。


選擇的特定組態取決於工作負載、CPU 資源及所需的記憶體。 OLTP 工作負載可以使用已針對 OLAP 工作負載最佳化的 SKU。 

供應項目的硬體基底 (除了客戶專屬的大小調整專案單位) 已通過 SAP Hana TDI 認證。 兩種不同類別的硬體會將 SKU 分成：

- S72、S72m、S144、S144m、S192、S192m 和 S192xm，稱之為「類型 I 類別」的 SKU。
- S384、S384m、S384xm、S384xxm、S576m、S576xm、S768m、S768xm 和 S960m，稱之為「類型 II 類別」的 SKU。

完整的 HANA 大型執行個體戳記並非僅配置給單一客戶使用。 這項事實也適用於透過 Azure 中所部署的網路網狀架構連線的計算與儲存資源的機架。 「HANA 大型執行個體」基礎結構和 Azure 一樣，會在下列三個層面部署彼此隔離的不同客戶&quot;租用戶&quot;：

- **網路**：透過虛擬網路在 HANA 大型執行個體戳記內區隔。
- **儲存體**：透過已被指派存放磁碟區的存放虛擬機器和租用戶之間的獨立存放磁碟區來區隔。
- **計算**：將伺服器單位指派給單一租用戶專用。 不對伺服器單位進行硬體分割或軟體分割。 不在租用戶之間共用單一伺服器或主機單位。 

不同租用戶之間的 HANA 大型執行個體單位部署不會讓彼此看到。 部署在不同租用戶的 HANA 大型執行個體單位無法在 HANA 大型執行個體戳記層級上直接與其他單位進行通訊。 只有同一個租用戶內的 HANA 大型執行個體單位可以在 HANA 大型執行個體戳記層級上與其他單位進行通訊。

為了進行計費，「大型執行個體」戳記中已部署的租用戶會指派給一個 Azure 訂用帳戶。 就網路而言，從同一個 Azure 註冊內的其他 Azure 訂用帳戶的虛擬網路加以存取，是可行的。 如果您使用同一個 Azure 區域中的另一個 Azure 訂用帳戶來部署，則也可以選擇尋求個別的 HANA 大型執行個體租用戶。

在「HANA 大型執行個體」上執行 SAP HANA，與在部署於 Azure 中的 VM 上執行 SAP HANA，有顯著的不同：

- SAP HANA on Azure (大型執行個體) 沒有虛擬化層。 您會獲得基礎裸機硬體的效能。
- 與 Azure 不同，SAP HANA on Azure (大型執行個體) 伺服器是特定客戶專用。 您無法對伺服器單位或主機進行硬體分割或軟體分割。 因此，HANA 大型執行個體單位會整個指派給租用戶，並整體指派給您。 重新啟動或關閉伺服器並不會自動導致作業系統和 SAP HANA 被部署到另一部伺服器。 (針對類型 I 類別的 SKU，唯一的例外狀況，是如果伺服器發生問題而必須在另一部伺服器上執行重新部署的情況)。
- 與 Azure 中配合最佳性價比來選取主機處理器類型的方式不同，為 SAP HANA on Azure (大型執行個體) 選擇的處理器類型是 Intel E7v3 和 E7v4 處理器系列中效能最佳的類型。

**後續步驟**
- 請參閱 [HLI 大小調整](hana-sizing.md)
