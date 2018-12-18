---
title: 跨雲端資料庫的分散式交易
description: Azure SQL Database 的彈性資料庫交易概觀
services: sql-database
ms.service: sql-database
ms.subservice: elastic-scale
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: 3147061f527621ba98dee84f4d347a6e883d61c0
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166464"
---
# <a name="distributed-transactions-across-cloud-databases"></a>跨雲端資料庫的分散式交易
Azure SQL Database (SQL DB) 的彈性資料庫交易可讓您在 SQL DB 中跨多個資料庫執行交易。 SQL DB 的彈性資料庫交易適用於使用 ADO .NET 的 .NET 應用程式，而且與以往熟悉使用 [System.Transaction](https://msdn.microsoft.com/library/system.transactions.aspx) 類別的程式設計經驗整合。 如要取得程式庫，請參閱 [.NET Framework 4.6.1 (Web 安裝程式)](https://www.microsoft.com/download/details.aspx?id=49981)。

在內部部署，這種案例通常需要執行 Microsoft Distributed Transaction Coordinator (MSDTC)。 因為 MSDTC 不適用於 Azure 中的平台即服務應用程式，協調分散式交易的功能現在已直接整合至 SQL DB。 應用程式可以連接到任何 SQL Database 來啟動分散式交易，而其中一個資料庫會明確協調分散式交易，如下圖所示。 

  ![Azure SQL Database 的分散式交易 - 使用彈性資料庫交易 ][1]

## <a name="common-scenarios"></a>常見案例
SQL DB 的彈性資料庫交易可讓應用程式對數個不同 SQL Database 中儲存的資料進行不可部分完成的變更。 預覽版著重於 C# 和 .NET 的用戶端開發經驗。 未來預計加入使用 T-SQL 的伺服器端經驗。  
彈性資料庫交易以下列案例為目標：

* Azure 中的多資料庫應用程式：在此案例中，資料垂直分割到 SQL DB 中的多個資料庫，使得不同種類的資料位於不同的資料庫。 某些作業需要變更兩個以上的資料庫中保存的資料。 應用程式使用彈性資料庫交易來協調資料庫之間的變更，確保不可部分完成性。
* Azure 中的分區化資料庫應用程式：在此案例中，資料層使用 [彈性資料庫用戶端程式庫](sql-database-elastic-database-client-library.md) 或自我分區化功能，在 SQL DB 中橫跨許多資料庫來水平分割資料。 一個顯著的使用案例是在分區化多租用戶應用程式中，當變更牽涉多個租用戶時，需要執行不可部分完成的變更。 例如，從一個租用戶轉移到另一個租用戶，而兩者位於不同的資料庫。 第二個案例是以細緻分區化來因應大型租用戶的容量需求，這又通常表示某些不可部分完成的作業需要延伸至用於相同租用戶的多個資料庫。 第三種案例是以不可部分完成的更新來參考資料庫之間複寫的資料。 現在可以利用預覽版，跨多個資料庫協調這幾方面不可部分完成的交易式作業。
  彈性資料庫交易使用兩階段認可，確保跨資料庫的交易不可部分完成性。 如果交易涉及的資料庫少於 100，則適合併入單一交易內。 不強制規定這些限制，但超出這些限制時，彈性資料庫交易的效能和成功率必然下降。

## <a name="installation-and-migration"></a>安裝和移轉
我們更新 .NET 程式庫 System.Data.dll 和 System.Transactions.dll，以支援在 SQL DB 中執行彈性資料庫交易。 DLL 確保必要時使用兩階段交易認可，以確保不可部分完成性。 若要使用彈性資料庫交易來開始開發應用程式，請安裝 [.NET Framework 4.6.1](https://www.microsoft.com/download/details.aspx?id=49981) 或更新版本。 在舊版 .NET Framework 上執行時，交易無法升級為分散式交易，將會引發例外狀況。

安裝之後，您就可以使用 System.Transactions 中的分散式交易 API 和 SQL DB 連接。 如果您有使用這些 API 的現有 MSDTC 應用程式，只要在安裝 4.6.1 Framework 之後，為 .NET 4.6 重建現有的應用程式即可。 如果專案以 .NET 4.6 為目標，它們會自動使用新 Framework 版本中更新的 DLL，而結合 SQL DB 連接的分散式交易 API 呼叫現在會成功。

請記住，彈性資料庫交易不需要安裝 MSDTC。 彈性資料庫交易直接由 SQL DB 管理。 這可大幅簡化雲端案例，因為 MSDTC 的部署不需要使用分散式交易和 SQL DB。 第 4 節更詳細地說明如何將彈性資料庫交易和必要的 .NET Framework 連同您的雲端應用程式一起部署到 Azure。

## <a name="development-experience"></a>開發經驗
### <a name="multi-database-applications"></a>多重資料庫應用程式
下列範例程式碼使用熟悉的 .NET System.Transactions 程式設計經驗。 TransactionScope 類別會在 .NET 中建立環境交易 (「環境交易」是位於目前執行緒中的交易)。TransactionScope 內開啟的所有連接都參與交易。 如果有不同的資料庫參與，交易會自動提升為分散式交易。 設定完成範圍來指出認可，即可控制交易的結果。

    using (var scope = new TransactionScope())
    {
        using (var conn1 = new SqlConnection(connStrDb1))
        {
            conn1.Open();
            SqlCommand cmd1 = conn1.CreateCommand();
            cmd1.CommandText = string.Format("insert into T1 values(1)");
            cmd1.ExecuteNonQuery();
        }

        using (var conn2 = new SqlConnection(connStrDb2))
        {
            conn2.Open();
            var cmd2 = conn2.CreateCommand();
            cmd2.CommandText = string.Format("insert into T2 values(2)");
            cmd2.ExecuteNonQuery();
        }

        scope.Complete();
    }

### <a name="sharded-database-applications"></a>分區化資料庫應用程式
SQL DB 的彈性資料庫交易也支援協調分散式交易，您需要使用彈性資料庫用戶端程式庫的 OpenConnectionForKey 方法，開啟相應放大資料層的連接。 假設變更跨數個不同分區化索引鍵值，而您需要保證交易一致性。 連接到裝載不同分區化索引鍵值的分區時，由 OpenConnectionForKey 代理連接。 在一般情況下可連接到不同分區，以確保交易保證需要分散式交易。 下列程式碼範例說明此方法。 其中假設使用一個稱為 shardmap 的變數，代表來自彈性資料庫用戶端程式庫的分區對應：

    using (var scope = new TransactionScope())
    {
        using (var conn1 = shardmap.OpenConnectionForKey(tenantId1, credentialsStr))
        {
            conn1.Open();
            SqlCommand cmd1 = conn1.CreateCommand();
            cmd1.CommandText = string.Format("insert into T1 values(1)");
            cmd1.ExecuteNonQuery();
        }

        using (var conn2 = shardmap.OpenConnectionForKey(tenantId2, credentialsStr))
        {
            conn2.Open();
            var cmd2 = conn2.CreateCommand();
            cmd2.CommandText = string.Format("insert into T1 values(2)");
            cmd2.ExecuteNonQuery();
        }

        scope.Complete();
    }


## <a name="net-installation-for-azure-cloud-services"></a>Azure 雲端服務的 .NET 安裝
Azure 會提供數個供應項目，以裝載 .NET 應用程式。 如需不同供應項目的比較，請參閱 [Azure App Service、雲端服務與虛擬機器之比較](../app-service/choose-web-site-cloud-service-vm.md)。 如果供應項目的客體 OS 小於彈性交易所需的 .NET 4.6.1，則您必須將客體 OS 升級至 4.6.1。 

對於 Azure App Service，目前將不支援升級至客體 OS。 對於 Azure 虛擬機器，只要登入 VM，並執行最新的 .NET Framework 的安裝程式即可。 對於 Azure 雲端服務，您需要將新版 .NET 的安裝包含在您部署的啟動工作中。 [在雲端服務角色上安裝 .NET](../cloud-services/cloud-services-dotnet-install-dotnet.md)中說明概念和步驟。  

請注意，相較於 .NET 4.6 的安裝程式，.NET 4.6.1 的安裝程式在 Azure 雲端服務上進行啟動程序期間，可能需要更多的暫存儲存空間。 為了確保能夠順利安裝，您必須在 ServiceDefinition.csdef 檔案中，於啟動工作的 LocalResources 區段和環境設定中，增加 Azure 雲端服務的暫存儲存體，如以下範例所示：

    <LocalResources>
    ...
        <LocalStorage name="TEMP" sizeInMB="5000" cleanOnRoleRecycle="false" />
        <LocalStorage name="TMP" sizeInMB="5000" cleanOnRoleRecycle="false" />
    </LocalResources>
    <Startup>
        <Task commandLine="install.cmd" executionContext="elevated" taskType="simple">
            <Environment>
        ...
                <Variable name="TEMP">
                    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='TEMP']/@path" />
                </Variable>
                <Variable name="TMP">
                    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='TMP']/@path" />
                </Variable>
            </Environment>
        </Task>
    </Startup>

## <a name="transactions-across-multiple-servers"></a>跨多部伺服器的交易
在 Azure SQL Database 中，支援跨不同邏輯伺服器的彈性資料庫交易。 當交易跨越邏輯伺服器的界限時，參與的伺服器首先必須進入一個相互通訊關聯性。 一旦建立通訊關聯性之後，任一部伺服器中的任何資料庫都可以和另一部伺服器中的資料庫一起參與彈性交易。 對於跨越兩個以上的邏輯伺服器的交易，任何一組邏輯伺服器都必須先具備通訊關聯性。

使用下列 PowerShell Cmdlet 管理跨伺服器的通訊關聯性，以進行彈性資料庫交易：

* **New-AzureRmSqlServerCommunicationLink**：使用這個 Cmdlet 建立 Azure SQL DB 中兩部邏輯伺服器之間的新通訊關聯性。 此關聯性是對稱的，也就是說，這兩部伺服器彼此都可以起始交易。
* **Get-AzureRmSqlServerCommunicationLink**：使用這個 Cmdlet 擷取現有的通訊關聯性及其屬性。
* **Remove-AzureRmSqlServerCommunicationLink**：使用這個 Cmdlet 移除現有的通訊關聯性。 

## <a name="monitoring-transaction-status"></a>監視交易狀態
使用 SQL DB 中的動態管理檢視 (DMV) 來監視進行中彈性資料庫交易的狀態和進度。 所有與交易相關的 DMV 都與 SQL DB 中的分散式交易有關聯。 您可以在這裡找到對應的 DMV 清單： [交易相關的動態管理檢視和函數 (Transact-SQL)](https://msdn.microsoft.com/library/ms178621.aspx)。

這些 DMV 特別有用：

* **sys.dm\_tran\_active\_transactions**：列出目前使用中的交易及其狀態。 UOW (工作單位) 資料行可以識別屬於相同分散式交易的不同子交易。 相同分散式交易內的所有交易具有相同的 UOW 值。 如需詳細資訊，請參閱 [DMV 文件](https://msdn.microsoft.com/library/ms174302.aspx) \(機器翻譯\)。
* **sys.dm\_tran\_database\_transactions**：提供交易的其他相關資訊，例如交易在記錄檔中的位置。 如需詳細資訊，請參閱 [DMV 文件](https://msdn.microsoft.com/library/ms186957.aspx) \(機器翻譯\)。
* **sys.dm\_tran\_locks**：提供目前進行中交易所持有的鎖定相關資訊。 如需詳細資訊，請參閱 [DMV 文件](https://msdn.microsoft.com/library/ms190345.aspx) \(機器翻譯\)。

## <a name="limitations"></a>限制
SQL DB 中的彈性資料庫交易目前有下列限制：

* 僅支援 SQL DB 中跨資料庫的交易。 其他 [X/Open XA](https://en.wikipedia.org/wiki/X/Open_XA) 資源提供者和 SQL DB 以外的資料庫無法參與彈性資料庫交易。 這表示彈性資料庫交易無法延伸到內部部署 SQL Server 與 Azure SQL Database。 對於內部部署的分散式交易，請繼續使用 MSDTC。 
* 僅支援來自 .NET 應用程式的用戶端協調交易。 目前已規劃 T-SQL 的伺服器端支援，例如 BEGIN DISTRIBUTED TRANSACTION，但尚未推出。 
* 不支援跨 WCF 服務的交易。 例如，您有執行交易的 WCF 服務方法。 納入交易範圍內的呼叫將會失敗，因為 [System.ServiceModel.ProtocolException](https://msdn.microsoft.com/library/system.servicemodel.protocolexception)。

## <a name="next-steps"></a>後續步驟
如有問題，請透過 [SQL Database 論壇](http://social.msdn.microsoft.com/forums/azure/home?forum=ssdsgetstarted)與我們連絡，如需要求增加功能，請將這些功能新增至 [SQL Database 意見反應論壇](https://feedback.azure.com/forums/217321-sql-database/)。

<!--Image references-->
[1]: ./media/sql-database-elastic-transactions-overview/distributed-transactions.png



