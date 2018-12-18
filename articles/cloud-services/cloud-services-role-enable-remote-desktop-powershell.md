---
title: 使用 PowerShell 啟用 Azure 雲端服務中角色的遠端桌面連線
description: 如何使用 PowerShell 設定的 Azure 雲端服務應用程式以允許遠端桌面連線
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
editor: ''
ms.assetid: bf2f70a4-20dc-4302-a91a-38cd7a2baa62
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/18/2017
ms.author: jeconnoc
ms.openlocfilehash: c5b70d40ed43cfc5d1c7a826c639d00d394733fb
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037989"
---
# <a name="enable-remote-desktop-connection-for-a-role-in-azure-cloud-services-using-powershell"></a>使用 PowerShell 啟用 Azure 雲端服務中角色的遠端桌面連線

> [!div class="op_single_selector"]
> * [Azure 入口網站](cloud-services-role-enable-remote-desktop-new-portal.md)
> * [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md)
> * [Visual Studio](cloud-services-role-enable-remote-desktop-visual-studio.md)

遠端桌面可讓您存取 Azure 內執行中角色的桌面。 您可以使用遠端桌面連線來疑難排解和診斷執行中應用程式的問題。

這篇文章描述如何使用 PowerShell 在雲端服務角色上啟用遠端桌面。 如需這篇文章所需要的必要條件，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 。 PowerShell 會利用遠端桌面延伸模組，因此在應用程式部署之後，您也可以啟用遠端桌面。

## <a name="configure-remote-desktop-from-powershell"></a>從 PowerShell 設定遠端桌面

[Set-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure/set-azureserviceremotedesktopextension?view=azuresmps-3.7.0) Cmdlet 可讓您在雲端服務部署的指定角色或所有角色上啟用遠端桌面。 此 Cmdlet 可讓您透過可接受 PSCredential 物件的 *Credential* 參數，指定遠端桌面使用者的使用者名稱和密碼。

如果您以互動方式使用 PowerShell，您可以呼叫 [Get-Credentials](https://technet.microsoft.com/library/hh849815.aspx) Cmdlet，輕鬆地設定 PSCredential 物件。

```
$remoteusercredentials = Get-Credential
```

此命令會顯示對話方塊，可讓您以安全的方式輸入遠端使用者的使用者名稱和密碼。

由於 PowerShell 在自動化案例中非常實用，您也可以透過不需要使用者互動的方式設定 **PSCredential** 物件。 您必須先設定安全的密碼。 首先，指定純文字密碼，然後使用 [ConvertTo-SecureString](https://technet.microsoft.com/library/hh849818.aspx)轉換成安全字串。 接下來，您需要使用 [ConvertFrom-SecureString](https://technet.microsoft.com/library/hh849814.aspx)，將這個安全字串轉換成加密的標準字串。 現在，您可以使用 [Set-Content](https://technet.microsoft.com/library/ee176959.aspx)，將此加密的標準字串儲存到檔案。

您也可以建立安全的密碼檔案，這樣就不需要每次都要輸入密碼。 此外，安全的密碼檔案也比純文字檔案安全。 使用下列 PowerShell 來建立安全的密碼檔案：

```
ConvertTo-SecureString -String "Password123" -AsPlainText -Force | ConvertFrom-SecureString | Set-Content "password.txt"
```

> [!IMPORTANT]
> 設定密碼時，請確定您符合 [複雜性需求](https://technet.microsoft.com/library/cc786468.aspx)。

若要從安全的密碼檔案建立認證物件，您必須讀取檔案內容，並使用 [ConvertTo-SecureString](https://technet.microsoft.com/library/hh849818.aspx)將它們轉換回安全字串。

[Set-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure/set-azureserviceremotedesktopextension?view=azuresmps-3.7.0) Cmdlet 也接受 *Expiration* 參數，它可指定使用者帳戶到期的 **日期時間** 。 例如，您可以設定帳戶在目前日期和時間的幾天後到期。

這個 PowerShell 範例示範如何在雲端服務上設定遠端桌面延伸模組：

```
$servicename = "cloudservice"
$username = "RemoteDesktopUser"
$securepassword = Get-Content -Path "password.txt" | ConvertTo-SecureString
$expiry = $(Get-Date).AddDays(1)
$credential = New-Object System.Management.Automation.PSCredential $username,$securepassword
Set-AzureServiceRemoteDesktopExtension -ServiceName $servicename -Credential $credential -Expiration $expiry
```
您可以選擇性地指定部署位置和您想要啟用遠端桌面的角色。 如果未指定這些參數，此 Cmdlet 會在「生產」  部署位置中的所有角色上啟用遠端桌面。

遠端桌面延伸模組與部署相關聯。 如果您為服務建立新的部署，您必須啟用該部署上的遠端桌面。 如果您想一律啟用遠端桌面，則您應該考慮將 PowerShell 指令碼整合到您的部署工作流程中。

## <a name="remote-desktop-into-a-role-instance"></a>從遠端桌面連接到角色執行個體

[Get-AzureRemoteDesktopFile](/powershell/module/servicemanagement/azure/get-azureremotedesktopfile?view=azuresmps-3.7.0) Cmdlet 用於從遠端桌面連接到雲端服務的特定角色執行個體。 您可以使用 *LocalPath* 參數將 RDP 檔案下載到本機。 或您可以使用 *Launch* 參數，直接啟動 [遠端桌面連線] 對話方塊來存取雲端服務角色執行個體。

```
Get-AzureRemoteDesktopFile -ServiceName $servicename -Name "WorkerRole1_IN_0" -Launch
```

## <a name="check-if-remote-desktop-extension-is-enabled-on-a-service"></a>檢查服務上是否已啟用遠端桌面延伸模組

[Get-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure/get-azureremotedesktopfile?view=azuresmps-3.7.0) Cmdlet 會顯示服務部署上的遠端桌面為啟用或停用。 此 Cmdlet 會傳回遠端桌面使用者的使用者名稱，以及已啟用遠端桌面延伸模組的角色。 根據預設，這會發生在部署位置上，而您可以選擇改為使用預備位置。

```
Get-AzureServiceRemoteDesktopExtension -ServiceName $servicename
```

## <a name="remove-remote-desktop-extension-from-a-service"></a>從服務移除遠端桌面延伸模組

如果您已在部署上啟用遠端桌面延伸模組，且需要更新遠端桌面設定，則必須先移除延伸模組。 然後使用新的設定將它再次啟用。 例如，如果您想為遠端使用者帳戶或是已到期的帳戶設定新密碼。 必須在已啟用遠端桌面延伸模組的現有部署上執行此動作。 對於新部署，您可以直接套用延伸模組。

若要從部署移除遠端桌面延伸模組，您可以使用 [Remove-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure/remove-azureserviceremotedesktopextension?view=azuresmps-3.7.0) Cmdlet。 您也可以選擇性地指定您想要移除遠端桌面延伸模組的部署位置和角色。

```
Remove-AzureServiceRemoteDesktopExtension -ServiceName $servicename -UninstallConfiguration
```

> [!NOTE]
> 若要完全移除延伸模組組態，您應該在呼叫 *remove* Cmdlet 時指定 **UninstallConfiguration** 參數。
>
> **UninstallConfiguration** 參數會將任何已套用到服務的延伸模組組態解除安裝。 每個延伸模組組態都與服務組態相關連。 呼叫 remove Cmdlet 時未指定 **UninstallConfiguration** 會取消<mark>部署</mark>與擴充組態的關聯，因此實際上會移除延伸模組。 不過，延伸模組組態仍然與服務相關聯。

## <a name="additional-resources"></a>其他資源

[如何設定雲端服務](cloud-services-how-to-configure-portal.md)
