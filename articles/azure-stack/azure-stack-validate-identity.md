---
title: 驗證 Azure Stack 的 Azure 身分識別 | Microsoft Docs
description: 使用 Azure Stack 整備檢查程式來驗證 Azure 身分識別。
services: azure-stack
documentationcenter: ''
author: brenduns
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 05/08/2018
ms.author: brenduns
ms.reviewer: ''
ms.openlocfilehash: fe5e7281cbe01ad11f667729df344f91ef1327d2
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "33937558"
---
# <a name="validate-azure-identity"></a>驗證 Azure 身分識別 
使用 Azure Stack 整備檢查程式工具 (AzsReadinessChecker) 來驗證 Azure Active Directory (Azure AD) 是否已準備好與 Azure Stack 搭配使用。 開始部署 Azure Stack 之前，請先驗證 Azure 身分識別解決方案。  

整備檢查程式會驗證：
 - Azure Active Directory (Azure AD) 為 Azure Stack 的識別提供者。
 - 您打算使用的 Azure AD 帳戶可以透過 Azure Active Directory 的全域管理員身分登入。 

驗證可確保環境已準備好讓 Azure Stack 在 Azure AD 中儲存 Azure Stack 使用者、應用程式、群組和服務主體的相關資訊。

## <a name="get-the-readiness-checker-tool"></a>取得整備檢查程式工具
從 [PSGallery](https://aka.ms/AzsReadinessChecker) 下載最新版的 Azure Stack 整備檢查程式工具 (AzsReadinessChecker)。  

## <a name="prerequisites"></a>先決條件
必須具備下列先決條件。

**執行這個工具的電腦：**
 - Windows 10 或 Windows Server 2016，具有網際網路連線能力。
 - PowerShell 5.1 或更新版本。 若要檢查版本，請執行下列 PowerShell 命令，然後再檢閱「主要」版本和「次要」版本：  

   > `$PSVersionTable.PSVersion`
 - 設定[適用於 Azure Stack 的 PowerShell](azure-stack-powershell-install.md)。 
 - 最新版的 [Microsoft Azure Stack 整備檢查程式](https://aka.ms/AzsReadinessChecker)工具。

**Azure Active Directory 環境：**
 - 識別您將用於 Azure Stack 的 Azure AD 帳戶，並確保它是 Azure Active Directory 全域管理員。
 - 識別 Azure AD 租用戶名稱。 租用戶名稱必須是 Azure Active Directory 的「主要」網域名稱。 例如，contoso.onmicrosoft.com。 
 - 識別您會使用的 AzureEnvironment：AzureCloud、AzureGermanCloud 或 AzureChinaCloud。

## <a name="validate-azure-identity"></a>驗證 Azure 身分識別 
1. 在符合先決條件的電腦上，開啟系統管理 PowerShell 提示字元，然後執行下列命令以安裝 AzsReadinessChecker：  

   > `Install-Module Microsoft.AzureStack.ReadinessChecker -Force`

2. 從 PowerShell 提示字元中，執行下列命令以將 $serviceAdminCredential 設定為 Azure AD 租用戶的服務管理員。  將 serviceadmin@contoso.onmicrosoft.com 取代為您的帳戶和租用戶。 
   > `$serviceAdminCredential = Get-Credential serviceadmin@contoso.onmicrosoft.com -Message "Enter Credentials for Service Administrator of Azure Active Directory Tenant"` 

3. 從 PowerShell 提示字元中，執行下列命令以開始驗證 Azure AD。 
   - 將 AzureEnvironment 的值指定為 AzureCloud、AzureGermanCloud 或 AzureChinaCloud。  
   - 指定 Azure Active Directory 租用戶名稱來取代 contoso.onmicrosoft.com。 

   > `Start-AzsReadinessChecker -AADServiceAdministrator $serviceAdminCredential -AzureEnvironment AzureCloud -AADDirectoryTenantName contoso.onmicrosoft.com`
4. 在此工具執行之後，請檢閱輸出。 確認登入和安裝需求的狀態都是**正常**。 驗證成功時會出現類似下圖的輸出： 
 
![run-validation](./media/azure-stack-validate-identity/validation.png)


## <a name="report-and-log-file"></a>報告與記錄檔
每當驗證執行時，它會將結果記錄到 **AzsReadinessChecker.log** 和 **AzsReadinessCheckerReport.json**。 PowerShell 中的驗證結果會一同顯示這些檔案的位置。

這些檔案可協助您在部署 Azure Stack 或調查驗證問題之前共用驗證狀態。  這兩個檔案會保存每個後續驗證檢查的結果。 此報告會向部署團隊提供身分識別設定的確認。 記錄檔可協助部署或支援小組調查驗證問題。 

根據預設，這兩個檔案都會寫入到 C:\Users\<使用者名稱>\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json。  
 - 使用執行命令列結尾的 **-OutputPath** ***&lt;path&gt;*** 參數來指定不同的報告位置。   
 - 使用執行命令結尾的 **-CleanReport** 參數，從 AzsReadinessCheckerReport.json 清除資訊。  關於此工具先前的執行。 

如需詳細資訊，請參閱 [Azure Stack 驗證報告](azure-stack-validation-report.md)。

## <a name="validation-failures"></a>驗證失敗
如果驗證檢查失敗，則會在 PowerShell 視窗中顯示有關失敗的詳細資料。 工具也會將相關資訊記錄至 AzsReadinessChecker.log。

下列範例會提供關於一般驗證失敗的指引。

### <a name="expired-or-temporary-password"></a>過期或暫時的密碼 
 
![密碼已過期](./media/azure-stack-validate-identity/expired-password.png)
**原因** - 帳戶無法登入，因為密碼已過期，或屬於暫時密碼。     

**解決方式** - 在 PowerShell 中執行下列命令，然後遵循提示來重設密碼。  
> `Login-AzureRMAccount`

或者，以帳戶身分登入 https://portal.azure.com，系統將會強制使用者變更密碼。
### <a name="unknown-user-type"></a>不明的使用者類型 
 
![未知的使用者](./media/azure-stack-validate-identity/unknown-user.png)
**原因** - 帳戶無法登入指定的 Azure Active Directory (AADDirectoryTenantName)。 在此範例中，AzureChinaCloud 會指定為 AzureEnvironment。

**解決方式** - 確認此帳戶對指定的 Azure 環境是有效的。 在 PowerShell 中，執行下列命令來確認此帳戶對特定環境是有效的：Login-AzureRmAccount – EnvironmentName AzureChinaCloud 
### <a name="account-is-not-an-administrator"></a>帳戶不是系統管理員 
 
![不是系統管理員](./media/azure-stack-validate-identity/not-admin.png)

**原因** - 雖然帳戶可以成功登入，但該帳戶不是 Azure Active Directory (AADDirectoryTenantName) 的系統管理員。  

**解決方式** - 以帳戶身分登入 https://portal.azure.com、移至 [Azure Active Directory] > [使用者] > [選取使用者] > [目錄角色]，然後確定使用者是**全域管理員**。  如果帳戶是使用者，請移至 [Azure Active Directory] > [自訂網域] 名稱，並確認您提供給 AADDirectoryTenantName 的名稱已標示為這個目錄的主要網域名稱。  在此範例中，這就是 contoso.onmicrosoft.com。 

Azure Stack 會要求網域名稱必須是主要網域名稱。

## <a name="next-steps"></a>後續步驟
[驗證 Azure 註冊](azure-stack-validate-registration.md)  
[檢視整備報告](azure-stack-validation-report.md)  
[一般 Azure Stack 整合考量](azure-stack-datacenter-integration.md)  

