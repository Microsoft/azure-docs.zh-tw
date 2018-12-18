---
title: 在 Azure Stack 中提供高可用性 SQL 資料庫 | Microsoft Docs
description: 了解如何使用 Azure Stack 來建立 SQL Server 資源提供者主機電腦和高可用性 SQL AlwaysOn 資料庫。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/16/2018
ms.author: jeffgilb
ms.reviewer: quying
ms.openlocfilehash: 17f06a08388720c4483ef1c187edf20ec8359121
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49386378"
---
# <a name="tutorial-offer-highly-available-sql-databases"></a>教學課程：提供高可用性 SQL 資料庫

如果您是 Azure Stack 操作者，便可設定讓伺服器 VM 裝載 SQL 伺服器資料庫。 在 SQL 主控伺服器已成功建立並受 Azure Stack 管理之後，已訂閱 SQL 服務的使用者便可輕鬆建立 SQL 資料庫。

此教學課程說明如何使用 Azure Stack 快速入門範本來建立 [SQL Server AlwaysOn 可用性群組](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server?view=sql-server-2017)、將其新增為「Azure Stack SQL 主控伺服器」，然後建立高可用性 SQL 資料庫。

您將了解：

> [!div class="checklist"]
> * 從範本建立 SQL Server AlwaysOn 可用性群組
> * 建立 Azure Stack SQL 主控伺服器
> * 建立高可用性 SQL 資料庫

在此教學課程中，將會使用可用的 Azure Stack Marketplace 項目來建立並設定一個有兩部 VM 的 SQL Server AlwaysOn 可用性群組。 

開始進行此教學課程中的步驟之前，請確定已成功安裝 [SQL Server 資源提供者](azure-stack-sql-resource-provider-deploy.md)且 Azure Stack Marketplace 中提供下列項目：

> [!IMPORTANT]
> 必須備妥下列各項，才能使用 Azure Stack 快速入門範本。

- [Windows Server 2016 Datacenter](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.WindowsServer) Marketplace 映像。
- Windows Server 2016 上的 SQL Server 2016 SP1 或 SP2 (Enterprise、Standard 或 Developer) 伺服器映像。 此教學課程會使用 [Windows Server 2016 上的 SQL Server 2016 SP2 Enterprise](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft.sqlserver2016sp2enterprisewindowsserver2016) Marketplace 映像。
- [SQL Server IaaS 延伸模組](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-agent-extension) 1.2.30 版或更新版本。 「SQL IaaS 延伸模組」會安裝所有 Windows 版本之 Marketplace SQL Server 項目所需的必要元件。 它可讓您在 SQL 虛擬機器上設定 SQL 特定的設定。 如果未在本機 Marketplace 安裝此延伸模組，SQL 的佈建就會失敗。
- [適用於 Windows 的自訂指令碼延伸模組](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.CustomScriptExtension) 1.9.1 版或更新版本。 「自訂指令碼延伸模組」是一個工具，可用來自動啟動部署後的 VM 自訂工作。
- [PowerShell Desired State Configuration (DSC)](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.DSC-arm) 2.76.0.0 版或更新版本。 DSC 是 Windows PowerShell 中的一個管理平台，可讓您部署和管理軟體服務的設定資料，以及管理這些服務的執行環境。

若要深入了解如何將項目新增至 Azure Stack Marketplace，請參閱 [Azure Stack Marketplace 概觀](azure-stack-marketplace.md)。

## <a name="create-a-sql-server-alwayson-availability-group"></a>建立 SQL Server AlwaysOn 可用性群組
請使用本節中的步驟，運用 [sql-2016-alwayson Azure Stack 快速入門範本](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/sql-2016-alwayson)來部署 SQL Server AlwaysOn 可用性群組。 此範本會在一個「AlwaysOn 可用性群組」中部署兩個 SQL Server Enterprise、Standard 或 Developer 執行個體。 它會建立下列資源︰

- 網路安全性群組
- 虛擬網路
- 四個儲存體帳戶 (一個用於 Active Directory (AD)、一個用於 SQL、一個用於檔案共用見證，還有一個用於 VM 診斷)
- 四個公用 IP 位址 (一個用於 AD、兩個用於每部 SQL VM，還有一個用於繫結至 SQL AlwaysOn 接聽程式的公用負載平衡器)
- 一個用於 SQL VM 且公用 IP 繫結至 SQL AlwaysOn 接聽程式的外部負載平衡器
- 一部針對具有單一網域的新樹系設定為網域控制站的 VM (Windows Server 2016)
- 兩部以 SQL Server 2016 SP1 或 SP2 Enterprise、Standard 或 Developer 版本設定並已叢集化的 VM (Windows Server 2016)。 這些必須是 Marketplace 映像。
- 一部設定為叢集之檔案共用見證的 VM (Windows Server 2016)
- 一個包含 SQL 和檔案共用見證 VM 的可用性設定組  

1. 登入系統管理入口網站：
    - 針對整合式系統部署，入口網站位址會依據您解決方案的區域和外部網域名稱而有所不同。 其格式將會是 https://adminportal.&lt;*區域*&gt;.&lt;*FQDN*&gt;。
    - 如果您使用「Azure Stack 開發套件」(ASDK)，則使用者入口網站位址為 [https://adminportal.local.azurestack.external](https://portal.local.azurestack.external)。

2. 選取 [\+ 建立資源] > [自訂]，然後選取 [範本部署]。

   ![自訂範本部署](media/azure-stack-tutorial-sqlrp/custom-deployment.png)


3. 在 [自訂部署] 刀鋒視窗上，選取 [編輯範本] > [快速入門範本]，然後使用可用自訂範本的下拉式清單來選取 [sql-2016-alwayson] 範本，按一下 [確定]，再按一下 [儲存]。

   ![選取快速入門範本](./media/azure-stack-tutorial-sqlrp/quickstart-template.png)


4. 在 [自訂部署] 刀鋒視窗上，選取 [編輯參數]，然後檢視預設值。 視需要修改值以提供所有必要的參數資訊，然後按一下 [確定]。<br><br> 請至少：

    - 為 ADMINPASSWORD、SQLSERVERSERVICEACCOUNTPASSWORD 及 SQLAUTHPASSWORD 參數提供複雜密碼。
    - 針對 DNSSUFFIX 參數 (如果是 ASDK 安裝，則為 **azurestack.external**)，以全小寫字母輸入用於反向對應的 DNS 尾碼。
    
    ![自訂部署參數](./media/azure-stack-tutorial-sqlrp/edit-parameters.png)

5. 在 [自訂部署]  刀鋒視窗上，選擇要使用的訂用帳戶，然後建立新資源群組或選取現有的資源群組來進行自訂部署。<br><br> 接著，選取資源群組位置 (如果是 ASDK 安裝，則為 [本機])，然後按一下 [建立]。 系統將會驗證自訂部署設定，然後開始進行部署。

    ![自訂部署參數](./media/azure-stack-tutorial-sqlrp/create-deployment.png)


6. 在系統管理入口網站中，選取 [資源群組]，然後選取您針對自訂部署建立的資源群組名稱 (在此範例中是 **resource-group**)。 請檢視部署狀態，以確定所有部署皆已順利完成。<br><br>接著，檢閱資源群組項目並選取 [SQLPIPsql\<資源群組名稱\>] 公用 IP 位址項目。 請記錄負載平衡器公用 IP 的公用 IP 位址和完整 FQDN。 您將需要將向「Azure Stack 操作員」提供此資訊，以便讓他們運用這個 SQL AlwaysOn 可用性群組來建立 SQL 主控伺服器。

   > [!NOTE]
   > 範本部署將需要數小時的時間才能完成。

   ![自訂部署參數](./media/azure-stack-tutorial-sqlrp/deployment-complete.png)

### <a name="enable-automatic-seeding"></a>啟用自動植入
在範本順利部署並設定 SQL AlwaysON 可用性群組之後，您必須在可用性群組中的每個 SQL Server 執行個體上啟用[自動植入](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/automatically-initialize-always-on-availability-group)。 

當您建立具備自動植入功能的可用性群組時，SQL Server 會自動為群組中的每個資料庫建立次要複本，而無須任何其他手動介入來確保 AlwaysOn 資料庫的高可用性。

請使用下列 SQL 命令，為 AlwaysOn 可用性群組設定自動植入。

在主要 SQL 執行個體上 (以主要執行個體 SQL Server 名稱取代 <InstanceName>)：

  ```sql
  ALTER AVAILABILITY GROUP [<availability_group_name>]
      MODIFY REPLICA ON '<InstanceName>'
      WITH (SEEDING_MODE = AUTOMATIC)
  GO
  ```

>  ![主要 SQL 執行個體指令碼](./media/azure-stack-tutorial-sqlrp/sql1.png)

在次要 SQL 執行個體上 (以 AlwaysOn 可用性群組名稱取代 <availability_group_name>)：

  ```sql
  ALTER AVAILABILITY GROUP [<availability_group_name>] GRANT CREATE ANY DATABASE
  GO
  ```
>  ![次要 SQL 執行個體指令碼](./media/azure-stack-tutorial-sqlrp/sql2.png)

### <a name="configure-contained-database-authentication"></a>設定自主資料庫驗證
將自主資料庫新增至可用性群組之前，請確定在裝載可用性群組之可用性複本的每個伺服器執行個體上，自主資料庫驗證伺服器選項已設定為 1。 如需詳細資訊，請參閱 [自主資料庫驗證](https://docs.microsoft.com/sql/database-engine/configure-windows/contained-database-authentication-server-configuration-option?view=sql-server-2017)。

請使用下列命令，為可用性群組中的每個 SQL Server 執行個體設定自主資料庫驗證伺服器選項：

  ```sql
  EXEC sp_configure 'contained database authentication', 1
  GO
  RECONFIGURE
  GO
  ```
>  ![設定自主資料庫驗證](./media/azure-stack-tutorial-sqlrp/sql3.png)

## <a name="create-an-azure-stack-sql-hosting-server"></a>建立 Azure Stack SQL 主控伺服器
在 SQL Server AlwayOn 可用性群組已建立並設定妥當之後，「Azure Stack 操作員」必須建立「Azure Stack SQL 主控伺服器」，才能將額外的容量提供給使用者來建立資料庫。 

請務必使用先前在建立 SQL AlwaysOn 可用性群組的資源群組時所記錄的 SQL 負載平衡器公用 IP (**SQLPIPsql\<資源群組名稱\>**) 或公用 IP 的完整 FQDN。 此外，您還需要知道用來存取 AlwaysOn 可用性群組中 SQL 執行個體的 SQL Server 驗證認證。

> [!NOTE]
> 此步驟必須由「Azure Stack 操作員」從 Azure Stack 系統管理入口網站執行。

有了 SQL AlwaysOn 可用性群組的負載平衡器接聽程式公用 IP 和 SQL 驗證登入資訊之後，Azure Stack 操作員現在便可[使用 SQL AlwaysOn 可用性群組來建立 SQL 主控伺服器](azure-stack-sql-resource-provider-hosting-servers.md#provide-high-availability-using-sql-always-on-availability-groups)。 

此外，也請確定您已建立一些方案和供應項目，以供使用者建立 SQL AlwaysOn 資料庫。 操作員將需要把 **Microsoft.SqlAdapter** 服務新增至方案中，並建立一個專門用於高可用性資料庫的新配額。 如需有關建立方案的詳細資訊，請參閱[方案、供應項目、配額和訂用帳戶概觀](azure-stack-plan-offer-quota-overview.md)。

> [!TIP]
> 將必須等到[已部署 SQL Server 資源提供者](azure-stack-sql-resource-provider-deploy.md)之後，才能新增 **Microsoft.SqlAdapter** 服務。

## <a name="create-a-highly-available-sql-database"></a>建立高可用性 SQL 資料庫
在已建立、設定 SQL AlwaysOn 可用性群組並由「Azure Stack 操作員」新增為「Azure Stack SQL 主控伺服器」之後，具有含 SQL Server 資料庫功能之訂用帳戶的租用戶使用者便可依照本節中的步驟建立支援 Always On 功能的 SQL 資料庫。 

> [!NOTE]
> 請從 Azure Stack 使用者入口網站，以具有可提供 SQL Server 功能 (Microsoft.SQLAdapter 服務) 之訂用帳戶的租用戶使用者身分執行這些步驟。

1. 登入使用者入口網站：
    - 針對整合式系統部署，入口網站位址會依據您解決方案的區域和外部網域名稱而有所不同。 其格式將會是 https://portal.&lt;*區域*&gt;.&lt;*FQDN*&gt;。
    - 如果您使用「Azure Stack 開發套件」(ASDK)，則使用者入口網站位址為 [https://portal.local.azurestack.external](https://portal.local.azurestack.external)。

2. 選取 [\+ 建立資源] > [資料 \+ 儲存體]，然後選取 [SQL Database]。<br><br>請提供必要的資料庫屬性資訊，包括名稱、定序、大小上限、訂用帳戶、資源群組，以及要用於部署的位置。 

   ![建立 SQL 資料庫](./media/azure-stack-tutorial-sqlrp/createdb1.png)

3. 選取 [SKU]，然後選擇要使用的適當「SQL 主控伺服器 SKU」。 在此範例中，「Azure Stack 操作員」已建立 **Enterprise-HA** SKU 來支援 SQL AlwaysOn 可用性群組的高可用性。

   ![選取 SKU](./media/azure-stack-tutorial-sqlrp/createdb2.png)

4. 選取 [登入] > **[建立新的登入]**，然後提供要用於新資料庫的 SQL 驗證認證。 完成時，請依序按一下 [確定] 和 [建立]，以開始資料庫部署程序。

   ![建立登入](./media/azure-stack-tutorial-sqlrp/createdb3.png)

5. 當 SQL 資料庫部署順利完成時，請檢閱資料庫屬性，以找出要用來連線至新高可用性資料庫的連接字串。 

   ![檢視連接字串](./media/azure-stack-tutorial-sqlrp/createdb4.png)




## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 從範本建立 SQL Server AlwaysOn 可用性群組
> * 建立 Azure Stack SQL 主控伺服器
> * 建立高可用性 SQL 資料庫

請前進到下一個教學課程，以了解如何：
> [!div class="nextstepaction"]
> [建立高可用性 MySQL 資料庫](azure-stack-tutorial-mysql.md)