---
title: 使用 Azure Log Analytics 警示來回應事件 | Microsoft Docs
description: 本教學課程可幫助您了解 Log Analytics 的警示，以識別您工作區中的重要資訊，並可主動通知您相關問題或叫用動作以嘗試更正問題。
services: log-analytics
documentationcenter: log-analytics
author: MGoedtel
manager: carmonm
editor: ''
ms.assetid: abb07f6c-b356-4f15-85f5-60e4415d0ba2
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/05/2018
ms.author: magoedte
ms.custom: mvc
ms.openlocfilehash: e15f440fd1818e2d5ce1e72881b8a0f769dec8fc
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48829897"
---
# <a name="respond-to-events-with-azure-monitor-alerts"></a>使用 Azure 監視器警示來回應事件
Azure 監視器中的警示可識別您 Log Analytics 存放庫中的重要資訊。 它們是由自動定期執行記錄搜尋的警示規則所建立，如果記錄搜尋的結果符合特定準則，則會建立一個警示記錄，而它可以設定為執行自動化的回應。  本教學課程是[建立和共用 Log Analytics 資料的儀表板](log-analytics-tutorial-dashboards.md)教學課程的延續。   

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立警示規則
> * 設定動作群組以傳送電子郵件通知

若要完成本教學課程中的範例，您目前必須有[連線至 Log Analytics 工作區](log-analytics-quick-collect-azurevm.md)的虛擬機器。  

## <a name="sign-in-to-azure-portal"></a>登入 Azure 入口網站
在 [https://portal.azure.com](https://portal.azure.com) 登入 Azure 入口網站。 

## <a name="create-alerts"></a>建立警示
警示是由 Azure 監視器中的警示規則所建立，可定期自動執行儲存的查詢或自訂的記錄搜尋。  建立警示的依據，可以是特定的效能計量、特定事件建立時、缺少某個事件或特定的時間間隔內建立了數個事件。  例如，當平均 CPU 使用量超過某閾值、偵測到未執行的更新，或偵測到特定 Windows 服務或 Linux 精靈未執行而產生事件時，均可利用警示來通知您。  如果記錄搜尋的結果符合特定準則，則會建立警示。 此規則接著可以自動執行一或多個動作，例如通知警示或叫用另一個處理序。 

在下列範例中，您將根據在[視覺化資料教學課程](log-analytics-tutorial-dashboards.md)中儲存的「Azure VM - 處理器使用率」查詢，來建立計量測量警示規則。  為每個超過閾值 (90%) 的虛擬機器建立警示。  

1. 在 Azure 入口網站中，按一下 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。
2. 在左側窗格中選取 [警示]，然後按一下頁面頂端的 [新增警示規則]，即可建立新的警示。<br><br> ![建立新的警示規則](./media/log-analytics-tutorial-response/alert-rule-02.png)<br>
3. 第一步驟，在 [建立警示] 區段下選取您的 Log Analytics 工作區作為資源，因為這是以記錄為基礎的警示訊號。  從下拉式清單中選擇特定**訂用帳戶**可篩選結果 (如果您有多個訂用帳戶)，其中包含稍早建立的 VM 和 Log Analytics 工作區。  從下拉式清單選取 **Log Analytics** 可篩選**資源類型**。  最後，選取**資源** **DefaultLAWorkspace**，然後按一下 [完成]。<br><br> ![建立警示步驟 1 的工作](./media/log-analytics-tutorial-response/alert-rule-03.png)<br>
4. 在 [警示準則] 區段下按一下 [新增準則]，可選取我們已儲存的查詢，然後指定警示規則所遵循的邏輯。  從 [設定訊號邏輯] 窗格中，選取清單中的 [Azure VM - 處理器使用率]。  窗格會隨即更新，以顯示警示的組態設定。  最上方會顯示所選訊號過去 30 分鐘內的結果及搜尋查詢本身。  
5. 使用下列資訊設定警示：  
   a. 從 [依據] 下拉式清單中，選取 [計量測量]。  計量測量會針對查詢中其值超過指定閾值的每個物件建立警示。  
   b. 針對 [條件]，選取 [大於]，並輸入 **90**作為 [閾值]。  
   c. 在 [觸發程序警示依據] 區段下方，選取 [連續違規]，然後從下拉式清單中選取 [大於] 並輸入 3 作為值。  
   d. 在 [評估依據] 下方，將 [期限] 值修改為 **30** 分鐘。 此規則會每隔五分鐘執行一次，並傳回在目前時間之前的三十分鐘內建立的記錄。  將時間週期設定為較大的時間範圍，可因應資料延遲的可能性，並確保查詢會傳回資料，以避免發生永遠不會引發警示的漏判。  
6. 按一下 [完成] 完成警示規則。<br><br> ![設定警示訊號](./media/log-analytics-tutorial-response/alert-signal-logic-02.png)<br> 
7. 現在移至第二個步驟，在 [警示規則名稱] 欄位中提供您的警示名稱，例如**大於 90% 的 CPU 百分比**。  指定詳細說明警示的 [描述]，然後從提供的選項中，選取 [重大 (Sev 0)]作為 [嚴重性] 的值。<br><br> ![設定警示詳細資料](./media/log-analytics-tutorial-response/alert-signal-logic-04.png)<br>
8. 若要在完成建立後立即啟動警示規則，請接受 [在建立時啟用規則] 的預設值。
9. 在第三個和最後一個步驟中，您將指定 [動作群組]，這可確保每次警示觸發時，系統會採取相同動作，並可用於您定義的每個規則中。  使用下列資訊來設定新的動作群組：  
   a. 選取 [新增動作群組]後會出現 [新增動作群組] 窗格。  
   b. 針對 [動作群組名稱] 指定名稱，例如 **IT Operations - Notify**，以及指定 [簡短名稱]，例如 **itops-n**。  
   c. 確認 [訂用帳戶]和 [資源群組] 的預設值正確無誤。 如果不正確，從下拉式清單中選取正確的項目。   
   d. 在 [動作] 區段下指定動作名稱，例如**傳送電子郵件**，並在 [動作類型] 下，從下拉式清單中選取 [電子郵件/SMS/推播/語音]。 [電子郵件/SMS/推播/語音] 屬性窗格會在右側開啟，以提供其他資訊。  
   e. 在 [電子郵件/SMS/推播/語音] 窗格中，啟用 [電子郵件]，並提供可接收訊息的有效電子郵件 SMTP 位址。  
   f. 按一下 [確定]  以儲存變更。<br><br> 

    ![建立新的動作群組](./media/log-analytics-tutorial-response/action-group-properties-01.png)

10. 按一下 [確定] 以完成動作群組。 
11. 按一下 [建立警示規則] 以完成警示規則。 此警示規則會立即開始執行。<br><br> ![完成建立新的警示規則](./media/log-analytics-tutorial-response/alert-rule-01.png)<br> 

## <a name="view-your-alerts-in-azure-portal"></a>在 Azure 入口網站中檢視警示
您現在已建立了警示，接著您可以在單一窗格中檢視 Azure 警示，並管理多個 Azure 訂用帳戶的所有警示規則。 它會列出所有警示規則 (已啟用或已停用)，並可根據目標資源、資源群組、規則名稱或狀態來排序。 其中包含所有已引發警示的彙總摘要，以及已設定/已啟用的警示規則總計。<br><br> ![Azure 警示狀態頁面](./media/log-analytics-tutorial-response/azure-alerts-02.png)  

當警示觸發時，表格會反映所選時間範圍內 (預設值是過去六小時) 發生的警示狀況和次數。  您的收件匣中應會有對應的電子郵件 (類似下列範例)，其中顯示違規的虛擬機器和符合此案例中搜尋查詢的前幾個結果。<br><br> ![警示電子郵件動作的範例](./media/log-analytics-tutorial-response/azure-alert-email-notification-01.png)

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已學會警示規則在以排定的間隔執行記錄搜尋和比對特定準則時，會如何主動識別和回應問題。

點選以下連結以查看預先建立的 Log Analytics 指令碼範例。  

> [!div class="nextstepaction"]
> [Log Analytics 指令碼範例](powershell-samples.md)