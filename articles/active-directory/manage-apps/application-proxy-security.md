---
title: Azure AD 應用程式 Proxy 的安全性考量 | Microsoft Docs
description: 涵蓋使用 Azure AD 應用程式 Proxy 的安全性考量
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
ms.date: 09/08/2017
ms.author: barbkess
ms.reviewer: harshja
ms.custom: it-pro
ms.openlocfilehash: 985ea1f16cff010041d61d808280cb47f2b77aa9
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39618354"
---
# <a name="security-considerations-for-accessing-apps-remotely-with-azure-ad-application-proxy"></a>使用 Azure AD 應用程式 Proxy 遠端存取應用程式的安全性考量

本文說明能在使用 Azure Active Directory 應用程式 Proxy 時保護使用者和應用程式安全的元件。

下圖顯示 Azure AD 如何讓您在內部部署應用程式實現安全的遠端存取。

 ![透過 Azure AD 應用程式 Proxy 進行安全遠端存取的示意圖](./media/application-proxy-security/secure-remote-access.png)

## <a name="security-benefits"></a>安全性優點

Azure AD 應用程式 Proxy 提供下列安全性優點︰

### <a name="authenticated-access"></a>已驗證的存取 

如果您選擇使用 Azure Active Directory 預先驗證，則只有已驗證的連線可以存取您的網路。

Azure AD 應用程式 Proxy 依賴 Azure AD Security Token Service (STS) 來進行所有驗證。  預先驗證 (就其本質) 會封鎖大量匿名攻擊，因為只有已驗證的身分識別可以存取後端應用程式。

如果您選擇 Passthrough 作為預先驗證方法，就無法獲得這項優點。 

### <a name="conditional-access"></a>條件式存取

在建立您的網路連線之前，先套用更豐富的原則控制。

使用[條件式存取](../conditional-access/overview.md)，就可以定義允許哪些流量存取後端應用程式上的限制。 您可以位置、驗證強度和使用者風險狀況作為基礎，來建立限制登入的原則。

您也可以使用條件式存取來設定 Multi-Factor Authentication 原則，為您的使用者驗證新增另一層的安全性。 

### <a name="traffic-termination"></a>流量終止

終止雲端中所有的流量。

Azure AD 應用程式 Proxy 是反向 Proxy，因此所有至後端應用程式的流量會在服務終止。 工作階段只能利用後端伺服器來重新建立，也就是說，後端伺服器不會對直接的 HTTP 流量公開。 此設定表示，您會受到更妥善的保護，可免於目標型攻擊。

### <a name="all-access-is-outbound"></a>所有存取都是輸出 

不需要開啟連往公司網路的輸入連線。

應用程式 Proxy 連接器只會使用連往 Azure AD 應用程式 Proxy 服務的輸出連線；亦即，您不需要開啟防火牆連接埠以供連入連線使用。 傳統 Proxy 需要周邊網路 (也稱為「DMZ」、「周邊網路」或「遮蔽式子網路」) 並在網路邊緣允許未經授權連線的存取權。 這種情節需要投資 Web 應用程式防火牆產品，以便分析流量及保護環境。 使用應用程式 Proxy，您就不需要周邊網路，因為所有連線皆為輸出方向，並且是透過安全通道來傳輸。

如需連接器的詳細資訊，請參閱[了解 Azure AD 應用程式 Proxy 連接器](application-proxy-connectors.md)。

### <a name="cloud-scale-analytics-and-machine-learning"></a>雲端級別分析與機器學習 

取得最新的安全性保護。

因為應用程式 Proxy 是 Azure Active Directory 的一部分，所以可以利用 [Azure AD Identity Protection](../active-directory-identityprotection.md)，其中包含來自 Microsoft Security Response Center 和 Digital Crimes Unit 的資料。 我們共同主動識別遭入侵的帳戶，並提供來自高風險登入的防護。我們將許多因素納入考量，藉以判斷哪些登入嘗試具有高度風險。 這些因素包括標記受感染的裝置、匿名網路，以及非典型或假位置。

這些報告和事件中有許多已可透過 API 與安全性資訊和事件管理 (SIEM) 系統整合。

### <a name="remote-access-as-a-service"></a>遠端存取即服務

您不必擔心維護及修補內部部署伺服器的事宜。

未更新的軟體仍需負責處理大量攻擊。 Azure AD 應用程式 Proxy 是 Microsoft 自有的網際網路級別服務，因此您永遠會獲得最新的安全性修補程式和升級。

為了改善 Azure AD 應用程式 Proxy 所發佈應用程式的安全性，我們會封鎖 Web 編目程式傀儡程式，使其無法對您的應用程式編製索引和進行保存。 每次 Web 編目程式傀儡程式嘗試擷取已發佈應用程式的傀儡程式設定時，應用程式 Proxy 會以含有 `User-agent: * Disallow: /` 的 robots.txt 檔案回覆。

### <a name="ddos-prevention"></a>DDOS 預防

透過應用程式 Proxy 發佈應用程式會受到保護，能抵禦分散式阻斷服務 (DDOS) 攻擊。

應用程式 Proxy 服務會監視嘗試連接應用程式和網路的流量。 如果要求從遠端存取應用程式的裝置數目升高，Microsoft 會調節網路的存取。 

Microsoft 會監看個別應用程式和整個訂用帳戶的傳輸模式。 如果某個應用程式收到高於一般的要求，存取該應用程式的要求會在一段不久時間內遭到拒絕。 如果收到的要求超過整個訂用帳戶的一般要求數目，則要求會遭拒而無法存取您的任何應用程式。 這個預防措施可防止應用程式伺服器因遠端存取要求而超出負載，讓內部部署使用者能持續存取他們的應用程式。 

## <a name="under-the-hood"></a>幕後

Azure AD 應用程式 Proxy 是由兩個部分組成︰

* 雲端架構服務︰此服務會在 Azure 中執行，且外部用戶端/使用者會連線到此服務。
* [內部部署連接器](application-proxy-connectors.md)︰這是內部部署元件，此連接器會接聽來自 Azure AD 應用程式 Proxy 服務的要求，並處理對內部應用程式的連線。 

以下為建立連接器與應用程式 Proxy 服務之間流量的時機︰

* 第一次設定連接器。
* 連接器會從應用程式 Proxy 服務提取設定資訊。
* 使用者會存取發佈的應用程式。

>[!NOTE]
>所有通訊會透過 SSL 發生，且一律源自於至應用程式 Proxy 服務的連接器。 此服務只會輸出。

連接器會使用用戶端憑證來驗證幾乎所有呼叫的應用程式 Proxy 服務。 此程序的唯一例外是可供建立用戶端憑證的初始設定步驟。

### <a name="installing-the-connector"></a>安裝連接器

第一次設定連接器時，會發生下列流程事件：

1. 連接器註冊服務會做為連接器安裝的一部分。 系統會提示使用者輸入其 Azure AD 系統管理員認證。 接著會向 Azure AD 應用程式 Proxy 服務顯示此驗證的必要權杖。
2. 應用程式 Proxy 服務會評估權杖。 它會檢查使用者在租用戶是否為公司系統管理員。 如果使用者不是系統管理員，就會終止流程。
3. 連接器會產生用戶端憑證要求，並與權杖一起傳遞至應用程式 Proxy 服務。 服務接著會驗證權杖，並簽署用戶端憑證要求。
4. 連接器可以使用此用戶端憑證，以便未來與應用程式 Proxy 服務通訊。
5. 連接器會使用其用戶端憑證從服務執行系統設定資料的初始提取，而它現在已準備好接受要求。

### <a name="updating-the-configuration-settings"></a>更新組態設定

每當應用程式 Proxy 服務更新組態設定時，就會發生下列流程事件︰

1. 連接器會使用其用戶端憑證連線至應用程式 Proxy 服務內的組態端點。
2. 用戶端憑證經過驗證後，應用程式 Proxy 服務會將組態資料傳回連接器 (例如，連接器應屬之連接器群組)。
3. 如果目前的憑證已存在超過 180 天，連接器就會產生新的憑證要求，每隔 180 天會有效地更新用戶端憑證。

### <a name="accessing-published-applications"></a>存取已發佈的應用程式

當使用者存取已發佈的應用程式時，會在應用程式 Proxy 服務和應用程式 Proxy 連接器之間進行下列事件：

1. [服務會驗證應用程式的使用者](#the-service-checks-the-configuration-settings-for-the-app)
2. [服務會將要求放在連接器佇列中](#The-service-places-a-request-in-the-connector-queue)
3. [連接器會處理來自佇列的要求](#the-connector-receives-the-request-from-the-queue)
4. [連接器會等候回應](#the-connector-waits-for-a-response)
5. [服務會將資料串流給使用者](#the-service-streams-data-to-the-user)

若要深入了解每個步驟中所發生的事項，請繼續閱讀。


#### <a name="1-the-service-authenticates-the-user-for-the-app"></a>1.服務會驗證應用程式的使用者

如果您將應用程式設定成使用 Passthrough 作為其預先驗證方法時，就會跳過本節中的步驟。

如果您將應用程式設定為以 Azure AD 預先驗證時，使用者會重新導向至 Azure AD STS 驗證，並採取下列步驟：

1. 應用程式 Proxy 會檢查特定應用程式的所有條件式存取原則需求。 這個步驟可確保已對應用程式指派使用者。 如果需要雙步驟驗證，驗證順序會提示使用者進行第二驗證方法。

2. 通過所有檢查後，Azure AD STS 會針對應用程式發出已簽署權杖，並將使用者重新導向回到應用程式 Proxy 服務。

3. 應用程式 Proxy 會驗證權杖已發給正確的應用程式。 它也會執行其他檢查，例如確保權杖是由 Azure AD 所簽署，且仍在有效期限內。

4. 應用程式 Proxy 會設定加密的驗證 cookie，以表示已發生應用程式驗證。 此 cookie 包含根據 Azure AD 的權杖和其他資料之到期時間戳記，例如以驗證為基礎的使用者名稱。 會使用僅應用程式 Proxy 服務所知的私密金鑰來加密此 cookie。

5. 應用程式 Proxy 會將使用者重新導向回到原始要求的 URL。

如果預先驗證步驟的任何部分失敗，使用者的要求會遭到拒絕，且使用者會顯示訊息，指出問題的來源。


#### <a name="2-the-service-places-a-request-in-the-connector-queue"></a>2.服務會將要求放在連接器佇列中

連接器保持對應用程式 Proxy 服務開啟輸出連線。 當要求傳入時，服務會在其中一個開啟連線上佇列要求，以供連接器挑選。

要求包含來自應用程式的項目，例如要求標頭、來自加密 cookie 的資料、提出要求的使用者，以及要求識別碼。 雖然來自加密 cookie 的資料會與要求一起傳送，但驗證 cookie 本身並不是。

#### <a name="3-the-connector-processes-the-request-from-the-queue"></a>3.連接器會處理來自佇列的要求。 

根據要求，應用程式 Proxy 會執行下列其中一個動作︰

* 如果要求是簡單的作業 (例如，主體內沒有資料現狀符合 RESTful「GET」要求)，連接器會建立連往目標內部資源的連線，然後等候回應。

* 如果要求在主體中具有與它相關聯的資料 (例如，RESTful「POST」作業)，連接器會使用用戶端憑證建立與應用程式 Proxy 執行個體的輸出連線。 它會建立此連線來要求資料，並開啟與內部部署資源的連線。 在收到來自連接器的要求後，應用程式 Proxy 服務會開始接受來自使用者的內容，並將資料轉送至連接器。 連接器會依次將資料轉送到內部資源。

#### <a name="4-the-connector-waits-for-a-response"></a>4.連接器會等候回應。

完成所有內容的要求並傳輸至後端後，連接器會等候回應。

在收到回應後，連接器會建立對應用程式 Proxy 服務的輸出連線，以傳回標頭詳細資料，並開始串流傳回的資料。

#### <a name="5-the-service-streams-data-to-the-user"></a>5.服務會將資料串流給使用者。 

應用程式的一些處理可能會在這裡發生。 如果您將應用程式中的應用程式 Proxy 應用程式設定為轉譯標頭或 URL，會視需要在此步驟進行該處理。


## <a name="next-steps"></a>後續步驟

[使用 Azure AD 應用程式 Proxy 時的網路拓撲考量](application-proxy-network-topology.md)

[了解 Azure AD 應用程式 Proxy 連接器](application-proxy-connectors.md)
