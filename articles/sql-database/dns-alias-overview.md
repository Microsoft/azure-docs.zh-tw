---
title: Azure SQL Database 的 DNS 別名 | Microsoft Docs
description: 您的應用程式可以連線到 Azure SQL Database 伺服器名稱的別名。 同時，您可以隨時變更別名所指向的 SQL Database，以便進行測試等等。
services: sql-database
author: MightyPen
manager: craigg
ms.service: sql-database
ms.custom: DNS alias
ms.topic: conceptual
ms.date: 02/05/2018
ms.reviewer: genemi;ayolubek
ms.author: dmalik
ms.openlocfilehash: da771f71566a912a3f172a1382bcd90ab0c3a0a2
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34646063"
---
# <a name="dns-alias-for-azure-sql-database"></a>Azure SQL Database 的 DNS 別名

Azure SQL Database 具有網域名稱系統 (DNS) 伺服器。 PowerShell 和 REST API 接受[呼叫，以便建立和管理 SQL Database 伺服器名稱的 DNS 別名](#anchor-powershell-code-62x)。

「DNS 別名」可用來取代 Azure SQL Database 伺服器名稱。 用戶端程式可以在其連接字串中使用別名。 DNS 別名提供的轉譯層可將用戶端程式重新導向至不同伺服器。 此轉譯層讓您免去必須尋找和編輯所有用戶端及其連接字串的難處。

DNS 別名的常見用法包含下列案例：

- 為 Azure SQL Server 建立好記的名稱。
- 在初始開發期間，您的別名可以參照一部測試 SQL Database 伺服器。 當應用程式上線時，您可以將別名修改為參照生產伺服器。 從測試轉換到生產環境時，不需要修改數個連線到資料庫伺服器之用戶端的組態。
- 假設您的應用程式中唯一的資料庫已移到另一個 SQL Database 伺服器。 您可以在此修改別名，而不需要修改數個用戶端的組態。

#### <a name="domain-name-system-dns-of-the-internet"></a>網際網路的網域名稱系統 (DNS)

網際網路會依賴 DNS。 DNS 會將好記的名稱轉譯為 Azure SQL Database 伺服器的名稱。

## <a name="scenarios-with-one-dns-alias"></a>具有一個 DNS 別名的案例

假設您需要將系統切換至新的 Azure SQL Database 伺服器。 在過去，您需要尋找並更新每個用戶端程式中的每個連接字串。 但是現在，如果連接字串使用 DNS 別名，就只必須更新別名屬性。

Azure SQL Database 的 DNS 別名功能適用於下列案例：

#### <a name="test-to-production"></a>測試到生產

當您開始開發用戶端程式時，讓這些程式在其連接字串中使用 DNS 別名。 您可讓別名的屬性指向 Azure SQL Database 伺服器的測試版本。

稍後當新系統在生產環境中上線時，您就可以更新別名的屬性以指向生產 SQL Database 伺服器。 不需要變更用戶端程式。

#### <a name="cross-region-support"></a>跨區域支援

災害復原可能會將 SQL Database 伺服器轉移至不同的地理區域。 若為使用 DNS 別名的系統，就不需要尋找及更新所有用戶端的所有連接字串。 而是可以將別名更新為參照目前裝載資料庫的新 SQL Database 伺服器。




## <a name="properties-of-a-dns-alias"></a>DNS 別名的屬性

下列屬性適用於 SQL Database 伺服器的每個 DNS 別名：

- *唯一名稱：* 就像伺服器名稱一樣，您所建立的每個別名名稱都是所有 Azure SQL Database 伺服器中的唯一名稱。

- *伺服器是必要的：* 無法建立 DNS 別名，除非該名稱參照正好一部伺服器，且該伺服器必須已存在。 更新後的別名必須一律參照正好一部現有的伺服器。
    - 當您卸除 SQL Database 伺服器時，Azure 系統也會卸除參照該伺服器的所有 DNS 別名。

- *未繫結至任何區域：* DNS 別名未繫結至任何區域。 您可以將任何 DNS 別名更新為參照位於任何地理區域的 Azure SQL Database 伺服器。
    - 不過，將別名更新為參照另一部伺服器時，這兩部伺服器都必須存在於相同的 Azure 訂用帳戶中。

- *權限：* 若要管理 DNS 別名，使用者必須擁有「伺服器參與者」權限或更高階的權限。 如需詳細資訊，請參閱[在 Azure 入口網站中開始使用角色型存取控制](../role-based-access-control/overview.md)。





## <a name="manage-your-dns-aliases"></a>管理 DNS 別名

REST API 和 PowerShell Cmdlet 可讓您以程式設計方式管理 DNS 別名。

#### <a name="rest-apis-for-managing-your-dns-aliases"></a>用於管理 DNS 別名的 REST API

<!-- TODO
??2 "soon" in the following live sentence, is not the best situation.
TODO update this subsection very soon after REST API docu goes live.
Dev = Magda Bojarska
Comment as of:  2018-01-26
-->

下列 Web 位置附近可取得 REST API 的文件：
- [Azure SQL Database REST API](https://docs.microsoft.com/rest/api/sql/)

此外，在 GitHub 中也可以看到 REST API：
- [Azure SQL Database 伺服器，DNS 別名 REST API](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/sql/resource-manager/Microsoft.Sql/preview/2017-03-01-preview/serverDnsAliases.json)

<a name="anchor-powershell-code-62x"/>

#### <a name="powershell-for-managing-your-dns-aliases"></a>用於管理 DNS 別名的 PowerShell

PowerShell Cmdlet 可用於呼叫 REST API。

用來管理 DNS 別名的 PowerShell Cmdlet 程式碼範例記載於：
- [適用於 Azure SQL Database 之 DNS 別名的 PowerShell](dns-alias-powershell.md)


程式碼範例中使用的 Cmdlet 如下所示：
- [New-AzureRMSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/AzureRM.Sql/New-AzureRmSqlServerDnsAlias?view=azurermps-5.1.1)：在 Azure SQL Database 服務系統中建立新的 DNS 別名。 此別名會參照 Azure SQL Database 伺服器 1。
- [Get-AzureRMSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/AzureRM.Sql/Get-AzureRmSqlServerDnsAlias?view=azurermps-5.1.1)：取得並列出所有指派給 SQL DB 伺服器 1 的 DNS 別名。
- [Set-AzureRMSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/AzureRM.Sql/Set-AzureRmSqlServerDnsAlias?view=azurermps-5.1.1)：修改別名已設定要參照的伺服器名稱 (從伺服器 1 改為 SQL DB 伺服器 2)。
- [Remove-AzureRMSqlServerDNSAlias](https://docs.microsoft.com/powershell/module/AzureRM.Sql/Remove-AzureRmSqlServerDnsAlias?view=azurermps-5.1.1)：使用別名的名稱，移除 SQL DB 伺服器 2 中的 DNS 別名。


上述 Cmdlet 已新增至 **AzureRM.Sql** 模組 (從 5.1.1 版模組起)。




## <a name="limitations-during-preview"></a>預覽期間的限制

目前，DNS 別名具有下列限制：

- *最多 2 分鐘的延遲：* 更新或移除 DNS 別名最多需要 2 分鐘的時間。
    - 不論是否有任何短暫的延遲，別名都會立即停止讓用戶端連線參照舊版伺服器。

- *DNS 查閱：* 現在，若要檢查指定的 DNS 別名是參照哪部伺服器，唯一的權威性方式便是執行 [DNS 查閱](https://docs.microsoft.com/windows-server/administration/windows-commands/nslookup)。

- [不支援資料表稽核](sql-database-auditing-and-dynamic-data-masking-downlevel-clients.md)：您無法在資料庫已啟用「資料表稽核」的 Azure SQL Database 伺服器上使用 DNS 別名。
    - 資料表稽核功能已淘汰。
    - 建議您改用 [Blob 稽核](sql-database-auditing.md)。




## <a name="related-resources"></a>相關資源

- [Azure SQL Database 的業務連續性概觀](sql-database-business-continuity.md)，包括災害復原。



## <a name="next-steps"></a>後續步驟

- [適用於 Azure SQL Database 之 DNS 別名的 PowerShell](dns-alias-powershell.md)

