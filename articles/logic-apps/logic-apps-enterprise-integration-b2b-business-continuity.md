---
title: B2B 整合帳戶的災害復原 - Azure 邏輯應用程式 | Microsoft Docs
description: 準備好在 Azure Logic Apps 中進行跨區域的災害復原
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: cf44af18-1fe5-41d5-9e06-cc57a968207c
ms.date: 04/10/2017
ms.openlocfilehash: 3d465123f814887282bf2b29a5b6e0836601c243
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43123899"
---
# <a name="cross-region-disaster-recovery-for-b2b-integration-accounts-in-azure-logic-apps"></a>Azure Logic Apps 中 B2B 整合帳戶的跨區域災害復原

B2B 工作負載涉及金錢交易，例如訂單和發票。 在災害事件期間，企業務必快速復原，才可符合與合作夥伴達成的商務層級 SLA。 本文示範如何建置 B2B 工作負載的商務持續性計劃。 

* 災害復原整備 
* 在災難事件期間容錯移轉到次要區域 
* 發生災難事件後切換回主要區域

## <a name="disaster-recovery-readiness"></a>災害復原整備  

1. 識別次要地區，並在次要地區中建立[整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)。

2. 對於執行狀態需要複寫到次要區域整合帳戶的必要訊息流程，新增夥伴、結構描述和合約。

   > [!TIP]
   > 請確定區域之間的整合帳戶成品命名慣例保持一致。 

3. 若要提取主要區域的執行狀態，請在次要地區中建立邏輯應用程式。 

   這個邏輯應用程式應該有觸發程序和動作。 
   觸發程序應該與主要區域整合帳戶連線，而且動作應該與次要地區整合帳戶連線。 
   根據時間間隔，觸發程序會輪詢主要區域執行狀態資料表，並提取新的記錄 (如果有)。 動作會將它們更新至次要地區整合帳戶。 
   這有助於從主要區域將累加式執行階段狀態更新到次要區域。

4. 邏輯應用程式整合帳戶中的商務持續性是設計來按照 B2B 通訊協定 (X12、AS2 和 EDIFACT) 進行支援。 若要了解詳細的步驟，請選取個別的連結。

5. 建議也將所有主要區域的資源部署在次要地區。 

   主要區域的資源包括 Azure SQL Database 或 Azure Cosmos DB、Azure 服務匯流排和用來傳訊的 Azure 事件中樞，Azure API 管理，以及 Azure App Service 的 Azure Logic Apps 功能。   

6. 建立從主要區域到次要地區的連線。 若要提取主要區域的執行狀態，請在次要地區中建立邏輯應用程式。 

   這個邏輯應用程式應該有觸發程序和動作。 
   觸發程序需連線至主要區域整合帳戶。 
   動作需連線至次要地區整合帳戶。 
   根據時間間隔，觸發程序會輪詢主要區域執行狀態資料表，並提取新的記錄 (如果有)。 
   動作會將它們更新至次要地區整合帳戶。 
   此程序有助於將累加式執行階段狀態從主要區域更新到次要地區。

Logic Apps 整合帳戶中的商務持續性會根據 B2B 通訊協定 X12、AS2 和 EDIFACT 提供支援。 如需使用 X12 和 AS2 的詳細步驟，請參閱本文中的 [X12](../logic-apps/logic-apps-enterprise-integration-b2b-business-continuity.md#x12) 和 [AS2](../logic-apps/logic-apps-enterprise-integration-b2b-business-continuity.md#as2)。

## <a name="fail-over-to-a-secondary-region-during-a-disaster-event"></a>在災難事件期間容錯移轉到次要地區

在災難事件期間，沒有商務持續性的主要區域可用時，會將流量導向次要區域。 次要地區可協助企業將職能快速復原，來符合與其合作夥伴達成的 RPO/RTO。 此外，從一個區域容錯移轉至另一個區域僅需要最低程度的作業過程。 

將控制編號從主要區域複製到次要地區會有預期的延遲。 若要避免在災害事件期間將重複產生的控制編號傳送給合作夥伴，建議使用 [PowerShell Cmdlet](https://blogs.msdn.microsoft.com/david_burgs_blog/2017/03/09/fresh-of-the-press-new-azure-powershell-cmdlets-for-upcoming-x12-connector-disaster-recovery) 在次要地區合約中增加控制編號。

## <a name="fall-back-to-a-primary-region-post-disaster-event"></a>發生災難事件後切換回主要區域

若要在可用時切換回主要區域，請遵循下列步驟︰

1. 停止接受次要地區中的合作夥伴發出的訊息。  

2. 使用 [PowerShell Cmdlet](https://blogs.msdn.microsoft.com/david_burgs_blog/2017/03/09/fresh-of-the-press-new-azure-powershell-cmdlets-for-upcoming-x12-connector-disaster-recovery) 對所有主要區域合約增加所產生的控制編號。  

3. 將資料流從次要地區導向主要區域。

4. 查看是否已啟用在次要地區中建立邏輯應用程式以從主要區域提取執行狀態。

## <a name="x12"></a>X12 

EDI X12 文件的商務持續性是根據控制編號：

> [!TIP]
> 您也可以使用 [X12 快速啟動範本](https://azure.microsoft.com/resources/templates/201-logic-app-b2b-disaster-recovery-replication/)來建立 Logic Apps。 建立主要和次要整合帳戶是使用範本的必要條件。 這個範本可讓您建立 2 個 Logic Apps，一個用於接收的控制編號，另一個用於產生的控制編號。 會在 Logic Apps 中建立個別的觸發程序和動作，將觸發程序連線至主要整合帳戶，並將動作連線至次要整合帳戶。

**先決條件**

若要啟用輸入訊息的災害復原，請選取 X12 合約之 [接收設定] 中的重複檢查設定。

![選取重複檢查設定](./media/logic-apps-enterprise-integration-b2b-business-continuity/dupcheck.png)  

1. 在次要地區中建立[邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。    

2. 搜尋 **X12**，並選取 [X12 - 當控制編號修改時]。   

   ![搜尋 x12](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn1.png)

   觸發程序會提示您建立整合帳戶的連線。 
   觸發程序需連線至主要區域整合帳戶。

3. 輸入連線名稱，選取清單中的 [主要區域整合帳戶]，然後選擇 [建立]。   

   ![主要區域整合帳戶名稱](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn2.png)

4. **開始控制編號同步處理的日期時間**設定是選用的。 **頻率**可以設為**天**、**小時**、**分鐘**或**秒**的間隔。   

   ![日期時間和頻率](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn3.png)

5. 選取 [新增步驟] > [新增動作]。

   ![選取 [新增步驟]，然後選取 [新增動作]](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn4.png)

6. 搜尋 **X12**，並選取 [X12 - 新增或更新控制編號]。   

   ![新增或更新控制編號](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn5.png)

7. 若要將動作連線到次要地區整合帳戶，請選取 [變更連線] > [新增新的連線]，取得可用整合帳戶的清單。 輸入連線名稱，選取清單中的 [次要地區整合帳戶]，然後選擇 [建立]。 

   ![次要地區整合帳戶名稱](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn6.png)

8. 按一下右上角的圖示，以切換到原始輸入。

   ![切換到原始輸入](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12rawinputs.png)

9. 從動態內容選擇器選取主體，並儲存邏輯應用程式。

   ![動態內容欄位](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn7.png)

   根據時間間隔，觸發程序會輪詢主要區域接收的控制編號資料表，並提取新的記錄。 
   動作會更新次要地區整合帳戶中的記錄。 
   如果沒有更新，則觸發程序狀態將顯示為**忽略**。   

   ![控制編號資料表](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12recevicedcn8.png)

根據時間間隔，累加式執行階段狀態會從主要區域複寫至次要地區。 在災難事件期間，沒有主要區域可用時，會對於商務持續性將流量導向次要區域。 

## <a name="edifact"></a>EDIFACT 

EDI EDIFACT 文件的商務持續性是根據控制編號。

**先決條件**

若要啟用輸入訊息的災害復原，請選取 EDIFACT 合約之 [接收設定] 中的重複檢查設定。

![選取重複檢查設定](./media/logic-apps-enterprise-integration-b2b-business-continuity/edifactdupcheck.png)  

1. 在次要地區中建立[邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。    

2. 搜尋 **EDIFACT**，並選取 [EDIFACT - 當控制編號修改時]。

   ![搜尋 EDIFACT](./media/logic-apps-enterprise-integration-b2b-business-continuity/edifactcn1.png)

   觸發程序會提示您建立整合帳戶的連線。 
   觸發程序需連線至主要區域整合帳戶。 

3. 輸入連線名稱，選取清單中的 [主要區域整合帳戶]，然後選擇 [建立]。    

   ![主要區域整合帳戶名稱](./media/logic-apps-enterprise-integration-b2b-business-continuity/X12CN2.png)

4. **開始控制編號同步處理的日期時間**設定是選用的。 **頻率**可以設為**天**、**小時**、**分鐘**或**秒**的間隔。    

   ![日期時間和頻率](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn3.png)

6. 選取 [新增步驟] > [新增動作]。    

   ![選取 [新增步驟]，然後選取 [新增動作]](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn4.png)

7. 搜尋 **EDIFACT**，並選取 [EDIFACT - 新增或更新控制編號]。   

   ![新增或更新控制編號](./media/logic-apps-enterprise-integration-b2b-business-continuity/EdifactChooseAction.png)

8. 若要將動作連線到次要地區整合帳戶，請選取 [變更連線] > [新增新的連線]，取得可用整合帳戶的清單。 輸入連線名稱，選取清單中的 [次要地區整合帳戶]，然後選擇 [建立]。

   ![次要地區整合帳戶名稱](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn6.png)

9. 按一下右上角的圖示，以切換到原始輸入。

   ![切換到原始輸入](./media/logic-apps-enterprise-integration-b2b-business-continuity/Edifactrawinputs.png)

10. 從動態內容選擇器選取主體，並儲存邏輯應用程式。   

   ![動態內容欄位](./media/logic-apps-enterprise-integration-b2b-business-continuity/X12CN7.png)

   根據時間間隔，觸發程序會輪詢主要區域接收的控制編號資料表，並提取新的記錄。
   動作會將記錄更新至次要地區整合帳戶。 
   如果沒有更新，則觸發程序狀態將顯示為**忽略**。

   ![控制編號資料表](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12recevicedcn8.png)

根據時間間隔，累加式執行階段狀態會從主要區域複寫至次要地區。 在災難事件期間，沒有主要區域可用時，會對於商務持續性將流量導向次要區域。 

## <a name="as2"></a>AS2 

使用 AS2 通訊協定的文件之商務持續性是根據訊息識別碼和 MIC 值。

> [!TIP]
> 您也可以使用 [AS2 快速啟動範本](https://github.com/Azure/azure-quickstart-templates/pull/3302)建立 Logic Apps。 建立主要和次要整合帳戶是使用範本的必要條件。 該範本可協助建立有觸發程序和動作的邏輯應用程式。 邏輯應用程式會建立從觸發程序到主要整合帳戶的連線，以及動作到建立次要整合帳戶的連線。

1. 在次要地區中建立[邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。  

2. 搜尋 **AS2**，並選取 [AS2 - 建立 MIC 值時]。   

   ![搜尋 AS2](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid1.png)

   觸發程序會提示您建立整合帳戶的連線。 
   觸發程序需連線至主要區域整合帳戶。 
   
3. 輸入連線名稱，選取清單中的 [主要區域整合帳戶]，然後選擇 [建立]。

   ![主要區域整合帳戶名稱](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid2.png)

4. **開始 MIC 值同步處理的日期時間**設定是選用的。 **頻率**可以設為**天**、**小時**、**分鐘**或**秒**的間隔。   

   ![日期時間和頻率](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid3.png)

5. 選取 [新增步驟] > [新增動作]。  

   ![選取 [新增步驟]，然後選取 [新增動作]](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid4.png)

6. 搜尋 **AS2**，並選取 [AS2 - 新增或更新 MIC 內容]。  

   ![MIC 新增或更新](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid5.png)

7. 若要將動作連線到次要整合帳戶，請選取 [變更連線] > [新增新的連線]，取得可用整合帳戶的清單。 輸入連線名稱，選取清單中的 [次要地區整合帳戶]，然後選擇 [建立]。

   ![次要地區整合帳戶名稱](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid6.png)

8. 按一下右上角的圖示，以切換到原始輸入。

   ![切換到原始輸入](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2rawinputs.png)

9. 從動態內容選擇器選取主體，並儲存邏輯應用程式。   

   ![動態內容](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid7.png)

   根據時間間隔，觸發程序會輪詢主要區域資料表，並提取新的記錄。 動作會將它們更新至次要地區整合帳戶。 
   如果沒有更新，則觸發程序狀態將顯示為**忽略**。  

   ![主要區域資料表](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid8.png)

根據時間間隔，累加式執行階段狀態會從主要區域複寫至次要地區。 在災難事件期間，沒有主要區域可用時，會對於商務持續性將流量導向次要區域。 

## <a name="next-steps"></a>後續步驟

[監視 B2B 訊息](logic-apps-monitor-b2b-message.md)

