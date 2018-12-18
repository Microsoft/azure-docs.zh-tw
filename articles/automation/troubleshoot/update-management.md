---
title: 針對更新管理的相關錯誤進行疑難排解
description: 了解如何針對更新管理問題進行疑難排解
services: automation
author: georgewallace
ms.author: gwallace
ms.date: 08/08/2018
ms.topic: conceptual
ms.service: automation
manager: carmonm
ms.openlocfilehash: 2e47320d5ad88edfa8ea6122f3a0abd104230974
ms.sourcegitcommit: 7b845d3b9a5a4487d5df89906cc5d5bbdb0507c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "42146357"
---
# <a name="troubleshooting-issues-with-update-management"></a>針對更新管理問題進行疑難排解

本文探討您使用「更新管理」時可能遇到的問題及其解決方案。

## <a name="general"></a>一般

### <a name="components-enabled-not-working"></a>案例：已啟用「更新管理」解決方案的元件，而此虛擬機器現在正在設定中

#### <a name="issue"></a>問題

上架後 15 分鐘，您會繼續在虛擬機器上看見下列訊息：

```
The components for the 'Update Management' solution have been enabled, and now this virtual machine is being configured. Please be patient, as this can sometimes take up to 15 minutes.
```

#### <a name="cause"></a>原因

這個錯誤可能是下列原因所造成：

1. 正在封鎖連回自動化帳戶的通訊。
2. 正在上架的 VM 可能來自已安裝 Microsoft Monitoring Agent 但未執行過 sysprep 的所複製機器。

#### <a name="resolution"></a>解決方案

1. 請瀏覽[網路規劃](../automation-hybrid-runbook-worker.md#network-planning)，了解必須允許哪些位址和連接埠，才能進行更新管理。
2. 如果使用複製的映像，請先對映像執行 sysprep，然後再安裝 MMA 代理程式。

## <a name="windows"></a>Windows

如果您在嘗試讓解決方案在虛擬機器上線時遇到問題，請查看本機電腦上 [應用程式及服務記錄檔] 底下的 [Operations Manager] 事件記錄檔中，是否有事件識別碼為 **4502** 且事件訊息包含 **Microsoft.EnterpriseManagement.HealthService.AzureAutomation.HybridAgent** 的事件。

下節特別說明特定的錯誤訊息，以及每個錯誤訊息的可能解決方式。 如需了解其他上線問題，請參閱[針對解決方案上線問題進行疑難排解](onboarding.md)。

### <a name="machine-already-registered"></a>案例：電腦已經註冊至不同的帳戶

#### <a name="issue"></a>問題

您收到下列錯誤訊息：

```
Unable to Register Machine for Patch Management, Registration Failed with Exception System.InvalidOperationException: {"Message":"Machine is already registered to a different account."}
```

#### <a name="cause"></a>原因

電腦已經在另一個工作區上線以供進行「更新管理」。

#### <a name="resolution"></a>解決方案

在電腦上藉由[刪除混合式 Runbook 群組](../automation-hybrid-runbook-worker.md#remove-a-hybrid-worker-group)來清除舊成品，然後再試一次。

### <a name="machine-unable-to-communicate"></a>案例：電腦無法與服務進行通訊

#### <a name="issue"></a>問題

您會收到下列其中一個錯誤訊息：

```
Unable to Register Machine for Patch Management, Registration Failed with Exception System.Net.Http.HttpRequestException: An error occurred while sending the request. ---> System.Net.WebException: The underlying connection was closed: An unexpected error occurred on a receive. ---> System.ComponentModel.Win32Exception: The client and server can't communicate, because they do not possess a common algorithm
```

```
Unable to Register Machine for Patch Management, Registration Failed with Exception Newtonsoft.Json.JsonReaderException: Error parsing positive infinity value.
```

```
The certificate presented by the service <wsid>.oms.opinsights.azure.com was not issued by a certificate authority used for Microsoft services. Contact your network administrator to see if they are running a proxy that intercepts TLS/SSL communication.
```

#### <a name="cause"></a>原因

可能有 Proxy、閘道或防火牆封鎖網路通訊。

#### <a name="resolution"></a>解決方案

檢閱您的網路並確定已允許適當的連接埠和位址。 如需「更新管理」和「混合式 Runbook 背景工作角色」所需的連接埠和位址清單，請參閱[網路需求](../automation-hybrid-runbook-worker.md#network-planning)。

### <a name="unable-to-create-selfsigned-cert"></a>案例：無法建立自我簽署憑證

#### <a name="issue"></a>問題

您會收到下列其中一個錯誤訊息：

```
Unable to Register Machine for Patch Management, Registration Failed with Exception AgentService.HybridRegistration. PowerShell.Certificates.CertificateCreationException: Failed to create a self-signed certificate. ---> System.UnauthorizedAccessException: Access is denied.
```

#### <a name="cause"></a>原因

「混合式 Runbook 背景工作角色」無法產生自我簽署憑證

#### <a name="resolution"></a>解決方案

確認系統帳戶具有 **C:\ProgramData\Microsoft\Crypto\RSA** 資料夾的讀取存取權，然後再試一次。

## <a name="linux"></a>Linux

### <a name="scenario-update-run-fails-to-start"></a>案例：更新執行無法開始

#### <a name="issue"></a>問題

更新執行無法在 Linux 電腦上開始。

#### <a name="cause"></a>原因

「Linux 混合式背景工作角色」狀況不良。

#### <a name="resolution"></a>解決方案

複製以下記錄檔並將其保留供疑難排解使用：

```
/var/opt/microsoft/omsagent/run/automationworker/worker.log
```

### <a name="scenario-update-run-starts-but-encounters-errors"></a>案例：更新執行開始，但發生錯誤

#### <a name="issue"></a>問題

更新執行開始，但在執行時發生錯誤。

#### <a name="cause"></a>原因

可能的原因：

* 套件管理員狀況不良
* 特定套件可能干擾雲端式修補
* 其他原因

#### <a name="resolution"></a>解決方案

如果在更新執行於 Linux 上成功開始後，於更新執行期間發生失敗，請查看來自該執行中受影響電腦的工作輸出。 您可能可以從電腦套件管理員找到可供研究和採取動作的特定錯誤訊息。 「更新管理」必須在套件管理員狀況良好的情況下，才能進行成功的更新部署。

在某些情況下，套件更新可能干擾「更新管理」，而導致更新部署無法完成。 如果看到此情況，您將必須將這些套件從未來的更新執行中排除，或自行手動安裝這些套件。

如果您無法解決修補問題，請在下一個更新部署開始**之前**，先複製以下記錄檔並將其保留供疑難排解使用：

```
/var/opt/microsoft/omsagent/run/automationworker/omsupdatemgmt.log
```

## <a name="next-steps"></a>後續步驟

如果您看不到問題或無法解決問題，請瀏覽下列其中一個通道以取得更多支援：

* 透過 [Azure 論壇](https://azure.microsoft.com/support/forums/)獲得由 Azure 專家所提供的解答
* 與 [@AzureSupport](https://twitter.com/azuresupport) 連繫－專為改善客戶體驗而設的官方 Microsoft Azure 帳戶，協助 Azure 社群連接至適當的資源，像是解答、支援及專家等。
* 如果需要更多協助，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。