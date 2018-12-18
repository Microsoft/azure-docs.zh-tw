---
title: 使用 Windows VM 系統指派的受控識別來存取 Azure Key Vault
description: 本教學課程會逐步引導您使用 Windows VM 系統指派的受控識別，以存取 Azure Key Vault。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: daveba
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/20/2017
ms.author: daveba
ms.openlocfilehash: fffa77990b0af3c710a60d2077962257ab50d5e7
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44163341"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-key-vault"></a>教學課程：使用 Windows VM 系統指派的受控識別來存取 Azure Key Vault 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

本教學課程說明如何將系統指派的受控識別用於 Windows 虛擬機器 (VM)，以存取 Azure Key Vault。 作為啟動程序，金鑰保存庫可讓您的用戶端應用程式，接著使用密碼存取未受 Azure Active Directory (AD) 保護的資源。 受控服務身分識別由 Azure 自動管理，並可讓您驗證支援 Azure AD 驗證的服務，而不需要將認證插入程式碼中。 

您會了解如何：


> [!div class="checklist"]
> * 授與 VM 存取權以取得 Key Vault 中的密碼 
> * 使用 VM 身分識別取得存取權杖，並使用它來擷取 Key Vault 的祕密 

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

- [登入 Azure 入口網站](https://portal.azure.com)

- [建立 Windows 虛擬機器](/azure/virtual-machines/windows/quick-create-portal)

- [在虛擬機器上啟用系統指派的受控識別](/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm#enable-system-assigned-identity-on-an-existing-vm)

## <a name="grant-your-vm-access-to-a-secret-stored-in-a-key-vault"></a>授與 VM 存取權以取得 Key Vault 中的密碼 
 
您的程式碼可以使用適用於 Azure 資源的受控識別來取得存取權杖，以向支援 Azure AD 驗證的資源進行驗證。  但是，並非所有 Azure 服務都支援 Azure AD 驗證。 若要使用適用於 Azure 資源的受控識別搭配那些服務，請將服務認證儲存在 Azure Key Vault 中，並使用 VM 的受控識別來存取 Key Vault，以擷取認證。 

首先，我們需要建立 Key Vault，並將存取 Key Vault 的權利授予 VM 系統指派的受控識別。   

1. 在左側導覽列的頂端，依序選取 [建立資源]  >  [安全性 + 身分識別]  >  [Key Vault]。  
2. 提供新 Key Vault 的**名稱**。 
3. 在與 VM (稍早建立) 相同的訂用帳戶和資源群組中找到 Key Vault。 
4. 選取 [存取原則]，然後按一下 [新增]。 
5. 在 [從範本設定] 中，選取 [密碼管理]。 
6. 選擇 [選取主體]，並在 [搜尋] 欄位中輸入您稍早建立的 VM 名稱。  在結果清單中選取 VM，然後按一下 [選取]。 
7. 按一下 [確定] 來完成新增存取原則，和 [確定] 來完成存取原則選取。 
8. 按一下 [建立] 即可完成建立 Key Vault。 

    ![替代映像文字](./media/msi-tutorial-windows-vm-access-nonaad/msi-blade.png)


接下來，將密碼新增至 Key Vault，好讓您稍後可以使用在 VM 中執行的程式碼來擷取密碼： 

1. 選取 [所有資源]，然後尋找並選取您所建立的 Key Vault。 
2. 選取 [密碼]，然後按一下 [新增]。 
3. 從 [上傳選項] 中選取 [手動]。 
4. 輸入密碼的名稱和值。  值可以是任何您想要的項目。 
5. 將啟動日期和到期日期保留空白，並將 [啟用] 設定為 [是]。 
6. 按一下 [建立] 來建立密碼。 
 
## <a name="get-an-access-token-using-the-vm-identity-and-use-it-to-retrieve-the-secret-from-the-key-vault"></a>使用 VM 身分識別取得存取權杖，並使用它來擷取 Key Vault 的密碼  

如果您未安裝 PowerShell 4.3.1 或更新版本，則必須[下載並安裝最新版](https://docs.microsoft.com/powershell/azure/overview)。

首先，我們會使用 VM 系統指派的受控識別來取得存取權杖，以向 Key Vault 進行驗證：
 
1. 在入口網站中，瀏覽至 [虛擬機器] 並移至您的 Windows 虛擬機器，在 [概觀] 中按一下 [連線]。
2. 輸入您建立 **Windows VM** 時新增的**使用者名稱**和**密碼**。  
3. 現在您已經建立虛擬機器的**遠端桌面連線**，請在遠端工作階段中開啟 PowerShell。  
4. 在 PowerShell 中，叫用租用戶上的 Web 要求，以在 VM 的特定連接埠中取得本機主機的權杖。  

    PowerShell 要求：
    
    ```powershell
    $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -Method GET -Headers @{Metadata="true"} 
    ```
    
    接下來，擷取完整的回應，它會儲存為 $response 物件中的 JavaScript 物件標記法 (JSON) 格式字串。  
    
    ```powershell
    $content = $response.Content | ConvertFrom-Json 
    ```
    
    再來，從回應中擷取存取權杖。  
    
    ```powershell
    $KeyVaultToken = $content.access_token 
    ```
    
    最後，使用 PowerShell 的 Invoke-WebRequest 命令來擷取您稍早在 Key Vault 中建立的祕密，在授權標頭中傳遞存取權杖。  您會需要 Key Vault 的 URL，其位於 Key Vault [概觀] 頁面的 [基本資訊]區段。  
    
    ```powershell
    (Invoke-WebRequest -Uri https://<your-key-vault-URL>/secrets/<secret-name>?api-version=2016-10-01 -Method GET -Headers @{Authorization="Bearer $KeyVaultToken"}).content 
    ```
    
    回應如下所示： 
    
    ```powershell
    {"value":"p@ssw0rd!","id":"https://mytestkeyvault.vault.azure.net/secrets/MyTestSecret/7c2204c6093c4d859bc5b9eff8f29050","attributes":{"enabled":true,"created":1505088747,"updated":1505088747,"recoveryLevel":"Purgeable"}} 
    ```
    
一旦您已從 Key Vault 擷取密碼，您便可以將其用來向需要名稱和密碼的服務進行驗證。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 Windows VM 系統指派的受控識別，來存取 Azure Key Vault。  若要深入了解 Azure Key Vault，請參閱：

> [!div class="nextstepaction"]
>[Azure 金鑰保存庫](/azure/key-vault/key-vault-whatis)
