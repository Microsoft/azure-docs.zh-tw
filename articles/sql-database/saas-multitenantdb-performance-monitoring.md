---
title: 監視多租用戶 SaaS 應用程式中分區化多租用戶 Azure SQL Database 的效能 | Microsoft Docs
description: 監視及管理多租用戶 SaaS 應用程式中分區化多租用戶 Azure SQL Database 的效能
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 09/14/2018
ms.openlocfilehash: 7681e3fabe9eb216da81d9f09dc584097bcbaf84
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48830000"
---
# <a name="monitor-and-manage-performance-of-sharded-multi-tenant-azure-sql-database-in-a-multi-tenant-saas-app"></a>監視及管理多租用戶 SaaS 應用程式中分區化多租用戶 Azure SQL Database 的效能

本教學課程會探索 SaaS 應用程式中使用的數個關鍵效能管理案例。 使用負載產生器來跨分區化多租用戶資料庫模擬活動，而且示範 SQL Database 的內建監視和警示功能。

Wingtip Tickets SaaS 多租用戶資料庫應用程式會使用分區化多租用戶資料模型，場地 (租用戶) 資料會在當中依租用戶識別碼跨可能為多個的資料庫進行分散。 與許多 SaaS 應用程式類似，預期的租用戶工作負載模式無法預測且偶爾發生。 換句話說，票證銷售可能會在任何時間發生。 若要利用這個一般資料庫使用模式，可以將資料庫調相應增加及相應減少，從而最佳化解決方案的成本。 使用這種類型的模式，請務必監視資料庫資源使用量，以確保合理地跨可能為多個的資料庫集區平衡負載。 您也必須確定個別的資料庫擁有足夠的資源，且未達到其 [DTU](sql-database-service-tiers.md#dtu-based-purchasing-model) 限制。 本教學課程將探討監視及管理資料庫的方式，以及如何採取矯正措施以回應工作負載的變化。

在本教學課程中，您將了解如何：

> [!div class="checklist"]

> * 透過執行提供的負載產生器，來模擬分區化多租用戶資料庫上的使用量
> * 在資料庫回應負載增加時加以監視
> * 相應增加資料庫，以回應增加的資料庫負載
> * 將租用戶佈建到單一租用戶資料庫

若要完成本教學課程，請確定已完成下列必要條件：

* 已部署 Wingtip Tickets SaaS 多租用戶資料庫應用程式。 若要在五分鐘內完成部署，請參閱[部署及探索 Wingtip Tickets SaaS 多租用戶資料庫應用程式](saas-multitenantdb-get-started-deploy.md)
* 已安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)

## <a name="introduction-to-saas-performance-management-patterns"></a>SaaS 效能管理模式簡介

管理資料庫效能包含了編譯及分析效能資料，然後透過調整參數回應此資料，以維持您應用程式的可接受回應時間。 

### <a name="performance-management-strategies"></a>效能管理策略

* 若要避免手動監視效能，最有效的方式是**設定在資料庫偏離正常範圍時觸發的警示**。
* 若要回應資料庫計算大小的短期波動，可以**相應增加或減少 DTU 層級**。 如果這樣的波動是以定期或可預測的方式發生，則**可排定自動調整資料庫**。 例如，當您知道夜間或者週末期間工作負載較輕時，就可以相應減少。
* 若要回應長期波動或租用戶中的變更，**個別租用戶可以移至其他租用戶**。
* 若要回應「個別」租用戶負載中的短期增加，**可由資料庫中取出個別租用戶並指派個別計算大小**。 一旦負載降低後，就可以讓租用戶返回到多租用戶資料庫。 事先知道時，則能預先移動租用戶以確保資料庫一律擁有它所需的資源，和避免影響多租用戶資料庫中的其他租用戶。 如果可預測此需求，例如場地因熱門活動而發生票券銷售熱潮，則此管理行為可以與應用程式整合。

[Azure 入口網站](https://portal.azure.com)提供大部分資源的內建監視與警示功能。 針對 SQL Database，可使用資料庫監視與警示功能。 這個內建的監視與警示功能是資源特定，因此針對少數資源使用很方便，但是搭配許多資源使用時則不方便。

在您處理許多資源的大量案例中，可以使用 [Log Analytics](https://azure.microsoft.com/services/log-analytics/)。 這是個別的 Azure 服務，可針對發出的診斷記錄和記錄分析工作區中收集的遙測提供分析。 Log Analytics 可以收集來自許多服務的遙測並用來查詢及設定警示。

## <a name="get-the-wingtip-tickets-saas-multi-tenant-database-application-source-code-and-scripts"></a>取得 Wingtip Tickets SaaS 多租用戶資料庫應用程式原始碼和指令碼

可在 [WingtipTicketsSaaS MultitenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB) GitHub 存放庫中使用 Wingtip Tickets SaaS 多租用戶資料庫指令碼和應用程式來源程式碼。 關於下載和解除封鎖 Wingtip Tickets SaaS 指令碼的步驟，請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)。

## <a name="provision-additional-tenants"></a>佈建其他租用戶

為了深入了解效能監視與管理大規模運作的方式，本教學課程要求您在分區化多租用戶資料庫中有多個租用戶。

如果您已在先前的教學課程中佈建批次的租用戶，請跳至[模擬所有租用戶資料庫上的使用量](#simulate-usage-on-all-tenant-databases)一節。

1. 在 **PowerShell ISE** 中開啟 …\\Learning Modules\\Performance Monitoring and Management\\Demo-PerformanceMonitoringAndManagement.ps1。 保持此指令碼開啟，因為您將在本教學課程期間執行數個案例。
1. 設定 **$DemoScenario** = **1**，_佈建批次的租用戶_
1. 按 **F5** 以執行指令碼。

指令碼會在幾分鐘內將 17 個租用戶部署至多租用戶資料庫。 

New-TenantBatch 指令碼會使用分區化多租用戶資料庫內的唯一租用戶索引鍵來建立新的租用戶，並使用租用戶名稱和場地類型將它們初始化。 這與應用程式佈建新租用戶的方式一致。 

## <a name="simulate-usage-on-all-tenant-databases"></a>模擬所有租用戶資料庫的使用情形

提供的 Demo-PerformanceMonitoringAndManagement.ps1 指令碼會模擬對多租用戶資料庫執行的工作負載。 使用其中一個可用的負載案例產生負載：

| 示範 | 案例 |
|:--|:--|
| 2 | 產生一般強度負載 (大約 30 個 DTU) |
| 3 | 每個租用戶產生具有更長高載的負載|
| 4 | 每個租用戶產生具有更高 DTU 高載的負載 (大約 70 個 DTU)|
| 5 | 在單一租用戶上產生高強度 (大約 90 個 DTU)，並在所有其他租用戶上產生標準強度的載入 |

負載產生器會將僅限「綜合」 CPU 的負載套用到每一個租用戶資料庫。 產生器會針對每個租用戶資料庫啟動作業，這會定期呼叫產生負載的預存程序。 負載層級 (單位為 DTU)、持續時間和間隔會跨所有的資料庫而變動，以模擬無法預測的租用戶活動。

1. 在 **PowerShell ISE** 中開啟 …\\Learning Modules\\Performance Monitoring and Management\\Demo-PerformanceMonitoringAndManagement.ps1。 保持此指令碼開啟，因為您將在本教學課程期間執行數個案例。
1. 設定 **$DemoScenario** = **2**，_產生一般強度負載_
1. 按下 **F5** 將負載套用到您的所有的租用戶。

Wingtip Tickets SaaS 多租用戶資料庫是 SaaS 應用程式，而實際 SaaS 應用程式上的負載通常是偶爾發生且無法預測的。 為了模擬此情況，負載產生器會產生跨所有租用戶散發的隨機負載。 需要數分鐘才會出現負載模式，所以在嘗試監視以下各節中的負載時，請執行負載產生器 3-5 分鐘。

> [!IMPORTANT]
> 負載產生器正於新的 PowerShell 視窗中以一系列作業的方式執行。 如果您關閉工作階段，負載產生器就會停止。 負載產生器會維持 job-invoking 狀態，它會在產生器啟動後佈建的任何新租用戶上產生負載。 使用 *CTRL-C* 來停止叫用新作業並結束指令碼。 負載產生器將會繼續執行，但只在現有租用戶上執行。

## <a name="monitor-resource-usage-using-the-azure-portal"></a>使用 Azure 入口網站監視資源使用量

若要監視從已套用的負載所產生的資源使用量，請開啟入口網站並移至包含租用戶的多租用戶資料庫 **tenants1**：

1. 開啟 [Azure 入口網站](https://portal.azure.com)並瀏覽至伺服器 tenants1-mt-&lt;USER&gt;。
1. 向下捲動並找出資料庫，然後按一下 [tenants1]。 此分區化多租用戶資料庫目前為止包含所建立的所有租用戶。

![資料庫圖表](./media/saas-multitenantdb-performance-monitoring/multitenantdb.png)

觀察 **DTU** 圖表。

## <a name="set-performance-alerts-on-the-database"></a>設定資料庫上的效能警示

在資料庫上設定 \>75% 使用率時觸發的警示，如下所示：

1. 在 [Azure 入口網站](https://portal.azure.com)中開啟 tenants1 資料庫 (在 tenants1-mt-&lt;USER&gt; 伺服器上)。
1. 按一下 [警示規則]，然後按一下 [+ 加入警示]：

   ![加入警示](media/saas-multitenantdb-performance-monitoring/add-alert.png)

1. 提供名稱，例如「高 DTU」，
1. 設定下列值：
   * **度量 = DTU 百分比**
   * **條件 = 大於**
   * **閾值 = 75**。
   * **期間 = 過去 30 分鐘內**
1. 將電子郵件地址新增至 [其他系統管理員電子郵件] 方塊，然後按一下 [確定]。

   ![設定警示](media/saas-multitenantdb-performance-monitoring/set-alert.png)

## <a name="scale-up-a-busy-database"></a>相應增加忙碌的資料庫

如果資料庫上的負載層級增加到超出資料庫最大值且達到 100% DTU 使用率，就會影響資料庫效能，可能拖慢查詢回應時間。

**簡短字詞**，請考慮向上擴充資料庫以提供額外的資源，或從多租戶資料庫移除租用戶 (將其從多租用戶資料庫移出至獨立的資料庫)。

就**長期**來說，請考慮最佳化查詢或使用索引來改善資料庫效能。 根據應用程式對效能問題的敏感度，最佳做法是在資料庫達到 100% DTU 使用量之前加以相應增加。 使用警示來事先警告您。

您可以藉由增加產生器所產生的負載，模擬忙碌的資料庫。 讓租用戶高載時間更頻繁且更長，增加多租戶資料庫上的負載而不變更個別租用戶的需求。 在入口網站或從 PowerShell 可以輕鬆地相應增加資料庫。 此練習使用入口網站。

1. 設定 *$DemoScenario* = **3**，_每個資料庫產生時間更長、更頻繁高載的負載_，以增加資料庫的彙總負載強度，而不變更每個租用戶所需的尖峰負載。
1. 按下 **F5** 將負載套用到您的所有租用戶資料庫。
1. 在 Azure 入口網站中移至 **tenants1** 資料庫。

監視上方圖表上增加的資料庫 DTU 使用量。 需要花數分鐘讓新的高負載開始作用，但您應該很快會看到資料庫開始達到最大使用率，而隨著負載穩定進入新的模式，它會快速地使資料庫多載。

1. 若要相應增加資料庫，請在 [設定] 刀鋒視窗中按一下 [定價層 (調整 DTU)]。
1. 將 [DTU] 設定調整為 **100**。 
1. 按一下 [套用] 以提交調整資料庫的要求。

返回至 **tenants1** > **概觀** 以檢視監視圖表。 監視為資料庫提供更多資源的影響 (雖然有一些租用戶和隨機負載，但是在您執行一段時間前，也不容易明確地看出)。 當您查看圖表時，請記住上方圖表上的 100% 現在代表 100 個 DTU，而下方圖表上的 100% 仍是 50 個 DTU。

整個過程中，資料庫會維持連線且完全可供使用。 應用程式程式碼應該一律撰寫為重試中斷的連線，且因此重新連線至資料庫。

## <a name="provision-a-new-tenant-in-its-own-database"></a>在它自己的資料庫中佈建新租用戶 

分區化多租用戶模型可讓您選擇是否要在多租用戶資料庫與其他租用戶佈建新的租用戶，或是在其自己的資料庫中佈建租用戶。 藉由在租用戶自己的資料庫中佈建租用戶，會受惠於個別資料庫中的隔離本質，可讓您獨立於其他租用戶的效能管理該租用戶、獨立於其他租用戶進行還原等等。例如，您可能會選擇將免費試用或一般客戶放在多租用戶資料庫中，而將進階客戶放在個別的資料庫。  如果您建立了隔離的單一租用戶資料庫，仍可在彈性集區中將它們管理共同，以最佳化資源成本。

如果您已在它自己的資料庫中佈建新的租用戶，請略過接下來的步驟。

1. 在 **PowerShell ISE** 中，開啟.…\\Learning Modules\\ProvisionTenants\\Demo-ProvisionTenants.ps1。 
1. 修改 **$TenantName = "Salix Salsa"** 和 **$VenueType  = "dance"**
1. 設定 **$Scenario** = **2**，_在新的單一租用戶資料庫中佈建租用戶_
1. 按 **F5** 以執行指令碼。

指令碼會在另一個資料庫中佈建此租用戶、使用目錄註冊資料庫與租用戶，然後在瀏覽器中開啟 [租用戶的事件] 頁面。 重新整理 [事件中樞] 頁面，您就會看到 "Salix Salsa" 已新增為場地。

## <a name="manage-performance-of-a-single-database"></a>管理單一資料庫的效能

如果多租用戶資料庫內的單一租用戶遇到持續高負載，它可能會傾向於支配資料庫資源，並會影響相同資料庫中的其他租用戶。 如果活動可能會持續一陣子，則可以暫時將租用戶移出資料庫，並移入其自己的單一租用戶資料庫。 這可讓該租用戶擁有超出其所需的額外資源，並將它與其他租用戶完全隔離。

本練習會模擬在銷售熱門事件的票券時，Salix Salsa 發生高負載的效果。

1. 開啟 …\\*Demo-PerformanceMonitoringAndManagement.ps1* 指令碼。
1. 設定 **$DemoScenario = 5**，_在單一租用戶上產生一般負載加上高負載 (大約 90 個 DTU)。_
1. 設定 **$SingleTenantName = Salix Salsa**
1. 使用 **F5** 執行指令碼。

移至入口網站並瀏覽至 **salixsalsa** > **概觀**以檢視監視圖表。 

## <a name="other-performance-management-patterns"></a>其他效能管理模式

**租用戶自助調整**

因為調整是可以透過管理 API 輕鬆呼叫的工作，您可以輕易地在您的租用戶面向應用程式中建立調整租用戶資料庫的能力，並以 SaaS 服務的功能形式提供。 例如，讓租用戶自行管理相應增加和減少，或許會直接影響其計費！

**依排程相應增加和減少資料庫，以符合使用模式**

當彙總租用戶使用量遵循可預測的使用模式時，您可以使用「Azure 自動化」依排程相應增加和減少資料庫。 例如，當您知道資源需求會下降時，在工作日下午 6 點後相應減少資料庫，而在上午 6 點之前重新相應增加。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 透過執行提供的負載產生器，來模擬分區化多租用戶資料庫上的使用量
> * 在資料庫回應負載增加時加以監視
> * 相應增加資料庫，以回應增加的資料庫負載
> * 將租用戶佈建到單一租用戶資料庫

## <a name="additional-resources"></a>其他資源

<!--* [Additional tutorials that build upon the Wingtip Tickets SaaS Multi-tenant Database application deployment](saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)-->
* [Azure 自動化](../automation/automation-intro.md)