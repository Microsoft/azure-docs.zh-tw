---
title: 在 Azure Stack 上部署 App Service 之前 | Microsoft Docs
description: 在 Azure Stack 上部署 App Service 之前必須完成的步驟
services: azure-stack
documentationcenter: ''
author: apwestgarth
manager: stefsch
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/20/2018
ms.author: anwestg
ms.openlocfilehash: 08fcaa44fc807f986324b8304010792af99fd838
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387988"
---
# <a name="before-you-get-started-with-app-service-on-azure-stack"></a>開始使用 Azure Stack 上的 App Service 之前

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

在部署 Azure Stack 上的 Azure App Service 之前，您必須完成本文中的先決條件步驟。

> [!IMPORTANT]
> 在部署 Azure App Service 1.3 之前，請先將 1807 更新套用到您的 Azure Stack 整合式系統，或部署最新的「Azure Stack 開發套件」(ASDK)。

## <a name="download-the-installer-and-helper-scripts"></a>下載安裝程式與協助程式指令碼

1. 下載 [Azure Stack 上的 App Service 部署協助程式指令碼](https://aka.ms/appsvconmashelpers)。
2. 下載 [Azure App Service on Azure Stack 安裝程式](https://aka.ms/appsvconmasinstaller)。
3. 從協助程式指令碼 .zip 檔案解壓縮檔案。 系統會解壓縮下列檔案和資料夾：

   - Common.ps1
   - Create-AADIdentityApp.ps1
   - Create-ADFSIdentityApp.ps1
   - Create-AppServiceCerts.ps1
   - Get-AzureStackRootCert.ps1
   - Remove-AppService.ps1
   - Modules 資料夾
     - GraphAPI.psm1

## <a name="high-availability"></a>高可用性

Azure Stack 1802 更新已增加對容錯網域的支援。 Azure Stack 上的 Azure App Service 的新部署將會分布至各個容錯網域並提供容錯移轉功能。

針對在 1802 更新之前部署的 Azure Stack 上的 Azure App Service 現有部署，請參閱[跨容錯網域重新平衡 App Service 資源提供者](azure-stack-app-service-fault-domain-update.md)一文。

此外，在高可用性設定中，部署所需的檔案伺服器和 SQL Server 執行個體。

## <a name="get-certificates"></a>取得憑證

### <a name="azure-resource-manager-root-certificate-for-azure-stack"></a>適用於 Azure Stack 的 Azure Resource Manager 根憑證

在可以連線到 Azure Stack 整合式系統或 Azure Stack 開發套件主機上特殊權限端點的電腦上，開啟已提高權限的 PowerShell 工作階段。

從您解壓縮協助程式指令碼所在的資料夾執行 *Get-AzureStackRootCert.ps1* 指令碼。 此指令碼會在與 App Service 建立憑證時所需之指令碼相同的資料夾中建立根憑證。

當您執行下列 PowerShell 命令時，必須提供特殊權限的端點和 AzureStack\CloudAdmin 的認證。

```PowerShell
    Get-AzureStackRootCert.ps1
```

#### <a name="get-azurestackrootcertps1-script-parameters"></a>Get-AzureStackRootCert.ps1 指令碼參數

| 參數 | 必要或選用 | 預設值 | 說明 |
| --- | --- | --- | --- |
| PrivilegedEndpoint | 必要 | AzS-ERCS01 | 特殊權限的端點 |
| CloudAdminCredential | 必要 | AzureStack\CloudAdmin | Azure Stack 雲端管理的網域帳戶認證 |

### <a name="certificates-required-for-asdk-deployment-of-azure-app-service"></a>Azure App Service 的 ASDK 部署所需的憑證

*Create-AppServiceCerts.ps1* 指令碼會搭配 Azure Stack 憑證授權單位運作，以建立 App Service 所需的四個憑證。

| 檔案名稱 | 使用 |
| --- | --- |
| _.appservice.local.azurestack.external.pfx | App Service 預設 SSL 憑證 |
| api.appservice.local.azurestack.external.pfx | App Service API SSL 憑證 |
| ftp.appservice.local.azurestack.external.pfx | App Service 發行者 SSL 憑證 |
| sso.appservice.local.azurestack.external.pfx | App Service 身分識別應用程式憑證 |

若要建立憑證，請依照下列步驟執行：

1. 使用 AzureStack\AzureStackAdmin 帳戶登入 Azure Stack 開發套件主機。
2. 開啟提升權限的 PowerShell 工作階段。
3. 從您解壓縮協助程式指令碼所在的資料夾執行 *Create-AppServiceCerts.ps1* 指令碼。 此指令碼會在與 App Service 建立憑證所需指令碼的相同資料夾中建立四個憑證。
4. 輸入密碼來保護 .pfx 檔案，並記下密碼。 您將必須在 Azure Stack 上的 App Service 安裝程式中輸入此密碼。

#### <a name="create-appservicecertsps1-script-parameters"></a>Create-AppServiceCerts.ps1 指令碼參數

| 參數 | 必要或選用 | 預設值 | 說明 |
| --- | --- | --- | --- |
| pfxPassword | 必要 | Null | 協助保護憑證私密金鑰的密碼 |
| DomainName | 必要 | local.azurestack.external | Azure Stack 區域和網域尾碼 |

### <a name="certificates-required-for-azure-stack-production-deployment-of-azure-app-service"></a>Azure App Service 的 Azure Stack 生產部署所需的憑證

若要在生產環境中執行資源提供者，您必須提供下列憑證：

- 預設網域憑證
- API 憑證
- 發行憑證
- 身分識別憑證

#### <a name="default-domain-certificate"></a>預設網域憑證

預設網域憑證放在「前端」角色。 對 Azure App Service 要求使用萬用字元或預設網域的使用者應用程式會使用此憑證。 憑證也用於原始檔控制作業 (Kudu)。

憑證的格式必須是 .pfx，而且應該是三主體的萬用字元憑證。 此需求允許一個憑證同時涵蓋用於原始檔控制作業的預設網域和 SCM 端點。

| 格式 | 範例 |
| --- | --- |
| \*.appservice.\<region\>.\<DomainName\>.\<extension\> | \*.appservice.redmond.azurestack.external |
| \*.scm.appservice.<region>.<DomainName>.<extension> | \*.scm.appservice.redmond.azurestack.external |
| \*.sso.appservice.<region>.<DomainName>.<extension> | \*.sso.appservice.redmond.azurestack.external |

#### <a name="api-certificate"></a>API 憑證

API 憑證位於管理角色中。 資源提供者會使用它來協助保護 API 呼叫。 用於發佈的憑證必須包含符合 API DNS 項目的主體。

| 格式 | 範例 |
| --- | --- |
| api.appservice.\<region\>.\<DomainName\>.\<extension\> | api.appservice.redmond.azurestack.external |

#### <a name="publishing-certificate"></a>發行憑證

發行者角色的憑證會在應用程式擁有者上傳內容時，保護其 FTPS 流量。 用於發佈的憑證必須包含符合 FTPS DNS 項目的主體。

| 格式 | 範例 |
| --- | --- |
| ftp.appservice.\<region\>.\<DomainName\>.\<extension\> | ftp.appservice.redmond.azurestack.external |

#### <a name="identity-certificate"></a>身分識別憑證

身分識別應用程式的憑證可讓：

- Azure Active Directory (Azure AD) 或 Active Directory 同盟服務 (AD FS) 目錄、Azure Stack 和 App Service 之間整合，以支援與計算資源提供者的整合。
- Azure App Service on Azure Stack 中進階開發人員工具的單一登入案例。

身分識別的憑證必須包含符合下列格式的主體。

| 格式 | 範例 |
| --- | --- |
| sso.appservice.\<region\>.\<DomainName\>.\<extension\> | sso.appservice.redmond.azurestack.external |

## <a name="virtual-network"></a>虛擬網路

Azure Stack 上的 Azure App Service 可讓您將資源提供者部署至現有的虛擬網路，或者讓您在部署時建立一個虛擬網路。 使用現有的虛擬網路時，便能以內部 IP 連線至 Azure Stack 上的 Azure App Service 所需檔案伺服器和 SQL Server。 安裝 Azure Stack 上的 Azure App Service 之前，您必須為虛擬網路設定下列位址範圍和子網路：

虛擬網路 - /16

子網路

- ControllersSubnet /24
- ManagementServersSubnet /24
- FrontEndsSubnet /24
- PublishersSubnet /24
- WorkersSubnet /21

## <a name="prepare-the-file-server"></a>準備檔案伺服器

Azure App Service 需要使用檔案伺服器。 在實際執行的部署中，必須將檔案伺服器設定為高度可用，且能夠處理失敗。

若是只部署 Azure Stack 開發套件，您可以使用[範例 Azure Resource Manager 部署範本](https://aka.ms/appsvconmasdkfstemplate)來部署已設定的單一節點檔案伺服器。 單一節點檔案伺服器會位於工作群組中。

>[!IMPORTANT]
> 如果您選擇在現有的虛擬網路中部署 App Service，則應該將「檔案伺服器」部署至與 App Service 不同的子網路。

### <a name="provision-groups-and-accounts-in-active-directory"></a>在 Active Directory 中佈建群組和帳戶

1. 建立下列 Active Directory 全域安全性群組：

   - FileShareOwners
   - FileShareUsers

2. 建立下列 Active Directory 帳戶做為服務帳戶：

   - FileShareOwner
   - FileShareUser

   基於安全性最佳做法，這些帳戶 (以及所有 Web 角色) 的使用者應該是獨一無二的，而且必須擁有強式使用者名稱和密碼。 使用下列條件設定密碼：

   - 啟用 [密碼永久有效]。
   - 啟用 [使用者不可變更密碼]。
   - 停用 [使用者必須在下次登入時變更密碼]。

3. 將帳戶新增至群組成員資格，如下所示：

   - 將 **FileShareOwner** 新增至 **FileShareOwners** 群組。
   - 將 **FileShareUser** 新增至 **FileShareUsers** 群組。

### <a name="provision-groups-and-accounts-in-a-workgroup"></a>在工作群組中佈建群組和帳戶

>[!NOTE]
> 當您設定檔案伺服器時，請從**系統管理員命令提示字元**中執行下列所有命令。 <br>***不要使用 PowerShell。***

當您使用 Azure Resource Manager 範本時，使用者已建立。

1. 執行下列命令來建立 FileShareOwner 與 FileShareUser 帳戶。 以您自己的值取代 `<password>`。

   ``` DOS
   net user FileShareOwner <password> /add /expires:never /passwordchg:no
   net user FileShareUser <password> /add /expires:never /passwordchg:no
   ```

2. 執行以下 WMIC 命令，將帳戶的密碼設定為永不過期：

   ``` DOS
   WMIC USERACCOUNT WHERE "Name='FileShareOwner'" SET PasswordExpires=FALSE
   WMIC USERACCOUNT WHERE "Name='FileShareUser'" SET PasswordExpires=FALSE
   ```

3. 建立本機群組 FileShareUsers 和 FileShareOwners，並在第一個步驟中為其新增帳戶：

   ``` DOS
   net localgroup FileShareUsers /add
   net localgroup FileShareUsers FileShareUser /add
   net localgroup FileShareOwners /add
   net localgroup FileShareOwners FileShareOwner /add
   ```

### <a name="provision-the-content-share"></a>佈建內容共用

內容共用包含租用戶網站內容。 在單一檔案伺服器上佈建內容共用的程序與在 Active Directory 和工作群組環境中均相同。 但是對於 Active Directory 中的容錯移轉叢集則不同。

#### <a name="provision-the-content-share-on-a-single-file-server-active-directory-or-workgroup"></a>在單一檔案伺服器上 (Active Directory 或工作群組) 佈建內容共用

在單一檔案伺服器上，請在提升權限的命令提示字元中執行下列命令。 以您的環境中的對應路徑，取代`C:\WebSites` 的值。

```DOS
set WEBSITES_SHARE=WebSites
set WEBSITES_FOLDER=C:\WebSites
md %WEBSITES_FOLDER%
net share %WEBSITES_SHARE% /delete
net share %WEBSITES_SHARE%=%WEBSITES_FOLDER% /grant:Everyone,full
```

### <a name="configure-access-control-to-the-shares"></a>設定共用的存取控制

在檔案伺服器上或容錯移轉叢集節點 (目前的叢集資源擁有者) 的提高權限命令提示字元中，執行下列命令。 以環境特有的值取代斜體的值。

#### <a name="active-directory"></a>Active Directory

```DOS
set DOMAIN=<DOMAIN>
set WEBSITES_FOLDER=C:\WebSites
icacls %WEBSITES_FOLDER% /reset
icacls %WEBSITES_FOLDER% /grant Administrators:(OI)(CI)(F)
icacls %WEBSITES_FOLDER% /grant %DOMAIN%\FileShareOwners:(OI)(CI)(M)
icacls %WEBSITES_FOLDER% /inheritance:r
icacls %WEBSITES_FOLDER% /grant %DOMAIN%\FileShareUsers:(CI)(S,X,RA)
icacls %WEBSITES_FOLDER% /grant *S-1-1-0:(OI)(CI)(IO)(RA,REA,RD)
```

#### <a name="workgroup"></a>工作群組

```DOS
set WEBSITES_FOLDER=C:\WebSites
icacls %WEBSITES_FOLDER% /reset
icacls %WEBSITES_FOLDER% /grant Administrators:(OI)(CI)(F)
icacls %WEBSITES_FOLDER% /grant FileShareOwners:(OI)(CI)(M)
icacls %WEBSITES_FOLDER% /inheritance:r
icacls %WEBSITES_FOLDER% /grant FileShareUsers:(CI)(S,X,RA)
icacls %WEBSITES_FOLDER% /grant *S-1-1-0:(OI)(CI)(IO)(RA,REA,RD)
```

## <a name="prepare-the-sql-server-instance"></a>準備 SQL Server 執行個體

針對 Azure Stack 上的 Azure App Service 託管和計量資料庫，您必須準備 SQL Server 執行個體，以用來存放 App Service 資料庫。

針對 Azure Stack 開發套件部署，您可以使用 SQL Server Express 2014 SP2 或更新版本。

若要用於生產環境及高可用性，您應該使用完整版本的 SQL Server 2014 SP2 或更新版本，啟用混合模式驗證，並在[高可用性配置](https://docs.microsoft.com/sql/sql-server/failover-clusters/high-availability-solutions-sql-server)中部署。

Azure Stack 上的 Azure App Service 的 SQL Server 執行個體必須能夠從所有 App Service 角色存取。 您可以在 Azure Stack 中的預設提供者訂用帳戶中部署 SQL Server。 或者，您可以使用組織中現有的基礎結構 (請確認可以連線到 Azure Stack)。 如果您使用 Azure Marketplace 映像，請記得設定適用的防火牆。

>[!NOTE]
> 您可以透過 Marketplace 管理功能取得一些 SQL IaaS 虛擬機器映像。 請確定在使用 Marketplace 項目部署 VM 之前，一律會下載最新版的 SQL IaaS 延伸模組。 SQL 映像與 Azure 中提供的 SQL VM 相同。 針對從這些映像建立的 SQL VM，IaaS 延伸模組和對應的入口網站增強功能，可提供自動修補和備份功能之類的功能。
>
針對任何 SQL Server 角色，您可以使用預設執行個體或具名執行個體。 如果您使用具名執行個體，請務必手動啟動 SQL Server Browser 服務並開啟連接埠 1434。

>[!IMPORTANT]
> 如果您選擇在現有的虛擬網路中部署 App Service，則應該將 SQL Server 部署至與 App Service 和「檔案伺服器」不同的子網路。
>

## <a name="create-an-azure-active-directory-application"></a>建立 Azure Active Directory 應用程式

設定 Azure AD 服務主體，以支援下列作業：

- 背景工作層上的虛擬機器擴展集整合。
- 對 Azure Functions 入口網站和進階開發人員工具使用 SSO。

這些步驟只適用於 Azure AD 保護的 Azure Stack 環境。

系統管理員必須設定 SSO 來執行下列動作：

- 啟用 App Service (Kudu) 內的進階開發人員工具。
- 讓 Azure Functions 入口網站體驗可供使用。

請遵循下列步驟：

1. 以 azurestack\AzureStackAdmin 身分開啟 PowerShell 執行個體。
2. 移至您在[先決條件步驟](https://docs.microsoft.com/azure/azure-stack/azure-stack-app-service-before-you-get-started#download-the-azure-app-service-on-azure-stack-installer-and-helper-scripts)中下載並解壓縮的指令碼位置。
3. [安裝適用於 Azure Stack 的 PowerShell](azure-stack-powershell-install.md)。
4. 執行 **Create-AADIdentityApp.ps1** 指令碼。 當系統提示時，請輸入您部署 Azure Stack 時使用的 Azure AD 租用戶識別碼。 例如，輸入 **myazurestack.onmicrosoft.com**。
5. 在 [認證] 視窗中，輸入您的 Azure AD 服務管理帳戶和密碼。 選取 [確定] 。
6. 輸入[稍早建立的憑證](https://docs.microsoft.com/azure/azure-stack/azure-stack-app-service-before-you-get-started#certificates-required-for-azure-app-service-on-azure-stack)的憑證檔案路徑和憑證密碼。 根據預設值，針對此步驟建立的憑證是 **sso.appservice.local.azurestack.external.pfx**。
7. 此指令碼會在租用戶 Azure AD 執行個體中建立新的應用程式。 請記下 PowerShell 輸出中傳回的應用程式識別碼。 安裝期間會需要這項資訊。
8. 開啟新的瀏覽器視窗，並以 [Azure Active Directory 服務管理員](https://portal.azure.com)身分登入 Azure 入口網站。
9. 開啟 Azure AD 資源提供者。
10. 選取 [應用程式註冊]。
11. 搜尋步驟 7 傳回的應用程式識別碼。 隨即列出 App Service 應用程式。
12. 選取清單中的 [應用程式]。
13. 選取 [Settings] \(設定) 。
14. 選取 [必要權限] > [授與權限] > [是]。

```PowerShell
    Create-AADIdentityApp.ps1
```

| 參數 | 必要或選用 | 預設值 | 說明 |
| --- | --- | --- | --- |
| DirectoryTenantName | 必要 | Null | Azure AD 租用戶識別碼。 提供 GUID 或字串。 例如，myazureaaddirectory.onmicrosoft.com。 |
| AdminArmEndpoint | 必要 | Null | 管理員 Azure Resource Manager 端點。 例如，adminmanagement.local.azurestack.external。 |
| TenantARMEndpoint | 必要 | Null | 租用戶 Azure Resource Manager 端點。 例如，management.local.azurestack.external。 |
| AzureStackAdminCredential | 必要 | Null | Azure AD Service 管理員的認證。 |
| CertificateFilePath | 必要 | Null | 稍早產生之身分識別應用程式憑證檔案的**完整路徑**。 |
| CertificatePassword | 必要 | Null | 協助保護憑證私密金鑰的密碼。 |
| 環境 | 選用 | AzureCloud | 所支援雲端環境的名稱，此雲端環境提供了目標「Azure Active Directory Graph 服務」。  允許的值為：'AzureCloud'、'AzureChinaCloud'、'AzureUSGovernment'、'AzureGermanCloud'.|

## <a name="create-an-active-directory-federation-services-application"></a>建立 Active Directory 同盟服務應用程式

對於受到 AD FS 保護的 Azure Stack 環境，您必須設定 AD FS 服務主體，以支援下列作業：

- 背景工作層上的虛擬機器擴展集整合。
- 對 Azure Functions 入口網站和進階開發人員工具使用 SSO。

系統管理員必須設定 SSO 來執行下列動作：

- 針對背景工作層上的虛擬機器擴展集整合設定服務主體。
- 啟用 App Service (Kudu) 內的進階開發人員工具。
- 讓 Azure Functions 入口網站體驗可供使用。

請遵循下列步驟：

1. 以 azurestack\AzureStackAdmin 身分開啟 PowerShell 執行個體。
2. 移至您在[先決條件步驟](https://docs.microsoft.com/azure/azure-stack/azure-stack-app-service-before-you-get-started#download-the-azure-app-service-on-azure-stack-installer-and-helper-scripts)中下載並解壓縮的指令碼位置。
3. [安裝適用於 Azure Stack 的 PowerShell](azure-stack-powershell-install.md)。
4. 執行 **Create-ADFSIdentityApp.ps1** 指令碼。
5. 在 [認證] 視窗中，輸入您的 AD FS 雲端管理帳戶和密碼。 選取 [確定] 。
6. 提供[稍早建立之憑證](https://docs.microsoft.com/azure/azure-stack/azure-stack-app-service-before-you-get-started#certificates-required-for-azure-app-service-on-azure-stack)的憑證檔案路徑和憑證密碼。 根據預設值，針對此步驟建立的憑證是 **sso.appservice.local.azurestack.external.pfx**。

```PowerShell
    Create-ADFSIdentityApp.ps1
```

| 參數 | 必要或選用 | 預設值 | 說明 |
| --- | --- | --- | --- |
| AdminArmEndpoint | 必要 | Null | 管理員 Azure Resource Manager 端點。 例如，adminmanagement.local.azurestack.external。 |
| PrivilegedEndpoint | 必要 | Null | 特殊權限的端點。 例如，AzS-ERCS01。 |
| CloudAdminCredential | 必要 | Null | Azure Stack 雲端管理的網域帳戶認證。 例如，Azurestack\CloudAdmin。 |
| CertificateFilePath | 必要 | Null | 識別應用程式憑證 PFX 檔案的**完整路徑**。 |
| CertificatePassword | 必要 | Null | 協助保護憑證私密金鑰的密碼。 |

## <a name="next-steps"></a>後續步驟

[安裝 App Service 資源提供者](azure-stack-app-service-deploy.md)
