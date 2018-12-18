---
title: 針對 Azure 自動化 Desired State Configuration (DSC) 問題進行疑難排解
description: 本文提供有關針對 Desired State Configuration (DSC) 問題進行疑難排解的資訊
services: automation
ms.service: automation
ms.component: ''
author: georgewallace
ms.author: gwallace
ms.date: 06/19/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 5d2eae67fcff74a7016f7f6125e31a9c8c2bda97
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37064410"
---
# <a name="troubleshoot-desired-state-configuration-dsc"></a>針對 Desired State Configuration (DSC) 問題進行疑難排解

本文提供有關針對 Desired State Configuration (DSC) 問題進行疑難排解的資訊。

## <a name="common-errors-when-working-with-desired-state-configuration-dsc"></a>使用 Desired State Configuration (DSC) 時的常見錯誤

### <a name="failed-not-found"></a>案例：節點處於失敗狀態，並發生「找不到」錯誤

#### <a name="issue"></a>問題

節點的報告具有「失敗」狀態且包含錯誤：

```
The attempt to get the action from server https://<url>//accounts/<account-id>/Nodes(AgentId=<agent-id>)/GetDscAction failed because a valid configuration <guid> cannot be found.
```

#### <a name="cause"></a>原因

此錯誤通常發生在將節點指派給設定名稱 (例如 ABC)，而不是指派給節點設定名稱 (例如 ABC.WebServer) 的情況下。

#### <a name="resolution"></a>解決方式

* 請確定您會使用「節點組態名稱」而不是「組態名稱」來指派節點。
* 您可以使用 Azure 入口網站或使用 PowerShell Cmdlet，將節點組態指派至節點。

  * 如要使用 Azure 入口網站來將節點組態指派給節點，請開啟 [DSC 節點] 分頁、選取某個節點，然後按一下 [指派節點組態] 按鈕。  
  * 如要使用 PowerShell Cmdlet 來將節點組態指派給節點，請使用 **Set-AzureRmAutomationDscNode** Cmdlet

### <a name="no-mof-files"></a>案例：編譯設定時，沒有產生任何節點設定 (MOF 檔案)

#### <a name="issue"></a>問題

您的 DSC 編譯工作因發生下列錯誤而暫停：

```
Compilation completed successfully, but no node configuration.mofs were generated.
```

#### <a name="cause"></a>原因

當 DSC 設定中緊接在 **Node** 關鍵字後面的運算式評估為 `$null` 時，便不會產生任何節點設定。

#### <a name="resolution"></a>解決方式

下列任何一個解決方案都可以修正此問題：

* 請確定組態定義中「Node」  關鍵字後方的運算式，不會被評估為 $null。
* 如果您在編譯組態時傳遞 ConfigurationData，請確定您傳遞的是組態向 [ConfigurationData](../automation-dsc-compile.md#configurationdata)要求的預期值。

### <a name="dsc-in-progress"></a>案例：DSC 節點報告變成停留在「進行中」狀態

#### <a name="issue"></a>問題

DSC 代理程式輸出：

```
No instance found with given property values
```

#### <a name="cause"></a>原因

您已將 WMF 版本升級，且有損毀的 WMI。

#### <a name="resolution"></a>解決方式

若要修正問題，請依照 [DSC 已知問題和限制](https://msdn.microsoft.com/powershell/wmf/5.0/limitation_dsc)一文中的指示進行操作。

### <a name="issue-using-credential"></a>案例：無法在 DSC 設定中使用認證

#### <a name="issue"></a>問題

您的 DSC 編譯工作因發生下列錯誤而暫停：

```
System.InvalidOperationException error processing property 'Credential' of type <some resource name>: Converting and storing an encrypted password as plaintext is allowed only if PSDscAllowPlainTextPassword is set to true.
```

#### <a name="cause"></a>原因

您已在設定中使用認證，但沒有提供適當的 **ConfigurationData**，以將每個節點設定的 **PSDscAllowPlainTextPassword** 設定為 True。

#### <a name="resolution"></a>解決方式

* 請確認傳入適當的 **ConfigurationData**，以便把組態中提到的每個節點組態的 **PSDscAllowPlainTextPassword** 設定為 True。 如需詳細資訊，請參閱 [Azure 自動化 DSC 中的資產](../automation-dsc-compile.md#assets)。

## <a name="next-steps"></a>後續步驟

如果您看不到問題或無法解決問題，請瀏覽下列其中一個管道以取得更多支援：

* 透過 [Azure 論壇](https://azure.microsoft.com/support/forums/)獲得由 Azure 專家所提供的解答
* 與 [@AzureSupport](https://twitter.com/azuresupport) 連繫－專為改善客戶體驗而設的官方 Microsoft Azure 帳戶，協助 Azure 社群連接至適當的資源，像是解答、支援及專家等。
* 如果需要更多協助，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。