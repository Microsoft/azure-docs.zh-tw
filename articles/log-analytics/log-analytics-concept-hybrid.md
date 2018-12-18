---
title: 使用 Azure Log Analytics 代理程式在混合式環境中收集資料 | Microsoft Docs
description: 此主題可協助您了解如何使用 Log Analytics 來收集資料並監視您內部部署環境或其他雲端環境中裝載的電腦。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/03/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 96feb52bd5702c899faa8d845969ae8ba0995504
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39495351"
---
# <a name="collect-data-in-a-hybrid-environment-with-log-analytics-agent"></a>使用 Log Analytics 代理程式在混合式環境中收集資料

Azure Log Analytics 可以從執行 Windows 或 Linux 作業系統的下列電腦收集並處理資料：

* [Azure 虛擬機器](log-analytics-quick-collect-azurevm.md) (使用 Log Analytics VM 延伸模組) 
* 您的資料中心作為實體伺服器或虛擬機器
* 雲端代管服務 (例如 Amazon Web 服務 (AWS)) 中的虛擬機器

在您環境中裝載的電腦可以直接連線到 Log Analytics。或者，如果您已經使用 System Center Operations Manager 2012 R2 或更新版本來監視這些電腦，則可以整合 Operations Manage 管理群組與 Log Analytics，並繼續維持您的 IT 服務作業程序。  

## <a name="overview"></a>概觀

![log-analytics-agent-direct-connect-diagram](media/log-analytics-concept-hybrid/log-analytics-on-prem-comms.png)

分析並處理收集的資料之前，您必須先在每一部您想要傳送資料到 Log Analytics 服務的電腦上安裝代理程式並連線。 您可以使用安裝程式、命令列或 Azure 自動化中的預期狀態設定 (DSC) 在您的內部部署電腦上安裝代理程式。 

適用於 Linux 與 Windows 的代理程式會透過 TCP 連接埠 443 對外與 Log Analytics 服務通訊，且如果電腦連線至防火牆或 Proxy 伺服器以透過網際網路通訊，請檢閱下面的需求，以了解所需的網路設定。  如果您的 IT 安全性原則不允許網路上的電腦連線到網際網路，您可以設定 [OMS 閘道](log-analytics-oms-gateway.md)，然後將代理程式設定為透過閘道連線至 Log Analytics。 代理程式就可以接收設定資訊，並傳送根據所啟用資料收集規則和解決方案來收集的資料。 

如果您已使用 System Center Operations Manager 2012 R2 或更新版本來監視電腦，則該電腦可以具有多重主目錄，並使用 Log Analytics 服務來收集資料並轉送到該服務，且仍然受到 [Operations Manager](log-analytics-om-agents.md) 監視。 由與 Log Analytics 整合之 Operations Manager 管理群組監視的 Linux 電腦不會收到資料來源設定並透過管理群組轉送收集的資料。 Windows 代理程式可已回報到最多四個工作區，而 Linux 代理程式只支援回報到單一工作區。  

適用於 Linux 與 Windows 的代理程式不僅可用於連線到 Log Analytics，也支援「Azure 自動化」以裝載「混合式 Runbook」背景工作角色與管理解決方案，例如「變更追蹤與更新管理」。  如需有關「混合式 Runbook」背景工作角色的詳細資訊，請參閱 [Azure 自動化混合式 Runbook 背景工作](../automation/automation-hybrid-runbook-worker.md)。  

## <a name="supported-windows-operating-systems"></a>支援的 Windows 作業系統
Windows 代理程式正式支援下列 Windows 作業系統版本：

* Windows Server 2008 Service Pack 1 (SP1) 或更新版本
* Windows 7 SP1 與更新版本

## <a name="supported-linux-operating-systems"></a>支援的 Linux 作業系統
以下為正式支援的 Linux 散發套件。  不過，Linux 代理程式也可能在未列出的其他散發套件上執行。  除非另有說明，列出的每個主要版本都支援所有次要版本。  

* Amazon Linux 2012.09 至 2015.09 (x86/x64)
* CentOS Linux 5、6 和 7 (x86/x64)  
* Oracle Linux 5、6 和 7 (x86/x64) 
* Red Hat Enterprise Linux Server 5、6 和 7 (x86/x64)
* Debian GNU/Linux 6、7 和 8 (x86/x64)
* Ubuntu 12.04 LTS、14.04 LTS、16.04 LTS (x86/x64)
* SUSE Linux Enterprise Server 11 和 12 (x86/x64)

## <a name="tls-12-protocol"></a>TLS 1.2 通訊協定
為了確保資料傳送至 Log Analytics 時的安全性，我們強烈建議您將代理程式設定為至少使用傳輸層安全性 (TLS) 1.2。 我們已發現較舊版本的 TLS/安全通訊端層 (SSL) 較易受到攻擊，而且在其目前的運作中仍允許回溯相容性，因此並**不建議使用**這些版本。  如需其它資訊，請檢閱[使用 TLS 1.2 安全地傳送](log-analytics-data-security.md#sending-data-securely-using-tls-12)。 

## <a name="network-firewall-requirements"></a>網路防火牆需求
下列資訊列出 Linux 和 Windows 代理程式與 Log Analytics 通訊所需的 Proxy 和防火牆設定資訊。  

|代理程式資源|連接埠 |方向 |略過 HTTPS 檢查|
|------|---------|--------|--------|   
|*.ods.opinsights.azure.com |連接埠 443 |輸入和輸出|是 |  
|*.oms.opinsights.azure.com |連接埠 443 |輸入和輸出|是 |  
|*.blob.core.windows.net |連接埠 443 |輸入和輸出|是 |  
|*.azure-automation.net |連接埠 443 |輸入和輸出|是 |  


如果您打算使用 Azure 自動化混合式 Runbook 背景工作角色連線到自動化服務並向其註冊，以便在您的環境中使用 Runbook，它必須具有[設定適用於混合式 Runbook 背景工作角色的網路](../automation/automation-hybrid-runbook-worker.md#network-planning)中所述的連接埠號碼和 URL 存取權。 

Windows 和 Linux 代理程式支援使用 HTTPS 通訊協定，透過 Proxy 伺服器或 OMS 閘道，與 Log Analytics 服務進行通訊。  不支援匿名和基本驗證 (使用者名稱/密碼)。  若是直接連線到服務的 Windows 代理程式，請在安裝期間或[部署後](log-analytics-agent-manage.md#update-proxy-settings)從控制台或使用 PowerShell 來指定 Proxy 設定。  

若是 Linux 代理程式，請在安裝期間或[安裝後](log-analytics-agent-manage.md#update-proxy-settings)透過修改 proxy.conf 設定檔來指定 Proxy 伺服器。  Linux 代理程式 Proxy 設定值的語法如下：

`[protocol://][user:password@]proxyhost[:port]`

> [!NOTE]
> 若您的 Proxy 伺服器不要求您進行驗證，Linux 代理程式仍會要求提供虛擬使用者/密碼。 這可以是任何使用者名稱或密碼。

|屬性| 說明 |
|--------|-------------|
|通訊協定 | https |
|user | 用於驗證 Proxy 的選擇性使用者名稱 |
|password | 用於驗證 Proxy 的選擇性密碼 |
|proxyhost | Proxy 伺服器/OMS 閘道的位址或 FQDN |
|連接埠 | Proxy 伺服器/OMS 閘道的選擇性連接埠號碼 |

例如：`https://user01:password@proxy01.contoso.com:30443`

> [!NOTE]
> 若您在您的密碼中使用特殊字元 (例如 "\@")，您會收到 Proxy 連線錯誤訊息，因為系統無法正確剖析該值。  若要暫時解決此問題，請使用 [URLDecode](https://www.urldecoder.org/) 之類的工具將 URL 中的密碼編碼。  

## <a name="install-and-configure-agent"></a>安裝及設定代理程式 
視您的需求使用不同的方法，即可將您的內部部署電腦直接與 Log Analytics 連線。 下表說明每個方法，您可以判斷哪個方法最適合您的組織。

|來源 | 方法 | 說明|
|-------|-------------|-------------|
| Windows 電腦|- [手動安裝](log-analytics-agent-windows.md)<br>- [Azure 動化 DSC](log-analytics-agent-windows.md#install-the-agent-using-dsc-in-azure-automation)<br>- [搭配 Azure Stack 的資源管理員範本](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/MicrosoftMonitoringAgent-ext-win) |從命令列或使用自動化方法 (例如 Azure 自動化 DSC、[System Center Configuration Manager](https://docs.microsoft.com/sccm/apps/deploy-use/deploy-applications)) 來安裝 Microsoft Monitoring 代理程式，或者如果您已經在您的資料中心部署 Microsoft Azure Stack，請使用 Azure Resource Manager 範本。| 
|Linux 電腦| [手動安裝](log-analytics-quick-collect-linux-computer.md)|安裝適用於 Linux 的代理程式需要 GitHub 上裝載的包裝函式指令碼。 | 
| System Center Operations Manager|[將 Operations Manager 與 Log Analytics 整合](log-analytics-om-agents.md) | 設定 Operations Manager 與 Log Analytics 之間的整合，以將收集的資料從 Linux 與 Windows 電腦報告轉送到管理群組。|  

## <a name="next-steps"></a>後續步驟

* 檢閱[資料來源](log-analytics-data-sources.md)以了解可用於從您的 Windows 或 Linux 系統收集資料的資料來源。 

* 了解 [記錄搜尋](log-analytics-log-searches.md) ，其可分析從資料來源和方案所收集的資料。 

* 了解可將功能加入 Log Analytics 並會將資料收集到 OMS 儲存機制的 [方案](log-analytics-add-solutions.md) 。
