---
title: 使用服務主體讓 Azure Analysis Services 工作自動化 | Microsoft Docs
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 76cadc453a696b8d19788525bfb69cf9cacd353d
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37448232"
---
# <a name="automation-with-service-principals"></a>使用服務主體進行自動化

服務主體是您在租用戶內建立的 Azure Active Directory 應用程式資源，用來執行自動資源和服務層級作業。 它們是種唯一的「使用者身分識別」，具有應用程式識別碼和密碼或憑證。 服務主體只有執行角色所定義工作所需的權限，以及為此所指派給服務主體的權限。 

在 Analysis Services 中，服務主體會與 Azure 自動化、PowerShell 自動模式、自訂用戶端應用程式和 Web 應用程式搭配使用，讓一般工作自動化。 例如，佈建伺服器、部署模型、資料重新整理、相應增加/相應減少，以及暫停/繼續都可以藉由使用服務主體來進行自動化。 權限會透過角色成員資格指派給服務主體；這與一般 Azure AD UPN 帳戶的做法很像。

## <a name="create-service-principals"></a>建立服務主體
 
服務主體可以在 Azure 入口網站中建立，或是使用 PowerShell 建立。 若要深入了解，請參閱：

[建立服務主體 - Azure 入口網站](../azure-resource-manager/resource-group-create-service-principal-portal.md)   
[建立服務主體 - PowerShell](../azure-resource-manager/resource-group-authenticate-service-principal.md)

## <a name="store-credential-and-certificate-assets-in-azure-automation"></a>在 Azure 自動化中儲存認證和憑證資產

服務主體認證和憑證可以安全地儲存在 Azure 自動化中，以進行 Runbook 作業。 若要深入了解，請參閱：

[Azure 自動化中的認證資產](../automation/automation-credentials.md)   
[Azure 自動化中的憑證資產](../automation/automation-certificates.md)

## <a name="add-service-principals-to-server-admin-role"></a>將服務主體新增至伺服器管理員角色

在您針對 Analysis Services 伺服器管理作業使用服務主體之前，必須將其新增至伺服器管理員角色。 若要深入了解，請參閱[將服務主體新增至伺服器管理員角色](analysis-services-addservprinc-admins.md)。

## <a name="service-principals-in-connection-strings"></a>連接字串中的服務主體

服務主體 appID 和密碼或憑證可以在連接字串中使用，這點與 UPN 非常類似。

### <a name="powershell"></a>PowerShell

透過 [AzureRM.AnalysisServices](https://www.powershellgallery.com/packages/AzureRM.AnalysisServices) 模組針對資源管理作業使用服務主體時，請使用 `Login-AzureRmAccount` Cmdlet。 透過 [SQLServer](https://www.powershellgallery.com/packages/SqlServer) 模組針對伺服器作業使用服務主體時，請使用 `Add-AzureAnalysisServicesAccount` Cmdlet。 

在下列範例中，appID 和密碼可用來執行模型資料庫重新整理作業：

```PowerShell
Param (

        [Parameter(Mandatory=$true)] [String] $AppId,
        [Parameter(Mandatory=$true)] [String] $PlainPWord,
        [Parameter(Mandatory=$true)] [String] $TenantId
       )
$PWord = ConvertTo-SecureString -String $PlainPWord -AsPlainText -Force

$Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $AppId, $PWord

Add-AzureAnalysisServicesAccount -Credential $Credential -ServicePrincipal -TenantId $TenantId -RolloutEnvironment "westcentralus.asazure.windows.net"

Invoke-ProcessTable -Server "asazure://westcentralus.asazure.windows.net/myserver" -TableName "MyTable" -Database "MyDb" -RefreshType "Full"
```

### <a name="amo-and-adomd"></a>AMO 和 ADOMD 

與用戶端應用程式和 Web 應用程式連線時，來自 NuGet 的 [AMO 和 ADOMD 用戶端程式庫](analysis-services-data-providers.md) 15.0.2 版及更新版本的可安裝套件，會使用下列語法在連接字串中支援服務主體：`app:AppID` 和密碼或 `cert:thumbprint`。 

在下列範例中，`appID` 和 `password` 可用來執行模型資料庫重新整理作業：

```C#
string appId = "xxx";
string authKey = "yyy";
string connString = $"Provider=MSOLAP;Data Source=asazure://westus.asazure.windows.net/<servername>;User ID=app:{appId};Password={authKey};";
Server server = new Server();
server.Connect(connString);
Database db = server.Databases.FindByName("adventureworks");
Table tbl = db.Model.Tables.Find("DimDate");
tbl.RequestRefresh(RefreshType.Full);
db.Model.SaveChanges();
```

## <a name="next-steps"></a>後續步驟
[使用 Azure PowerShell 來登入](https://docs.microsoft.com/powershell/azure/authenticate-azureps)   
[將服務主體新增至伺服器管理員角色](analysis-services-addservprinc-admins.md)   