---
title: Azure Log Analytics 中的 IT 服務管理連接器 | Microsoft Docs
description: 本文提供 IT Service Management Connector (ITSMC) 的概觀，以及說明如何使用此解決方案，在 Azure Log Analytics 集中監視及管理 ITSM 工作項目，並快速解決所有問題。
services: log-analytics
documentationcenter: ''
author: jyothirmaisuri
manager: riyazp
editor: ''
ms.assetid: 0b1414d9-b0a7-4e4e-a652-d3a6ff1118c4
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/24/2018
ms.author: v-jysur
ms.component: ''
ms.openlocfilehash: 5f5b03090f374f936e03d487596ca0462feea348
ms.sourcegitcommit: 3856c66eb17ef96dcf00880c746143213be3806a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48042423"
---
# <a name="connect-azure-to-itsm-tools-using-it-service-management-connector"></a>使用 IT Service Management Connector 將 Azure 連線到 ITSM 工具

![IT 服務管理連接器符號](./media/log-analytics-itsmc/itsmc-symbol.png)

IT Service Management Connector (ITSMC) 可讓您連線 Azure 和支援的 IT 服務管理 (ITSM) 產品/服務。

諸如 Log Analytics 和 Azure 監視器等 Azure 服務會提供工具，來偵測及分析您的 Azure 與非 Azure 資源問題並進行疑難排解。 不過，與問題相關的工作項目通常位於 ITSM 產品/服務中。 ITSM Connector 提供 Azure 與 ITSM 工具之間的雙向連線，以協助您更快速地解決問題。

ITSMC 支援與下列 ITSM 工具連線：

-   ServiceNow
-   System Center Service Manager
-   Provance
-   Cherwell

使用 ITSMC，您可以

-  根據您的 Azure 警示 (計量警示、活動記錄警示和 Log Analytics 警示)，在 ITSM 工具中建立工作項目。
-  您可以選擇將事件和變更要求資料從您的 ITSM 工具同步處理到 Azure Log Analytics 工作區。


您可以使用下列步驟，開始使用 ITSM Connector：

1.  [新增 ITSM Connector 解決方案](#adding-the-it-service-management-connector-solution)
2.  [建立 ITSM 連線](#creating-an-itsm-connection)
3.  [使用連線](#using-the-solution)


##  <a name="adding-the-it-service-management-connector-solution"></a>新增 IT 服務管理連接器解決方案

您需要先新增 ITSM Connector 解決方案，才能建立連線。

1.  在 Azure 入口網站中，按一下 [+新增] 圖示。

    ![Azure 新增資源](./media/log-analytics-itsmc/azure-add-new-resource.png)

2.  在 Marketplace 中搜尋 **IT Service Management Connector**，然後按一下 [建立]。

    ![新增 ITSMC 解決方案](./media/log-analytics-itsmc/add-itsmc-solution.png)

3.  在 [OMS 工作區] 區段中，選取您要安裝解決方案的 Azure Log Analytics 工作區。
4.  在 [OMS 工作區設定] 區段中，選取您要建立解決方案資源的資源群組。

    ![ITSMC 工作區](./media/log-analytics-itsmc/itsmc-solution-workspace.png)

5.  按一下頁面底部的 [新增] 。

部署解決方案資源時，通知會出現在視窗右上方。


## <a name="creating-an-itsm--connection"></a>建立 ITSM 連線

一旦您安裝解決方案之後，就可以建立連線。

若要建立連線，您將需要準備 ITSM 工具，以允許來自 ITSM Connector 解決方案的連線。  

根據您所連線的 ITSM 產品，使用下列步驟：

- [System Center Service Manager (SCSM)](log-analytics-itsmc-connections.md#connect-system-center-service-manager-to-it-service-management-connector-in-azure)
- [ServiceNow](log-analytics-itsmc-connections.md#connect-servicenow-to-it-service-management-connector-in-azure)
- [Provance](log-analytics-itsmc-connections.md#connect-provance-to-it-service-management-connector-in-azure)  
- [Cherwell](log-analytics-itsmc-connections.md#connect-cherwell-to-it-service-management-connector-in-azure)

一旦您備妥 ITSM 工具之後，請遵循下列步驟來建立連線：

1.  移至 [所有資源]、尋找 **ServiceDesk(YourWorkspaceName)**。
2.  在左窗格的 [工作區資料來源] 下方，按一下 [ITSM 連線]。
    ![ITSM 連線](./media/log-analytics-itsmc/itsm-connections.png)

    此頁面會顯示連線的清單。
3.  按一下 [ **加入連接**]。

    ![新增 ITSM 連線](./media/log-analytics-itsmc/add-new-itsm-connection.png)

4.  指定連線設定，如[設定與 ITSM 產品/服務的 ITSMC 連線](log-analytics-itsmc-connections.md)中所述。

    > [!NOTE]

    > 根據預設，ITSMC 每隔 24 小時就會將連線的設定資料重新整理一次。 若要針對所做的任何編輯或範本更新將連線的資料立即重新整理，按一下連線刀鋒視窗上的 [同步] 按鈕。

    ![連線重新整理](./media/log-analytics-itsmc/itsmc-connections-refresh.png)


## <a name="using-the-solution"></a>使用解決方案
   藉由使用 ITSM Connector 解決方案，您可以從 Azure 警示、Log Analytics 警示及 Log Analytics 記錄檔記錄中建立工作項目。

## <a name="create-itsm-work-items-from-azure-alerts"></a>從 Azure 警示建立 ITSM 工作項目

一旦建立 ITSM 連線之後，就可使用 [動作群組] 中的 [ITSM 動作]，根據 Azure 警示，在您的 ITSM 工具中建立工作項目。

動作群組提供以模組化且可重複使用的方式來針對 Azure 警示觸發動作。 您可以在 Azure 入口網站中，搭配計量警示、活動記錄警示及 Azure Log Analytics 警示使用動作群組。

請使用下列程序：

1. 在 Azure 入口網站中，按一下 [監視]。
2. 在左窗格中，按一下 [動作群組]。 [新增動作群組] 視窗隨即出現。

    ![動作群組](media/log-analytics-itsmc/action-groups.png)

3. 提供動作群組的 [名稱] 與 [簡稱]。 選取您要在其中建立動作群組的 [資源群組] 與 [訂用帳戶]。

    ![動作群組詳細資料](media/log-analytics-itsmc/action-groups-details.png)

4. 在 [動作] 清單中，從 [動作類型] 的下拉式功能表選取 [ITSM]。 提供動作的**名稱**，然後按一下 [編輯詳細資料]。
5. 選取 Log Analytics 工作區所在位置的 [訂用帳戶]。 選取**連線**名稱 (您的 ITSM 連接器名稱)，後面接著您的工作區名稱。 例如，"MyITSMMConnector(MyWorkspace)"。

    ![ITSM 動作詳細資料](./media/log-analytics-itsmc/itsm-action-details.png)

6. 從下拉式功能表選取 [工作項目] 類型。
   選擇使用現有範本，或填滿 ITSM 產品所需的欄位。
7. 按一下 [確定]。

建立/編輯 Azure 警示規則時，請使用具有 ITSM 動作的動作群組。 警示觸發時，會在 ITSM 工具中建立/更新工作項目。

>[!NOTE]

> 如需 ITSM 動作的價格相關資訊，請參閱動作群組的[價格頁面](https://azure.microsoft.com/pricing/details/monitor/)。


## <a name="visualize-and-analyze-the-incident-and-change-request-data"></a>將事件和變更要求資料視覺化並加以分析

根據您在設定連線時的設定，ITSM Connector 最多可將 120 天的事件和變更要求資料同步處理。 [下一節](#additional-information)會提供適用於此資料的記錄檔記錄結構描述。

事件和變更要求資料可以使用解決方案中的 ITSM Connector 儀表板進行視覺化。

![Log Analytics 畫面](./media/log-analytics-itsmc/itsmc-overview-sample-log-analytics.png)

儀表板也會提供連接器狀態的相關資訊，可用來作為分析任何連線問題的起點。

您也可以在服務對應解決方案中，將針對受影響電腦同步處理的事件視覺化。

服務對應可自動探索 Windows 和 Linux 系統上的應用程式元件，並對應服務之間的通訊。 如果您想到您的伺服器，ADM 可讓您以互連系統 (提供重要服務) 的形式檢視它們。 不需要進行任何設定，只要安裝了代理程式，服務對應就會顯示橫跨任何 TCP 連接架構的伺服器、處理程序和連接埠之間的連接。 [深入了解](../operations-management-suite/operations-management-suite-service-map.md)。

如果您使用服務對應解決方案，則可檢視 ITSM 解決方案中建立的服務台項目，如下列範例中所示：

![Log Analytics 畫面](./media/log-analytics-itsmc/itsmc-overview-integrated-solutions.png)

詳細資訊︰[服務對應](../operations-management-suite/operations-management-suite-service-map.md)


## <a name="additional-information"></a>其他資訊

### <a name="data-synced-from-itsm-product"></a>從 ITSM 產品同步的資料
事件和變更要求會根據連線的設定，從您的 ITSM 產品同步處理到您的 Log Analytics 工作區。

下列資訊說明 ITSMC 所收集的資料範例：

> [!NOTE]

> 根據匯入 Log Analytics 的工作項目類型，**ServiceDesk_CL** 會包含下列欄位︰

**工作項目︰** **事件**  
ServiceDeskWorkItemType_s="Incident"

**欄位**

- ServiceDeskConnectionName
- 服務台識別碼
- State
- 急迫性
- 影響
- 優先順序
- 升級
- 建立者
- 解決者
- 關閉者
- 來源
- 指派對象
- 類別
- 標題
- 說明
- 建立日期
- 關閉日期
- 解決日期
- 上次修改日期
- 電腦


**工作項目︰** **變更要求**

ServiceDeskWorkItemType_s="ChangeRequest"

**欄位**
- ServiceDeskConnectionName
- 服務台識別碼
- 建立者
- 關閉者
- 來源
- 指派對象
- 標題
- 類型
- 類別
- State
- 升級
- 衝突狀態
- 急迫性
- 優先順序
- 風險
- 影響
- 指派對象
- 建立日期
- 關閉日期
- 上次修改日期
- 要求日期
- 計劃開始日期
- 計劃結束日期
- 工作開始日期
- 工作結束日期
- 說明
- 電腦

## <a name="output-data-for-a-servicenow-incident"></a>ServiceNow 事件的輸出資料

| Log Analytics 欄位 | ServiceNow 欄位 |
|:--- |:--- |
| ServiceDeskId_s| 數字 |
| IncidentState_s | State |
| Urgency_s |急迫性 |
| Impact_s |影響|
| Priority_s | 優先順序 |
| CreatedBy_s | 開啟者 |
| ResolvedBy_s | 解決者|
| ClosedBy_s  | 關閉者 |
| Source_s| 連絡人型別 |
| AssignedTo_s | 指派對象  |
| Category_s | 類別 |
| Title_s|  簡短說明 |
| Description_s|  注意 |
| CreatedDate_t|  已開啟 |
| ClosedDate_t| 關閉|
| ResolvedDate_t|已解決|
| 電腦  | 設定項目 |

## <a name="output-data-for-a-servicenow-change-request"></a>ServiceNow 變更要求的輸出資料

| Log Analytics | ServiceNow 欄位 |
|:--- |:--- |
| ServiceDeskId_s| 數字 |
| CreatedBy_s | 要求者 |
| ClosedBy_s | 關閉者 |
| AssignedTo_s | 指派對象  |
| Title_s|  簡短說明 |
| Type_s|  類型 |
| Category_s|  類別 |
| CRState_s|  State|
| Urgency_s|  急迫性 |
| Priority_s| 優先順序|
| Risk_s| 風險|
| Impact_s| 影響|
| RequestedDate_t  | 要求的日期 |
| ClosedDate_t | 關閉日期 |
| PlannedStartDate_t  |     計劃開始日期 |
| PlannedEndDate_t  |   計劃結束日期 |
| WorkStartDate_t  | 實際開始日期 |
| WorkEndDate_t | 實際結束日期|
| Description_s | 說明 |
| 電腦  | 設定項目 |


## <a name="troubleshoot-itsm-connections"></a>針對 ITSM 連線進行疑難排解
1.  如果從已連線的來源 UI 連線失敗，並顯示 [儲存連線時發生錯誤] 訊息，請執行下列步驟：
 - 對於 ServiceNow、 Cherwell 和 Provance 連線，  
    - 請確定您已正確輸入每個連線的使用者名稱、密碼、用戶端識別碼及用戶端密碼。  
    - 請檢查您在對應的 ITSM 產品中是否擁有足夠權限來進行連線。  
 - 對於服務管理連線，  
    - 請確定已成功部署 Web 應用程式，且已建立混合式連線。 若要確認是否已成功與內部部署 Service Manager 機器建立連線，請瀏覽 Web 應用程式 URL，如文件中針對建立[混合式連線](log-analytics-itsmc-connections.md#configure-the-hybrid-connection)所述。  

2.  如果 ServiceNow 的資料未同步處理至 Log Analytics，請確定 ServiceNow 執行個體並非處在睡眠中。 ServiceNow 開發人員執行個體閒置很長的時間時，有時會進入睡眠狀態。 否則，請回報問題。
3.  如果引發 OMS 警示，但未在 ITSM 產品中建立工作項目，或未將設定項目建立為/連結至工作項目，或需任何其他一般資訊，請查看下列位置：
 -  ITSMC：解決方案會顯示連線/工作項目/電腦等的摘要。按一下顯示 [連接器狀態] 的磚，這會利用相關的查詢帶您前往 [記錄搜尋]。 如需詳細資訊，請查看 ERROR 為 LogType_S 的記錄檔記錄。
 - [記錄搜尋] 頁面：使用查詢 `*`ServiceDeskLog_CL`*` 直接檢視錯誤/相關資訊。

## <a name="troubleshoot-service-manager-web-app-deployment"></a>針對 Service Manager Web 應用程式部署進行疑難排解
1.  如果是有關 Web 應用程式部署的任何問題，請確定您在所述的訂用帳戶中擁有足夠權限來建立/部署資源。
2.  當您執行[指令碼](log-analytics-itsmc-service-manager-script.md)時，如果收到「物件參考未設定為物件的執行個體」錯誤訊息，請確定您已在 [使用者設定] 區段中輸入有效值。
3.  如果您無法建立服務匯流排轉送命名空間，請確定所需的資源提供者已在訂用帳戶中註冊。 如果未註冊，請以手動方式從 Azure 入口網站建立服務匯流排轉送命名空間。 您也可以在建立它時，同時從 Azure 入口網站[建立混合式連線](log-analytics-itsmc-connections.md#configure-the-hybrid-connection)。


## <a name="contact-us"></a>與我們連絡

如需關於 IT 服務管理連接器的任何查詢或意見反應，請與我們連絡：[omsitsmfeedback@microsoft.com](mailto:omsitsmfeedback@microsoft.com)。

## <a name="next-steps"></a>後續步驟
[將 ITSM 產品/服務新增至 IT 服務管理連接器](log-analytics-itsmc-connections.md)。
