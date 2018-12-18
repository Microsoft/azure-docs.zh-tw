---
title: Azure 上的 SAP HANA (大型執行個體) 的監視 | Microsoft Docs
description: 監視 Azure 上的 SAP HANA (大型執行個體)。
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
ms.date: 09/10/2018
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e5a5d462be5555090d1dfced5fa07c9b748eb312
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44345653"
---
# <a name="how-to-monitor-sap-hana-large-instances-on-azure"></a>如何監視 Azure 上的 SAP HANA (大型執行個體)

Azure 上的 SAP HANA (大型執行個體) 與任何其他 IaaS 部署並無不同 - 您必須監視 OS 和應用程式在執行什麼作業，以及這些應用程式使用下列資源的情形：

- CPU
- 記憶體
- 網路頻寬
- 磁碟空間

使用 Azure 虛擬機器時，您必須了解上述資源類別是否足夠，或是否已耗盡。 以下提供每個這些不同類別的更多詳細資料：

**CPU 資源耗用量：** 系統會強制執行 SAP 為針對 HANA 的特定工作負載定義的比例，以確保有足夠的 CPU 資源可用來處理儲存在記憶體中的資料。 不過，可能會有因為遺漏索引或類似的問題，而導致 HANA 耗用許多執行查詢的 CPU 的情況。 這意謂著您不僅應該監視特定 HANA 服務所耗用的 CPU 資源，也應該監視 HANA 大型執行個體單位的 CPU 資源耗用量。

**記憶體耗用量：** 從 HANA 內部監視與從 HANA 外部在單位上監視一樣重要。 在 HANA 內，您可以監視資料如何耗用 HANA 已配置的記憶體，以便保持在 SAP 所要求的大小指導方針限制內。 您也可以監視「大型執行個體」層級的記憶體耗用量，以確保所安裝的額外非 HANA 軟體不會耗用太多記憶體而與 HANA 爭用記憶體。

**網路頻寬：** Azure VNet 閘道對於將資料移入 Azure VNet 的頻寬方面有所限制，因此監視 VNet 內所有 Azure VM 所接收的資料相當有用，可了解您有多接近所選 Azure 閘道 SKU 的限制。 在「HANA 大型執行個體」單位上，監視連入和連出網路流量，並隨著時間記錄所處理的磁碟區，是有意義的。

**磁碟空間：** 磁碟空間耗用量通常會隨著時間增加。 最常見的原因包括：資料量增加、執行交易記錄備份、儲存追蹤檔案，以及執行儲存體快照集。 因此，監視磁碟空間使用量並管理與「HANA 大型執行個體」單位關聯的磁碟空間相當重要。

如果是**類型 II SKU** 的 HANA 大型執行個體，伺服器會隨附預先載入的系統診斷工具。 您可以利用這些診斷工具執行系統健康情況檢查。 執行下列命令可在 /var/log/health_check 產生健康情況檢查記錄檔。
```
/opt/sgi/health_check/microsoft_tdi.sh
```
當您透過 Microsoft 支援服務小組對問題進行疑難排解時，可能也會要求您使用這些診斷工具提供記錄檔。 您可以使用下列命令壓縮檔案。
```
tar  -czvf health_check_logs.tar.gz /var/log/health_check
```

**後續步驟**

- 請參閱[如何監視 Azure 上的 SAP HANA (大型執行個體)](troubleshooting-monitoring.md)。