---
title: Azure Stack 上的 SQL 主控伺服器 | Microsoft Docs
description: 如何新增 SQL 執行個體以便透過 SQL 配接器資源提供者佈建。
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
ms.date: 09/27/2018
ms.author: jeffgilb
ms.reviewer: quying
ms.openlocfilehash: 8fc24e58af51a249e3305dbe1496c499387be6b1
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407901"
---
# <a name="add-hosting-servers-for-the-sql-resource-provider"></a>為 SQL 資源提供者新增主控伺服器

您可以使用 [Azure Stack](azure-stack-poc.md) 內 VM 上的 SQL 執行個體，或 Azure Stack 環境外的執行個體，只要資源提供者能夠連線到該執行個體均可。

> [!NOTE]
> SQL 資料庫應建立在 SQL 資源提供者伺服器上。 SQL 資源提供者應建立在預設提供者訂用帳戶中，而 SQL 主控伺服器則應建立在可計費的使用者訂用帳戶中。 資源提供者伺服器不應該用來裝載使用者資料庫。

## <a name="overview"></a>概觀

在您新增 SQL 主控伺服器之前，請檢閱下列必要與一般需求。

### <a name="mandatory-requirements"></a>必要需求

* 啟用 SQL Server 執行個體上的 SQL 驗證。 由於 SQL 資源提供者 VM 未加入網域，因此只能使用 SQL 驗證連線至主控伺服器。
* 安裝在 Azure Stack 上時，將 SQL 執行個體的 IP 位址設定為公用。 資源提供者和使用者 (例如 Web Apps) 會透過使用者網路通訊，因此需要連線到此網路上的 SQL 執行個體。

### <a name="general-requirements"></a>一般需求

* 資源提供者和使用者工作負載專用的 SQL 執行個體。 您無法使用任何其他取用者正在使用的 SQL 執行個體。 這項限制也適用於 App Service。
* 設定具有適當權限層級的帳戶以供資源提供者使用 (如下所述)。
* 您負責管理 SQL 執行個體及其主機。  例如，資源提供者不會套用更新、處理備份，或處理認證輪換。

### <a name="sql-server-virtual-machine-images"></a>SQL Server 虛擬機器映像

您可以透過 Marketplace 管理功能取得 SQL IaaS 虛擬機器映像。 這些映像與 Azure 中提供的 SQL VM 相同。

在使用 Marketplace 項目部署 SQL VM 前，務必先下載最最版的 **SQL IaaS 擴充功能**。 IaaS 擴充功能和對應的入口網站增強功能可提供自動修補和備份之類的其他功能。 如需此擴充功能的詳細資訊，請參閱[使用 SQL Server 代理程式延伸模組 (Resource Manager) 自動化 Azure 虛擬機器上的管理工作](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-agent-extension)。

> [!NOTE]
> Marketplace 中所有位於 Windows 上的 SQL 映像都_需要_ SQL IaaS 延伸模組；若未下載此延伸模組，VM 將無法進行部署。 它無法與以 Linux 為基礎的 SQL 虛擬機器映像搭配使用。

有其他選項可用於部署 SQL VM，包含 [Azure Stack 快速入門資源庫](https://github.com/Azure/AzureStack-QuickStart-Templates)中的範本。

> [!NOTE]
> 安裝在多節點 Azure Stack 上的任何主伺服器，都必須從使用者訂用帳戶 (而非預設提供者訂用帳戶) 建立。 必須透過使用者入口網站或透過具有適當登入的 PowerShell 工作階段來加以建立。 所有主控伺服器都是可計費的 VM，而且必須具有適當的 SQL 授權。 服務管理員_可以_是該訂用帳戶的擁有者。

### <a name="required-privileges"></a>需要的權限

您可以建立具有比 SQL 系統管理員更低權限的系統管理使用者。 此使用者只需要下列作業的權限：

* 資料庫：建立、改變、具有內含項目 (僅限 Always On)、卸除、備份
* 可用性群組：改變、聯結、新增/移除資料庫
* 登入：建立、選取、改變、卸除、撤銷
* 選取作業：\[master\].\[sys\].\[availability_group_listeners\] (AlwaysOn)、sys.availability_replicas (AlwaysOn)、sys.databases、\[master\].\[sys\].\[dm_os_sys_memory\]、SERVERPROPERTY、\[master\].\[sys\].\[availability_groups\] (AlwaysOn)、sys.master_files

### <a name="additional-security-information"></a>額外的安全性資訊

下列資訊提供額外的安全性指導分針：

* 所有 Azure Stack 儲存體都是使用 BitLocker 加密的，因此 Azure Stack 上的任何 SQL 執行個體都會使用加密的 Blob 儲存體。
* SQL 資源提供者完全支援 TLS 1.2。 確定透過 SQL RP 管理的任何 SQL Server _僅_針對 TLS 1.2 設定，而且 RP 將預設為該設定。 所有支援的 SQL Server 版本都支援 TLS 1.2，請參閱 [Microsoft SQL Server 的 TLS 1.2 支援](https://support.microsoft.com/en-us/help/3135244/tls-1-2-support-for-microsoft-sql-server) \(機器翻譯\)。
* 使用 SQL Server 組態管理員設定 **ForceEncryption** 選項，以確保與 SQL server 的所有通訊一律都會經過加密。 請參閱[設定伺服器強制加密連線](https://docs.microsoft.com/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine?view=sql-server-2017#ConfigureServerConnections)。
* 確定任何用戶端應用程式也都會透過加密連線進行通訊。
* RP 會設定為信任 SQL Server 執行個體所使用的憑證。

## <a name="provide-capacity-by-connecting-to-a-standalone-hosting-sql-server"></a>透過連線到獨立式主控 SQL 伺服器來提供容量

您可以使用運用任何版本的 SQL Server 2014 或 SQL Server 2016 的獨立 (非 HA) SQL 伺服器。 確定您擁有的認證適用於具備系統管理員權限的帳戶。

若要新增已設定的獨立主控伺服器，請遵循下列步驟：

1. 以服務管理員身分登入 Azure Stack 操作員入口網站。

2. 選取 [所有服務]&gt;[管理資源]&gt;[SQL 主控伺服器]。

   ![SQL 主控伺服器](./media/azure-stack-sql-rp-deploy/sqlhostingservers.png)

   在 [SQL 主控伺服器] 下方，您可以將 SQL 資源提供者連線到作為資源提供者後端的 SQL Server 執行個體。

   ![SQL 配接器儀表板](./media/azure-stack-sql-rp-deploy/sqlrp-hostingserver.png)

3. 按一下 [新增]，然後在 [新增 SQL 主控伺服器] 刀鋒視窗上提供 SQL Server 執行個體的連線詳細資料。

   ![新增 SQL 主控伺服器](./media/azure-stack-sql-rp-deploy/sqlrp-newhostingserver.png)

    或者，提供執行個體名稱，若執行個體並未指派給預設連接埠 1433，則可提供連接埠號碼。

   > [!NOTE]
   > 只要使用者和管理 Azure Resource Manager 可以存取 SQL 執行個體，資源提供者就可以控制它。 SQL 執行個體「必須」專門配置給資源提供者。

4. 當您新增伺服器時，您必須將它們指派給現有的 SKU，或建立新的 SKU。 在 [Add a Hosting Server] \(新增主控伺服器\) 下方，選取 [SKU]。

   * 若要使用現有的 SKU，選擇可用的 SKU，然後選取 [建立]。
   * 若要建立 SKU，請選取 [+ Create new SKU] \(+ 建立新的 SKU\)。 在 [Create SKU] \(建立 SKU\) 中，輸入必要的資訊，然後選取 [確定]。

     ![建立 SKU](./media/azure-stack-sql-rp-deploy/sqlrp-newsku.png)

## <a name="provide-high-availability-using-sql-always-on-availability-groups"></a>使用 SQL Always On 可用性群組提供高可用性

設定 SQL Always On 執行個體需要額外的步驟，並且需要三部 VM (或實體機器)。本文假設您已非常熟悉 Always On 可用性群組的功能。 如需詳細資訊，請參閱下列文章：

* [Azure 虛擬機器上的 SQL Server Always On 可用性群組簡介](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-overview)
* [Always On 可用性群組 (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2017)

> [!NOTE]
> SQL 配接器資源提供者_僅_支援適用於 Always On 可用性群組的 SQL 2016 SP1 企業版或更新版本執行個體。 此配接器組態需要新的 SQL 功能，例如自動植入。

### <a name="automatic-seeding"></a>自動植入

您必須在 SQL Server 的每個執行個體上，為每個可用性群組啟用 [自動植入](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/automatically-initialize-always-on-availability-group)。

若要在所有執行個體上啟用自動植入，請在主要複本上編輯下列 SQL 命令，然後針對每個次要執行個體執行：

  ```sql
  ALTER AVAILABILITY GROUP [<availability_group_name>]
      MODIFY REPLICA ON '<secondary_node>'
      WITH (SEEDING_MODE = AUTOMATIC)
  GO
  ```

可用性群組必須以方括弧括住。

在次要節點上，執行下列 SQL 命令：

  ```sql
  ALTER AVAILABILITY GROUP [<availability_group_name>] GRANT CREATE ANY DATABASE
  GO
  ```

### <a name="configure-contained-database-authentication"></a>設定自主資料庫驗證

將自主資料庫新增至可用性群組之前，請確定在裝載可用性群組之可用性複本的每個伺服器執行個體上，自主資料庫驗證伺服器選項已設定為 1。 如需詳細資訊，請參閱[自主資料庫驗證伺服器組態選項](https://docs.microsoft.com/sql/database-engine/configure-windows/contained-database-authentication-server-configuration-option?view=sql-server-2017)。

使用這些命令設定每個執行個體的自主資料庫驗證伺服器選項：

  ```sql
  EXEC sp_configure 'contained database authentication', 1
  GO
  RECONFIGURE
  GO
  ```

### <a name="to-add-sql-always-on-hosting-servers"></a>若要新增 SQL Always On 主控伺服器：

1. 以服務管理員身分登入 Azure Stack 管理入口網站。

2. 選取 [瀏覽]&gt;[管理資源]&gt;[SQL 主控伺服器] &gt; [+新增]。

   使用 [SQL 主控伺服器] 之下，您可以將 SQL Server 資源提供者連線到作為資源提供者後端的 SQL Server 實際執行個體。

3. 在表單中填妥 SQL Server 執行個體的連線詳細資料。 確定您使用 Always On 接聽程式的 FQDN 位址 (以及選擇性的連接埠號碼和執行個體名稱)。 針對您以系統管理員權限設定的帳戶，提供資訊。

4. 核取 [Always On 可用性群組] 方塊，即可啟用 SQL Always On 可用性群組執行個體的支援。

   ![啟用 Always On](./media/azure-stack-sql-rp-deploy/AlwaysOn.PNG)

5. 將 SQL Always On 執行個體新增至 SKU。

   > [!IMPORTANT]
   > 您不能在相同 SKU 中混合獨立伺服器與 Always On 執行個體。 嘗試在新增第一個主控伺服器之後混合類型會導致錯誤。

## <a name="sku-notes"></a>SKU 注意事項

您可以使用 SKU 區分服務供應項目。 例如，您可以擁有一個具備以下特性的 SQL Enterprise 執行個體：
  
* 高容量
* 高效能
* 高可用性

在此版本中，無法將 SKU 指派給特定使用者或群組。

 最多需要一小時才能在入口網站中看到 SKU。 在完整建立 SKU 前，使用者無法建立資料庫。

>[!TIP]
>使用可反映和描述 SKU 中伺服器容量的 SKU 名稱，例如容量與效能。 名稱可作為輔助，來幫助使用者將其資料庫部署至適當的 SKU。

最佳作法是，SKU 中的所有主控伺服器應具有相同的資源和效能特性。

## <a name="make-the-sql-databases-available-to-users"></a>將 SQL 資料庫提供給使用者使用

建立方案和供應項目，將 SQL 資料庫提供給使用者使用。 將 **Microsoft.SqlAdapter** 服務新增到方案，然後建立新配額。

## <a name="next-steps"></a>後續步驟

[新增資料庫](azure-stack-sql-resource-provider-databases.md)
