---
title: Azure SQL Database 受控執行個體自訂 DNS | Microsoft Docs
description: 本主題描述 Azure SQL Database 受控執行個體的自訂 DNS 設定選項。
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: bonova, carlrab
manager: craigg
ms.date: 09/23/2018
ms.openlocfilehash: f26ea763d48d03fe7e981b7abbbe64e573ec0b3a
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47224268"
---
# <a name="configuring-a-custom-dns-for-azure-sql-database-managed-instance"></a>設定 Azure SQL Database 受控執行個體的自訂 DNS

Azure SQL Database 受控執行個體必須部署在 Azure [虛擬網路 (VNet)](../virtual-network/virtual-networks-overview.md) 內。 在少數案例中，例如連結的伺服器連線至您雲端或混合式環境中的其他 SQL 執行個體時，就必須從受控執行個體解析私人主機名稱。 在此情況下，您需要在 Azure 內設定自訂 DNS 。 由於受控執行個體會為其內部工作使用相同的 DNS，因此虛擬網路 DNS 設定必須與受控執行個體相容。 

為了讓自訂 DNS 設定和受控執行個體相容，您需要： 
- 設定自訂 DNS 伺服器，讓它可以解析公用網域名稱 
- 將 Azure 遞迴解析程式的 DNS IP 位址 168.63.129.16，放置在虛擬網路 DNS 清單結尾 
 
## <a name="setting-up-custom-dns-servers-configuration"></a>設定自訂 DNS 伺服器組態

1. 在 Azure 入口網站中，找到您的 VNet 的自訂 DNS 選項。

   ![自訂 DNS 選項](./media/sql-database-managed-instance-custom-dns/custom-dns-option.png) 

2. 切換至 [自訂]，輸入您的自訂 DNS 伺服器 IP 位址，以及 Azure 的遞迴解析程式 IP 位址，168.63.129.16。 

   ![自訂 DNS 選項](./media/sql-database-managed-instance-custom-dns/custom-dns-server-ip-address.png) 

   > [!IMPORTANT]
   > 未在 DNS 清單中設定 Azure 的遞迴解析程式，可能會導致受控執行個體在自訂 DNS 伺服器因故無法使用時進入錯誤狀態。 要從該狀態復原，您可能需要在 VNet 中以相容網路原則建立新的執行個體、重新建立執行個體層級資料，並復原您的資料庫。 將 Azure 的遞迴解析程式設定為 DNS 清單中的最後一個項目，可確保即使在所有自訂 DNS 伺服器都失敗時，仍然可解析公用名稱。 請參閱 [VNet 設定](sql-database-managed-instance-vnet-configuration.md)。

## <a name="next-steps"></a>後續步驟

- 如需概觀，請參閱[受控執行個體是什麼](sql-database-managed-instance.md)。
- 如需示範如何新建受控執行個體的教學課程，請參閱[建立受控執行個體](sql-database-managed-instance-get-started.md)。
- 如需有關設定受控執行個體 VNet 的詳細資訊，請參閱[受控執行個體 VNet 組態](sql-database-managed-instance-vnet-configuration.md)
