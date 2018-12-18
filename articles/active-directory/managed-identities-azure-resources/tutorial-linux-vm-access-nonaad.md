---
title: 使用 VM 系統指派的受控識別來存取 Azure Key Vault
description: 本教學課程會逐步引導您使用 Linux VM 系統指派的受控識別，以存取 Azure Resource Manager。
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
ms.openlocfilehash: 274005d7cea5833f9653038b710a7dbc248f0c04
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47221514"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-key-vault"></a>教學課程：使用 VM 系統指派的受控識別來存取 Azure Key Vault 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

本教學課程說明如何將系統指派的受控識別用於 Linux 虛擬機器 (VM)，以存取 Azure Key Vault。 作為啟動程序，金鑰保存庫可讓您的用戶端應用程式，接著使用密碼存取未受 Azure Active Directory (AD) 保護的資源。 適用於 Azure 資源的受控識別會自動由 Azure 管理，可讓您向支援 Azure AD 驗證的服務進行驗證，而不需要將認證插入程式碼中。 

您會了解如何：

> [!div class="checklist"]
> * 授與 VM 存取權以取得 Key Vault 中的密碼 
> * 使用 VM 身分識別取得存取權杖，並使用它來擷取 Key Vault 的密碼 
 
## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

- [登入 Azure 入口網站](https://portal.azure.com)

- [建立 Linux 虛擬機器](/azure/virtual-machines/linux/quick-create-portal)

- [在虛擬機器上啟用系統指派的受控識別](/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm#enable-system-assigned-identity-on-an-existing-vm)

## <a name="grant-your-vm-access-to-a-secret-stored-in-a-key-vault"></a>授與 VM 存取權以取得 Key Vault 中的密碼  

您的程式碼可以使用適用於 Azure 資源的受控服務識別來取得存取權杖，向支援 Azure Active Directory 驗證的資源進行驗證。 但是，並非所有 Azure 服務都支援 Azure AD 驗證。 若要使用適用於 Azure 資源的受控識別搭配那些服務，請將服務認證儲存在 Azure Key Vault 中，並使用適用於 Azure 資源的受控識別來存取 Key Vault，以擷取認證。 

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
 
## <a name="get-an-access-token-using-the-vms-identity-and-use-it-to-retrieve-the-secret-from-the-key-vault"></a>使用 VM 身分識別取得存取權杖，並使用它來擷取 Key Vault 的密碼  

若要完成這些步驟，您需要 SSH 用戶端。  如果您使用 Windows，您可以在[適用於 Linux 的 Windows 子系統](https://msdn.microsoft.com/commandline/wsl/about)中使用 SSH 用戶端。 如果您需要設定 SSH 用戶端金鑰的協助，請參閱[如何在 Azure 上搭配 Windows 使用 SSH 金鑰](../../virtual-machines/linux/ssh-from-windows.md)，或[如何在 Azure 中建立和使用 Linux VM 的 SSH 公開和私密金鑰組](../../virtual-machines/linux/mac-create-ssh-keys.md)。
 
1. 在入口網站中，瀏覽至 [Linux VM]，並在 [概觀] 中按一下 [連線]。 
2. 使用您所選擇的 SSH 用戶端來**連線**到 VM。 
3. 在終端機視窗中，使用 CURL 向 Azure 資源端點的本機受控識別提出要求，以取得 Azure Key Vault 的存取權杖。  
 
    存取權杖的 CURL 要求如下。  
    
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -H Metadata:true  
    ```
    此回應包含您存取 Resource Manager 所需的存取權杖。 
    
    回應：  
    
    ```bash
    {"access_token":"eyJ0eXAi...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://vault.azure.net",
    "token_type":"Bearer"} 
    ```
    
    您可以使用此存取權杖來向 Azure Key Vault 進行驗證。  下一個 CURL 要求會說明如何使用 CURL 和 Key Vault REST API 從 Key Vault 讀取密碼。  您會需要 Key Vault 的 URL，其位於 Key Vault [概觀] 頁面的 [基本資訊] 區段。  您也需要在前一次呼叫取得的存取權杖。 
        
    ```bash
    curl https://<YOUR-KEY-VAULT-URL>/secrets/<secret-name>?api-version=2016-10-01 -H "Authorization: Bearer <ACCESS TOKEN>" 
    ```
    
    回應如下所示： 
    
    ```bash
    {"value":"p@ssw0rd!","id":"https://mytestkeyvault.vault.azure.net/secrets/MyTestSecret/7c2204c6093c4d859bc5b9eff8f29050","attributes":{"enabled":true,"created":1505088747,"updated":1505088747,"recoveryLevel":"Purgeable"}} 
    ```
    
一旦您已從 Key Vault 擷取密碼，您便可以將其用來向需要名稱和密碼的服務進行驗證。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 Linux VM 系統指派的受控識別，來存取 Azure Key Vault。  若要深入了解 Azure Key Vault，請參閱：

> [!div class="nextstepaction"]
>[Azure 金鑰保存庫](/azure/key-vault/key-vault-whatis)




