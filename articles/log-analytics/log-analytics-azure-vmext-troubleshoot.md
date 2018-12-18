---
title: 針對 Azure Log Analytics VM 擴充功能進行疑難排解 | Microsoft Docs
description: 說明適用於 Windows 與 Linux Azure VM 的 Log Analytics VM 擴充功能最常見問題的徵兆、原因和解決方法。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: ''
ms.service: log-analytics
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/08/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 700d6b2c3bcd39aed38bf75556bcdcb59d1ab78b
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37128821"
---
# <a name="troubleshooting-the-log-analytics-vm-extension"></a>針對 Log Analytics VM 擴充功能進行疑難排解
本文可協助您針對在使用執行 Microsoft Azure 之 Windows 與 Linux 虛擬機器的 Log Analytics VM 擴充功能時可能遇到的錯誤進行疑難排解，並建議可能的解決方法。

若要確認擴充功能的狀態，請從 Azure 入口網站執行下列步驟。

1. 登入 [Azure 入口網站](http://portal.azure.com)。
2. 在 Azure 入口網站中，按一下 [所有服務]。 在資源清單中，輸入**虛擬機器**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [虛擬機器]。
3. 在虛擬機器清單中，尋找目標虛擬機器並加以選取。
3. 在虛擬機器上，按一下 [擴充功能]。
4. 檢查清單以查看是否已啟用 Log Analytics 擴充功能。  針對 Linux，代理程式將會被列為 **OMSAgentforLinux**。針對 Windows，代理程式將會被列為 **MicrosoftMonitoringAgent**。

   ![VM 擴充功能檢視](./media/log-analytics-azure-vmext-troubleshoot/log-analytics-vmview-extensions.png)

4. 按一下擴充功能以檢視其詳細資料。 

   ![VM 擴充功能詳細資料](./media/log-analytics-azure-vmext-troubleshoot/log-analytics-vmview-extensiondetails.png)

## <a name="troubleshooting-azure-windows-vm-extension"></a>針對 Azure Windows VM 擴充功能進行疑難排解

如果 *Microsoft Monitoring Agent* VM 擴充功能尚未安裝或沒有回報，您可以執行下列步驟來針對問題進行疑難排解。

1. 使用 [KB 2965986](https://support.microsoft.com/kb/2965986#mt1) 中的步驟，檢查 Azure VM 代理程式是否已安裝且正確運作。
   * 您也可以檢閱 VM 代理程式記錄檔 `C:\WindowsAzure\logs\WaAppAgent.log`
   * 如果記錄檔不存在，則表示未安裝 VM 代理程式。
   * [安裝 Azure VM 代理程式](log-analytics-quick-collect-azurevm.md#enable-the-log-analytics-vm-extension)
2. 使用下列步驟，確認 Microsoft Monitoring Agent 擴充活動訊號工作正在執行︰
   * 登入虛擬機器
   * 開啟工作排程器，找出 `update_azureoperationalinsight_agent_heartbeat` 工作
   * 確認此工作已啟用，且每一分鐘執行一次
   * 檢查 `C:\WindowsAzure\Logs\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\heartbeat.log` 中的活動訊號記錄檔
3. 檢閱 `C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent` 中的 Microsoft Monitoring Agent VM 擴充記錄檔
4. 確定虛擬機器可以執行 PowerShell 指令碼
5. 確定 C:\Windows\temp 的權限未變更
6. 在虛擬機器上提高權限的 PowerShell 視窗中輸入下列命令，以檢視 Microsoft Monitoring Agent 的狀態 `  (New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg').GetCloudWorkspaces() | Format-List`
7. 檢閱 `C:\Windows\System32\config\systemprofile\AppData\Local\SCOM\Logs` 中的 Microsoft Monitoring Agent 安裝記錄檔

如需詳細資訊，請參閱[針對 Windows 擴充功能進行疑難排解](../virtual-machines/windows/extensions-oms.md)。

## <a name="troubleshooting-linux-vm-extension"></a>針對 Linux VM 擴充功能進行疑難排解
如果*適用於 Linux 的 OMS 代理程式* VM 擴充功能尚未安裝或沒有回報，您可以執行下列步驟來針對問題進行疑難排解。

1. 如果擴充狀態是「未知」，請檢閱 VM 代理程式記錄檔 `/var/log/waagent.log`，以檢查 Azure VM 代理程式是否已安裝且正常運作
   * 如果記錄檔不存在，則表示未安裝 VM 代理程式。
   * [在 Linux VM 上安裝 Azure VM 代理程式](log-analytics-quick-collect-azurevm.md#enable-the-log-analytics-vm-extension)
2. 若是其他不良狀態，請檢閱 `/var/log/azure/Microsoft.EnterpriseCloud.Monitoring.OmsAgentForLinux/*/extension.log` 和 `/var/log/azure/Microsoft.EnterpriseCloud.Monitoring.OmsAgentForLinux/*/CommandExecution.log` 中的 OMS Agent for Linux VM 擴充記錄檔
3. 如果擴充狀態良好，但資料未上傳，請檢閱 `/var/opt/microsoft/omsagent/log/omsagent.log` 中的 OMS Agent for Linux 記錄檔

如需詳細資訊，請參閱[針對 Linux 擴充功能進行疑難排解](../virtual-machines/linux/extensions-oms.md)。

## <a name="next-steps"></a>後續步驟

如需與在 Azure 外部電腦上所裝載之適用於 Linux 的 OMS 代理程式相關的其他疑難排解指引，請參閱[針對 Azure Log Analytics Linux 代理程式進行疑難排解](log-analytics-agent-linux-support.md)。  
