---
title: 在 Azure 自動化帳戶中建立監看員工作
description: 了解如何在 Azure 自動化帳戶中建立監看員工作，以監看在資料夾中建立的新檔案。
services: automation
ms.service: automation
ms.component: process-automation
author: eamonoreilly
ms.author: eamono
ms.topic: conceptual
ms.date: 03/19/2017
ms.openlocfilehash: 0cc215d6643c86460a1d5471aa1eed8fdf18e028
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34194728"
---
# <a name="create-an-azure-automation-watcher-tasks-to-track-file-changes-on-a-local-machine"></a>建立 Azure 自動化監看員工作，以追蹤本機電腦上的檔案變更

Azure 自動化會使用監看員工作來監看事件和觸發動作。 本教學課程將逐步引導您建立監看員工作，以監視目錄中何時新增了檔案。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 匯入監看員 Runbook
> * 建立自動化變數
> * 建立動作 Runbook
> * 建立監看員工作
> * 觸發監看員
> * 檢查輸出

## <a name="prerequisites"></a>先決條件

若要完成此教學課程，需要有下列項目：

* Azure 訂用帳戶。 如果您沒有這類帳戶，可以[啟用自己的 MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或註冊[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* [自動化帳戶](automation-offering-get-started.md)以保存監看員和動作 Runbook，以及監看員工作。
* 執行監看員工作的[混合式 Runbook 背景工作角色](automation-hybrid-runbook-worker.md)。

## <a name="import-a-watcher-runbook"></a>匯入監看員 Runbook

本教學課程會使用名為 **Watch-NewFile** 的監看員 Runbook 來尋找目錄中的新檔案。 監看員 Runbook 會擷取資料夾檔案的最後已知寫入時間，並查看比該浮水印還新的任何檔案。 在此步驟中，您會將此 Runbook 匯入自動化帳戶。

1. 開啟自動化帳戶，然後按一下 [Runbook] 頁面。
1. 按一下 [瀏覽資源庫] 按鈕。
1. 搜尋「監看員 Runbook」，選取 [會尋找目錄中新檔案的監看員 Runbook]，然後選取 [匯入]。
  ![從 UI 匯入自動化 Runbook](media/automation-watchers-tutorial/importsourcewatcher.png)
1. 提供 Runbook 的名稱和描述，然後選取 [確定]，以將 Runbook 匯入自動化帳戶。
1. 選取 [編輯]，然後按一下 [發佈]。 出現提示時，選取 [是] 以發佈 Runbook。

## <a name="create-an-automation-variable"></a>建立自動化變數

[自動化變數](automation-variables.md)可用來儲存先前之 Runbook 從每個檔案所讀取並儲存的時間戳記。 

1. 選取 [共用資源] 底下的 [變數]，然後選取 [+ 新增變數]。
1. 輸入 "Watch-NewFileTimestamp" 作為 [名稱]
1. 選取 [日期時間] 作為 [類型]。
1. 按一下 [建立] 按鈕。 這會建立自動化變數。

## <a name="create-an-action-runbook"></a>建立動作 Runbook

在監看員工作中，動作 Runbook 會用來處理從監看員 Runbook 傳來的資料。 在此步驟中，您要更新匯入預先定義的動作 Runbook，其名稱為 "Process-NewFile"。

1. 瀏覽至自動化帳戶，然後選取 [程序自動化] 分類下的 [Runbook]。
1. 按一下 [瀏覽資源庫] 按鈕。
1. 搜尋「監看員動作」並選取 [處理監看員 Runbook 所觸發之事件的監看員動作]，然後選取 [匯入]。
  ![從 UI 匯入動作 Runbook](media/automation-watchers-tutorial/importsourceaction.png)
1. 提供 Runbook 的名稱和描述，然後選取 [確定]，以將 Runbook 匯入自動化帳戶。
1. 選取 [編輯]，然後按一下 [發佈]。 出現提示時，選取 [是] 以發佈 Runbook。

## <a name="create-a-watcher-task"></a>建立監看員工作

監看員工作包含兩個部分。 監看員和動作。 監看員會依據監看員工作中所定義的間隔來執行。 來自監看員 Runbook 的資料會傳遞至動作 Runbook。 在此步驟中，您要將監看員工作設定為參考前面步驟中所定義的監看員和動作 Runbook。

1. 瀏覽至自動化帳戶，然後選取 [程序自動化] 分類下的 [監看員工作]。
1. 選取 [監看員工作] 頁面，然後按一下 [+ 新增監看員工作] 按鈕。
1. 輸入 "WatchMyFolder" 作為名稱。

1. 選取 [設定監看員]，然後選取 [Watch-NewFile] Runbook。

1. 為各個參數輸入下列值︰

   * **FOLDERPATH** - 混合式背景工作角色上的資料夾，會於其中建立新檔案。 d:\examplefiles
   * **EXTENSION** - 保留空白以處理所有副檔名。
   * **RECURSE** - 請讓此值保持使用預設值。
   * **RUN SETTINGS** - 挑選混合式背景工作角色。

1. 按一下 [確定]，然後按一下 [選取] 以返回監看員頁面。
1. 選取 [設定動作]，然後選取 [Process-NewFile] Runbook。
1. 針對參數輸入下列值︰

   *    **EVENTDATA** - 保留為空白。 資料會從監看員 Runbook 傳入。  
   *    **回合設定** - 保留為 [Azure]，因為此 Runbook 是在自動化服務中執行。

1. 按一下 [確定]，然後按一下 [選取] 以返回監看員頁面。
1. 按一下 [確定] 以建立監看員工作。

![從 UI 設定監看員動作](media/automation-watchers-tutorial/watchertaskcreation.png)

## <a name="trigger-a-watcher"></a>觸發監看員

為了測試監看員是否會如預期般運作，您需要建立測試檔案。

遠端連線至混合式背景工作角色。 開啟 **PowerShell** 並在資料夾中建立測試檔案。
  
   ```PowerShell-interactive
   New-Item -Name ExampleFile1.txt
   ```

下列範例會顯示預期的輸出。

```
    Directory: D:\examplefiles


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       12/11/2017   9:05 PM              0 ExampleFile1.txt
```

## <a name="inspect-the-output"></a>檢查輸出

1. 瀏覽至自動化帳戶，然後選取 [程序自動化] 分類下的 [監看員工作]。
1. 選取 "WatchMyFolder" 監看員工作。
1. 按一下 [串流] 底下的 [檢視監看員串流]，確認監看員已找到新檔案並已啟動動作 Runbook。
1. 若要查看動作 Runbook 作業，請按一下 [檢視監看員動作作業]。 您可以選取各個作業以檢視作業的詳細資料。

   ![UI 中的監看員動作作業](media/automation-watchers-tutorial/WatcherActionJobs.png)

您可以在下列範例中看到系統找到新檔案時的預期輸出：

```
Message is Process new file...



Passed in data is @{FileName=D:\examplefiles\ExampleFile1.txt; Length=0}
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 匯入監看員 Runbook
> * 建立自動化變數
> * 建立動作 Runbook
> * 建立監看員工作
> * 觸發監看員
> * 檢查輸出

遵循此連結以深入了解如何編寫自己的 Runbook。

> [!div class="nextstepaction"]
> [我的第一個 PowerShell Runbook](automation-first-runbook-textual-powershell.md)。
