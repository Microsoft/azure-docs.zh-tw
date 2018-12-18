---
title: 使用 Azure Log Analytics 最佳化 SQL Server 環境 | Microsoft Docs
description: 透過 Azure Log Analytics，您可以使用 SQL 健康情況檢查方案，定期評估環境的風險和健康狀況。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: e297eb57-1718-4cfe-a241-b9e84b2c42ac
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/19/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 2dcbd0cab055e8a2b2128b1f83f5c080a0887b85
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43122515"
---
# <a name="optimize-your-sql-environment-with-the-sql-server-health-check-solution-in-log-analytics"></a>在 Log Analytics 中使用 SQL Server 健康情況檢查方案最佳化 SQL 環境

![SQL 健康情況檢查標誌](./media/log-analytics-sql-assessment/sql-assessment-symbol.png)

您可以使用 SQL 健康情況檢查方案定期評估伺服器環境的風險和健康情況。 本文將協助您安裝方案，讓您可以針對潛在問題採取修正動作。

此方案能針對已部署的伺服器基礎結構提供依照優先順序排列的具體建議清單。 建議分為六個焦點領域，它們可以幫助您快速了解風險並採取修正動作。

智慧套件提供的建議乃源自 Microsoft 工程師數千次客戶拜訪所得到的知識和經驗。 每項建議均提供問題的影響層面，以及如何實作建議變更等相關指引。

您可以選擇對組織而言最重要的焦點區域，同時追蹤經營無風險且健康狀態良好之環境的進度。

加入方案且評估完成之後，系統會將焦點區域的摘要資訊顯示在環境之基礎結構的 [SQL 健康情況檢查] 儀表板中。 下列章節說明如何使用 [SQL 健康情況檢查] 儀表板上的資訊，您可以在這裡檢視 SQL Server 基礎結構的建議動作並予以實施。

![SQL 健康情況檢查圖格的影像](./media/log-analytics-sql-assessment/sql-healthcheck-summary-tile.png)

![SQL 健康情況檢查儀表板的影像](./media/log-analytics-sql-assessment/sql-healthcheck-dashboard-01.png)

## <a name="prerequisites"></a>必要條件

* SQL 健康情況檢查方案需要在已安裝 Microsoft Monitoring Agent (MMA) 的每部電腦上安裝 .NET Framework 4 的支援版本。  MMA 代理程式會由 System Center 2016 - Operations Manager 和 Operations Manager 2012 R2 及 Log Analytics 服務使用。  
* 方案支援 SQL Server 2012、2014 和 2016 版本。
* Log Analytics 工作區，可以從 Azure 入口網站中的 Azure 市集將 SQL 健康情況檢查方案新增至此。  若要安裝方案，您必須是 Azure 訂用帳戶中的系統管理員或參與者。

  > [!NOTE]
  > 您加入方案之後，AdvisorAssessment.exe 檔案會以代理程式加入伺服器中。 組態資料會先讀取後再傳送至雲端中的 Log Analytics 服務，以便進行處理。 會將邏輯套用至接收的資料，且雲端服務會記錄資料。
  >
  >

若要對您的 SQL Server 伺服器執行健康情況檢查，則這些伺服器需要代理程式，並且須連線到 Log Analytics，您可以使用下列其中一個支援方法來達成要求：

1. 如果 System Center 2016 - Operations Manager 或 Operations Manager 2012 R2 已不再監視伺服器，則安裝 [Microsoft Monitoring Agent (MMA)](log-analytics-windows-agent.md)。
2. 如果伺服器受到 System Center 2016 - Operations Manager 或 Operations Manager 2012 R2 監視，而管理群組未與 Log Analytics 服務整合，則該伺服器可以是具有 Log Analytics 的多重主目錄，以便收集資料並轉送至該服務，且仍然受到 Operations Manager 監視。  
3. 除此之外，如果您的 Operations Manager 管理群組已與服務整合，則在工作區中啟用方案後，您需要讓服務依循[新增代理程式的受控電腦](log-analytics-om-agents.md#connecting-operations-manager-to-log-analytics)下的步驟，來新增網域控制站以收集資料。  

SQL Server 上的代理程式會向 Operations Manager 管理群組報告、收集資料、轉送至其指派的管理伺服器，然後直接從管理伺服器傳送至 Log Analytics 服務。  資料並不會寫入 Operations Manager 資料庫。  

如果 SQL Server 由 Operations Manager 監視，則您需要設定 Operations Manager 執行身分帳戶。 如需詳細資訊，請參閱底下的 [Log Analytics 的 Operations Manager 執行身分帳戶](#operations-manager-run-as-accounts-for-log-analytics)。

## <a name="sql-health-check-data-collection-details"></a>SQL 健康情況檢查的資料收集詳細資料
SQL 健康情況檢查會使用您已啟用的代理程式，從下列來源收集資料：

* Windows Management Instrumentation (WMI)
* 登錄
* 效能計數器
* SQL Server 動態管理檢視結果

資料會收集到 SQL Server，並每隔七天轉送給 Log Analytics。

## <a name="operations-manager-run-as-accounts-for-log-analytics"></a>Log Analytics 的 Operations Manager 執行身分帳戶
Log Analytics 會使用 Operations Manager 代理程式及管理群組來收集資料，並將資料傳送給 Log Analytics 服務。 Log Analytics 會建立工作負載的管理套件以提供加值服務。 每個工作負載都需要具有特定的工作負載權限，才能在不同的安全性內容中執行管理套件，例如網域使用者帳戶。 您需要藉由設定 Operations Manager 執行身分帳戶來提供認證資訊。

請使用下列資訊來設定 SQL 健康情況檢查的 Operations Manager 執行身分帳戶。

### <a name="set-the-run-as-account-for-sql-health-check"></a>設定 SQL 健康情況檢查的執行身分帳戶
 如果您已經在使用 SQL Server 管理套件，您應該使用該執行身分的設定。

#### <a name="to-configure-the-sql-run-as-account-in-the-operations-console"></a>在 Operations 主控台中設定 SQL 執行身分帳戶
> [!NOTE]
> 依預設，管理組件中的工作流程會在本機系統帳戶的安全性內容中執行。 如果您使用 Microsoft Monitoring Agent 直接連線到服務，而非直接向 Operations Manager 管理群組報告，請略過以下步驟 1-5，並執行 T-SQL 或 PowerShell 範例，指定 AUTHORITY\SYSTEM 為使用者名稱。
>
>

1. 在 Operations Manager 中開啟 Operations 主控台，然後按一下 [管理] 。
2. 在 [執行身分組態] 下方，按一下 [設定檔]，並開啟 [OMS SQL 評估執行身分設定檔]。
3. 在 [執行身分帳戶] 頁面上，按一下 [新增]。
4. 選取包含 SQL Server 所需認證的 Windows 執行身分帳戶，或按一下 [新增]  建立一個。

   > [!NOTE]
   > 執行身分帳戶類型必須是 Windows。 執行身分帳戶也必須屬於裝載 SQL Server 執行個體的所有 Windows 伺服器上的本機系統管理員群組。
   >
   >
5. 按一下 [檔案] 。
6. 修改，然後在每個 SQL Server 執行個體上執行下列 T-SQL 範例，授與執行身分帳戶所需的最小權限以執行 SQL 健康情況檢查。 不過，如果執行身分帳戶已是 SQL Server 執行個體上 sysadmin 伺服器角色的一部分，您就不需要這樣做。

```
    ---
    -- Replace <UserName> with the actual user name being used as Run As Account.
    USE master

    -- Create login for the user, comment this line if login is already created.
    CREATE LOGIN [<UserName>] FROM WINDOWS

    -- Grant permissions to user.
    GRANT VIEW SERVER STATE TO [<UserName>]
    GRANT VIEW ANY DEFINITION TO [<UserName>]
    GRANT VIEW ANY DATABASE TO [<UserName>]

    -- Add database user for all the databases on SQL Server Instance, this is required for connecting to individual databases.
    -- NOTE: This command must be run anytime new databases are added to SQL Server instances.
    EXEC sp_msforeachdb N'USE [?]; CREATE USER [<UserName>] FOR LOGIN [<UserName>];'

```

#### <a name="to-configure-the-sql-run-as-account-using-windows-powershell"></a>使用 Windows PowerShell 設定 SQL 執行身分帳戶
以您的資訊更新它之後，開啟 PowerShell 視窗並執行下列指令碼：

```
    import-module OperationsManager
    New-SCOMManagementGroupConnection "<your management group name>"

    $profile = Get-SCOMRunAsProfile -DisplayName "OMS SQL Assessment Run As Profile"
    $account = Get-SCOMrunAsAccount | Where-Object {$_.Name -eq "<your run as account name>"}
    Set-SCOMRunAsProfile -Action "Add" -Profile $Profile -Account $Account
```

## <a name="understanding-how-recommendations-are-prioritized"></a>了解建議的排列方式
智慧套件會為每項建議指派加權值，該值能顯現建議的相對重要性。 唯有重要性排行前十名的建議會出現在清單中。

### <a name="how-weights-are-calculated"></a>加權的計算方式
加權是彙集以下三個重要因素的值：

* 識別之疑難引發問題的 *機率* 。 機率較高等同於建議的整體分數較高。
* 疑難對組織的 *影響力* (如果確實引發問題)。 影響力較高等同於建議的整體分數較高。
* 實作建議所需的 *勞力* 。 勞力較高等同於建議的整體分數較低。

每項建議之加權的表示採用每個焦點區域之總分的百分比。 例如，如果針對安全性和法務遵循焦點區域之建議的分數為 5%，代表實作該項建議能增加 5% 的安全性和法務遵循整體分數。

### <a name="focus-areas"></a>焦點區域
**安全性和法務遵循** - 這個重點區域會顯示下列項目的建議：潛在安全性威脅和填補缺口、公司原則，以及技術、法律和法務遵循要求。

**可用性和業務續航力** - 這個重點區域會顯示下列項目的建議：服務可用性、基礎結構備援和企業保護。

**效能和延展性** - 這個重點區域會顯示建議來協助貴組織的 IT 基礎結構成長、確定您的 IT 環境是否符合目前的效能需求，而且能夠回應不斷變動的基礎結構需求。

**升級、移轉和部署** - 這個重點區域會顯示建議來協助您升級、移轉和將 SQL Server 部署到您的現有基礎結構。

**作業和監視** - 這個重點區域會顯示建議來協助您的 IT 作業更加順暢、執行預防性維護並將效能最大化。

**變更和組態管理** - 這個重點區域會顯示建議來協助保護每日作業，確保變更不會對您的基礎結構造成負面影響，同時建立變更控管程序，並追蹤及審核系統組態。

### <a name="should-you-aim-to-score-100-in-every-focus-area"></a>我應該為每個焦點區域訂定 100% 的分數嗎？
不一定。 建議乃源自 Microsoft 工程師上千次客戶拜訪所得到的知識和經驗。 然而，世界上沒有兩個一模一樣的伺服器基礎結構，因此特定建議與您的關聯性可能會有所增減。 例如，如果您的虛擬機器並未暴露在網際網路中，某些安全性建議的關聯性就會降低。 對於提供低優先順序臨機操作資料收集和報告的服務來說，某些可用性建議的關聯性就會降低。 會對成熟企業造成重大影響的問題，不見得會對新公司造成同等嚴重的影響。 因此，建議您先找出自己的優先焦點區域，然後觀察一段時間內的分數變化。

每項建議都包含其重要性的指引。 在已知 IT 服務之本質和組織之商務需求的情況下，您應使用該指引來評估實作建議的適當性。

## <a name="use-health-check-focus-area-recommendations"></a>使用健康情況檢查焦點區域建議
在使用 Log Analytics 中的評估方案之前，您必須先安裝方案。  安裝之後，您可以在 Azure 入口網站中的方案頁面上，使用 SQL 健康情況檢查圖格來檢視建議摘要。

檢視基礎結構的總結法務遵循評估結果，然後再深入鑽研建議事項。

### <a name="to-view-recommendations-for-a-focus-area-and-take-corrective-action"></a>檢視的焦點區域的建議並採取更正措施
1. 在 [https://portal.azure.com](https://portal.azure.com) 上登入 Azure 入口網站。
2. 在 Azure 入口網站中，按一下左下角的 [更多服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。
3. 在 [Log Analytics 訂用帳戶] 窗格中，選取工作區，然後按一下 [概觀] 圖格。  
4. 在 [概觀] 頁面上，按一下 [SQL 健康情況檢查] 圖格。
5. 在 [健康情況檢查] 頁面中檢閱任一焦點區域分葉中的摘要資訊，然後按一下焦點區域以檢視建議。
6. 在任一焦點區域頁面中，您可以檢視針對環境且按照優先順序排列的建議。 按一下 [受影響的物件]  下方的建議，可檢視建議提出原因的詳細資料。<br><br> ![SQL 健康情況檢查建議的影像](./media/log-analytics-sql-assessment/sql-healthcheck-dashboard-02.png)<br>
7. 您可以採取 [建議動作] 中所建議的更正動作。 當您解決某個項目後，後續評估會記錄您實施的建議動作並提高法務遵循分數。 更正後的項目將以**通過的物件**呈現。

## <a name="ignore-recommendations"></a>忽略建議
如果您有想要忽略的建議，則可以建立 Log Analytics 將用來防止建議出現在您評估結果的文字檔。

### <a name="to-identify-recommendations-that-you-will-ignore"></a>識別您將忽略的建議
1. 在 Azure 入口網站中的 Log Analytics 工作區頁面上，針對您選取的工作區，按一下 [記錄搜尋] 圖格。
2. 使用下列查詢來列出您環境中電腦的失敗建議。

    ```
    Type=SQLAssessmentRecommendation RecommendationResult=Failed | select Computer, RecommendationId, Recommendation | sort Computer
    ```

    >[!NOTE]
    > 如果您的工作區已升級為[新的 Log Analytics 查詢語言](log-analytics-log-search-upgrade.md)，則以上查詢會變更如下。
    >
    > `SQLAssessmentRecommendation | where RecommendationResult == "Failed" | sort by Computer asc | project Computer, RecommendationId, Recommendation`

    以下是顯示記錄檔搜尋查詢的螢幕擷取畫面︰<br><br> ![失敗的建議](./media/log-analytics-sql-assessment/sql-assess-failed-recommendations.png)<br>

3. 選擇您想要忽略的建議。 您將使用下一個程序中的 RecommendationId 值。

### <a name="to-create-and-use-an-ignorerecommendationstxt-text-file"></a>建立及使用 IgnoreRecommendations.txt 文字檔案
1. 建立名為 IgnoreRecommendations.txt 的檔案。
2. 在個別行上貼上或輸入您想要 Log Analytics 忽略之每個建議的各個 RecommendationId，然後儲存並關閉檔案。
3. 將檔案放在您想要 Log Analytics 忽略建議之每一部電腦的下列資料夾中。
   * 在具有 Microsoft Monitoring Agent 的電腦 (直接連線或透過 Operations Manager 連線) 上 - *SystemDrive*:\Program Files\Microsoft Monitoring Agent\Agent
   * 在 Operations Manager 管理伺服器上 - *SystemDrive*:\Program Files\Microsoft System Center 2012 R2\Operations Manager\Server
   * 在 Operations Manager 2016 管理伺服器上 - *SystemDrive*:\Program Files\Microsoft System Center 2016\Operations Manager\Server

### <a name="to-verify-that-recommendations-are-ignored"></a>驗證已忽略建議
1. 在執行下一個排定的評估之後，依預設是每隔 7 天執行一次，指定的建議會標示為忽略，且不會出現在評估儀表板。
2. 您可以使用下列記錄搜尋查詢列出所有已忽略的建議。

    ```
    Type=SQLAssessmentRecommendation RecommendationResult=Ignored | select Computer, RecommendationId, Recommendation | sort Computer
    ```

    >[!NOTE]
    > 如果您的工作區已升級為[新的 Log Analytics 查詢語言](log-analytics-log-search-upgrade.md)，則以上查詢會變更如下。
    >
    > `SQLAssessmentRecommendation | where RecommendationResult == "Ignored" | sort by Computer asc | project Computer, RecommendationId, Recommendation`

3. 如果您稍後決定想要查看忽略的建議，請移除任何 IgnoreRecommendations.txt 檔案，或從中移除 RecommendationID。

## <a name="sql-health-check-solution-faq"></a>SQL 健康情況檢查方案常見問題集
*執行健康情況檢查的頻率為何？*

* 每隔七天執行檢查。

是否有設定檢查執行頻率的方法？

* 目前沒有。

*如果我在加入 SQL 健康情況檢查方案後探索到另一部伺服器，方案也會檢查這部伺服器嗎？*

* 是的。在探索到該伺服器之後，會每隔 7 天檢查一次該伺服器。

*如果把伺服器除役，何時能將它從健康情況檢查中移除？*

* 如果伺服器在 3 週內未提交任何資料，智慧套件便會將其移除。

*負責收集資料之處理序的名稱為何？*

* AdvisorAssessment.exe

*收集資料需要花費多少時間？*

* 伺服器上的實際資料收集需費時約 1 小時。 對於擁有大量 SQL 執行個體或資料庫的伺服器，資料收集可能需要花費更久的時間。

*收集的資料類型為何？*

* 收集的資料類型如下：
  * WMI
  * 登錄
  * 效能計數器
  * SQL 動態管理檢視 (DMV)。

*是否有設定資料收集時間的方法？*

* 目前沒有。

*為什麼我必須設定執行身分帳戶？*

* 智慧套件會針對 SQL Server 執行少量的 SQL 查詢。 為了要執行 SQL 查詢，智慧套件必須使用具備「檢視伺服器狀態」權限的執行身分帳戶。  此外，為了要查詢 WMI，智慧套件還需要本機系統管理員認證。

*為什麼只顯示前 10 項建議？*

* 與其提供鉅細靡遺的工作清單，我們建議您先著重於解決優先建議事項。 解決後，智慧套件將會提供其他建議。 如果您想要查看詳細清單，可以使用 Log Analytics 記錄搜尋來檢視所有建議。

*是否有忽略建議的方法？*

* 是，請參閱上面的 [忽略建議](#ignore-recommendations) 一節。

## <a name="next-steps"></a>後續步驟
* [搜尋記錄](log-analytics-log-searches.md)可讓您了解如何分析詳細的 SQL 健康情況檢查資料和建議。
