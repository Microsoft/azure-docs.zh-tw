---
title: 設定混合式 Windows 電腦的 Azure Log Analytics 代理程式 | Microsoft Docs
description: 了解如何針對在 Azure 外部電腦上執行的 Windows 部署 Log Analytics 代理程式，並使用 Log Analytics 來啟用資料收集。
services: log-analytics
documentationcenter: log-analytics
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 08/02/2018
ms.author: magoedte
ms.custom: mvc
ms.component: na
ms.openlocfilehash: 91a656b055dbe882bdd5e5c208cc5d8955c74041
ms.sourcegitcommit: fc5555a0250e3ef4914b077e017d30185b4a27e6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39480434"
---
# <a name="configure-log-analytics-agent-for-windows-computers-in-a-hybrid-environment"></a>為混合式環境中的 Windows 電腦設定 Log Analytics 代理程式
[Azure Log Analytics](log-analytics-overview.md) 可直接從資料中心或其他雲端環境內的實體或虛擬 Windows 電腦，將資料收集到單一存放庫，來進行詳細的分析和相互關聯。  本快速入門向您示範如何以幾個簡單步驟來設定和收集 Windows 電腦的資料。  針對 Azure Windows VM，請參閱下列主題：[收集關於 Azure 虛擬機器的資料](log-analytics-quick-collect-azurevm.md)。  

若要了解支援的組態，請檢閱[支援的 Windows 作業系統](log-analytics-concept-hybrid.md#supported-windows-operating-systems)和[網路防火牆組態](log-analytics-concept-hybrid.md#network-firewall-requirements)。
 
如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="log-in-to-azure-portal"></a>登入 Azure 入口網站
在 [https://portal.azure.com](https://portal.azure.com) 上登入 Azure 入口網站。 

## <a name="create-a-workspace"></a>建立工作區
1. 在 Azure 入口網站中，按一下 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。<br><br> ![Azure 入口網站](media/log-analytics-quick-collect-azurevm/azure-portal-01.png)<br><br>  
2. 按一下 [建立]，然後選取下列項目的選項：

  * 為新的 [OMS 工作區] 提供名稱，例如，*DefaultLAWorkspace*。 
  * 如果選取的預設值不合適，請從下拉式清單中選取要連結的 [訂用帳戶]。
  * 對於 [資源群組]，選取包含一或多個 Azure 虛擬機器的現有資源群組。  
  * 選取要部署 VM 的 [位置]。  如需詳細資訊，請查看 [Log Analytics 的可用區域](https://azure.microsoft.com/regions/services/)。  
  * 如果您要在 2018 年 4 月 2 之後建立的新訂用帳戶中建立工作區，系統會自動使用「每 GB」定價方案和選項來選取將無法使用的定價層。  如果您要為在 4 月 2 日之前建立的現有訂用帳戶，或已繫結至現有 EA 註冊的訂用帳戶建立工作區，請選取您偏好的定價層。  如需特定層的詳細資訊，請參閱 [Log Analytics 價格詳細資料](https://azure.microsoft.com/pricing/details/log-analytics/)。

        ![Create Log Analytics resource blade](media/log-analytics-quick-collect-azurevm/create-loganalytics-workspace-02.png)<br>  

3. 在 [OMS 工作區] 窗格上提供必要資訊之後，按一下 [確定]。  

在驗證資訊及建立工作區時，您可以在功能表的 [通知] 底下追蹤其進度。 

## <a name="obtain-workspace-id-and-key"></a>取得工作區識別碼和金鑰
安裝適用於 Windows 的 Microsoft Monitoring Agent 之前，您必須有 Log Analytics 工作區的工作區識別碼和金鑰。  安裝精靈必須要有這項資訊，才能正確設定代理程式，並確保它能與 Log Analytics 順利進行通訊。  

1. 在 Azure 入口網站中，按一下左上角的 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。
2. 在 Log Analytics 工作區清單中，選取稍早建立的 *DefaultLAWorkspace*。
3. 選取 [進階設定]。<br><br> ![Log Analytics 進階設定](media/log-analytics-quick-collect-azurevm/log-analytics-advanced-settings-01.png)<br><br>  
4. 選取 [連接的來源]，然後選取 [Windows 伺服器]。   
5. [工作區識別碼] 和 [主要金鑰] 右邊的值。 將兩者複製並貼到您最愛的編輯器。   

## <a name="install-the-agent-for-windows"></a>安裝適用於 Windows 的代理程式
下列步驟會在您的電腦上，使用 Microsoft Monitoring Agent 的安裝程式來安裝並設定適用於 Azure 和 Azure Government Cloud 中 Log Analytics 的代理程式。  

1. 在 [Windows 伺服器] 頁面上，根據 Windows 作業系統的處理器架構，選取適當的 [下載 Windows 代理程式] 版本來下載。
2. 執行安裝程式以在您的電腦上安裝代理程式。
2. 在 [歡迎] 頁面中按 [下一步]。
3. 閱讀 [授權條款] 頁面上的授權，然後按一下 [我接受]。
4. 在 [目的資料夾] 頁面上，變更或保留預設的安裝資料夾，然後按 [下一步]。
5. 在 [代理程式安裝選項] 頁面上，選擇將代理程式連線到 Azure Log Analytics (OMS)，然後按 [下一步]。   
6. 在 [Azure Log Analytics] 頁面上，執行下列操作：
   1. 貼上您先前複製的**工作區識別碼**和**工作區金鑰 (主要金鑰)**。  如果電腦應該向 Azure Government Cloud 中的 Log Analytics 工作區回報，請從 [Azure 雲端] 下拉式清單中選取 [Azure 美國政府]。  
   2. 如果電腦需要透過 Proxy 伺服器與 Log Analytics 服務進行通訊，請按一下 [進階]，然後提供 Proxy 伺服器的 URL 和連接埠號碼。  如果您的 Proxy 伺服器會要求驗證，請輸入要向 Proxy 伺服器進行驗證的使用者名稱和密碼，然後按 [下一步]。  
7. 提供完必要的組態設定之後，按 [下一步]。<br><br> ![貼上工作區識別碼和主索引鍵](media/log-analytics-quick-collect-windows-computer/log-analytics-mma-setup-laworkspace.png)<br><br>
8. 在 [安裝準備就緒] 頁面上，檢閱您的選擇，然後按一下 [安裝]。
9. 在 [設定成功完成] 頁面上，按一下 [完成]。

完成時，[Microsoft 監視代理程式] 會出現在 [控制台] 中。 您可以檢閱您的組態，並確認代理程式已連線到 Log Analytics。 已連線時，在 [Azure Log Analytics (OMS)] 索引標籤上，代理程式會顯示下列訊息︰**Microsoft Monitoring Agent 已成功連接到 Microsoft Operations Management Suite 服務。**<br><br> ![MMA 對 Log Analytics 的連線狀態](media/log-analytics-quick-collect-windows-computer/log-analytics-mma-laworkspace-status.png)

## <a name="collect-event-and-performance-data"></a>收集事件和效能資料
Log Analytics 可以從 Windows 事件記錄檔收集事件，以及收集您針對長期分析和報告指定的效能計數器，並在偵測到特定條件時採取動作。  請依照下列步驟來設定從 Windows 事件記錄檔收集事件，以及收集數個常用的效能計數器，來開始進行作業。  

1. 在 Azure 入口網站中，按一下左下角的 [更多服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。
2. 選取 [進階設定]。<br><br> ![Log Analytics 進階設定](media/log-analytics-quick-collect-azurevm/log-analytics-advanced-settings-01.png)<br><br> 
3. 選取 [資料]，然後選取 [Windows 事件記錄檔]。  
4. 您可以輸入記錄檔的名稱，來新增事件記錄檔。  輸入 **System**，然後按一下加號 **+**。  
5. 在表格中，檢查 [錯誤] 和 [警告] 嚴重性。   
6. 按一下頁面頂端的 [儲存] 來儲存設定。
7. 選取 [Windows 效能資料] 以啟用收集 Windows 電腦上的效能計數器。 
8. 當您第一次為新的 Log Analytics 工作區設定 Windows 效能計數器時，系統會提供選項，讓您快速建立數個常用的計數器。 這些計數器旁邊皆會列出核取方塊。<br> ![選取的預設 Windows 效能計數器](media/log-analytics-quick-collect-azurevm/windows-perfcounters-default.png)。<br> 按一下 [新增選定的效能計數器]。  隨即會新增且收集取樣間隔時間的預設值為 10 秒。  
9. 按一下頁面頂端的 [儲存] 來儲存設定。

## <a name="view-data-collected"></a>檢視收集的資料
既然您已啟用資料收集，現在即可執行簡單的記錄搜尋範例，以查看來自目標電腦的一些資料。  

1. 在 Azure 入口網站中，於選取的工作區底下，按一下 [記錄搜尋] 圖格。  
2. 在 [記錄搜尋] 窗格的查詢欄位中輸入 `Perf`，然後按 Enter 鍵，或按一下查詢欄位右邊的搜尋按鈕。<br><br> ![Log Analytics 記錄搜尋查詢範例](media/log-analytics-quick-collect-linux-computer/log-analytics-portal-queryexample.png)<br><br> 例如，下圖中的查詢會傳回 735 筆效能記錄。<br><br> ![Log Analytics 記錄搜尋結果](media/log-analytics-quick-collect-windows-computer/log-analytics-search-perf.png)

## <a name="clean-up-resources"></a>清除資源
不再需要代理程式時，您可以從 Windows 電腦移除代理程式，並刪除 Log Analytics 工作區。  

若要移除代理程式，請執行下列步驟。

1. 開啟 [ **控制台**]。
2. 開啟 [程式和功能]。
3. 在 [程式和功能] 中，選取 [Microsoft Monitoring Agent]，然後按一下 [解除安裝]。

若要刪除工作區，請選取您先前建立的 Log Analytics 工作區，然後在資源頁面上，按一下 [刪除]。<br><br> ![刪除 Log Analytics 資源](media/log-analytics-quick-collect-azurevm/log-analytics-portal-delete-resource.png)

## <a name="next-steps"></a>後續步驟
既然您正在從內部部署 Linux 電腦收集作業和效能資料，即可輕鬆開始針對收集的資料「免費」進行探索、分析及採取行動。  

若要了解如何檢視和分析資料，請繼續進行本教學課程。   

> [!div class="nextstepaction"]
> [在 Log Analytics 中檢視或分析資料](log-analytics-tutorial-viewdata.md)
