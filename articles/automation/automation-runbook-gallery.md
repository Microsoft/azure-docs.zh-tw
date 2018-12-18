---
title: Azure 自動化的 Runbook 和模組資源庫
description: 來自 Microsoft 和社群的 Runbook 和模組可供您在 Azure 自動化環境中安裝及使用。  此文章說明如何存取這些資源以及將您的 Runbook 貢獻至資源庫。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 09/11/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 988f77dd0f8f31f6e31e5371ed263d4c4195c105
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45984868"
---
# <a name="runbook-and-module-galleries-for-azure-automation"></a>Azure 自動化的 Runbook 和模組資源庫
您可以存取各種已由 Microsoft 和社群建置的案例，而不是在 Azure 自動化中建立您自己的 Runbook 和模組。  您可以不加修改地使用這些案例，或者使用它們做為起點並針對您的特定需求進行編輯。

您可以從 [Runbook 資源庫](#runbooks-in-runbook-gallery)取得 Runbook，從 [PowerShell 資源庫](#modules-in-powerShell-gallery)取得模組。  您也可以透過共用您開發的案例來參與社群，請參閱[將 Runbook 新增至資源庫](automation-runbook-gallery.md#adding-a-runbook-to-the-runbook-gallery)

## <a name="runbooks-in-runbook-gallery"></a>Runbook 資源庫中的 Runbook
[Runbook 資源庫](http://gallery.technet.microsoft.com/scriptcenter/site/search?f\[0\].Type=RootCategory&f\[0\].Value=WindowsAzure&f\[1\].Type=SubCategory&f\[1\].Value=WindowsAzure_automation&f\[1\].Text=Automation)提供各種來自 Microsoft 和社群的 Runbook，您可以匯入至 Azure 自動化。 您可以從 [TechNet 指令碼中心](https://gallery.technet.microsoft.com/scriptcenter/site/upload)託管的資源庫下載 Runbook，或者，您可以直接從 Azure 入口網站的資源庫匯入 Runbook。

若要直接從 Runbook 資源庫匯入，您只能使用 Azure 入口網站。 您無法使用 Windows PowerShell 執行此函式。

> [!NOTE]
> 您應該驗證您從 Runbook 資源庫取得的任何 Runbook 的內容，並且在生產環境中安裝和執行它們時小心謹慎。
> 
> 

### <a name="to-import-a-runbook-from-the-runbook-gallery-with-the-azure-portal"></a>使用 Azure 入口網站從 Runbook 資源庫匯入 Runbook
1. 在 Azure 入口網站中，開啟您的自動化帳戶。
2. 在 [程序自動化] 底下，按一下 [Runbook 資源庫]
3. 找出您想要的資源庫項目，並且選取以檢視其詳細資料。 您可以在左邊輸入發行者和類型的其他搜尋參數。
   
    ![瀏覽資源庫](media/automation-runbook-gallery/browse-gallery.png)
5. 按一下 [檢視來源專案]  以檢視 [TechNet 指令碼中心](http://gallery.technet.microsoft.com/)中的項目。
6. 若要匯入項目，請按一下以檢視其詳細資料，然後按一下 [匯入]  按鈕。
   
    ![匯入按鈕](media/automation-runbook-gallery/gallery-item-detail.png)
7. 選擇性變更 Runbook 的名稱，然後按一下 [確定]  以匯入 Runbook。
8. Runbook 會出現在自動化帳戶的 [Runbook] 索引標籤上。

### <a name="adding-a-runbook-to-the-runbook-gallery"></a>將 Runbook 新增至 Runbook 資源庫
Microsoft 鼓勵您將您認為可能有助於其他客戶的 Runbook 新增至 Runbook 資源庫。  您可以透過[將它上傳到指令碼中心](http://gallery.technet.microsoft.com/site/upload)來新增 Runbook，並且將下列詳細資料納入考量。

* 您必須針對 [類別] 指定 [Windows Azure]，針對 [子類別] 指定[自動化]，讓 Runbook 顯示在精靈中。  
* 上傳必須是單一 .ps1 或 .graphrunbook 檔案。  如果 Runbook 需要任何模組、子 Runbook 或資產，則您應該在提交的說明中和 Runbook 的註解區段中列出。  如果您有需要多個 Runbook 的案例，則分別將每個上傳並且在各自的說明中列出相關 Runbook 的名稱。 請確定您使用相同的標記，這樣才會出現在相同的類別中。 使用者必須閱讀說明，以得知案例需要其他 Runbook 才能運作。
* 如果您要發佈 [圖形化 Runbook]  \(而非圖形化工作流程)，請新增 "GraphicalPS" 標籤。 
* 使用 [ **插入程式碼區段** ] 圖示將 PowerShell 或 PowerShell 工作流程程式碼片段插入說明中。
* 上傳的摘要會顯示在 Runbook 資源庫結果中，因此，您應該提供詳細資訊，幫助使用者識別 Runbook 的功能。
* 您應該對上傳項目指派 1 到 3 個下列標記。  在精靈中，Runbook 會列在符合其標記的類別底下。  精靈會略過不在此清單上的任何標記。 如果您未指定任何相符的標記，Runbook 會列在 [其他] 類別底下。
  
  * Backup 
  * 產能管理
  * 變更控制
  * 法規遵循
  * 開發/測試環境
  * 災害復原
  * 監視
  * 修補
  * 佈建
  * 補救
  * VM 生命週期管理
* 一小時更新自動化資源庫一次，因此您不會立即看到您的貢獻。

## <a name="modules-in-powershell-gallery"></a>PowerShell 資源庫中的模組
PowerShell 模組包含您可以在 Runbook 中使用的 Cmdlet，您可以安裝在 Azure 自動化中的現有模組可於 [PowerShell 資源庫](http://www.powershellgallery.com)取得。  您可以從 Azure 入口網站啟動此資源庫，然後直接安裝至 Azure 自動化，也可以手動下載及安裝。  

### <a name="to-import-a-module-from-the-automation-module-gallery-with-the-azure-portal"></a>透過 Azure 入口網站從自動化模組資源庫匯入模組
1. 在 Azure 入口網站中，開啟您的自動化帳戶。
2. 在 [共用資源] 下選取 [模組]，以開啟模組清單。
4. 從頁面頂端按一下 [瀏覽資源庫]。
   
    ![模組資源庫](media/automation-runbook-gallery/modules-blade.png) <br>
5. 在 [瀏覽資源庫] 頁面上，您可以依下列欄位來搜尋：
   
   * 模組名稱
   * 標記
   * 作者
   * Cmdlet/DSC 資源名稱
6. 尋找您感興趣的模組，並選取以檢視其詳細資料。  
   當您向下切入到特定的模組時，您可以檢視有關該模組的詳細資訊，包括返回 PowerShell 資源庫的連結、任何必要的相依性，以及該模組包含的所有 cmdlet 和 (或) DSC 資源。
   
    ![PowerShell 模組的詳細資料](media/automation-runbook-gallery/gallery-item-details-blade.png) <br>
7. 若要將該模組直接安裝至 Azure 自動化，請按一下 [匯入]  按鈕。
8. 按一下 [匯入] 按鈕時，在 [匯入] 窗格上，您會看到將要匯入的模組名稱。 如果已安裝所有相依性，[確定] 按鈕會啟動。 如果缺少相依性，需先匯入它們之後才能匯入該模組。
9. 在 [匯入] 分頁上按一下 [確定] 以匯入模組。 當 Azure 自動化將模組匯入至您的帳戶時，它會擷取有關模組和 Cmdlet 的中繼資料。 因為必須解壓縮每個活動，此步驟可能需要幾分鐘的時間。
10. 您會收到初始通知，表示正在部署模組，而完成時還會收到另一個通知。
11. 匯入模組之後，您可以看到可用的活動，還可以將其資源用於 Runbook 和「預期狀態設定」中。

> [!NOTE]
> Azure 自動化不支援僅支援 PowerShell 核心的模組，該模組無法匯入 Azure 入口網站中，或直接從 PowerShell 資源庫部署。

## <a name="python-runbooks"></a>Python Runbook

[指令碼中心資源庫](https://gallery.technet.microsoft.com/scriptcenter/site/search?f%5B0%5D.Type=RootCategory&f%5B0%5D.Value=WindowsAzure&f%5B1%5D.Type=ProgrammingLanguage&f%5B1%5D.Value=Python&f%5B1%5D.Text=Python&sortBy=Date&username=)中提供 Python Runbook。 您可以貢獻 Python Runbook 給指令碼中心資源庫。 當您這麼做時，請務必在上傳您的貢獻項目時新增標籤 **Python**。

## <a name="requesting-a-runbook-or-module"></a>要求 Runbook 或模組
您可以將要求傳送至 [使用者心聲](https://feedback.azure.com/forums/246290-azure-automation/)。  如果您需要協助撰寫 Runbook 或有關於 PowerShell 的問題，請將問題張貼至我們的 [論壇](http://social.msdn.microsoft.com/Forums/windowsazure/en-US/home?forum=azureautomation&filter=alltypes&sort=lastpostdesc)。

## <a name="next-steps"></a>後續步驟
* 若要開始使用 Runbook，請參閱 [在 Azure 自動化中建立或匯入 Runbook](automation-creating-importing-runbook.md)
* 若要了解含有 Runbook 的 PowerShell 和 PowerShell 工作流程之間的差異，請參閱 [了解 PowerShell 工作流程](automation-powershell-workflow.md)

