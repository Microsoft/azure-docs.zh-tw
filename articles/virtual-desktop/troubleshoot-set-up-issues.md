---
title: Windows 虛擬桌面環境主機集區建立-Azure
description: 如何在 Windows 虛擬桌面環境的設定期間進行疑難排解並解決租使用者和主機集區的問題。
author: Heidilohr
ms.topic: troubleshooting
ms.date: 09/14/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 0a5439a9d1fd43154379c1dc1a95a6e98b6e877b
ms.sourcegitcommit: fc23b4c625f0b26d14a5a6433e8b7b6fb42d868b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2021
ms.locfileid: "98539630"
---
# <a name="host-pool-creation"></a>主機集區建立

>[!IMPORTANT]
>此內容適用於具有 Azure Resource Manager Windows 虛擬桌面物件的 Windows 虛擬桌面。 如果您使用不含 Azure Resource Manager 物件的 Windows 虛擬桌面 (傳統版)，請參閱[這篇文章](./virtual-desktop-fall-2019/troubleshoot-set-up-issues-2019.md)。

本文涵蓋 Windows 虛擬桌面租使用者和相關工作階段主機集區基礎結構的初始設定期間的問題。

## <a name="provide-feedback"></a>提供意見反應

請瀏覽 [Windows 虛擬桌面技術社群](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop)，與產品小組和活躍的社群成員一起討論 Windows 虛擬桌面服務。

## <a name="acquiring-the-windows-10-enterprise-multi-session-image"></a>取得 Windows 10 企業版多會話映射

若要使用 Windows 10 企業版多會話映射，請移至 [Azure Marketplace]，選取 [**開始**  >  **Microsoft Windows 10** ] >，然後 [Windows 10 企業版多會話1809版](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsdesktop.windows-10?tab=PlansAndPrice)。

## <a name="issues-with-using-the-azure-portal-to-create-host-pools"></a>使用 Azure 入口網站建立主機集區的問題

### <a name="error-create-a-free-account-appears-when-accessing-the-service"></a>錯誤：存取服務時出現「建立免費帳戶」

> [!div class="mx-imgBorder"]
> ![顯示「建立免費帳戶」訊息之 Azure 入口網站的影像](media/create-new-account.png)

**原因**：您用來登入 Azure 的帳戶中沒有使用中的訂用帳戶，或該帳戶沒有可查看訂用帳戶的許可權。

**修正**：使用至少具有參與者層級存取權的帳戶，登入您將在其中部署工作階段主機虛擬機器 (vm) 的訂用帳戶。

### <a name="error-exceeding-quota-limit"></a>錯誤：「超過配額限制」

如果您的作業超過配額限制，您可以執行下列其中一項動作：

- 使用相同的參數建立新的主機集區，但 Vm 和 VM 核心數目較少。

- 在瀏覽器中開啟您在 [>statusmessage] 欄位中看到的連結，提交要求以增加您的 Azure 訂用帳戶在指定的 VM SKU 上的配額。

### <a name="error-cant-see-user-assignments-in-app-groups"></a>錯誤：無法在應用程式群組中看到使用者指派。

原因：當您將訂用帳戶從 1 Azure Active Directory (AD) 租使用者移到另一個時，通常會發生此錯誤。 如果您的舊指派仍系結至舊的 Azure AD 租使用者，則 Azure 入口網站將會失去其追蹤。

修正：您必須將使用者重新指派給應用程式群組。

## <a name="azure-resource-manager-template-errors"></a>Azure Resource Manager 範本錯誤

請遵循下列指示，針對 Azure Resource Manager 範本和 PowerShell DSC 的不成功部署進行疑難排解。

1. 使用 [Azure Resource Manager 的 View 部署作業](../azure-resource-manager/templates/deployment-history.md)，檢查部署中的錯誤。
2. 如果部署中沒有任何錯誤，請使用 View 活動記錄檔審核活動記錄中的錯誤， [以對資源進行審核動作](../azure-resource-manager/management/view-activity-logs.md)。
3. 一旦識別出錯誤，請使用錯誤訊息和資源來 [對常見的 Azure 部署錯誤進行疑難排解，Azure Resource Manager](../azure-resource-manager/templates/common-deployment-errors.md) 來解決問題。
4. 刪除在先前部署期間建立的任何資源，然後再次嘗試部署範本。

### <a name="error-your-deployment-failedhostnamejoindomain"></a>錯誤：您的部署失敗 ... \<hostname> /joindomain

> [!div class="mx-imgBorder"]
> ![您的部署失敗螢幕擷取畫面。](media/failure-joindomain.png)

原始錯誤的範例：

```Error
 {"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details.
 Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"Conflict","message":"{\r\n \"status\": \"Failed\",\r\n \"error\":
 {\r\n \"code\": \"ResourceDeploymentFailure\",\r\n \"message\": \"The resource operation completed with terminal provisioning state 'Failed'.
 \",\r\n \"details\": [\r\n {\r\n \"code\": \"VMExtensionProvisioningError\",\r\n \"message\": \"VM has reported a failure when processing
 extension 'joindomain'. Error message: \\\"Exception(s) occurred while joining Domain 'diamondsg.onmicrosoft.com'\\\".\"\r\n }\r\n ]\r\n }\r\n}"}]}
```

**原因1：** 提供用來將 Vm 加入網域的認證不正確。

**修正1：** 如果 Vm 未加入 [工作階段主機 VM](troubleshoot-vm-configuration.md)設定中的網域，請參閱「不正確的認證」錯誤。

**原因2：** 功能變數名稱無法解析。

**修正2：** 請參閱錯誤：[工作階段主機 VM](troubleshoot-vm-configuration.md)設定中的 [功能變數名稱無法解析](troubleshoot-vm-configuration.md#error-domain-name-doesnt-resolve)。

**原因3：** 您的虛擬網路 (VNET) DNS 設定會設定為 **預設值**。

若要修正此問題，請執行下列動作：

1. 開啟 Azure 入口網站並移至 [ **虛擬網路** ] 索引標籤。
2. 尋找您的 VNET，然後選取 [ **DNS 伺服器**]。
3. [DNS 伺服器] 功能表應該會出現在畫面的右側。 選取該功能表上的 [ **自訂**]。
4. 請確定 [自訂] 底下列出的 DNS 伺服器符合您的網域控制站或 Active Directory 網域。 如果您沒有看到您的 DNS 伺服器，您可以在 [ **新增 dns 伺服器** ] 欄位中輸入其值來新增。

### <a name="error-your-deployment-failedunauthorized"></a>錯誤：您的部署失敗...\未授權

```Error
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"Unauthorized","message":"{\r\n \"Code\": \"Unauthorized\",\r\n \"Message\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\",\r\n \"Target\": null,\r\n \"Details\": [\r\n {\r\n \"Message\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\"\r\n },\r\n {\r\n \"Code\": \"Unauthorized\"\r\n },\r\n {\r\n \"ErrorEntity\": {\r\n \"ExtendedCode\": \"52020\",\r\n \"MessageTemplate\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\",\r\n \"Parameters\": [\r\n \"default\"\r\n ],\r\n \"Code\": \"Unauthorized\",\r\n \"Message\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\"\r\n }\r\n }\r\n ],\r\n \"Innererror\": null\r\n}"}]}
```

**原因：** 您所使用的訂用帳戶是一種無法存取客戶嘗試部署之區域中所需功能的類型。 例如，MSDN、免費版或教育版訂閱可能會顯示此錯誤。

**修正：** 將您的訂用帳戶類型或區域變更為可以存取所需功能的帳戶。

### <a name="error-vmextensionprovisioningerror"></a>錯誤：VMExtensionProvisioningError

> [!div class="mx-imgBorder"]
> ![您的部署因終端機布建狀態失敗而失敗的螢幕擷取畫面。](media/failure-vmextensionprovisioning.png)

**原因1：** Windows 虛擬桌面環境發生暫時性錯誤。

**原因2：** 連接發生暫時性錯誤。

**修正：** 使用 PowerShell 登入，確認 Windows 虛擬桌面環境狀況良好。 [使用 PowerShell 在建立主機集](create-host-pools-powershell.md)區中手動完成 VM 註冊。

### <a name="error-the-admin-username-specified-isnt-allowed"></a>錯誤：不允許指定的管理員使用者名稱

> [!div class="mx-imgBorder"]
> ![您的部署失敗的螢幕擷取畫面，其中指定的系統管理員不允許。](media/failure-username.png)

原始錯誤的範例：

```Error
 { …{ "provisioningOperation":
 "Create", "provisioningState": "Failed", "timestamp": "2019-01-29T20:53:18.904917Z", "duration": "PT3.0574505S", "trackingId":
 "1f460af8-34dd-4c03-9359-9ab249a1a005", "statusCode": "BadRequest", "statusMessage": { "error": { "code": "InvalidParameter", "message":
 "The Admin Username specified is not allowed.", "target": "adminUsername" } … }
```

**原因：** 提供的密碼包含禁止的子字串 (admin、administrator、root) 。

**修正：** 更新使用者名稱或使用不同的使用者。

### <a name="error-vm-has-reported-a-failure-when-processing-extension"></a>錯誤：VM 回報在處理延伸模組時失敗

> [!div class="mx-imgBorder"]
> ![在您的部署中已完成終端布建狀態的資源作業螢幕擷取畫面失敗。](media/failure-processing.png)

原始錯誤的範例：

```Error
{ … "code": "ResourceDeploymentFailure", "message":
 "The resource operation completed with terminal provisioning state 'Failed'.", "details": [ { "code":
 "VMExtensionProvisioningError", "message": "VM has reported a failure when processing extension 'dscextension'.
 Error message: \"DSC Configuration 'SessionHost' completed with error(s). Following are the first few:
 PowerShell DSC resource MSFT_ScriptResource failed to execute Set-TargetResource functionality with error message:
 One or more errors occurred. The SendConfigurationApply function did not succeed.\"." } ] … }
```

**原因：** PowerShell DSC 擴充功能無法取得 VM 上的系統管理員存取權。

**修正：** 確認使用者名稱和密碼有虛擬機器的系統管理存取權，然後再次執行 Azure Resource Manager 範本。

### <a name="error-deploymentfailed--powershell-dsc-configuration-firstsessionhost-completed-with-errors"></a>錯誤： DeploymentFailed-PowerShell DSC 設定 ' FirstSessionHost ' 已完成，但發生錯誤 (s) 

> [!div class="mx-imgBorder"]
> ![部署失敗的螢幕擷取畫面，其中 PowerShell DSC 設定 ' FirstSessionHost ' 已完成，但發生錯誤 (s) 。](media/failure-dsc.png)

原始錯誤的範例：

```Error
{
    "code": "DeploymentFailed",
   "message": "At least one resource deployment operation failed. Please list
 deployment operations for details. 4 Please see https://aka.ms/arm-debug for usage details.",
 "details": [
         { "code": "Conflict",
         "message": "{\r\n \"status\": \"Failed\",\r\n \"error\": {\r\n \"code\":
         \"ResourceDeploymentFailure\",\r\n \"message\": \"The resource
         operation completed with terminal provisioning state 'Failed'.\",\r\n
         \"details\": [\r\n {\r\n \"code\":
        \"VMExtensionProvisioningError\",\r\n \"message\": \"VM has
              reported a failure when processing extension 'dscextension'.
              Error message: \\\"DSC Configuration 'FirstSessionHost'
              completed with error(s). Following are the first few:
              PowerShell DSC resource MSFT ScriptResource failed to
              execute Set-TargetResource functionality with error message:
              One or more errors occurred. The SendConfigurationApply
              function did not succeed.\\\".\"\r\n }\r\n ]\r\n }\r\n}"  }

```

**原因：** PowerShell DSC 擴充功能無法取得 VM 上的系統管理員存取權。

**修正：** 確認提供的使用者名稱和密碼有虛擬機器的系統管理存取權，然後再次執行 Azure Resource Manager 範本。

### <a name="error-deploymentfailed--invalidresourcereference"></a>錯誤： DeploymentFailed – InvalidResourceReference

原始錯誤的範例：

```Error
{"code":"DeploymentFailed","message":"At least one resource deployment operation
failed. Please list deployment operations for details. Please see https://aka.ms/arm-
debug for usage details.","details":[{"code":"Conflict","message":"{\r\n \"status\":
\"Failed\",\r\n \"error\": {\r\n \"code\": \"ResourceDeploymentFailure\",\r\n
\"message\": \"The resource operation completed with terminal provisioning state
'Failed'.\",\r\n \"details\": [\r\n {\r\n \"code\": \"DeploymentFailed\",\r\n
\"message\": \"At least one resource deployment operation failed. Please list
deployment operations for details. Please see https://aka.ms/arm-debug for usage
details.\",\r\n \"details\": [\r\n {\r\n \"code\": \"BadRequest\",\r\n \"message\":
\"{\\r\\n \\\"error\\\": {\\r\\n \\\"code\\\": \\\"InvalidResourceReference\\\",\\r\\n
\\\"message\\\": \\\"Resource /subscriptions/EXAMPLE/resourceGroups/ernani-wvd-
demo/providers/Microsoft.Network/virtualNetworks/wvd-vnet/subnets/default
referenced by resource /subscriptions/EXAMPLE/resourceGroups/ernani-wvd-
demo/providers/Microsoft.Network/networkInterfaces/erd. Please make sure that
the referenced resource exists, and that both resources are in the same
region.\\\",\\r\\n\\\"details\\\": []\\r\\n }\\r\\n}\"\r\n }\r\n ]\r\n }\r\n ]\r\n }\r\n}"}]}
```

**原因：** 資源組名的一部分用於範本所建立的特定資源。 由於名稱與現有資源相符，因此範本可能會從不同的群組中選取現有的資源。

**修正：** 執行 Azure Resource Manager 範本以部署工作階段主機 Vm 時，請讓訂用帳戶資源組名的前兩個字元是唯一的。

### <a name="error-deploymentfailed--invalidresourcereference"></a>錯誤： DeploymentFailed – InvalidResourceReference

原始錯誤的範例：

```Error
{"code":"DeploymentFailed","message":"At least one resource deployment operation
failed. Please list deployment operations for details. Please see https://aka.ms/arm-
debug for usage details.","details":[{"code":"Conflict","message":"{\r\n \"status\":
\"Failed\",\r\n \"error\": {\r\n \"code\": \"ResourceDeploymentFailure\",\r\n
\"message\": \"The resource operation completed with terminal provisioning state
'Failed'.\",\r\n \"details\": [\r\n {\r\n \"code\": \"DeploymentFailed\",\r\n
\"message\": \"At least one resource deployment operation failed. Please list
deployment operations for details. Please see https://aka.ms/arm-debug for usage
details.\",\r\n \"details\": [\r\n {\r\n \"code\": \"BadRequest\",\r\n \"message\":
\"{\\r\\n \\\"error\\\": {\\r\\n \\\"code\\\": \\\"InvalidResourceReference\\\",\\r\\n
\\\"message\\\": \\\"Resource /subscriptions/EXAMPLE/resourceGroups/ernani-wvd-
demo/providers/Microsoft.Network/virtualNetworks/wvd-vnet/subnets/default
referenced by resource /subscriptions/EXAMPLE/resourceGroups/DEMO/providers/Microsoft.Network/networkInterfaces
/EXAMPLE was not found. Please make sure that the referenced resource exists, and that both
resources are in the same region.\\\",\\r\\n \\\"details\\\": []\\r\\n }\\r\\n}\"\r\n
}\r\n ]\r\n }\r\n ]\r\n }\r\n\
```

**原因：** 此錯誤是因為使用 Azure Resource Manager 範本建立的 NIC 與 VNET 中的另一個 NIC 具有相同的名稱。

**修正：** 使用不同的主機前置詞。

### <a name="error-deploymentfailed--error-downloading"></a>錯誤： DeploymentFailed-下載錯誤

原始錯誤的範例：

```Error
\\\"The DSC Extension failed to execute: Error downloading
https://catalogartifact.azureedge.net/publicartifacts/rds.wvd-provision-host-pool-
2dec7a4d-006c-4cc0-965a-02bbe438d6ff-prod
/Artifacts/DSC/Configuration.zip after 29 attempts: The remote name could not be
resolved: 'catalogartifact.azureedge.net'.\\nMore information about the failure can
be found in the logs located under
'C:\\\\WindowsAzure\\\\Logs\\\\Plugins\\\\Microsoft.Powershell.DSC\\\\2.77.0.0' on
the VM.\\\"
```

**原因：** 此錯誤是因為靜態路由、防火牆規則或 NSG 封鎖了與 Azure Resource Manager 範本系結之 zip 檔案的下載。

**修正：** 移除封鎖靜態路由、防火牆規則或 NSG。 （選擇性）在文字編輯器中開啟 Azure Resource Manager 範本 json 檔案、取得 zip 檔案的連結，然後將資源下載至允許的位置。

### <a name="error-cant-delete-a-session-host-from-the-host-pool-after-deleting-the-vm"></a>錯誤：刪除 VM 之後，無法從主機集區刪除工作階段主機

**原因：** 您必須先刪除工作階段主機，才能刪除 VM。

**修正：** 讓工作階段主機進入清空模式、登出工作階段主機中的所有使用者，然後刪除主機。

## <a name="next-steps"></a>後續步驟

- 如需 Windows 虛擬桌面疑難排解和擴大追蹤的概觀，請參閱[疑難排解概觀、意見反應和支援](troubleshoot-set-up-overview.md)。
- 若要針對在 Windows 虛擬桌面中設定虛擬機器 (VM) 時的問題進行疑難排解，請參閱[工作階段主機虛擬機器設定](troubleshoot-vm-configuration.md)。
- 若要針對與 Windows 虛擬桌面代理程式或會話連線相關的問題進行疑難排解，請參閱針對 [常見的 Windows 虛擬桌面代理程式問題進行疑難排解](troubleshoot-agent.md)。
- 若要對 Windows 虛擬桌面用戶端連線問題進行疑難排解，請參閱 [Windows 虛擬桌面服務連接](troubleshoot-service-connection.md)。
- 若要疑難排解遠端桌面用戶端的問題，請參閱 [遠端桌面用戶端疑難排解](troubleshoot-client.md)
- 若要針對使用 PowerShell 搭配 Windows 虛擬桌面時的問題進行疑難排解，請參閱 [Windows 虛擬桌面 PowerShell](troubleshoot-powershell.md)。
- 若要深入瞭解此服務，請參閱 [Windows 虛擬桌面環境](environment-setup.md)。
- 若要進行疑難排解教學課程，請參閱[教學課程：針對 Resource Manager 範本部署進行疑難排解](../azure-resource-manager/templates/template-tutorial-troubleshoot.md)。
- 若要了解稽核動作，請參閱 [使用 Resource Manager 來稽核作業](../azure-resource-manager/management/view-activity-logs.md)。
- 若要了解部署期間可採取哪些動作來判斷錯誤，請參閱 [檢視部署作業](../azure-resource-manager/templates/deployment-history.md)。
