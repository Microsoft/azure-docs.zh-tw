---
title: 混合式身分識別設計 - Azure 採用策略 | Microsoft Docs
description: 透過條件式存取控制，Azure Active Directory 會在驗證使用者時以及允許存取應用程式之前，檢查您挑選的特定條件。 一旦符合這些條件，就會驗證使用者並允許存取應用程式。
documentationcenter: ''
services: active-directory
author: billmath
manager: mtillman
editor: ''
ms.assetid: b92fa5a9-c04c-4692-b495-ff64d023792c
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/30/2018
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: 5aeefe82255d40938eff09d7920e2fd5aca94c95
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46308297"
---
# <a name="define-a-hybrid-identity-adoption-strategy"></a>定義混合式身分識別採用策略
在這項工作中，您將為混合式身分識別解決方案，定義混合式身分識別採用策略，以滿足下列幾節已討論的商務需求：

* [判斷商務需求](plan-hybrid-identity-design-considerations-business-needs.md)
* [判斷目錄同步處理需求](plan-hybrid-identity-design-considerations-directory-sync-requirements.md)
* [判斷多重要素驗證需求](plan-hybrid-identity-design-considerations-multifactor-auth-requirements.md)

## <a name="define-business-needs-strategy"></a>定義商務需求策略
第一項工作在於判斷組織商務需求。  這可能非常廣泛，一不小心就會發生範圍蔓延。  剛開始要保持簡單，但要時常記住所規劃的設計應該兼顧和推動未來的變更。  無論是簡單或極為複雜的設計，Azure Active Directory 都是支援 Office 365、Microsoft Online Services 和雲端感知應用程式的 Microsoft 身分識別平台。

## <a name="define-an-integration-strategy"></a>定義整合策略
Microsoft 有三個主要的整合案例，分別為雲端身分識別、同步處理身分識別和同盟身分識別。  您應該規劃採用這些整合策略的其中一個。  您選擇的策略可能不同，決策的因素可能包括您想要提供何種使用者體驗、是否有現有的基礎結構，以及什麼最符合成本效益。  

![](./media/plan-hybrid-identity-design-considerations/integration-scenarios.png)

上圖所定義的案例如下：

* **雲端身分識別**：這些是僅存在雲端的身分識別。  就 Azure AD 來說，它們實際上就位於您的 Azure AD 目錄中。
* **同步處理**：這些是存在於內部部署和雲端的身分識別。  使用 Azure AD Connect 時，將會以現有的 Azure AD 帳戶建立或聯結這些使用者。  在所謂的密碼雜湊中，使用者的密碼雜湊會從內部部署環境同步處理到雲端。  使用同步處理時，必須注意如果在內部部署環境中停用使用者，可能需要經過 3 小時，Azure AD 中才會顯示該帳戶狀態。  這是因為有同步處理時間間隔。
* **同盟**：這些身分識別同時存在於內部部署和雲端。  使用 Azure AD Connect 時，將會以現有的 Azure AD 帳戶建立或聯結這些使用者。  

> [!NOTE]
> 如需同步處理選項的詳細資訊，請參閱 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
> 
> 

下表有助於判斷下列每個策略的優缺點：

| 策略 | 優點 | 缺點 |
| --- | --- | --- |
| **雲端身分識別** |在小型組織中易於管理。 <br> 沒有項目可以在內部安裝或更新。 沒有其他硬體需求<br>使用者離職時輕易停用 |使用者存取雲端中的工作負載時必須登入 <br> 雲端和內部部署身分識別的密碼可能相同，也可能不相同 |
| **已同步處理** |內部部署密碼會驗證內部部署和雲端目錄 <br>對小型、中型或大型組織來說易於管理 <br>使用者可以對一些資源進行單一登入 (SSO) <br> Microsoft 對於同步處理的慣用方法 <br> 易於管理 |基於公司的特定原則，有些客戶可能不願意將目錄與雲端同步 |
| **同盟** |使用者可以有單一登入 (SSO)  <br>如果使用者終止或離開時，可以將帳戶立即停用並撤銷存取權。<br> 支援同步處理所無法解決的進階案例 |安裝和設定的詳細步驟 <br> 較高的維護 <br> STS 基礎結構可能需要額外的硬體 <br> 安裝同盟伺服器可能需要額外的硬體。 如果使用 AD FS，則需要額外的軟體 <br> 需要大量的設定才能使用 SSO <br> 重大失敗點是，同盟伺服器關閉時使用者會無法驗證 |

### <a name="client-experience"></a>用戶端體驗
您使用的策略決定使用者的登入體驗。  下表提供使用者所預期之登入體驗的相關資訊。  所有同盟識別提供者並非在所有案例中都支援 SSO。

**加入網域和私人網路應用程式**：

|  | 同步處理身分識別 | 同盟身分識別 |
| --- | --- | --- |
| 網頁瀏覽器 |表單架構驗證 |單一登入，有時需要提供組織識別碼 |
| Outlook |提示輸入認證 |提示輸入認證 |
| 商務用 Skype (Lync) |提示輸入認證 |在 Lync 中需要單一登入，在 Exchange 中會提示輸入認證 |
| OneDrive for Business |提示輸入認證 |單一登入 |
| Office Pro Plus 訂用帳戶 |提示輸入認證 |單一登入 |

**外部或未受信任的來源**：

|  | 同步處理身分識別 | 同盟身分識別 |
| --- | --- | --- |
| 網頁瀏覽器 |表單架構驗證 |表單架構驗證 |
| Outlook、商務用 Skype (Lync)、商務用 OneDrive、Office 訂用帳戶 |提示輸入認證 |提示輸入認證 |
| Exchange ActiveSync |提示輸入認證 |在 Lync 中需要單一登入，在 Exchange 中會提示輸入認證 |
| 行動應用程式 |提示輸入認證 |提示輸入認證 |

如果您已在工作 1 確定您擁有第三方的 IdP，或是要使用 IdP 來提供與 Azure AD 的同盟，您需要留意下列支援功能︰

* 任何與 SP-Lite 設定檔相容的 SAML 2.0 提供者，皆可支援對 Azure AD 和相關聯應用程式的驗證
* 支援被動驗證，可協助對 OWA、SPO 等的驗證。
* 可透過 SAML 2.0 增強型用戶端設定檔 (ECP) 支援 Exchange Online 用戶端

您也必須知道無法使用哪些功能：

* 如果沒有 WS-Trust/同盟支援，其他所有使用中用戶端都會中斷
  * 這表示沒有 Office 2016 之前的 Lync 用戶端、OneDrive 用戶端、Office 訂用帳戶、Office Mobile
* Office 轉換為被動驗證可讓它們支援純 SAML 2.0 IdP，但支援仍然以個別用戶端為準

> [!NOTE]
> 如需最新的清單，請參閱 https://aka.ms/ssoproviders 一文。
> 
> 

## <a name="define-synchronization-strategy"></a>定義同步處理策略
在這項工作中，您將定義用於將組織的內部部署資料同步處理至雲端的工具，以及您應該使用的拓撲。  因為大部分的組織都使用 Active Directory，我們稍微詳細提供有關使用 Azure AD Connect 來解決上述問題的資訊。  針對沒有 Active Directory 的環境，也提供有關使用 FIM 2010 R2 或 MIM 2016 來協助規劃此策略的資訊。  不過，未來的 Azure AD Connect 版本將支援 LDAP 目錄，因此，視您的時間表而定，這項資訊可以有幫助。

### <a name="synchronization-tools"></a>同步處理工具
多年來，已出現許多同步處理工具並用於各種案例。  在所有支援的案例中，Azure AD Connect 是目前首選的工具。  AAD 同步和 DirSync 也依然存在，甚至現在就在您的環境中。 

> [!NOTE]
> 如需每一項工具支援的功能的最新資訊，請參閱 [目錄整合工具比較](plan-hybrid-identity-design-considerations-tools-comparison.md) 一文。  
> 
> 

### <a name="supported-topologies"></a>支援的拓撲
定義同步處理策略時，必須判斷使用的拓撲。 您可以根據步驟 2 中已判斷的資訊，判斷適合使用的拓撲。 單一樹系、單一 Azure AD 拓撲最常見，由單一 Active Directory 樹系和單一 Azure AD 執行個體所組成。  這將用於大部分的案例中，也是在使用 Azure AD Connect Express 安裝時預期的拓撲，如下圖所示。

![](./media/plan-hybrid-identity-design-considerations/single-forest.png) 單一樹系案例。在大型組織，甚至小型組織中，擁有多個樹系很平常，如圖 5 所示。

> [!NOTE]
> 如需不同的內部部署和 Azure AD 拓撲與 Azure AD Connect 同步處理的詳細資訊，請參閱 [Azure AD Connect 的拓撲](plan-connect-topologies.md)一文。
> 
> 

![](./media/plan-hybrid-identity-design-considerations/multi-forest.png) 

多樹系案例

如果是這樣，當下列條件成立時，應該採用多樹系單一 Azure AD 拓撲：

* 使用者在所有樹系中只有 1 個身分識別 – 下面的唯一識別使用者一節有更詳細的說明。
* 使用者向他們的身分識別所在的樹系驗證
* UPN 和來源錨點 (固定 ID) 來自此樹系
* Azure AD Connect 可存取所有樹系 – 這表示不需要加入網域，而且可以放入 DMZ 中 (如果有利於這一點)。
* 使用者只有一個信箱
* 對於 Exchange 全域通訊清單 (GAL) 中可見的屬性，裝載使用者信箱的樹系有最佳的資料品質
* 如果使用者沒有信箱，則任何樹系都可用於提供這些值
* 如果您有連結的信箱，則不同的樹系中還有另一個帳戶用來登入。

> [!NOTE]
> 同時存在於內部部署和雲端的物件透過唯一識別碼「連接」。 就「目錄同步作業」而言，這個唯一識別碼稱為 SourceAnchor。 就「單一登入」而言，這稱為 ImmutableId。 [Azure AD Connect 的設計概念](plan-connect-design-concepts.md#sourceanchor) 。
> 
> 

如果上述條件不成立，而且您有多個使用中的帳戶或多個信箱，Azure AD Connect 會挑選其中一個，其他全部忽略。  如果您有連結的信箱，但沒有其他帳戶，這些帳戶不會匯出至 Azure AD，而且該使用者不會是任何群組的成員。  這不同以往在 DirSync 中的情形，主要是為了更充分支援這些多樹系案例。 下圖顯示多樹系案例。

![](./media/plan-hybrid-identity-design-considerations/multiforest-multipleAzureAD.png) 

**多樹系多個 Azure AD 案例**

在組織的 Azure AD 中，建議只有單一目錄，但只要 Azure AD Connect 同步作業伺服器與 Azure AD 目錄之間維持 1:1 關聯性，則仍然支援。  對於每個 Azure AD 執行個體，您需要安裝 Azure AD Connect。  此外，Azure AD 是隔離設計，一個 Azure AD 執行個體中的使用者看不到其他執行個體中的使用者。

可以且支援將一個 Active Directory 內部部署執行個體連接到多個 Azure AD 目錄，如下圖所示：

![](./media/plan-hybrid-identity-design-considerations/single-forest-flitering.png) 

**單一樹系篩選案例**

若要這麼做，下列條件必須成立：

* Azure AD Connect 同步作業伺服器必須設定篩選，以各自有一組互斥的物件。  例如，將每個伺服器的範圍限定於特定網域或 OU。
* DNS 網域只能在單一 Azure AD 目錄中註冊，因此內部部署 AD 中的使用者 UPN 必須使用個別的命名空間。
* 一個 Azure AD 執行個體中的使用者只能看到他們的執行個體中的使用者。  他們看不到其他執行個體中的使用者
* 只有其中一個 Azure AD 目錄可以啟用 Exchange 與內部部署 AD 混合
* 回寫也有互斥性。  這造成此拓撲不支援部分回寫功能，因為這些功能都假設使用單一內部部署組態。  其中包括：
  * 使用預設組態的群組回寫
  * 裝置寫回

不支援下列各項，而且不應該選擇作為實作：

* 不支援多個 Azure AD Connect 同步作業伺服器連接到相同的 Azure AD 目錄，即使它們已設定為同步處理一組互斥的物件也一樣
* 它不支援同步處理相同使用者至多個 Azure AD 目錄。 
* 也不支援進行組態變更，讓一個 Azure AD 中的使用者顯示為另一個 Azure AD 目錄中的連絡人。 
* 也不支援將 Azure AD Connect 同步處理修改為連接到多個 Azure AD 目錄。
* Azure AD 目錄在設計上是隔離的。 它不支援將 Azure AD Connect 同步處理變更為從另一個 Azure AD 目錄讀取資料，以嘗試在目錄之間建置一般和統一的 GAL。 也不支援使用 Azure AD Connect 同步處理將使用者匯出為另一個內部部署 AD的連絡人。

> [!NOTE]
> 如果您的組織禁止您網路上的電腦連線到網際網路，這篇文章列出您在用戶端電腦的輸出允許清單和 Internet Explorer 信任的網站區域中應該加入的端點 (FQDN、IPv4 和 IPv6 位址範圍)，以確保您的電腦可以成功使用 Office 365。 如需詳細資訊，請參閱 [Office 365 URL 和 IP 位址範圍](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2?ui=en-US&rs=en-US&ad=US)。
> 
> 

## <a name="define-multi-factor-authentication-strategy"></a>定義多重要素驗證策略
在這項工作中，您將定義要使用的多重要素驗證策略。  Azure Multi-Factor Authentication 有兩個不同的版本。  其中一個是雲端型，另一個是使用 Azure MFA Server 的內部部署型。  根據您以上所做的評估，您可以判斷何者是策略的正確解決方案。  使用下表來判斷哪一個設計選項最符合公司的安全性需求：

多重要素設計選項：

| 要保護的資產 | 雲端 MFA | MFA 內部部署 |
| --- | --- | --- |
| Microsoft 應用程式 |是 |是 |
| 應用程式資源庫中的 SaaS 應用程式 |是 |是 |
| 透過 Azure AD App Proxy 發佈的 IIS 應用程式 |是 |是 |
| 非透過 Azure AD App Proxy 發行的 IIS 應用程式 |no |是 |
| VPN、RDG 等遠端存取 |no |是 |

即使已選定策略的解決方案，您仍然需要使用上述評估來判斷使用者位於何處。  這可能會造成解決方案變更。  使用下表來協助判斷：

| 使用者位置 | 建議的設計選項 |
| --- | --- |
| Azure Active Directory |雲端中的 Multi-Factor Authentication |
| Azure AD 和使用 AD FS 同盟的內部部署 AD |兩者 |
| Azure AD 和內部部署 AD，使用 Azure AD Connect，沒有密碼同步處理 |兩者 |
| Azure AD 和內部部署，使用 Azure AD Connect，搭配密碼同步處理 |兩者 |
| 內部部署 AD |Multi-Factor Authentication Server |

> [!NOTE]
> 您也應該確定您選取的多重要素驗證設計選項，支援您的設計所需的功能。  如需詳細資訊，請參閱 [選擇合適的多重因素安全性解決方案](../authentication/concept-mfa-whichversion.md#what-am-i-trying-to-secure)。
> 
> 

## <a name="multi-factor-auth-provider"></a>Multi-Factor Auth Provider
依預設，擁有 Azure Active Directory 租用戶的全域管理員可以使用 Multi-Factor Authentication。 不過，如果您想要將多重要素驗證延伸到所有使用者，及/或想要讓全域管理員利用管理入口網站、自訂問候語及報告等功能，您必須購買並設定 Multi-Factor Authentication 提供者。

> [!NOTE]
> 您也應該確定您選取的多重要素驗證設計選項，支援您的設計所需的功能。 
> 
> 

## <a name="next-steps"></a>後續步驟
[判斷資料保護需求](plan-hybrid-identity-design-considerations-dataprotection-requirements.md)

## <a name="see-also"></a>另請參閱
[設計考量概觀](plan-hybrid-identity-design-considerations-overview.md)

