---
title: Azure AD Connect：支援的拓撲 | Microsoft Docs
description: 本主題詳細說明 Azure AD Connect 的受支援和不受支援拓撲
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 1034c000-59f2-4fc8-8137-2416fa5e4bfe
ms.service: active-directory
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: identity
ms.topic: article
ms.date: 10/09/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: ef72829d507d6a471ec9a8972ead262da7f7b582
ms.sourcegitcommit: 7b0778a1488e8fd70ee57e55bde783a69521c912
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49068686"
---
# <a name="topologies-for-azure-ad-connect"></a>Azure AD Connect 的拓撲
本文說明使用 Azure AD Connect 同步處理做為重要整合解決方案的各種內部部署和 Azure Active Directory (Azure AD) 拓撲。 本文包含受支援和不受支援的組態。


以下是文章中圖片的圖例︰

| 說明 | 符號 |
| --- | --- |
| 內部部署 Active Directory 樹系 |![內部部署 Active Directory 樹系](./media/plan-connect-topologies/LegendAD1.png) |
| 內含篩選匯入的內部部署 Active Directory |![內含篩選匯入的 Active Directory](./media/plan-connect-topologies/LegendAD2.png) |
| Azure AD Connect 同步處理伺服器 |![Azure AD Connect 同步處理伺服器](./media/plan-connect-topologies/LegendSync1.png) |
| Azure AD Connect 同步處理伺服器「預備模式」 |![Azure AD Connect 同步處理伺服器「預備模式」](./media/plan-connect-topologies/LegendSync2.png) |
| GALSync 與 Forefront Identity Manager (FIM) 2010 或 Microsoft Identity Manager (MIM) 2016 |![GALSync 與 FIM 2010 或 MIM 2016](./media/plan-connect-topologies/LegendSync3.png) |
| Azure AD Connect 同步處理伺服器，詳細說明 |![Azure AD Connect 同步處理伺服器，詳細說明](./media/plan-connect-topologies/LegendSync4.png) |
| Azure AD |![Azure Active Directory](./media/plan-connect-topologies/LegendAAD.png) |
| 不受支援的案例 |![不受支援的案例](./media/plan-connect-topologies/LegendUnsupported.png) |


> [!IMPORTANT]
> Microsoft 不支援在正式記載的組態或動作以外修改和操作 Azure AD Connect 同步處理。 任何這些組態或動作都可能導致 Azure AD Connect 同步處理發生不一致或不支援的狀態。如此一來，Microsoft 無法提供這類部署的技術支援人員。


## <a name="single-forest-single-azure-ad-tenant"></a>單一樹系、單一 Azure AD 租用戶
![單一樹系和單一租用戶的拓撲](./media/plan-connect-topologies/SingleForestSingleDirectory.png)

最常見的拓撲是單一內部部署樹系 (內含一或多個網域) 和單一 Azure AD 租用戶。 對於 Azure AD 驗證，會使用密碼雜湊同步處理。 Azure AD Connect 的快速安裝僅支援此拓撲。

### <a name="single-forest-multiple-sync-servers-to-one-azure-ad-tenant"></a>單一樹系、多部同步處理伺服器連接到一個 Azure AD 租用戶
![不受支援、已篩選的單一樹系拓撲](./media/plan-connect-topologies/SingleForestFilteredUnsupported.png)

不支援多部的 Azure AD Connect 同步處理伺服器連線到相同 Azure AD 租用戶 (除了 [預備伺服器](#staging-server)之外)。 即使這些伺服器設定為同步處理互斥的一組物件，也不支援。 如果您無法從單一伺服器觸達樹系中的所有網域，或者想要將負載分散到數個伺服器，您可能已經考慮過這個拓撲。

## <a name="multiple-forests-single-azure-ad-tenant"></a>多個樹系、單一 Azure AD 租用戶
![多個樹系和單一租用戶的拓撲](./media/plan-connect-topologies/MultiForestSingleDirectory.png)

許多組織都有包含多個內部部署 Active Directory 樹系的環境。 擁有多個內部部署 Active Directory 的原因有很多。 常見的範例如包含帳戶資源樹系，和合併或收購的結果。

當您擁有多個樹系，所有樹系必須可由單一 Azure AD Connect 同步處理伺服器連線。 您不需要將伺服器加入網域。 如有必要觸達所有的樹系，您可以將伺服器放置於周邊網路 (也稱為 DMZ、非軍事區域及遮蔽式子網路)。

Azure AD Connect 安裝精靈會提供數個選項以合併多個樹系中表示的使用者。 目標是使用者只會在 Azure AD 中顯示一次。 有一些常見的拓撲，您可以在安裝精靈中的自訂安裝路徑中設定。 在 [專門識別您的使用者] 頁面上，選取表示拓撲的對應選項。 只對使用者設定合併。 重複群組不會和預設組態合併。

常見的拓撲將在[個別拓撲](#multiple-forests-separate-topologies)、[完整網狀](#multiple-forests-full-mesh-with-optional-galsync)和[帳戶資源拓撲](#multiple-forests-account-resource-forest)相關章節中討論。

Azure AD Connect 同步處理中的預設組態假設：

* 每個使用者只有一個啟用的帳戶，且此帳戶所在之樹系用於驗證使用者。 這項假設適用於密碼雜湊同步處理、傳遞驗證和同盟。 UserPrincipalName 和 sourceAnchor/immutableID 來自此樹系
* 每個使用者只有一個信箱。
* 裝載使用者信箱的樹系具有最佳資料品質，以供屬性在 Exchange 全域通訊清單 (GAL) 中顯示。 如果使用者沒有信箱，則任何樹系皆可用於提供這些屬性值。
* 如果您有連結的信箱，則不同的樹系中還有帳戶用來登入。

如果您的環境不符合這些假設，則會發生下列情況：

* 如果您有多個使用中的帳戶或多個信箱，同步處理引擎會挑選其中一個，其他全部忽略。
* 沒有使用中帳戶的已連結的信箱不會匯出至 Azure AD。 使用者帳戶不會顯示為任何群組中的成員。 DirSync 中已連結的信箱一律會顯示為一般信箱。 這個變更是刻意的不同行為，可更有效支援多個樹系案例。

您可以在[了解預設組態](concept-azure-ad-connect-sync-default-configuration.md)中找到更多詳細資料。

### <a name="multiple-forests-multiple-sync-servers-to-one-azure-ad-tenant"></a>多個樹系、多部同步處理伺服器連接到一個 Azure AD 租用戶
![多個樹系和多個同步處理伺服器的不受支援拓撲](./media/plan-connect-topologies/MultiForestMultiSyncUnsupported.png)

不支援超過一部的 Azure AD Connect 同步處理伺服器連線到單一 Azure AD 租用戶。 例外狀況是使用 [預備伺服器](#staging-server)。

### <a name="multiple-forests-separate-topologies"></a>多個樹系，個別拓撲
![跨所有目錄僅顯示使用者一次的選項](./media/plan-connect-topologies/MultiForestUsersOnce.png)

![多個樹系和個別拓撲的描述](./media/plan-connect-topologies/MultiForestSeperateTopologies.png)

在此環境中，所有內部部署樹系會被視為個別的實體。 使用者不會顯示在其他樹系中。 每個樹系具有自己的 Exchange 組織，且在樹系間沒有 GALSync。 這個拓撲可能是合併/收購之後，或是在一個每個業務單位各自獨立作業的組織中。 在 Azure AD 中，這些樹系將會在相同的組織中並與統一 GAL 一起出現。 在上圖中，每個樹系中的個別物件都會在 Metaverse 中顯示一次，並在目標 Azure AD 租用戶中彙總。

### <a name="multiple-forests-match-users"></a>多個樹系：比對使用者
這些案例的常見狀況是通訊群組和安全性群組可以包含使用者、連絡人和外部安全性主體 (FSP) 的組合。 FSP 可在 Active Directory Domain Services (ADDS) 中用來代表安全性群組中來自其他樹系的成員。 在 Azure AD 中，所有 FSP 都會解析為實際物件。

### <a name="multiple-forests-full-mesh-with-optional-galsync"></a>多個樹系：內含選擇性 GALSync 的完整網狀
![當使用者身分識別跨多個目錄存在時，使用郵件屬性進行比對的選項](./media/plan-connect-topologies/MultiForestUsersMail.png)

![多個樹系的完整網狀拓撲](./media/plan-connect-topologies/MultiForestFullMesh.png)

完整網狀拓撲可讓使用者和資源位於任何樹系中。 通常，在樹系之間有雙向信任關係。

如果 Exchange 出現在一個以上的樹系中，有可能是選擇性的內部部署 GALSync 解決方案。 然後每個使用者可以顯示為所有其他樹系中的連絡人。 GALSync 通常會透過 FIM 2010 或 MIM 2016 實作。 Azure AD Connect 無法用於內部部署 GALSync。

在此案例中，身分識別物件是透過郵件屬性來聯結。 在一個樹系中擁有信箱的使用者會與其他樹系中的連絡人聯結。

### <a name="multiple-forests-account-resource-forest"></a>多個樹系：帳戶資源樹系
![當身分識別跨多個目錄存在時，用來使用 ObjectSID 和 msExchMasterAccountSID 屬性進行比對的選項](./media/plan-connect-topologies/MultiForestUsersObjectSID.png)

![多個樹系的帳戶資源樹系拓撲](./media/plan-connect-topologies/MultiForestAccountResource.png)

在帳戶資源樹系拓撲中，您會有一個以上的*帳戶*樹系內含作用中的使用者帳戶。 您也會有一或多個*資源*樹系具有停用的帳戶。

在此案例中，一個 (或多個) 資源樹系信任所有帳戶樹系。 此資源樹系通常具有擴充的 Active Directory 結構描述與 Exchange 和 Lync。 所有的 Exchange 和 Lync 服務以及其他共用的服務都位於此樹系。 使用者在此樹系中擁有停用的使用者帳戶，而信箱會連結至帳戶樹系。

## <a name="office-365-and-topology-considerations"></a>Office 365 和拓撲考量
有些 Office 365 工作負載對受支援的拓撲有某些限制：

| 工作負載 | 限制 |
| --------- | --------- |
| Exchange Online | 如需有關 Exchange Online 所支援之混合式技術的詳細資訊，請參閱[具有多個 Active Directory 樹系的混合部署](https://technet.microsoft.com/library/jj873754.aspx)。 |
| 商務用 Skype | 使用多個內部部署樹系時，只會支援帳戶資源樹系拓撲。 如需詳細資訊，請參閱[商務用 Skype Server 2015 的環境需求](https://technet.microsoft.com/library/dn933910.aspx)。 |

如果您是較大型的組織，那麼就應該考量使用 [Office 365 PreferredDataLocation](how-to-connect-sync-feature-preferreddatalocation.md) 功能。 它可讓您定義使用者的資源要放置於哪個資料中心區域。

## <a name="staging-server"></a>預備伺服器
![在拓撲中的預備伺服器](./media/plan-connect-topologies/MultiForestStaging.png)

Azure AD Connect 支援以「預備模式」安裝第二部伺服器。 此模式中的伺服器會讀取所有已連接目錄中的資料，但是不會將任何資料寫入已連接的目錄。 它使用一般的同步處理循環，因此也有身分識別資料的更新複本。

在主要伺服器失敗的災害中，您可以容錯移轉到預備伺服器。 您在 Azure AD Connect 精靈中執行這項操作。 第二部伺服器可以位於不同的資料中心，因為沒有和主要伺服器共用基礎結構。 您必須手動將主要伺服器上進行的任何組態變更複製到第二部伺服器。

您可以使用預備伺服器以測試新的自訂組態以及它對您的資料所造成的影響。 您可以預覽變更並調整組態。 當您滿意新的組態時，您可以讓預備伺服器成為使用中的伺服器，並將舊的使用中伺服器設定為預備模式。

您也可以使用這個方法來取代使用中的同步處理伺服器。 準備新的伺服器，並且將其設定為預備模式。 請確定它處於良好狀態、停用預備模式 (讓它成為使用中)，並關閉目前作用中的伺服器。

如果您想要在不同的資料中心有多個備份，您也可以擁有一個以上的預備伺服器。

## <a name="multiple-azure-ad-tenants"></a>多個 Azure AD 租用戶
我們建議一個組織在 Azure AD 中有單一租用戶。
在您計劃使用多個 Azure AD 租用戶之前，請參閱文章 [Azure AD 中的系統管理單位管理](../users-groups-roles/directory-administrative-units.md)。 它涵蓋了常見的案例，您可以使用單一租用戶。

![多個樹系和多個租用戶的拓撲](./media/plan-connect-topologies/MultiForestMultiDirectory.png)

Azure AD Connect 同步處理伺服器和 Azure AD 租用戶之間有 1:1 的關聯性。 在每個 Azure AD 租用戶中，您將需要一個 Azure AD Connect 同步處理伺服器安裝。 Azure AD 租用戶執行個體是由設計隔離。 也就是在一個租用戶中的使用者無法看到其他租用戶中的使用者。 如果您想要這樣的分隔方式，這是支援的組態。 否則，您應該使用單一 Azure AD 租用戶模型。

### <a name="each-object-only-once-in-an-azure-ad-tenant"></a>每個物件只在 Azure AD 租用戶運作一次
![已篩選的單一樹系拓撲](./media/plan-connect-topologies/SingleForestFiltered.png)

在此拓撲中，一個 Azure AD Connect 同步處理伺服器會連接到每個 Azure AD 租用戶。 Azure AD Connect 同步處理伺服器必須設定篩選，以各自有一組要操作的互斥物件。 例如，將每個伺服器的範圍限定於特定網域或組織單位。

DNS 網域只能在單一 Azure AD 租用戶中註冊。 內部部署 Active Directory 執行個體中的使用者 UPN 也必須使用個別的命名空間。 例如，在上圖中，三個個別的 UPN 尾碼都登錄在內部部署 Active Directory 執行個體中：contoso.com、fabrikam.com 和 wingtiptoys.com。 每個內部部署 Active Directory 網域中的使用者會使用不同的命名空間。

>[!NOTE]
>「全域通訊清單同步處理」(GalSync) 在此拓撲中不會自動執行，需要額外的自訂 MIM 實作以確保每個租用戶在 Exchange Online 和「商務用 Skype Online」中都擁有完整的「全域通訊清單」(GAL)。


對於另外支援的案例，此拓撲具有下列限制︰

* 只有其中一個 Azure AD 租用戶可以啟用 Exchange 與內部部署 Active Directory 執行個體的混合。
* Windows 10 裝置只能與一個 Azure AD 租用戶相關聯。
* 適用於密碼雜湊同步處理和傳遞驗證的單一登入 (SSO) 選項只能與一個 Azure AD 租用戶搭配使用。

物件互斥集的需求也適用於寫回。 此拓撲不支援部分寫回功能，因為這些功能假設單一內部部署組態。 這些功能包括：

* 使用預設組態的群組回寫。
* 裝置回寫。

### <a name="each-object-multiple-times-in-an-azure-ad-tenant"></a>每個物件在 Azure AD 租用戶運作多次
![單一樹系與多個租用戶的不受支援拓撲](./media/plan-connect-topologies/SingleForestMultiDirectoryUnsupported.png) ![單一樹系與多個連接器的不受支援拓撲](./media/plan-connect-topologies/SingleForestMultiConnectorsUnsupported.png)

下列工作不受支援：

* 將相同使用者同步處理至多個 Azure AD 租用戶。
* 進行組態變更，讓一個 Azure AD 中的使用者顯示為另一個 Azure AD 租用戶中的連絡人。
* 將 Azure AD Connect 同步處理修改為連線到多個 Azure AD 租用戶。

### <a name="galsync-by-using-writeback"></a>使用回寫 GALSync
![多個樹系和多個目錄的不受支援拓撲，具有將焦點放在 Azure AD 的 GALSync](./media/plan-connect-topologies/MultiForestMultiDirectoryGALSync1Unsupported.png) ![多個樹系和多個目錄的不受支援拓撲，具有將焦點放在內部部署 Azure AD 的 GALSync](./media/plan-connect-topologies/MultiForestMultiDirectoryGALSync2Unsupported.png)

Azure AD 租用戶在設計上是隔離的。 下列工作不受支援：

* 將 Azure AD Connect 同步處理的組態變更為從另一個 Azure AD 租用戶讀取資料。
* 使用 Azure AD Connect 同步處理將使用者匯出為另一個內部部署 Active Directory 執行個體的連絡人。

### <a name="galsync-with-on-premises-sync-server"></a>具備內部部署同步處理伺服器的 GALSync
![多個樹系和多個目錄拓撲中的 GALSync](./media/plan-connect-topologies/MultiForestMultiDirectoryGALSync.png)

您可以使用 FIM 2010 或 MIM 2016 內部部署來同步處理兩個 Exchange 組織之間的使用者 (透過 GALSync)。 一個組織中的使用者將會顯示為其他組織中的外部使用者/連絡人。 這些不同的內部部署 Active Directory 執行個體可同步處理至它們自己的 Azure AD 租用戶。

## <a name="next-steps"></a>後續步驟
若要了解如何安裝這些案例的 Azure AD Connect，請參閱[自訂 Azure AD Connect 安裝](how-to-connect-install-custom.md)。

深入了解 [Azure AD Connect 同步](how-to-connect-sync-whatis.md) 組態。

深入了解[整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
