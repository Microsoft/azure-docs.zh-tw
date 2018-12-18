---
title: 啟用 Azure 雲端服務中角色的遠端桌面連線
description: 如何設定 Azure 雲端服務應用程式以允許遠端桌面連線
services: cloud-services
author: ghogen
manager: douge
ms.assetid: f5727ebe-9f57-4d7d-aff1-58761e8de8c1
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.topic: conceptual
ms.workload: azure-vs
ms.date: 03/06/2018
ms.author: ghogen
ms.openlocfilehash: 703e969fe31def329be60037cceba27864063b4e
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304045"
---
# <a name="enable-remote-desktop-connection-for-a-role-in-azure-cloud-services-using-visual-studio"></a>使用 Visual Studio 啟用 Azure 雲端服務中角色的遠端桌面連線

> [!div class="op_single_selector"]
> * [Azure 入口網站](cloud-services-role-enable-remote-desktop-new-portal.md)
> * [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md)
> * [Visual Studio](cloud-services-role-enable-remote-desktop-visual-studio.md)

遠端桌面可讓您存取 Azure 內執行中角色的桌面。 您可以使用遠端桌面連線來疑難排解和診斷執行中應用程式的問題。

Visual Studio 針對雲端服務所提供的發佈精靈包含一個選項，可在發佈流程期間使用您提供的認證來啟用遠端桌面。 使用 Visual Studio 2017 15.4 版和更早版本時，適合使用此選項。

不過，使用 Visual Studio 2017 15.5 版和更新版本時，建議您避免透過發佈精靈來啟用遠端桌面，除非您僅以單一開發人員的身分來工作。 只要是可能會有其他開發人員開啟專案的情況，您就應該改為透過 Azure 入口網站、PowerShell 或從持續部署工作流程中的發行管線來啟用遠端桌面。 之所以會如此建議，是因為 Visual Studio 與雲端服務 VM 上之遠端桌面的通訊方式有所變更，如本文中所述。

## <a name="configure-remote-desktop-through-visual-studio-2017-version-154-and-earlier"></a>透過 Visual Studio 2017 15.4 版和更早版本設定遠端桌面

當您使用 Visual Studio 2017 15.4 版和更早版本時，可以使用發佈精靈中的 [允許所有角色使用遠端桌面] 選項。 您還是可以搭配 Visual Studio 2017 15.5 版和更新版本來使用此精靈，但不可使用遠端桌面選項。

1. 在 Visual Studio 中，於 [方案總管] 中以滑鼠右鍵按一下您的雲端服務專案，然後選擇 [發佈]，以啟動發佈精靈。

2. 視需要登入您的 Azure 訂用帳戶，然後選取 [下一步]。

3. 在 [設定] 頁面上，選取 [允許所有角色使用遠端桌面]，然後選取 [設定...] 連結以開啟 [遠端桌面組態] 對話方塊。

4. 在對話方塊底部，選取 [更多選項]。 此命令會顯示下拉式清單，您可在其中建立或選擇憑證，以便能夠在透過遠端桌面連線時加密認證資訊。

   > [!Note]
   > 遠端桌面連線所需的憑證不同於用於其他 Azure 作業的憑證。 遠端存取憑證必須具有私密金鑰。

5. 從清單中選取憑證，或選擇 [&lt;建立...&gt;]。 如果您要建立新的憑證，請在出現提示時為新的憑證提供易記名稱，然後選取 [確定]。 新的憑證會出現在下拉式清單方塊中。

6. 提供使用者名稱和密碼。 您無法使用現有的帳戶。 請勿使用 "Administrator" 作為新帳戶的使用者名稱。

7. 選擇帳戶的到期日期，在此日期之後，遠端桌面連線會遭到封鎖。

8. 提供所有必要的資訊之後，選取 [確定]。 Visual Studio 會將遠端桌面設定新增至您專案的 `.cscfg` 和 `.csdef` 檔案中，包括使用所選憑證進行加密的密碼。

9. 使用 [下一步] 按鈕完成所有剩餘的步驟，然後在準備好發佈雲端服務時選取 [發佈]。 如果還沒準備好發佈，則選取 [取消]，並在系統提示時回答 [是] 來儲存變更。 您稍後可以使用這些設定來發佈雲端服務。

## <a name="configure-remote-desktop-when-using-visual-studio-2017-version-155-and-later"></a>在使用 Visual Studio 2017 15.5 版和更新版本時設定遠端桌面

在 Visual Studio 2017 15.5 版和更新版本中，您仍可搭配使用發佈精靈與雲端服務專案。 如果您僅以單一開發人員的身分來工作，則也可以使用 [允許所有角色使用遠端桌面] 選項。

如果您是以小組成員的身分來工作，則應該改為使用 [Azure 入口網站](cloud-services-role-enable-remote-desktop-new-portal.md)或 [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md) 來啟用 Azure 雲端服務上的遠端桌面。

之所以會如此建議，是因為 Visual Studio 2017 15.5 版和更新版本與雲端服務 VM 的通訊方式有所變更。 透過發佈精靈來啟用遠端桌面時，較早版本的 Visual Studio 會透過所謂的「RDP 外掛程式」來與 VM 通訊。 Visual Studio 2017 15.5 版和更新版本則是改用更安全且更有彈性的「RDP 擴充功能」來進行通訊。 這項變更也符合一項事實，那就是用來啟用遠端桌面的 Azure 入口網站和 PowerShell 方法也會使用 RDP 擴充功能。

Visual Studio 在與 RDP 擴充功能通訊時，會透過 SSL 傳輸純文字密碼。 不過，專案的組態檔只會儲存加密的密碼，並只能使用原本用來將其加密的本機憑證來解密為純文字。

如果您每次都是從相同的開發電腦部署雲端服務專案，則可以取得該本機憑證。 在此情況下，您仍可以使用發佈精靈中的 [允許所有角色使用遠端桌面] 選項。

不過，如果您或其他開發人員要從不同的電腦部署雲端服務專案，則這些其他電腦就沒有解密密碼所需的憑證。 因此，您會看到下列錯誤訊息：

```output
Applying remote desktop protocol (RDP) extension.
Certificate with thumbprint [thumbprint] doesn't exist.
```

您每次部署雲端服務時都可以變更密碼，但這個動作對於需要使用遠端桌面的每一位使用者來說都很不方便。

如果您要與小組共用專案，則最好先將發佈精靈中的選項清除，然後改為直接透過 [Azure 入口網站](cloud-services-role-enable-remote-desktop-new-portal.md)或使用 [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md) 來啟用遠端桌面。

### <a name="deploying-from-a-build-server-with-visual-studio-2017-version-155-and-later"></a>使用 Visual Studio 2017 15.5 版和更新版本從組建伺服器進行部署

您可以從在組建代理程式中安裝了 Visual Studio 2017 15.5 版或更新版本的組建伺服器來部署雲端服務專案 (例如，透過 Azure DevOps Services)。 透過這種安排，部署會在可使用加密憑證的同一台電腦中進行。

若要使用 Azure DevOps Services 中的 RDP 擴充功能，請在您的組建管線中包含下列詳細資料：

1. 在您的 MSBuild 引數中包含 `/p:ForceRDPExtensionOverPlugin=true`，以確定部署可與 RDP 擴充功能 (而不是 RDP 外掛程式) 搭配運作。 例如︰

    ```
    msbuild AzureCloudService5.ccproj /t:Publish /p:TargetProfile=Cloud /p:DebugType=None
        /p:SkipInvalidConfigurations=true /p:ForceRDPExtensionOverPlugin=true
    ```

1. 在建置步驟完成後，請新增 **Azure 雲端服務部署**步驟，並設定其屬性。

1. 在部署步驟完成後，請新增 **Azure Powershell** 步驟，將其 [顯示名稱] 屬性設定為 [Azure 部署：啟用 RDP 擴充功能] (或另一個適合的名稱)，並選取適當的 Azure 訂用帳戶。

1. 將 [指令碼類型] 設定為 [內嵌]，並將下列程式碼貼到 [內嵌指令碼] 欄位中  (您也可以在您的專案中使用此指令碼建立 `.ps1` 檔案，將 [指令碼類型] 設定為 [指令碼檔案路徑]，並將 [指令碼路徑] 設定為指向該檔案)。

    ```ps
    Param(
        [Parameter(Mandatory=$True)]
        [string]$username,

        [Parameter(Mandatory=$True)]
        [string]$password,

        [Parameter(Mandatory=$True)]
        [string]$serviceName,

        [Datetime]$expiry = ($(Get-Date).AddYears(1))
    )

    Write-Host "Service Name: $serviceName"
    Write-Host "User Name: $username"
    Write-Host "Expiry: $expiry"

    $securepassword = ConvertTo-SecureString -String $password -AsPlainText -Force
    $credential = New-Object System.Management.Automation.PSCredential $username,$securepassword

    # Try to remote existing RDP Extensions
    try
    {
        $existingRDPExtension = Get-AzureServiceRemoteDesktopExtension -ServiceName $servicename
        if ($existingRDPExtension -ne $null)
        {
            Remove-AzureServiceRemoteDesktopExtension -ServiceName $servicename -UninstallConfiguration
        }
    }
    catch
    {
    }

    Set-AzureServiceRemoteDesktopExtension -ServiceName $servicename -Credential $credential -Expiration $expiry -Verbose
    ```

## <a name="connect-to-an-azure-role-by-using-remote-desktop"></a>使用遠端桌面連接到 Azure 角色

您在 Azure 上發佈雲端服務並啟用遠端桌面之後，就可以使用 Visual Studio 伺服器總管來登入雲端服務 VM：

1. 在 [伺服器總管] 中展開 [Azure]  節點，然後展開某個雲端服務的節點及其中一個角色，以顯示執行個體的清單。

2. 以滑鼠右鍵按一下執行個體節點，然後選取 [使用遠端桌面連線]。

3. 輸入您先前建立的使用者名稱和密碼。 您現在已登入遠端工作階段。

## <a name="additional-resources"></a>其他資源

[如何設定雲端服務](cloud-services-how-to-configure-portal.md)
