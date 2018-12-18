---
title: Azure SQL Database - 複本的讀取查詢 |Microsoft 文件
description: Azure SQL Database 可讓您使用唯讀複本功能 (稱為讀取相應放大)，對唯讀工作負載進行負載平衡。
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: carlrab
manager: craigg
ms.date: 09/18/2018
ms.openlocfilehash: d82f4e03176911804702db2ea18a5bc9a95583a3
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47158694"
---
# <a name="use-read-only-replicas-to-load-balance-read-only-query-workloads-preview"></a>使用唯讀複本對唯讀查詢工作負載進行負載平衡 (預覽)

**讀取相應放大**可讓您使用唯讀複本功能對 Azure SQL Database 的唯讀工作負載進行負載平衡。 

## <a name="overview-of-read-scale-out"></a>讀取相應放大概觀

進階層 ([ DTU 型購買模型](sql-database-service-tiers-dtu.md)) 或業務關鍵層 ([vCore 型購買模型](sql-database-service-tiers-vcore.md)) 中的每個資料庫，都會自動佈建數個 AlwaysOn 複本，以支援可用性 SLA。

![唯讀複本](media/sql-database-managed-instance/business-critical-service-tier.png)

這些複本會使用與一般資料庫連線所使用的讀寫複本相同的計算大小進行佈建。 **讀取相應放大**功能可讓您使用其中一個唯讀複本功能對 SQL Database 的唯讀工作負載進行負載平衡，而不共用讀寫複本。 這種方式的唯讀工作負載將會與主要讀寫工作負載隔離，而且不會影響其效能。 此功能適用於包含邏輯上分隔唯讀工作負載 (例如分析) 的應用程式，因此可在不需額外費用的情況下使用這個額外容量獲得效能優勢。

若要對特定資料庫使用讀取相應放大功能，您必須在建立資料庫時或者以後明確地啟用它，方法是藉由使用 PowerShell 叫用 [Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase) 或 [New-AzureRmSqlDatabase](/powershell/module/azurerm.sql/new-azurermsqldatabase) Cmdlet，或者透過 Azure Resource Manager REST API 使用[資料庫 - 建立或更新](/rest/api/sql/databases/createorupdate)方法來改變它的組態。 

為資料庫啟用讀取相應放大之後，系統會根據在應用程式連接字串中設定的 `ApplicationIntent` 屬性，將連線到該資料庫的應用程式導向到該資料庫的讀寫複本或唯讀複本。 如需 `ApplicationIntent` 屬性的詳細資訊，請參閱[指定應用程式意圖](https://docs.microsoft.com/sql/relational-databases/native-client/features/sql-server-native-client-support-for-high-availability-disaster-recovery#specifying-application-intent)。

如果讀取相應放大已停用，或您在不支援的服務層中設定 ReadScale 屬性，所有連線都會被導向至與 `ApplicationIntent` 屬性無關的讀寫複本。

> [!NOTE]
> 在預覽期間，唯讀複本並不支援查詢資料存放區和擴充事件。

## <a name="data-consistency"></a>資料一致性

Alwasy ON 的其中一個優點是複本一律處於交易一致狀態，但在不同的時間點，不同的複本之間可能會有些微延遲。 讀取相應放大支援工作階段層級一致性。 這表示，如果唯讀工作階段在無法使用複本所造成的連線錯誤之後重新連線，就可能會被重新導向至與讀寫複本不完全同步的複本。 同樣地，如果應用程式使用讀寫工作階段寫入資料，並且使用唯讀工作階段立即加以讀取，最新的資料可能無法立即顯示。 這是因為對複本執行的交易記錄重做不是同步的。

> [!NOTE]
> 區域內的複寫延遲較低，這種情況很罕見。


## <a name="connecting-to-a-read-only-replica"></a>連線至唯讀複本

當您為資料庫啟用讀取相應放大時，用戶端所提供的連接字串中的 `ApplicationIntent` 選項會指出連線應路由至寫入複本還是唯讀複本。 具體來說，如果 `ApplicationIntent` 值為 `ReadWrite` (預設值)，則連線將會導向至資料庫的讀寫複本。 這與現有行為相同。 如果 `ApplicationIntent` 值為 `ReadOnly`，則連線會路由傳送至唯讀的複本。

例如，下列連接字串會將用戶端連線至唯讀複本 (請將角括號中的項目取代為您的環境適用的值，並去除角括號)：

```SQL
Server=tcp:<server>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadOnly;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```

下列連接字串會將用戶端連線至讀寫複本 (請將角括號中的項目取代為您的環境適用的值，並去除角括號)：

```SQL
Server=tcp:<server>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadWrite;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;

Server=tcp:<server>.database.windows.net;Database=<mydatabase>;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```

您可以執行下列查詢，確認是否已連線到唯讀複本。 它會在連線到唯讀複本時傳回 READ_ONLY。


```SQL
SELECT DATABASEPROPERTYEX(DB_NAME(), 'Updateability')
```
> [!NOTE]
> 在任何指定時間，ReadOnly 工作階段只能存取其中一個 AlwaysON 複本。

## <a name="enable-and-disable-read-scale-out-using-azure-powershell"></a>使用 Azure PowerShell 啟用和停用讀取相應放大

要在 Azure PowerShell 中管理讀取相應放大，必須使用 2016 年 12 月版的 Azure PowerShell 或更新版本。 如需最新 PowerShell 版本的相關資訊，請參閱 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)。

在 Azure PowerShell 中叫用 [Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase) Cmdlet，並傳入所需的 `-ReadScale` 參數值 (`Enabled` 或 `Disabled`)，以啟用或停用讀取相應放大。 或者，您可能使用 [New-AzureRmSqlDatabase](/powershell/module/azurerm.sql/new-azurermsqldatabase) Cmdlet 建立會啟用讀取相應放大的新資料庫。

例如，若要為現有資料庫啟用讀取相應放大 (請將角括號中的項目取代為您的環境適用的值，並去除角括號)：

```powershell
Set-AzureRmSqlDatabase -ResourceGroupName <myresourcegroup> -ServerName <myserver> -DatabaseName <mydatabase> -ReadScale Enabled
```

若要為現有資料庫停用讀取相應放大 (請將角括號中的項目取代為您的環境適用的值，並去除角括號)：

```powershell
Set-AzureRmSqlDatabase -ResourceGroupName <myresourcegroup> -ServerName <myserver> -DatabaseName <mydatabase> -ReadScale Disabled
```

若要建立會啟用讀取相應放大的新資料庫 (請將角括號中的項目取代為您的環境適用的值，並去除角括號)：

```powershell
New-AzureRmSqlDatabase -ResourceGroupName <myresourcegroup> -ServerName <myserver> -DatabaseName <mydatabase> -ReadScale Enabled -Edition Premium
```

## <a name="enabling-and-disabling-read-scale-out-using-the-azure-sql-database-rest-api"></a>使用 Azure SQL Database REST API 啟用和停用讀取相應放大

若要建立會啟用讀取相應放大的資料庫，或是為現有資料庫啟用或停用讀取相應放大，請將 `readScale` 屬性設為 `Enabled` 或 `Disabled` 以建立或更新對應的資料庫實體，如下列範例要求所示。

```rest
Method: PUT
URL: https://management.azure.com/subscriptions/{SubscriptionId}/resourceGroups/{GroupName}/providers/Microsoft.Sql/servers/{ServerName}/databases/{DatabaseName}?api-version= 2014-04-01-preview
Body:
{
   "properties":
   {
      "readScale":"Enabled"
   }
} 
```

如需詳細資訊，請參閱[資料庫 - 建立或更新](/rest/api/sql/databases/createorupdate)。

## <a name="using-read-scale-out-with-geo-replicated-databases"></a>對異地複寫的資料庫使用讀取縮放

如果您要使用讀取縮放對異地複寫資料庫上的唯讀工作負載進行負載平衡 (例如作為容錯移轉群組的成員)，請確定主要與異地複寫的次要資料庫上都已啟用讀取縮放。 這可確保當您的應用程式在容錯移轉後連線到新的主要時，會有相同的負載平衡效果。 如果您要連線到啟用讀取縮放的異地複寫次要資料庫，則會使用與路由傳送主要資料庫上連線的相同方式，將設定 `ApplicationIntent=ReadOnly` 的工作階段路由傳送至其中一個複本。  未設定 `ApplicationIntent=ReadOnly` 的工作階段會路由傳送至異地複寫次要的主要複本，這也是唯讀狀態。 由於異地複寫的次要資料庫具有與主要資料庫不同的端點，因此在過去若要存取次要，不需要設定 `ApplicationIntent=ReadOnly`。 為了確保回溯相容性，`sys.geo_replication_links` DMV 會顯示 `secondary_allow_connections=2` (允許所有用戶端連線)。

> [!NOTE]
> 預覽期間不支援在次要資料庫的本機複本之間執行循環配置資源或任何其他負載平衡路由。 


## <a name="next-steps"></a>後續步驟

- 如需使用 PowerShell 來設定讀取相應放大的相關資訊，請參閱 [Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase) 或 [New-AzureRmSqlDatabase](/powershell/module/azurerm.sql/new-azurermsqldatabase) Cmdlet。
- 如需使用 REST API 來設定讀取相應放大的相關資訊，請參閱[資料庫 - 建立或更新](/rest/api/sql/databases/createorupdate)。
