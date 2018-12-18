---
title: Azure Marketplace 和 AppSource 發行者指導方針 | Azure
description: 適用於應用程式與服務發行者的 Azure Marketplace 和 AppSource 指導方針
services: Azure, Marketplace, Compute, Storage, Networking, Blockchain, Security
documentationcenter: ''
author: jm-aditi-ms
manager: pabutler
editor: ''
ms.assetid: ''
ms.service: marketplace
ms.workload: ''
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 06/13/2018
ms.author: ellacroi
ms.openlocfilehash: 4da6f5c4513477d1adbf4d7645a66de112eeab23
ms.sourcegitcommit: 638599eb548e41f341c54e14b29480ab02655db1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/21/2018
ms.locfileid: "36307480"
---
# <a name="guidelines"></a>指導方針  

<!--
## Guidelines for AppSource  
-->
---  

## <a name="guidelines-for-azure-marketplace"></a>適用於 Azure Marketplace 的指導方針  

### <a name="guidelines-for-creating-a-microsoft-id-to-manage-a-marketplace-account"></a>建立 Microsoft ID 以管理 Marketplace 帳戶的指導方針  
如果多個人員需要存取與您用來建立 Marketplace 帳戶相同的 Microsoft ID，您應該遵循下列指導方針，以便建立公司帳戶。 

>[!IMPORTANT]
>若要授權多位使用者存取您的 Microsoft 開發人員中心 (Dev Center) 帳戶，Microsoft 建議您使用 Azure Active Directory (Azure AD) 將角色指派給個別使用者。 每位使用者都必須使用個別 Azure AD 認證進行登入，以存取該帳戶。 請使用註冊至貴公司網域中的電子郵件地址，來建立您的 Microsoft ID；Microsoft 建議不要將電子郵件指派給個人。 例如 `windowsapps@fabrikam.com`。  
>*   如需詳細資訊，請造訪[問題：Azure AD 同盟網域中的 Microsoft ID](#issue-microsoft-id-in-an-azure-ad-federated-domain) 一節。  

*   請盡可能只讓最少數的開發人員能夠存取 Microsoft ID。 
*   設定公司電子郵件通訊群組清單 (DL)，其中包含每一位必須存取您開發人員中心帳戶的使用者。 將 DL 電子郵件地址新增至您的安全性資訊。 DL 可讓清單上的所有員工在要求時接收安全碼，以及管理您 Microsoft ID 的安全性資訊。 如果設定通訊群組清單不可行，則個人電子郵件帳戶的擁有者必須能在出現提示時，存取及共用安全碼。  
    *   例如，將新的安全性資訊新增至 Microsoft ID 時，或是從新裝置存取 Microsoft ID 時，會對擁有者發出提示。  
*   新增重要小組成員可以存取的公司電話號碼 (不需要分機號碼)。  
*   一般情況下，您應該需要開發人員使用受信任的裝置來登入您的開發人員中心帳戶。 所有重要小組成員都應該能存取受信任的裝置。 使用受信任的裝置存取時，如果有人要存取開發人員中心帳戶，會降低傳送安全密碼的需求。  
*   如果您必須授與從非信任的電腦存取開發人員中心帳戶的權限，請將存取權限制為低於五位開發人員。 在理想情況下，您的開發人員應該從共用相同地理和網路位置的電腦存取該帳戶。  
*   請經常檢閱並確認您的安全性資訊。  
    *   若要檢視您的安全性資訊，請瀏覽 [安全性設定] 頁面：[account.live.com/proofs/Manage](https://account.live.com/proofs/Manage)。

開發人員中心帳戶主要應從受信任的電腦進行存取。 因為每個開發人員中心帳戶每週產生的代碼數目有所限制，所以請務必從受信任的電腦存取。 使用受信任的電腦也可獲得最安全且一致的登入體驗。 
*   如需有關其他開發人員中心帳戶指導方針和安全性的詳細資訊，請瀏覽 [開啟開發人員帳戶] 頁面：[docs.microsoft.com/windows/uwp/publish/opening-a-developer-account](https://docs.microsoft.com/windows/uwp/publish/opening-a-developer-account)。 

---  

#### <a name="issue-microsoft-id-in-an-azure-ad-federated-domain"></a>問題：Azure AD 同盟網域中的 Microsoft ID  
貴公司帳戶可透過 Azure Active Directory (Azure AD) 進行同盟。 如果您嘗試使用與 Azure AD 同盟的公司電子郵件地址來建立 Microsoft ID，則會收到錯誤訊息。 如果您收到錯誤訊息，請洽詢您的 IT 團隊，以確認您的帳戶是否透過 Azure AD 同盟。 Azure AD 同盟的電子郵件是已知的問題，Microsoft 已著手進行解決。  
*   如需有關 Azure AD 的詳細資訊，請瀏覽 [Azure Active Directory 文件] 頁面：[docs.microsoft.com/azure/active-directory](https://docs.microsoft.com/azure/active-directory)。

Microsoft 建議的因應措施。 請遵循下列步驟，在 `outlook.com` 網域中建立新的電子郵件地址，然後建立規則來轉送您的通訊。  
1.  請移至 [建立帳戶] 頁面，然後按一下 [取得新的電子郵件地址] 連結。 
    *   若要註冊您的 Microsoft ID，請瀏覽 [建立帳戶] 頁面：[signup.live.com/signup](https://signup.live.com/signup)。  
2.  建立新的電子郵件地址，然後輸入密碼。 隨即建立新的 Microsoft ID 和 `outlook.com` 網域中的電子郵件信箱。 請繼續進行註冊程序，直到完成建立帳戶。  

    >[!IMPORTANT]
    >若要在開發人員中心註冊，您必須使用註冊為 Microsoft ID 的電子郵件地址或通訊群組清單。 Microsoft 建議您使用通訊群組清單，以去除對個人的倚賴。 如果您的電子郵件地址或通訊群組清單尚未註冊，則必須立即註冊。    

    >[!Important]
    >如果您任何的電子郵件地址位於 `Microsoft` 公司網域，則不能用來在開發人員中心進行註冊。  

3.  您使用 Outlook 電子郵件地址建立 Microsoft ID 之後，請登入 Outlook 信箱。 建立電子郵件轉寄規則。 電子郵件轉寄規則應該會將此 Outlook 信箱中收到的所有電子郵件，移至您在自己網域中建立的電子郵件地址，來管理您的 Marketplace 帳戶。  
    *   若要登入 Outlook 信箱，請造訪 Outlook 頁面：[outlook.live.com/owa](https://outlook.live.com/owa)。  
    *   如需轉送規則的詳細資訊，請瀏覽 Outlook Web App 中的使用規則，以自動將訊息轉送到位於 [support.office.com/article/Use-rules-in-Outlook-Web-App-to-automatically-forward-messages-to-another-account-1433e3a0-7fb0-4999-b536-50e05cb67fed](https://support.office.com/article/Use-rules-in-Outlook-Web-App-to-automatically-forward-messages-to-another-account-1433e3a0-7fb0-4999-b536-50e05cb67fed) 的另一個帳戶頁面。  

1.  轉送規則會將 Outlook 電子郵件帳戶中接收到的所有電子郵件及通訊，傳送到註冊至貴公司網域中的電子郵件地址。 您的 `outlook.com` 電子郵件地址必須用來在開發人員中心和 Cloud Partner 入口網站進行驗證。  

## <a name="next-steps"></a>後續步驟
*   請瀏覽 [Azure Marketplace 和 AppSource 發行者指南](./marketplace-publishers-guide.md)網頁。  
 
---  
