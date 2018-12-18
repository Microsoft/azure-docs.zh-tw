---
title: Azure AD Connect 和 Azure AD Connect Health 安裝藍圖。 | Microsoft Docs
description: 此文件提供適用於安裝 Azure AD Connect 和 Connect Health 的安裝選項和路徑概觀。
services: active-directory
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/18/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 04d950b0a8001fd76e3754b73a9d7c4c5fe2893d
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49318634"
---
# <a name="azure-ad-connect-and-azure-ad-connect-health-installation-roadmap"></a>Azure AD Connect 和 Azure AD Connect Health 安裝藍圖

## <a name="install-azure-ad-connect"></a>安裝 Azure AD Connect。

> [!IMPORTANT]
> Microsoft 不支援在正式記載的動作以外修改和操作 Azure AD Connect 同步處理。 任何這些動作都可能會導致 Azure AD Connect 同步處理的不一致或不受支援狀態。如此一來，Microsoft 無法提供這類部署的技術支援人員。

您可以在 [Microsoft 下載中心](http://go.microsoft.com/fwlink/?LinkId=615771)下載 Azure AD Connect。

| 解決方法 | 案例 |
| --- | --- |
| 開始之前 - [硬體和必要條件](how-to-connect-install-prerequisites.md) |<li>開始安裝 Azure AD Connect 之前所應完成的步驟。</li> |
| [快速設定](how-to-connect-install-express.md) |<li>如果您有單一樹系 AD，則這是建議使用的選項。</li> <li>使用密碼同步處理以相同的密碼進行使用者登入。</li> |
| [自訂設定](how-to-connect-install-custom.md) |<li>有多個樹系時使用。 支援許多內部部署[拓撲](plan-connect-topologies.md)。</li> <li>自訂您登入的選項，例如傳遞驗證、同盟的 ADFS 或使用第三方識別提供者。</li> <li>自訂同步處理功能，例如篩選和回寫。</li> |
| [從 DirSync 升級](how-to-dirsync-upgrade-get-started.md) |<li>在您有已在執行中的現有 DirSync 伺服器時使用。</li> |
| [從 Azure AD Sync 或 Azure AD Connect 升級](how-to-upgrade-previous-version.md) |<li>視您的偏好而定會有數種不同的方法。</li> |

[安裝後](how-to-connect-post-installation.md) ，您應該確認它是依照預期方式運作，並將授權指派給使用者。

### <a name="next-steps-to-install-azure-ad-connect"></a>安裝 Azure AD Connect 的後續步驟
|話題 |連結|  
| --- | --- |
|下載 Azure AD Connect | [下載 Azure AD Connect](http://go.microsoft.com/fwlink/?LinkId=615771)|
|使用快速設定進行安裝 | [快速安裝 Azure AD Connect](./how-to-connect-install-express.md)|
|使用自訂設定進行安裝 | [自訂 Azure AD Connect 安裝](./how-to-connect-install-custom.md)|
|從 DirSync 升級 | [從 Azure AD Sync 工具 (DirSync) 升級](./how-to-dirsync-upgrade-get-started.md)|
|安裝後 | [驗證安裝和指派授權 ](how-to-connect-post-installation.md)|

### <a name="learn-more-about-install-azure-ad-connect"></a>深入了解安裝 Azure AD Connect
您也想要準備 [操作](how-to-connect-sync-operations.md) 考量。 您可以準備一個待命伺服器，以便在[災害](how-to-connect-sync-operations.md#disaster-recovery)發生時輕鬆地進行容錯移轉。 如果您打算進行頻繁的組態變更，則應該規劃 [預備模式](how-to-connect-sync-operations.md#staging-mode) 伺服器。

|話題 |連結|  
| --- | --- |
|支援的拓撲 | [Azure AD Connect 的拓撲](plan-connect-topologies.md)|
|設計概念 | [Azure AD Connect 的設計概念](plan-connect-design-concepts.md)|
|用於安裝的帳戶 | [進一步了解 Azure AD Connect 認證和權限](reference-connect-accounts-permissions.md)|
|作業規劃 | [Azure AD Connect 同步處理：作業工作和考量](how-to-connect-sync-operations.md)|
|使用者登入選項 | [Azure AD Connect 使用者登入選項](plan-connect-user-signin.md)|

## <a name="configure-sync-features"></a>設定同步處理功能
Azure AD Connect 隨附數個您可以選擇性地開啟或預設為啟用的功能。 在特定案例與拓撲中，有些功能有時可能需要進行其他設定。

[篩選](how-to-connect-sync-configure-filtering.md) 是在您想要限制 Azure AD 可同步處理哪些物件時使用。 預設會同步處理所有使用者、連絡人、群組和 Windows 10 電腦。 您可以根據網域、OU 或屬性變更篩選。

[密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)會將 Active Directory 中的密碼雜湊同步處理至 Azure AD。 使用者可在內部部署與雲端中使用相同的密碼，但只能在一個地方管理此密碼。 因為它使用您的內部部署 Active Directory 做為授權單位，所以您也可以使用自己的密碼原則。

[密碼回寫](../authentication/quickstart-sspr.md) 將可讓您的使用者在雲端中變更和重設其密碼，並套用您的內部部署密碼原則。

[裝置回寫](how-to-connect-device-writeback.md) 將可讓 Azure AD 中註冊的裝置寫回至內部部署 Active Directory，以便可以使用該裝置進行條件式存取。

[防止意外刪除](how-to-connect-sync-feature-prevent-accidental-deletes.md) 功能預設為開啟，它會保護您的雲端目錄，避免同時進行多次刪除。 根據預設，它每回允許 500 次刪除。 您可以根據您組織的大小來變更此設定。

[自動升級](how-to-connect-install-automatic-upgrade.md) ，而且會確保您的 Azure AD Connect 永遠保持最新版本。

### <a name="next-steps-to-configure-sync-features"></a>設定同步處理功能的後續步驟
|話題 |連結|  
| --- | --- |
|設定篩選 | [Azure AD Connect 同步處理：設定篩選](how-to-connect-sync-configure-filtering.md)|
|密碼雜湊同步處理 | [密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)|
|傳遞驗證 | [傳遞驗證](how-to-connect-pta.md)
|密碼回寫 | [開始使用密碼管理](../authentication/quickstart-sspr.md)|
|裝置回寫 | [在 Azure AD Connect 中啟用裝置回寫](how-to-connect-device-writeback.md)|
|防止意外刪除 | [Azure AD Connect 同步處理：防止意外刪除](how-to-connect-sync-feature-prevent-accidental-deletes.md)|
|自動升級 | [Azure AD Connect：自動升級](how-to-connect-install-automatic-upgrade.md)|

## <a name="customize-azure-ad-connect-sync"></a>自訂 Azure AD Connect 同步處理
Azure AD Connect 同步處理隨附一個適用於大部分客戶和拓撲的預設組態。 但總是會有一些情況，預設組態並不適用，因而必須進行調整。 支援依照本節和連結主題所述進行變更。

如果您之前沒有使用過同步處理拓撲，建議您從 [技術概念](how-to-connect-sync-technical-concepts.md)中了解拓撲的基本概念和術語。 Azure AD Connect 是 MIIS2003、ILM2007 和 FIM2010 的進化。 即使有些東西相同，但改變的卻更多。

[預設組態](concept-azure-ad-connect-sync-default-configuration.md) 會假設組態中可能有多個樹系。 在那些拓撲中，使用者物件可能會表示為另一個樹系中的連絡人。 使用者也可能在另一個資源樹系中具有連結的信箱。 預設組態的行為已載明於 [使用者和連絡人](concept-azure-ad-connect-sync-user-and-contacts.md)中。

同步處理中的組態模型稱為 [宣告式佈建](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)。 進階屬性流程會使用 [函式](reference-connect-sync-functions-reference.md) 來表示屬性轉換。 您可以使用 Azure AD Connect 隨附的工具來查看整個組態。 如果您需要進行組態變更，請務必遵循 [最佳作法](how-to-connect-sync-best-practices-changing-default-configuration.md) ，以便更輕鬆地採用新版本。

### <a name="next-steps-to-customize-azure-ad-connect-sync"></a>自訂 Azure AD Connect 同步處理的後續步驟。
|話題 |連結|  
| --- | --- |
|所有 Azure AD Connect 同步處理文章 | [Azure AD Connect 同步處理](how-to-connect-sync-whatis.md)|
|技術概念 | [Azure AD Connect 同步處理：技術概念](how-to-connect-sync-technical-concepts.md)|
|了解預設組態 | [Azure AD Connect 同步處理：了解預設組態](concept-azure-ad-connect-sync-default-configuration.md)|
|了解使用者和連絡人 | [Azure AD Connect 同步處理：了解使用者和連絡人](concept-azure-ad-connect-sync-user-and-contacts.md)|
|宣告式佈建 | [Azure AD Connect 同步處理：了解宣告式佈建運算式](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)|
|變更預設組態 | [變更預設組態的最佳作法](how-to-connect-sync-best-practices-changing-default-configuration.md)|

## <a name="configure-federation-features"></a>設定同盟功能

Azure AD Connect 提供數種功能，可簡化使用 AD FS 與 Azure AD 的同盟，以及管理您的同盟信任。 Azure AD Connect 在 Windows Server 2012R2 或更新版本上支援 AD FS。

即使您未使用 Azure AD Connect 來管理您的同盟信任，請[更新 AD FS 伺服器陣列的 SSL 憑證](how-to-connect-fed-ssl-update.md)。

[新增 AD FS 伺服器](how-to-connect-fed-management.md#addadfsserver)至伺服器陣列，以視需要擴大伺服器陣列。

輕鬆按幾下即可使用 Azure AD [修復信任](how-to-connect-fed-management.md#repairthetrust)。

ADFS 可以設定為支援 [多個網域](how-to-connect-install-multiple-domains.md)。 例如，您可以有多個必須用於同盟的最上層網域。

如果您的 ADFS 伺服器尚未設定為自動從 Azure AD 更新憑證，或如果您使用非 ADFS 解決方案，則當您必須 [更新憑證](how-to-connect-fed-o365-certs.md)時您將會收到通知。

### <a name="next-steps-to-configure-federation-features"></a>設定同盟功能的後續步驟
|話題 |連結|  
| --- | --- |
|AD FS 的所有發行項 | [Azure AD Connect 和同盟](how-to-connect-fed-whatis.md)|
|設定 ADFS 與子網域 | [與 Azure AD 同盟的多網域支援](how-to-connect-install-multiple-domains.md)|
|管理 AD FS 伺服器陣列 | [使用 Azure AD Connect 管理和自訂 AD FS](how-to-connect-fed-management.md)|
|手動更新同盟憑證 | [續約 Office 365 和 Azure AD 的同盟憑證](how-to-connect-fed-o365-certs.md)|


## <a name="get-started-with-azure-ad-connect-health"></a>開始使用 Azure AD Connect Health
若要開始使用 Azure AD Connect Health，請使用下列步驟：

1. [取得 Azure AD Premium](../fundamentals/active-directory-get-started-premium.md) 或[開始試用](https://azure.microsoft.com/trial/get-started-active-directory/)。
2. 在身分識別伺服器上[下載和安裝 Azure AD Connect Health 代理程式](#download-and-install-azure-ad-connect-health-agent)。
3. 在 [https://aka.ms/aadconnecthealth](https://aka.ms/aadconnecthealth) 檢視 Azure AD Connect Health 儀表板。

> [!NOTE]
> 請記住，您必須先在目標伺服器上安裝 Azure AD Connect Health 代理程式，Azure AD Connect Health 儀表板中才會出現資料。
>
>

## <a name="download-and-install-azure-ad-connect-health-agent"></a>下載和安裝 Azure AD Connect Health 代理程式
* 確定您符合 Azure AD Connect Health 的[需求](how-to-connect-health-agent-install.md#requirements)。
* 開始使用適用於 AD FS 的 Azure AD Connect Health
    * [下載適用於 AD FS 的 Azure AD Connect Health 代理程式](http://go.microsoft.com/fwlink/?LinkID=518973)
    * [請參閱安裝指示](how-to-connect-health-agent-install.md#installing-the-azure-ad-connect-health-agent-for-ad-fs)。
* 開始使用適用於同步處理的 Azure AD Connect Health
    * [下載並安裝最新版的 Azure AD Connect](http://go.microsoft.com/fwlink/?linkid=615771)。 適用於同步處理的健康狀態代理程式將會隨著 Azure AD Connect 安裝 (1.0.9125.0 或更高版本) 一起安裝。
* 開始使用適用於 AD DS 的 Azure AD Connect Health
    * [下載適用於 AD DS 的 Azure AD Connect Health 代理程式](http://go.microsoft.com/fwlink/?LinkID=820540)。
    * [請參閱安裝指示](how-to-connect-health-agent-install.md#installing-the-azure-ad-connect-health-agent-for-ad-ds)。


## <a name="azure-ad-connect-health-portal"></a>Azure AD Connect Health 入口網站
Azure AD Connect Health 入口網站會顯示警示、效能監視和使用情況分析。 https://aka.ms/aadconnecthealth URL 會帶您前往 Azure AD Connect Health 的主要刀鋒視窗。 您可以將刀鋒視窗視為視窗。 在主要刀鋒視窗上，您會看到 [快速啟動]、Azure AD Connect Health 中的服務以及其他設定選項。 請參閱下列螢幕擷取畫面，以及螢幕擷取畫面後面的簡短說明。 部署代理程式之後，健康狀態服務會自動識別 Azure AD Connect Health 所監視的服務。

> [!NOTE]
> 如需授權資訊，請參閱 [Azure AD Connect Health 常見問題集](reference-connect-health-faq.md)或 [Azure AD 定價頁面](https://aka.ms/aadpricing)。
    
![Azure AD Connect Health 入口網站](./media/whatis-hybrid-identity-health/portalsidebar.png)

* **快速入門**︰當您選取此選項時，[快速入門] 刀鋒視窗隨即開啟。 您可以選取 [取得工具] 下載 Azure AD Connect Health 代理程式。 您也可以存取文件，並提供意見反應。
* **Azure Active Directory Connect (同步處理)**：此選項會顯示 Azure AD Connect Health 目前正在監視的 Azure AD Connect 伺服器。 [同步錯誤] 項目會依照類別顯示第一個上線同步處理服務的基本同步錯誤。 當您選取 [同步處理服務] 時，開啟的刀鋒視窗會顯示 Azure AD Connect 伺服器的相關資訊。 如需深入了解這些功能，請參閱[使用 Azure AD Connect Health 進行同步處理](how-to-connect-health-sync.md)。
* **Active Directory Federation Services**：此選項會顯示 Azure AD Connect Health 目前正在監視的所有 AD FS 服務。 當您選取執行個體時，開啟的刀鋒視窗會顯示該服務執行個體的相關資訊。 這項資訊包括概觀、屬性、警示、監視和使用情況分析。 如需深入了解這些功能，請參閱[搭配 AD FS 使用 Azure AD Connect Health](how-to-connect-health-adfs.md)。
* **Active Directory Domain Services**：此選項會顯示 Azure AD Connect Health 目前正在監視的所有 AD DS 樹系。 當您選取樹系時，開啟的刀鋒視窗會顯示該樹系的相關資訊。 這項資訊包括基本資訊概觀、網域控制站儀表板、複寫狀態儀表板、警示和監視。 如需深入了解這些功能，請參閱[在 AD DS 使用 Azure AD Connect Health](how-to-connect-health-adds.md)。
* **設定**︰本節包含開啟或關閉下列功能的選項︰

  - [設定]  項目包含代理程式的基本組態。 自動升級設定可將 Azure AD Connect Health 代理程式自動更新為最新版本：您將會在發佈最新版的 Azure AD Connect Health 代理程式時，自動更新為最新版本。 此選項預設為啟用狀態。 僅允許 Microsoft 存取您的 Azure AD 目錄的健康狀況資料供疑難排解之用：如果啟用此選項，Microsoft 可以看到您所看到的相同資料。 這項資訊可能有助於進行問題的疑難排解和協助。 此選項預設為停用狀態。
* **角色型存取控制 (IAM)** 是用來管理角色基底中 Connect Health 資料存取權的區段。 

## <a name="next-steps"></a>後續步驟

- [硬體和先決條件](how-to-connect-install-prerequisites.md) 
- [快速設定](how-to-connect-install-express.md)
- [自訂設定](how-to-connect-install-custom.md)
- [密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)|
- [傳遞驗證](how-to-connect-pta.md)
- [Azure AD Connect 和同盟](how-to-connect-fed-whatis.md)
- [安裝 Azure AD Connect Health 代理程式](how-to-connect-health-agent-install.md) 
- [Azure AD Connect 同步處理](how-to-connect-sync-whatis.md)
