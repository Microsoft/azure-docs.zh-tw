---
title: SAP HANA on Azure (大型執行個體) 的概觀 | Microsoft Docs
description: 如何部署 SAP HANA on Azure (大型執行個體) 的概觀。
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
ms.openlocfilehash: c177dbad1145dee6eda3202d8076997cc7673dfc
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44026916"
---
#  <a name="what-is-sap-hana-on-azure-large-instances"></a>什麼是 SAP HANA on Azure (大型執行個體)？

SAP HANA on Azure (大型執行個體) 是 Azure 獨有的解決方案。 除了提供虛擬機器讓您部署和執行 SAP HANA 外，Azure 還可讓您在您專屬的裸機伺服器上執行並部署 SAP HANA。 SAP HANA on Azure (大型執行個體) 解決方案會建置在指派給您的非共用主機/伺服器裸機硬體上。 伺服器硬體會內嵌在更大的戳記中，該戳記會包含計算/伺服器、網路和儲存體基礎結構。 這是經過 HANA Tailored Data Center Integration (TDI) 認證的組合。 SAP HANA on Azure (大型執行個體) 提供不同的 SKU 或大小。 單位可具有 36 Intel CPU 核心和 768 GB 記憶體，並移至具有多達 480 Intel CPU 核心和多達 24 TB 記憶體的單位。

客戶在基礎結構戳記內是利用租用戶來區隔，如下所示：

- **網路**：透過每個被指派租用戶的客戶所擁有的虛擬網路，在基礎結構堆疊內區隔客戶。 一個租用戶只會指派給單一客戶。 但一個客戶可以擁有多個租用戶。 租用戶的網路區隔會在基礎結構戳記層級禁止租用戶之間的網路通訊，即使租用戶屬於相同客戶亦然。
- **儲存體元件**：透過已被指派存放磁碟區的存放虛擬機器來區隔。 存放磁碟區只能指派給一個存放虛擬機器。 一個存放虛擬機器只會指派給已通過 SAP HANA TDI 認證之基礎結構堆疊中的一個租用戶。 因此，指派給存放虛擬機器的存放磁碟區只能在一個特定且相關的租用戶中受到存取。 它們在已部署的不同租用戶之間是不可見的。
- **伺服器或主機**：伺服器或主機單位不會在客戶或租用戶之間共用。 部署給客戶的伺服器或主機是不可分割的裸機計算單位，並且只會指派給一個租用戶。 您*無法*使用任何可讓您與其他客戶共用主機或伺服器的硬體分割或軟體分割。 指派給特定租用戶之存放虛擬機器的存放磁碟區會掛接到這類伺服器。 一個租用戶可以獨佔方式獲得不同 SKU 的一個到多個伺服器單位。
- SAP HANA on Azure (大型執行個體) 的基礎結構戳記內會部署許多不同的租用戶，並透過租用戶概念在網路、儲存體和計算層級將這些租用戶彼此區隔。 


這些裸機伺服器單位只支援用來執行 SAP HANA。 SAP 應用程式層或工作負載中介軟體層會在虛擬機器中執行。 執行 SAP HANA on Azure (大型執行個體) 單位的基礎結構戳記會連線至 Azure 網路服務骨幹。 如此，即可提供 SAP HANA on Azure (大型執行個體) 單位與虛擬機器之間的低延遲連線。

在多份說明 SAP HANA on Azure (大型執行個體) 的文件中，本文件是其中之一。 本文件將介紹解決方案所提供的基本架構、責任和服務。 此外也會討論解決方案的高階功能。 其他四份文件會詳述並深入探討其他大多數的層面，例如網路和連線。 SAP HANA on Azure (大型執行個體) 的文件並未說明 SAP NetWeaver 安裝方面或在 VM 中部署 SAP NetWeaver 方面的內容。 Azure 上的 SAP NetWeaver 可在相同 Azure 說明文件容器中的個別文件中找到。 


HANA 大型執行個體指南的不同文件涵蓋下列領域：

- [Azure 上的 SAP HANA (大型執行個體) 概觀和架構](hana-overview-architecture.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Azure 上的 SAP HANA (大型執行個體) 的基礎結構和連線](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [在 Azure 上安裝和設定 SAP HANA (大型執行個體)](hana-installation.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Azure 上的 SAP HANA (大型執行個體) 高可用性和災害復原](hana-overview-high-availability-disaster-recovery.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Azure 上的 SAP HANA (大型執行個體) 疑難排解和監視](troubleshooting-monitoring.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [使用 STONITH 在 SUSE 中進行高可用性設定](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/ha-setup-with-stonith)
- [類型 II SKU 的 OS 備份和還原](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/os-backup-type-ii-skus)

**後續步驟**
- 請參閱[認識詞彙](hana-know-terms.md)