---
title: Start-AzsReadinessChecker Cmdlet 參考 | Microsoft Docs
description: Azure Stack 整備檢查程式模組的 PowerShell Cmdlet 說明。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/26/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: 60e9a790a9b74bce7ccbdd58b320ad969c0932f3
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49079273"
---
# <a name="start-azsreadinesschecker-cmdlet-reference"></a>Start-AzsReadinessChecker Cmdlet 參考

模組：Microsoft.AzureStack.ReadinessChecker

此模組只包含單一 Cmdlet。  此 Cmdlet 會為 Azure Stack 執行一或多個預先部署或預先服務函式。

## <a name="syntax"></a>語法
```PowerShell
Start-AzsReadinessChecker
       [-CertificatePath <String>]
       -PfxPassword <SecureString>
       -RegionName <String>
       -FQDN <String>
       -IdentitySystem <String>
       [-CleanReport]
       [-OutputPath <String>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
```

```PowerShell
Start-AzsReadinessChecker
       [-CertificatePath <String>]
       -PfxPassword <SecureString>
       -DeploymentDataJSONPath <String>
       [-CleanReport]
       [-OutputPath <String>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
```

```PowerShell
Start-AzsReadinessChecker
       -PaaSCertificates <Hashtable>
       -DeploymentDataJSONPath <String>
       [-OutputPath <String>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
```

```PowerShell
Start-AzsReadinessChecker
       -PaaSCertificates <Hashtable>
       -RegionName <String>
       -FQDN <String>
       -IdentitySystem <String>
       [-OutputPath <String>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
```

```PowerShell
Start-AzsReadinessChecker
       -RegionName <String>
       -FQDN <String>
       -IdentitySystem <String>
       -Subject <OrderedDictionary>
       -RequestType <String>
       [-IncludePaaS]
       -OutputRequestPath <String>
       [-OutputPath <String>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
```

```PowerShell
Start-AzsReadinessChecker
       -PfxPassword <SecureString>
       -PfxPath <String>
       -ExportPFXPath <String>
       [-OutputPath <String>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
```


```PowerShell
Start-AzsReadinessChecker
       -AADServiceAdministrator <PSCredential>
       -AADDirectoryTenantName <String>
       -IdentitySystem <String>
       -AzureEnvironment <String>
       [-CleanReport]
       [-OutputPath <String>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
```

```PowerShell
Start-AzsReadinessChecker
       -AADServiceAdministrator <PSCredential>
       -DeploymentDataJSONPath <String>
       [-CleanReport]
       [-OutputPath <String>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
```

```PowerShell
Start-AzsReadinessChecker
       -RegistrationAccount <PSCredential>
       -RegistrationSubscriptionID <Guid>
       -AzureEnvironment <String>
       [-CleanReport]
       [-OutputPath <String>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
```

```PowerShell
Start-AzsReadinessChecker
       -RegistrationAccount <PSCredential>
       -RegistrationSubscriptionID <Guid>
       -DeploymentDataJSONPath <String>
       [-CleanReport]
       [-OutputPath <String>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
```

```PowerShell
Start-AzsReadinessChecker
       -ReportPath <String>
       [-ReportSections <String>]
       [-Summary]
       [-OutputPath <String>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
```





 ## <a name="description"></a>說明
**Start-AzsReadinessChecker** Cmdlet 會驗證憑證、Azure 帳戶、Azure 訂用帳戶和 Azure Active Directory。 在部署 Azure Stack 之前，或在執行 Azure Stack 服務動作 (例如密碼輪替) 前，請先執行驗證。 此 Cmdlet 也可用來產生基礎結構憑證的憑證簽署要求，以及選擇性地產生 PaaS 憑證的要求。  最後，此 Cmdlet 可以重新封裝 PFX 憑證，以修正常見的封裝問題。

## <a name="examples"></a>範例
**範例：產生憑證簽署要求**

```PowerShell
$regionName = 'east'
$externalFQDN = 'azurestack.contoso.com'
$subjectHash = [ordered]@{"OU"="AzureStack";"O"="Microsoft";"L"="Redmond";"ST"="Washington";"C"="US"}
Start-AzsReadinessChecker -regionName $regionName -externalFQDN $externalFQDN -subject $subjectHash -IdentitySystem ADFS -requestType MultipleCSR
```

在此範例中，Start-AzsReadinessChecker 會針對區域名稱為 "east"、外部 FQDN 為 "azurestack.contoso.com" 的 ADFS Azure Stack 部署所適用的憑證產生多個憑證簽署要求 (CSR)

**範例：驗證憑證**
```PowerShell
$password = Read-Host -Prompt "Enter PFX Password" -AsSecureString
Start-AzsReadinessChecker -CertificatePath .\Certificates\ -PfxPassword $password -RegionName east -FQDN azurestack.contoso.com -IdentitySystem AAD
```

在此範例中，系統會基於安全考量發出 PFX 密碼的提示，而 Start-AzsReadinessChecker 會在相對的資料夾 "Certificates" 中檢查是否有區域名稱為 "east"、外部 FQDN 為 "azurestack.contoso.com" 的 AAD 部署所適用的有效憑證 

**範例：驗證部署資料的憑證 (部署和支援)**
```PowerShell
$password = Read-Host -Prompt "Enter PFX Password" -AsSecureString
Start-AzsReadinessChecker -CertificatePath .\Certificates\ -PfxPassword $password -DeploymentDataJSONPath .\deploymentdata.json
```
在此部署和支援範例中，系統會基於安全考量發出 PFX 密碼的提示，而 Start-AzsReadinessChecker 會在相對的資料夾 "Certificates" 中檢查是否有身分識別、區域和外部 FQDN 皆是從為部署產生的部署資料 JSON 檔案中讀取的部署所適用的有效憑證。 

**範例：驗證 PaaS 憑證**
```PowerShell
$PaaSCertificates = @{
    'PaaSDBCert' = @{'pfxPath' = '<Path to DBAdapter PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
    'PaaSDefaultCert' = @{'pfxPath' = '<Path to Default PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
    'PaaSAPICert' = @{'pfxPath' = '<Path to API PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
    'PaaSFTPCert' = @{'pfxPath' = '<Path to FTP PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
    'PaaSSSOCert' = @{'pfxPath' = '<Path to SSO PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
}
Start-AzsReadinessChecker -PaaSCertificates $PaaSCertificates – RegionName east -FQDN azurestack.contoso.com
```

在此範例中，會以每個 PaaS 憑證的路徑和密碼建構雜湊表。 連接埠可以省略。 Start-AzsReadinessChecker 會檢查每個 PFX 路徑是否存在，並使用區域 ‘east’ 和外部 FQDN ‘azurestack.contoso.com’ 加以驗證。

**範例：驗證部署資料的 PaaS 憑證**
```PowerShell
$PaaSCertificates = @{
    'PaaSDBCert' = @{'pfxPath' = '<Path to DBAdapter PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
    'PaaSDefaultCert' = @{'pfxPath' = '<Path to Default PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
    'PaaSAPICert' = @{'pfxPath' = '<Path to API PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
    'PaaSFTPCert' = @{'pfxPath' = '<Path to FTP PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
    'PaaSSSOCert' = @{'pfxPath' = '<Path to SSO PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
}
Start-AzsReadinessChecker -PaaSCertificates $PaaSCertificates -DeploymentDataJSONPath .\deploymentdata.json
```

在此範例中，會以每個 PaaS 憑證的路徑和密碼建構雜湊表。 連接埠可以省略。 Start-AzsReadinessChecker 會檢查每個 PFX 路徑是否存在，並使用從為部署產生的部署資料 JSON 檔案中讀取的區域和外部 FQDN 加以驗證。 

**範例：驗證 Azure 身分識別**
```PowerShell
$serviceAdminCredential = Get-Credential -Message "Enter Credentials for Service Administrator of Azure Active Directory Tenant e.g. serviceadmin@contoso.onmicrosoft.com"
Start-AzsReadinessChecker -AADServiceAdministrator $serviceAdminCredential -AzureEnvironment AzureCloud -AzureDirectoryTenantName azurestack.contoso.com
```

在此範例中，系統會基於安全考量發出服務管理員帳戶認證的提示，而 Start-AzsReadinessChecker 會檢查 Azure 帳戶和 Azure Active Directory 是否適用於租用戶目錄名稱為 "azurestack.contoso.com" 的 AAD 部署


**範例：驗證部署資料的 Azure 身分識別 (部署支援)**
```PowerSHell
$serviceAdminCredential = Get-Credential -Message "Enter Credentials for Service Administrator of Azure Active Directory Tenant e.g. serviceadmin@contoso.onmicrosoft.com"
Start-AzsReadinessChecker -AADServiceAdministrator $serviceAdminCredential -DeploymentDataJSONPath .\contoso-depploymentdata.json
```

在此範例中，系統會基於安全考量發出服務管理員帳戶認證的提示，而 Start-AzsReadinessChecker 會檢查 Azure 帳戶和 Azure Active Directory 是否適用於 AzureCloud 和 TenantName 皆是從為部署產生的部署資料 JSON 檔案中讀取的 AAD 部署。


**範例：驗證 Azure 註冊**
```PowerShell
$registrationCredential = Get-Credential -Message "Enter Credentials for Subscription Owner"e.g. subscriptionowner@contoso.onmicrosoft.com"
$subscriptionID = "f7c26209-cd2d-4625-86ba-724ebeece794"
Start-AzsReadinessChecker -RegistrationAccount $registrationCredential -RegistrationSubscriptionID $subscriptionID -AzureEnvironment AzureCloud
```

在此範例中，系統會基於安全考量發出訂用帳戶擁有者認證的提示，接著 Start-AzsReadinessChecker 會對指定的帳戶和訂用帳戶執行驗證，以確定它可以用於 Azure Stack 註冊。 


**範例：驗證部署資料的 Azure 註冊 (部署小組)**
```PowerShell
$registrationCredential = Get-Credential -Message "Enter Credentials for Subscription Owner"e.g. subscriptionowner@contoso.onmicrosoft.com"
$subscriptionID = "f7c26209-cd2d-4625-86ba-724ebeece794"
Start-AzsReadinessChecker -RegistrationAccount $registrationCredential -RegistrationSubscriptionID $subscriptionID -DeploymentDataJSONPath .\contoso-deploymentdata.json
```

在此範例中，系統會基於安全考量發出訂用帳戶擁有者認證的提示，接著 Start-AzsReadinessChecker 會對指定的帳戶和訂用帳戶執行驗證，以確定它可以用於有其他詳細資料是從為部署產生的部署資料 JSON 檔案中讀取的 Azure Stack 註冊。

**範例：匯入/匯出 PFX 套件**
```PowerShell
$password = Read-Host -Prompt "Enter PFX Password" -AsSecureString
Start-AzsReadinessChecker -PfxPassword $password -PfxPath .\certificates\ssl.pfx -ExportPFXPath .\certificates\ssl_new.pfx
```

在此範例中，系統會基於安全考量發出 PFX 密碼的提示。 ssl.pfx 檔案 會匯入本機電腦憑證存放區中，然後以相同的密碼重新匯出並儲存為 ssl_new.pfx。  此程序適用於憑證驗證以旗標指出私密金鑰未設定本機電腦屬性、憑證鏈結已中斷、PFX 中有不相關的憑證存在，或憑證鏈結順序錯誤等情況。


**範例：檢視驗證報告 (部署支援)**
```PowerShell
Start-AzsReadinessChecker -ReportPath Contoso-AzsReadinessReport.json
```

在此範例中，部署或支援小組會皆來客戶 (Contoso) 傳來的整備報告，並使用 Start-AzsReadinessChecker 檢視 Contoso 所執行之驗證的狀態。

**範例：僅檢視憑證驗證的驗證報告摘要 (部署和支援)**
```PowerShell
Start-AzsReadinessChecker -ReportPath Contoso-AzsReadinessReport.json -ReportSections Certificate -Summary
```

在此範例中，部署或支援小組會皆來客戶 (Contoso) 傳來的整備報告，並使用 Start-AzsReadinessChecker 檢視 Contoso 所執行之憑證驗證的摘要狀態。



## <a name="required-parameters"></a>必要參數
> -RegionName

指定 Azure Stack 部署的區域名稱。
|  |  |
|----------------------------|--------------|
|輸入：                       |字串        |
|位置：                   |已命名         |
|預設值：              |None          |
|接受管線輸入：      |False         |
|接受萬用字元： |False         |

> -FQDN    

指定 Azure Stack 部署的外部 FQDN，同時設定為 ExternalFQDN 和 ExternalDomainName 別名。
|  |  |
|----------------------------|--------------|
|輸入：                       |字串        |
|位置：                   |已命名         |
|預設值：              |ExternalFQDN、ExternalDomainName |
|接受管線輸入：      |False         |
|接受萬用字元： |False         |

 

> -IdentitySystem    

為 Azure Stack 部署分別指定 Azure Active Directory 和 Active Directory 同盟服務的身分識別系統有效值 AAD 或 ADFS。
|  |  |
|----------------------------|--------------|
|輸入：                       |字串        |
|位置：                   |已命名         |
|預設值：              |None          |
|有效值：               |'AAD','ADFS'  |
|接受管線輸入：      |False         |
|接受萬用字元： |False         |

> -PfxPassword    

指定與 PFX 憑證檔案相關聯的密碼。
|  |  |
|----------------------------|---------|
|輸入：                       |SecureString |
|位置：                   |已命名    |
|預設值：              |None     |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -PaaSCertificates

指定包含 PaaS 憑證之路徑和密碼的雜湊表。
|  |  |
|----------------------------|---------|
|輸入：                       |雜湊表 |
|位置：                   |已命名    |
|預設值：              |None     |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -DeploymentDataJSONPath

指定 Azure Stack 部署的資料 JSON 組態檔。 此檔案是為部署產生的。
|  |  |
|----------------------------|---------|
|輸入：                       |字串   |
|位置：                   |已命名    |
|預設值：              |None     |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -PfxPath

指定有問題而需要以匯入/匯出常式修正的憑證 (如此工具中的憑證驗證所指出) 的路徑。
|  |  |
|----------------------------|---------|
|輸入：                       |字串   |
|位置：                   |已命名    |
|預設值：              |None     |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -ExportPFXPath  

指定匯入/匯出常式所產生之 PFX 檔案的目的地路徑。  
|  |  |
|----------------------------|---------|
|輸入：                       |字串   |
|位置：                   |已命名    |
|預設值：              |None     |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -Subject

指定用來產生憑證要求之主體已排序的字典。
|  |  |
|----------------------------|---------|
|輸入：                       |OrderedDictionary   |
|位置：                   |已命名    |
|預設值：              |None     |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -RequestType

指定憑證要求的 SAN 類型。 有效值為 MultipleCSR、SingleCSR。
- *MultipleCSR* 會產生多個憑證要求，每項服務各一個。
- *SingleCSR* 會為所有服務產生一個憑證要求。   

|  |  |
|----------------------------|---------|
|輸入：                       |字串   |
|位置：                   |已命名    |
|預設值：              |None     |
|有效值：               |'MultipleCSR','SingleCSR' |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -OutputRequestPath

指定憑證要求檔案的目的地路徑，目錄必須已存在。
|  |  |
|----------------------------|---------|
|輸入：                       |字串   |
|位置：                   |已命名    |
|預設值：              |None     |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -AADServiceAdministrator

指定要用於 Azure Stack 部署的 Azure Active Directory 服務管理員。
|  |  |
|----------------------------|---------|
|輸入：                       |PSCredential   |
|位置：                   |已命名    |
|預設值：              |None     |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -AADDirectoryTenantName

指定要用於 Azure Stack 部署的 Azure Active Directory 名稱。
|  |  |
|----------------------------|---------|
|輸入：                       |字串   |
|位置：                   |已命名    |
|預設值：              |None     |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -AzureEnvironment

指定要用於 Azure Stack 部署與註冊的帳戶、目錄和訂用帳戶所屬的 Azure 服務執行個體。
|  |  |
|----------------------------|---------|
|輸入：                       |字串   |
|位置：                   |已命名    |
|預設值：              |None     |
|有效值：               |'AzureCloud','AzureChinaCloud','AzureGermanCloud' |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -RegistrationAccount

指定要用於 Azure Stack 註冊的註冊帳戶。
|  |  |
|----------------------------|---------|
|輸入：                       |字串   |
|位置：                   |已命名    |
|預設值：              |None     |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -RegistrationSubscriptionID

指定要用於 Azure Stack 註冊的註冊訂用帳戶識別碼。
|  |  |
|----------------------------|---------|
|輸入：                       |Guid     |
|位置：                   |已命名    |
|預設值：              |None     |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |

> -ReportPath

指定整備報告的路徑，預設值為目前的目錄和預設報告名稱。
|  |  |
|----------------------------|---------|
|輸入：                       |字串   |
|位置：                   |已命名    |
|預設值：              |全部      |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |



## <a name="optional-parameters"></a>選擇性參數
> -CertificatePath     

指定只有憑證所需的憑證資料夾存在的路徑。

Azure Active Directory 身分識別系統的 Azure Stack 部署所需的資料夾為：

ACSBlob、ACSQueue、ACSTable、Admin Portal、ARM Admin、ARM Public、KeyVault、KeyVaultInternal、Public Portal

Active Directory 同盟服務身分識別系統的 Azure Stack 部署所需的資料夾為：

ACSBlob、ACSQueue、ACSTable、ADFS、Admin Portal、ARM Admin、ARM Public、Graph、KeyVault、KeyVaultInternal、Public Portal


|  |  |
|----------------------------|---------|
|輸入：                       |字串   |
|位置：                   |已命名    |
|預設值：              |.\Certificates |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |


> -IncludePaaS  

指定 PaaS 服務/主機名稱是否應新增至憑證要求。


|  |  |
|----------------------------|------------------|
|輸入：                       |SwitchParameter   |
|位置：                   |已命名             |
|預設值：              |False             |
|接受管線輸入：      |False             |
|接受萬用字元： |False             |


> -ReportSections        

指定是否僅顯示報告摘要，而省略詳細資料。
|  |  |
|----------------------------|---------|
|輸入：                       |字串   |
|位置：                   |已命名    |
|預設值：              |全部      |
|有效值：               |'Certificate','AzureRegistration','AzureIdentity','Jobs','All' |
|接受管線輸入：      |False    |
|接受萬用字元： |False    |


> -Summary 

指定是否僅顯示報告摘要，而省略詳細資料。
|  |  |
|----------------------------|------------------|
|輸入：                       |SwitchParameter   |
|位置：                   |已命名             |
|預設值：              |False             |
|接受管線輸入：      |False             |
|接受萬用字元： |False             |


> -CleanReport  

移除先前的執行和驗證歷程記錄，並將驗證寫入至新報表。
|  |  |
|----------------------------|------------------|
|輸入：                       |SwitchParameter   |
|別名：                    |cf                |
|位置：                   |已命名             |
|預設值：              |False             |
|接受管線輸入：      |False             |
|接受萬用字元： |False             |


> -OutputPath    

指定用來儲存整備 JSON 報告和詳細記錄檔的自訂路徑。  如果路徑不存在，工具會嘗試建立目錄。
|  |  |
|----------------------------|------------------|
|輸入：                       |字串            |
|位置：                   |已命名             |
|預設值：              |$ENV:TEMP\AzsReadinessChecker  |
|接受管線輸入：      |False             |
|接受萬用字元： |False             |


> -Confirm  

在執行 Cmdlet 前提示您確認。
|  |  |
|----------------------------|------------------|
|輸入：                       |SwitchParameter   |
|別名：                    |cf                |
|位置：                   |已命名             |
|預設值：              |False             |
|接受管線輸入：      |False             |
|接受萬用字元： |False             |


> -WhatIf  

顯示執行 Cmdlet 後會發生的情況。 Cmdlet 並不會執行。
|  |  |
|----------------------------|------------------|
|輸入：                       |SwitchParameter   |
|別名：                    |wi                |
|位置：                   |已命名             |
|預設值：              |False             |
|接受管線輸入：      |False             |
|接受萬用字元： |False             |

 
