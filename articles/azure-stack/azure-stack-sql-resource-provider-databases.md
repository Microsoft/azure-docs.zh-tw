---
title: 在 Azure Stack 上使用 SQL 配接器資源提供者所提供的資料庫 | Microsoft Docs
description: 如何建立及管理使用 SQL 配接器資源提供者佈建的 SQL 資料庫
services: azure-stack
documentationCenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/16/2018
ms.author: jeffgilb
ms.reviewer: quying
ms.openlocfilehash: 604e96d03d66cfde049316a1e6e99f07fee5032a
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49362372"
---
# <a name="create-sql-databases"></a>建立 SQL 資料庫

您可以在使用者入口網站中建立及管理自助式資料庫。 Azure Stack 使用者需要供應項目中包含 SQL 資料庫服務的訂用帳戶。

1. 登入 [Azure Stack](azure-stack-poc.md) 使用者入口網站。

2. 選取 [+ 新增] &gt; [資料 + 儲存體] &gt; [SQL Server 資料庫] &gt; [新增]。

3. 在 [建立資料庫] 底下輸入必要的資訊，例如 [資料庫名稱] 和 [大小上限 (MB)]。

   >[!NOTE]
   >資料庫大小必須至少為 64 MB，以便您可以在部署資料庫之後增加該大小。

   視您環境的需求設定其他設定。

4. 在 [建立資料庫] 底下選取 [SKU]。 在 [選取 SKU] 底下選取資料庫的 SKU。

   ![建立 Database](./media/azure-stack-sql-rp-deploy/newsqldb.png)

   >[!NOTE]
   >將主控伺服器新增至 Azure Stack 時，系統會指派一個 SKU 給它們。 並會在 SKU 的主控伺服器集區中建立資料庫。

5. 選取 [登入]。
6. 在 [選取登入] 底下選擇現有的登入，或選取 [+ 建立新的登入]。
7. 在 [新增登入] 底下，輸入 [資料庫登入] 的名稱和 [密碼]。

   >[!NOTE]
   >這些設定為僅供您存取此資料庫而建立的 SQL 驗證認證。 此登入使用者名稱必須是全域唯一的。 您可以將登入設定重複用於其他使用相同 SKU 的資料庫。

   ![建立新的資料庫登入](./media/azure-stack-sql-rp-deploy/create-new-login.png)

8. 選取 [確定] 完成部署資料庫。

在部署資料庫之後顯示的 [基本資訊] 底下，記下 [連接字串]。 您可以在需要存取 SQL Server 資料庫的任何應用程式中使用這個字串。

![擷取連接字串](./media/azure-stack-sql-rp-deploy/sql-db-settings.png)

## <a name="sql-always-on-databases"></a>SQL AlwaysOn 資料庫

根據設計，AlwaysOn 資料庫的處理方式不同於獨立伺服器環境中的處理方式。 如需詳細資訊，請參閱[Azure 虛擬機器上的 SQL Server AlwaysOn 可用性群組簡介](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-overview)。

### <a name="verify-sql-always-on-databases"></a>驗證 SQL AlwaysOn 資料庫

下列螢幕擷取畫面顯示如何使用 SQL Server Management Studio 來查看 SQL AlwaysOn 中的資料庫狀態。

![AlwaysOn 資料庫狀態](./media/azure-stack-sql-rp-deploy/verifyalwayson.png)

AlwaysOn 資料庫應該會顯示為已同步處理，其可在所有 SQL 執行個體中使用並顯示於可用性群組中。 在上一個螢幕擷取畫面中，資料庫範例是 newdb1，而其狀態為 **newdb1 (已同步處理)**。

### <a name="delete-an-alwayson-database"></a>刪除 AlwaysOn 資料庫

當您從資源提供者中刪除 SQL AlwaysOn 資料庫時，SQL 就會從主要複本和可用性群組中刪除資料庫。

然後，SQL 會在其他複本上將資料庫置於還原狀態，除非已觸發，否則並不會卸除資料庫。 如果未卸除資料庫，次要複本會進入「未進行同步處理」狀態。

## <a name="next-steps"></a>後續步驟

[維護 SQL Server 資源提供者](azure-stack-sql-resource-provider-maintain.md)
