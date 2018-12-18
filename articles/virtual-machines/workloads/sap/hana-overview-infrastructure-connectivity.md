---
title: SAP HANA on Azure (大型執行個體) 的基礎結構和連線 | Microsoft Docs
description: 設定必要的連線基礎結構以使用 SAP HANA on Azure (大型執行個體)。
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
ms.openlocfilehash: c61dffc6fd9d0c65f5e925daebdf2a02cfb5d6ba
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44161351"
---
# <a name="sap-hana-large-instances-deployment"></a>SAP HANA (大型執行個體) 部署 

在您與 Microsoft 企業帳戶小組之間的 SAP HANA on Azure (大型執行個體) 購買程序完成之後，Microsoft 需要下列資訊才能部署「HANA 大型執行個體單位」：

- 客戶名稱
- 業務連絡人資訊 (包括電子郵件地址和電話號碼)
- 技術連絡人資訊 (包括電子郵件地址和電話號碼)
- 技術網路連絡人資訊 (包括電子郵件地址和電話號碼)
- Azure 部署區域 (自 2017 年 7 月起：美國西部、美國東部、澳大利亞東部、澳大力亞東南部、西歐和北歐)
- 確認 SAP HANA on Azure (大型執行個體) SKU (組態)
- 如「HANA 大型執行個體」的＜概觀與架構＞文件中所述，針對每個作為部署目的地的「Azure 區域」：
    - 將 Azure VNet 連接到「HANA 大型執行個體」之 ER-P2P 連線的 /29 IP 位址
    - 用於「HANA 大型執行個體伺服器 IP 集區」的 /24 CIDR 區塊
- 每個連接到 HANA 大型執行個體之 Azure VNet 的 Vnet 位址空間屬性中所使用的 IP 位址範圍值
- 每個「HANA 大型執行個體」系統的資料：
  - 所需的主機名稱 - 最好是完整的網域名稱。
  - 來自伺服器 IP 集區位址範圍的所需 HANA 大型執行個體單位 IP 位址 - 請記住，伺服器 IP 集區位址範圍中的前 30 個 IP 位址，是保留給在 HANA 大型執行個體中供內部使用的 IP 位址。
  - SAP HANA 執行個體的 SAP HANA SID 名稱 (這是建立必要 SAP HANA 相關磁碟區所需的名稱)。 在 NFS 磁碟區上建立 sidadm 權限需要有 HANA SID，這些磁碟區會附加至 HANA 大型執行個體單位。 也會用做為掛接磁碟區的其中一個名稱元件。 如果您想要在單位上執行多個 HANA 執行個體，您需要列出多個 HANA SID。 每一個都會分別指派一組磁碟區。
  - 必須要有 sidadm 使用者在 Linux OS 中所擁有的 groupid，才能建立必要的 SAP HANA 相關磁碟區。 SAP HANA 安裝通常會建立群組識別碼為 1001 的 sapsys 群組。 sidadm 使用者為該群組的成員
  - 必須要有 sidadm 使用者在 Linux OS 中所擁有的 userid，才能建立必要的 SAP HANA 相關磁碟區。 如果要在單位上執行多個 HANA 執行個體，您需要列出所有的 <sid> adm 使用者。 
- Azure HANA 大型執行個體上 SAP HANA 要直接連接的 Azure 訂用帳戶的 Azure 訂用帳戶識別碼。 此訂用帳戶識別碼參考 Azure 訂用帳戶，會要支付 HANA 大型執行個體單位的費用。

在您提供資訊之後，Microsoft 就會佈建 SAP HANA on Azure (大型執行個體)，並且會傳回將 Azure VNet 連接到「HANA 大型執行個體」及存取「HANA 大型執行個體」單位所需的資訊。

閱讀本文章之前，請熟悉 [HANA 大型執行個體的常見術語](hana-know-terms.md)和 [HANA 大型執行個體 SKU](hana-available-skus.md)。

當 Microsoft 部署好 HANA 大型執行個體後，您可以使用下列順序來與之連線：

1. [將 Azure VM 連線到 HANA 大型執行個體](hana-connect-azure-vm-large-instances.md)
2. [將 VNet 連線到 HANA 大型執行個體 ExpressRoute](hana-connect-vnet-express-route.md)
3. [其他網路需求 (選擇性)](hana-additional-network-requirements.md)

**後續步驟**

- 請參閱[將 Azure VM 連線到 HANA 大型執行個體](hana-connect-azure-vm-large-instances.md)。
- 請參閱[將 VNet 連線到 HANA 大型執行個體 ExpressRoute](hana-connect-vnet-express-route.md)。