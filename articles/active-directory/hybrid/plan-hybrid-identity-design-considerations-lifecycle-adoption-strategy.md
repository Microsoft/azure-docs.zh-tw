---
title: 混合式身分識別設計 - Azure 生命週期採用策略 | Microsoft Docs
description: 協助根據每個生命週期階段可用的選項，定義混合式身分識別管理工作。
documentationcenter: ''
services: active-directory
author: billmath
manager: mtillman
editor: ''
ms.assetid: 420b6046-bd9b-4fce-83b0-72625878ae71
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/30/2018
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: 43aaddc224403f69f90452c024f24eab8c5bdcd2
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46304697"
---
# <a name="determine-hybrid-identity-lifecycle-adoption-strategy"></a>判斷混合式身分識別生命週期採用策略
在這項工作中，您將為混合式身分識別解決方案定義身分識別管理策略，以滿足您在 [判斷混合式身分識別管理工作](plan-hybrid-identity-design-considerations-hybrid-id-management-tasks.md)中定義的商務需求。

若要根據此步驟稍早所呈現的端對端身分識別生命週期，以定義混合式身分識別管理工作，您必須考慮每個生命週期階段可用的選項。

## <a name="access-management-and-provisioning"></a>存取管理和佈建
組織要有良好的帳戶存取管理解決方案，才能在整個組織內準確追蹤誰存取什麼資訊。

存取控制是集中式、單點佈建系統的一項重要功能。 存取控制除了保護機密資訊，還能找出未經授權或不再需要的現有帳戶。 為了控制過時帳戶，佈建系統會將擁有帳戶的使用者的帳戶資訊與授權資訊連結在一起。 授權使用者身分識別資訊通常是在人力資源的資料庫和目錄中維護。

複雜 IT 企業中的帳戶包含數百個參數來定義授權，這些詳細資料可由佈建系統控制。 使用您從授權來源從提供的資料，可以識別新的使用者。 存取要求核准功能會啟始處理程序來核准 (或拒絕) 佈建資源給他們。

| 生命週期管理階段 | 內部部署 | 雲端 | 混合式 |
| --- | --- | --- | --- |
| 帳戶管理和佈建 |您可以利用 Active Directory ® 網域服務 (AD DS) 伺服器角色，建立可擴充、安全且容易管理的基礎結構來管理使用者與資源，並支援 Microsoft® Exchange Server 等具有目錄功能的應用程式。 <br><br> [您可以透過身分識別管理員在 AD DS 中佈建群組](https://technet.microsoft.com/library/ff686261.aspx) <br>[ 您可以在 AD DS 中佈建使用者](https://technet.microsoft.com/library/ff686263.aspx) <br><br> 基於安全性考量，系統管理員可以使用存取控制來管理使用者對共用資源的存取權。 在 Active Directory 中，存取控制的管理方式是在物件層級設定物件的不同存取層級 (或權限)，例如完全控制、寫入、讀取或沒有存取權。 Active Directory 中的存取控制定義不同的使用者如何使用 Active Directory 物件。 根據預設，Active Directory 中的物件權限會設定為最安全的設定。 |您必須為每一個會存取 Microsoft 雲端服務的使用者建立帳戶。 您也可以變更使用者帳戶，或在已不需要時將它們刪除。 根據預設，使用者沒有系統管理員權限，但是您可以選擇性地指派給他們。 <br><br> Azure Active Directory 的其中一項主要功能是管理對資源的存取權。 這些資源可以是目錄的一部分，例如透過目錄中的角色或目錄外部的資源 (例如 SaaS 應用程式、Azure 服務以及 SharePoint 網站或內部部署資源) 管理物件的權限。 <br><br> Azure Active Directory 的存取管理解決方案以安全性群組為核心。 資源擁有者 (或目錄的系統管理員) 可以指派群組，對所擁有的資源提供特定的存取權限。 群組的成員會取得存取權，而資源擁有者可以將管理群組成員清單的權限委派給其他人 – 例如部門經理或服務台系統管理員<br> <br> ＜在 Azure AD 中管理群組＞一節中提供有關透過群組來管理存取權的詳細資訊。 |透過同步處理與同盟將 Active Directory 身分識別延伸至雲端 |

## <a name="role-based-access-control"></a>角色型存取控制
角色型存取控制 (RBAC) 使用角色和佈建原則來評估、測試和強制執行商務程序和規則，以授與存取權給使用者。 金鑰系統管理員建立佈建原則，將使用者指派到角色，並定義這些角色對資源的權限集。 RBAC 擴充身分識別管理解決方案，使用軟體型處理程序並減少佈建程序中的使用者手動互動。
Azure AD RBAC 可讓公司限制個人存取 Azure 入口網站之後可執行的作業數目。 使用 RBAC 來控制存取入口網站時，IT 系統管理員可以利用下列存取管理方法來委派存取：

* **群組型角色指派**：您可以指派存取權給可從本機 Active Directory 同步處理的 Azure AD 群組。 這可讓您運用組織目前在群組管理工具和程序方面所做的投資。 您也可以使用 Azure AD Premium 的委派群組管理功能。
* **運用 Azure 中內建的角色**：您可以使用三個角色 — 擁有者、參與者和讀者，以確保使用者和群組只擁有他們執行工作所需的權限。
* **細微控制資源的存取**：您可以針對特定的訂用帳戶、資源群組或個別 Azure 資源 (例如網站或資料庫)，將角色指派給使用者和群組。 如此一來，您可以確保使用者能夠存取他們需要的資源，但不能存取他們不需要管理的資源。

## <a name="provisioning-and-other-customization-options"></a>佈建和其他自訂選項
您的小組可以根據商務計劃和需求，決定將身分識別解決方案自訂到何種程度。 例如，大型企業可能需要根據時間表制定分段式推展計劃，以開發工作流程和自訂配接器，漸增地佈建跨地理位置廣泛使用的應用程式。 另一個自訂計劃可能是針對兩個以上的應用程式，這些應用程式在成功測試後就要佈建到整個組織。 您可以自訂使用者與應用程式之間的互動，也可以變更資源的佈建程序以支援自動化佈建。

您可以取消佈建以移除服務或元件。 例如，取消佈建帳戶表示從資源中刪除該帳戶。

混合式資源佈建模型結合要求型和角色型方法 (Azure AD 同時支援這兩者)。 在一部分的員工或受控系統中，企業可以透過角色型指派將存取自動化。 企業也可以透過要求型的模型，處理其他所有存取要求或例外狀況。 有些企業剛開始會採用手動指派，然後逐漸演化成混合式模型，目標是在未來達到完全角色型的部署。

基於商業理由，其他公司可能認為達到完全角色型佈建不切實際，而以混合式方法為期望的目標。 還有其他公司只要有要求型佈建就很滿足，不想投入更多心力來定義和管理角色型、自動化佈建原則。

## <a name="license-management"></a>授權管理
Azure AD 中的群組型授權管理可讓系統管理員將使用者指派到安全性群組，Azure AD 會自動指派授權給該群組的所有成員。 如果後來在群組中新增或移除使用者，將會適當地自動指派或移除授權。

您可以使用您從內部部署 AD 同步處理或在 Azure AD 中管理的群組。 再搭配 Azure AD Premium 的自助式群組管理，您就可以輕鬆地將授權指派工作委派給適當的決策人員。 發生授權衝突和遺失位置資料等問題時，保證會自動解決。

## <a name="self-regulating-user-administration"></a>自我控管使用者管理
當組織開始將資源佈建到所有內部組織時，您需要實作自我控管使用者管理功能。 您可以感受到跨組織界限佈建使用者的優點和好處。 在此環境中，使用者狀態的變更會跨組織界限和地理位置，自動反映在存取權限上。 您可以降低佈建成本，並簡化存取和核准程序。 對於組織中的端對端存取管理，此實作可發揮實作角色型存取控制的全部潛能。 您可以透過自動化程序來控管使用者佈建，以降低管理成本。 針對大量使用者的情況，您可以自動強制執行安全性原則，簡化並集中化使用者生命週期管理和資源佈建，以改善安全性。

> [!NOTE]
> 如需詳細資訊，請參閱「設定 Azure AD 實現自助式應用程式存取管理」
> 
> 

以授權為基礎 (以權利為基礎) 的 Azure AD 服務的運作方式，是在您的 Azure AD 目錄/服務租用戶中啟用訂用帳戶。 一旦訂用帳戶啟用，服務功能就可以由目錄/服務系統管理員管理，並由授權的使用者使用。 

## <a name="integration-with-other-3rd-party-providers"></a>」 與其他協力廠商提供者整合

Azure Active Directory 為數千個 SaaS 應用程式和內部部署 Web 應用程式提供單一登入和增強的應用程式存取安全性。 如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](../develop/quickstart-v1-integrate-apps-with-azure-ad.md)

## <a name="define-synchronization-management"></a>定義同步處理管理
將內部部署目錄與 Azure AD 整合可提供一個通用身分識別來存取雲端和內部部署資源，讓使用者變得更有生產力。 透過此整合，使用者和組織可以享受到下列優點：

* 組織可以運用 Windows Server Active Directory，然後連線到 Azure Active Directory，提供跨內部部署或雲端架構服務的通用混合式身分識別給使用者。
* 系統管理員可以根據應用程式資源、裝置和使用者身分識別、網路位置及多因素驗證，提供條件式存取。
* 使用者可以透過 Azure AD 中的帳戶，將通用身分識別運用到 Office 365、Intune、SaaS 應用程式和協力廠商應用程式。
* 開發人員可以利用通用身分識別模型來建置應用程式，將應用程式整合到 Active Directory 內部部署或 Azure，成為雲端架構應用程式

下圖提供身分識別同步處理程序的高階觀點範例。

![Sync](./media/plan-hybrid-identity-design-considerations/identitysync.png)

身分識別同步處理程序

請檢閱下表來比較同步處理選項：

| 同步處理管理選項 | 優點 | 缺點 |
| --- | --- | --- |
| 同步型 (透過 DirSync 或 AADConnect) |從內部部署和雲端同步的使用者和群組 <br>  **原則控制**：系統管理員可以透過 Active Directory 設定帳戶原則，以管理密碼原則、工作站、限制、鎖定控制等，而不必在雲端中執行其他工作。  <br>  **存取控制**：可以限制對雲端服務的存取，以透過公司環境、線上伺服器或兩者來存取服務。 <br>  減少支援電話：如果使用者要記住的密碼越少，就越不會忘記。 <br>  安全性：使用者身分識別和資訊受到保護，因為單一登入中使用的所有伺服器和服務都由內部部署掌控。 <br>  支援增強式驗證：您可以對雲端服務使用增強式驗證 (也稱為雙重要素驗證)。 不過，如果使用增強式驗證，則必須使用單一登入。 | |
| 同盟型 (透過 AD FS) |由 Security Token Service (STS) 啟用。 當您設定 STS 以提供 Microsoft 雲端服務的單一登入存取時，您會在內部部署 STS 與您在 Azure AD 租用戶中指定的同盟網域之間建立同盟信任。 <br> 可讓使用者使用同一組認證來取得多個資源的存取權 <br>使用者不需要維護多組認證。 但是，使用者必須向每個參與的資源提供認證。支援 B2B 和 B2C 案例。 |需要專業人員來部署及維護專用的內部部署 AD FS 伺服器。 如果您打算對 STS 使用 AD FS，使用增強式驗證會受到限制。 如需詳細資訊，請參閱 [設定 AD FS 2.0 的進階選項](http://go.microsoft.com/fwlink/?linkid=235649)。 |

> [!NOTE]
> 如需詳細資訊，請參閱 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
> 
> 

## <a name="see-also"></a>另請參閱
[設計考量概觀](plan-hybrid-identity-design-considerations-overview.md)

