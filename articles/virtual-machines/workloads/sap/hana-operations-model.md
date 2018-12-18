---
title: SAP HANA on Azure (大型執行個體) 的作業模型 | Microsoft Docs
description: SAP HANA on Azure (大型執行個體) 的作業模型。
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
ms.openlocfilehash: 776057a177904d086974258b5221ab474f4b1bcb
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44030221"
---
# <a name="operations-model-and-responsibilities"></a>作業模型和職責

SAP HANA on Azure (大型執行個體) 提供的服務可與 Azure IaaS 服務合作。 您會獲得一個「HANA 大型執行個體」執行個體，其中已安裝針對 SAP HANA 最佳化的作業系統。 就像使用 Azure IaaS VM 時一樣，大多數強化 OS、安裝其他軟體、安裝 HANA、操作 OS 和 HANA，以及更新 OS 和 HANA 的工作，都是由您負責。 Microsoft 不會強制您進行 OS 更新或 HANA 更新。

![SAP HANA on Azure (大型執行個體) 的職責](./media/hana-overview-architecture/image2-responsibilities.png)

如圖所示，SAP HANA on Azure (大型執行個體) 是一個多租用戶的 IaaS 供應項目。 大部分的職責劃分皆以 OS 基礎結構為界。 Microsoft 負責作業系統線以下的所有服務層面。 您則負責該線以上的各個服務層面。 作業系統是您的責任。 您可以繼續使用目前針對合規性、安全性、應用程式管理、基礎與 OS 管理可能採用的大多數內部部署方法。 這些系統在所有方面都會顯得它們彷彿就在您的網路中一樣。

這項服務已針對 SAP HANA 進行最佳化，因此有些部分您需要與 Microsoft 合作，才能使用根本基礎結構功能獲得最佳結果。

以下清單提供有關每一層及您職責的更多詳細資料：

**網路**：執行 SAP HANA 的大型執行個體戳記的所有內部網路。 您的責任包括對儲存體的存取、執行個體之間的連線 (用於相應放大及其他功能)、對架構的連線，以及連線至在 VM 中裝載 SAP 應用程式層的 Azure。 它也包含用於災害復原用途複寫的「Azure 資料中心」之間的 WAN 連線。 所有網路都會依租用戶分割，並已套用服務品質。

**儲存體**：除了用於快照的虛擬化分割儲存體之外，也包括用於 SAP HANA 伺服器所需之所有磁碟區的虛擬化分割儲存體。 

**伺服器**：用以執行指派給租用戶之 SAP HANA DB 的專用實體伺服器。 類型 I 類別 SKU 的伺服器是抽象的硬體。 使用這些類型的伺服器，即會在設定檔中收集和維護伺服器設定，而您可以將其從一個實體硬體移至另一個實體硬體。 透過操作對設定檔進行的這類 (手動) 移動，與 Azure 服務修復有一點類似。 類型 II 類別 SKU 的伺服器不提供這類功能。

**SDDC**：可將資料中心當作一個軟體定義的實體來管理的管理軟體。 它可讓 Microsoft 基於延展、可用性及效能等理由將資源集中存放。

**O/S**：您所選擇在伺服器上執行的 OS (SUSE Linux 或 Red Hat Linux)。 提供給您的 OS 映像，是個別 Linux 廠商提供給 Microsoft 執行 SAP HANA 的映像。 您必須有 Linux 廠商的訂用帳戶，才能取得特定的 SAP HANA 最佳化映像。 向 OS 廠商註冊映像是您的責任。 

自 Microsoft 交付時開始，您即須負責對 Linux 作業系統採取進一步的修補措施。 這項修補包括成功安裝 SAP HANA 可能需要、而特定 Linux 廠商的 SAP HANA 最佳化 OS 映像並未包含的其他套件。 (如需詳細資訊，請參閱 SAP 的 HANA 安裝文件和 SAP 附註)。 

您必須負責執行 OS 的運作問題或最佳化所需的 OS 修補，以及與特定伺服器硬體有關的驅動程式。 OS 的任何安全性或功能性修補，也是您的責任。 

您的責任還包括下列項目的監視和容量規劃︰

- CPU 資源耗用量。
- 記憶體耗用量。
- 與可用空間、IOPS 及延遲相關的磁碟區。
- 「HANA 大型執行個體」與 SAP 應用程式層之間的網路磁碟區流量。

「HANA 大型執行個體」的根本基礎結構提供備份和還原 OS 磁碟區的功能。 使用這項功能也是您的職責所在。

**中介軟體**：主要是「SAP HANA 執行個體」。 管理、操作及監視是您的職責。 您可以利用隨附的功能，使用儲存體快照來進行備份和還原以及災害復原。 這些功能是由基礎結構所提供的。 您的職責還包括使用這些功能來設計高可用性或災害復原、利用它們，以及監視並判斷儲存體快照集是否已順利執行。

**資料**：您受 SAP HANA 管理的資料，以及其他位於磁碟區或檔案共用上的資料 (例如備份檔案)。 您的職責包括監視磁碟可用空間和管理磁碟區上的內容。 您也必須負責監視磁碟區和儲存體快照的備份是否已順利執行。 將資料順利複寫到災害復原網站則是 Microsoft 的職責。

**應用程式**：SAP 應用程式執行個體，或就非 SAP 應用程式而言，則是這些應用程式的應用程式層。 您的責任包括部署、管理、操作及監視這些應用程式。 您必須負責對 CPU 資源耗用量、記憶體耗用量、Azure 儲存體耗用量和虛擬網路內的網路頻寬耗用量進行容量規劃。 您也須負責對資源耗用量進行容量規劃，舉凡虛擬網路到 SAP HANA on Azure (大型執行個體)，皆包含在內。

**WAN**：您為工作負載建立的從內部部署到 Azure 部署的連線。 所有具有 HANA 大型執行個體的客戶都會使用 Azure ExpressRoute 連線。 此連線不是 SAP HANA on Azure (大型執行個體) 解決方案的一部分。 您必須負責此連線的設定。

**封存**：您可能會偏好使用自己的方法在儲存體帳戶中封存資料複本。 封存需要管理、合規性、成本及操作。 您須負責在 Azure 上產生封存複本和備份，並以符合規範的方式加以儲存。

請參閱 [SAP HANA on Azure (大型執行個體) SLA](https://azure.microsoft.com/support/legal/sla/sap-hana-large/v1_0/)。

**後續步驟**
- 請參閱 [SAP HANA on Azure (大型執行個體) 架構](hana-architecture.md)