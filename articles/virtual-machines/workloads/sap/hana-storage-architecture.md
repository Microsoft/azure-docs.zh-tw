---
title: SAP HANA on Azure (大型執行個體) 的儲存體架構 | Microsoft Docs
description: 如何部署 SAP HANA on Azure (大型執行個體) 的儲存體架構。
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
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 614d6aef4a2b7be551574fd3c8e25e2a3e3c1c07
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44030512"
---
# <a name="sap-hana-large-instances-storage-architecture"></a>SAP HANA (大型執行個體) 儲存體架構

SAP HANA on Azure (大型執行個體) 的儲存體配置是由傳統部署模型透過 SAP 建議的指引進行設定。 這些指引列載於 [SAP HANA 儲存體需求](http://go.sap.com/documents/2015/03/74cdb554-5a7c-0010-82c7-eda71af511fa.html)白皮書中。

類型 I 類別的 HANA 大型執行個體通常隨附存放磁碟區四倍的記憶體磁碟區。 針對類型 II 類別的 HANA 大型執行個體單位，存放裝置將不會超過四倍。 這些單位會隨附一個用來儲存 HANA 交易記錄備份的磁碟區。 如需詳細資訊，請參閱[在 Azure 上安裝和設定 SAP HANA (大型執行個體)](hana-installation.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

關於儲存區配置，請參閱下表。 該表格列出了利用不同 HANA 大型執行個體單位提供之不同磁碟區的粗估容量。

| HANA 大型執行個體 SKU | hana/data | hana/log | hana/shared | hana/logbackups |
| --- | --- | --- | --- | --- |
| S72 | 1,280 GB | 512 GB | 768 GB | 512 GB |
| S72m | 3,328 GB | 768 GB |1,280 GB | 768 GB |
| S192 | 4,608 GB | 1,024 GB | 1,536 GB | 1,024 GB |
| S192m | 11,520 GB | 1,536 GB | 1,792 GB | 1,536 GB |
| S192xm |  11,520 GB |  1,536 GB |  1,792 GB |  1,536 GB |
| S384 | 11,520 GB | 1,536 GB | 1,792 GB | 1,536 GB |
| S384m | 12,000 GB | 2,050 GB | 2,050 GB | 2,040 GB |
| S384xm | 16,000 GB | 2,050 GB | 2,050 GB | 2,040 GB |
| S384xxm |  20,000 GB | 3,100 GB | 2,050 GB | 3,100 GB |
| S576m | 20,000 GB | 3,100 GB | 2,050 GB | 3,100 GB |
| S576xm | 31,744 GB | 4,096 GB | 2,048 GB | 4,096 GB |
| S768m | 28,000 GB | 3,100 GB | 2,050 GB | 3,100 GB |
| S768xm | 40,960 GB | 6,144 GB | 4,096 GB | 6,144 GB |
| S960m | 36,000 GB | 4,100 GB | 2,050 GB | 4,100 GB |


實際部署的磁碟區可能會因為部署和用來顯示磁碟區大小的工具而不同。

如果您要細分 HANA 大型執行個體 SKU，以下提供一些可能的劃分片段範例：

| 記憶體磁碟分割，以 GB 為單位 | hana/data | hana/log | hana/shared | hana/log/backup |
| --- | --- | --- | --- | --- |
| 256 | 400 GB | 160 GB | 304 GB | 160 GB |
| 512 | 768 GB | 384 GB | 512 GB | 384 GB |
| 768 | 1,280 GB | 512 GB | 768 GB | 512 GB |
| 1,024 | 1,792 GB | 640 GB | 1,024 GB | 640 GB |
| 1,536 | 3,328 GB | 768 GB | 1,280 GB | 768 GB |


這些大小是粗略的磁碟區數目，會根據部署和用來查看磁碟區的工具而有些微不同。 此外還有其他的磁碟分割大小，例如 2.5 TB。 這些儲存體大小會使用上述磁碟分割所使用的類似公式來計算。 「磁碟分割」一詞不表示作業系統、記憶體或 CPU 資源會以任何方式進行磁碟分割。 它表示適用於您可能想要在單一 HANA 執行個體單位上部署之不同 HANA 執行個體的儲存體磁碟分割。 

您可能需要更多儲存體。 您可以購買額外的儲存體 (以 1 TB 為單位)，以增加儲存體。 此額外儲存體可新增為額外的磁碟區。 它也可用來擴充一或多個現有磁碟區。 您無法將磁碟區的大小降低為原始部署的大小以及上述表格中多數記載的大小。 您也無法變更磁碟區的名稱或掛接的名稱。 上述的存放磁碟區已連接至 HANA 大型執行個體單位以作為 NFS4 磁碟區。

您可以使用儲存體快照來進行備份和還原以及災害復原。 如須詳細資訊，請參閱 [Azure 上的 SAP Hana (大型執行個體) 高可用性和災害復原](hana-overview-high-availability-disaster-recovery.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

如需適用於您的案例的儲存配置詳細資料，請參閱 [HLI 支援案例](hana-supported-scenario.md)。

## <a name="run-multiple-sap-hana-instances-on-one-hana-large-instance-unit"></a>在一個 HANA 大型執行個體單位上執行多個 SAP HANA 執行個體

您可以在 HANA 大型執行個體單位上，裝載多個使用中的 SAP HANA 執行個體。 若要提供儲存體快照集和災害復原等功能，這類設定在每個執行個體上都需要一個磁碟區組。 目前可使用下列方式來細分 HANA 大型執行個體單位：

- **S72、S72m、S144、S192**：遞增量為 256 GB，最小起始單位為 256 GB。 不同的遞增量 (例如 256 GB 和 512 GB) 可組合為該單位的記憶體最大值。
- **S144m 和 S192m**：遞增量為 256 GB，最小單位為 512 GB。 不同的遞增量 (例如 512 GB 和 768 GB) 可組合為該單位的記憶體最大值。
- **類型 II 類別**：遞增量為 512 GB，最小起始單位為 2 TB。 不同的遞增量 (例如 512 GB、1 TB 和 1.5 TB) 可組合為該單位的記憶體最大值。

執行多個 SAP HANA 執行個體的範例可能顯示如下。

| SKU | 記憶體大小 | 儲存體大小 | 具有多個資料庫的大小 |
| --- | --- | --- | --- |
| S72 | 768 GB | 3 TB | 1x768 GB HANA 執行個體<br /> 或 1x512 GB 執行個體 + 1x256 GB 執行個體<br /> 或 3x256 GB 執行個體 | 
| S72m | 1.5 TB | 6 TB | 3x512GB HANA 執行個體<br />或 1x512 GB 執行個體 + 1x1 TB 執行個體<br />或 6x256 GB 執行個體<br />或 1x1.5 TB 執行個體 | 
| S192m | 4 TB | 16 TB | 8x512 GB 執行個體<br />或 4x1 TB 執行個體<br />或 4x512 GB 執行個體 + 2x1 TB 執行個體<br />或 4x768 GB 執行個體 + 2x512 GB 執行個體<br />或 1x4 TB 執行個體 |
| S384xm | 8 TB | 22 TB | 4x2 TB 執行個體<br />或 2x4 TB 執行個體<br />或 2x3 TB 執行個體 + 1x2 TB 執行個體<br />或 2x2.5 TB 執行個體 + 1x3 TB 執行個體<br />或 1x8 TB 執行個體 |


此外還有其他變化形式。 

## <a name="encryption-of-data-at-rest"></a>待用資料加密
用於「HANA 大型執行個體」的儲存體可允許在資料儲存於磁碟上時，對其進行透明加密。 部署 HANA 大型執行個體單位時，您可以啟用這種加密。 您也可以在部署就緒變更為加密磁碟區。 從非加密到已加密磁碟區的轉移過程是透明的，並不需要停機。 

利用類型 I 類別的 SKU，就會將儲存開機 LUN 的磁碟區加密。 對於類型 II 類別的 HANA 大型執行個體 SKU，您需要使用 OS 方法來將開機 LUN 加密。 如需詳細資訊，請連絡 Microsoft 服務管理小組。

**後續步驟**
- 請參閱 [HANA 大型執行個體支援的案例](hana-supported-scenario.md)