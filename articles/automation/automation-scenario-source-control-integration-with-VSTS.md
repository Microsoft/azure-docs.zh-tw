---
title: 整合 Azure 自動化與 Azure DevOps Services 原始檔控制
description: 案例將逐步引導您設定 Azure 自動化帳戶與 Azure DevOps Services 原始檔控制的整合。
services: automation
author: eamonoreilly
ms.author: eamono
keywords: azure powershell, Azure DevOps Services, 原始檔控制, 自動化
ms.service: automation
ms.component: process-automation
ms.topic: conceptual
ms.date: 03/19/2017
ms.openlocfilehash: ef21060e98972dd7bc561f9a4311fa0c4bdec3b4
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227056"
---
# <a name="azure-automation-scenario---automation-source-control-integration-with-azure-devops"></a>Azure 自動化案例 - 自動化原始檔控制與 Azure DevOps 的整合

在此案例中，您有 Azure DevOps 專案可用來管理原始檔控制下的 Azure 自動化 Runbook 或 DSC 組態。
本文說明如何整合 Azure DevOps 與 Azure 自動化環境，以便每次簽入時都能持續整合。

## <a name="getting-the-scenario"></a>取得案例

此案例包含兩個 PowerShell Runbook，可直接從 Azure 入口網站的 [Powerbook 資源庫](automation-runbook-gallery.md)匯入，也可以從 [PowerShell 資源庫](https://www.powershellgallery.com)下載。

### <a name="runbooks"></a>Runbook

Runbook | 說明| 
--------|------------|
Sync-VSTS | 完成簽入時，從 Azure DevOps 原始檔控制匯入 Runbook 或組態。 如果以手動方式執行，會將所有 Runbook 或設定匯入並發佈到「自動化」帳戶。| 
Sync-VSTSGit | 完成簽入時，從 Git 原始檔控制下的 Azure DevOps 匯入 Runbook 或組態。 如果以手動方式執行，會將所有 Runbook 或設定匯入並發佈到「自動化」帳戶。|

### <a name="variables"></a>變數

變數 | 說明|
-----------|------------|
VSToken | 您所建立的安全變數資產，其中包含 Azure DevOps 個人存取權杖。 您可以了解如何在 [Azure DevOps 驗證頁面](/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate)建立 Azure DevOps 個人存取權杖。
## <a name="installing-and-configuring-this-scenario"></a>安裝和設定此案例

在 Azure DevOps 中建立[個人存取權杖](/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate)，用來將 Runbook 或設定同步處理到您的自動化帳戶。

![](media/automation-scenario-source-control-integration-with-VSTS/VSTSPersonalToken.png) 

在自動化帳戶中建立[安全變數](automation-variables.md)以儲存個人存取權杖，讓 Runbook 可向 Azure DevOps 驗證並將 Runbook 或組態同步處理到自動化帳戶。 您可以將此命名為 VSToken。

![](media/automation-scenario-source-control-integration-with-VSTS/VSTSTokenVariable.png)

請匯入會將 Runbook 或設定同步處理到自動化帳戶的 Runbook。 如果您使用 Azure DevOps 原始檔控制，可以使用 [Azure DevOps 範例 Runbook](https://www.powershellgallery.com/packages/Sync-VSTS)，如果搭配 Git 使用 Azure DevOps，則可以使用來自 [PowerShell 資源庫](https://www.powershellgallery.com)的 [Azure DevOps with Git 範例 Runbook](https://www.powershellgallery.com/packages/Sync-VSTSGit)，然後再部署至自動化帳戶。

![](media/automation-scenario-source-control-integration-with-VSTS/VSTSPowerShellGallery.png)

您現在可以[發佈](automation-creating-importing-runbook.md#publishing-a-runbook)此 Runbook，以便建立 Webhook. 
![](media/automation-scenario-source-control-integration-with-VSTS/VSTSPublishRunbook.png)

建立此 Sync-VSTS Runbook 的 [Webhook](automation-webhooks.md)，並填入參數，如下所示。 請務必複製 Webhook URL，因為您將需要它來作為 Azure DevOps 中的服務掛勾。 VSAccessTokenVariableName 是您稍早建立的安全變數名稱 (VSToken)，可保存個人存取權杖。 

與 Azure DevOps (Sync-VSTS.ps1) 整合需要下列參數：
### <a name="sync-vsts-parameters"></a>Sync-VSTS Parameters

參數 | 說明| 
--------|------------|
WebhookData | 這包含從 Azure DevOps 服務掛勾傳送的簽入資訊。 您應該將此參數保留為空白。| 
ResourceGroup | 這是自動化帳戶所在資源群組的名稱。|
AutomationAccountName | 與 Azure DevOps 同步處理的自動化帳戶名稱。|
VSFolder | Azure DevOps 中有 Runbook 與組態存在的資料夾名稱。|
VSAccount | Azure DevOps 組織的名稱。| 
VSAccessTokenVariableName | 保留 Azure DevOps 個人存取權杖的安全變數 (VSToken) 的名稱。| 


![](media/automation-scenario-source-control-integration-with-VSTS/VSTSWebhook.png)

如果您搭配 GIT 使用 Azure DevOps (Sync-VSTSGit.ps1)，將需要下列參數。

參數 | 說明|
--------|------------|
WebhookData | 這將包含從 Azure DevOps 服務勾點傳送的簽入資訊。 您應該將此參數保留為空白。| 
ResourceGroup | 這是自動化帳戶所在資源群組的名稱。|
AutomationAccountName | 與 Azure DevOps 同步處理的自動化帳戶名稱。|
VSAccount | Azure DevOps 組織的名稱。|
VSProject | Azure DevOps 中有 Runbook 與組態存在的專案名稱。|
GitRepo | Git 儲存機制的名稱。|
GitBranch | Azure DevOps Git 存放庫中分支的名稱。|
資料夾 | Azure DevOps Git 分支中資料夾的名稱。|
VSAccessTokenVariableName | 保留 Azure DevOps 個人存取權杖的安全變數 (VSToken) 的名稱。|

![](media/automation-scenario-source-control-integration-with-VSTS/VSTSGitWebhook.png)

針對會在程式碼簽入時觸發此 Webhook 的資料夾，在 Azure DevOps 中建立服務掛勾以供簽入使用。 選取 [Webhook] 作為建立新訂用帳戶時要與之整合的服務。 您可以在 [Azure DevOps 服務掛勾說明文件](https://www.visualstudio.com/en-us/docs/marketplace/integrate/service-hooks/get-started)深入了解服務掛勾。
![](media/automation-scenario-source-control-integration-with-VSTS/VSTSServiceHook.png)

您現在應該能夠執行將 Runbook 和設定簽入至 Azure DevOps 的所有簽入作業，並讓這些自動同步處理至您的自動化帳戶。

![](media/automation-scenario-source-control-integration-with-VSTS/VSTSSyncRunbookOutput.png)

如果您以手動方式執行此 Runbook 而不是由 Azure DevOps 觸發，則可以將 webhookdata 參數保留為空白，它將會從指定的 Azure DevOps 資料夾執行完整同步處理。

如果您想要將案例解除安裝，請從 Azure DevOps 移除其服務掛勾，刪除 Runbook 和 VSToken 變數。
