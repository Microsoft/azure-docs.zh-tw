---
title: 針對 Azure Cloud Shell 進行疑難排解 | Microsoft Docs
description: 針對 Azure Cloud Shell 進行疑難排解
services: azure
documentationcenter: ''
author: maertendMSFT
manager: hemantm
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/24/2018
ms.author: damaerte
ms.openlocfilehash: 0056364883d5a4a350e5b35374e1fc3abd0c7bea
ms.sourcegitcommit: 7b845d3b9a5a4487d5df89906cc5d5bbdb0507c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "42141676"
---
# <a name="troubleshooting--limitations-of-azure-cloud-shell"></a>Azure Cloud Shell 的疑難排解和限制

針對 Azure Cloud Shell 問題進行疑難排的已知解決方式包括：

## <a name="general-troubleshooting"></a>一般疑難排解

### <a name="early-timeouts-in-firefox"></a>在 FireFox 中提前逾時

- **詳細資料**：Cloud Shell 會利用開放式 Websocket 將輸入/輸出傳遞至瀏覽器。 FireFox 預設的原則會提早關閉 Websocket，而造成在 Cloud Shell 中提前逾時。
- **解決辦法**：開啟 FireFox，然後在 URL 方塊中瀏覽至 "about:config"。 搜尋 "network.websocket.timeout.ping.request"，然後將值從 0 變更為 10。

### <a name="disabling-cloud-shell-in-a-locked-down-network-environment"></a>在鎖定的網路環境中停用 Cloud Shell

- **詳細資料**：建議系統管理員阻止其使用者存取 Cloud Shell。 Cloud Shell 存取 `ux.console.azure.com` 網域時可能會遭到拒絕，進而停止任何對 Cloud Shell 進入點的存取，例如 portal.azure.com、shell.azure.com、Visual Studio Code 的 Azure 帳戶擴充及 docs.microsoft.com。
- **解決方法**：透過環境的網路設定來限制 `ux.console.azure.com` 的存取權。 Cloud Shell 圖示仍會顯示在 portal.azure.com 中，但無法成功連線至該服務。

### <a name="storage-dialog---error-403-requestdisallowedbypolicy"></a>儲存體對話方塊 - 錯誤：403 RequestDisallowedByPolicy

- **詳細資料**：因為您的系統管理員設置的 Azure 原則，透過 Cloud Shell 建立儲存體帳戶失敗。錯誤訊息包括：`The resource action 'Microsoft.Storage/storageAccounts/write' is disallowed by one or more policies.`
- **解決辦法**：連絡您的 Azure 系統管理員，請他移除或更新拒絕儲存體建立的 Azure 原則。

### <a name="storage-dialog---error-400-disallowedoperation"></a>儲存體對話方塊 - 錯誤：400 DisallowedOperation

- **詳細資料**：使用 Azure Active Directory 訂用帳戶時，您無法建立儲存體。
- **解決方式**：使用能夠建立儲存體資源的 Azure 訂用帳戶。 Azure AD 訂用帳戶無法建立 Azure 資源。

### <a name="terminal-output---error-failed-to-connect-terminal-websocket-cannot-be-established-press-enter-to-reconnect"></a>終端機輸出 - 錯誤：無法與終端機連線: 無法建立 websocket。 按 `Enter` 重新連線。
- **詳細資料**：Cloud Shell 必須能夠與 Cloud Shell 基礎結構建立 websocket 連線。
- **解決辦法**：確認您已將您的網路設定設定為啟用傳送 https 要求和 websocket 要求至 *.console.azure.com 中的網域。

### <a name="set-your-cloud-shell-connection-to-support-using-tls-12"></a>設定您的 Cloud Shell 連線以支援使用 TLS 1.2
 - **詳細資料**：若要定義 Cloud Shell 連線的 TLS 版本，您必須設定瀏覽器特有的設定。
 - **解決方式**：瀏覽至瀏覽器的安全性設定，然後選取 [使用 TLS 1.2] 旁的核取方塊。

## <a name="bash-troubleshooting"></a>Bash 疑難排解

### <a name="cannot-run-the-docker-daemon"></a>無法執行 Docker 精靈

- **詳細資料**：Cloud Shell 會運用容器來裝載您的殼層環境，因此系統會不允許執行精靈。
- **解決方式**：運用預設安裝的 [docker-machine](https://docs.docker.com/machine/overview/) 以從遠端 Docker 主機管理 Docker 容器。

## <a name="powershell-troubleshooting"></a>PowerShell 疑難排解

### <a name="gui-applications-are-not-supported"></a>不支援 GUI 應用程式

- **詳細資料**：使用者啟動 GUI 應用程式時，不會傳回提示。 例如，當使用者複製已啟用雙因素驗證的私人 GitHub 存放庫時，會顯示一個對話方塊，用以完成雙因素驗證。
- **解決方式**：關閉並重新開啟殼層。

### <a name="troubleshooting-remote-management-of-azure-vms"></a>針對 Azure VM 遠端管理進行疑難排解

- **詳細資料**：基於 WinRM 的預設 Windows 防火牆設定，使用者可能會看到下列錯誤：`Ensure the WinRM service is running. Remote Desktop into the VM for the first time and ensure it can be discovered.`
- **解決方式**：執行 `Enable-AzureRmVMPSRemoting`，在所有目標機器上啟用 PowerShell 遠端所有層面的功能。

### <a name="dir-does-not-update-the-result-in-azure-drive"></a>`dir` 不會更新 Azure 磁碟機中的結果

- **詳細資料**：根據預設，為了最佳化使用者體驗，`dir` 的結果會快取到 Azure 磁碟機中。
- **解決方式**：在您建立、更新或移除 Azure 資源後，請執行 `dir -force` 以更新 Azure 磁碟機中的結果。

## <a name="general-limitations"></a>一般限制

Azure Cloud Shell 具有下列已知限制：

### <a name="system-state-and-persistence"></a>系統狀態和持續性

提供 Cloud Shell 工作階段的電腦只是暫時性，在工作階段閒置 20 分鐘後就會回收。 Cloud Shell 需要掛接 Azure 檔案共用。 因此，您的訂用帳戶必須能夠設定儲存體資源，才可存取 Cloud Shell。 其他考量包括：

- 在掛接的儲存體中，只會保存 `clouddrive` 目錄內的修改。 在 Bash 中，也會保存 `$HOME` 目錄。
- 只能從您的[已指派區域](persisting-shell-storage.md#mount-a-new-clouddrive)內掛接 Azure 檔案共用。
  - 在 Bash 中，會執行 `env` 來尋找設定為 `ACC_LOCATION` 的區域。
- Azure 檔案只支援本機備援儲存體和異地備援儲存體帳戶。

### <a name="browser-support"></a>瀏覽器支援

Cloud Shell 支援下列最新版的瀏覽器：

- Microsoft Edge
- Microsoft Internet Explorer
- Google Chrome
- Mozilla Firefox
- Apple Safari
  - 不支援 Safari 私密瀏覽模式。

### <a name="copy-and-paste"></a>複製和貼上

[!include [copy-paste](../../includes/cloud-shell-copy-paste.md)]

### <a name="for-a-given-user-only-one-shell-can-be-active"></a>針對指定的使用者，只能有一個殼層作用中

使用者一次只能啟動一種類型的 Shell：**Bash** 或 **PowerShell**。 不過，您可能一次會執行多個 Bash 或 PowerShell 執行個體。 在 Bash 或 PowerShell 之間交換會讓 Cloud Shell 重新啟動，這樣會終止現有工作階段。

### <a name="usage-limits"></a>使用限制

Cloud Shell 主要用於互動式的使用案例。 因此，任何長時間執行而沒有互動的工作階段會在不發出警告的情況下結束。

### <a name="user-permissions"></a>使用者權限

權限設定為沒有 sudo 存取權的一般使用者。 不會保存 `$Home` 目錄之外的任何安裝。

## <a name="bash-limitations"></a>Bash 限制

### <a name="editing-bashrc"></a>編輯 .bashrc

編輯 .bashrc 時請小心，這麼做可能會導致 Cloud Shell 發生未預期的錯誤。

## <a name="powershell-limitations"></a>PowerShell 限制

### <a name="preview-version-of-azuread-module"></a>AzureAD 模組的預覽版

以 .NET Standard 為基礎的模組目前有預覽版 `AzureAD.Standard.Preview` 可供使用。 此模組提供與 `AzureAD` 相同的功能。

### <a name="sqlserver-module-functionality"></a>`SqlServer` 模組功能

包含在 Cloud Shell 中的 `SqlServer` 模組只提供發行前版本的 PowerShell Core 支援。 具體來說，`Invoke-SqlCmd` 目前還無法使用。

### <a name="default-file-location-when-created-from-azure-drive"></a>從 Azure 磁碟機建立時的預設檔案位置

使用 PowerShell Cmdlet 時，使用者無法在 Azure 磁碟機底下建立檔案。 當使用者使用其他工具 (例如 vim 或 nano) 來建立新檔案時，檔案預設會儲存至 `$HOME` 資料夾。

### <a name="commands-that-create-gui-pop-ups-are-not-supported"></a>不支援建立 GUI 快顯視窗的命令

如果使用者執行的命令會建立 Windows 對話方塊，例如 `Connect-AzureAD` 或 `Connect-AzureRmAccount`，其將會看到如下錯誤訊息：`Unable to load DLL 'IEFRAME.dll': The specified module could not be found. (Exception from HRESULT: 0x8007007E)`。

### <a name="tab-completion-can-throw-psreadline-exception"></a>Tab 鍵自動完成可能擲回 PSReadline 例外狀況

如果使用者的 PSReadline EditMode 設定為 Emacs，並且嘗試透過 Tab 鍵自動完成來顯示所有可能的值，但視窗尺寸過小而無法顯示所有可能值，PSReadline 就會擲回例外狀況。

### <a name="large-gap-after-displaying-progress-bar"></a>顯示進度列之後出現過大間距

如果命令或使用者動作顯示進度列，而使 Tab 鍵在 `Azure:` 磁碟機中自動完成，則可能會因為資料指標設定錯誤，導致先前進度列的所在位置出現間隔。

### <a name="random-characters-appear-inline"></a>隨機出現內嵌字元

資料指標位置序列程式碼 (如 `5;13R`)，可能會出現在使用者輸入的內容中。 可以手動移除這些字元。

## <a name="personal-data-in-cloud-shell"></a>Cloud Shell 中的個人資料

Azure Cloud Shell 會謹慎處理您的個人資料，Azure Cloud Shell 服務擷取和儲存的資料是用於提供您專屬體驗的預設值，例如您最近使用的殼層、慣用字型大小、慣用字型以及用以支援雲端磁碟機的檔案共用詳細資訊等。 若您想匯出或刪除這些資料，請參考下列指示。

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

### <a name="export"></a>匯出
若要匯出 Cloud Shell 為您儲存的使用者設定 (例如慣用殼層、字型大小和字型)，請執行下列命令。

1. [![](https://shell.azure.com/images/launchcloudshell.png "啟動 Azure Cloud Shell")](https://shell.azure.com)
2. 在 Bash 或 PowerShell 中執行下列命令：

Bash：

  ```
  token="Bearer $(curl http://localhost:50342/oauth2/token --data "resource=https://management.azure.com/" -H Metadata:true -s | jq -r ".access_token")"
  curl https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -H Authorization:"$token" -s | jq
  ```

PowerShell：

  ```powershell
  $token= ((Invoke-WebRequest -Uri "$env:MSI_ENDPOINT`?resource=https://management.core.windows.net/" -Headers @{Metadata='true'}).content |  ConvertFrom-Json).access_token
  ((Invoke-WebRequest -Uri https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -Headers @{Authorization = "Bearer $token"}).Content | ConvertFrom-Json).properties | Format-List
```

### <a name="delete"></a>刪除
若要刪除 Cloud Shell 為您儲存的使用者設定 (例如慣用殼層、字型大小和字型)，請執行下列命令。 下次您啟動 Cloud Shell 時，系統會要求您再次設置檔案共用。 

>[!Note]
> 如果您刪除您的使用者設定，實際的 Azure 檔案共用將不會刪除。 請移至 Azure 檔案以完成該動作。

1. [![](https://shell.azure.com/images/launchcloudshell.png "啟動 Azure Cloud Shell")](https://shell.azure.com)
2. 在 Bash 或 PowerShell 中執行下列命令：

Bash：

  ```
  token="Bearer $(curl http://localhost:50342/oauth2/token --data "resource=https://management.azure.com/" -H Metadata:true -s | jq -r ".access_token")"
  curl -X DELETE https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -H Authorization:"$token"
  ```

PowerShell：

  ```powershell
  $token= ((Invoke-WebRequest -Uri "$env:MSI_ENDPOINT`?resource=https://management.core.windows.net/" -Headers @{Metadata='true'}).content |  ConvertFrom-Json).access_token
  Invoke-WebRequest -Method Delete -Uri https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -Headers @{Authorization = "Bearer $token"}
  ```
