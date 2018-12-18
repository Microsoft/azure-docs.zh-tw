---
title: 加強 Azure 中的遠端管理安全性 | Microsoft Docs
description: 本文探討管理 Microsoft Azure 環境時提升遠端管理安全性的步驟，這些環境包括雲端服務、虛擬機器及自訂應用程式。
services: security
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: TomSh
ms.assetid: 2431feba-3364-4a63-8e66-858926061dd3
ms.service: security
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2017
ms.author: terrylan
ms.openlocfilehash: efab734cc7d6facf82f622b95c12a1a27e99c5f7
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45575927"
---
# <a name="security-management-in-azure"></a>Azure 的安全性管理
Azure 訂閱者可從多種裝置管理其雲端環境，這些裝置包括管理工作站、開發人員的電腦，甚至是具有工作專用權限的特殊權限使用者裝置。 有時候，管理功能是透過 Web 式主控台來執行，例如 [Azure 入口網站](https://azure.microsoft.com/features/azure-portal/)。 至於其他時候，則可能會從內部部署系統，透過虛擬私人網路 (VPN)、終端機服務、用戶端應用程式通訊協定或 Azure 服務管理 API (SMAPI) (以程式設計方式) 直接連線至 Azure。 此外，用戶端端點也可以加入網域或是遭到隔離且非受控，例如平板電腦或智慧型手機。

雖然多項存取和管理功能可提供一組豐富的選項，但選項太多也可能會讓雲端部署承受巨大風險。 因而難以管理、追蹤和稽核管理動作。 選項太多也可能會因為用來管理雲端服務之用戶端端點所進行的存取不受管制而招致安全性威脅。 使用一般工作站或私人工作站來開發和管理基礎結構將會打開無法預期的威脅媒介，例如網頁瀏覽 (例如水坑攻擊) 或電子郵件 (例如社交工程和網路釣魚)。

![][1]

在這類環境中，發生攻擊的可能性會增加，因為其難以建構安全性原則和機制來適當管理各式各樣的端點對 Azure 介面 (例如 SMAPI) 的存取。

### <a name="remote-management-threats"></a>遠端管理威脅
攻擊者通常會嘗試透過入侵帳戶認證 (例如，透過暴力破解密碼、網路釣魚和蒐集認證) 或誘騙使用者執行有害程式碼 (例如，從具有偷渡式下載的有害網站或從有害電子郵件的附件) 來取得特殊權限存取權。 遠端管理的雲端環境由於具有能隨時隨地存取的特性，因此若帳戶遭到入侵，風險將會大增。

即使主要的系統管理員帳戶受到嚴密控管，攻擊者仍可使用較低層級的使用者帳戶來利用安全性策略中的弱點。 因為缺乏適當的安全性訓練而讓帳戶資訊意外洩漏或曝光也可能會導致入侵事件發生。

若使用者工作站也用來執行系統管理工作，便可能會在許多不同的地方遭到入侵。 無論使用者是在瀏覽網頁、使用第三方工具和開放原始碼工具，還是開啟含有特洛伊木馬程式的有害文件，都會面臨入侵風險。

一般情況下，大多數會導致資料外洩的鎖定式攻擊，追根究底都是桌上型電腦上的瀏覽器入侵、外掛程式 (例如 Flash、PDF、Java) 和魚叉式網路釣魚 (電子郵件) 所造成。 這些電腦在用來開發或管理其他資產時，可能會具有可供存取運作中伺服器或網路裝置以執行作業的系統管理層級權限或服務層級權限。

### <a name="operational-security-fundamentals"></a>作業安全性基本概念
如需提升管理和作業時的安全性，您可以減少可能的進入點數目以盡可能縮減用戶端的受攻擊面。 這可以透過下列安全性原則來達成：「區分職責」和「隔離環境」。

讓敏感性功能彼此隔離可減少某個層級的錯誤導致另一個層級出現漏洞的可能性。 範例：

* 系統管理工作也不應該與可能會造成入侵的活動合併 (例如，系統管理員的電子郵件中有惡意程式碼，進而感染基礎結構伺服器)。
* 用於高敏感性作業的工作站也不應該是用於高風險用途 (例如瀏覽網際網路) 的相同系統。

藉由移除不必要的軟體來減少系統的受攻擊面。 範例：

* 如果裝置的主要目的是要管理雲端服務，則標準的系統管理、支援或開發工作站皆不應該要求安裝電子郵件用戶端或其他生產力應用程式。

具有基礎結構元件系統管理員存取權的用戶端系統，應該受到最嚴格的原則所約束，以降低安全性風險。 範例：

* 安全性原則可以加入會拒絕裝置開放存取網際網路和使用嚴格防火牆組態的群組原則設定。
* 如果需要直接存取，請使用網際網路通訊協定安全性 (IPsec) VPN。
* 針對管理和開發設定不同的 Active Directory 網域。
* 隔離和篩選管理工作站的網路流量。
* 使用反惡意程式碼軟體。
* 實作 Multi-Factor Authentication 以降低遭竊認證的風險。

合併存取資源和避免使用非受控端點也可簡化管理工作。

### <a name="providing-security-for-azure-remote-management"></a>為 Azure 的遠端管理提供安全性
Azure 提供了安全性機制來協助系統管理員管理 Azure 雲端服務和虛擬機器。 這些機制包括︰

* 驗證和[角色型存取控制](../role-based-access-control/role-assignments-portal.md)。
* 監視、記錄和稽核。
* 憑證和加密通訊。
* Web 管理入口網站。
* 網路封包篩選。

透過用戶端安全性組態和管理閘道的資料中心部署，就能限制並監視系統管理員對於雲端應用程式和資料的存取。

> [!NOTE]
> 本文的某些建議可能會導致資料、網路或計算資源使用量增加，並可能增加授權或訂用帳戶成本。
>
>

## <a name="hardened-workstation-for-management"></a>強化管理工作站
強化工作站的目標是要去除其他所有功能，只留下其運作所需的最重要功能，盡可能縮小潛在的受攻擊面。 系統強化包括安裝最少量的服務和應用程式、限制應用程式執行、限制網路只能存取所需資源，以及讓系統隨時保持最新狀態。 此外，使用經過強化的管理工作站能夠將系統管理工具和活動與其他使用者工作隔離開來。

在內部部署企業環境中，您可以透過專用管理網路、必須用身分卡片才能進入的伺服器機房以及在受保護的網路區域上執行的工作站，限制實體基礎結構的受攻擊面。 在雲端或混合式 IT 模型中，由於無法實際接觸到 IT 資源，因此想要努力讓管理服務保持安全會是更複雜的工作。 實作保護解決方案需要小心軟體設定、安全性為主的處理程序及完善的原則。

在鎖定的工作站中使用最低權限的最少軟體使用量來管理雲端 (以及開發應用程式)，可藉由將遠端管理和開發環境標準化來降低引發安全性事件的風險。 強化後的工作站組態可透過關閉惡意程式碼和入侵程式使用的許多常見手段，來協助避免用來管理重要雲端資源的帳戶遭到入侵。 具體而言，您可以使用 [Windows AppLocker](http://technet.microsoft.com/library/dd759117.aspx) 和 Hyper-V 技術來控制和隔離用戶端系統行為並減輕威脅，包括電子郵件或網際網路瀏覽。

在強化後的工作站上，系統管理員會執行標準使用者帳戶 (它會封鎖執行系統管理層級)，且相關聯的應用程式會由允許清單進行控制。 強化後之工作站的基本要素如下︰

* 作用中的掃描和修補。 部署反惡意程式碼軟體、定期執行弱點掃描，以及及時使用最新的安全性更新來更新所有工作站。
* 有限的功能。 解除安裝任何不需要的應用程式，並停用不必要的 (啟動) 服務。
* 強化網路。 使用 Windows 防火牆規則，僅允許與 Azure 管理相關的有效 IP 位址、連接埠和 URL。 確定也已封鎖工作站的輸入遠端連線。
* 執行限制。 僅允許執行一組管理所需的預先定義可執行檔 (稱為「預設拒絕」)。 根據預設，除非程式明確定義於允許清單中，否則應該拒絕使用者執行程式的權限。
* 最小特殊權限。 管理工作站的使用者不應該擁有本機電腦本身的任何系統管理特殊權限。 如此一來，他們才無法變更系統組態或系統檔案，不論是有意或無意。

透過在 Active Directory Domain Services (AD DS) 中使用[群組原則物件](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-admin-guide-administer-group-policy) (GPO)，並透過 (本機) 管理網域將其套用到所有管理帳戶，您即可強制執行上述所有要素。

### <a name="managing-services-applications-and-data"></a>管理服務、應用程式和資料
Azure 雲端服務組態是透過 Azure 入口網站或 SMAPI，經由 Windows PowerShell 命令列介面或利用這些符合 REST 限制之介面的自建應用程式來執行。 使用這些機制的服務包括 Azure Active Directory (Azure AD)、Azure 儲存體、Azure 網站和 Azure 虛擬網路等。

虛擬機器部署的應用程式會視需要提供自己的用戶端工具和介面 (例如 Microsoft Management Console (MMC))、企業管理主控台 (例如 Microsoft System Center 或 Windows Intune) 或其他管理應用程式 (例如 Microsoft SQL Server Management Studio)。 這些工具通常位在企業環境或用戶端網路中。 它們可能仰賴需要直接、具狀態之連線的特定網路通訊協定，例如遠端桌面通訊協定 (RDP)。 有些則可能會有不應該透過網際網路公開發佈或存取的具有 Web 功能的介面。

您可以使用 [Multi-Factor Authentication](../active-directory/authentication/multi-factor-authentication.md)、[X.509 管理憑證](https://blogs.msdn.microsoft.com/azuresecurity/2015/07/13/certificate-management-in-azure-dos-and-donts/)和防火牆規則來限制存取 Azure 中的基礎結構和平台服務管理。 Azure 入口網站和 SMAPI 需要傳輸層安全性 (TLS)。 不過，您部署至 Azure 的服務和應用程式需要您根據應用程式採取合適的保護措施。 這些機制可以透過標準化的強化後工作站組態更容易地經常啟用。

### <a name="management-gateway"></a>管理閘道
若要集中管理所有系統管理存取權並簡化監視與記錄，您可以在內部部署網路中部署連線到 Azure 環境的專用[遠端桌面閘道](https://technet.microsoft.com/library/dd560672) (RD 閘道) 伺服器。

遠端桌面閘道是以原則為基礎 RDP Proxy 服務，會強制執行安全性需求。 同時實作 RD 閘道與 Windows Server 網路存取保護 (NAP)，可協助確保只有符合 Active Directory 網域服務 (AD DS) 群組原則物件 (GPO) 所建立之特定安全性健全狀況準則的用戶端可以連線。 此外：

* 在 RD 閘道上佈建 [Azure 管理憑證](http://msdn.microsoft.com/library/azure/gg551722.aspx)，使它成為可以存取 Azure 入口網站的唯一主機。
* 將 RD 閘道加入至相同的[管理網域](http://technet.microsoft.com/library/bb727085.aspx)以做為系統管理員的工作站。 當您在具有對 Azure AD 之單向信任的網域內使用網站間 IPsec VPN 或 ExpressRoute 時，或是如果您要同盟內部部署 AD DS 執行個體與 Azure AD 之間的認證，就必須這麼做。
* 設定[用戶端連線授權原則](http://technet.microsoft.com/library/cc753324.aspx)，讓 RD 閘道驗證用戶端電腦名稱是否有效 (已加入網域)，並允許存取 Azure 入口網站。
* 針對 [Azure VPN](https://azure.microsoft.com/documentation/services/vpn-gateway/) 使用 IPsec 以進一步防止管理流量遭到竊聽以及權杖遭竊，或考慮使用透過 [Azure ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) 的隔離網際網路連結。
* 針對透過 RD 閘道登入的系統管理員啟用 Multi-Factor Authentication (透過 [Azure Multi-Factor Authentication](../active-directory/authentication/multi-factor-authentication.md)) 或智慧卡驗證。
* 在 Azure 中設定來源 [IP 位址限制](http://azure.microsoft.com/blog/2013/08/27/confirming-dynamic-ip-address-restrictions-in-windows-azure-web-sites/)或[網路安全性群組](../virtual-network/security-overview.md)以將允許的管理端點數目降到最低。

## <a name="security-guidelines"></a>安全性方針
協助保護搭配雲端使用之系統管理員工作站的做法，通常會與用於任何內部部署工作站的做法類似 - 例如，最小化的組建和嚴格的權限。 雲端管理的幾項特點則更類似於遠端或頻外企業管理。 這些特點包括使用和稽核認證、增強安全性的遠端存取以及威脅偵測和回應。

### <a name="authentication"></a>驗證
您可以使用 Azure 登入限制來限制用於存取系統管理工具的來源 IP 位址和稽核存取要求。 若要協助 Azure 識別管理用戶端 (工作站及/或應用程式)，您可以同時設定 SMAPI (透過客戶開發的工具，例如 Windows PowerShell Cmdlet) 和 Azure 入口網站，要求除了 SSL 憑證外，還必須安裝用戶端管理憑證。 我們也建議系統管理員存取需要 Multi-Factor Authentication。

您部署至 Azure 的某些應用程式或服務可能會針對使用者和系統管理員存取擁有自己的驗證機制，而其他應用程式或服務則會充分利用 Azure AD。 根據您是透過 Active Directory Federation Services (AD FS)、使用目錄同步作業或僅在雲端中維護使用者帳戶來同盟認證，使用 [Microsoft Identity Manager](https://technet.microsoft.com/library/mt218776.aspx) (Azure AD Premium 的一部分) 可協助您管理資源之間的身分識別生命週期。

### <a name="connectivity"></a>連線能力
有數種機制可供協助保護用戶端與 Azure 虛擬網路的連線。 這些機制的其中兩個 ([網站間 VPN](https://channel9.msdn.com/series/Azure-Site-to-Site-VPN) (S2S) 和[點對站 VPN](../vpn-gateway/vpn-gateway-point-to-site-create.md) (P2S)) 可使用業界標準 IPsec (S2S) 或[安全通訊端通道通訊協定](https://technet.microsoft.com/magazine/2007.06.cableguy.aspx) (SSTP) (P2S) 來進行加密和通道傳輸。 當 Azure 連接至公開的 Azure 服務管理 (例如 Azure 入口網站) 時，Azure 需要超文字安全傳輸通訊協定 (HTTPS)。

未透過 RD 閘道連線至 Azure 的獨立強化後工作站應該使用 SSTP 架構的點對站 VPN 來建立與 Azure 虛擬網路的初始連線，然後再從 VPN 通道建立與個別虛擬機器的 RDP 連線。

### <a name="management-auditing-vs-policy-enforcement"></a>管理稽核與原則強制執行
一般而言，有兩種方法可用來協助保護管理程序︰稽核和原則強制執行。 同時採用這兩種方法可進行全面控制，但並非所有情況下都能這麼做。 此外，每一種方法在管理安全性時都需要不同程度的風險、成本和心力，特別是當它涉及對個人和系統架構所給予的信任程度時。

監視、記錄和稽核可為追蹤和了解系統管理活動提供基礎，但受限於所產生的資料量，它不一定都能鉅細靡遺地稽核所有動作。 不過，稽核管理原則的效果是最佳作法。

包含嚴格存取控制的原則強制執行具有可控制系統管理員動作的程式設計機制，並可協助確保使用所有可能的保護措施。 記錄可提供強制執行的證明，以及什麼人在何時從什麼地方做了什麼動作的記錄。 記錄也可讓您稽核和交叉核對系統管理員如何遵循原則的相關資訊，而且它也能提供活動的證據。

## <a name="client-configuration"></a>用戶端組態
針對強化後的工作站，我們有三種主要組態建議。 這三者之間最大的差異在於成本、可用性和存取性，但它們提供的所有選項都有類似的安全性設定檔。 下表扼要分析其各自的優點與風險。 (請注意，「公司電腦」指的是將為所有網域使用者 (不論角色為何) 部署的標準桌面電腦組態)。

| 組態 | 優點 | 缺點 |
| --- | --- | --- |
| 獨立的強化後工作站 |受到嚴格控制的工作站 |專用桌上型電腦的成本較高 |
| - | 降低應用程式入侵風險 |增加管理工作 |
| - | 清楚區分職責 | - |
| 以公司電腦做為虛擬機器 |降低硬體成本 | - |
| - | 隔離角色和應用程式 | - |
| Windows To Go 與 BitLocker 磁碟機加密 |與大部分電腦相容 |資產追蹤 |
| - | 符合成本效益且具有可攜性 | - |
| - | 隔離的管理環境 |- |

請務必讓強化後的工作站做為主機而非客體，且主機作業系統和硬體之間沒有任何東西。 遵循「乾淨來源原則」(也稱為「安全來源」) 表示主機應該是最安全的。 否則，強化後的工作站 (客體) 在其裝載所在的系統上將容易受到攻擊。

您可以透過專用系統映像為每一部強化後的工作站進一步隔離系統管理功能，使其只具有管理選定的 Azure 和雲端應用程式所需的工具和權限，以及用於必要工作的特定本機 AD DS GPO。

對於沒有任何內部部署基礎結構的 IT 環境 (例如，因為所有伺服器都在雲端而不能存取 GPO 的本機 AD DS 執行個體)，[Microsoft Intune](https://technet.microsoft.com/library/jj676587.aspx) 之類的服務可以簡化工作站組態的部署和維護工作。

### <a name="stand-alone-hardened-workstation-for-management"></a>用於管理的獨立強化後工作站
使用獨立的強化後工作站時，系統管理員會使用一部電腦或膝上型電腦來進行系統管理工作，並使用另一個不同的電腦或膝上型電腦來進行非系統管理工作。 專門負責管理 Azure 服務的工作站不需要安裝其他應用程式。 此外，使用的工作站若支援[信賴平台模組](https://technet.microsoft.com/library/cc766159) (TPM) 或類似的硬體層級密碼編譯技術，將有助於進行裝置驗證和預防特定攻擊。 TPM 也可以使用 [BitLocker 磁碟機加密](https://technet.microsoft.com/library/cc732774.aspx)來支援系統磁碟機的完整磁碟區保護。

在獨立的強化後工作站案例中 (如下所示)，Windows 防火牆 (或非 Microsoft 用戶端防火牆) 的本機執行個體會設定為封鎖輸入連線，例如 RDP。 系統管理員可以登入強化後的工作站，並在與 Azure 虛擬網路建立 VPN 連線之後啟動連線至 Azure 的 RDP 工作階段，但無法登入公司電腦並使用 RDP 連線至強化後的工作站本身。

![][2]

### <a name="corporate-pc-as-virtual-machine"></a>以公司電腦做為虛擬機器
在部署個別的獨立強化後工作站需要高昂成本或無法撥出此預算的情況下，強化後的工作站可以裝載用來執行非系統管理工作的虛擬機器。

![][3]

若要避免使用單一工作站來進行系統管理和其他日常工作所可能引發的諸多安全性風險，您可以在強化後的工作站部署 Windows Hyper-V 虛擬機器。 此虛擬機器可以做為公司電腦來使用。 公司電腦環境可以與主機保持區隔，以減少其受攻擊面，並讓使用者的日常活動 (例如電子郵件) 不會與敏感的系統管理工作共存。

公司電腦虛擬機器會在受保護的空間內執行，並提供使用者應用程式。 主機仍是「乾淨來源」，並且會在根作業系統中強制執行嚴格的網路原則 (例如，封鎖來自虛擬機器的 RDP 存取)。

### <a name="windows-to-go"></a>Windows To Go
需要獨立的強化後工作站的另一個替代方式是使用 [Windows To Go](https://technet.microsoft.com/library/hh831833.aspx) 磁碟機，這個功能可支援用戶端 USB 開機功能。 Windows To Go 可讓使用者將相容的電腦開機到從加密 USB 快閃磁碟機執行的隔離系統映像。 因為映像可以完全由公司的 IT 團隊負責管理、有嚴格的安全性原則、最小的作業系統組建和 TPM 支援，因此Windows To Go 可以提升對遠端系統管理端點的控制能力。

在下圖中，可攜式映像是已加入網域的系統，其已預先設定為僅連線至 Azure、需要 Multi-Factor Authentication，並且會封鎖所有非管理流量。 如果使用者將同一部電腦開機到標準公司映像，並嘗試存取 Azure 管理工具的 RD 閘道，工作階段即會遭到封鎖。 Windows To Go 會成為根層級作業系統，而且不需要可能更容易遭受外部攻擊的其他層 (主機作業系統、Hypervisor、虛擬機器)。

![][4]

請務必注意，比起一般的桌上型電腦，USB 快閃磁碟機更容易遺失。 使用 BitLocker 來加密整個磁碟區時若能搭配強式密碼，攻擊者就更不可能使用磁碟機映像來進行有害活動。 此外，如果遺失 USB 快閃磁碟機，則撤銷和[發出新的管理憑證](https://technet.microsoft.com/library/hh831574.aspx)以及快速重設密碼可以降低風險。 系統管理稽核記錄存放在 Azure 而非用戶端，將可進一步減少遺失資料的可能性。

## <a name="best-practices"></a>最佳作法
當您管理 Azure 中的應用程式和資料時，請考慮下列額外的方針。

### <a name="dos-and-donts"></a>建議事項和避免事項
請勿因為工作站已封鎖起來，就認為不需要滿足其他常見的安全性需求。 因為系統管理員帳戶通常擁有提高權限的存取層級，因此潛在風險會提高。 下表顯示風險和其替代安全作法的範例。

| 避免事項 | 建議事項 |
| --- | --- |
| 請勿以電子郵件寄送用於系統管理員存取權或其他機密資料的認證 (例如 SSL 或管理憑證) |用聲音提供帳戶名稱和密碼 (但不要將它們儲存在語音郵件中) 以維持機密性、遠端安裝用戶端/伺服器憑證 (透過加密工作階段)、從受保護的網路共用下載，或透過卸除式媒體手動發佈。 |
| - | 主動管理您的管理憑證生命週期。 |
| 請勿在應用程式儲存體中儲存未加密或未雜湊的帳戶密碼 (例如在試算表、SharePoint 網站或檔案共用中)。 |建立安全性管理原則和系統強化原則，並將它們套用至您的開發環境。 |
| - | 使用 [Enhanced Mitigation Experience Toolkit 5.5](https://technet.microsoft.com/security/jj653751) 憑證釘選規則，以確保能正確存取 Azure SSL/TLS 網站。 |
| 請勿讓系統管理員共用帳戶和密碼，或在多個使用者帳戶或服務重複使用密碼，特別是用於社交媒體或其他非系統管理活動的帳戶或服務。 |建立專用的 Microsoft 帳戶來管理您的 Azure 訂用帳戶，此帳戶不會用於個人電子郵件。 |
| 請勿以電子郵件寄送組態檔。 |應該從信任的來源 (例如，加密的 USB 快閃磁碟機) 而非從可輕易入侵的機制 (例如電子郵件) 安裝組態檔和設定檔。 |
| 請勿使用弱式或簡單的登入密碼。 |強制使用強式密碼原則、到期循環 (首次使用時變更)、主控台逾時和自動帳戶鎖定。 使用用戶端密碼管理系統搭配 Multi-Factor Authentication 來存取密碼保存庫。 |
| 請勿對網際網路公開管理連接埠。 |鎖定 Azure 連接埠和 IP 位址來限制管理存取。 如需詳細資訊，請參閱 [Azure 網路安全性](http://download.microsoft.com/download/4/3/9/43902EC9-410E-4875-8800-0788BE146A3D/Windows%20Azure%20Network%20Security%20Whitepaper%20-%20FINAL.docx)技術白皮書。 |
| - | 針對所有管理連線使用防火牆、VPN 和 NAP。 |

## <a name="azure-operations"></a>Azure 作業
在 Microsoft 的 Azure 作業內，存取 Azure 之生產系統的作業工程師和支援人員，會使用[強化後的工作站電腦與其中所佈建的 VM](#stand-alone-hardened-workstation-for-management)來進行內部的公司網路存取和執行應用程式 (例如電子郵件、內部網路等)。 所有管理工作站電腦都有 TPM、主機開機磁碟機已使用 BitLocker 加密，而且它們已加入 Microsoft 主要公司網域中的特殊組織單位 (OU)。

系統強化是透過群組原則以集中式的軟體更新來強制執行。 為了進行稽核和分析，會從管理工作站收集事件記錄 (例如安全性和 AppLocker) 並儲存到中央位置。

此外，會使用 Microsoft 網路上需要雙因素驗證的專用 Jumpbox 來連線到 Azure 的生產網路。

## <a name="azure-security-checklist"></a>Azure 安全性檢查清單
將系統管理員可在強化後的工作站上執行的工作數目降至最低，有助於盡量降低開發和管理環境中的受攻擊面。 請使用下列技術來協助保護強化後的工作站︰

* IE 強化。 Internet Explorer 瀏覽器 (或任何類似用途的網頁瀏覽器) 因為會與外部伺服器廣泛互動，所以是有害程式碼的主要進入點。 請檢閱您的用戶端原則並強制要求在受保護模式中執行、停用附加元件、停用檔案下載，以及使用 [Microsoft SmartScreen](https://technet.microsoft.com/library/jj618329.aspx) 篩選。 確定會顯示安全性警告。 利用網際網路區域，並建立已為其設定合理強化的信任網站清單。 封鎖其他所有網站和瀏覽器中的程式碼，例如 ActiveX 和 Java。
* 標準使用者。 以標準使用者的身分執行有許多好處，最重要的好處是透過惡意程式碼竊取系統管理員認證會變得更困難。 此外，標準使用者帳戶沒有根作業系統的提高的權限，而且許多組態選項和 API 已依預設鎖定起來。
* AppLocker。 您可以使用 [AppLocker](http://technet.microsoft.com/library/ee619725.aspx) 來限制使用者可以執行的程式和指令碼。 您可以在稽核或強制模式中執行 AppLocker。 根據預設，AppLocker 的允許規則可讓具有系統管理員權杖的使用者執行用戶端上的所有程式碼。 此規則的存在是為了避免系統管理員將自己鎖定，而且只適用於提高權限的權杖。 另請參閱做為 Windows Server [核心安全性](http://technet.microsoft.com/library/dd348705.aspx)一部分的「程式碼完整性」。
* 程式碼簽署。 為系統管理員使用的所有工具和指令碼進行程式碼簽署，可提供方便管理的機制來部署應用程式鎖定原則。 雜湊不會隨著程式碼的快速變更而做調整，而且檔案路徑不會提供高度安全性。 您應該將 AppLocker 規則結合 PowerShell [執行原則](http://technet.microsoft.com/library/ee176961.aspx)，此原則只允許[執行](http://technet.microsoft.com/library/hh849812.aspx)特定已簽署的程式碼和指令碼。
* 群組原則。 建立全域系統管理原則以套用至任何用於管理的網域工作站 (並封鎖來自其他所有用途的存取)，以及套用至在這些工作站上進行驗證的使用者帳戶。
* 已增強安全性的佈建。 保護您的基準強化後工作站映像以防遭到竄改。 使用加密和隔離等安全性措施來儲存映像、虛擬機器和指令碼，並限制存取 (或許是使用可稽核的簽入/簽出程序)。
* 修補。 維護一致的組建 (或針對開發、作業和其他系統管理工作使用不同的映像)、定期掃描變更和惡意程式碼、讓組建保持最新狀態，並且只在需要時才啟用機器。
* 加密。 確定管理工作站有 TPM 以便能夠更安全地啟用[加密檔案系統](https://technet.microsoft.com/library/cc700811.aspx) (EFS) 和 BitLocker。 如果您使用 Windows To Go，請只搭配 BitLocker 使用加密的 USB 金鑰。
* 控管。 使用 AD DS GPO 來控制所有系統管理員的 Windows 介面，例如檔案共用。 將管理工作站納入稽核、監視和記錄程序內。 追蹤所有系統管理員和開發人員的存取和使用活動。

## <a name="summary"></a>總結
使用強化後的工作站組態來管理 Azure 雲端服務、虛擬機器和應用程式，可協助您避免遠端管理重要 IT 基礎結構所產生的眾多風險和威脅。 Azure 和 Windows 皆可提供相關機制供您保護和控制通訊、驗證和用戶端行為。

## <a name="next-steps"></a>後續步驟
除了本白皮書所提到的特定項目外，下列資源還可提供更多有關 Azure 和相關 Microsoft 服務的一般資訊：

* [保護特殊權限存取](https://technet.microsoft.com/library/mt631194.aspx) – 取得設計和建置安全的系統管理工作站以管理 Azure 的技術詳細資料
* [Microsoft 信任中心](https://microsoft.com/en-us/trustcenter/cloudservices/azure) - 了解可保護 Azure 網狀架構和在 Azure 上執行之工作負載的 Azure 平台功能
* [Microsoft 安全性回應中心](https://technet.microsoft.com/security/dn440717.aspx) -- 可在其中回報 Microsoft 安全性弱點 (包括 Azure 的問題) 或透過電子郵件傳送給 [secure@microsoft.com](mailto:secure@microsoft.com)
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) – 隨時掌握 Azure 安全性的最新消息

<!--Image references-->
[1]: ./media/azure-security-management/typical-management-network-topology.png
[2]: ./media/azure-security-management/stand-alone-hardened-workstation-topology.png
[3]: ./media/azure-security-management/hardened-workstation-enabled-with-hyper-v.png
[4]: ./media/azure-security-management/hardened-workstation-using-windows-to-go-on-a-usb-flash-drive.png
