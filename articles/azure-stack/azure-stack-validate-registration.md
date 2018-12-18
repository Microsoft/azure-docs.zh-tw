---
title: 驗證 Azure Stack 的 Azure 註冊 | Microsoft Docs
description: 使用 Azure Stack 整備檢查程式來驗證 Azure 註冊。
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
ms.date: 06/08/2018
ms.author: brenduns
ms.reviewer: ''
ms.openlocfilehash: 57869de8a99c65810da0c75f81c75d93eac88412
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47090811"
---
# <a name="validate-azure-registration"></a>驗證 Azure 註冊 
使用 Azure Stack 整備檢查程式工具 (AzsReadinessChecker) 來驗證 Azure 訂用帳戶是否已準備好與 Azure Stack 搭配使用。 開始部署 Azure Stack 之前，請先驗證註冊。 整備檢查程式會驗證：
- 您使用的 Azure 訂用帳戶是否為支援的類型。 訂用帳戶必須是雲端服務提供者 (CSP) 或 Enterprise 合約 (EA)。 
- 您用來向 Azure 註冊訂用帳戶的帳戶可以登入 Azure，且為訂用帳戶的擁有者。 

如需 Azure Stack 註冊的詳細資訊，請參閱[向 Azure 註冊 Azure Stack](azure-stack-registration.md)。 

## <a name="get-the-readiness-checker-tool"></a>取得整備檢查程式工具
下載最新版的 Azure Stack 整備檢查程式工具 (AzsReadinessChecker)，其位於 [PSGallery](https://aka.ms/AzsReadinessChecker)。  

## <a name="prerequisites"></a>必要條件
必須具備下列先決條件。

**執行這個工具的電腦：**
 - Windows 10 或 Windows Server 2016，具有網際網路連線能力。
 - PowerShell 5.1 或更新版本。 若要檢查版本，請執行下列 PowerShell 命令，然後再檢閱「主要」版本和「次要」版本：  

    >`$PSVersionTable.PSVersion` 
 - 設定[適用於 Azure Stack 的 PowerShell](azure-stack-powershell-install.md)。 
 - 下載最新版的 [Microsoft Azure Stack 整備檢查程式](https://aka.ms/AzsReadinessChecker)工具。  

**Azure Active Directory 環境：**
 - 識別會與 Azure Stack 搭配使用的 Azure 訂用帳戶擁有者的帳戶使用者名稱和密碼。  
 - 識別您會使用的 Azure 訂用帳戶的訂用帳戶識別碼。 
 - 識別您會使用的 AzureEnvironment：AzureCloud、AzureGermanCloud 或 AzureChinaCloud。

## <a name="validate-azure-registration"></a>驗證 Azure 註冊
1. 在符合先決條件的電腦上，開啟系統管理 PowerShell 提示字元，然後執行下列命令以安裝 AzsReadinessChecker。
    > `Install-Module Microsoft.AzureStack.ReadinessChecker -Force`

2. 從 PowerShell 提示字元中，執行下列命令以將 $registrationCredential 設定為訂用帳戶擁有者的帳戶。   將 subscriptionowner@contoso.onmicrosoft.com 取代為您的帳戶和租用戶。 
    > `$registrationCredential = Get-Credential subscriptionowner@contoso.onmicrosoft.com -Message "Enter Credentials for Subscription Owner"`

3. 從 PowerShell 提示字元中，執行下列命令以將 $subscriptionID 設定為您會使用的 Azure 訂用帳戶。 將 xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx 取代為您自己的訂用帳戶識別碼。  
     > `$subscriptionID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"` 

4. 從 PowerShell 提示字元中，執行下列命令以開始驗證訂用帳戶 
   - 將 AzureEnvironment 的值指定為 AzureCloud、AzureGermanCloud 或 AzureChinaCloud。  
   - 提供 Azure Active Directory 系統管理員和 Azure Active Directory 租用戶名稱。 

   > `Start-AzsReadinessChecker -RegistrationAccount $registrationCredential -AzureEnvironment AzureCloud -RegistrationSubscriptionID $subscriptionID`

5. 在此工具執行之後，請檢閱輸出。 確認登入和註冊需求的狀態都是正常。 驗證成功時會出現類似下圖的輸出：  
![run-validation](./media/azure-stack-validate-registration/registration-validation.png)


## <a name="report-and-log-file"></a>報告與記錄檔
每當驗證執行時，它會將結果記錄到 **AzsReadinessChecker.log** 和 **AzsReadinessCheckerReport.json**。 PowerShell 中的驗證結果會一同顯示這些檔案的位置。 

這些檔案可協助您在部署 Azure Stack 或調查驗證問題之前共用驗證狀態。 這兩個檔案會保存每個後續驗證檢查的結果。 此報告會向部署團隊提供身分識別設定的確認。 記錄檔可協助部署或支援小組調查驗證問題。 

根據預設，這兩個檔案都會寫入到 C:\Users\<使用者名稱>\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json。  
 - 使用執行命令列結尾的 **-OutputPath** ***&lt;path&gt;*** 參數來指定不同的報告位置。   
 - 使用執行命令結尾的 **-CleanReport** 參數，從 AzsReadinessCheckerReport.json 清除資訊。  關於此工具先前的執行。 如需詳細資訊，請參閱 [Azure Stack 驗證報告](azure-stack-validation-report.md)。

## <a name="validation-failures"></a>驗證失敗
如果驗證檢查失敗，則會在 PowerShell 視窗中顯示有關失敗的詳細資料。 工具也會將相關資訊記錄至 AzsReadinessChecker.log。

下列範例會提供關於一般驗證失敗的指引。

### <a name="user-must-be-an-owner-of-the-subscription"></a>使用者必須是訂用帳戶的擁有者   
![subscription-owner](./media/azure-stack-validate-registration/subscription-owner.png)
**原因** - 帳戶不是 Azure 訂用帳戶的系統管理員。   

**解決方式** - 使用屬於 Azure 訂用帳戶系統管理員的帳戶，以作為 Azure Stack 部署的使用量計費帳戶。


### <a name="expired-or-temporary-password"></a>過期或暫時的密碼 
![expired-password](./media/azure-stack-validate-registration/expired-password.png)
**原因** - 帳戶無法登入，因為密碼已過期，或屬於暫時密碼。     

**解決方式** - 在 PowerShell 中執行並遵循提示來重設密碼。 
  > `Login-AzureRMAccount` 

或者，以帳戶身分登入 https://portal.azure.com，系統將會強制使用者變更密碼。


### <a name="microsoft-accounts-are-not-supported-for-registration"></a>不支援註冊的 Microsoft 帳戶  
![unsupported-account](./media/azure-stack-validate-registration/unsupported-account.png)
**原因** - 指定了 Microsoft 帳戶 (例如 Outlook.com 或 Hotmail.com)。  系統不支援這些帳戶。

**解決方式** - 使用雲端服務提供者 (CSP) 或 Enterprise 合約 (EA) 中的帳戶和訂用帳戶。 


### <a name="unknown-user-type"></a>不明的使用者類型  
![unknown-user](./media/azure-stack-validate-registration/unknown-user.png)
**原因** - 帳戶無法登入指定的 Azure Active Directory 環境。 在此範例中，AzureChinaCloud 會指定為 AzureEnvironment。  

**解決方式** - 確認此帳戶對指定的 Azure 環境是有效的。 在 PowerShell 中，執行下列命令來確認此帳戶對特定環境是有效的。     
  > `Login-AzureRmAccount -EnvironmentName AzureChinaCloud`


## <a name="next-steps"></a>後續步驟
[驗證 Azure 身分識別](azure-stack-validate-identity.md)
[檢視整備報告](azure-stack-validation-report.md)
[一般 Azure Stack 整合考量](azure-stack-datacenter-integration.md)

