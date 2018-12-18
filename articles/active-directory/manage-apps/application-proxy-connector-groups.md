---
title: 在 Azure AD 應用程式 Proxy 中使用連接器群組在個別的網路和位置上發佈應用程式 | Microsoft Docs
description: 涵蓋如何建立和管理「Azure AD 應用程式 Proxy」中的連接器群組。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/31/2018
ms.author: barbkess
ms.reviewer: harshja
ms.custom: H1Hack27Feb2017; it-pro
ms.openlocfilehash: e149df09bf424d33fa9abdf8108b3b79534a8599
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39364968"
---
# <a name="publish-applications-on-separate-networks-and-locations-using-connector-groups"></a>使用連接器群組在個別的網路和位置上發佈應用程式

客戶在越來越多的案例和應用程式中利用 Azure AD 的應用程式 Proxy。 因此我們啟用更多的拓撲使應用程式 Proxy 更具彈性。 您可以建立應用程式 Proxy 連接器群組，如此便可以指派特定連接器以提供特定應用程式。 此功能讓您有更多的控制能力與方法來最佳化應用程式 Proxy 部署。 

每個應用程式 Proxy 連接器都會指派給連接器群組。 隸屬於相同連接器群組的所有連接器會做為高可用性及負載平衡的個別單位。 所有的連接器皆屬於連接器群組。 如果您未建立群組，則所有的連接器皆會在預設群組中。 系統管理員可以建立新的群組，並在 Azure 入口網站中將連接器指派給群組。 

所有應用程式均會指派給連接器群組。 如果您未建立群組，則所有的連接器皆會指派給預設群組。 但是，如果您將連接器組織成群組，可以設定每個應用程式使用特定連接器群組。 在此情況下，只有該群組中的連接器會在收到要求時為應用程式提供服務。 如果您的應用程式裝載在不同的位置中，這是個很有用的功能。 您可以根據位置來建立連接器群組，讓應用程式一律由實體位置相近的連接器提供服務。

>[!TIP] 
>如果您有大規模的應用程式 Proxy 部署，請勿將任何應用程式指派給預設連接器群組。 如此一來，新的連接器便不會收到任何即時流量，除非您將新連接器指派給作用中的連接器群組。 此設定也可以將連接器移回預設群組的方式，藉此讓連接器進入閒置模式，如此您便可以在不影響使用者的情況下執行維護。

## <a name="prerequisites"></a>必要條件
若要將連接器組成群組，您必須確定 [已安裝多個連接器](application-proxy-enable.md)。 當您安裝新的連接器時，它會自動加入「預設」  連接器群組。

## <a name="create-connector-groups"></a>建立連接器群組
使用以下步驟建立您所要的連接器群組，數量不拘。 

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 選取 [Azure Active Directory]  >  [企業應用程式]  >  [應用程式 Proxy]。
2. 選取 [新增連接器群組]。 [新增連接器群組] 刀鋒視窗隨即出現。

   ![選取新的連接器群組](./media/application-proxy-connector-groups/new-group.png)

3. 指定新連接器群組的名稱，然後使用下拉式功能表來選取哪些連接器屬於此群組。
4. 選取 [ **儲存**]。

## <a name="assign-applications-to-your-connector-groups"></a>將應用程式指派給您的連接器群組
已透過應用程式 Proxy 發佈的每個應用程式均按照這些步驟處理。 您可以在首次發佈應用程式時，將應用程式指派給連接器群組，也可以隨時使用這些步驟變更指派。   

1. 從您目錄的管理儀表板中，選取 [企業應用程式] > [所有應用程式] > 您想要指派給連接器群組的應用程式 > [應用程式 Proxy]。
2. 使用 [連接器群組] 下拉式功能表來選取應用程式所要使用的群組。
3. 按一下 [儲存]  以套用變更。

## <a name="use-cases-for-connector-groups"></a>連接器群組的使用案例 

連接器群組可用於多種不同狀況，包括：

### <a name="sites-with-multiple-interconnected-datacenters"></a>具有多個互連資料中心的網站

許多組織都有數個彼此連接的資料中心。 在此情況下，您會希望儘量將流量保留在資料中心內，因為跨資料中心的連結既昂貴、速度又慢。 您可以在每個資料中心都部署連接器，來只為資料中心內的應用程式提供服務。 這種方法可將跨資料中心的連結減到最少，讓使用者體驗完全的流暢性。

### <a name="applications-installed-on-isolated-networks"></a>安裝在隔離網路上的應用程式

應用程式可以在不是主要的公司網路一部分的網路上託管。 您可以使用連接器群組將專用連接器安裝在隔離網路上，以同時將應用程式與網路隔離。 當協力廠商為您的組織維護特定應用程式時，就會發生這種情形。 

連接器群組可讓您安裝僅發佈特定應用程式之網路的專用連接器，使外包應用程式管理給協力廠商更輕鬆且更安全。

### <a name="applications-installed-on-iaas"></a>IaaS 上已安裝應用程式 

對於安裝在 IaaS 上以供雲端存取的應用程式，連接器群組提供通用的服務來保護對所有應用程式的存取。 連接器群組不會對公司網路產生額外的相依性，或是造成不完整的應用程式體驗。 連接器可安裝在每個雲端資料中心，而且只為該網路中的應用程式提供服務。 您可以安裝數個連接器以獲得高可用性。

以擁有數個虛擬機器連線到本身 IaaS 主控之虛擬網路的組織為例。 若要允許員工使用這些應用程式，這些私人網路會使用站台對站 VPN 連線到公司網路。 針對位於內部部署的員工，這會提供良好的體驗。 但是，它可能不適合遠端員工，因為它需要額外內部部署基礎結構來路由存取權，如您所見下圖顯示︰

![AzureAD Iaas 網路](./media/application-proxy-connector-groups/application-proxy-iaas-network.png)
  
使用 Azure AD 應用程式 Proxy 連接器群組，您可以啟用一般服務來保護所有應用程式，而無須在您的公司網路上建立其他相依性︰

![AzureAD Iaas 多雲端廠商](./media/application-proxy-connector-groups/application-proxy-multiple-cloud-vendors.png)

### <a name="multi-forest--different-connector-groups-for-each-forest"></a>多樹系 – 每個樹系不同的連接器群組

大部分已部署應用程式 Proxy 的客戶會執行 Kerberos 限制委派 (KCD) 來使用單一登入 (SSO) 功能。 若要達到此目的，連接器的電腦需要加入可將使用者委派至應用程式的網域。 KCD 支援跨樹系功能。 但對於擁有不同的多樹系環境且它們之間沒有信任的公司來說，單一連接器無法用於所有的樹系。 

在此情況下，每個樹系可以部署特定連接器，並設定為僅為該特定樹系使用者提供服務所發佈的應用程式提供服務。 每個連接器群組代表不同的樹系。 雖然租用戶和大部分的體驗會針對所有樹系進行整合，使用者可以使用 Azure AD 群組指派給其樹系的應用程式。
 
### <a name="disaster-recovery-sites"></a>災害復原網站

您可以根據您站台的實作方式，使用災害復原 (DR) 網站採用兩種不同的方法︰

* 如果您的 DR 網站是在主動-主動模式中建置，其中與主要站台完全相同，且具有相同的網路與 AD 設定，您可以在與主要站台相同的連接器群組中，在 DR 網站上建立連接器。 這可讓 Azure AD 為您偵測容錯移轉。
* 如果 DR 網站是獨立於主要網站，您可以在 DR 網站中建立不同的連接器群組，並且 1) 擁有備份應用程式，或 2) 視需要手動將現有的應用程式轉向至 DR 連接器群組。
 
### <a name="serve-multiple-companies-from-a-single-tenant"></a>從單一租用戶為多家公司提供服務

有許多不同的方式可實作模型，其中單一服務提供者可部署及為多家公司維護 Azure AD 的相關服務。 連接器群組可協助系統管理員將連接器和應用程式分成不同的群組。 適合小型公司的一個方法是擁有單一 Azure AD 租用戶，同時不同公司都有自己的網域名稱和網路。 M&A 案例和情況也是如此，其中單一 IT 部門會針對規範或企業原因為幾家公司提供服務。 

## <a name="sample-configurations"></a>範例組態

您可以實作一些範例，包括下列連接器群組。
 
### <a name="default-configuration--no-use-for-connector-groups"></a>預設組態 – 不使用連接器群組

如果您不使用連接器群組，您的組態會看起來像這樣︰

![AzureAD 沒有連接器群組](./media/application-proxy-connector-groups/application-proxy-sample-config-1.png)
 
這個組態就對小型部署和測試就已足夠。 如果您的組織具有平面網路拓撲，則它也會正常運作。
 
### <a name="default-configuration-and-an-isolated-network"></a>預設組態和隔離的網路

此組態是預設的演化，其中有在隔離網路 (例如 IaaS 虛擬網路) 中執行的特定應用程式︰ 

![AzureAD 沒有連接器群組](./media/application-proxy-connector-groups/application-proxy-sample-config-2.png)
 
### <a name="recommended-configuration--several-specific-groups-and-a-default-group-for-idle"></a>建議的組態 – 幾個特定的群組和預設的閒置群組

大型及複雜組織的建議組態是具有預設連接器群組做為不提供任何應用程式服務並用於閒置或新安裝連接器的群組。 所有應用程式會使用自訂的連接器群組來提供服務。 這可啟用上述案例的所有複雜度。

下列範例中，公司有兩個資料中心 (A 和 B)，並具有兩個連接器可為每個站台提供服務。 每個網站都有不同的應用程式在其上執行。 

![AzureAD 沒有連接器群組](./media/application-proxy-connector-groups/application-proxy-sample-config-3.png)
 
## <a name="next-steps"></a>後續步驟

* [了解 Azure AD 應用程式 Proxy 連接器](application-proxy-connectors.md)
* [啟用單一登入](application-proxy-single-sign-on.md)


