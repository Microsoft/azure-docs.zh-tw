---
title: 為 Azure Stack 整合式系統部署驗證 Azure Stack 公開金鑰基礎結構憑證 | Microsoft Docs
description: 說明如何驗證 Azure Stack 整合式系統的 Azure Stack PKI 憑證。 包括如何使用 Azure Stack 憑證檢查工具。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/24/2018
ms.author: mabrigg
ms.reviewer: ppacent
ms.openlocfilehash: e381d2ed3c6a972d776dd31f311fcebe2e35823a
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42917078"
---
# <a name="validate-azure-stack-pki-certificates"></a>驗證 Azure Stack PKI 憑證

[從 PowerShell 資源庫](https://aka.ms/AzsReadinessChecker)可取得本文中所述的 Azure Stack 整備檢查工具。 您可以使用此工具來驗證[產生的 PKI 憑證](azure-stack-get-pki-certs.md)是否適用於預先部署。 您應保留足夠的時間來測試及重新發出憑證，以驗證憑證 (如有需要的話)。

整備檢查工具會執行下列憑證驗證：

- **讀取 PFX**  
    確認 PFX 檔案有效且密碼正確，並在公用資訊未受到密碼保護時發出警告。 
- **簽章演算法**  
    確認簽章演算法不是 SHA1。
- **私密金鑰**  
    確認私密金鑰存在，且是以本機電腦屬性匯出。 
- **信任鏈結**  
    確認信任鏈結完整包括自我簽署憑證的檢查。
- **DNS 名稱**  
    檢查 SAN 包含每個端點的相關 DNS 名稱，或支援萬用字元是否存在。
- **金鑰使用方式**  
    檢查金鑰使用方式是否包含數位簽章和金鑰加密，而增強金鑰使用方法包含伺服器驗證和用戶端驗證。
- **金鑰大小**  
    確認金鑰大小為 2048 或更大。
- **鏈結順序**  
    檢查其他鏈結的順序，確認順序正確。
- **其他憑證**  
    請確定除了相關的分葉憑證及其鏈結之外，PFX 中沒有封裝其他憑證。
- **沒有設定檔**  
    檢查新的使用者無須載入使用者設定檔即可載入 PFX 資料，在憑證服務期間模擬 gMSA 帳戶的行為。

> [!IMPORTANT]  
> PKI 憑證是 PFX 檔案，且密碼都應該視為機密資訊。

## <a name="prerequisites"></a>必要條件

在驗證 Azure Stack 部署的 PKI 憑證之前，您的系統應該符合下列必要條件：

- Microsoft Azure Stack 整備檢查工具
- 遵循[準備指示](azure-stack-prepare-pki-certs.md)匯出的 SSL 憑證
- DeploymentData.json
- Windows 10 或 Windows Server 2016

## <a name="perform-core-services-certificate-validation"></a>執行核心服務憑證驗證

使用下列步驟來進行準備，以及驗證用於部署和祕密輪替的 Azure Stack PKI 憑證：

1. 執行下列 Cmdlet，以從 PowerShell (5.1 或更新版本) 提示字元安裝 **AzsReadinessChecker**：

    ````PowerShell  
        Install-Module Microsoft.AzureStack.ReadinessChecker -force 
    ````

2. 建立憑證目錄結構。 在下列範例中，您可以將 `<c:\certificates>` 變更為您所選擇的新目錄路徑。

    ````PowerShell  
    New-Item C:\Certificates -ItemType Directory
    
    $directories = 'ACSBlob','ACSQueue','ACSTable','ADFS','Admin Portal','ARM Admin','ARM Public','Graph','KeyVault','KeyVaultInternal','Public Portal'
    
    $destination = 'c:\certificates'
    
    $directories | % { New-Item -Path (Join-Path $destination $PSITEM) -ItemType Directory -Force}
    ````
    
    > [!Note]  
    > 如果您使用 AD FS 作為身分識別系統，則需要 AD FS 和 Graph。
    
     - 將您的憑證放在上一個步驟中建立的適當目錄。 例如︰  
        - `c:\certificates\ACSBlob\CustomerCertificate.pfx`
        - `c:\certificates\Certs\Admin Portal\CustomerCertificate.pfx`
        - `c:\certificates\Certs\ARM Admin\CustomerCertificate.pfx`

3. 在 PowerShell 視窗中，將 **RegionName** 和 **FQDN** 的值變更為適合 Azure Stack 環境的值，並執行下列命令：

    ````PowerShell  
    $pfxPassword = Read-Host -Prompt "Enter PFX Password" -AsSecureString 

    Start-AzsReadinessChecker -CertificatePath c:\certificates -pfxPassword $pfxPassword -RegionName east -FQDN azurestack.contoso.com -IdentitySystem AAD 

    ````

4. 檢查輸出，所有憑證都通過所有測試。 例如︰

    ````PowerShell
    AzsReadinessChecker v1.1803.405.3 started
    Starting Certificate Validation

    Starting Azure Stack Certificate Validation 1.1803.405.3
    Testing: ARM Public\ssl.pfx
        Read PFX: OK
        Signature Algorithm: OK
        Private Key: OK
        Cert Chain: OK
        DNS Names: OK
        Key Usage: OK
        Key Size: OK
        Chain Order: OK
        Other Certificates: OK
    Testing: ACSBlob\ssl.pfx
        Read PFX: OK
        Signature Algorithm: OK
        Private Key: OK
        Cert Chain: OK
        DNS Names: OK
        Key Usage: OK
        Key Size: OK
        Chain Order: OK
        Other Certificates: OK
    Detailed log can be found C:\AzsReadinessChecker\CertificateValidation\CertChecker.log

    Finished Certificate Validation

    AzsReadinessChecker Log location: C:\AzsReadinessChecker\AzsReadinessChecker.log
    AzsReadinessChecker Report location: 
    C:\AzsReadinessChecker\AzsReadinessReport.json
    AzsReadinessChecker Completed
    ````

### <a name="known-issues"></a>已知問題

**徵兆**：測試會略過

**原因**：如果不符合相依性，AzsReadinessChecker 會略過某些測試：

 - 如果信任鏈結失敗，則會略過其他憑證。

    ````PowerShell  
    Testing: ACSBlob\singlewildcard.pfx
        Read PFX: OK
        Signature Algorithm: OK
        Private Key: OK
        Cert Chain: OK
        DNS Names: Fail
        Key Usage: OK
        Key Size: OK
        Chain Order: OK
        Other Certificates: Skipped
    Details:
    The certificate records '*.east.azurestack.contoso.com' do not contain a record that is valid for '*.blob.east.azurestack.contoso.com'. Please refer to the documentation for how to create the required certificate file.
    The Other Certificates check was skipped because Cert Chain and/or DNS Names failed. Follow the guidance to remediate those issues and recheck. 
    Detailed log can be found C:\AzsReadinessChecker\CertificateValidation\CertChecker.log

    Finished Certificate Validation

    AzsReadinessChecker Log location: C:\AzsReadinessChecker\AzsReadinessChecker.log
    AzsReadinessChecker Report location (for OEM): C:\AzsReadinessChecker\AzsReadinessChecker.log
    AzsReadinessChecker Completed
    ````

**解析**：遵循每個憑證測試集合下詳細資料區段中的工具指引。

## <a name="perform-platform-as-a-service-certificate-validation"></a>執行平台即服務憑證驗證

如果規劃了 SQL/MySQL 或 App Service 部署，請使用下列步驟來準備及驗證用於平台即服務 (PaaS) 憑證的 Azure Stack PKI 憑證。

1.  執行下列 Cmdlet，以從 PowerShell (5.1 或更新版本) 提示字元安裝 **AzsReadinessChecker**：

    ````PowerShell  
      Install-Module Microsoft.AzureStack.ReadinessChecker -force
    ````

2.  建立巢狀雜湊表，並在其中針對每個需要驗證的 PaaS 憑證包含其路徑和密碼。 在 PowerShell 視窗中執行：

    ```PowerShell
        $PaaSCertificates = @{
        'PaaSDBCert' = @{'pfxPath' = '<Path to DBAdapter PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
        'PaaSDefaultCert' = @{'pfxPath' = '<Path to Default PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
        'PaaSAPICert' = @{'pfxPath' = '<Path to API PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
        'PaaSFTPCert' = @{'pfxPath' = '<Path to FTP PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
        'PaaSSSOCert' = @{'pfxPath' = '<Path to SSO PFX>';'pfxPassword' = (ConvertTo-SecureString -String '<Password for PFX>' -AsPlainText -Force)}
        }
    ```

3.  將 **RegionName** 和 **FQDN** 的值變更為符合 Azure Stack 環境，以啟動驗證。 然後，執行：

    ```PowerShell
    Start-AzsReadinessChecker -PaaSCertificates $PaaSCertificates -RegionName east -FQDN azurestack.contoso.com 
    ```
4.  檢查輸出，並檢查所有憑證都通過所有測試。

    ```PowerShell
    AzsReadinessChecker v1.1805.425.2 started
    Starting PaaS Certificate Validation
    
    Starting Azure Stack Certificate Validation 1.0 
    Testing: PaaSCerts\wildcard.appservice.pfx
        Read PFX: OK
        Signature Algorithm: OK
        Private Key: OK
        Cert Chain: OK
        DNS Names: OK
        Key Usage: OK
        Key Size: OK
        Chain Order: OK
        Other Certificates: OK
    Testing: PaaSCerts\api.appservice.pfx
        Read PFX: OK
        Signature Algorithm: OK
        Private Key: OK
        Cert Chain: OK
        DNS Names: OK
        Key Usage: OK
        Key Size: OK
        Chain Order: OK
        Other Certificates: OK
    Testing: PaaSCerts\wildcard.dbadapter.pfx
        Read PFX: OK
        Signature Algorithm: OK
        Private Key: OK
        Cert Chain: OK
        DNS Names: OK
        Key Usage: OK
        Key Size: OK
        Chain Order: OK
        Other Certificates: OK
    Testing: PaaSCerts\sso.appservice.pfx
        Read PFX: OK
        Signature Algorithm: OK
        Private Key: OK
        Cert Chain: OK
        DNS Names: OK
        Key Usage: OK
        Key Size: OK
    ```

## <a name="using-validated-certificates"></a>使用驗證的憑證

一旦您的憑證已通過 AzsReadinessChecker 的驗證，您就可以在 Azure Stack 部署中使用憑證，或將其用來進行 Azure Stack 秘密輪替。 

 - 針對部署，您可以安全地將憑證傳送給您的部署工程師，讓他們可以將其複製到部署主機上，如 [Azure Stack PKI 需求文件](azure-stack-pki-certs.md)中所述。
 - 針對秘密輪替，您可以遵循 [Azure Stack 秘密輪替文件](azure-stack-rotate-secrets.md)中的指示，使用憑證來更新 Azure Stack 環境中公開基礎結構端點的舊憑證。
 - 針對 PaaS 服務，您可以遵循[在 Azure Stack 中提供服務的概觀文件](azure-stack-offer-services-overview.md)，使用憑證在 Azure Stack 中安裝 SQL、MySQL 和 App Service 資源提供者。

## <a name="next-steps"></a>後續步驟

[資料中心身分識別整合](azure-stack-integrate-identity.md)