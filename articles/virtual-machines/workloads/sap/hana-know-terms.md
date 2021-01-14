---
title: 認識 SAP HANA on Azure (大型執行個體) 的詞彙 | Microsoft Docs
description: 認識 SAP HANA on Azure (大型執行個體) 的詞彙。
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/20/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c1e4e223e7932f7fc8699c04a94b079b7f16e3d5
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98198895"
---
# <a name="know-the-terms"></a>認識詞彙

《架構和技術部署指南》中廣泛使用數個常見的定義。 請注意下列詞彙及其意義：

- **IaaS**：基礎結構即服務。
- **PaaS**：平台即服務。
- **SaaS**：軟體即服務。
- **SAP 元件**：個別的 sap 應用程式，例如 ERP 中央元件 (ECC) 、商務倉儲 (BW) 、方案管理員或企業版入口網站 (EP) 。 SAP 元件可以傳統 ABAP 或 Java 技術為基礎，或以非 NetWeaver 應用程式 (例如商務物件) 為基礎。
- **Sap 環境**：一或多個以邏輯方式分組的 SAP 元件，以執行商務功能，例如開發、品質保證、訓練、嚴重損壞修復或生產環境。
- **Sap 環境**：指的是 IT 環境中的整個 SAP 資產。 SAP 環境包含所有生產和非生產環境。
- **SAP 系統**︰DBMS 層與應用程式層的組合，例如 SAP ERP 開發系統、SAP BW 測試系統、SAP CRM 生產系統等。 Azure 部署不支援在內部部署與 Azure 之間分割這兩層。 SAP 系統可以在內部部署或在 Azure 中部署。 您可以將 SAP 環境的不同系統部署到 Azure 或內部部署中。 例如，您可以在 Azure 中部署 SAP CRM 開發和測試系統，而在內部部署 SAP CRM 生產系統。 就 SAP HANA on Azure (大型執行個體) 而言，您應在 VM 中裝載 SAP 系統的 SAP 應用程式層，並在 SAP HANA on Azure (大型執行個體) 戳記中的某個單位上裝載相關的 SAP HANA 執行個體。
- **大型執行個體戳記**︰經 SAP HANA TDI 認證，並專門用來執行 Azure 內的 SAP HANA 執行個體的硬體基礎結構堆疊。
- **SAP HANA on Azure (大型執行個體)**：Azure 中的供應項目的正式名稱；此供應項目可在經 SAP HANA TDI 認證並部署在不同 Azure 區域的「大型執行個體」戳記中的硬體上執行 HANA 執行個體。 相關詞彙 *HANA 大型執行個體* 是 *SAP HANA on Azure (大型執行個體)* 的簡稱，廣泛使用於本技術部署指南中。
- **跨單位**：說明將 VM 部署到在內部部署資料中心與 Azure 之間具有站對站、多網站或 Azure ExpressRoute 連線能力的 Azure 訂用帳戶的案例。 在一般 Azure 文件中，這類部署也會描述為跨單位案例。 之所以連線，是為了將內部部署網域、內部部署 Azure Active Directory/OpenLDAP 和內部部署 DNS 擴充到 Azure 中。 內部部署架構會擴充至 Azure 訂用帳戶的 Azure 資產。 透過此擴充，VM 可以是內部部署網域的一部分。 

   內部部署網域的網域使用者可以存取伺服器，並可在這些 VM 上執行服務 (例如 DBMS 服務)。 在內部部署的 VM 與 Azure 部署的 VM 之間可進行通訊和名稱解析。 這是部署大部分 SAP 資產時的典型案例。 如需詳細資訊，請參閱 [AZURE VPN 閘道](../../../vpn-gateway/vpn-gateway-about-vpngateways.md) ，並 [使用 Azure 入口網站建立具有站對站連線的虛擬網路](../../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)。
- **租用戶**：部署於 HANA 大型執行個體戳記的客戶會隔離到 *租用戶* 中。 每個租用戶的網路、儲存體和計算層會彼此區隔開來。 指派給不同租用戶的儲存體和計算單位無法看到彼此，也無法在 HANA 大型執行個體戳記層級上彼此通訊。 客戶可以選擇部署到不同的租用戶。 即使如此，租用戶彼此之間還是無法在 HANA 大型執行個體戳記層級上進行通訊。
- **SKU 類別**：對於 HANA 大型執行個體，會提供下列兩種 SKU：
    - **類型 I 類別**： S72、S72m、S96、S144、S144m、S192、S192m、S192xm、S224 和 S224m
    - **類型 II 類別**：S384、S384m、S384xm、S384xxm、S576m、S576xm、S768m、S768xm 和 S960m
- **戳記**：定義「HANA 大型實例」的 Microsoft 內部部署大小。 在部署 HANA 大型實例單位之前，您必須在資料中心位置部署包含計算、網路和存放機架的 HANA 大型實例戳記。 這類部署稱為「HANA 大型實例」戳記或取自修訂 4 (請參閱下面的) ，以使用「**大型實例**」資料列的替代詞彙
- **修訂**：適用于 HANA 大型實例戳記的兩個不同戳記修訂。 這些與 Azure 虛擬機器主機的架構和鄰近性不同
    - "修訂 3" (Rev 3) ：是從2016年中部署的原始設計
    - 「修訂4」 (Rev 4) ：這是一項新的設計，可提供更接近 Azure 虛擬機器主機的內容，以及 Azure Vm 與 HANA 大型實例單位之間較低的網路延遲 
    - 「修訂4.2」 (Rev 4.2) ：在現有的修訂 4 Dc 上，資源會更名至 BareMetal 基礎結構。  客戶可以從 Azure 入口網站的 BareMetal 實例存取其資源。 

您可以透過其他多項資源了解如何在雲端中部署 SAP 工作負載。 如果您想要在 Azure 中執行 SAP HANA 的部署，您必須熟悉並了解 Azure IaaS 的主體及 Azure IaaS 上的 SAP 工作負載部署。 在繼續作業之前，請參閱[在 Azure 虛擬機器上使用 SAP 解決方案](get-started.md)以取得詳細資訊。 

**後續步驟**
- 請參閱 [HLI 認證](hana-certification.md)