---
title: Azure SQL Database 的虛擬網路服務端點和規則 | Microsoft Docs
description: 將子網路標示為虛擬網路服務端點。 然後將端點標示為虛擬網路規則，以列在 Azure SQL Database 的 ACL 中。 您的 SQL Database 將會接受來自子網路上所有虛擬機器和其他節點的通訊。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: DhruvMsft
ms.author: dmalik
ms.reviewer: vanto, genemi
manager: craigg
ms.date: 09/18/2018
ms.openlocfilehash: 90138664e5eab9110f51bbd3d3755dec0ed59ea8
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166804"
---
# <a name="use-virtual-network-service-endpoints-and-rules-for-azure-sql-database-and-sql-data-warehouse"></a>對 Azure SQL Database 和 SQL 資料倉儲使用虛擬網路服務端點和規則

「虛擬網路規則」是一項防火牆功能，可控制 Azure [SQL Database](sql-database-technical-overview.md) 或 [SQL 資料倉儲](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)伺服器是否接受虛擬網路中的特定子網路所傳來的通訊。 本文說明為何虛擬網路規則功能有時是讓 Azure SQL Database 安全地接受通訊的最佳選項。

> [!NOTE]
> 本主題適用於 Azure SQL 伺服器，以及在 Azure SQL Server 上建立的 SQL Database 和 SQL 資料倉儲資料庫。 為了簡單起見，參考 SQL Database 和 SQL 資料倉儲時都會使用 SQL Database。

若要建立虛擬網路規則，必須先有[虛擬網路服務端點][vm-virtual-network-service-endpoints-overview-649d]規則可供參考。

#### <a name="how-to-create-a-virtual-network-rule"></a>如何建立虛擬網路規則

如果只是建立虛擬網路規則，您可以直接跳到[本文稍後](#anchor-how-to-by-using-firewall-portal-59j)的步驟和說明。






<a name="anch-terminology-and-description-82f" />

## <a name="terminology-and-description"></a>術語和描述

**虛擬網路：** Azure 訂用帳戶可以有相關聯的虛擬網路。

**子網路：** 虛擬網路包含**子網路**。 您有的任何 Azure 虛擬機器 (VM) 會指派給子網路。 一個子網路可以包含多個 VM 或其他計算節點。 計算虛擬網路外部的節點無法存取虛擬網路，除非您設定安全性來允許存取。

**虛擬網路服務端點：**[虛擬網路服務端點][vm-virtual-network-service-endpoints-overview-649d]是一個子網路，其屬性值包含一或多個正式的 Azure 服務類型名稱。 本文中我們探討類型名稱 **Microsoft.Sql**，它參考名為 SQL Database 的 Azure 服務。

**虛擬網路規則：** SQL Database 伺服器的虛擬網路規則是 SQL Database 伺服器的存取控制清單 (ACL) 中所列的子網路。 子網路必須包含 **Microsoft.Sql** 類型名稱，才能列在 SQL Database 的 ACL 中。

虛擬網路規則會指示 SQL Database 伺服器接受來自子網路上每個節點的通訊。







<a name="anch-benefits-of-a-vnet-rule-68b" />

## <a name="benefits-of-a-virtual-network-rule"></a>虛擬網路規則的優點

除非您採取動作，否則子網路上的 VM 無法與 SQL Database 通訊。 建立通訊的一個動作，就是建立虛擬網路規則。 為了選擇虛擬網路 (VNet) 規則方法，基本上需要經過一番比較與對照討論，以確實了解防火牆所提供的各種只能擇其一的安全性選項。

#### <a name="a-allow-access-to-azure-services"></a>A. 允許存取 Azure 服務

防火牆窗格有一個標示為 [允許存取 Azure 服務] 的 [開啟/關閉] 按鈕。 [開啟] 設定允許來自所有 Azure IP 位址和所有 Azure 子網路的通訊。 這些 Azure IP 或子網路可能不是您所擁有。 此 [開啟] 設定可能超過您預計 SQL Database 應該開放的幅度。 虛擬網路規則功能提供更細微的控制。

#### <a name="b-ip-rules"></a>B. IP 規則

SQL Database 防火牆可讓您指定 IP 位址範圍，以接受來自此範圍內的通訊進入 SQL Database。 此方法對 Azure 私人網路外部的穩定 IP 位址很適合。 但 Azure 私人網路內部的許多節點都以「動態」IP 位址設定。 動態 IP 位址可能變更，例如當 VM 重新啟動時。 在生產環境中，請勿在防火牆規則中指定動態 IP 位址。

您可以取得 VM 的「靜態」IP 位址，以挽回 IP 選項。 如需詳細資料，請參閱[使用 Azure 入口網站設定虛擬機器的私人 IP 位址][vm-configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal-321w]。

不過，靜態 IP 方法可能變得難以管理，在大規模使用時成本很高。 虛擬網路規則較容易建立和管理。

#### <a name="c-cannot-yet-have-sql-database-on-a-subnet"></a>C. 子網路上還不能有 SQL Database

如果 Azure SQL Database 伺服器是虛擬網路中某個子網路的節點，則虛擬網路內的所有節點都可以與 SQL Database 通訊。 在此情況下，VM 可以與 SQL Database 通訊，而不需要任何虛擬網路規則或 IP 規則。

不過，截至 2017 年 9 月，Azure SQL Database 服務還不是可指派給子網路的服務。






<a name="anch-details-about-vnet-rules-38q" />

## <a name="details-about-virtual-network-rules"></a>虛擬網路規則的詳細資料

本節描述虛擬網路規則的一些詳細資料。

#### <a name="only-one-geographic-region"></a>只有一個地理區域

每個虛擬網路服務端點只套用至一個 Azure 區域。 端點無法讓其他區域接受來自子網路的通訊。

任何虛擬網路規則只以套用其基礎端點的區域為對象。

#### <a name="server-level-not-database-level"></a>伺服器層級，非資料庫層級

每個虛擬網路規則會套用至整個 Azure SQL Database 伺服器，而不只是伺服器上的一個特定資料庫。 換句話說，虛擬網路規則是在伺服器層級套用，而不是資料庫層級。

- 相反地，IP 規則可以在任一個層級套用。

#### <a name="security-administration-roles"></a>安全性管理角色

有一組獨立的安全性角色負責管理虛擬網路服務端點。 下列每個角色都需要採取動作：

- **網路管理員：** &nbsp;開啟端點。
- **資料庫管理員：** &nbsp;更新存取控制清單 (ACL)，將給定的子網路新增至 SQL Database 伺服器。

RBAC 替代方案：

「網路管理員」和「資料庫管理員」角色的能力已超過管理虛擬網路規則所需。 只需要其中一部分能力。

在 Azure 中，您可以選擇使用[角色型存取控制 (RBAC)][ rbac-what-is-813s] 來建立單一自訂安全性角色，而且只給予一部分必要的能力。 可使用此自訂角色來代替，不必動用到「網路管理員」或「資料庫管理員」。將使用者新增至自訂角色，而不要新增至其他兩個主要的系統管理員角色，就可縮小安全性曝露面。

> [!NOTE]
> 某些案例中，Azure SQL Database 和 VNet 子網路是在不同的訂用帳戶。 在這些情況下，您必須確保下列設定：
> - 兩個訂用帳戶在相同的 Azure Active Directory 租用戶中。
> - 使用者具備啟動作業的必要權限，例如啟用服務端點、將 VNet 子網路新增至指定的伺服器。

## <a name="limitations"></a>限制

對於 Azure SQL Database，虛擬網路規則功能具有下列限制：

- Web 應用程式可以對應到 VNet/子網路中的私人 IP。 即使服務端點已從指定的 VNet/子網路上開啟，從 Web 應用程式至伺服器的連線仍具有 Azure 公用 IP 來源，而非 VNet/子網路來源。 若要啟用從 Web 應用程式到具有 VNet 防火牆規則伺服器的連線，您必須在伺服器上 [允許 Azure 服務存取伺服器]。

- 在針對 SQL Database 的防火牆中，每個虛擬網路規則都會參考一個子網路。 裝載所有這些參考子網路的地理區域，必須和裝載 SQL Database 的地理區域相同。

- 在任何給定的虛擬網路中，每個 Azure SQL Database 伺服器最多只能有 128 個 ACL 項目。

- 虛擬網路規則只套用至 Azure Resource Manager 虛擬網路，而不是[傳統部署模型][arm-deployment-model-568f]網路。

- 開啟 Azure SQL Database 的虛擬網路服務端點也可啟用 MySQL 和 PostgreSQL Azure 服務的端點。 不過，開啟端點時，嘗試從端點連線到您的 MySQL 或 PostgreSQL 執行個體將會失敗。
    - 根本原因是該 MySQL 和 PostgreSQL 目前不支援執行 ACL。

- 在防火牆上，IP 位址範圍會套用到下列網路項目，但虛擬網路規則不這麼做：
    - [站對站 (S2S) 虛擬私人網路 (VPN)][vpn-gateway-indexmd-608y]
    - 透過 [ExpressRoute][expressroute-indexmd-744v] 的內部部署

#### <a name="considerations-when-using-service-endpoints"></a>使用服務端點時的注意事項
使用 Azure SQL Database 的服務端點時，請檢閱下列注意事項：

- **輸出至 Azure SQL Database 公用 IP 是必需的**：必須針對 Azure SQL Database IP 開啟網路安全性群組 (NSG)，才能夠進行連線。 為了完成此操作，您可以使用適用於 Azure SQL Database 的 NSG [服務標籤](../virtual-network/security-overview.md#service-tags)。

#### <a name="expressroute"></a>ExpressRoute

如果您的網路使用 [ExpressRoute][expressroute-indexmd-744v] 連線至 Azure 網路，則每個線路在 Microsoft Edge 會設定兩個公用 IP 位址。 兩個 IP 位址可使用 Azure 公用對等互連來連線至 Microsoft 服務，例如 Azure 儲存體。

若要允許從您的線路與 Azure SQL Database 通訊，您必須為電路的公用 IP 位址建立 IP 網路規則。 若要尋找您 ExpressRoute 線路的公用 IP 位址，請使用 Azure 入口網站開啟具有 ExpressRoute 的支援票證。


<!--
FYI: Re ARM, 'Azure Service Management (ASM)' was the old name of 'classic deployment model'.
When searching for blogs about ASM, you probably need to use this old and now-forbidden name.
-->

## <a name="impact-of-removing-allow-azure-services-to-access-server"></a>移除 [允許 Azure 服務存取伺服器] 的影響。

許多使用者想要從他們的 Azure SQL 伺服器中移除 [允許 Azure 服務存取伺服器]，並取代為 VNet 防火牆規則。
不過移除這個會影響下列 Azure SQLDB 功能：

#### <a name="import-export-service"></a>匯入匯出服務
Azure SQLDB 匯入匯出服務是在 Azure 中的 VM 上執行。 這些 VM 不在您的 VNet 中，因此連線到您的資料庫時會得到一組 Azure IP。 若移除 [允許 Azure 服務存取伺服器]，這些 VM 將無法存取您的資料庫。
您可以解決這個問題。 使用 DACFx API 直接在您的程式碼中執行 BACPAC 匯入或匯出。 請確定在您已設定防火牆規則的 VNet 子網路中的 VM 上，已部署這個 API。

#### <a name="sql-database-query-editor"></a>SQL Database 查詢編輯器
Azure SQL Database 的查詢編輯器會部署在 Azure 中的 VM 上。 這些 VM 不在您的 VNet 中。 因此，連線到您的資料庫時，VM 會得到一組 Azure IP。 若移除 [允許 Azure 服務存取伺服器]，這些 VM 將無法存取您的資料庫。

#### <a name="table-auditing"></a>資料表稽核
目前有兩種做法可以啟用 SQL Database 的稽核。 當您在您的 Azure SQL Server 上啟用服務端點之後，資料表稽核會失敗。 這裡的解決之道是改用 Blob 稽核。

#### <a name="impact-on-data-sync"></a>資料同步的影響
Azure SQLDB 擁有使用 Azure IP 連線到資料庫的資料同步功能。 使用服務端點時，您可能會關閉邏輯伺服器的 [允許 Azure 服務存取伺服器] 存取權。 這項動作將會中斷資料同步功能。

## <a name="impact-of-using-vnet-service-endpoints-with-azure-storage"></a>使用 VNet 服務端點搭配 Azure 儲存體的影響

Azure 儲存體已實作功能，可讓您限制連線至儲存體帳戶的連線。
如果您選擇使用這項功能，並使用 Azure SQL Server 正在使用的儲存體帳戶，可能會遇到問題。 接下來是受此影響的 Azure SQLDB 功能的清單和討論。

#### <a name="azure-sqldw-polybase"></a>Azure SQLDW PolyBase
PolyBase 通常用於將資料從儲存體帳戶載入 Azure SQLDW。 如果您正在載入資料的來源儲存體帳戶限制只能存取一組 VNet 子網路，從 PolyBase 到帳戶的連線會中斷。 目前沒有方法可降低此風險，您可以連絡 Microsoft 支援服務以了解詳細資訊。

#### <a name="azure-sqldb-blob-auditing"></a>Azure SQLDB Blob 稽核
Blob 稽核會將稽核記錄推送到您自己的儲存體帳戶。 如果這個儲存體帳戶使用 VNet 服務端點功能，則 Azure SQLDB 與儲存體帳戶的連線將會中斷。

## <a name="adding-a-vnet-firewall-rule-to-your-server-without-turning-on-vnet-service-endpoints"></a>在不開啟 VNET 服務端點的情況下將 VNET 防火牆規則新增至伺服器

很久以前，在尚未增強這項功能之前，您必須先開啟 VNet 服務端點，才可以在防火牆中實作即時的 VNet 規則。 端點會將指定的 VNet 子網路與 Azure SQL Database 建立關聯。 但是從 2018 年 1 月起，您不再需要這樣做，只需設定 **IgnoreMissingServiceEndpoint** 旗標即可。

只是設定防火牆規則不能協助您保護伺服器。 您也必須開啟 VNet 服務端點，安全性才會生效。 當您開啟服務端點時，您的 VNet 子網路會停機，直到完成關閉到開啟的轉換。 特別是大型的 VNet，這會更明顯。 您可以使用 **IgnoreMissingServiceEndpoint** 旗標來減少或排除在轉換期間的停機時間。

您可以使用 PowerShell 設定 **IgnoreMissingServiceEndpoint** 旗標。 如需詳細資訊，請參閱[使用 PowerShell 建立 Azure SQL Database 的虛擬網路服務端點和規則][sql-db-vnet-service-endpoint-rule-powershell-md-52d]。


## <a name="errors-40914-and-40615"></a>錯誤 40914 和 40615

連線錯誤 40914 與「虛擬網路規則」有關，這些規則會在 Azure 入口網站中的 [防火牆] 窗格中指定。 錯誤 40615 很類似，但它與防火牆上的「IP 位址規則」相關。

#### <a name="error-40914"></a>錯誤 40914

訊息文字：無法開啟登入所要求的伺服器 '*[server-name]*'。 用戶端不得存取該伺服器。

錯誤說明：用戶端所在的子網路含有虛擬網路伺服器端點。 但 Azure SQL Database 伺服器沒有將和 SQL Database 通訊的權限授與子網路的虛擬網路規則。

錯誤解決方式：在 Azure 入口網站的 [防火牆] 窗格上，使用虛擬網路規則控制來為子網路[新增虛擬網路規則](#anchor-how-to-by-using-firewall-portal-59j)。

#### <a name="error-40615"></a>錯誤 40615

訊息文字：無法開啟登入所要求的伺服器 '{0}'。 不允許 IP 位址為 '{1}' 的用戶端存取伺服器。

錯誤說明：未授權 IP 位址連線到 Azure SQL Database 伺服器，但用戶端嘗試從其連線。 伺服器防火牆沒有允許用戶端從指定的 IP 位址與 SQL 資料庫通訊的 IP 位址規則。

錯誤解決方式：輸入用戶端的 IP 位址作為 IP 規則。 您可以使用 Azure 入口網站中的 [防火牆] 窗格來執行這項工作。


數個 SQL Database 錯誤訊息的清單記錄[於此][sql-database-develop-error-messages-419g]。





<a name="anchor-how-to-by-using-firewall-portal-59j" />

## <a name="portal-can-create-a-virtual-network-rule"></a>入口網站可以建立虛擬網路規則

本節說明如何使用 [Azure 入口網站][http-azure-portal-link-ref-477t]在 Azure SQL Database 中建立「虛擬網路規則」。 此規則會指示 SQL Database 接受一個標記為「虛擬網路服務端點」的特定子網路所傳來的通訊。

> [!NOTE]
> 如果您想要將服務端點新增至 Azure SQL Database 伺服器的 VNet 防火牆規則，請先確認已為子網路開啟服務端點。
>
> 若未在子網路上開啟服務端點，入口網站將會要求您加以啟用。 在您新增規則的相同刀鋒視窗上，按一下 [啟用] 按鈕。

#### <a name="powershell-alternative"></a>PowerShell 替代方案

PowerShell 指令碼也可以建立虛擬網路規則。 重要的 Cmdlet **New-AzureRmSqlServerVirtualNetworkRule**。 如有興趣，請參閱[使用 PowerShell 建立 Azure SQL Database 的虛擬網路服務端點和規則][sql-db-vnet-service-endpoint-rule-powershell-md-52d]。

#### <a name="rest-api-alternative"></a>REST API 替代方案

SQL VNet 動作的 PowerShell cmdlet 會在內部呼叫 REST API。 您可以直接呼叫 REST API。

- [虛擬網路規則：作業][rest-api-virtual-network-rules-operations-862r]

#### <a name="prerequisites"></a>必要條件

您必須已有一個子網路是以 Azure SQL Database 相關的特定虛擬網路服務端點「類型名稱」所標記。

- 相關的端點類型名稱是 **Microsoft.Sql**。
- 如果您的子網路可能未標記類型名稱，請參閱[確認您的子網路是端點][sql-db-vnet-service-endpoint-rule-powershell-md-a-verify-subnet-is-endpoint-ps-100]。

<a name="a-portal-steps-for-vnet-rule-200" />

### <a name="azure-portal-steps"></a>Azure 入口網站步驟

1. 登入 [Azure 入口網站][http-azure-portal-link-ref-477t]。

2. 然後，在入口網站中瀏覽至 [SQL 伺服器] &gt; [防火牆/虛擬網路]。

3. 將 [允許存取 Azure 服務] 控制項設為 [關閉]。

    > [!IMPORTANT]
    > 如果讓此控制項保持 [開啟]，Azure SQL Database 伺服器會接受來自任何子網路的通訊。 就安全性觀點而言，讓此控制項保持 [開啟] 可能使存取過多。 Microsoft Azure 虛擬網路服務端點功能，連同 SQL Database 的虛擬網路規則功能，可縮小安全性曝露面。

4. 按一下 [虛擬網路] 區段中的 [+ 新增現有的] 控制項。

    ![按一下新增現有的 (子網路端點，作為 SQL 規則)。][image-portal-firewall-vnet-add-existing-10-png]

5. 在新的 [建立/更新] 窗格中，將您的 Azure 資源名稱填入控制項中。

    > [!TIP]
    > 必須包含子網路的正確 [位址首碼]。 您可以在入口網站中找到值。
    > 瀏覽 [所有資源] &gt;  **[所有類型]** &gt;  **[虛擬網路]**。 篩選條件會顯示您的虛擬網路。 按一下您的虛擬網路，然後按一下 [子網路]。 [位址範圍] 資料行具有您需要的位址首碼。

    ![填入新規則的欄位。][image-portal-firewall-create-update-vnet-rule-20-png]

6. 按一下窗格底端附近的 [確定]按鈕。

7.  在防火牆窗格上查看產生的虛擬網路規則。

    ![在防火牆窗格上查看新的規則。][image-portal-firewall-vnet-result-rule-30-png]


> [!NOTE]
> 規則有下列狀態：
> - **就緒：** 指出您起始的作業已成功。
> - **失敗：** 指出您起始的作業已失敗。
> - **刪除：** 只適用於刪除作業，指出規則已被刪除，且不再套用。
> - **進行中：** 指出正在進行作業。 作業處於此狀態時，會套用舊規則。


<a name="anchor-how-to-links-60h" />

## <a name="related-articles"></a>相關文章

- [Azure 虛擬網路服務端點][vm-virtual-network-service-endpoints-overview-649d]
- [Azure SQL Database 伺服器層級和資料庫層級防火牆規則][sql-db-firewall-rules-config-715d]

Azure SQL Database 的虛擬網路規則功能，已於 2017 年 9 月底推出。

## <a name="next-steps"></a>後續步驟

- [使用 PowerShell 建立 Azure SQL Database 的虛擬網路服務端點和虛擬網路規則][sql-db-vnet-service-endpoint-rule-powershell-md-52d]
- [虛擬網路規則：的作業][rest-api-virtual-network-rules-operations-862r] (使用 REST API)



<!-- Link references, to images. -->

[image-portal-firewall-vnet-add-existing-10-png]: media/sql-database-vnet-service-endpoint-rule-overview/portal-firewall-vnet-add-existing-10.png

[image-portal-firewall-create-update-vnet-rule-20-png]: media/sql-database-vnet-service-endpoint-rule-overview/portal-firewall-create-update-vnet-rule-20.png

[image-portal-firewall-vnet-result-rule-30-png]: media/sql-database-vnet-service-endpoint-rule-overview/portal-firewall-vnet-result-rule-30.png



<!-- Link references, to text, Within this same Github repo. -->

[arm-deployment-model-568f]: ../azure-resource-manager/resource-manager-deployment-model.md

[expressroute-indexmd-744v]: ../expressroute/index.yml

[rbac-what-is-813s]:../role-based-access-control/overview.md

[sql-db-firewall-rules-config-715d]: sql-database-firewall-configure.md

[sql-database-develop-error-messages-419g]: sql-database-develop-error-messages.md

[sql-db-vnet-service-endpoint-rule-powershell-md-52d]: sql-database-vnet-service-endpoint-rule-powershell.md

[sql-db-vnet-service-endpoint-rule-powershell-md-a-verify-subnet-is-endpoint-ps-100]: sql-database-vnet-service-endpoint-rule-powershell.md#a-verify-subnet-is-endpoint-ps-100

[vm-configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal-321w]: ../virtual-network/virtual-networks-static-private-ip-arm-pportal.md

[vm-virtual-network-service-endpoints-overview-649d]: https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview

[vpn-gateway-indexmd-608y]: ../vpn-gateway/index.yml



<!-- Link references, to text, Outside this Github repo (HTTP). -->

[http-azure-portal-link-ref-477t]: https://portal.azure.com/

[rest-api-virtual-network-rules-operations-862r]: https://docs.microsoft.com/rest/api/sql/virtualnetworkrules



<!-- ??2
#### Syntax related articles
- REST API Reference, including JSON

- Azure CLI

- ARM templates
-->
