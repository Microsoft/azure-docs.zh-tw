---
title: SAP HANA on Azure (大型執行個體) 的認證 | Microsoft Docs
description: SAP HANA on Azure (大型執行個體) 的認證。
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
ms.openlocfilehash: 1346018d99a1c26290ad8928d9794f3b55050910
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44030231"
---
# <a name="certification"></a>認證

除了 NetWeaver 認證之外，SAP 還需要特殊的 SAP HANA 認證，才能在特定的基礎結構 (例如 Azure IaaS) 上支援 SAP HANA。

NetWeaver 和 SAP HANA 認證 (就某種程度而言) 的核心 SAP 附註是 [SAP 附註 #1928533 - Azure 上的 SAP 應用程式︰支援的產品和 Azure VM 類型](https://launchpad.support.sap.com/#/notes/1928533)。

SAP Hana on Azure (大型執行個體) 單位的憑證記錄位於 [SAP HANA 認證 IaaS 平台](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure)網站中。 

SAP Hana 認證 IaaS 平台網站中提及的 SAP HANA on Azure (大型執行個體) 類型可讓 Microsoft 和 SAP 客戶能夠在 Azure 中部署大型 SAP Business Suite、SAP BW、S/4 HANA、BW/4HANA 或其他 SAP HANA 工作負載。 此解決方案是以經 SAP-HANA 認證的專用硬體戳記 ([SAP HANA tailored Data Center Integration - TDI](https://scn.sap.com/docs/DOC-63140)) 為基礎。 如果您執行 SAP HANA TDI 設定的解決方案，則所有的 SAP HANA 型應用程式 (例如 SAP Business Suite on SAP HANA、SAP BW on SAP HANA、S4/HANA 及 BW4/HANA) 都能在硬體基礎結構上運作。

相較於在 VM 中執行 SAP HANA，此解決方案有其優點。 它可提供大得多的記憶體磁碟區。 若要啟用此解決方案，您必須了解下列重要層面：

- SAP 應用程式層和非 SAP 應用程式是在裝載於一般 Azure 硬體戳記中的 VM 上執行。
- 客戶內部部署基礎結構、資料中心及應用程式部署都會透過 ExpressRoute (建議使用) 或虛擬私人網路 (VPN) 連線至雲端平台。 Active Directory 和 DNS 也會擴充到 Azure 中。
- 用於 HANA 工作負載的 SAP HANA 資料庫執行個體會在 SAP HANA on Azure (大型執行個體) 上執行。 「大型執行個體」戳記會連線至 Azure 網路中，讓在 VM 中執行的軟體能夠與在「HANA 大型執行個體」中執行的 HANA 執行個體互動。
- SAP HANA on Azure (大型執行個體) 的硬體是已預先安裝 SUSE Linux Enterprise Server 或 Red Hat Enterprise Linux 的 IaaS 中所提供的專用硬體。 和使用虛擬機器一樣，進一步的作業系統更新和維護也是由您負責。
- 安裝 HANA 或任何在 HANA 大型執行個體的單位上執行 SAP HANA 所需的額外元件，是您的負責。 您也須負責執行 SAP HANA on Azure 的所有個別進行中作業和管理。
- 除了這裡所述的解決方案之外，您也可以在連接到 SAP HANA on Azure (大型執行個體) 的 Azure 訂用帳戶中安裝其他元件。 範例包括可用來與 SAP HANA 資料庫通訊或直接通訊的元件，例如跳板伺服器、RDP 伺服器、SAP HANA Studio、適用於 SAP BI 案例的 SAP Data Services，或網路監視解決方案。
- 和在 Azure 中一樣，HANA 大型執行個體也可支援高可用性和災害復原功能。

**後續步驟**
- 請參閱[HLI 可用的 SKU](hana-available-skus.md) 