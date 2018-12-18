---
title: Azure SQL Database 和資料倉儲防火牆規則 | Microsoft Docs
description: 了解如何以伺服器層級防火牆規則設定 SQL 資料庫和 SQL 資料倉儲防火牆，以管理存取權，並設定 SQL Database 的資料庫層級防火牆規則。
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: VanMSFT
ms.author: vanto
ms.reviewer: carlrab
manager: craigg
ms.date: 09/20/2018
ms.openlocfilehash: bc246031e57fd8e28cddf9f4e6e170b0695d7579
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166743"
---
# <a name="azure-sql-database-and-sql-data-warehouse-firewall-rules"></a>Azure SQL Database 和 SQL 資料倉儲防火牆規則 

Microsoft Azure [SQL Database](sql-database-technical-overview.md) 和 [SQL 資料倉儲](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)可為 Azure 和其他網際網路型應用程式提供關聯式資料庫服務。 為了協助保護您的資料，防火牆會防止對您的資料庫伺服器的所有存取，直到您指定哪些電腦擁有權限。 此防火牆會根據每一個要求的來源 IP 位址來授與資料庫存取權。

> [!NOTE]
> 本主題適用於 Azure SQL 伺服器，以及在 Azure SQL Server 上建立的 SQL Database 和 SQL 資料倉儲資料庫。 為了簡單起見，參考 SQL Database 和 SQL 資料倉儲時都會使用 SQL Database。

#### <a name="virtual-network-rules-as-alternatives-to-ip-rules"></a>虛擬網路規則可作為 IP 規則的替代方案

除了 IP 規則，防火牆也會管理「虛擬網路規則」。 虛擬網路規則是以虛擬網路服務端點為基礎。 在某些情況下，相較於 IP 規則，最好使用虛擬網路規則。 若要深入了解，請參閱 [Azure SQL Database 的虛擬網路服務端點和規則](sql-database-vnet-service-endpoint-rule-overview.md)。

## <a name="overview"></a>概觀

一開始，防火牆會封鎖對您的 Azure SQL Server 的所有 Transact-SQL 存取。 若要開始使用 Azure SQL Server，您必須指定一或多個伺服器層級防火牆規則，啟用您的 Azure SQL Server 存取。 使用防火牆規則來指定允許網際網路的哪些 IP 位址範圍，以及 Azure 應用程式是否可以嘗試連接到 Azure SQL Server。

若只要選擇性地授與 Azure SQL 伺服器上其中一個資料庫的存取權，您必須為所需的資料庫建立資料庫層級規則。 請為資料庫防火牆規則指定一個 IP 位址範圍，該範圍需在伺服器層級防火牆規則中指定的範圍之外，並確認用戶端的 IP 位址在資料庫層級規則中指定的範圍之內。

> [!IMPORTANT]
> SQL 資料倉儲只支援伺服器層級的防火牆規則，並不支援資料庫層級的防火牆規則。

來自網際網路和 Azure 的連線嘗試必須先通過防火牆，才能到達您的 Azure SQL Server 或 SQL Database，如下圖所示：

   ![圖解防火牆設定。][1]

* **伺服器層級防火牆規則：** 這些規則可讓用戶端存取整個 Azure SQL Server，也就是相同邏輯伺服器內的所有資料庫。 這些規則會儲存在 **master** 資料庫。 使用入口網站或使用 Transact-SQL 陳述式即可設定伺服器層級防火牆規則。 若要使用 Azure 入口網站或 PowerShell 建立伺服器層級的防火牆規則，您必須是訂用帳戶擁有者或訂用帳戶參與者。 若要使用 Transact-SQL 建立伺服器層級的防火牆規則，您必須以伺服器層級的主體登入或 Azure Active Directory 系統管理員身分連接到 SQL 資料庫執行個體 (這表示具有 Azure 層級權限的使用者必須先建立伺服器層級的防火牆規則)。
* **資料庫層級防火牆規則：** 這些規則可讓用戶端存取同部邏輯伺服器內的特定 (安全) 資料庫。 您可針對每個資料庫 (包括 **master** database) 建立這些規則，其會存放在個別的資料庫中。 master 與使用者資料庫的資料庫層級防火牆規則只能使用 Transact-SQL 陳述式建立及管理，且僅可在您已設定第一個伺服器層級防火牆之後進行設定。 如果您在資料庫層級防火牆規則中指定的 IP 位址範圍是在伺服器層級防火牆規則中指定的範圍之外，只有具有資料庫層級範圍中的 IP 位址的那些用戶端才可以存取資料庫。 對於資料庫，您最多可以有 128 個資料庫層級防火牆規則。 如需設定資料庫層級防火牆規則的詳細資訊，請參閱稍後述於 [sp_set_database_firewall_rule (Azure SQL Database)](https://msdn.microsoft.com/library/dn270010.aspx) 的範例。

**建議：** Microsoft 建議在可行時使用資料庫層級防火牆規則來增強安全性，並且讓您的資料庫更具有可攜性。 當您有多個資料庫具有相同存取需求，且不想花時間個別設定每個資料庫時，請對系統管理員使用伺服器層級的防火牆規則。

> [!Important]
> Windows Azure SQL Database 支援最多 128 個防火牆規則。
>

> [!Note]
> 如需可讓業務持續運作的可攜式資料庫資訊，請參閱[災害復原的驗證需求](sql-database-geo-replication-security-config.md)。
>

### <a name="connecting-from-the-internet"></a>從網際網路連線

當電腦嘗試從網際網路連線到資料庫伺服器時，防火牆會先根據資料庫層級的防火牆規則，檢查連線所要求之資料庫的來源 IP 位址︰

* 如果要求的 IP 位址在資料庫層級防火牆規則中指定的其中一個範圍內，就會允許連線至包含規則的 SQL Database。
* 如果要求的 IP 位址不在資料庫層級防火牆規則中指定的其中一個範圍內，則會檢查伺服器層級防火牆規則。 如果要求的 IP 位址在伺服器層級防火牆規則中指定的其中一個範圍內，就會允許連線。 伺服器層級防火牆規則適用於 Azure SQL Server 上的所有 SQL Database。  
* 如果要求的 IP 位址不在任何資料庫層級或伺服器層級防火牆規則中指定的範圍內，連線要求會失敗。

> [!NOTE]
> 若要從您的本機電腦存取 Azure SQL Database，請確定您的網路和本機電腦上的防火牆允許 TCP 連接埠 1433 上的傳出通訊。
> 

### <a name="connecting-from-azure"></a>從 Azure 連線
若要允許應用程式從 Azure 連接到您的 Azure SQL Server，必須啟用 Azure 連接。 當 Azure 的應用程式嘗試連線到您的資料庫伺服器時，防火牆會確認是否允許 Azure 連線。 開始和結束位址等於 0.0.0.0 的防火牆設定表示允許這些連線。 如果不允許連線嘗試，要求就不會到達 Azure SQL Database 伺服器。

> [!IMPORTANT]
> 這個選項會設定防火牆，以允許所有來自 Azure 的連線，包括來自其他客戶訂用帳戶的連線。 選取這個選項時，請確定您的登入和使用者權限會限制為只有授權的使用者才能存取。
> 

## <a name="creating-and-managing-firewall-rules"></a>建立和管理防火牆規則
您可使用 [Azure 入口網站](https://portal.azure.com/)，或是以程式設計方式使用 [Azure PowerShell](https://docs.microsoft.com/powershell/module/azurerm.sql)、[Azure CLI](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-create) 或 [REST API](https://docs.microsoft.com/rest/api/sql/firewallrules)，建立第一個伺服器層級防火牆設定。 後續的伺服器層級防火牆規則可以使用這些方法，以及透過 Transact-SQL 來建立和管理。 

> [!IMPORTANT]
> 資料庫層級防火牆規則只能使用 Transact-SQL 來建立和管理。 
>

為了改進效能，系統會暫時在資料庫層級快取伺服器層級防火牆規則。 若應重新整理快取，請參閱 [DBCC FLUSHAUTHCACHE](https://msdn.microsoft.com/library/mt627793.aspx)。 

> [!TIP]
> 您可以使用 [SQL Database 稽核](sql-database-auditing.md)來稽核伺服器等級和資料庫等級防火牆的變更。
>

## <a name="manage-firewall-rules-using-the-azure-portal"></a>使用 Azure 入口網站管理防火牆規則

若要在 Azure 入口網站中設定伺服器層級防火牆規則，您可前往 Azure SQL Database 的 [概觀] 頁面，或是 Azure Database 邏輯伺服器的 [概觀] 頁面。

> [!TIP]
> 如需教學課程，請參閱[使用 Azure 入口網站建立 DB](sql-database-get-started-portal.md)。
>

**從資料庫概觀頁面**

1. 若要從資料庫概觀頁面設定伺服器層級防火牆規則，請依下圖所示按一下工作列上的 [設定伺服器防火牆]：此時會開啟 [防火牆設定] 頁面。

      ![伺服器防火牆規則](./media/sql-database-get-started-portal/server-firewall-rule.png) 

2. 按一下工具列上的 [新增用戶端 IP]，以新增您目前所用電腦的 IP 位址，然後再按一下 [儲存]。 系統便會為目前的 IP 位址建立伺服器層級防火牆規則。

      ![設定伺服器防火牆規則](./media/sql-database-get-started-portal/server-firewall-rule-set.png) 

**從伺服器概觀頁面**

伺服器的概觀頁面隨即開啟，其中會顯示完整伺服器名稱 (例如 **mynewserver20170403.database.windows.net**)，並提供進一步的組態選項。

1. 若要從伺服器概觀頁面設定伺服器層級規則，按一下左側功能表中 [設定] 下方的 [防火牆]： 

2. 按一下工具列上的 [新增用戶端 IP]，以新增您目前所用電腦的 IP 位址，然後再按一下 [儲存]。 系統便會為目前的 IP 位址建立伺服器層級防火牆規則。

## <a name="manage-firewall-rules-using-transact-sql"></a>使用 Transact-SQL 管理防火牆規則
| 目錄檢視或預存程序 | Level | 說明 |
| --- | --- | --- |
| [sys.firewall_rules](https://msdn.microsoft.com/library/dn269980.aspx) |伺服器 |顯示目前的伺服器層級防火牆規則 |
| [sp_set_firewall_rule](https://msdn.microsoft.com/library/dn270017.aspx) |伺服器 |建立或更新伺服器層級防火牆規則 |
| [sp_delete_firewall_rule](https://msdn.microsoft.com/library/dn270024.aspx) |伺服器 |移除伺服器層級防火牆規則 |
| [sys.database_firewall_rules](https://msdn.microsoft.com/library/dn269982.aspx) |資料庫 |顯示目前的伺服器層級防火牆規則 |
| [sp_set_database_firewall_rule](https://msdn.microsoft.com/library/dn270010.aspx) |資料庫 |建立或更新伺服器層級防火牆規則 |
| [sp_delete_database_firewall_rule](https://msdn.microsoft.com/library/dn270030.aspx) |資料庫 |移除資料庫層級防火牆規則 |


下列範例會檢閱現有的規則、在 Contoso 伺服器上啟用 IP 位址範圍，以及刪除防火牆規則：
   
```sql
SELECT * FROM sys.firewall_rules ORDER BY name;
```
  
接著，加入防火牆規則。
   
```sql
EXECUTE sp_set_firewall_rule @name = N'ContosoFirewallRule',
   @start_ip_address = '192.168.1.1', @end_ip_address = '192.168.1.10'
```

若要刪除伺服器層級防火牆規則，請執行 sp_delete_firewall_rule 預存程序。 下列範例會刪除名為 ContosoFirewallRule 的規則：
   
```sql
EXECUTE sp_delete_firewall_rule @name = N'ContosoFirewallRule'
```   

## <a name="manage-firewall-rules-using-azure-powershell"></a>使用 Azure PowerShell 管理防火牆規則
| Cmdlet | Level | 說明 |
| --- | --- | --- |
| [Get-AzureRmSqlServerFirewallRule](/powershell/module/azurerm.sql/get-azurermsqlserverfirewallrule) |伺服器 |返回目前的伺服器層級防火牆規則 |
| [New-AzureRmSqlServerFirewallRule](/powershell/module/azurerm.sql/new-azurermsqlserverfirewallrule) |伺服器 |建立新的伺服器層級防火牆規則 |
| [Set-AzureRmSqlServerFirewallRule](/powershell/module/azurerm.sql/set-azurermsqlserverfirewallrule) |伺服器 |更新現有伺服器層級防火牆規則的屬性 |
| [Remove-AzureRmSqlServerFirewallRule](/powershell/module/azurerm.sql/remove-azurermsqlserverfirewallrule) |伺服器 |移除伺服器層級防火牆規則 |


下列範例會使用 PowerShell 設定伺服器層級防火牆規則：

```powershell
New-AzureRmSqlServerFirewallRule -ResourceGroupName "myResourceGroup" `
    -ServerName $servername `
    -FirewallRuleName "AllowSome" -StartIpAddress "0.0.0.0" -EndIpAddress "0.0.0.0"
```

> [!TIP]
> 如需快速入門內容中的 PowerShell 範例資訊，請參閱[建立 DB - PowerShell](sql-database-powershell-samples.md)以及[使用 PowerShell 建立單一資料庫和設定防火牆規則](scripts/sql-database-create-and-configure-database-powershell.md)
>

## <a name="manage-firewall-rules-using-azure-cli"></a>使用 Azure CLI 管理防火牆規則
| Cmdlet | Level | 說明 |
| --- | --- | --- |
|[az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-create)|伺服器|建立伺服器防火牆規則|
|[az sql server firewall-rule list](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-list)|伺服器|列出伺服器上的防火牆規則|
|[az sql server firewall-rule show](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-show)|伺服器|顯示防火牆規則的詳細資料|
|[az sql server firewall-rule update](/cli/azure/sql/server/firewall-rule##az-sql-server-firewall-rule-update)|伺服器|更新防火牆規則|
|[az sql server firewall-rule delete](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-delete)|伺服器|刪除防火牆規則|

下列範例會使用 Azure CLI 設定伺服器層級防火牆規則： 

```azurecli-interactive
az sql server firewall-rule create --resource-group myResourceGroup --server $servername \
    -n AllowYourIp --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP]
> 如需快速入門內容中的 Azure CLI 範例資訊，請參閱[建立 DB - Azure CLI](sql-database-cli-samples.md) 以及[使用 Azure CLI 建立單一資料庫和設定防火牆規則](scripts/sql-database-create-and-configure-database-cli.md)
>

## <a name="manage-firewall-rules-using-rest-api"></a>使用 REST API 建立防火牆規則
| API | Level | 說明 |
| --- | --- | --- |
| [列出防火牆規則](https://docs.microsoft.com/rest/api/sql/FirewallRules/ListByServer) |伺服器 |顯示目前的伺服器層級防火牆規則 |
| [建立或更新防火牆規則](https://docs.microsoft.com/rest/api/sql/FirewallRules/CreateOrUpdate) |伺服器 |建立或更新伺服器層級防火牆規則 |
| [刪除防火牆規則](https://docs.microsoft.com/rest/api/sql/FirewallRules/Delete) |伺服器 |移除伺服器層級防火牆規則 |

## <a name="server-level-firewall-rule-versus-a-database-level-firewall-rule"></a>伺服器層級防火牆規則與資料庫層級防火牆規則
問： 某個資料庫的使用者是否應該完全與另一個資料庫隔離？   
  如果是，請使用資料庫層級防火牆規則授與存取權。 這可避免使用伺服器層級防火牆規則，該規則允許透過防火牆存取所有資料庫，這樣會使得您的防禦程度降低。   
 
問： IP 位址的使用者是否需要存取所有資料庫？   
  使用伺服器層級防火牆規則來減少您必須設定防火牆規則的次數。   

問： 設定防火牆規則的人員或小組是否只能透過 Azure 入口網站、PowerShell 或 REST API 進行存取？   
  您必須使用伺服器層級防火牆規則。 資料庫層級防火牆規則只能使用 Transact-SQL 來設定。  

問： 設定防火牆規則的人員或小組是否不被允許具有資料庫層級的高層級權限？   
  使用伺服器層級防火牆規則。 若要使用 Transact-SQL 設定資料庫層級防火牆規則，至少需要資料庫層級的 `CONTROL DATABASE` 權限。  

問： 設定或稽核防火牆規則的人員或小組，是否正在為許多 (也許數百個) 資料庫集中管理防火牆規則？   
  此選項取決於您的需求和環境。 伺服器層級防火牆規則可能會比較容易設定，但是指令碼可以設定資料庫層級的規則。 即使您使用伺服器層級防火牆規則，您可能需要稽核資料庫防火牆規則，以查看具備資料庫 `CONTROL` 權限的使用者是否已建立資料庫層級防火牆規則。   

問： 我是否可以混合使用伺服器層級和資料庫層級防火牆規則？   
  是。 部分使用者 (例如系統管理員) 可能需要伺服器層級防火牆規則。 其他使用者 (例如資料庫應用程式的使用者) 可能需要資料庫層級防火牆規則。   

## <a name="troubleshooting-the-database-firewall"></a>針對資料庫防火牆問題進行疑難排解
當對於 Microsoft Azure SQL Database 服務的存取未如預期運作時，請考慮下列幾點：

* **本機防火牆組態：** 在您的電腦可以存取 Azure SQL Database 之前，您可能需要在電腦上為 TCP 連接埠 1433 建立防火牆例外狀況。 如果您是在 Azure 雲端界限內建立連接，您可能必須開啟其他連接埠。 如需詳細資訊，請參閱[針對 ADO.NET 4.5 及 SQL Database 的 1433 以外的連接埠](sql-database-develop-direct-route-ports-adonet-v12.md)的〈**SQL Database：外部與內部**〉一節。
* **網路位址轉譯 (NAT)：** 由於 NAT，您的電腦用來連接到 Azure SQL Database 的 IP 位址，可能會不同於您電腦 IP 組態設定中顯示的 IP 位址。 若要檢視電腦用來連接到 Azure 的 IP 位址，請登入入口網站，並在裝載您資料庫的伺服器上瀏覽至 [設定] 索引標籤。 在 [允許的 IP 位址] 區段底下，[目前的用戶端 IP 位址] 隨即顯示。 對 [允許的 IP 位址] 按一下 [新增]，以允許此電腦存取伺服器。
* **允許清單的變更尚未生效：** Azure SQL Database 防火牆組態變更可能會延遲最多 5 分鐘才能生效。
* **登入未獲授權或使用不正確的密碼：** 如果 Azure SQL Database 伺服器上的登入沒有權限，或所使用的密碼不正確，與 Azure SQL Database 伺服器的連線就會遭到拒絕。 建立防火牆設定只會讓用戶端有機會嘗試連線至您的伺服器；每個用戶端必須提供必要的安全性認證。 如需準備登入的詳細資訊，請參閱[管理 Azure SQL Database 中的資料庫、登入和使用者](sql-database-manage-logins.md)。
* **動態 IP 位址：** 如果您有使用動態 IP 位址的網際網路連線，並且在通過防火牆時遇到問題，您可以嘗試下列其中一個解決方案：
  
  * 要求您的網際網路服務提供者 (ISP) 將可以存取 Azure SQL Database 伺服器的 IP 位址範圍指派給您的用戶端電腦，然後將 IP 位址範圍新增為防火牆規則。
  * 改為針對您的用戶端電腦取得靜態 IP 位址，然後將 IP 位址新增為防火牆規則。

## <a name="next-steps"></a>後續步驟

- 如需建立資料庫和伺服器層級防火牆規則的快速入門，請參閱[建立 Azure SQL Database](sql-database-get-started-portal.md)。
- 如需從開放原始碼或協力廠商應用程式連接到 Azure SQL Database 的說明，請參閱 [SQL Database 的用戶端快速入門程式碼範例](https://msdn.microsoft.com/library/azure/ee336282.aspx)。
- 如需詳細資訊，請參閱[針對 ADO.NET 4.5 及 SQL Database 的 1433 以外的連接埠](sql-database-develop-direct-route-ports-adonet-v12.md)的〈**SQL Database：外部與內部**〉一節
- 如需 Azure SQL Database 安全性的概觀，請參閱[保護您的資料庫](sql-database-security-overview.md)

<!--Image references-->
[1]: ./media/sql-database-firewall-configure/sqldb-firewall-1.png
