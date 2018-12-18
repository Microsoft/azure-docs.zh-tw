---
title: 在 Azure 自動化中測試 Runbook
description: 在 Azure 自動化中發佈 Runbook 之前，您可以先加以測試，以確保 Runbook 能夠如預期般運作。  本文說明如何測試 Runbook 及檢視其輸出。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 03/16/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: ebeaa8eb75373fc94f7e4e714e36d1167fd7f060
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34192093"
---
# <a name="testing-a-runbook-in-azure-automation"></a>在 Azure 自動化中測試 Runbook
當您測試 Runbook 時， [草稿版本](automation-creating-importing-runbook.md#publishing-a-runbook) 會執行，而且它執行的任何動作都會完成。 不會建立任何工作歷程記錄，但 [測試輸出] 窗格中會顯示[輸出](automation-runbook-output-and-messages.md#output-stream)與[警告和錯誤](automation-runbook-output-and-messages.md#message-streams)串流。 只有將 [$VerbosePreference 變數](automation-runbook-output-and-messages.md#preference-variables)設為 Continue，傳送給[詳細資訊串流](automation-runbook-output-and-messages.md#message-streams)的訊息才會顯示在 [輸出] 窗格中。

即使執行的是草稿版本，Runbook 仍會正常執行工作流程，並會針對環境中的資源執行任何動作。 因此，您只能在非生產資源中測試 Runbook。

測試每個 [Runbook 類型](automation-runbook-types.md) 的程序都相同，而且無論是在文字式編輯器或 Azure 入口網站的圖形化編輯器中進行測試，都沒有任何差別。  

## <a name="to-test-a-runbook-in-the-azure-portal"></a>在 Azure 入口網站中測試 Runbook
您可以在 Azure 入口網站中使用任何 [Runbook 類型](automation-runbook-types.md) 。

1. 您可以在[文字編輯器](automation-edit-textual-runbook.md)或[圖形化編輯器](automation-graphical-authoring-intro.md)中開啟 Runbook 的草稿版本。
2. 按一下  [測試] 按鈕以開啟 [測試] 刀鋒視窗。
3. 如果 Runbook 含有參數，這些參數會列在左窗格中，您可以在此提供值，以供測試使用。
4. 如果您想要在[混合式 Runbook 背景工作](automation-hybrid-runbook-worker.md)上執行測試，則需將 [執行設定] 變更為 [混合式背景工作]，然後選取目標群組的名稱。  否則，請保留預設值 **Azure**，以便在雲端中執行測試。
5. 按一下 [啟動]  按鈕以啟動測試。
6. 如果 Runbook 是 [PowerShell 工作流程](automation-runbook-types.md#powershell-workflow-runbooks)或[圖形化](automation-runbook-types.md#graphical-runbooks) Runbook，則在測試過程中，您可以使用 [輸出] 窗格下方的按鈕來停止或暫停 Runbook。 暫停 Runbook 時，它會先完成目前的活動後才暫停。 Runbook 一旦暫停，您可以選擇停止或重新啟動。
7. 您可以在 [輸出] 窗格中檢查 Runbook 的輸出。

## <a name="next-steps"></a>後續步驟
* 若要了解如何建立或匯入 Runbook，請參閱 [建立或匯入 Azure 自動化中的 Runbook](automation-creating-importing-runbook.md)
* 若要深入了解圖形化編寫，請參閱 [Azure 自動化中的圖形化編寫](automation-graphical-authoring-intro.md)
* 若要開始使用 PowerShell 工作流程 Runbook，請參閱 [我的第一個 PowerShell 工作流程 Runbook](automation-first-runbook-textual.md)
* 若要深入了解設定 Runbook 以傳回狀態訊息和錯誤，包括建議的做法，請參閱 [Azure 自動化中的 Runbook 輸出和訊息](automation-runbook-output-and-messages.md)

