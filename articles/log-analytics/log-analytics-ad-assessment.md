---
title: 使用 Azure Log Analytics 最佳化 Active Directory 環境 | Microsoft Docs
description: 您可以使用 Active Directory 健康情況檢查方案定期評估環境的風險和健全狀況。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: 81eb41b8-eb62-4eb2-9f7b-fde5c89c9b47
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 10/27/2017
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 5b9bd5bf21f378f92b7dd6e027d1bb0c05620dd6
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37128838"
---
# <a name="optimize-your-active-directory-environment-with-the-active-directory-health-check-solution-in-log-analytics"></a>在 Log Analytics 中使用 Active Directory 健康情況檢查方案來最佳化 Active Directory 環境

![AD 健康情況檢查標誌](./media/log-analytics-ad-assessment/ad-assessment-symbol.png)

您可以使用 Active Directory 健康情況檢查方案定期評估伺服器環境的風險和健全狀況。 本文協助您安裝和使用此解決方案，讓您可以針對潛在問題採取修正動作。

此方案能針對已部署的伺服器基礎結構提供依照優先順序排列的具體建議清單。 建議分為四個焦點區域，它們可以幫助您快速了解風險並採取動作。

建議乃源自 Microsoft 工程師數千次客戶拜訪所得到的知識和經驗。 每項建議均提供問題的影響層面，以及如何實作建議變更等相關指引。

您可以選擇對組織而言最重要的焦點區域，同時追蹤經營無風險且健康狀態良好之環境的進度。

加入方案且檢查完成之後，系統會將焦點區域的摘要資訊顯示在環境之基礎結構的 [AD 健康情況檢查] 儀表板中。 下列章節說明如何使用 [AD 健康情況檢查] 儀表板上的資訊，您可以在這裡檢視 Active Directory 伺服器基礎結構的建議動作並予以實施。  

![AD 健康情況檢查圖格的影像](./media/log-analytics-ad-assessment/ad-healthcheck-summary-tile.png)

![AD 健康情況檢查儀表板的影像](./media/log-analytics-ad-assessment/ad-healthcheck-dashboard-01.png)

## <a name="prerequisites"></a>先決條件

* Active Directory 健康情況檢查方案需要在已安裝 Microsoft Monitoring Agent (MMA) 的每部電腦上安裝 .NET Framework 4.5.2 或以上的支援版本。  MMA 代理程式會由 System Center 2016 - Operations Manager 和 Operations Manager 2012 R2 及 Log Analytics 服務使用。
* 方案支援執行 Windows Server 2008 和 2008 R2、Windows Server 2012 和 2012 R2 及 Windows Server 2016 的網域控制站。
* Log Analytics 工作區，可以從 Azure 入口網站中的 Azure 市集將 Active Directory 健康情況檢查方案新增至此。  不需要進一步的組態。

  > [!NOTE]
  > 您加入方案之後，AdvisorAssessment.exe 檔案會以代理程式加入伺服器中。 組態資料會先讀取後再傳送至雲端中的 Log Analytics 服務，以便進行處理。 會將邏輯套用至接收的資料，且雲端服務會記錄資料。
  >
  >

若要對您的網域控制站執行健康情況檢查，以讓網域成員受到評估，則這些控制站需要代理程式，並且須連線到 Log Analytics，您可以使用下列其中一個支援方法來達成要求：

1. 如果 System Center 2016 - Operations Manager 或 Operations Manager 2012 R2 已不再監視網域控制站，則安裝 [Microsoft Monitoring Agent (MMA)](log-analytics-windows-agent.md)。
2. 如果控制站受到 System Center 2016 - Operations Manager 或 Operations Manager 2012 R2 監視，而管理群組未與 Log Analytics 服務整合，則該網域控制站可以是具有 Log Analytics 的多重主目錄，以便收集資料並轉送至該服務，且仍然受到 Operations Manager 監視。  
3. 除此之外，如果您的 Operations Manager 管理群組已與服務整合，則在工作區中啟用方案後，您需要讓服務依循[新增代理程式的受控電腦](log-analytics-om-agents.md#connecting-operations-manager-to-log-analytics)下的步驟，來新增網域控制站以收集資料。  

網域控制站上的代理程式會向 Operations Manager 管理群組報告、收集資料、轉送至其指派的管理伺服器，然後直接從管理伺服器傳送至 Log Analytics 服務。  資料並不會寫入 Operations Manager 資料庫。  

## <a name="active-directory-health-check-data-collection-details"></a>Active Directory 健康情況檢查資料收集詳細資訊

Active Directory 健康情況檢查會使用您已啟用的代理程式，從下列來源收集資料：

- 登錄
- LDAP
- .NET Framework
- 事件記錄檔
- Active Directory 服務介面 (ADSI)
- Windows PowerShell
- 檔案資料
- Windows Management Instrumentation (WMI)
- DCDIAG 工具 API
- 檔案複寫服務 (NTFRS) API
- 自訂 C# 程式碼

資料會收集到網域控制站，並每隔七天轉送給 Log Analytics。  

## <a name="understanding-how-recommendations-are-prioritized"></a>了解建議的排列方式
智慧套件會為每項建議指派加權值，該值能顯現建議的相對重要性。 只會顯示最重要的 10 個建議。

### <a name="how-weights-are-calculated"></a>加權的計算方式
加權是彙集以下三個重要因素的值：

* 識別之疑難引發問題的機率。 機率較高等同於建議的整體分數較高。
* 疑難對組織的 *影響力* (如果確實引發問題)。 影響力較高等同於建議的整體分數較高。
* 實作建議所需的 *勞力* 。 勞力較高等同於建議的整體分數較低。

每項建議之加權的表示採用每個焦點區域之總分的百分比。 例如，如果針對安全性和法務遵循焦點區域之建議的分數為 5%，代表實作該項建議能增加 5% 的安全性和法務遵循整體分數。

### <a name="focus-areas"></a>焦點區域
**安全性和法務遵循** - 這個重點區域會顯示下列項目的建議：潛在安全性威脅和填補缺口、公司原則，以及技術、法律和法務遵循要求。

**可用性和業務續航力** - 這個重點區域會顯示下列項目的建議：服務可用性、基礎結構備援和企業保護。

**效能和延展性** - 這個重點區域會顯示建議來協助貴組織的 IT 基礎結構成長、確定您的 IT 環境是否符合目前的效能需求，而且能夠回應不斷變動的基礎結構需求。

**升級、移轉和部署** - 這個焦點區域顯示建議來協助您升級、移轉和部署 Active Directory 到您的現有基礎結構。

### <a name="should-you-aim-to-score-100-in-every-focus-area"></a>我應該為每個焦點區域訂定 100% 的分數嗎？
不一定。 建議乃源自 Microsoft 工程師上千次客戶拜訪所得到的知識和經驗。 然而，世界上沒有兩個一模一樣的伺服器基礎結構，因此特定建議與您的關聯性可能會有所增減。 例如，如果您的虛擬機器並未暴露在網際網路中，某些安全性建議的關聯性就會降低。 對於提供低優先順序臨機操作資料收集和報告的服務來說，某些可用性建議的關聯性就會降低。 會對成熟企業造成重大影響的問題，不見得會對新公司造成同等嚴重的影響。 因此，建議您先找出自己的優先焦點區域，然後觀察一段時間內的分數變化。

每項建議都包含其重要性的指引。 在已知 IT 服務之本質和組織之商務需求的情況下，您應使用該指引來評估實作建議的適當性。

## <a name="use-health-check-focus-area-recommendations"></a>使用健康情況檢查焦點區域建議
安裝之後，您可以在 Azure 入口網站中的方案頁面上，使用健康情況檢查圖格來檢視建議摘要。

檢視基礎結構的總結法務遵循評估結果，然後再深入鑽研建議事項。

### <a name="to-view-recommendations-for-a-focus-area-and-take-corrective-action"></a>檢視的焦點區域的建議並採取更正措施
3. 按一下 Azure 入口網站中您 Log Analytics 工作區的 [概觀] 圖格。
4. 在 [概觀] 頁面上，按一下 [Active Directory 健康情況檢查] 圖格。
5. 在 [健康情況檢查] 頁面中檢閱任一焦點區域分葉中的摘要資訊，然後按一下焦點區域以檢視建議。
6. 在任一焦點區域頁面中，您可以檢視針對環境且按照優先順序排列的建議。 按一下 [受影響的物件]  下方的建議，可檢視建議提出原因的詳細資料。<br><br> ![健康情況檢查建議的影像](./media/log-analytics-ad-assessment/ad-healthcheck-dashboard-02.png)
7. 您可以採取 [建議動作] 中所建議的更正動作。 當您解決某個項目後，後續評估會記錄您實施的建議動作並提高法務遵循分數。 更正後的項目將以**通過的物件**呈現。

## <a name="ignore-recommendations"></a>忽略建議
如果您有想要忽略的建議，則可以建立 Log Analytics 將用來防止建議出現在您評估結果的文字檔。

### <a name="to-identify-recommendations-that-you-will-ignore"></a>識別您將忽略的建議
1. 在 Azure 入口網站中的 Log Analytics 工作區頁面上，針對您選取的工作區，按一下 [記錄搜尋] 圖格。
2. 使用下列查詢來列出您環境中電腦的失敗建議。

    ```
    ADAssessmentRecommendation | where RecommendationResult == "Failed" | sort by Computer asc | project Computer, RecommendationId, Recommendation
    ```
    以下是顯示記錄檔搜尋查詢的螢幕擷取畫面︰<br><br> ![失敗的建議](./media/log-analytics-ad-assessment/ad-failed-recommendations.png)

3. 選擇您想要忽略的建議。 您將使用下一個程序中的 RecommendationId 值。

### <a name="to-create-and-use-an-ignorerecommendationstxt-text-file"></a>建立及使用 IgnoreRecommendations.txt 文字檔案
1. 建立名為 IgnoreRecommendations.txt 的檔案。
2. 在個別行上貼上或輸入您想要 Log Analytics 忽略之每個建議的各個 RecommendationId，然後儲存並關閉檔案。
3. 將檔案放在您想要 Log Analytics 忽略建議之每一部電腦的下列資料夾中。
   * 在具有 Microsoft Monitoring Agent 的電腦 (直接連線或透過 Operations Manager 連線) 上 - *SystemDrive*:\Program Files\Microsoft Monitoring Agent\Agent
   * 在 Operations Manager 2012 R2 管理伺服器上 - *SystemDrive*:\Program Files\Microsoft System Center 2012 R2\Operations Manager\Server
   * 在 Operations Manager 2016 管理伺服器上 - *SystemDrive*:\Program Files\Microsoft System Center 2016\Operations Manager\Server

### <a name="to-verify-that-recommendations-are-ignored"></a>驗證已忽略建議
在執行下一個排定的健康情況檢查之後 (依預設是每隔 7 天)，指定的建議會標示為 [已略過]，且不會出現在儀表板中。

1. 您可以使用下列記錄搜尋查詢列出所有已忽略的建議。

    ```
    ADAssessmentRecommendation | where RecommendationResult == "Ignored" | sort by Computer asc | project Computer, RecommendationId, Recommendation
    ```

2. 如果您稍後決定想要查看忽略的建議，請移除任何 IgnoreRecommendations.txt 檔案，或從中移除 RecommendationID。

## <a name="ad-health-check-solutions-faq"></a>AD 健康情況檢查方案常見問題集
*執行健康情況檢查的頻率為何？*

* 每隔七天執行檢查。

*是否有設定健康情況檢查執行頻率的方法？*

* 目前沒有。

*如果我在加入健康情況檢查方案後探索到另一部伺服器，方案也會檢查這部伺服器嗎？*

* 是的。在探索到該伺服器之後，會每隔 7 天檢查一次該伺服器。

*如果把伺服器除役，何時能將它從健康情況檢查中移除？*

* 如果伺服器在 3 週內未提交任何資料，智慧套件便會將其移除。

*負責收集資料之處理序的名稱為何？*

* AdvisorAssessment.exe

*收集資料需要花費多少時間？*

* 伺服器上的實際資料收集需費時約 1 小時。 對於擁有大量 Active Directory 伺服器的伺服器，資料收集可能需要花費更久的時間。

*是否有設定資料收集時間的方法？*

* 目前沒有。

*為什麼只顯示前 10 項建議？*

* 與其提供鉅細靡遺的工作清單，我們建議您先著重於解決優先建議事項。 解決後，智慧套件將會提供其他建議。 如果您想要查看詳細清單，可以使用記錄檔搜尋來檢視所有建議。

*是否有忽略建議的方法？*

* 是，請參閱上面的 [忽略建議](#ignore-recommendations) 一節。

## <a name="next-steps"></a>後續步驟
* 使用[ Log Analytics 中的記錄搜尋](log-analytics-log-searches.md)，可讓您了解如何分析詳細的 AD 健康情況檢查資料和建議。
