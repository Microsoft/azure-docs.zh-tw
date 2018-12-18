---
title: 在 Azure 入口網站中建立 Azure BizTalk 服務 | Microsoft Docs
description: 了解如何在 Azure 入口網站中佈建或建立 BizTalk 服務；MABS，WABS
services: biztalk-services
documentationcenter: ''
author: MandiOhlinger
manager: anneta
editor: ''
ms.assetid: 3ad18876-a649-40d6-9aa0-1509c1d62c43
ms.service: biztalk-services
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 11/07/2016
ms.author: mandia
ms.openlocfilehash: 61776b19ba0ee273b78e3b0a6f610e5701251dd0
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/11/2017
ms.locfileid: "26632715"
---
# <a name="create-biztalk-services-using-the-azure-portal"></a>使用 Azure 入口網站建立 BizTalk 服務

> [!INCLUDE [BizTalk Services is being retired, and replaced with Azure Logic Apps](../../includes/biztalk-services-retirement.md)]

> [!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)]

> [!TIP]
> 若要登入 Azure 入口網站，您需要 Azure 帳戶和 Azure 訂用帳戶。 如果沒有帳戶，您可在幾分鐘內建立免費試用帳戶。 查看 [Azure 免費試用](http://go.microsoft.com/fwlink/p/?LinkID=239738)。


## <a name="CreateService"></a>建立 BizTalk 服務

> [!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)]

有一些無法完成的作業，視 BizTalk 服務的狀態而定。 如需這些作業的清單，請參閱 [BizTalk 服務狀態圖](biztalk-service-state-chart.md)。

## <a name="post-provisioning-steps"></a>佈建後續步驟
* [在本機電腦上安裝憑證](#InstallCert)
* [新增實際執行備妥憑證](#AddCert)
* [取得存取控制命名空間](#ACS)

#### <a name="InstallCert"></a>在本機電腦上安裝憑證

> [!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)]

#### <a name="AddCert"></a>新增實際執行備妥憑證

> [!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)]

#### <a name="ACS"></a>取得存取控制命名空間

> [!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)]

從 Visual Studio 部署 BizTalk 服務專案時，您可以輸入此存取控制命名空間。 為 BizTalk 服務自動建立存取控制命名空間。

存取控制值可用於任何應用程式。 當建立 Azure BizTalk 服務時，此存取控制命名空間會利用 BizTalk 服務部署來控制驗證。 如果想要變更訂閱或管理命名空間，請在左導覽窗格中選取 [ACTIVE DIRECTORY]  ，然後選取您的命名空間。 工作列會列出您的選項。

按一下 [ **管理** ]，即可開啟存取控制管理入口網站。 在存取控制管理入口網站中，BizTalk 服務會使用 [服務身分識別]：  
![存取控制管理入口網站中的 ACS 服務身分識別][ACSServiceIdentities]

存取控制服務身分識別是一組認證，可讓應用程式或用戶端直接使用 Azure AD 存取控制進行驗證，並接收權杖。

> [!IMPORTANT]
> BizTalk 服務會使用 [擁有者] 做為預設服務識別，並使用 [密碼] 值。 如果您使用對稱金鑰值而不是密碼值，則可能發生下列錯誤。<br/><br/>*無法利用指定的認證連接至存取控制管理服務帳戶*
> 
> 

[管理您的 ACS 命名空間](https://msdn.microsoft.com/library/azure/hh674478.aspx) 列出一些指導方針和建議。

## <a name="requirements-explained"></a>說明各項需求
這些需求並不適用於免費版本。

<table border="1">
<tr bgcolor="FAF9F9">
        <td><strong>您需要什麼</strong></td>
        <td><strong>您為何需要它</strong></td>
</tr>
<tr>
<td>Azure 訂閱</td>
<td>訂用帳戶可決定誰能夠登入 Azure。 帳戶持有者可在 <a HREF="https://account.windowsazure.com/Subscriptions">Azure 訂用帳戶</a>中建立訂用帳戶。
<br/><br/>
Azure 帳戶可擁有多個訂用帳戶，只要使用者取得允許皆可管理這些帳戶。 例如，您的 Azure 帳戶持有者建立一個名稱為 <em>BizTalkServiceSubscription</em> 的訂用帳戶，並給與您公司內的 BizTalk 系統管理員 (例如 ContosoBTSAdmins@live.com) 存取此訂閱的權限。 在此案例中，BizTalk 系統管理員可以登入 Azure，並對訂用帳戶中的所有代管服務 (包括 Azure BizTalk 服務) 具有完整系統管理員權限。 BizTalk 系統管理員不是 Azure 帳戶持有者，因此無權存取任何任何計費資訊。
<br/><br/>
<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=267577">在 Azure 中管理訂用帳戶和儲存體帳戶</a>提供更多相關資訊。
</td>
</tr>
<tr>
<td>連接字串</td>
<td>儲存 Azure BizTalk 服務所使用的資料表、檢視和預存程序，包括追蹤資料。
<br/><br/>
在建立 BizTalk 服務時，您可使用現有的 Azure SQL Server、Azure SQL Database，或自動建立新的伺服器或資料庫。
<br/><br/>
系統會自動設定 SQL Database 調整。 一般來說，預設的調整對 BizTalk 服務已夠用。 修改調整會影響定價。 請參閱 <a HREF="http://go.microsoft.com/fwlink/p/?LinkID=234930">Azure SQL Database 中的帳戶和計費</a>
<br/><br/>
<strong>注意</strong>
<br/>
<ul>
<li> 當您建立新的 Azure SQL Server 和 Database 時，即會自動啟用 Azure 服務。 BizTalk 服務需要啟用 Azure 服務。</li>
<li>如果在現有的 Azure SQL Server 上建立新的 Azure SQL Database，則不會變更伺服器的防火牆規則。 因此，可能不允許其他 Azure 服務存取伺服器的資料庫。</li>
</ul>
</td>
</tr>
<tr>
<td>Azure 存取控制命名空間</td>
<td>利用 Azure BizTalk 服務進行驗證。 從 Visual Studio 部署 BizTalk 服務專案時，您可以輸入此存取控制命名空間。 當您建立 BizTalk 服務時，存取控制命名空間會自動建立。</td>
</tr>

<tr>
<td>Azure 儲存體帳戶</td>
<td>允許使用 BizTalk 服務所用的資料表、Blob 和佇列，以儲存下列檔案：

<ul>
<li>監視 BizTalk 服務的記錄檔。 </li>
<li>在夥伴之間建立 X12 或 AS2 協定時，您可以啟用封存功能來儲存訊息內容。 此資料儲存在此儲存體帳戶中。</li>
</ul>
<br/>
當建立 BizTalk 服務時，您可以使用現有的儲存體帳戶，或自動建立新的儲存體帳戶。
<br/><br/>
預設儲存體設定對 BizTalk 服務已夠用。
<br/><br/>
當建立儲存體帳戶時，會自動建立主要金鑰和次要金鑰。 這些金鑰可控制儲存體帳戶的存取。 BizTalk 服務會自動使用主要金鑰。
<br/><br/>
請參閱<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=285671">儲存體</a>以取得更多資訊。
</td>
</tr>

<tr>
<td>SSL 專用憑證</td>
<td>
當建立 Azure BizTalk 服務時，也會建立一個 HTTPS URL，其中包括 BizTalk 服務名稱。 此 URL 會自動設定為使用自我簽署的開發專用憑證。 若為生產案例，您需要私密的 SSL 憑證。
<br/><br/>
<strong>重要 SSL 憑證資訊</strong>


<ul>
<li>憑證到期日必須少於 5 年。</li>
<li>所有專用憑證都需要一個密碼。 知道此密碼，而且最佳作法為與系統管理員分享此密碼。</li>
<li>自我簽署憑證是在測試/開發環境中使用。 當使用自我簽署憑證時，請將憑證匯入至您的個人憑證存放區和受信任的根授權單位憑證存放區。</li>
</ul>
<br/>將實際執行憑證要求傳送至您的憑證授權單位時，請提供下列憑證內容：
<br/>

<ul>
<li><strong>增強金鑰使用方法</strong>：Azure BizTalk 服務至少需要伺服器驗證。</li>
<li><strong>一般名稱</strong>：輸入 Azure BizTalk 服務 URL 的完整網域名稱 (FQDN)。 請參閱本文中的<a HREF="#CreateService">建立 BizTalk 服務</a>。</li>
</ul>
<br/>
在建立 BizTalk 服務後，可以加入新的或不同的憑證。
</td>
</tr>
</table>
<!---Loc Comment: Please, check link [Create a BizTalk Service] since it is not redirecting to any location.--->



## <a name="hybrid-connections"></a>混合式連線
當您建立 Azure BizTalk 服務時，[ **混合式連線** ] 索引標籤可供使用：

![混合式連線索引標籤][HybridConnectionTab]

混合式連線可將 Azure 網站或 Azure 行動服務連線到使用靜態 TCP 連接埠的任何內部部署資源，例如 SQL Server、MySQL、HTTP Web API、行動服務及大多數的自訂 Web 服務。  混合式連線和 BizTalk 配接器服務不同。 BizTalk 配接器服務是用於將 Azure BizTalk 服務連線至內部部署企業營運 (LOB) 系統。

 查閱 [混合式連線](integration-hybrid-connection-overview.md) 以深入了解，其中包含建立和管理混合式連線。

## <a name="next-steps"></a>後續步驟
現在，既然已建立 BizTalk 服務，請讓自己熟悉一下各種不同的 [BizTalk 服務：儀表板、監視和調整索引標籤](biztalk-dashboard-monitor-scale-tabs.md)。 您的 BizTalk 服務已準備好可供您的應用程式使用。 若要開始建立應用程式，請移至 [Azure BizTalk 服務](http://go.microsoft.com/fwlink/p/?LinkID=235197)(英文)。

## <a name="see-also"></a>另請參閱
* [BizTalk 服務：版本圖表](biztalk-editions-feature-chart.md)<br/>
* [BizTalk 服務：狀態圖表](biztalk-service-state-chart.md)<br/>
* [BizTalk 服務：備份與還原](biztalk-backup-restore.md)<br/>
* [BizTalk 服務：節流](biztalk-throttling-thresholds.md)<br/>
* [BizTalk 服務：簽發者名稱和簽發者金鑰](biztalk-issuer-name-issuer-key.md)<br/>
* [如何開始使用 Azure BizTalk 服務 SDK](http://go.microsoft.com/fwlink/p/?LinkID=302335)<br/>
* [混合式連線](integration-hybrid-connection-overview.md)

[NewBizTalkService]: ./media/biztalk-provision-services/WABS_NewBizTalkService.png
[NEWButton]: ./media/biztalk-provision-services/WABS_New.png
[ProgressComplete]: ./media/biztalk-provision-services/WABS_ProgressComplete.png
[ACSConnectInfo]: ./media/biztalk-provision-services/WABS_ACSConnectInformation.png
[QuickGlance]: ./media/biztalk-provision-services/WABS_QuickGlance.png
[ACSServiceIdentities]: ./media/biztalk-provision-services/WABS_ACSServiceIdentities.png
[HybridConnectionTab]: ./media/biztalk-provision-services/WABS_HybridConnectionTab.png
