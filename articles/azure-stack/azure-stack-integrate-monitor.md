---
title: 整合外部監視解決方案與 Azure Stack | Microsoft Docs
description: 了解如何將 Azure Stack 與您的資料中心的外部監視解決方案整合。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PowerShell
ms.topic: article
ms.date: 10/15/2018
ms.author: jeffgilb
ms.reviewer: thoroet
ms.openlocfilehash: 66cd20eaa401261bcb18bedbbc16f5bcf40ee192
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49342978"
---
# <a name="integrate-external-monitoring-solution-with-azure-stack"></a>整合外部監視解決方案與 Azure Stack

針對外部監視 Azure Stack 基礎結構，您需要監視 Azure Stack 軟體、實體電腦和實體網路交換器。 每個區域都會提供方法來擷取健康情況和警示資訊：

- Azure Stack 軟體提供以 REST 為基礎的 API 來擷取健康情況和警示。 儲存空間直接存取、存放裝置健康狀態和警示等軟體定義技術，都是軟體監視的一部分。
- 實體電腦可讓健康情況和警示資訊透過基礎板管理控制器 (BMC) 提供。
- 實體網路裝置可讓健康情況和警示資訊透過 SNMP 通訊協定提供。

每個 Azure Stack 解決方案隨附硬體生命週期主機。 此主機會針對實體伺服器及網路裝置執行原始設備製造商 (OEM) 硬體廠商的監視軟體。 如有需要，您可以略過這些監視解決方案，並直接與資料中心內現有的監視解決方案整合。

> [!IMPORTANT]
> 您使用的外部監視解決方案必須是無代理程式。 您無法在 Azure Stack 元件內部安裝協力廠商代理程式。

下圖顯示整合 Azure Stack 整合系統、硬體生命週期主機、外部監視解決方案和外部票證/資料收集系統之間的流量。

![顯示 Azure Stack、監視以及票證解決方案之間流量的圖表。](media/azure-stack-integrate-monitor/MonitoringIntegration.png)  

本文說明如何將 Azure Stack 與外部監視解決方案 (例如 System Center Operations Manager 和 Nagios) 整合。 也包含如何使用 PowerShell 或 REST API 呼叫，以程式設計方式處理警示的資訊。

## <a name="integrate-with-operations-manager"></a>與 Operations Manager 整合

您可以將 Operations Manager 用於 Azure Stack 的外部監視。 Microsoft Azure Stack 適用的 System Center 管理組件可讓您使用單一 Operations Manager 執行個體來監視多個 Azure Stack 部署。 管理組件會使用健康情況資源提供者和更新資源提供者 REST API 來與 Azure Stack 進行通訊。 如果您打算略過硬體生命週期主機上執行的 OEM 監視軟體，您可以安裝廠商管理組件來監視實體伺服器。 您也可以使用 Operations Manager 網路裝置探索來監視網路交換器。

Azure Stack 的管理組件提供下列功能：

- 您可以管理多個 Azure Stack 部署
- 支援 Azure Active Directory (Azure AD) 和 Active Directory 同盟服務 (AD FS)
- 您可以擷取和關閉警示
- 提供健康情況和容量儀表板
- 包含當修補和更新 (P&U) 進行時的自動維護模式偵測
- 包含針對部署和區域的強制更新工作
- 您可以將自訂資訊新增至區域
- 支援通知與報告

您可以下載 Microsoft Azure Stack 適用的 System Center 管理組件和相關聯的[使用者指南](https://www.microsoft.com/en-us/download/details.aspx?id=55184)，或直接從 Operations Manager 使用。

如需票證解決方案，您可以將 Operations Manager 與 System Center Service Manager 整合。 整合的產品連接器會啟用雙向通訊，可讓您在解決 Service Manager 中的服務要求之後關閉 Azure Stack 和 Operations Manager 中的警示。

下圖顯示 Azure Stack 與現有的 System Center 部署的整合。 您可以進一步使用 System Center Orchestrator 或 Service Management Automation (SMA) 來將 Service Manager 自動化，以在 Azure Stack 中執行作業。

![顯示與 OM、Service Manager 和 SMA 整合的圖表。](media/azure-stack-integrate-monitor/SystemCenterIntegration.png)

## <a name="integrate-with-nagios"></a>與 Nagios 整合

Nagios 監視外掛程式隨著協力廠商 Cloudbase 解決方案開發，其提供是依據寬鬆免費軟體授權 - MIT (麻省理工學院)。

外掛程式是使用 Python 撰寫，並利用健康情況資源提供者 REST API。 可提供在 Azure Stack 中擷取並關閉警示的基本功能。 類似 System Center 管理組件，它可讓您新增多個 Azure Stack 部署，以及傳送通知。

此外掛程式可與 Nagios Enterprise 和 Nagios Core 搭配使用。 您可以從 [這裡](https://exchange.nagios.org/directory/Plugins/Cloud/Monitoring-AzureStack-Alerts/details)下載。 下載網站也包含安裝和設定詳細資料。

### <a name="plugin-parameters"></a>外掛程式參數

使用下列參數來設定外掛程式檔案 "Azurestack_plugin.py"：

| 參數 | 說明 | 範例 |
|---------|---------|---------|
| *arm_endpoint* | Azure Resource Manager (系統管理員) 端點 |https://adminmanagement.local.azurestack.external |
| *api_endpoint* | Azure Resource Manager (系統管理員) 端點  | https://adminmanagement.local.azurestack.external |
| *Tenant_id* | 管理訂用帳戶識別碼 | 透過系統管理員入口網站或 PowerShell 擷取 |
| *User_name* | 操作員訂用帳戶使用者名稱 | operator@myazuredirectory.onmicrosoft.com |
| *User_password* | 操作員訂用帳戶密碼 | mypassword |
| *Client_id* | 用戶端 | 0a7bdc5c-7b57-40be-9939-d4c5fc7cd417* |
| *region* |  Azure Stack 區域名稱 | local |
|  |  |

*提供的 PowerShell GUID 是通用的。 您可以對每個部署使用它。

## <a name="use-powershell-to-monitor-health-and-alerts"></a>使用 PowerShell 來監視健康情況和警示

如果您不是使用 Operations Manager、Nagios 或以 Nagios 基礎的解決方案，可以使用 PowerShell 來啟用廣泛的監視解決方案，以與 Azure Stack 整合。

1. 若要使用 PowerShell，請確定您已針對 Azure Stack 操作員環境[安裝和設定 PowerShell ](azure-stack-powershell-configure-quickstart.md)。 在可以連線資源管理員 (系統管理員) 端點的本機電腦上安裝 PowerShell (https://adminmanagement.[region].[External_FQDN])。

2. 以 Azure Stack 操作員身分執行下列命令以連線到 Azure Stack 環境：

   ```PowerShell  
    Add-AzureRMEnvironment -Name "AzureStackAdmin" -ArmEndpoint https://adminmanagement.[Region].[External_FQDN]

   Add-AzureRmAccount -EnvironmentName "AzureStackAdmin"
   ```

3. 使用如下列範例的命令來處理警示：
   ```PowerShell
    #Retrieve all alerts
    Get-AzsAlert

    #Filter for active alerts
    $Active=Get-AzsAlert | Where {$_.State -eq "active"}
    $Active

    #Close alert
    Close-AzsAlert -AlertID "ID"

    #Retrieve resource provider health
    Get-AzsRPHealth

    #Retrieve infrastructure role instance health
    $FRPID=Get-AzsRPHealth|Where-Object {$_.DisplayName -eq "Capacity"}
    Get-AzsRegistrationHealth -ServiceRegistrationId $FRPID.RegistrationId

    ```

## <a name="learn-more"></a>深入了解

如需內建的健康情況監視的相關資訊，請參閱[在 Azure Stack 中監視健康情況和警示](azure-stack-monitor-health.md)。

## <a name="next-steps"></a>後續步驟

[安全性整合](azure-stack-integrate-security.md)
