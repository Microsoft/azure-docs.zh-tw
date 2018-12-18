---
title: SAP HANA on Azure (大型執行個體) 的上線需求 | Microsoft Docs
description: SAP HANA on Azure (大型執行個體) 的上線需求。
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
ms.openlocfilehash: d287f79f4161c509f96b679d4b794c2906f2e020
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44030224"
---
# <a name="onboarding-requirements"></a>上線需求

此清單彙編了執行 SAP HANA on Azure (大型執行個體) 的需求。

**Microsoft Azure**

- 可連結到 SAP HANA on Azure (大型執行個體) 的 Azure 訂用帳戶。
- Microsoft 頂級支援合約。 如需有關在 Azure 中執行 SAP 的特定資訊，請參閱 [SAP 支援附註 #2015553 - Microsoft Azure 上的 SAP：支援的必要條件](https://launchpad.support.sap.com/#/notes/2015553)。 如果您使用 HANA 大型執行個體單位搭配 384 和更多 CPU，您也需要擴充頂級支援合約以包含 Azure 快速回應。
- 在執行 SAP 大小調整演練之後，知道您所需的 HANA 大型執行個體 SKU。

**網路連線**

- 內部部署與 Azure 之間的 ExpressRoute：若要將您的內部部署資料中心連線到 Azure，請務必向您的 ISP 訂購一個至少 1 Gbps 的連線。 

**作業系統**

- SUSE Linux Enterprise Server 12 for SAP Applications 的授權。

   > [!NOTE] 
   > Microsoft 提供的作業系統未向 SUSE 註冊。 它不會連線至 Subscription Management Tool 執行個體。

- 在 Azure 中部署於 VM 上的 SUSE Linux Subscription Management Tool。 此工具提供可讓 SUSE 註冊 SAP HANA on Azure (大型執行個體) 並對其分別進行更新的功能。 (在「HANA 大型執行個體」資料中心內無法存取網際網路)。 
- Red Hat Enterprise Linux for SAP Hana 6.7 或 7.x 的授權。

   > [!NOTE]
   > Microsoft 提供的作業系統未向 Red Hat 註冊。 它不會連線至 Red Hat Subscription Manager 執行個體。

- 在 Azure 中部署於 VM 上的 Red Hat Subscription Manager。 Red Hat Subscription Manager 提供可讓 Red Hat 註冊 SAP HANA on Azure (大型執行個體) 並對其分別進行更新的功能。 (在 Azure 大型執行個體戳記上部署的租用戶內無法進行直接的網際網路存取)。
- SAP 會要求您與 Linux 提供者之間有一份支援合約。 這項需求並不會因「HANA 大型執行個體」解決方案或您在 Azure 中執行 Linux 的事實而消除。 與使用一些 Linux Azure 資源庫映像時不同，服務費用*並未*包含在「HANA 大型執行個體」的解決方案內容中。 您必須負責達到 SAP 對 Linux 散發者之支援合約的相關需求。 
   - 針對 SUSE Linux，請查閱 [SAP 附註 #1984787 - SUSE Linux Enterprise Server 12：安裝注意事項](https://launchpad.support.sap.com/#/notes/1984787)和 [SAP 附註 #1056161 - SAP 應用程式的 SUSE 優先支援](https://launchpad.support.sap.com/#/notes/1056161)中的支援合約需求。
   - 針對 Red Hat Linux，您必須擁有包含支援和服務 (HANA 大型執行個體的作業系統更新) 的正確訂用帳戶層級。 Red Hat 建議使用 SAP 解決方案適用的 Red Hat Enterprise Linux 訂用帳戶。 請參閱 https://access.redhat.com/solutions/3082481。 

如需具有不同 Linux 版本之 SAP HANA 版本的支援矩陣，請參閱 [SAP 附註 #2235581](https://launchpad.support.sap.com/#/notes/2235581)。

如需作業系統與 HLI 韌體/驅動程式版本的相容矩陣，請參閱 [HLI 的作業系統升級](os-upgrade-hana-large-instance.md)。


> [!IMPORTANT] 
> 針對類型 II 單元，此時只支援 SLES 12 SP2 作業系統版本。 


**資料庫**

- SAP HANA (平台或企業版) 的授權和軟體安裝元件。

**應用程式**

- 任何連線至 SAP HANA 及相關 SAP 支援合約之 SAP 應用程式的授權和軟體安裝元件。
- 所使用與 SAP HANA on Azure (大型執行個體) 環境及相關支援合約有關的任何非 SAP 應用程式的授權和軟體安裝元件。

**技能**

- Azure IaaS 及其元件的相關經驗與知識。
- 在 Azure 中部署 SAP 工作負載的相關經驗與知識。
- 經認證的 SAP HANA 安裝人員。
- 可設計以 SAP HANA 為中心之高可用性和災害復原的 SAP 架構師技能。

**SAP**

- 預期您是 SAP 客戶且與 SAP 有支援合約。
- 針對在類型 II 類別的 HANA 大型執行個體 SKU 上的實作，請洽詢 SAP 以取得 SAP HANA 的版本以及大型調升規模硬體的最終設定。

**後續步驟**
- 請參閱 [SAP HANA on Azure (大型執行個體) 架構](hana-architecture.md)