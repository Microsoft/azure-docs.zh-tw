---
title: Azure 自動化之圖形化 Runbook 中的錯誤處理
description: 本文說明如何在 Azure 自動化的圖形化 Runbook 中實作錯誤處理邏輯。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 03/16/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 14112a9a9f64f20540a8f7e1d37cd31017238e59
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34192705"
---
# <a name="error-handling-in-azure-automation-graphical-runbooks"></a>Azure 自動化之圖形化 Runbook 中的錯誤處理

所要考量的主要 Runbook 設計主體是找出 Runbook 可能遇到的不同問題。 這些問題包括成功、預期的錯誤狀態，以及非預期的錯誤情況。

Runbook 應該包含錯誤處理。 若要驗證活動的輸出，或以圖形化 Runbook 處理錯誤，您可以使用 Windows PowerShell 程式碼活動、定義活動輸出連結的條件式邏輯，或套用其他方法。          

如果 Runbook 活動發生非終止錯誤，其後的任何活動往往不管此錯誤仍會進行處理。 此錯誤可能會產生例外狀況，但仍允許執行下一個活動。 這是 PowerShell 設計用來處理錯誤的方式。    

執行期間可能發生的 PowerShell 錯誤類型為終止或非終止。 終止和非終止錯誤之間的差異如下︰

* **終止錯誤**︰執行期間所發生的嚴重錯誤，此錯誤會完全終止命令 (或指令碼執行)。 範例包括不存在的 Cmdlet、防止 Cmdlet 執行的語法錯誤，或其他嚴重錯誤。

* **非終止錯誤**︰非嚴重錯誤，會允許繼續執行，而不理會失敗。 範例包括操作錯誤，例如找不到檔案錯誤和權限問題。

Azure 自動化的圖形化 Runbook 已納入錯誤處理功能。 您現在可以將例外狀況變成非終止錯誤，並建立活動之間的錯誤連結。 此處理可讓 Runbook 作者捕捉到錯誤，以及管理已實現或非預期的狀況。  

## <a name="when-to-use-error-handling"></a>何時使用錯誤處理

每當有重要活動擲回錯誤或例外狀況時，務必要避免繼續處理 Runbook 中的下一個活動，並適當地處理錯誤。 當 Runbook 支援商務或服務營運流程時，這點格外重要。

對於每個可產生錯誤的活動，Runbook 作者均可新增指向任何其他活動的錯誤連結。 目的地活動可以是任何類型，包括程式碼活動、叫用 Cmdlet、叫用另一個 Runbook 等等。

此外，目的地活動也可以有連出連結。 這些連結可以是一般連結或錯誤連結。 這表示 Runbook 作者可以實作複雜的錯誤處理邏輯，而不需向程式碼活動報告。 建議作法是建立具有常用功能的專用錯誤處理 Runbook，但您不一定要照做。 將錯誤處理邏輯放在 PowerShell 程式碼活動中也並非是唯一的選項。  

例如，請考慮嘗試啟動 VM 的 Runbook 並在其上安裝應用程式。 如果 VM 未正確啟動，它會執行兩個動作︰

1. 傳送關於這個問題的通知。
2. 改為啟動另一個會自動佈建新 VM 的 Runbook。

解決方案之一是擁有錯誤連結，以指向可處理步驟 1 的活動。 例如，您可以將 **Write-Warning** Cmdlet 連接至步驟 2 的活動，例如 **Start-AzureRmAutomationRunbook** Cmdlet。

將這兩個活動放在不同的錯誤處理 Runbook 中，並遵循稍早建議的指導方針，也可以讓此行為廣泛用於許多 Runbook。 在呼叫此錯誤處理 Runbook 之前，您可以從原始 Runbook 中的資料建構自訂訊息，然後將此訊息做為參數傳遞至錯誤處理 Runbook。

## <a name="how-to-use-error-handling"></a>如何使用錯誤處理

每個活動均有組態設定可將例外狀況變成非終止錯誤。 此設定預設為停用狀態。 建議您在想要用來處理錯誤的任何活動上啟用此設定。  

藉由啟用此組態，您可以確保活動中的終止和非終止錯誤都會被當作非終止錯誤處理，並且可透過錯誤連結進行處理。  

進行此設定之後，您就可以建立可處理錯誤的活動。 如果活動產生任何錯誤，則會出現連出的錯誤連結，而不會出現一般連結，即使活動也產生一般輸出。<br><br> ![自動化 Runbook 的錯誤連結範例](media/automation-runbook-graphical-error-handling/error-link-example.png)

在下列範例中，Runbook 會擷取一個變數，其中包含虛擬機器的電腦名稱。 它會嘗試使用下一個活動啟動虛擬機器。<br><br> ![自動化 Runbook 的錯誤處理範例](media/automation-runbook-graphical-error-handling/runbook-example-error-handling.png)<br><br>      

依設定，**Get-AutomationVariable** 活動和 **Start-AzureRmVm** 會將例外狀況轉換為錯誤。 如果無法取得變數或啟動 VM，就會產生錯誤。<br><br> ![自動化 Runbook 的錯誤處理活動設定](media/automation-runbook-graphical-error-handling/activity-blade-convertexception-option.png)

錯誤連結會從這些活動流向單一**錯誤管理**活動 (程式碼活動)。 此活動已設定了簡單的 PowerShell 運算式，其使用 Throw 關鍵字來停止處理，以及使用 $Error.Exception.Message 來取得說明目前例外狀況的訊息。<br><br> ![自動化 Runbook 的錯誤處理程式碼範例](media/automation-runbook-graphical-error-handling/runbook-example-error-handling-code.png)


## <a name="next-steps"></a>後續步驟

* 若要深入了解圖形化 Runbook 中的連結和連結類型，請參閱 [Azure 自動化中的圖形化編寫](automation-graphical-authoring-intro.md#links-and-workflow)。

* 若要深入了解 Runbook 執行方式、如何監視 Runbook 作業，以及其他技術性詳細資料，請參閱[追蹤 Runbook 作業](automation-runbook-execution.md)。
