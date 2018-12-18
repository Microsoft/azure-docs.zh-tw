---
title: 使用 Windows PowerShell 指令碼來發佈至開發和測試環境 | Microsoft Docs
description: 了解如何從 Visual Studio 使用 Windows PowerShell 指令碼來發佈至開發和測試環境。
services: visual-studio-online
author: ghogen
manager: douge
assetId: 5fff1301-5469-4d97-be88-c85c30f837c1
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 11/11/2016
ms.author: ghogen
ms.openlocfilehash: 99d723eee6bd5b60289af5490e4b1cd6a855cabb
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319144"
---
# <a name="using-windows-powershell-scripts-to-publish-to-dev-and-test-environments"></a>使用 Windows PowerShell 指令碼來發行至開發和測試環境

當您在 Visual Studio 中建立 Web 應用程式時，您可以產生 Windows PowerShell 指令碼，以供稍後用來將網站自動發佈至 Azure 做為 Azure App Service 或虛擬機器中的 Web 應用程式。 您可以在 Visual Studio 編輯器中編輯和擴充 Windows PowerShell 指令碼以符合需求，或將指令碼整合到現有組建、測試和發佈指令碼。

使用這些指令碼，您可以佈建網站的自訂版本 (也稱為開發和測試環境)，以做臨時使用。 例如，您可以在 Azure 虛擬機器上或網站的預備位置上設定網站的特定版本，以執行測試套件、重現錯誤、測試錯誤修正、試驗提議的變更，或設定用來進行示範或展示的自訂環境。 在建立用來發佈專案的指令碼後，您可以視需要重新執行指令碼來重建相同環境，或對 Web 應用程式的自有組建執行指令碼以建立用於測試的自訂環境。

## <a name="prerequisites"></a>必要條件

* Azure SDK 2.3 或更新版本。 請參閱 [Visual Studio 下載](http://go.microsoft.com/fwlink/?LinkID=624384)。 (要產生 Web 專案的指令碼並不需要用到 Azure SDK。 這項功能是供 Web 專案使用，而非供雲端服務中的 Web 角色使用。)
* Azure PowerShell 0.7.4 或更新版本。 請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。
* [Windows PowerShell 3.0](http://go.microsoft.com/?linkid=9811175) 或更新版本。

## <a name="additional-tools"></a>其他工具

我們有提供其他工具和資源，以供您在 Visual Studio 中使用 PowerShell 進行 Azure 開發。 請參閱 [PowerShell Tools for Visual Studio](http://go.microsoft.com/fwlink/?LinkId=404012)。

## <a name="generating-the-publish-scripts"></a>產生發佈指令碼

您可以遵循 [這些指示](virtual-machines/windows/classic/web-app-visual-studio.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)，在建立新專案時產生裝載網站之虛擬機器的發佈指令碼。 您也可以 [在 Azure App Service 中產生 Web 應用程式的發佈指令碼](app-service/scripts/app-service-powershell-deploy-github.md)。

## <a name="scripts-that-visual-studio-generates"></a>Visual Studio 所產生的指令碼

Visual Studio 會產生名為 **PublishScripts** 的方案層級資料夾，並內含兩個 Windows PowerShell 檔案，一個是用於虛擬機器或網站的發佈指令碼，一個是含有可在指令碼中使用之函式的模組。 Visual Studio 也會產生 JSON 格式的檔案，以指定您要部署之專案的詳細資料。

### <a name="windows-powershell-publish-script"></a>Windows PowerShell 發佈指令碼

發佈指令碼含有部署至網站或虛擬機器的特定發佈步驟。 Visual Studio 提供語法著色功能，可用來進行 Windows PowerShell 開發。 有提供函式說明，而且您可以自由編輯指令碼中的函式以符合您不斷變化的需求。

### <a name="windows-powershell-module"></a>Windows PowerShell 模組

Visual Studio 所產生的 Windows PowerShell 模組包含發佈指令碼所使用的函式。 不可修改這些 Azure PowerShell 函式。 請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

### <a name="json-configuration-file"></a>JSON 組態檔

JSON 檔案建立在 [組態]  資料夾，其包含的組態資料可確切指定要部署至 Azure 的資源。 Visual Studio 所產生之檔案的名稱是 project-name-WAWS-dev.json (如果您建立網站) 或 project name-VM-dev.json (如果您建立虛擬機器)。 以下是建立網站時所產生之 JSON 組態檔的範例。 其中大多數的值都簡單易懂。 網站名稱是由 Azure 產生，因此可能不符合您的專案名稱。

```json
{
    "environmentSettings": {
        "webSite": {
            "name": "WebApplication26632",
            "location": "West US"
        },
        "databases": [{
            "connectionStringName": "DefaultConnection",
            "databaseName": "WebApplication26632_db",
            "serverName": "YourDatabaseServerName",
            "user": "sqluser2",
            "password": "",
            "edition": "",
            "size": "",
            "collation": "",
            "location": "West US"
        }]
    }
}
```

在建立虛擬機器時，JSON 組態檔看起來類似下面範例。 雲端服務會建立為虛擬機器的容器。 虛擬機器包含用來透過 HTTP 和 HTTPS 進行 Web 存取的常用端點以及供 Web Deploy 使用的端點，後者可讓您從本機電腦、遠端桌面和 Windows PowerShell 發佈至網站。

```json
{
    "environmentSettings": {
        "cloudService": {
            "name": "myusernamevm1",
            "affinityGroup": "",
            "location": "West US",
            "virtualNetwork": "",
            "subnet": "",
            "availabilitySet": "",
            "virtualMachine": {
                "name": "myusernamevm1",
                "vhdImage": "a699494373c04fc0bc8f2bb1389d6106__Win2K8R2SP1-Datacenter-201403.01-en.us-127GB.vhd",
                "size": "Small",
                "user": "vmuser1",
                "password": "",
                "enableWebDeployExtension": true,
                "endpoints": [{
                        "name": "Http",
                        "protocol": "TCP",
                        "publicPort": "80",
                        "privatePort": "80"
                    },
                    {
                        "name": "Https",
                        "protocol": "TCP",
                        "publicPort": "443",
                        "privatePort": "443"
                    },
                    {
                        "name": "WebDeploy",
                        "protocol": "TCP",
                        "publicPort": "8172",
                        "privatePort": "8172"
                    },
                    {
                        "name": "Remote Desktop",
                        "protocol": "TCP",
                        "publicPort": "3389",
                        "privatePort": "3389"
                    },
                    {
                        "name": "Powershell",
                        "protocol": "TCP",
                        "publicPort": "5986",
                        "privatePort": "5986"
                    }
                ]
            }
        },
        "databases": [{
            "connectionStringName": "",
            "databaseName": "",
            "serverName": "",
            "user": "",
            "password": ""
        }],
        "webDeployParameters": {
            "iisWebApplicationName": "Default Web Site"
        }
    }
}
```

您可以編輯 JSON 組態以變更執行發佈指令碼時會進行的作業。 `cloudService` 和 `virtualMachine` 是必要區段，但如果您不需要 `databases` 區段，則可以將它刪除。 Visual Studio 所產生的預設組態檔中的空白屬性是選用屬性；預設組態檔中具有值的屬性則是必要屬性。

如果您有內含多個部署環境 (稱為位置) 的網站而不是在 Azure 中的單一生產網站，您可以在 JSON 組態檔的網站名稱中加入位置名稱。 例如，如果您擁有名為 **mysite** 的網站和名為 **test** 的位置，則 URI 是 `mysite-test.cloudapp.net`，但要在組態檔中使用的正確名稱是 mysite(test)。 只有在訂用帳戶中已存在網站和位置時才能這麼做。 如果不存在，請執行指令碼 (不指定位置) 來建立網站，然後在 [Azure 入口網站](https://portal.azure.com/)中建立位置，之後再以修改過的網站名稱執行指令碼。 如需 Web 應用程式部署位置的詳細資訊，請參閱 [針對 Azure App Service 中的 Web 應用程式設定預備環境](app-service/web-sites-staged-publishing.md)。

## <a name="how-to-run-the-publish-scripts"></a>如何執行發佈指令碼

如果您之前未曾執行過 Windows PowerShell 指令碼，您必須先設定執行原則以啟用要執行的指令碼。 如果 Windows PowerShell 指令碼容易受到涉及執行指令碼的惡意程式碼或病毒的威脅，這項安全性功能可防止使用者執行 Windows PowerShell 指令碼。

### <a name="run-the-script"></a>執行指令碼

1. 建立專案的 Web Deploy 封裝。 Web Deploy 封裝是經過壓縮的封存檔 (.zip 檔案)，內含您想要複製到網站或虛擬機器的檔案。 您可以在 Visual Studio 中為任何 Web 應用程式建立 Web Deploy 封裝。

![建立 Web Deploy 封裝](./media/vs-azure-tools-publishing-using-powershell-scripts/IC767885.png)

如需詳細資訊，請參閱 [如何：在 Visual Studio 中建立 Web 部署封裝](https://msdn.microsoft.com/library/dd465323.aspx)。 您也可以自動建立 Web Deploy 封裝，如 [自訂和擴充發佈指令碼 (#customizing-and-extending-publish-scripts)]所述。

1. 在 [方案總管] 中，開啟指令碼的內容功能表，然後選擇 [以 PowerShell ISE 開啟]。
1. 如果第一次在此電腦上執行 Windows PowerShell 指令碼，請以系統管理員權限開啟命令提示字元視窗，並輸入下列命令：

    ```powershell
    Set-ExecutionPolicy RemoteSigned
    ```

1. 使用下列命令登入 Azure。

    ```powershell
    Add-AzureAccount
    ```

出現提示時，提供您的使用者名稱和密碼。

請注意，當您自動執行指令碼時，這個提供 Azure 認證的方法將沒有作用。 您應該改用 `.publishsettings` 檔案來提供認證。 請使用 **Get-AzurePublishSettingsFile** 命令從 Azure 下載檔案，之後使用 **Import-AzurePublishSettingsFile** 匯入檔案 (此程序只需執行一次)。 如需詳細指示，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

1. (選用) 如果您想要建立虛擬機器、資料庫和網站等 Azure 資源，而不要發佈 Web 應用程式，請使用 **Publish-WebApplication.ps1** 命令搭配設定為 JSON 組態檔的 **-Configuration** 引數。 此命令列使用 JSON 組態檔來決定要建立哪些資源。 因為它使用其他命令列引數的預設設定，所以會建立資源，但不會發佈 Web 應用程式。 –Verbose 選項可讓您進一步了解會發生什麼事。

    ```powershell
    Publish-WebApplication.ps1 -Verbose –Configuration C:\Path\WebProject-WAWS-dev.json
    ```

1. 如下列其中一個範例所示使用 **Publish-WebApplication.ps1** 命令叫用指令碼並發佈 Web 應用程式。 如果您需要覆寫任何其他引數的預設設定，例如訂用帳戶名稱、發佈封裝名稱、虛擬機器認證或資料庫伺服器認證，可以指定這些參數。 使用 **–Verbose** 選項可檢視發佈程序進度的詳細資訊。

    ```powershell
    Publish-WebApplication.ps1 –Configuration C:\Path\WebProject-WAWS-dev-json `
    –SubscriptionName Contoso `
    -WebDeployPackage C:\Documents\Azure\ADWebApp.zip `
    -DatabaseServerPassword @{Name="dbServerName";Password="adminPassword"} `
    -Verbose
    ```

如果您要建立虛擬機器，其命令如下所示。 此範例也顯示如何指定多個資料庫的認證。 針對這些指令碼所建立的虛擬機器，其 SSL 憑證不是來自受信任的根授權單位。 因此，您需要使用 **–AllowUntrusted** 選項。

```powershell
Publish-WebApplication.ps1 `
-Configuration C:\Path\ADVM-VM-test.json `
-SubscriptionName Contoso `
-WebDeployPackage C:\Path\ADVM.zip `
-AllowUntrusted `
-VMPassword @{name = "vmUserName"; password = "YourPasswordHere"} `
-DatabaseServerPassword @{Name="server1";Password="adminPassword1"}, @{Name="server2";Password="adminPassword2"} `
-Verbose
```

指令碼可以建立資料庫，但不會建立資料庫伺服器。 如果您想要建立資料庫伺服器，您可以使用 Azure 模組中的 **New-AzureSqlDatabaseServer** 函式。

## <a name="customizing-and-extending-the-publish-scripts"></a>自訂和擴充發佈指令碼

您可以自訂發佈指令碼和 JSON 組態檔。 Windows PowerShell 模組 **AzureWebAppPublishModule.psm1** 中的函式不可進行修改。 如果您想要指定不同的資料庫或變更虛擬機器的某些屬性，請編輯 JSON 組態檔。 如果您想要擴充指令碼的功能來自動建置和測試專案，您可以在 **Publish-WebApplication.ps1**中實作函式虛設常式。

若要自動建置專案，請將呼叫 MSBuild 的程式碼加入 `New-WebDeployPackage` ，如此程式碼範例所示。 MSBuild 命令的路徑會因為您所安裝的 Visual Studio 版本而有所不同。 若要取得正確路徑，您可以使用函式 **Get-MSBuildCmd**，如此範例所示。

### <a name="to-automate-building-your-project"></a>自動建置專案

1. 在全域參數區段新增 `$ProjectFile` 參數。

    ```powershell
    [Parameter(Mandatory = $false)]
    [ValidateScript({Test-Path $_ -PathType Leaf})]
    [String]
    $ProjectFile,
    ```

1. 將函式 `Get-MSBuildCmd` 複製到指令碼檔案。

    ```powershell
    function Get-MSBuildCmd
    {
            process
    {

                $path =  Get-ChildItem "HKLM:\SOFTWARE\Microsoft\MSBuild\ToolsVersions\" |
                                    Sort-Object {[double]$_.PSChildName} -Descending |
                                    Select-Object -First 1 |
                                    Get-ItemProperty -Name MSBuildToolsPath |
                                    Select -ExpandProperty MSBuildToolsPath

                $path = (Join-Path -Path $path -ChildPath 'msbuild.exe')

            return Get-Item $path
        }
    }
    ```

1. 以下列程式碼取代 `New-WebDeployPackage`，並取代建構 `$msbuildCmd` 的程式行中的預留位置。 此程式碼係用於 Visual Studio 2017。 如果您使用 Visual Studio 2015，請將 **VisualStudioVersion** 屬性變更為 `14.0` (對於 Visual Studio 2013 則為 `12.0`)。

    ```powershell
    function New-WebDeployPackage
    {
        #Write a function to build and package your web application
    ```

若要建置 Web 應用程式，使用 MsBuild.exe。 如需說明，請參閱下列網址中的 MSBuild 命令列參考：[http://go.microsoft.com/fwlink/?LinkId=391339](http://go.microsoft.com/fwlink/?LinkId=391339)

```powershell
Write-VerboseWithTime 'Build-WebDeployPackage: Start'

$msbuildCmd = '"{0}" "{1}" /T:Rebuild;Package /P:VisualStudioVersion=15.0 /p:OutputPath="{2}\MSBuildOutputPath" /flp:logfile=msbuild.log,v=d' -f (Get-MSBuildCmd), $ProjectFile, $scriptDirectory

Write-VerboseWithTime ('Build-WebDeployPackage: ' + $msbuildCmd)
```

### <a name="start-execution-of-the-build-command"></a>開始執行建置命令

```powershell
$job = Start-Process cmd.exe -ArgumentList('/C "' + $msbuildCmd + '"') -WindowStyle Normal -Wait -PassThru

if ($job.ExitCode -ne 0) {
    throw('MsBuild exited with an error. ExitCode:' + $job.ExitCode)
}

#Obtain the project name
$projectName = (Get-Item $ProjectFile).BaseName

#Construct the path to web deploy zip package
$DeployPackageDir =  '.\MSBuildOutputPath\_PublishedWebsites\{0}_Package\{0}.zip' -f $projectName

#Get the full path for the web deploy zip package. This is required for MSDeploy to work
$WebDeployPackage = Resolve-Path –LiteralPath $DeployPackageDir

Write-VerboseWithTime 'Build-WebDeployPackage: End'

return $WebDeployPackage
}
```

1. 在此程式行前呼叫 `New-WebDeployPackage` 函式：`$Config = Read-ConfigFile $Configuration` (若為 Web 應用程式) 或 `$Config = Read-ConfigFile $Configuration -HasWebDeployPackage:([Bool]$WebDeployPackage)` (若為虛擬機器)。

    ```powershell
    if($ProjectFile)
    {
    $WebDeployPackage = New-WebDeployPackage
    }
    ```

1. 從命令列使用傳遞 `$Project` 引數叫用自訂指令碼，如下列範例所示：

    ```powershell
    .\Publish-WebApplicationVM.ps1 -Configuration .\Configurations\WebApplication5-VM-dev.json `
    -ProjectFile ..\WebApplication5\WebApplication5.csproj `
    -VMPassword @{Name="VMUser";Password="Test.123"} `
    -AllowUntrusted `
    -Verbose
    ```

若要自動測試應用程式，請將程式碼加入 `Test-WebApplication`。 請務必要將 **Publish-WebApplication.ps1** 中呼叫這些函式的程式行取消註解。 如果您沒有提供實作，您可以使用 Visual Studio 手動建置專案，然後執行發佈指令碼以發佈至 Azure。

## <a name="publishing-function-summary"></a>發佈函式摘要

若要取得可以在 Windows PowerShell 命令提示字元使用之函式的說明，請使用 `Get-Help function-name`命令。 說明中會包括參數說明和範例。 **AzureWebAppPublishModule.psm1** 和 **Publish-WebApplication.ps1** 指令碼原始程式檔中也有相同的說明文字。 指令碼和說明都已當地語系化為 Visual Studio 所使用的語言。

**AzureWebAppPublishModule**

| 函式名稱 | 說明 |
| --- | --- |
| Add-AzureSQLDatabase |建立新的 Azure SQL Database。 |
| Add-AzureSQLDatabases |從 Visual Studio 所產生的 JSON 組態檔中的值建立 Azure SQL Database。 |
| Add-AzureVM |建立 Azure 虛擬機器並傳回所部署 VM 的 URL。 函式會設定必要條件，然後呼叫 **New-AzureVM** 函式 (Azure 模組) 以建立新的虛擬機器。 |
| Add-AzureVMEndpoints |將新的輸入端點加入至虛擬機器，並傳回具有新端點的虛擬機器。 |
| Add-AzureVMStorage |在目前的訂用帳戶中建立新的 Azure 儲存體帳戶。 帳戶名稱開頭是 "devtest"，後面接著唯一的英數字元字串。 此函式會傳回新儲存體帳戶的名稱。 指定新儲存體帳戶的位置或同質群組。 |
| Add-AzureWebsite |使用指定的名稱和位置建立網站。 此函式會呼叫 Azure 模組中的 **New-AzureWebsite** 函式。 如果訂用帳戶還沒有具有指定名稱的網站，此函式會建立該網站並傳回網站物件。 否則，它會傳回 `$null`。 |
| Backup-Subscription |在指令碼範圍的 `$Script:originalSubscription` 變數中儲存目前的 Azure 訂用帳戶。 此函式會在指令碼範圍中，儲存目前的 Azure 訂用帳戶 (由 `Get-AzureSubscription -Current` 取得) 與其儲存體帳戶，以及此指令碼所變更的訂用帳戶 (儲存在 `$UserSpecifiedSubscription` 變數中) 與其儲存體帳戶。 透過儲存這些值，您可以使用函式 (例如 `Restore-Subscription`) 將原始的目前訂用帳戶和儲存體帳戶還原為目前狀態 (如果目前狀態已變更)。 |
| Find-AzureVM |取得指定的 Azure 虛擬機器。 |
| Format-DevTestMessageWithTime |在訊息前面加上日期和時間。 此函式是專為寫入 Error 和 Verbose 串流的訊息所設計。 |
| Get-AzureSQLDatabaseConnectionString |組合連接字串來連線到 Azure SQL Database。 |
| Get-AzureVMStorage |傳回指定位置或同質群組中具有 "devtest *" (不區分大小寫) 名稱模式的第一個儲存體帳戶的名稱。如果 "devtest*" 儲存體帳戶不符合位置或同質群組，此函式會忽略它。 指定位置或同質群組。 |
| Get-MSDeployCmd |傳回執行 MsDeploy.exe 工具的命令。 |
| New-AzureVMEnvironment |在訂用帳戶中尋找或建立符合 JSON 組態檔中的值的虛擬機器。 |
| Publish-WebPackage |使用 MsDeploy.exe 和 Web 發佈封裝 .Zip 檔案將資源部署至網站。 此函式不會產生任何輸出。 如果呼叫 MSDeploy.exe 失敗，此函式會擲回例外狀況。 若要取得更詳細的輸出，請使用 **-Verbose** 選項。 |
| Publish-WebPackageToVM |驗證參數值，然後呼叫 **Publish-WebPackage** 函式。 |
| Read-ConfigFile |驗證 JSON 組態檔並傳回所選值的雜湊資料表。 |
| Restore-Subscription |將目前的訂用帳戶重設為原始訂用帳戶。 |
| Test-AzureModule |如果所安裝的 Azure 模組版本為 0.7.4 或更新版本，則傳回 `$true` 。 如果尚未安裝模組或模組為較舊的版本，則傳回 `$false` 。 此函式沒有參數。 |
| Test-AzureModuleVersion |如果 Azure 模組的版本為 0.7.4 或更新版本，則傳回 `$true` 。 如果尚未安裝模組或模組為較舊的版本，則傳回 `$false` 。 此函式沒有參數。 |
| Test-HttpsUrl |將輸入的 URL 轉換為 System.Uri 物件。 如果 URL 為絕對值，而且其配置為 https，則傳回 `$True` 。 如果 URL 是相對值、其配置不是 HTTPS 或輸入的字串無法轉換成 URL，則傳回 `$false` 。 |
| Test-Member |如果屬性或方法是物件的成員，則傳回 `$true` 。 否則傳回 `$false`。 |
| Write-ErrorWithTime |寫入前面會加上目前時間的錯誤訊息。 此函式會呼叫 **Format-DevTestMessageWithTime** 函式，以在將訊息寫入 Error 串流之前在訊息前面加上時間。 |
| Write-HostWithTime |將前面會加上目前時間的訊息寫入主機程式 (**Write-Host**)。 寫入主機程式的效果並不一定。 大部分裝載 Windows PowerShell 的程式會將這些訊息寫入標準輸出。 |
| Write-VerboseWithTime |寫入前面會加上目前時間的詳細資訊訊息。 它會呼叫 **Write-Verbose**，所以只有當指令碼搭配 **Verbose** 參數執行或當 [VerbosePreference] 喜好設定設為 [繼續] 時，才會顯示訊息。 |

**Publish-WebApplication**

| 函式名稱 | 說明 |
| --- | --- |
| New-AzureWebApplicationEnvironment |建立 Azure 資源，例如網站或虛擬機器。 |
| New-WebDeployPackage |此函式未實作。 您可以在此函式新增命令來建置專案。 |
| Publish-AzureWebApplication |將 Web 應用程式發佈至 Azure。 |
| Publish-WebApplication |建立並部署 Visual Studio Web 專案的 Web Apps、虛擬機器、SQL Database 和儲存體帳戶。 |
| Test-WebApplication |此函式未實作。 您可以在此函式新增命令來測試應用程式。 |

## <a name="next-steps"></a>後續步驟
請參閱[使用 Windows PowerShell 撰寫指令碼](https://technet.microsoft.com/library/bb978526.aspx)來深入了解 PowerShell 指令碼，並參閱[指令碼中心](https://azure.microsoft.com/documentation/scripts/)內的其他 Azure PowerShell 指令碼。
