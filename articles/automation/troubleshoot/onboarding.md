---
title: 針對將更新管理、變更追蹤和清查上線時的錯誤進行疑難排解
description: 了解如何針對將更新管理、變更追蹤和清查解決方案上線時的錯誤進行疑難排解
services: automation
author: georgewallace
ms.author: gwallace
ms.date: 06/19/2018
ms.topic: conceptual
ms.service: automation
manager: carmonm
ms.openlocfilehash: 044cb56b8991a1eb2dd6a1d35be621f2ffab3250
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37064247"
---
# <a name="troubleshoot-errors-when-onboarding-solutions"></a>針對將解決方案上線時的錯誤進行疑難排解

當您將更新管理或變更追蹤和清查等解決方案上線時，可能會發生錯誤。 本文描述可能發生的各種錯誤及解決方法。

## <a name="general-errors"></a>一般錯誤

### <a name="computer-grou-query-format-error"></a>案例：ComputerGroupQueryFormatError

#### <a name="issue"></a>問題

這個錯誤碼表示未正確地將用來將目標設定為解決方案的已儲存搜尋電腦群組查詢格式化。 

#### <a name="cause"></a>原因

您可能已變更查詢，或者系統可能已變更該查詢。

#### <a name="resolution"></a>解決方案

您可以刪除此解決方案的查詢，然後將解決方案重新上線，這樣會重新建立查詢。 您可以在工作區內的 [儲存的搜尋] 下方找到該查詢。 查詢的名稱是 **MicrosoftDefaultComputerGroup**，而查詢的類別是與這個查詢相關聯的解決方案名稱。 如果已啟用多個解決方案，**MicrosoftDefaultComputerGroup** 會在 [儲存的搜尋] 下方多次顯示。

### <a name="policy-violation"></a>案例：PolicyViolation

#### <a name="issue"></a>問題

這個錯誤碼表示部署因違反一個或多個原則而失敗。

#### <a name="cause"></a>原因 

有一個原則導致作業無法完成。

#### <a name="resolution"></a>解決方案

若要成功部署解決方案，您必須考量改變指定的原則。 由於可定義的原則有許多不同類型，因此所需的特定變更就要根據違反的原則來決定。 例如，如果在定義原則的資源群組中，拒絕對於該資源群組內特定類型資源之內容的變更權限，您可以執行下列任何一項作業：

* 完全移除原則。
* 嘗試上架到不同的資源群組。
* 透過下列方法修改原則，例如：
  * 將原則的目標重新設定為特定資源 (例如設定為特定的自動化帳戶)。
  * 對於原則已設定為拒絕的資源集合進行修改。

檢查 Azure 入口網站右上角的通知，或瀏覽至包含您自動化帳戶的資源群組，然後選取 [設定] 下方的 [部署] 來檢視失敗的部署。 若要深入了解 Azure 原則，請造訪：[Azure 原則的概觀](../../azure-policy/azure-policy-introduction.md?toc=%2fazure%2fautomation%2ftoc.json)。

## <a name="mma-extension-failures"></a>MMA 延伸模組失敗

部署解決方案時，會部署各種相關的資源。 其中一個資源是 Microsoft Monitoring Agent 延伸模組或適用於 Linux 的 OMS 代理程式。 這些是虛擬機器的客體代理程式所安裝的虛擬機器延伸模組，負責與設定的 Operations Management Suite (OMS) 工作區通訊，以在稍後協調下載二進位檔，以及您要上線的解決方案在開始執行之後相依的其他檔案。
您通常最先察覺 MMA 或適用於 Linux 的 OMS 代理程式安裝失敗，是透過通知中樞中出現的通知。 按一下通知可提供特定失敗的進一步資訊。 巡覽至資源群組資源，再到其中的部署項目，也會提供所發生部署失敗的詳細資料。
安裝 MMA 或適用於 Linux 的 OMS 代理程式失敗的原因可能有許多，解決這些失敗所採取的步驟會因問題而有所不同。 以下是特定疑難排解步驟。

下一節描述上線時可能發生並導致 MMA 延伸模組部署失敗的各種問題。

### <a name="webclient-exception"></a>案例：在 WebClient 要求期間發生例外狀況

虛擬機器上的 MMA 延伸模組無法與外部資源通訊且部署會失敗。

#### <a name="issue"></a>問題

以下是傳回的錯誤訊息範例：

```
Please verify the VM has a running VM agent, and can establish outbound connections to Azure storage.
```

```
'Manifest download error from https://<endpoint>/<endpointId>/Microsoft.EnterpriseCloud.Monitoring_MicrosoftMonitoringAgent_australiaeast_manifest.xml. Error: UnknownError. An exception occurred during a WebClient request.
```

#### <a name="cause"></a>原因

此錯誤的一些可能原因包括：

* VM 中已設定只允許特定連接埠的 Proxy。

* 防火牆設定已封鎖對必要連接埠和位址的存取。

#### <a name="resolution"></a>解決方案

確定您已開啟適當的連接埠和位址進行通訊。 如需連接埠和位址清單，請參閱[規劃您的網路](../automation-hybrid-runbook-worker.md#network-planning)。

### <a name="transient-environment-issue"></a>案例：安裝因暫時性環境問題而失敗

由於其他安裝或動作封鎖安裝，在部署期間安裝 Microsoft Monitoring Agent 延伸模組會失敗

#### <a name="issue"></a>問題

以下是可能傳回的錯誤訊息範例：

```
The Microsoft Monitoring Agent failed to install on this machine. Please try to uninstall and reinstall the extension. If the issue persists, please contact support.
```

```
'Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.4) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.4\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 1618'
```

```
'Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.2) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.2\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 1601'
```

#### <a name="cause"></a>原因

此錯誤的一些可能原因包括：

* 其他安裝正在進行
* 在範本部署期間已觸發系統重新啟動

#### <a name="resolution"></a>解決方案

此錯誤本質上是暫時性錯誤。 請重試部署以安裝延伸模組。

### <a name="installation-timeout"></a>案例：安裝逾時

MMA 延伸模組的安裝因逾時而未完成。

#### <a name="issue"></a>問題

以下是可能傳回的錯誤訊息範例：

```
Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.4) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.4\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 15614
```

#### <a name="cause"></a>原因

此錯誤是由於虛擬機器在安裝期間的負載過重所致。

### <a name="resolution"></a>解決方案

請在 VM 負載較低時，嘗試安裝 MMA 延伸模組。

## <a name="next-steps"></a>後續步驟

如果您看不到問題或無法解決問題，請瀏覽下列其中一個通道以取得更多支援：

* 透過 [Azure 論壇](https://azure.microsoft.com/support/forums/)獲得由 Azure 專家所提供的解答
* 與 [@AzureSupport](https://twitter.com/azuresupport) 連繫－專為改善客戶體驗而設的官方 Microsoft Azure 帳戶，協助 Azure 社群連接至適當的資源，像是解答、支援及專家等。
* 如果需要更多協助，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。
