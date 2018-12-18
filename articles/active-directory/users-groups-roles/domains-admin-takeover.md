---
title: 系統管理員接管 Azure Active Directory 中非受控目錄或影子租用戶 | Microsoft Docs
description: 如何接管 Azure Active Directory 之非受控目錄 (影子租用戶) 中的 DNS 網域名稱。
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: users-groups-roles
ms.topic: article
ms.workload: identity
ms.date: 04/06/2017
ms.author: curtand
ms.reviewer: elkuzmen
ms.custom: it-pro
ms.openlocfilehash: 52ae7da666acaf234920a7f03afe3766f29a1e85
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39629118"
---
# <a name="take-over-an-unmanaged-directory-as-administrator-in-azure-active-directory"></a>如何以系統管理員身分接管 Azure Active Directory 中非受控目錄
本文說明接管 Azure Active Directory (Azure AD) 之非受控目錄中 DNS 網域名稱的兩種方式。 當自助使用者註冊使用 Azure AD 的雲端服務時，系統會根據其電子郵件網域將其新增至非受控 Azure AD 目錄。 如需有關自助式或「病毒式」服務註冊的詳細資訊，請參閱[什麼是自助式 Azure Active Directory 註冊？](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-self-service-signup)

## <a name="decide-how-you-want-to-take-over-an-unmanaged-directory"></a>決定要如何接管非受控目錄
在管理員接管的過程中，您可以依照[將自訂網域名稱新增到 Azure AD](../fundamentals/add-custom-domain.md) 中所述，證明擁有權。 下一節會更詳細地說明管理員體驗，但其摘要如下：

* 當您執行非受控 Azure 目錄的[「內部」管理員接管](#internal-admin-takeover)時，系統會將您新增為非受控目錄的全域管理員。 系統不會將任何使用者、網域或服務方案移轉至您所管理的任何其他目錄。

* 當您執行非受控 Azure 目錄的[「外部」管理員接管](#external-admin-takeover)時，需將非受控目錄的 DNS 網域名稱新增至受控 Azure 目錄。 當您新增網域名稱時，系統會在受控 Azure 目錄中建立使用者與資源的對應，以便讓使用者繼續存取服務而不中斷。 

## <a name="internal-admin-takeover"></a>內部管理員接管

有些包含 SharePoint 和 OneDrive 的產品 (例如 Office 365) 並不支援外部接管。 如果您的情況與此相符，或如果您是管理員，而想要接管使用自助式註冊之使用者所建立的非受控或「影子」租用戶，則您可以藉由內部管理員接管來執行此操作。

1. 透過以 Power BI 之類的工具進行註冊，在非受控租用戶中建立使用者內容。 為了便於範例說明，這些步驟會假設該路徑。

2. 開啟 [Power BI 網站](https://powerbi.com)，然後選取 [免費開始]。 輸入使用組織網域名稱的使用者帳戶，例如 `admin@fourthcoffee.xyz`。 輸入驗證碼之後，請查看您的電子郵件是否有確認碼。

3. 在來自 Power BI 的確認電子郵件中，選取 [是，這是我]。

4. 使用 Power BI 使用者帳戶來登入 [Office 365 系統管理中心](https://portal.office.com/adminportal/Home)。 您會收到指導您**成為管理員**的訊息，這是已經在非受控租用戶中驗證之網域名稱的管理員。 請選取 [是，我想要成為管理員]。
  
  ![[成為管理員] 的第一個螢幕擷取畫面](./media/domains-admin-takeover/become-admin-first.png)
  
5. 在您的網域名稱登錄器新增 TXT 記錄，以證明您擁有網域名稱 **fourthcoffee.xyz**。 在此範例中為 GoDaddy.com。
  
  ![新增網域名稱的 txt 記錄](./media/domains-admin-takeover/become-admin-txt-record.png)

在您的網域名稱登錄器驗證 DNS TXT 記錄之後，您便可以管理 Azure AD 租用戶。

完成上述步驟之後，您現在便已成為 Office 365 中 Fourth Coffee 租用戶的全域管理員。 若要將網域名稱與您的其他 Azure 服務整合，您可以將它從 Office 365 中移除，然後新增至 Azure 中其他的受控租用戶。

### <a name="adding-the-domain-name-to-a-managed-tenant-in-azure-ad"></a>將網域名稱新增至 Azure AD 中的受控租用戶 

1. 開啟 [Office 365 系統管理中心](https://portal.office.com/adminportal/Home)。
2. 選取 [使用者] 索引標籤，然後使用 *user@fourthcoffeexyz.onmicrosoft.com* 這類未使用自訂網域名稱的名稱來建立新使用者帳戶。 
3. 確定新使用者帳戶具有 Azure AD 租用戶的全域管理員權限。
4. 在「Office 365 系統管理中心」中開啟 [網域] 索引標籤，選取網域名稱，然後選取 [移除]。 
  
  ![從 Office 365 移除網域名稱](./media/domains-admin-takeover/remove-domain-from-o365.png)
  
5. 如果您在 Office 365 中有任何使用者或群組參考已移除的網域名稱，就必須將他們重新命名為 .onmicrosoft.com 網域。 如果您強制刪除網域名稱，系統就會自動將所有使用者重新命名，在此範例中是重新命名為 *user@fourthcoffeexyz.onmicrosoft.com*。
  
6. 使用具備 Azure AD 租用戶全域管理員身分的帳戶來登入 [Azure AD 系統管理中心](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview)。
  
7. 選取 [自訂網域名稱]，然後新增網域名稱。 您將必須輸入 DNS TXT 記錄來驗證網域名稱擁有權。 
  
  ![新增至 Azure AD 的網域](./media/domains-admin-takeover/add-domain-to-azure-ad.png)
  
> [!NOTE]
> 如果將網域名稱移除，任何 Power BI 或 Azure Rights Management 服務使用者只要在 Office 365 租用戶中已獲指派授權，就必須儲存其儀表板。 他們必須使用 *user@fourthcoffeexyz.onmicrosoft.com* 這類使用者名稱而不是 *user@fourthcoffee.xyz* 來進行登入。

## <a name="external-admin-takeover"></a>外部管理員接管

如果您已經使用 Azure 服務或 Office 365 來管理租用戶，您將無法新增自訂網域名稱，如果該網域名稱已經在另一個 Azure AD 租用戶中驗證過。 不過，您可以從 Azure AD 中的受控租用戶，以外部管理員接管的方式，接管非受控租用戶。 一般程序會按照[將自訂網域名稱新增到 Azure AD](../fundamentals/add-custom-domain.md)一文所述。

當您驗證網域名稱的擁有權時，Azure AD 會將網域名稱從非受控租用戶中移除，然後移至您現有的租用戶。 非受控目錄之外部管理員接管所需的 DNS TXT 驗證程序與內部管理員接管相同。 差異在於下列項目也會隨著網域名稱一起移過去：

- 使用者
- 訂用帳戶
- 授權指派

### <a name="support-for-external-admin-takeover"></a>對外部管理員接管的支援
以下是支援外部管理員接管的線上服務：

- Power BI
- Azure Rights Management
- Exchange Online

支援的服務方案包括：

- Power BI Free
- Power BI Pro
- PowerApps Free
- PowerFlow Free
- 個人版 RMS
- Microsoft Stream
- Dynamics 365 免費試用版

針對服務方案中包括 SharePoint、OneDrive 或「商務用 Skype」(例如透過 Office 免費訂用帳戶或 Office Basic SKU) 的所有服務，都不支援外部管理員接管。 您可以選擇性地使用 [**ForceTakeover** 選項](#azure-ad-powershell-cmdlets-for-the-forcetakeover-option)，將網域名稱從非受控租用戶移除，並在所需的租用戶上加以驗證。 此 ForceTakeover 選項不會移動使用者，也不會保留訂用帳戶的存取權。 相反地，此選項只會移動網域名稱。 

#### <a name="more-information-about-rms-for-individuals"></a>個人版 RMS 的詳細資訊

對於[個人版 RMS](/azure/information-protection/rms-for-individuals)，當非受控租用戶與您擁有的租用戶位於相同區域中時，會額外移動自動建立的 [Azure 資訊保護租用戶金鑰](/azure/information-protection/plan-implement-tenant-key)和[預設保護範本](/azure/information-protection/configure-usage-rights#rights-included-in-the-default-templates)並包含網域名稱。 

當非受控租用戶位於不同的區域時，金鑰和範本並不會移動。 例如，非受控租用戶位於歐洲，而您擁有的租用戶位於北美洲。 

雖然個人版 RMS 是針對支援 Azure AD 驗證來開啟受保護內容所設計，但它並不會阻止使用者同時保護內容。 如果使用者是使用個人版 RMS 訂用帳戶來保護內容，且金鑰和範本未移動，則在網域接管之後將無法存取該內容。    

### <a name="azure-ad-powershell-cmdlets-for-the-forcetakeover-option"></a>Azure AD 的 ForceTakeover 選項 PowerShell Cmdlet
您可以在 [PowerShell 範例](#powershell-example)中看到使用這些 Cmdlet。


Cmdlet | 使用量 
------- | -------
`connect-msolservice` | 出現提示時，登入您的受控租用戶。
`get-msoldomain` | 顯示與目前租用戶關聯的網域名稱。
`new-msoldomain –name <domainname>` | 將網域名稱以「未驗證」狀態 (尚未執行任何 DNS 驗證) 新增至租用戶。
`get-msoldomain` | 網域名稱現在包含在與受控租用戶關聯的網域名稱清單中，但其狀態會是 [未驗證]。
`get-msoldomainverificationdns –Domainname <domainname> –Mode DnsTxtRecord` | 提供要放到網域之新 DNS TXT 記錄中的資訊 (MS=xxxxx)。 驗證可能不會立即進行，因為 TXT 記錄需要一些時間傳播，所以請先稍候幾分鐘，再考慮使用 **-ForceTakeover** 選項。 
`confirm-msoldomain –Domainname <domainname> –ForceTakeover Force` | <li>如果您的網域名稱仍然未驗證，就可以著手執行 **-ForceTakeover** 選項。 它會驗證是否已建立 TXT 記錄，然後啟動接管程序。<li>您應該只有在強制執行外部管理員接管時 (例如當非受控租用戶的 Office 365 服務封鎖接管時)，才將 **-ForceTakeover** 選項新增至 Cmdlet。
`get-msoldomain` | 網域清單現在會將網域名稱顯示為 [已驗證]。

### <a name="powershell-example"></a>PowerShell 範例

1. 使用用來回應自助式供應項目的認證來連接至 Azure AD：
  ````
    import-module MSOnline
    $msolcred = get-credential
    
    connect-msolservice -credential $msolcred
  ````
2. 取得網域清單：
  
  ````
    Get-MsolDomain
  ````
3. 執行 Get-MsolDomainVerificationDns Cmdlet 以建立挑戰：
  ````
    Get-MsolDomainVerificationDns –DomainName *your_domain_name* –Mode DnsTxtRecord
  
    For example:
  
    Get-MsolDomainVerificationDns –DomainName contoso.com –Mode DnsTxtRecord
  ````

4. 複製從此命令傳回的值 (挑戰)。 例如︰
  ````
    MS=32DD01B82C05D27151EA9AE93C5890787F0E65D9
  ````
5. 在公用 DNS 命名空間中，建立 DNS txt 記錄，其中包含您在上一個步驟中複製的值。 此記錄的名稱是父系網域的名稱，因此如果您使用 Windows Server 的 DNS 角色來建立此資源記錄，請將記錄名稱留白，而只要將此值貼到文字方塊中即可。
6. 執行 onfirm-MsolDomain Cmdlet 來驗證挑戰：
  
  ````
    Confirm-MsolEmailVerifiedDomain -DomainName *your_domain_name*
  ````
  
  例如︰
  
  ````
    Confirm-MsolEmailVerifiedDomain -DomainName contoso.com
  ````

成功挑戰會讓您回到提示，但不會產生錯誤。

## <a name="next-steps"></a>後續步驟
* [將自訂網域名稱新增到 Azure AD](../fundamentals/add-custom-domain.md)
* [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)
* [Azure PowerShell](/powershell/azure/overview)
* [Azure Cmdlet 參考](/powershell/azure/get-started-azureps)
* [Set-MsolCompanySettings](/powershell/module/msonline/set-msolcompanysettings?view=azureadps-1.0)

<!--Image references-->
[1]: ./media/active-directory-self-service-signup/SelfServiceSignUpControls.png
