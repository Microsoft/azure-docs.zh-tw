---
title: 探索您具有 Azure 自動化的機器上安裝了哪些軟體 | Microsoft Docs
description: 使用清查探索您的環境中安裝了哪些軟體在電腦上。
services: automation
keywords: 清查、自動化、變更、追蹤
author: jennyhunter-msft
ms.author: jehunte
ms.date: 04/11/2018
ms.topic: tutorial
ms.service: automation
ms.component: change-inventory-management
ms.custom: mvc
manager: carmonm
ms.openlocfilehash: d8fb8ca84f68e6e8c503375b3b2fae1ce1eb72ba
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46989204"
---
# <a name="discover-what-software-is-installed-on-your-azure-and-non-azure-machines"></a>探索您的 Azure 電腦和非 Azure 電腦上安裝了哪些軟體

在本教學課程中，您會了解如何探索您環境中安裝了哪些軟體。 您可以在電腦上收集並檢視軟體、檔案、Linux 精靈、Windows 服務和 Windows 登錄機碼的清查。 追蹤您電腦的設定可協助您找出環境中的操作問題，並進一步了解您電腦的狀態。

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 啟用解決方案
> * 使 Azure 虛擬機器上線
> * 讓非 Azure VM 上架
> * 檢視已安裝的軟體
> * 搜尋已安裝軟體的清查記錄

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要：

* Azure 訂用帳戶。 如果您沒有這類帳戶，可以[啟用自己的 MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或註冊[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* [自動化帳戶](automation-offering-get-started.md)可保存監看員和動作 Runbook，以及監看員工作。
* 要上架的[虛擬機器](../virtual-machines/windows/quick-create-portal.md)。

## <a name="log-in-to-azure"></a>登入 Azure

在 http://portal.azure.com 上登入 Azure 入口網站。

## <a name="enable-change-tracking-and-inventory"></a>啟用變更追蹤和清查

您必須先在本教學課程中啟用變更追蹤和清查。 如果您先前已啟用 [變更追蹤] 解決方案，就不需要此步驟。

瀏覽至您的自動化帳戶，然後選取 [組態管理] 下的 [清查]。

選擇 Log Analytics 工作區與自動化帳戶，然後按一下 [啟用] 來啟用解決方案。 啟用解決方案最多需要 15 分鐘。

![清查上架設定橫幅](./media/automation-tutorial-installed-software/enableinventory.png)

若要啟用此解決方案，請設定位置、Log Analytics 工作區，以及要使用的自動化帳戶，然後按一下 [啟用]。 如果欄位呈現灰色，就表示已啟用 VM 的另一個自動化解決方案，且必須使用相同的工作區和自動化帳戶。

[Log Analytics](../log-analytics/log-analytics-overview.md?toc=%2fazure%2fautomation%2ftoc.json) 工作區用來收集功能和服務 (例如清查) 所產生的資料。
工作區提供單一位置來檢閱和分析來自多個來源的資料。

啟用解決方案可能需要 15 分鐘。 在此期間，請勿關閉瀏覽器視窗。
啟用解決方案之後，有關在 VM 上已安裝軟體和變更的相關資訊會流向 Log Analytics。
可能需要 30 分鐘到 6 小時，資料才可供分析。

## <a name="onboard-a-vm"></a>讓 VM 上架

在您的自動化帳戶中，瀏覽置 [組態管理] 下的 [清查]。

選取 [+ 新增 Azure VM]，這會開啟 [虛擬機器] 頁面，並可讓您從清單中選取現有的 VM。 選取您要上架的 VM。 在開啟的頁面上，按一下 [啟用] 以啟用 VM 上的解決方案。 系統會將 Microsoft Management Agent 部署至 VM　　，並將代理程式設定為能與啟用方案時設定的 Log Analytics 工作區通訊。 這需要數分鐘的時間來完成上架。 此時，您可以從清單中選取新的 VM，然後讓另一個 VM 上架。

## <a name="onboard-a-non-azure-machine"></a>讓非 Azure 機器上線

若要新增非 Azure 機器，請安裝 [Windows](../log-analytics/log-analytics-agent-windows.md) 或 [Linux](automation-linux-hrw-install.md) 的代理程式 (視作業系統而定)。 安裝代理程式之後，瀏覽至您的自動化帳戶，然後移至 [組態管理] 下的 [清查]。 當您按一下 [管理機器] 時，您會看到向 Log Analytics 工作區回報的機器清單，而其中並沒有啟用解決方案。 為您的環境選取適當選項。

* **在所有可用機器上啟用** - 此選項可在此時向您 Log Analytics 工作區回報的所有機器上，啟用解決方案。
* **在所有可用機器和未來機器上啟用** - 此選項可在向您 Log Analytics 工作區回報的所有機器上，以及之後會新增至工作區的所有未來機器上，啟用解決方案。
* **在選取的機器上啟用** - 此選項只會在您選取的電腦上啟用解決方案。

![管理機器](./media/automation-tutorial-installed-software/manage-machines.png)

## <a name="view-installed-software"></a>檢視已安裝的軟體

一旦啟用變更追蹤和清查解決方案後，您就可以檢視 [清查] 頁面上的結果。

從您的自動化帳戶中，選取 [組態管理] 下的 [清查]。

在 [清查] 頁面上，按一下 [軟體] 索引標籤。

在 [軟體] 索引標籤上，有一份資料表列出已找到的軟體。 軟體會依軟體名稱及版本群組。

每一筆軟體記錄的高階詳細資料都可在資料表中進行檢視。 這些詳細資料包括軟體名稱、版本、發行者、上次重新整理的時間 (由群組中電腦所報告的最近重新整理時間) 和電腦 (具有該軟體的電腦計數)。

![軟體清查](./media/automation-tutorial-installed-software/inventory-software.png)

按一下資料列可檢視軟體記錄的屬性，以及具有該軟體之電腦的名稱。

若要尋找特定軟體或軟體的群組，您可以在軟體清單上方的文字方塊中直接搜尋。
篩選器可讓您以軟體名稱、版本或發行者作為基礎進行搜尋。

例如，搜尋 "Contoso" 會傳回名稱、發行者或版本中包含 "Contoso" 的所有軟體。

## <a name="search-inventory-logs-for-installed-software"></a>搜尋已安裝軟體的清查記錄

清查所產生的記錄資料會傳送到 Log Analytics。 若要透過執行查詢來搜尋記錄，請選取 [清查] 視窗頂端的 [Log Analytics]。

清查資料會儲存在 **ConfigurationData** 類型之下。
下列的 Log Analytics 查詢範例會傳回清查結果，其中「發行者」是 "Microsoft Corporation"。

```loganalytics
ConfigurationData
| where ConfigDataType == "Software"
| where Publisher == "Microsoft Corporation"
| summarize arg_max(TimeGenerated, *) by SoftwareName, Computer
```

若要深入了解在 Log Analytics 中執行和搜尋記錄檔，請參閱 [Azure Log Analytics](../log-analytics/log-analytics-queries.md)。

### <a name="single-machine-inventory"></a>單一電腦清查

若要查看單一電腦的軟體清查，您可以從 Azure VM 資源頁面存取清查，或是使用 Log Analytics 來篩選到對應的機器。
下列範例 Log Analytics 查詢會傳回名為 ContosoVM 的電腦之軟體清單。

```loganalytics
ConfigurationData
| where ConfigDataType == "Software"
| summarize arg_max(TimeGenerated, *) by SoftwareName, CurrentVersion
| where Computer =="ContosoVM"
| render table
| summarize by Publisher, SoftwareName
```

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何檢視軟體清查，例如要如何：

> [!div class="checklist"]
> * 啟用解決方案
> * 使 Azure 虛擬機器上線
> * 讓非 Azure VM 上架
> * 檢視已安裝的軟體
> * 搜尋已安裝軟體的清查記錄

如需深入了解，請繼續閱讀變更追蹤和清查解決方案的概觀。

> [!div class="nextstepaction"]
> [變更管理及清查解決方案](automation-change-tracking.md)