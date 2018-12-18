---
title: 安裝內部部署資料閘道 - Azure Logic Apps | Microsoft Docs
description: 您必須先下載並安裝內部部署資料閘道，才能從 Azure Logic Apps 存取內部部署資料
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: yshoukry, LADocs
ms.topic: article
ms.date: 07/20/2018
ms.openlocfilehash: 5fc4ccacaaedfc3fe6c77fa9a0ad693530bdde93
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48855420"
---
# <a name="install-on-premises-data-gateway-for-azure-logic-apps"></a>安裝 Azure Logic Apps 的內部部署資料閘道

您必須先在本機電腦下載並安裝內部部署資料閘道，才能從 Azure Logic Apps 連線至內部部署資料來源。 此閘道可作為橋樑，讓內部部署 (不在雲端) 資料來源與邏輯應用程式之間快速地傳輸和加密資料。 本文說明如何下載、安裝及設定您的內部部署資料閘道。 

您可以使用相同的閘道安裝搭配其他服務，例如 Power BI、Microsoft Flow、PowerApps 及 Azure Analysis Services。 深入了解[資料閘道的運作方式](#gateway-cloud-service)。

<a name="supported-connections"></a>

此閘道在 Azure Logic Apps 中針對下列資料來源支援[內部部署連接器](../connectors/apis-list.md#on-premises-connectors)︰

*   BizTalk Server 2016
*   檔案系統
*   IBM DB2  
*   IBM Informix
*   IBM MQ
*   MySQL
*   Oracle 資料庫
*   PostgreSQL
*   SAP 應用程式伺服器 
*   SAP 訊息伺服器
*   SharePoint Server
*   SQL Server
*   Teradata

如需如何使用閘道與其他服務的資訊，請參閱下列文章：

* [Microsoft Power BI 內部部署資料閘道](https://powerbi.microsoft.com/documentation/powerbi-gateway-onprem/)
* [Microsoft PowerApps 內部部署資料閘道](https://powerapps.microsoft.com/tutorials/gateway-management/)
* [Microsoft Flow 內部部署資料閘道](https://flow.microsoft.com/documentation/gateway-manage/)
* [Azure Analysis Services 內部部署資料閘道](../analysis-services/analysis-services-gateway.md)

<a name="requirements"></a>

## <a name="prerequisites"></a>必要條件

* 具有 [Azure 訂用帳戶](https://docs.microsoft.com/azure/architecture/cloud-adoption-guide/adoption-intro/subscription-explainer)的[公司或學校帳戶](../active-directory/fundamentals/sign-up-organization.md) 

  在閘道安裝期間，您會登入此帳戶，以便將閘道安裝與您的 Azure 訂用帳戶相關聯。 
  之後，在 Azure 入口網站中為您的閘道安裝建立 Azure 資源時，也會使用相同的帳戶。 
  如果您還沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。

* 您的本機電腦需求如下：

  **最低需求**
  * .NET Framework 4.5.2
  * 64 位元版本的 Windows 7 或 Windows Server 2008 R2 (或更新版本)

  **建議需求**
  * 8 核心 CPU
  * 8 GB 記憶體
  * 64 位元版本的 Windows Server 2012 R2 (或更新版本)

* **重要考量**

  * 您只可以在本機電腦上安裝內部部署資料閘道，而無法在網域控制站上安裝。 不過，閘道不必安裝在和資料來源相同的電腦上。 此外，所有資料來源只需要一個閘道，因此不需要針對每個資料來源安裝閘道。

    > [!TIP]
    > 若要盡量減少延遲，您可以將閘道安裝在最靠近資料來源的位置或同一部電腦上，但前提是您有相關權限。

  * 將閘道安裝在連線至網際網路、一律開啟且「不會」進入睡眠模式的電腦上。 否則，閘道將無法執行。 此外，透過無線網路的效能可能會受到影響。

  * 在安裝期間，您只能以[公司或學校帳戶](../active-directory/sign-up-organization.md)登入，且該帳戶必須是由 Azure Active Directory (Azure AD) 所管理，而非 Microsoft 帳戶。 
  此外，請確定此帳戶不是 Azure B2B (來賓) 帳戶。 
  當您為閘道建立 Azure 資源以便註冊閘道安裝時，也必須在 Azure 入口網站中使用相同的登入帳戶。 
  接著，在建立從邏輯應用程式到內部部署資料來源的連線時，可以選取此閘道資源。 
  [為何必須使用公司或學校的 Azure AD 帳戶？](#why-azure-work-school-account)

  > [!TIP]
  > 如果您已註冊 Office 365 供應項目，但未提供實際的公司電子郵件，您的登入地址會如此範例所示：`username@domain.onmicrosoft.com` 
  >
  > 若要使用具有 [Visual Studio 標準訂用帳戶](https://visualstudio.microsoft.com/vs/pricing/)的 Microsoft 帳戶，請先使用您的 Microsoft 帳戶，[在 Azure Active Directory 中建立一個目錄 (租用戶)](../active-directory/develop/quickstart-create-new-tenant.md)，或使用預設目錄。 
  > 請將使用者和密碼新增至目錄，然後將您訂用帳戶的存取權授與該使用者。 
  > 接著，您就可以在安裝閘道時，使用此使用者名稱和密碼來進行登入。

  * 為您的閘道安裝選取的區域，可藉由建立 Azure 資源來決定您稍後在 Azure 中註冊閘道的位置。 
  當您在 Azure 中建立此閘道資源時，必須選取與閘道安裝「相同」的位置。 預設區域是與 Azure AD 租用戶 (用來管理您的 Azure 帳戶) 相同的位置，但您可以在閘道安裝期間變更此位置。

  * 如果您已經有使用 14.16.6317.4 版之前的安裝程式所設定的閘道，就無法藉由執行最新版的安裝程式來變更閘道位置。 不過，您可以使用最新版的安裝程式，將新的閘道設定為您想要的位置。
  
    如果您的閘道安裝程式版本比 14.16.6317.4 還舊，但尚未安裝閘道，則可以下載並使用最新版的安裝程式。

<a name="install-gateway"></a>

## <a name="install-data-gateway"></a>安裝資料閘道

1. [在本機電腦上下載、儲存並執行閘道安裝程式](https://aka.ms/on-premises-data-gateway-installer)。

2. 接受預設安裝路徑，或指定您要用來安裝閘道的電腦位置。

3. 檢閱並接受使用規定和隱私權聲明，然後選擇 [安裝]。

   ![接受使用規定和隱私權聲明](./media/logic-apps-gateway-install/accept-terms.png)

4. 成功安裝閘道之後，請提供公司或學校帳戶的電子郵件地址，然後選擇 [登入]。

   ![使用公司或學校帳戶登入](./media/logic-apps-gateway-install/sign-in-gateway-install.png)

5. 選擇 [在這部電腦上註冊新的閘道] > [下一步]，即可透過[閘道雲端服務](#gateway-cloud-service)註冊您的閘道安裝。 

   ![註冊閘道](./media/logic-apps-gateway-install/register-new-gateway.png)

6. 為您的閘道安裝提供下列資訊：

   * 您想要的安裝名稱 

   * 您想要建立的修復金鑰，該金鑰必須至少有八個字元

     > [!IMPORTANT]
     > 將您的修復金鑰儲存並保留在安全的地方。 當您變更閘道的位置，或遷移、復原或取代現有閘道時，需要此金鑰。

   * 確認您的修復金鑰 

     ![設定閘道](./media/logic-apps-gateway-install/set-up-gateway.png)

7. 檢查針對閘道安裝所使用的閘道雲端服務和 Azure 服務匯流排選取的區域。 

   ![檢查區域](./media/logic-apps-gateway-install/check-region.png)

   > [!IMPORTANT]
   > 若要在完成安裝閘道之後變更此區域，您需要該閘道安裝的修復金鑰。 此外，您必須將閘道解除安裝後再重新安裝。 如需詳細資訊，請參閱[變更位置、遷移、復原或取代現有閘道](#update-gateway-installation)。

   *為什麼要變更閘道安裝的區域？* 

   例如，若要減少延遲，您可以將閘道的區域變更為與邏輯應用程式相同的區域。 
   或者，您可以選取最接近的內部部署資料來源區域。 
   「Azure 中的閘道資源」和邏輯應用程式可位於不同位置。

8. 若要接受預設區域，請選擇 [設定]。 或者，若要變更預設區域，請遵循下列步驟：

   1. 在目前區域旁邊，選取 [變更區域]。 

      ![變更區域](./media/logic-apps-gateway-install/change-region.png)

   2. 在下一頁上，開啟 [選取區域] 清單，選取您想要的區域，然後選擇 [完成]。

      ![選取其他區域](./media/logic-apps-gateway-install/select-region-gateway-install.png)

9. 在確認頁面出現之後，請選擇 [關閉]。 

   安裝程式會確認您的閘道現在已上線，而且可供使用。

   ![已完成的閘道](./media/logic-apps-gateway-install/finished-gateway-default-location.png)

10. [為閘道安裝建立 Azure 資源](../logic-apps/logic-apps-gateway-connection.md)，立即在 Azure 中註冊您的閘道。 

## <a name="enable-high-availability"></a>啟用高可用性

當您有一個以上的閘道安裝並將它們設定為叢集時，內部部署資料閘道可支援高可用性。 當您要建立另一個閘道時，如果有現有的閘道，可以選擇地建立高可用性叢集。 這些叢集會將閘道器組織成群組，有助於避免出現單一失敗點。 若要使用這項功能，請檢閱下列需求和考量：

* 只有某些連接器支援高可用性，例如檔案系統連接器和其他同類的連接器。 
     
* 在與主要閘道相同的 Azure 訂用帳戶內，您必須已經有至少一個閘道安裝，以及該安裝的修復金鑰。 

* 主要閘道必須執行 2017 年 11 月或更新版本的閘道更新。

符合這些需求之後，當您建立下一個閘道時，請選取 [新增至現有的閘道叢集]，為您的叢集選取主要閘道，然後提供該主要閘道的修復金鑰。
如需詳細資訊，請參閱[適用於內部部署資料閘道的高可用性叢集](https://docs.microsoft.com/power-bi/service-gateway-high-availability-clusters)。

<a name="update-gateway-installation"></a>

## <a name="change-location-migrate-restore-or-take-over-existing-gateway"></a>變更位置、遷移、還原或取代現有閘道

如果您必須變更閘道的位置、閘道安裝移至新的電腦、復原受損的閘道，或取得現有閘道的擁有權，您會需要在閘道安裝期間所提供的修復金鑰。 這個動作會中斷舊閘道的連線。

1. 從電腦的 [控制台]，移至 [程式和功能]。 在程式清單中，選取 [內部部署資料閘道]，然後選擇 [解除安裝]。

2. [重新安裝內部部署資料閘道](https://aka.ms/on-premises-data-gateway-installer)。

3. 安裝程式開啟後，以先前用來安裝閘道的同一個公司或學校帳戶進行登入。

4. 選取 [遷移、還原或取代現有閘道]，然後選擇 [下一步]。

   ![選取 [遷移、還原或取代現有閘道]](./media/logic-apps-gateway-install/migrate-recover-take-over-gateway.png)

5. 在 [可用的閘道] 或 [可用的閘道叢集] 之下，選取您想要變更的閘道安裝。 輸入閘道安裝的修復金鑰。 

   ![選取主要閘道](./media/logic-apps-gateway-install/select-existing-gateway.png)

6. 若要變更區域，請選取 [變更區域] 和新的區域。

7. 完成之後，請選擇 [設定]。

## <a name="configure-proxy-or-firewall"></a>設定 Proxy 或防火牆

內部部署資料閘道會對 [Azure 服務匯流排](https://azure.microsoft.com/services/service-bus/)建立輸出連線。 如果您的工作環境要求流量經由 Proxy 存取網際網路，此限制可能會使資料閘道無法連線到閘道雲端服務。 若要判斷您的網路是否使用 Proxy，請檢閱 superuser.com 上的這篇文章： 

[如何知道我使用的 Proxy 伺服器為何？(SuperUser.com)](https://superuser.com/questions/346372/how-do-i-know-what-proxy-server-im-using) 

若要為閘道提供 Proxy 資訊，請參閱[設定 Proxy 設定](https://docs.microsoft.com/power-bi/service-gateway-proxy)。 若要檢查 Proxy 或防火牆是否會封鎖連線，請確認電腦確實可以連線到網際網路和 [Azure 服務匯流排](https://azure.microsoft.com/services/service-bus/)。 從 PowerShell 命令提示字元中，執行此命令︰

`Test-NetConnection -ComputerName watchdog.servicebus.windows.net -Port 9350`

> [!NOTE]
> 此命令只會測試網路連線和對 Azure 服務匯流排的連線。 這個命令和閘道或閘道雲端服務 (會加密並儲存認證和閘道詳細資料) 沒有任何關係。 
>
> 此外，這個命令僅適用於 Windows Server 2012 R2 或更新版本，以及 Windows 8.1 或更新版本。 對於較舊的作業系統版本，您可以使用 Telnet 來測試連線能力。 深入了解 [Azure 服務匯流排和混合式解決方案](../service-bus-messaging/service-bus-messaging-overview.md)。

**TcpTestSucceeded** 設定為 **True** 的結果應該類似此範例：

```text
ComputerName           : watchdog.servicebus.windows.net
RemoteAddress          : 70.37.104.240
RemotePort             : 5672
InterfaceAlias         : vEthernet (Broadcom NetXtreme Gigabit Ethernet - Virtual Switch)
SourceAddress          : 10.120.60.105
PingSucceeded          : False
PingReplyDetails (RTT) : 0 ms
TcpTestSucceeded       : True
```

如果 **TcpTestSucceeded** 未設為 **True**，防火牆可能會封鎖您的閘道。 如果您想要更詳細的資訊，請將 **ComputerName** 和 **Port** 值取代為本文的[設定連接埠](#configure-ports)之下所列的值。

防火牆也可能會封鎖 Azure 服務匯流排到 Azure 資料中心的連線。 若發生此情況，請核准 (解除封鎖) 您所在區域之資料中心的所有 IP 位址。 如需這些 IP 位址，您可以[在此取得 Azure IP 位址清單](https://www.microsoft.com/download/details.aspx?id=41653)。

## <a name="configure-ports"></a>設定連接埠

閘道會建立對 [Azure 服務匯流排](https://azure.microsoft.com/services/service-bus/)的輸出連線，並與下列輸出連接埠進行通訊︰TCP 443 (預設)、5671、5672 和 9350 到 9354。 閘道不需要輸入連接埠。 深入了解 [Azure 服務匯流排和混合式解決方案](../service-bus-messaging/service-bus-messaging-overview.md)。

閘道會使用這些完整的網域名稱：

| 網域名稱 | 輸出連接埠 | 說明 | 
| ------------ | -------------- | ----------- | 
| *.analysis.windows.net | 443 | HTTPS | 
| *.core.windows.net | 443 | HTTPS | 
| *.frontend.clouddatahub.net | 443 | HTTPS | 
| *.login.windows.net | 443 | HTTPS | 
| *.microsoftonline-p.com | 443 | 用於驗證 (視設定而定)。 | 
| *.msftncsi.com | 443 | 當 Power BI 服務無法連線至閘道時，用來測試網際網路連線能力。 | 
| *.servicebus.windows.net | 443、9350-9354 | 透過 TCP 之服務匯流排轉送上的接聽程式 (需要 443 才能取得「存取控制」權杖) | 
| *.servicebus.windows.net | 5671-5672 | 進階訊息佇列通訊協定 (AMQP) | 
| login.microsoftonline.com | 443 | HTTPS | 
||||

在某些情況下，Azure 服務匯流排連線會使用 IP 位址而非完整網域名稱來建立。 所以，建議在防火牆中將您的資料區域 IP 位址列入允許清單。 若要將 IP 位址 (而不是網域) 加入允許清單，您可以下載並使用 [Microsoft Azure Datacenter IP 範圍清單](https://www.microsoft.com/download/details.aspx?id=41653)。 此清單中的 IP 位址必須採用[無類別網域間路由 (CIDR)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) 標記法。

### <a name="force-https-communication-with-azure-service-bus"></a>強制使用 Azure 服務匯流排進行 HTTPS 通訊

某些 Proxy 只允許送至連接埠 80 和 443 的流量。 根據預設，與 Azure 服務匯流排的通訊會發生於 443 以外的連接埠上。
您可以強制閘道透過 HTTPS (而非直接 TCP ) 來與 Azure 服務匯流排通訊，但是這麼做會大幅降低效能。 若要執行這項工作，請按照下列步驟進行：

1. 瀏覽至內部部署資料閘道用戶端的位置，您通常可以在此找到該位置：```C:\Program Files\On-premises data gateway\Microsoft.PowerBI.EnterpriseGateway.exe```

   否則，若要尋找用戶端位置，請在同一部電腦上開啟 [服務] 主控台，尋找 [內部部署資料閘道服務]，然後檢視 [可執行檔的路徑] 屬性。

2. 開啟這個「組態」檔案：**Microsoft.PowerBI.DataMovement.Pipeline.GatewayCore.dll.config**

3. 將 **ServiceBusSystemConnectivityModeString** 值從 **AutoDetect** 變更為 **Https**：

   ```html
   <setting name="ServiceBusSystemConnectivityModeString" serializeAs="String">
      <value>Https</value>
   </setting>
   ```

<a name="windows-service-account"></a>

## <a name="windows-service-account"></a>Windows 服務帳戶

內部部署資料閘道會以名為「內部部署資料閘道服務」的 Windows 服務形式執行，但使用 "NT SERVICE\PBIEgwService" 作為其「登入身分」帳戶認證。 根據預設，內部部署資料閘道對於您安裝閘道的電腦擁有「以服務方式登入」權限。 若要在 Azure 入口網站中建立和維護閘道，Windows 服務帳戶必須至少具備**參與者**權限。 

> [!NOTE]
> Windows 服務帳戶與用來連線至內部部署資料來源的帳戶不同，也與用來登入雲端服務的公司或學校帳戶不同。

<a name="restart-gateway"></a>

## <a name="restart-gateway"></a>重新啟動閘道

資料閘道會以 Windows 服務的形式來執行，因此和任何其他 Windows 服務一樣，您可以透過多種方式啟動和停止閘道。 例如，您可以在閘道執行所在的電腦上以提高的權限開啟命令提示字元，並執行下列其中一個命令︰

* 若要停止服務，請執行此命令：
  
  `net stop PBIEgwService`

* 若要啟動服務，請執行此命令：
  
  `net start PBIEgwService`

## <a name="tenant-level-administration"></a>租用戶層級管理 

針對其他使用者已安裝並設定的所有閘道，租用戶系統管理員目前無法從單一位置來管理這些閘道。 如果您是租用戶系統管理員，您可以讓組織內的使用者將您新增為所安裝每個閘道的系統管理員。 如此一來，您即可透過 [閘道設定] 頁面或透過 [PowerShell 命令](https://docs.microsoft.com/power-bi/service-gateway-high-availability-clusters#powershell-support-for-gateway-clusters)管理組織中的所有閘道。 

<a name="gateway-cloud-service"></a>

## <a name="how-does-the-gateway-work"></a>閘道如何運作？

資料閘道可讓邏輯應用程式、閘道雲端服務和內部部署資料來源之間，快速且安全地進行通訊。 閘道雲端服務會將資料來源認證和閘道詳細資料予以加密並儲存。 此服務也會在邏輯應用程式、內部部署資料閘道和內部部署資料來源之間路由傳送查詢和其結果。 

閘道可搭配防火牆運作，而且僅使用輸出連線。 源自閘道代理程式的所有流量都是安全輸出流量。 閘道會在加密通道上經過 Azure 服務匯流排轉送來自內部部署來源的資料。 此服務匯流排會建立閘道與呼叫服務之間的通道，但不會儲存任何資料。 透過閘道傳送的所有資料都會加密。

![diagram-for-on-premises-data-gateway-flow](./media/logic-apps-gateway-install/how-on-premises-data-gateway-works-flow-diagram.png)

下列步驟說明當雲端使用者與連線到內部部署資料來源的元素互動時，會發生什麼狀況︰

1. 閘道雲端服務會建立查詢，以及資料來源的加密認證，並將查詢傳送至閘道的佇列處理。

2. 閘道雲端服務會分析該查詢，並將要求推送至 Azure 服務匯流排。

3. 內部部署資料閘道會輪詢 Azure 服務匯流排是否有待處理的要求。

4. 閘道收到查詢、解密認證，並使用這些認證連線至資料來源。

5. 閘道將查詢傳送至資料來源執行。

6. 結果會從資料來源傳送回閘道，然後再到閘道雲端服務。 閘道雲端服務接著就會使用結果。

<a name="faq"></a>

## <a name="frequently-asked-questions"></a>常見問題集

### <a name="general"></a>一般

**問**︰我在雲端中的資料來源 (例如 Azure SQL Database) 是否需要閘道？ <br/>
**答**：否，閘道只會連線至內部部署資料來源。

**問**︰閘道必須安裝在與資料來源相同的電腦上嗎？ <br/>
**答**：否，閘道會使用所提供的連線資訊來連線到資料來源。 在此請將閘道視為用戶端應用程式。 閘道只需要能夠連線到所提供的伺服器名稱。

<a name="why-azure-work-school-account"></a>

**問︰**︰為何必須使用公司或學校帳戶進行登入？ <br/>
**答**︰安裝內部部署資料閘道時，您只能使用公司或學校帳戶。 登入帳戶會儲存在由 Azure Active Directory (Azure AD) 所管理的租用戶中。 Azure AD 帳戶的使用者主體名稱 (UPN) 通常會與電子郵件地址相符。

**問**︰我的認證儲存在哪裡？ <br/>
**答**：您針對資料來源輸入的認證會以加密方式儲存在閘道雲端服務中。 認證會在內部部署資料閘道進行解密。

**問**︰網路頻寬是否有任何需求？ <br/>
**答**︰檢查您的網路連線有良好的輸送量。 每個環境都不同，但是傳送的資料量會影響結果。 若要保證內部部署資料來源與 Azure 資料中心之間有一定的輸送量等級，請嘗試 [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/)。 若要協助測量您的輸送量，請嘗試 Azure 速度測試之類的外部工具。

**問**︰從閘道向資料來源執行查詢時的延遲為何？ 最佳的架構為何？ <br/>
**答**：若要減少網路延遲，請盡可能靠近資料來源安裝閘道。 如果您可以在實際的資料來源上安裝閘道，此鄰近程度就能將所產生的延遲降到最低。 此外，請考慮 Azure 資料中心的鄰近程度。 例如，如果您的服務使用美國西部資料中心，而且在 Azure VM 中裝載了 SQL Server，則您會希望 Azure VM 也位於美國西部。 此鄰近程度能將延遲降到最低，並避免 Azure VM 的輸出費用。

**問**︰結果如何傳送回雲端？ <br/>
**答**︰結果會透過 Azure 服務匯流排傳送。

**問**︰是否有任何從雲端至閘道的輸入連線？ <br/>
**答**︰否，閘道使用連至 Azure 服務匯流排的輸出連線。

**問**︰如果我封鎖輸出連線會如何？ 我需要開啟什麼嗎？ <br/>
**答**：查看閘道使用的連接埠和主機。

**問**：實際的 Windows 服務名稱為何？ <br/>
**答**：在工作管理員的 [服務] 索引標籤中，服務名稱為 "PBIEgwService" 或 Power BI Enterprise Gateway Service。 在 [服務] 主控台中，服務名稱為「內部部署資料閘道服務」。 Windows 服務會使用 "NT SERVICE\PBIEgwService" 作為服務 SID (SSID)。

**問**︰閘道 Windows 服務可以使用 Azure Active Directory 帳戶執行嗎？ <br/>
**答**：否，Windows 服務必須要有有效的 Windows 帳戶。

### <a name="disaster-recovery"></a>災害復原

**問**︰災害復原有哪些選項？ <br/>
**答**：您可以使用修復金鑰還原或移動閘道。 當您安裝閘道時，請指定修復金鑰。

**問**︰修復金鑰有什麼好處？ <br/>
**答**：修復金鑰可在災害發生後讓您有辦法移轉或復原閘道設定。

## <a name="troubleshooting"></a>疑難排解

本節說明您在設定及使用內部部署資料閘道時可能遇到的一些常見問題。

**問：**：我的閘道安裝為何失敗？ <br/>
**答**：如果目標電腦上的防毒軟體已過期，就可能發生此問題。 您可以更新防毒軟體或停用防毒軟體，但僅限於閘道安裝期間，然後再次啟用該軟體。

**問**：為何在 Azure 中建立閘道資源時看不到我的閘道安裝？ <br/>
**答**：發生此問題的原因如下：

* 您的閘道安裝已經由 Azure 中的其他閘道資源加以註冊及宣告。 建立閘道資源之後，閘道安裝就不會出現在執行個體清單中。
若要在 Azure 入口網站中檢查您的閘道註冊，請使用 [內部部署資料閘道] 類型，檢閱「所有」 Azure 訂用帳戶的所有 Azure 資源。 

* 安裝閘道的人員的 Azure AD 身分識別與登入 Azure 入口網站的人員不同。 請檢查您已使用安裝閘道的相同身分識別進行登入。

[!INCLUDE [existing-gateway-location-changed](../../includes/logic-apps-existing-gateway-location-changed.md)]

**問**︰閘道記錄在哪裡？ <br/>
**答**：請參閱本文稍後的[**記錄**一節](#logs)。

**問**︰如何查看有哪些查詢正要傳送至內部部署資料來源？ <br/>
**答**：您可以啟用查詢追蹤，其中包含要傳送的查詢。 完成疑難排解時，請務必將查詢追蹤變更回原始值。 讓查詢追蹤保持開啟會產生較大的記錄。

您也可以查看資料來源具備的工具是否有追蹤查詢。 例如，您可以使用 SQL Server 和 Analysis Services 的擴充事件或 SQL Profiler。

### <a name="outdated-gateway-version"></a>過期的閘道版本

閘道版本過期時，可能會出現很多問題。 良好的一般做法是確定您有最新版本。 如果有一個月以上未更新閘道，您可以考慮安裝最新版的閘道，並看看是否能重現問題。

### <a name="error-failed-to-add-user-to-group--2147463168-pbiegwservice-performance-log-users"></a>錯誤︰無法將使用者新增到群組。 (-2147463168 PBIEgwService Performance Log Users)

如果您嘗試在不支援的網域控制站上安裝閘道，可能會收到這個錯誤。 請確定您是在非網域控制站的電腦上部署閘道。

<a name="logs"></a>

### <a name="logs"></a>記錄檔

若要協助進行疑難排解，請一律從收集並檢視閘道記錄著手。 您有幾種方式可收集記錄，但是在您安裝閘道之後，最簡單的選項是透過閘道安裝程式的使用者介面。 

1. 在電腦上，開啟內部部署資料閘道安裝程式。
2. 在左側功能表上，選取 [診斷]。
3. 在 [閘道記錄] 之下，選取 [匯出記錄]。

   ![從閘道安裝程式匯出記錄](./media/logic-apps-gateway-install/export-logs.png)

以下是您可以找到各種記錄的其他位置：

| 記錄類型 | 位置 | 
|----------|----------| 
| **安裝程式記錄** | %localappdata%\Temp\On-premises_data_gateway_<*yyyymmdd*>.<*number*>.log | 
| **組態記錄** | C:\Users\<*username*>\AppData\Local\Microsoft\On-premises data gateway\GatewayConfigurator<*yyyymmdd*>.<*number*>.log | 
| **企業閘道服務記錄** | C:\Users\PBIEgwService\AppData\Local\Microsoft\On-premises data gateway\Gateway<*yyyymmdd*>.<*number*>.log | 
||| 

**事件記錄**

若要尋找閘道的事件記錄，請遵循下列步驟：

1. 在具有閘道安裝的電腦上，開啟 [事件檢視器]。 
2. 展開 [事件檢視器 (本機)] > [應用程式和服務記錄]。 
3. 選取 [內部部署資料閘道服務]。

   ![檢視閘道的事件記錄](./media/logic-apps-gateway-install/event-viewer.png)

### <a name="telemetry"></a>遙測

如需額外的監視和疑難排解，您可以開啟並收集遙測。 

1. 瀏覽至內部部署資料閘道用戶端的位置，您通常可以在此找到該位置：```C:\Program Files\On-premises data gateway```

   否則，若要尋找用戶端位置，請在同一部電腦上開啟 [服務] 主控台，尋找 [內部部署資料閘道服務]，然後檢視 [可執行檔的路徑] 屬性。

2. 開啟這個「組態」檔案：**Microsoft.PowerBI.DataMovement.Pipeline.GatewayCore.dll.config**

3. 將 **SendTelemetry** 值變更為 **true**：

   ```html
   <setting name="SendTelemetry" serializeAs="String">
      <value>true</value>
   </setting>
   ```

4. 儲存您的變更，然後重新啟動 Windows 服務。

### <a name="review-slow-query-performance"></a>檢閱緩慢查詢效能

如果您發現透過閘道執行的查詢速度很慢，您可以開啟額外記錄來輸出查詢及其持續時間。 這些記錄可以協助您找出哪些查詢緩慢或長時間執行。 若要調整查詢效能，您可能必須修改您的資料來源，例如，調整 SQL Server 查詢的索引。

若要判斷查詢的持續時間，請遵循下列步驟：

1. 瀏覽至與閘道用戶端相同的位置，您通常可以在此找到該位置：```C:\Program Files\On-premises data gateway```

   否則，若要尋找用戶端位置，請在同一部電腦上開啟 [服務] 主控台，尋找 [內部部署資料閘道服務]，然後檢視 [可執行檔的路徑] 屬性。

2. 依照描述開啟和編輯下列組態檔：

   * **Microsoft.PowerBI.DataMovement.Pipeline.GatewayCore.dll.config**

     在此檔案中，將 **EmitQueryTraces** 值從 **false** 變更為 **true**，讓您的閘道可以記錄從閘道傳送到資料來源的查詢：

     ```html
     <setting name="EmitQueryTraces" serializeAs="String">
        <value>true</value>
     </setting>
     ```

     > [!IMPORTANT]
     > 開啟 EmitQueryTraces 設定，可能會大幅增加以閘道使用量為基礎的記錄大小。 完成記錄檢閱之後，確定您再次將 EmitQueryTraces 重設為 **false**，而不是將此設定長期保持開啟狀態。

   * **Microsoft.PowerBI.DataMovement.Pipeline.Diagnostics.dll.config**

     若要擁有您的閘道記錄詳細資訊項目，包括顯示持續時間的項目，請執行下列任一步驟，將 **TracingVerbosity** 值從 **4** 變更為 **5**： 

     * 在此組態檔中，將 **TracingVerbosity** 值從 **4** 變更為 **5** 

       ```html
       <setting name="TracingVerbosity" serializeAs="String">
          <value>5</value>
       </setting>
       ```

     * 開啟閘道安裝程式，選取 [診斷]，開啟 [額外記錄]，然後選擇 [套用]：

       ![開啟額外記錄](./media/logic-apps-gateway-install/turn-on-additional-logging.png)

     > [!IMPORTANT]
     > 開啟 TracingVerbosity 設定，可能會大幅增加以閘道使用量為基礎的記錄大小。 完成記錄檢閱之後，請務必在閘道安裝程式中關閉 [額外記錄]或再次於組態檔中將 TracingVerbosity 重設為 **4**，而不是將此設定長期保留開啟狀態。

3. 若要尋找查詢的持續時間，請遵循下列步驟：

   1. [匯出](#logs)並開啟閘道記錄。

   2. 若要尋找查詢，請搜尋活動類型，例如： 

      | 活動類型 | 說明 | 
      |---------------|-------------| 
      | MGEQ | 透過 ADO.NET 執行的查詢。 | 
      | MGEO | 透過 OLEDB 執行的查詢。 | 
      | MGEM | 從 Mashup 引擎執行的查詢。 | 
      ||| 

   3. 請注意第二個 GUID，這是要求識別碼。

   4. 繼續搜尋活動類型，直到您找到名為 "FireActivityCompletedSuccessfullyEvent" 並以毫秒計算持續期間的項目。 
   確認此項目具有相同的要求識別碼，例如：

      ```text 
      DM.EnterpriseGateway Verbose: 0 : 2016-09-26T23:08:56.7940067Z DM.EnterpriseGateway    baf40f21-2eb4-4af1-9c59-0950ef11ec4a    5f99f566-106d-c8ac-c864-c0808c41a606    MGEQ    21f96cc4-7496-bfdd-748c-b4915cb4b70c    B8DFCF12 [DM.Pipeline.Common.TracingTelemetryService] Event: FireActivityCompletedSuccessfullyEvent (duration=5004)
      ```

      > [!NOTE] 
      > "FireActivityCompletedSuccessfullyEvent" 項目是詳細資訊項目，除非 "TracingVerbosity" 設定位於層級 5 才會記錄。

### <a name="trace-traffic-with-fiddler"></a>使用 Fiddler 追蹤流量

[Fiddler](http://www.telerik.com/fiddler) 是一個 Telerik 公司開發，用來監視 HTTP 流量的免費工具。 您可以從用戶端電腦，使用 Power BI 服務來查閱此流量。 此服務可能會顯示錯誤和其他相關資訊。

## <a name="next-steps"></a>後續步驟
    
* [從邏輯應用程式連線至內部部署資料](../logic-apps/logic-apps-gateway-connection.md)
* [企業整合功能](../logic-apps/logic-apps-enterprise-integration-overview.md)
* [適用於 Azure Logic Apps 的連接器](../connectors/apis-list.md)
