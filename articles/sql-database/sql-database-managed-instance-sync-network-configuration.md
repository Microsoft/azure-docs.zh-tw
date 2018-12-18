---
title: Azure App Service - 同步處理網路組態 | Microsoft Docs
description: 本文討論如何同步處理 Azure App Service 裝載方案的網路組態。
ms.service: sql-database
author: srdan-bozovic-msft
manager: craigg
ms.custom: managed instance
ms.topic: conceptual
ms.date: 03/07/2018
ms.author: srbozovi
ms.reviewer: bonova, carlrab
ms.openlocfilehash: 57c4dd523a5dffc48a2d2d403d2a440a8d6cde95
ms.sourcegitcommit: c2c64fc9c24a1f7bd7c6c91be4ba9d64b1543231
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39257885"
---
# <a name="sync-networking-configuration-for-azure-app-service-hosting-plan"></a>同步處理 Azure App Service 裝載方案的網路組態

可能會發生這樣的情況：雖然您已[整合您的應用程式與 Azure 虛擬網路](../app-service/web-sites-integrate-with-vnet.md)，但無法建立與受控執行個體的連線。 您可以嘗試重新整理服務方案的網路組態。 

## <a name="sync-network-configuration-for-app-service-hosting-plan"></a>同步處理 App Service 裝載方案的網路組態

若要這樣做，請遵循下列步驟：  

1. 移至您的 Web 應用程式 App Service 方案。
 
   ![App Service 方案](./media/sql-database-managed-instance-sync-networking/app-service-plan.png)

2. 按一下 [網路]，然後按一下 [按一下這裡進行管理]。
 
   ![管理服務方案](./media/sql-database-managed-instance-sync-networking/manage-plan.png)

3. 選取您的 VNet，按一下 [同步處理網路]。 
 
   ![同步網路網路](./media/sql-database-managed-instance-sync-networking/sync.png)

4. 等待同步處理完成。
  
   ![同步處理完成](./media/sql-database-managed-instance-sync-networking/sync-done.png)

您現在可以嘗試重新與您的受控執行個體建立連線。

## <a name="next-steps"></a>後續步驟

- 如需有關設定受控執行個體 VNet 的詳細資訊，請參閱[受控執行個體 VNet 組態](sql-database-managed-instance-vnet-configuration.md)。
