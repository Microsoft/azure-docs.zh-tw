---
title: Azure 自動化中的原始檔控制整合
description: 本文說明在 Azure 自動化中與 GitHub 的原始檔控制整合。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 09/17/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: c2d13a409d095bca64da781e5c5ca58553f9710c
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47045827"
---
# <a name="source-control-integration-in-azure-automation"></a>Azure 自動化中的原始檔控制整合

原始檔控制可讓您利用 GitHub 或 Azure Dev Ops 原始檔控制存放庫中的指令碼，使自動化帳戶中的 Runbook 保持最新狀態。 原始檔控制可讓您輕鬆地與小組共同作業、追蹤變更，以及回復至舊版的 Runbook。 例如，原始檔控制可讓您將原始檔控制中的不同分支同步處理至您的開發、測試或生產自動化帳戶，以輕鬆地將已在開發環境中測試過的程式碼提升至生產自動化帳戶。

Azure 自動化支援三種類型的原始檔控制：

* GitHub
* Visual Studio Team Services (Git)
* Visual Studio Team Services (TFVC)

## <a name="pre-requisites"></a>先決條件

* 原始檔控制存放庫 (GitHub 或 Visual Studio Team Services)
* [執行身分帳戶和連接](manage-runas-account.md)

> [!NOTE]
> 原始檔控制同步處理作業會在使用者的自動化帳戶之下執行，而且會以與其他自動化作業相同的費率計費。

## <a name="configure-source-control"></a>設定原始檔控制

在您的自動化帳戶內，選取 [原始檔控制 (預覽)]，然後按一下 [+ 新增]

![選取原始檔控制](./media/source-control-integration/select-source-control.png)

選擇 [原始檔控制類型]，按一下 [驗證]。

檢閱應用程式要求權限頁面，然後按一下 [接受]。

在 [原始檔控制摘要] 頁面上，填入資訊並按一下 [儲存]。 下表顯示可用欄位的說明。

|屬性  |說明  |
|---------|---------|
|原始檔控制名稱     | 原始檔控制的易記名稱        |
|原始檔控制類型     | 原始檔控制來源的類型。 可用選項包括：</br> Github</br>Visual Studio Team Services (Git)</br>Visual Studio Team Services (TFVC)        |
|存放庫     | 存放庫或專案的名稱。 這是提取自原始檔控制存放庫。 範例：$/ContosoFinanceTFVCExample         |
|分支     | 此分支可從中提取原始程式檔。 分支目標不適用於 TFVC 原始檔控制類型。          |
|資料夾路徑     | 包含要同步處理之 Runbook 的資料夾。範例：/Runbooks         |
|自動同步處理     | 在原始檔控制存放庫中進行認可時開啟或關閉自動同步處理         |
|發佈 Runbook     | 如果設定為 [開啟]，從原始檔控制同步處理 Runbook 之後，就會自動發佈 Runbook。         |
|說明     | 用來提供其他詳細資料的文字欄位        |

![原始檔控制摘要](./media/source-control-integration/source-control-summary.png)

## <a name="syncing"></a>同步處理

如果在設定原始檔控制整合時設定自動同步處理，則會自動啟動初始同步處理。 如果未設定自動同步處理，請從 [原始檔控制 (預覽)] 頁面上的表格中選取來源。 按一下 [開始同步處理] 開始同步處理程序。  

您可以按一下 [同步處理作業] 索引標籤，以檢視目前或先前同步處理作業的狀態。在 [原始檔控制] 下拉式清單中選取原始檔控制。

![同步處理狀態](./media/source-control-integration/sync-status.png)

按一下作業可讓您檢視作業輸出。 以下是原始檔控制同步作業的輸出範例。

```output
========================================================================================================

Azure Automation Source Control Public Preview.
Supported runbooks to sync: PowerShell Workflow, PowerShell Scripts, DSC Configurations, Graphical, and Python 2.

Setting AzureRmEnvironment.

Getting AzureRunAsConnection.

Logging in to Azure...

Source control information for syncing:

[Url = https://ContosoExample.visualstudio.com/ContosoFinanceTFVCExample/_versionControl] [FolderPath = /Runbooks]

Verifying url: https://ContosoExample.visualstudio.com/ContosoFinanceTFVCExample/_versionControl

Connecting to VSTS...


Source Control Sync Summary:


2 files synced:
 - ExampleRunbook1.ps1
 - ExampleRunbook2.ps1



========================================================================================================
```

## <a name="disconnecting-source-control"></a>中斷原始檔控制的連線

若要中斷與原始檔控制存放庫的連線，請在自動化帳戶的 [帳戶設定] 之下開啟 [原始檔控制 (預覽)]。

選取您想要移除的原始檔控制。 在 [原始檔控制摘要] 頁面上，按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

若要深入了解 Runbook 類型、其優點和限制，請參閱 [Azure 自動化 Runbook 類型](automation-runbook-types.md)
