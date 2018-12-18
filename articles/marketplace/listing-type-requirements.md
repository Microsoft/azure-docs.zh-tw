---
title: 依清單類型的需求 | Azure
description: 本文說明合作夥伴若要了解如何發佈應用程式到 Azure Marketplace 的資格條件和發佈需求。
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
ms.date: 06/05/2018
ms.author: ellacroi
ms.openlocfilehash: 07a62dfa2d7e1c71daf547c5aa7c8c7d15830bfd
ms.sourcegitcommit: 638599eb548e41f341c54e14b29480ab02655db1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/21/2018
ms.locfileid: "36309311"
---
# <a name="requirements-by-listing-type"></a>依清單類型別需求  
技術與行銷內容需求會因店面、供應項目類型和清單類型而異。 請檢閱以下規格，檢查您的合規性。  
1. 店面需求：  
    *   [AppSource](#storefront-requirements-appSource)  
    *   [Azure Marketplace](#storefront-requirements-azure-marketplace)  
2. 清單類型及供應項目類型需求：  
    *   如需有關清單類型和供應項目類型的詳細資訊，請瀏覽 [判斷您解決方案的清單類型] 網頁：[docs.microsoft.com/azure/marketplace/determine-your-listing-type](./determine-your-listing-type.md)。  

## <a name="storefront-requirements-appsource"></a>店面需求：AppSource  
下表說明在 AppSource 發行的必要條件需求。  
| 需求 | 詳細資料 | 必要或建議 |  
|:--- |:--- |:--- |  
| ***Azure Active Directory (Azure AD)*** | 您的應用程式必須允許 Azure Active Directory 同盟單一登入 (Azure AD 同盟 SSO) 並同意啟用。<ul> <li>如需有關啟用 Azure AD 同盟 SSO 的詳細資訊，請瀏覽 [設定不在 Azure Active Directory 應用程式庫中的應用程式的單一登入] 網頁：[docs.microsoft.com/azure/active-directory/active-directory-saas-custom-apps](https://docs.microsoft.com/azure/active-directory/active-directory-saas-custom-apps)。</li> </ul> | 必要 |   
| ***與 Microsoft Cloud 服務整合*** | 您的應用程式應與 Microsoft Power BI、Cortana Intelligence、Microsoft Azure 服務等其他 Microsoft Cloud 服務整合。<ul> <li>Microsoft 雲端服務的範例為物聯網。</li> </ul> | 建議 |  
| ***目標對象*** | 您的應用程式必須以企業營運使用者與企業擁有者為目標。 | 必要 | 
| ***商用軟體即服務 (SaaS) 應用程式*** | 您的應用程式必須符合下列需求。<ul> <li>企業營運 SaaS 應用程式</li> <li>著重於業務流程</li> <li>以商務客戶為目標</li> <li>讓使用者能夠透過其公司認證 (使用者名稱與密碼) 登入</li> </ul> | 必要 |  
| ***免費試用期與試用版體驗*** | 您的應用程式必須包含下列其中一個選項，客戶才能在一段有限時間內免費使用您的應用程式。<ul> <li>提供 `try` 方法，客戶才能在 AppSource 內啟動您應用程式的試用版</li> <li>在 AppSource 提供 `request trial` 選項，客戶才能要求使用您應用程式的試用版</li> </ul>您提供的免費試用版必須讓客戶能在預先設定的期間內試用應用程式，無須支付額外費用。 | 必要 |  
| ***輕鬆設定、立即可用的解決方案*** | 應用程式的安裝與設定必須簡單快速而且無須自訂。 | 必要 |  
| ***潛在客戶管理*** | 獲得來自店面的潛在客戶之前，您必須啟用 CRM 以接受潛在客戶資料。<ul> <li>CRM 的範例包括 Marketo、Microsoft Dynamics 或 Salesforce</li> </ul> | 必要 |  
| ***隱私權原則及使用規定*** | 您的應用程式必須提供公用 URL，連結至您的隱私權原則網頁。 您的使用條款必須在發行期間以文字形式提供。 | 必要 |  
| ***支援*** | 您的應用程式必須使用公用 URL，提供前往您的客戶支援網頁的連結。 如果您的應用程式是試用版，您必須在試用期間提供支援，而且不收取額外費用。 | 必要 |  

## <a name="storefront-requirements-azure-marketplace"></a>店面需求：Azure Marketplace  
以下是 Azure Marketplace 中清單類型的必要條件需求。  
| 需求 | 詳細資料 | 清單類型 |  
|:--- |:--- |:--- |  
| ***參與原則*** | 您的應用程式必須遵守 Azure Marketplace 參與原則。<ul> <li>如需參與原則的相關詳細資訊，請參閱 Azure Marketplace 參與原則網頁：[azure.microsoft.com/support/legal/marketplace/participation-policies](https://azure.microsoft.com/support/legal/marketplace/participation-policies)。</li></ul> | list<br />交易<br />試用版 |  
| ***與 Microsoft 整合*** | 您的供應項目應使用或擴充 Microsoft Azure 服務類型，例如計算、網路或儲存體。 您的供應項目也應符合現有的 Azure Marketplace 類別，例如資料庫、安全性或網路功能。<ul> <li>如需 Marketplace 供應項目的詳細資訊，請瀏覽 Marketplace 應用程式網頁：[azuremarketplace.microsoft.com/marketplace/apps](https://azuremarketplace.microsoft.com/marketplace/apps)。</li> </ul> | list<br />交易<br />試用版 |  
| ***目標對象*** | 您的供應項目必須專為 IT 專業人員、雲端開發人員或其他技術客戶角色提供。 | list<br />交易<br />試用版 |  
| ***潛在客戶管理*** | 獲得來自店面的潛在客戶之前，您必須啟用 CRM (Marketo、Microsoft Dynamics 或 Salesforce)，以接受潛在客戶資料。 | list<br />交易<br />試用版 |  
| ***隱私權原則及使用規定*** | 您的應用程式必須提供公用 URL，連結至您的隱私權原則網頁。 您的使用條款必須在發行期間以文字形式提供。 | list<br />交易<br />試用版 |  
| ***支援*** | 您的供應項目必須提供公用 URL，連結至您的客戶支援網頁。 如果您的供應項目是試用版，您必須在試用期間提供支援，且不收取額外費用。 | 交易<br />試用版 |    

## <a name="non-transact-listings"></a>非交易清單  
本章節說明所有不使用交易清單類型的供應項目類型。 

### <a name="list"></a>列出  
「清單」清單類型包括下列在市集店面上的「供應項目」類型。  

| 供應項目類型 | 店面 | 詳細資料 |  
|:---        |:---        |:---     |  
| 諮詢服務 | AppSource | [需求：AppSource：清單：諮詢服務](#requirements-appsource-list-consulting-services) |  
| 諮詢服務 | Azure Marketplace | [需求：Azure Marketplace：清單：諮詢服務](#requirements-azure-marketplace-list-consulting-services) |  
| 與我連絡 | AppSource | [](#) |  
| 與我連絡 | Azure Marketplace | [需求：AppSource：清單：與我連絡](#requirements-azure-marketplace-list-contact-me) |  

#### <a name="requirements-appsource-list-consulting-service"></a>需求：AppSource：清單：諮詢服務  

| 需求 | 詳細資料 |  
|:--- |:--- |  
| 服務供應項目特性 | 您的諮詢服務必須符合下列準則。<ul> <li>以固定範圍、固定期間、固定價格 (或免費) 的形式提供。</li> <li>主要為售前導向。</li> <li>以單一客戶為限。</li> <li>在現場進行。</li> </ul> |  
| 諮詢服務的合作夥伴需求 | 您的服務符合相關領域的準則。<table><tr><th>解決方案領域</th><th>準則</th></tr><tr><td>Dynamics 365 for Customer Engagement</td><td>具備銀級或金級「雲端客戶關係管理」專長認證。</td></tr><tr><td>Dynamics 365 for Finance and Operations，Enterprise edition</td><td>具備銀級或金級「企業資源規劃」專長認證，且過去 12 個月的雲端作業收益達 $25,000 以上。</td></tr><tr><td>Dynamics 365 for Finance and Operations，Business edition</td><td>以「雲端服務提供者」(CSP) 或「數位記錄合作夥伴」(DPOR) 的身分為一或多位客戶提供服務。</td></tr><tr><td>Power BI</td><td>符合「解決方案合作夥伴」準則。</td></tr><tr><td>PowerApps</td><td>具有「合作夥伴展示工具」解決方案。</td></tr></table><ul> <li>如需有關客戶關係管理的詳細資訊，請瀏覽「雲端客戶關係管理」頁面：[partner.microsoft.com/membership/cloud-customer-relationship-management-competency](https://partner.microsoft.com/membership/cloud-customer-relationship-management-competency)。</li> <li>如需有關資源規劃的詳細資訊，請瀏覽「企業資源規劃」頁面：[partner.microsoft.com/membership/enterprise-resource-planning-competency](https://partner.microsoft.com/membership/enterprise-resource-planning-competency)。</li> <li>如需有關 CSP 的詳細資訊，請瀏覽「雲端服務提供者」頁面：[partner.microsoft.com/cloud-solution-provider](https://partner.microsoft.com/cloud-solution-provider)。</li> <li>如需有關 DPOR 的詳細資訊，請瀏覽「數位記錄合作夥伴和合作夥伴關聯」頁面：[partner.microsoft.com/membership/digital-partner-of-record](https://partner.microsoft.com/membership/digital-partner-of-record)。</li> <li>如需有關解決方案合作夥伴準則的詳細資訊，請瀏覽「解決方案合作夥伴概觀和獎勵」文件：[www.microsoftpartnerserverandcloud.com/_layouts/download.aspx?SourceUrl=Hosted%20Documents/Power%20BI%20Program%20Overview%20%26%20Incentives.pdf](https://www.microsoftpartnerserverandcloud.com/_layouts/download.aspx?SourceUrl=Hosted%20Documents/Power%20BI%20Program%20Overview%20%26%20Incentives.pdf) \(英文\)。</li> <li>如需有關合作夥伴展示工具的詳細資訊，請瀏覽「合作夥伴展示工具」頁面：[powerapps.microsoft.com/partner-showcase](https://powerapps.microsoft.com/partner-showcase)。</li> </ul> |  

#### <a name="requirements-azure-marketplace-list-consulting-service"></a>需求：Azure Marketplace：清單：諮詢服務  

| 需求 | 詳細資料 |  
|:--- |:--- |  
| 服務供應項目特性 | 您的諮詢服務必須符合下列準則。<ul> <li>以固定範圍、固定期間、固定價格 (或免費) 的形式提供。</li> <li>主要為售前導向。</li> <li>以單一客戶為限。</li> <li>在現場進行。</li> </ul> |  
| 諮詢服務的合作夥伴需求 | 您必須具備以下其中一項您服務相關領域的銀級或金級專長認證。 <table><tr><th>解決方案領域</th><th>專長認證</th></tr><td>雲端平台與基礎結構</td><td>雲端平台<br />資料中心</td><tr><td>應用程式開發與 ISV</td><td>影片應用程式開發<br />應用程式整合<br />DevOps</td></tr><tr><td>資料管理與分析</td><td>資料分析<br />資料平台</td></tr></table><ul> <li>如需有關專長認證的詳細資訊，請瀏覽「透過 Microsoft 合作夥伴網路取得專長認證」頁面：[partner.microsoft.com/membership/competencies](https://partner.microsoft.com/membership/competencies)。</li> <li>如需有關清單的詳細資訊，請瀏覽「Azure Marketplace 諮詢服務」頁面：[docs.microsoft.com/azure/marketplace/consulting-services](https://docs.microsoft.com/azure/marketplace/consulting-services)。</li></ul> |  

<!-- #### Requirements: Azure Marketplace: List: Contact Me -->

---   

### <a name="trial"></a>試用版  

| 供應項目類型 | 店面 | 詳細資料 |  
|:---        |:---        |:---     |  
| 免費 / SaaS 試用版 | AppSource | [清單類型需求：試用版](#listing-type-requirements-trial) |  
| 免費 / SaaS 試用版 | Azure Marketplace | [需求：Azure Marketplace：試用版：免費試用 / SaaS 試用版](#requirements-azure-marketplace-trial-free-trial-/-saas-trial) |  
| 互動式示範 | AppSource | [清單類型需求：試用版](#listing-type-requirements-trial) |  
| 互動式示範 | Azure Marketplace | [需求：Azure Marketplace：試用版：互動式示範](#requirements-azure-marketplace-trial-interactive-demo) |  
| 試用產品 | AppSource | [清單類型需求：試用版](#listing-type-requirements-trial) |  
| 試用產品 | Azure Marketplace | [需求：Azure Marketplace：試用版：試用產品](#requirements-azure-marketplace-trial-test-drive) |  

#### <a name="requirements-azure-marketplace-trial"></a>需求：Azure Marketplace：試用版  

| 需求 | 詳細資料 |  
|:--- |:--- |  
| 免費試用期與試用版體驗 | 客戶可以在限時內免費使用您的應用程式。<br /><br />客戶不需要支付任何供應項目或應用程式的授權或訂閱費用。 客戶不需要支付基本 Microsoft 第一方產品或服務的費用。 所有試用版選項都會部署到您的 Azure 訂用帳戶。 您可完全控制成本最佳化和管理。<br /><br />您可以選擇免費試用版、互動式示範或試用產品。 無論您的選擇為何，免費試用版都必須讓客戶能在預先設定的期間內試用應用程式，無須支付額外費用。<ul> <li>若要開始建立試用產品的程序，請傳送電子郵件與 [amp-testdrive@microsoft.com](mailto:amp-testdrive@microsoft.com) 聯繫。</li> </ul>注意：Azure Marketplace SaaS 試用版體驗必須允許客戶以其工作認證來登入。<ul> <li>如需詳細資訊，請瀏覽 AppSource 試用體驗章節：[docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-appsource-certified#appsource-trial-experiences](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-appsource-certified#appsource-trial-experiences)。</li> </ul> |  
| 輕鬆設定、立即可用的解決方案 | 應用程式的設定和安全必須既簡單又快速。 |  
| 可用性 / 執行時間 | SaaS 應用程式或平台的執行時間必須至少達到 99.9%。 |  
| Azure Active Directory | 您的供應項目必須允許 Azure Active Directory (Azure AD) 同盟單一登入 (SSO) (Azure AD 同盟 SSO) 並啟用同意功能。 |  

#### <a name="requirements-azure-marketplace-trial-free-trial--saas-trial"></a>需求：Azure Marketplace：試用版：免費試用 / SaaS 試用版  

| 優點 | 需求 |  
|:--- |:--- |  
| 可讓客戶先試用產品，再以自動化方法轉換成付費訂用帳戶。 此外，可向客戶提供概念證明，並與 Microsoft 銷售團隊共同參與。 | 您的解決方案是虛擬機器或解決方案範本。<br /><br />您的解決方案是 SaaS 供應項目，且您提供多租用戶 SaaS 產品。<br /><br />您的首次執行體驗能夠讓客戶快速地啟動並執行。<br /><br />您有單一租用戶，但會將客戶新增為來賓使用者。 |  

#### <a name="requirements-azure-marketplace-trial-interactive-demo"></a>需求：Azure Marketplace：試用版：互動式示範  

| 優點 | 需求 |  
|:--- |:--- |  
| 無須進行複雜的設定，即可讓客戶了解您解決方案的實際運作情況。 | 您的解決方案需要複雜的設定而難以在試用期間內完成。 |  

#### <a name="requirements-azure-marketplace-trial-test-drive"></a>需求：Azure Marketplace：試用版：試用產品  

| 優點 | 需求 |  
|:--- |:--- |  
| 可讓客戶先試用產品再購買。<br /><br />使用預先設定的設定來提供引導式解決方案體驗。<br /><br />以下是使用試用產品時可享有的額外優點。<ul> <li>使用者縮小搜尋範圍在 Marketplace 上的搜尋有 27% 只顯示提供試用產品的供應項目。</li> <li>提供試用產品的供應項目可比未提供的供應項目多產生 38% 的潛在客戶。</li> <li>在 Marketplace 上取得的新客戶有 36% 來自試用產品。</li> <li>試用產品可讓 Microsoft 現場銷售人員更了解您的產品，而協力共同銷售。</li> </ul> | 您的解決方案是使用單一租用戶的虛擬機器、解決方案範本或 SaaS 應用程式，或很複雜而難以佈建。 <br /><br />您無法將試用版轉換成付費供應項目。 |  

---

## <a name="transact-specific-listings"></a>交易特定清單

### <a name="transact"></a>交易  

| 供應項目類型 | 店面 | 詳細資料 |   
|:---        |:---        | :--- |  
| Azure 應用程式：受控應用程式 | Azure Marketplace | [需求：Azure Marketplace：交易：Azure 應用程式：受控應用程式](#requirements-azure-marketplace-transact-azure-apps-managed-app) |  
| Azure 應用程式：解決方案範本 | Azure Marketplace | [需求：Azure Marketplace：交易：Azure 應用程式：解決方案範本](#requirements-azure-marketplace-transact-azure-apps-solution-template) |  
| 容器 | Azure Marketplace | [需求：Azure Marketplace：交易：容器](#requirements-azure-marketplace-transact-container) |  
| SaaS 應用程式  | Azure Marketplace | [需求：Azure Marketplace：交易：SaaS 應用程式](#requirements-azure-marketplace-transact-saas-app) |  
| 虛擬機器 | Azure Marketplace | [需求：Azure Marketplace：交易：虛擬機器](#requirements-azure-marketplace-transact-virtual-machine) |  

<!-- #### Requirements: Azure Marketplace: Transact: Azure apps: Managed app  

#### Requirements: Azure Marketplace: Transact: Azure apps: Solution template   -->

#### <a name="requirements-azure-marketplace-transact-container"></a>需求：Azure Marketplace：交易：容器  

| 需求 | 詳細資料 |  
|:--- |:--- |  
| 計費和計量 | 支援免費或 BYOL 計費模型。 |  
| 以 Docker 為基礎的映像 | 容器映像必須以 Docker 映像格式為基礎，且必須從 Azure 容器登錄中提取。 |  

#### <a name="requirements-azure-marketplace-transact-saas-app"></a>需求：Azure Marketplace：交易：SaaS 應用程式  

| 需求 | 詳細資料 |  
|:--- |:--- |  
| 計費和計量 | 您供應項目的定價為每月均一價。 目前不支援以用量計價及根據用量*調整*的選項。 |  
| 取消 | 客戶可隨時取消您的供應項目。 |  
| 交易登陸頁面 | 裝載 Azure 共同品牌的交易登陸頁面。 登陸頁面可讓客戶建立及管理 SaaS 服務帳戶。 |  
| SaaS 訂用帳戶 API | 提供一項服務，能與 SaaS 訂用帳戶互動，以建立、更新及刪除使用者帳戶與服務方案。 必須在 24 小時內支援所有重大的 API 變更。 任何需定期更新非重大的 API 變更。 |  

#### <a name="requirements-azure-marketplace-transact-virtual-machine"></a>需求：Azure Marketplace：交易：虛擬機器  

| 需求 | 詳細資料 |  
|:--- |:--- | 
| 計費和計量 | 您的 VM 必須支援 BYOL 或每月計費「預付型方案」。 |  
| Azure 相容的虛擬硬碟 (VHD) | VM 必須建置在 Windows 或 Linux 上。<ul> <li>如需有關建立 Linux VHD 的詳細資訊，請瀏覽＜建立與 Azure 相容的 VHD (以 Linux 為基礎)＞一節：[docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#2-create-an-azure-compatible-vhd-linux-based](https://docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#2-create-an-azure-compatible-vhd-linux-based)。</li> <li>如需有關建立 Windows VHD 的詳細資訊，請瀏覽＜建立與 Azure 相容的 VHD (以 Windows 為基礎)＞一節：[docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#3-create-an-azure-compatible-vhd-windows-based](https://docs.microsoft.com/azure/marketplace-publishing/marketplace-publishing-vm-image-creation#3-create-an-azure-compatible-vhd-windows-based)。</li> </ul> |  

## <a name="next-steps"></a>後續步驟
*   請瀏覽 [Azure Marketplace 和 AppSource 發行者指南](./marketplace-publishers-guide.md)網頁。  
 
---  
