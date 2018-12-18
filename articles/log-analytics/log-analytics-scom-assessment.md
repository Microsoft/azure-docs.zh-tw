---
title: 使用 Azure Log Analytics 最佳化 System Center Operations Manager 環境 | Microsoft Docs
description: 您可以使用 System Center Operations Manager 健康情況檢查解決方案，定期評估環境的風險和健康狀態。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: 49aad8b1-3e05-4588-956c-6fdd7715cda1
ms.service: log-analytics
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/25/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 7ce8afa04751cd38e64b9ed920a6f863781e3ad1
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43126276"
---
# <a name="optimize-your-environment-with-the-system-center-operations-manager-health-check-preview-solution"></a>使用 System Center Operations Manager 健康情況檢查 (預覽) 解決方案進行環境最佳化

![System Center Operations Manager 健康情況檢查符號](./media/log-analytics-scom-assessment/scom-assessment-symbol.png)

您可以使用 System Center Operations Manager 健康情況檢查解決方案，定期評估 System Center Operations Manager 管理群組的風險和健康狀態。 本文協助您安裝、設定和使用此解決方案，讓您可以針對潛在問題採取修正動作。

此方案能針對已部署的伺服器基礎結構提供依照優先順序排列的具體建議清單。 建議分為四個焦點領域，幫助您快速了解風險並採取修正動作。

智慧套件提供的建議乃源自 Microsoft 工程師數千次客戶拜訪所得到的知識和經驗。 每項建議均提供問題的影響層面，以及如何實作建議變更等相關指引。

您可以選擇對組織而言最重要的焦點區域，同時追蹤經營無風險且健康狀態良好之環境的進度。

新增解決方案並進行評定之後，焦點區域的摘要資訊會顯示在基礎結構的 [System Center Operations Manager 健康情況檢查] 儀表板。 下列章節說明如何使用 [System Center Operations Manager 健康情況檢查] 儀表板上的資訊，您可以在這裡檢視並採用針對 Operations Manager 環境建議的動作。

![System Center Operations Manager 解決方案圖格](./media/log-analytics-scom-assessment/log-analytics-scom-healthcheck-tile.png)

![System Center Operations Manager 健康情況檢查儀表板](./media/log-analytics-scom-assessment/log-analytics-scom-healthcheck-dashboard-01.png)

## <a name="installing-and-configuring-the-solution"></a>安裝和設定方案

此解決方案適用於 Microsoft System Operations Manager 2012 Service Pack (SP) 1 和 2012 R2。

請使用下列資訊來安裝和設定方案。

 - 在使用 Log Analytics 中的健康情況檢查解決方案之前，您必須先安裝解決方案。 從 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.SCOMAssessmentOMS?tab=Overview) 安裝解決方案。

 - 將解決方案新增至工作區之後，儀表板上的 [System Center Operations Manager 健康情況檢查] 圖格會顯示額外設定必要的訊息。 按一下圖格，然後遵循頁面中所述的設定步驟

 ![System Center Operations Manager 儀表板圖格](./media/log-analytics-scom-assessment/scom-configrequired-tile.png)

> [!NOTE]
> System Center Operations Manager 的組態可以使用指令碼完成，方法為遵循 Log Analytics 中解決方案組態頁面中所述的步驟。

 若要透過 Operations Manager Operations 主控台設定評定，請依下列順序執行下面的步驟：
1. [設定 System Center Operations Manager 健康情況檢查的執行身分帳戶](#operations-manager-run-as-accounts-for-log-analytics)  
2. [設定 System Center Operations Manager 健康情況檢查規則](#configure-the-assessment-rule)

## <a name="system-center-operations-manager-assessment-data-collection-details"></a>收集 System Center Operations Manager 評定資料的詳細資料

System Center Operations Manager 評定會從下列來源收集資料：

* 登錄
* Windows Management Instrumentation (WMI)
* 事件記錄檔
* 檔案資料
* 從您所指定的管理伺服器，透過 PowerShell 和 SQL 查詢直接從 Operations Manager 收集。  

資料會收集到管理伺服器，並每隔七天轉送給 Log Analytics。  

## <a name="operations-manager-run-as-accounts-for-log-analytics"></a>Log Analytics 的 Operations Manager 執行身分帳戶

Log Analytics 會建立工作負載的管理套件以提供加值服務。 每個工作負載都需要具有特定的工作負載權限，才能在不同的安全性內容中執行管理套件，例如網域使用者帳戶。 請使用具特殊權限的認證設定 Operations Manager 執行身分帳戶。 如需詳細資訊，請參閱 Operations Manager 文件中的[如何建立執行身分帳戶](https://technet.microsoft.com/library/hh321655(v=sc.12).aspx)。

請使用下列資訊來設定 System Center Operations Manager 健康情況檢查的 Operations Manager 執行身分帳戶。

### <a name="set-the-run-as-account"></a>設定執行身分帳戶

執行身分帳戶必須符合下列需求，才能繼續作業︰

* 屬於網域使用者帳戶，且此帳戶是所有支援任何 Operations Manager 角色(管理伺服器；裝載了作業、資料倉儲和 ACS 資料庫的 SQL Server；報告、Web 主控台和閘道伺服器) 之伺服器上的本機 Administrators 群組成員。
* 要評估的管理群組之 Operation Manager 系統管理員角色
* 如果帳戶沒有 SQL 系統管理員權限，則請執行[指令碼](#sql-script-to-grant-granular-permissions-to-the-run-as-account)，將細微權限授與給裝載了一個或所有 Operations Manager 資料庫之每個 SQL Server 執行個體上的帳戶。

1. 在 Operations Manager 主控台中，選取 [管理] 導覽按鈕。
2. 在 [執行身分設定] 下，按一下 [帳戶]。
3. 在 [建立執行身分帳戶] 精靈中，於 [簡介] 頁面上按 [下一步]。
4. 在 [一般屬性] 頁面上，於 [執行身分帳戶類型:] 清單中選取 [Windows]。
5. 在 [顯示名稱] 文字方塊中輸入顯示名稱，在 [說明] 方塊中選擇性地輸入說明，然後按 [下一步]。
6. 在 [散發安全性] 頁面上，選取 [較安全]。
7. 按一下頁面底部的 [新增] 。  

您現已建立執行身分帳戶，接下來您必須將其鎖定在管理群組中的管理伺服器，並與預先定義的執行身分設定檔相關聯，如此一來，工作流程才會使用認證來執行。  

1. 在 [執行身分設定]、[帳戶] 下，於 [結果] 窗格中對您稍早建立的帳戶連按兩下。
2. 在 [散發] 索引標籤上，按一下 [選取的電腦] 方塊的 [新增]，並新增要做為帳戶散發目標的管理伺服器。  按 [確定] 兩次以儲存變更。
3. 在 [執行身分設定] 下，按一下 [設定檔]。
4. 搜尋「SCOM 評定設定檔」。
5. 設定檔名稱應該是︰「Microsoft System Center Advisor SCOM 評定執行身分設定檔」。
6. 以滑鼠右鍵按一下其屬性並更新，然後新增您稍早建立的執行身分帳戶。

### <a name="sql-script-to-grant-granular-permissions-to-the-run-as-account"></a>授與細微權限給執行身分帳戶的 SQL 指令碼

執行下列 SQL 指令碼，將必要權限授與給裝載了作業、資料倉儲和 ACS 資料庫之 Operations Manager 所使用的 SQL Server 執行個體上的執行身分帳戶。

```
-- Replace <UserName> with the actual user name being used as Run As Account.
USE master

-- Create login for the user, comment this line if login is already created.
CREATE LOGIN [UserName] FROM WINDOWS


--GRANT permissions to user.
GRANT VIEW SERVER STATE TO [UserName]
GRANT VIEW ANY DEFINITION TO [UserName]
GRANT VIEW ANY DATABASE TO [UserName]

-- Add database user for all the databases on SQL Server Instance, this is required for connecting to individual databases.
-- NOTE: This command must be run anytime new databases are added to SQL Server instances.
EXEC sp_msforeachdb N'USE [?]; CREATE USER [UserName] FOR LOGIN [UserName];'

Use msdb
GRANT SELECT To [UserName]
Go

--Give SELECT permission on all Operations Manager related Databases

--Replace the Operations Manager database name with the one in your environment
Use [OperationsManager];
GRANT SELECT To [UserName]
GO

--Replace the Operations Manager DatawareHouse database name with the one in your environment
Use [OperationsManagerDW];
GRANT SELECT To [UserName]
GO

--Replace the Operations Manager Audit Collection database name with the one in your environment
Use [OperationsManagerAC];
GRANT SELECT To [UserName]
GO

--Give db_owner on [OperationsManager] DB
--Replace the Operations Manager database name with the one in your environment
USE [OperationsManager]
GO
ALTER ROLE [db_owner] ADD MEMBER [UserName]

```

### <a name="configure-the-health-check-rule"></a>設定健康情況檢查規則

System Center Operations Manager 健康情況檢查解決方案的管理套件包含名為「Microsoft System Center Advisor SCOM 評定執行評定規則」的規則。 此規則負責執行健康情況檢查。 若要啟用規則和設定頻率，請使用下列程序。

根據預設，Microsoft System Center Advisor SCOM 評定執行評定規則已停用。 若要執行健康情況檢查，您必須在管理伺服器上啟用此規則。 使用下列步驟。

#### <a name="enable-the-rule-for-a-specific-management-server"></a>針對特定的管理伺服器啟用此規則

1. 在 Operations Manager Operations 主控台的 [撰寫] 工作區中，在 [規則] 窗格中搜尋規則「Microsoft System Center Advisor SCOM 評定執行評定規則」。
2. 在搜尋結果中，選取包含文字「類型︰管理伺服器」的規則。
3. 以滑鼠右鍵按一下規則，然後按一下 [覆寫] > [針對以下類別的特定物件: 管理伺服器]。
4.  在可用的管理伺服器清單中，選取應該執行此規則的管理伺服器。  這應該是您先前所設定，要將執行身分帳戶與其產生關聯的同一個管理伺服器。
5.  針對 [已啟用] 參數值，務必將覆寫值變更為 [True]。<br><br> ![override parameter](./media/log-analytics-scom-assessment/rule.png)

    仍在此視窗中，使用下一個程序來設定執行頻率。

#### <a name="configure-the-run-frequency"></a>設定執行頻率

依預設，評定會設為每 10,080 分鐘 (或 7 天) 執行一次。 您可以將值覆寫為最小值 1440 分鐘 (或一天)。 此值代表連續執行評定之間所需的最短時間間隔。 若要覆寫間隔，請使用下列步驟。

1. 在 Operations Manager 主控台的 [撰寫] 工作區中，在 [規則] 區段中搜尋規則「Microsoft System Center Advisor SCOM 評定執行評定規則」。
2. 在搜尋結果中，選取包含文字「類型︰管理伺服器」的規則。
3. 以滑鼠右鍵按一下規則，然後按一下 [覆寫規則] > [針對以下類別的所有物件: 管理伺服器]。
4. 將 [間隔] 參數值變更為您想要的間隔值。 在下列範例中，此值設為 1440 分鐘 (一天)。<br><br> ![interval parameter](./media/log-analytics-scom-assessment/interval.png)<br>  

    如果此值設為 1440 分鐘內，則規則會每天執行一次。 在此範例中，此規則會忽略間隔值，且每天執行一次。


## <a name="understanding-how-recommendations-are-prioritized"></a>了解建議的排列方式

智慧套件會為每項建議指派加權值，該值能顯現建議的相對重要性。 只會顯示最重要的 10 個建議。

### <a name="how-weights-are-calculated"></a>加權的計算方式

加權是彙集以下三個重要因素的值：

- 識別之疑難引發問題的 *機率* 。 機率較高等同於建議的整體分數較高。
- 疑難對組織的 *影響力* (如果確實引發問題)。 影響力較高等同於建議的整體分數較高。
- 實作建議所需的 *勞力* 。 勞力較高等同於建議的整體分數較低。

每項建議之加權的表示採用每個焦點區域之總分的百分比。 例如，如果 [可用性與商務持續性] 焦點區域中的建議得分 5%，則實作該建議可讓整個「可用性與商務持續性」分數增加 5%。

### <a name="focus-areas"></a>焦點區域

**可用性和業務續航力** - 這個重點區域會顯示下列項目的建議：服務可用性、基礎結構備援和企業保護。

**效能和延展性** - 這個重點區域會顯示建議來協助貴組織的 IT 基礎結構成長、確定您的 IT 環境是否符合目前的效能需求，而且能夠回應不斷變動的基礎結構需求。

**升級、移轉和部署** - 這個焦點區域會顯示建議，協助您升級、移轉和將 SQL Server 部署至現有的基礎結構。

**作業和監視** - 這個重點區域會顯示建議來協助您的 IT 作業更加順暢、執行預防性維護並將效能最大化。

### <a name="should-you-aim-to-score-100-in-every-focus-area"></a>我應該為每個焦點區域訂定 100% 的分數嗎？

不一定。 建議乃源自 Microsoft 工程師上千次客戶拜訪所得到的知識和經驗。 然而，世界上沒有兩個一模一樣的伺服器基礎結構，因此特定建議與您的關聯性可能會有所增減。 例如，如果您的虛擬機器並未暴露在網際網路中，某些安全性建議的關聯性就會降低。 對於提供低優先順序臨機操作資料收集和報告的服務來說，某些可用性建議的關聯性就會降低。 會對成熟企業造成重大影響的問題，不見得會對新公司造成同等嚴重的影響。 因此，建議您先找出自己的優先焦點區域，然後觀察一段時間內的分數變化。

每項建議都包含其重要性的指引。 根據您的 IT 服務性質和組織的商務需求，請使用本指引來評估是否適合實作建議。

## <a name="use-health-check-focus-area-recommendations"></a>使用健康情況檢查焦點區域建議

在使用 Log Analytics 中的健康情況檢查解決方案之前，您必須先安裝解決方案。 若要深入了解如何安裝解決方案，請參閱[安裝管理解決方案](log-analytics-add-solutions.md)。 安裝之後，您可以在 Azure 入口網站中的工作區 [概觀] 頁面上，使用 [System Center Operations Manager 健全狀況檢查] 圖格來檢視建議摘要。

檢視基礎結構的總結法務遵循評估結果，然後再深入鑽研建議事項。

### <a name="to-view-recommendations-for-a-focus-area-and-take-corrective-action"></a>檢視的焦點區域的建議並採取更正措施
1. 在 [https://portal.azure.com](https://portal.azure.com) 上登入 Azure 入口網站。
2. 在 Azure 入口網站中，按一下左下角的 [更多服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。
3. 在 [Log Analytics 訂用帳戶] 窗格中，選取工作區，然後按一下 [工作區摘要] 功能表項目。  
4. 在 [概觀] 頁面上，按一下 [System Center Operations Manager 健康情況檢查] 圖格。
5. 在 [System Center Operations Manager 健康情況檢查] 頁面上，檢閱其中一個焦點區域刀鋒視窗中的摘要資訊，然後按一下一個刀鋒視窗來檢視該焦點區域的建議。
6. 在任一焦點區域頁面中，您可以檢視針對環境且按照優先順序排列的建議。 按一下 [受影響的物件]  下方的建議，可檢視建議提出原因的詳細資料。<br><br> ![focus area](./media/log-analytics-scom-assessment/log-analytics-scom-healthcheck-dashboard-02.png)<br>
7. 您可以採取 [建議動作] 中所建議的更正動作。 當您解決某個項目後，後續評估會記錄您實施的建議動作並提高法務遵循分數。 更正後的項目將以**通過的物件**呈現。

## <a name="ignore-recommendations"></a>忽略建議

如果您有想要忽略的建議，則可以建立供 Log Analytics 用來防止建議出現在您評估結果的文字檔。

### <a name="to-identify-recommendations-that-you-want-to-ignore"></a>若要識別您想要忽略的建議
1. 在 Azure 入口網站中的 Log Analytics 工作區頁面上，針對您選取的工作區，按一下 [記錄搜尋] 功能表項目。
2. 使用下列查詢來列出您環境中電腦的失敗建議。

    ```
    Type=SCOMAssessmentRecommendationRecommendationResult=Failed | select Computer, RecommendationId, Recommendation | sort Computer
    ```

    >[!NOTE]
    > 如果您的工作區已升級為[新的 Log Analytics 查詢語言](log-analytics-log-search-upgrade.md)，則以上查詢會變更如下。
    >
    > `SCOMAssessmentRecommendationRecommendation | where RecommendationResult == "Failed" | sort by Computer asc | project Computer, RecommendationId, Recommendation`

    以下是顯示記錄檔搜尋查詢的螢幕擷取畫面︰<br><br> ![log search](./media/log-analytics-scom-assessment/scom-log-search.png)<br>

3. 選擇您想要忽略的建議。 在下一個程序中，您將使用 RecommendationId 的值。

### <a name="to-create-and-use-an-ignorerecommendationstxt-text-file"></a>建立及使用 IgnoreRecommendations.txt 文字檔案

1. 建立名為 IgnoreRecommendations.txt 的檔案。
2. 在個別行上貼上或輸入您想要 Log Analytics 忽略之每個建議的各個 RecommendationId，然後儲存並關閉檔案。
3. 將檔案放在您想要 Log Analytics 忽略建議之每一部電腦的下列資料夾中。
4. 在 Operations Manager 管理伺服器上 - *SystemDrive*:\Program Files\Microsoft System Center 2012 R2\Operations Manager\Server。

### <a name="to-verify-that-recommendations-are-ignored"></a>驗證已忽略建議

1. 在執行下一個排定的評定之後 (依預設是每隔 7 天)，指定的建議會標示為 [已略過]，且不會出現在健康情況檢查儀表板中。
2. 您可以使用下列記錄搜尋查詢列出所有已忽略的建議。

    ```
    Type=SCOMAssessmentRecommendationRecommendationResult=Ignored | select  Computer, RecommendationId, Recommendation | sort  Computer
    ```

    >[!NOTE]
    > 如果您的工作區已升級為[新的 Log Analytics 查詢語言](log-analytics-log-search-upgrade.md)，則以上查詢會變更如下。
    >
    > `SCOMAssessmentRecommendationRecommendation | where RecommendationResult == "Ignore" | sort by Computer asc | project Computer, RecommendationId, Recommendation`

3. 如果您稍後決定想要查看忽略的建議，請移除任何 IgnoreRecommendations.txt 檔案，或從中移除 RecommendationID。

## <a name="system-center-operations-manager-health-check-solution-faq"></a>System Center Operations Manager 健康情況檢查解決方案常見問題集

*我已將健康情況檢查解決方案新增到 Log Analytics 工作區。但沒看到建議。為什麼？* 新增解決方案之後，請使用下列步驟在 Log Analytics 儀表板上檢視建議。  

- [設定 System Center Operations Manager 健康情況檢查的執行身分帳戶](#operations-manager-run-as-accounts-for-log-analytics)  
- [設定 System Center Operations Manager 健康情況檢查規則](#configure-the-health-check-rule)


是否有設定檢查執行頻率的方法？ 是。 請參閱[設定執行頻率](#configure-the-run-frequency)。

如果我在新增 System Center Operations Manager 評定解決方案之後探索了另一部伺服器，該伺服器也會受到檢查嗎？ 是，在探索之後，便會從那一刻起對它進行檢查，預設是每隔 7 天一次。

*負責收集資料之處理序的名稱為何？* AdvisorAssessment.exe

AdvisorAssessment.exe 程序在哪裡執行？ AdvisorAssessment.exe 會在啟用健康情況檢查規則的管理伺服器的 HealthService 處理序之下執行。 使用這個程序時，將會透過遠端資料收集來探索您的整個環境。

收集資料需要花費多少時間？ 在伺服器上資料收集需要花費約 1 小時。 在有許多 Operations Manager 執行個體或資料庫的環境中可能更久。

如果我將評定間隔設為少於 1440 分鐘會怎樣？ 評定已預先設定為最多一天執行一次。 如果您將間隔值覆寫為少於 1440 分鐘的值，則評定會使用 1440 分鐘做為間隔值。

如何知道是否未通過必要條件？ 如果健康情況檢查已執行，但您沒有看到結果，很可能是健康情況檢查的某些必要條件未通過。 您可以在記錄檔搜尋中執行查詢︰`Operation Solution=SCOMAssessment` 和 `SCOMAssessmentRecommendation FocusArea=Prerequisites`，以查看未通過的必要條件。

*必要條件未通過中出現 `Failed to connect to the SQL Instance (….).` 訊息。有什麼問題？* 用來收集資料的處理序 AdvisorAssessment.exe 會在管理伺服器的 HealthService 處理序下執行。 在健康情況檢查過程中，此處理序會嘗試連接至 Operations Manager 資料庫所在的 SQL Server。 當防火牆規則封鎖 SQL Server 執行個體的連接時，就會發生此錯誤。

*收集的資料類型為何？* 透過 Windows PowerShell、SQL 查詢和檔案資訊收集器收集的資料類型如下︰WMI 資料 - 登錄資料 - 事件記錄檔資料 - Operations Manager 資料。

*為什麼我必須設定執行身分帳戶？* Operations Manager 中會執行各種 SQL 查詢。 您必須使用具有必要權限的執行身分帳戶，它們才能夠執行。 此外，查詢 WMI 還需要本機系統管理員認證。

*為什麼只顯示前 10 項建議？* 與其列出鉅細靡遺的工作清單，我們建議您先專注於解決優先的建議。 解決後，智慧套件將會提供其他建議。 如果您想要查看詳細清單，可以使用記錄檔搜尋來檢視所有建議。

*是否有忽略建議的方法？* 是，請參閱[忽略建議](#Ignore-recommendations)。


## <a name="next-steps"></a>後續步驟

- [搜尋記錄](log-analytics-log-searches.md)可讓您了解如何分析詳細的 System Center Operations Manager 健康情況檢查資料和建議。
