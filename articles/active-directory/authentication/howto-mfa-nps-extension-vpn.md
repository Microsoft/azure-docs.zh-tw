---
title: 使用網路原則伺服器擴充功能來整合 VPN 與 Azure MFA | Microsoft Docs
description: 使用 Microsoft Azure 的網路原則伺服器擴充功能來整合您的 VPN 基礎結構與 Azure MFA。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 0754b13704810816b1650010a00185031ca41f5d
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46297566"
---
# <a name="integrate-your-vpn-infrastructure-with-azure-mfa-by-using-the-network-policy-server-extension-for-azure"></a>使用 Azure 的網路原則伺服器擴充功能來整合您的 VPN 基礎結構與 Azure MFA

## <a name="overview"></a>概觀

Azure 的網路原則伺服器 (NPS) 擴充功能可讓組織使用以雲端為基礎的 [Azure Multi-Factor Authentication (MFA)](howto-mfaserver-nps-rdg.md) (可提供雙步驟驗證) 來保護遠端驗證撥入使用者服務 (RADIUS) 用戶端驗證。

本文提供指示，引導您使用 Azure 的 NPS 擴充功能來整合 NPS 基礎結構與 MFA。 此程序可針對嘗試使用 VPN 連線到您網路的使用者提供安全的雙步驟驗證。 

網路原則與存取服務為組織提供執行下列作業的能力：

* 指派管理和控制網路要求的中央位置，以指定：

    * 誰可以連線 
    
    * 一天當中允許連線的時間 
    
    * 連線的持續時間
    
    * 用戶端連線必須使用的安全性層級

    在每個 VPN 或遠端桌面閘道伺服器位於中央位置後，再為它們指定原則。 系統會使用 RADIUS 通訊協定來提供集中式的驗證、授權和計量 (AAA)。 

* 建立並強制執行網路存取保護 (NAP) 用戶端健康原則，以判斷要讓裝置不受限制還是受限制地存取網路資源。

* 提供強制驗證和授權的方法，以供存取具有 802.1x 功能的無線存取點及乙太網路交換器。   
如需詳細資訊，請參閱[網路原則伺服器](https://docs.microsoft.com/windows-server/networking/technologies/nps/nps-top)。 

為了加強安全性並提供高水準的合規性，組織可以整合 NPS 與 Azure Multi-Factor Authentication 來確保使用雙步驟驗證的使用者能夠連線到 VPN 伺服器上的虛擬連接埠。 使用者若要獲得存取權，就必須提供其使用者名稱和密碼的組合以及使用者所掌握的其他資訊。 這項資訊必須是受信任且無法輕易複製的。 可以是行動電話號碼、有線電話號碼或行動裝置上的應用程式。

在 Azure 的 NPS 擴充功能推出前，想要對整合式的 NPS 與 MFA 環境實作雙步驟驗證的客戶，必須在內部部署環境中另外設定及維護一部 MFA Server。 此類型的驗證由使用 RADIUS 的遠端桌面閘道和 Azure Multi-Factor Authentication Server 提供。

隨著 Azure 的 NPS 擴充功能推出，組織現在可以部署內部部署型 MFA 解決方案或雲端型 MFA 解決方案，以保護 RADIUS 用戶端驗證。
 
## <a name="authentication-flow"></a>驗證流程
當使用者連線到 VPN 伺服器上的虛擬連接埠時，必須先使用各種通訊協定通過驗證。 通訊協定允許使用使用者名稱和密碼的組合及憑證式驗證方法。 

除了驗證和確認身分識別外，使用者還必須擁有適當的撥入權限。 在簡單的實作中，可提供存取權的撥入權限是直接設定於 Active Directory 使用者物件上。 

![使用者屬性](./media/howto-mfa-nps-extension-vpn/image1.png)

在簡單的實作中，每一部 VPN 伺服器都會根據每個本機 VPN 伺服器上所定義的原則來授與或拒絕存取權。

而在擴充性更高的較大型實作中，負責授與或拒絕 VPN 存取權的原則則集中在 RADIUS 伺服器上。 在這些情況下，VPN 伺服器會擔任存取伺服器 (RADIUS 用戶端)，以將連線要求和帳戶訊息轉寄給 RADIUS 伺服器。 若要連線到 VPN 伺服器上的虛擬連接埠，使用者必須先通過驗證，並符合 RADIUS 伺服器上所集中定義的條件。 

當 Azure 的 NPS 擴充功能與 NPS 整合時，即可提供成功的驗證流程，如下所示：

1. VPN 伺服器收到來自 VPN 使用者的驗證要求，其中包含用來連線到某項資源 (例如，遠端桌面工作階段) 的使用者名稱和密碼。 

2. VPN 伺服器作為 RADIUS 用戶端，以將要求轉換為 RADIUS *Access-Request* 訊息，並將此訊息 (密碼會經過加密) 傳送至 NPS 擴充功能安裝所在的 RADIUS 伺服器。 

3. 使用者名稱和密碼組合在 Active Directory 中進行驗證。 如果使用者名稱或密碼不正確，RADIUS 伺服器會傳送 *Access-Reject* 訊息。 

4. 如果 NPS 連線要求與網路原則中所指定的所有條件 (例如，一天當中的時間或群組成員資格限制) 皆能符合，NPS 擴充功能就會觸發要求，以便使用 Azure Multi-Factor Authentication 進行第二項驗證。 

5. Azure Multi-Factor Authentication 會與 Azure Active Directory 通訊、擷取使用者的詳細資料，並使用使用者所設定的方法執行第二項驗證 (撥打行動電話、簡訊或行動裝置應用程式)。 

6. MFA 挑戰成功後，Azure Multi-Factor Authentication 會將結果傳遞給 NPS 擴充功能。

7. 當連線嘗試通過驗證並獲得授權後，擴充功能安裝所在的 NPS 就會傳送 RADIUS *Access-Accept* 訊息給 VPN 伺服器 (RADIUS 用戶端)。

8. 使用者會獲得 VPN 伺服器上之虛擬連接埠的存取權，並建立加密的 VPN 通道。

## <a name="prerequisites"></a>必要條件
本節會詳述在整合 MFA 與遠端桌面閘道之前必須完成的必要條件。 開始之前，您必須先具備下列必要條件：

* VPN 基礎結構
* 網路原則與存取服務角色
* Azure Multi-Factor Authentication 授權
* Windows Server 軟體
* 程式庫
* 與內部部署 Active Directory 同步的 Azure Active Directory (Azure AD) 
* Azure Active Directory GUID 識別碼

### <a name="vpn-infrastructure"></a>VPN 基礎結構
本文假設您已有使用 Microsoft Windows Server 2016 的運作中 VPN 基礎結構，而且 VPN 伺服器目前尚未設定為將連線要求轉寄給 RADIUS 伺服器。 在本文中，您要將 VPN 基礎結構設定為使用中央 RADIUS 伺服器。

如果您沒有已在運作的 VPN 基礎結構，則可以按照眾多 VPN 設定教學課程 (可於 Microsoft 和第三方廠商的網站上找到) 所提供的指導方針快速建立 VPN 基礎結構。 
            
### <a name="the-network-policy-and-access-services-role"></a>網路原則與存取服務角色

網路原則與存取服務可提供 RADIUS 伺服器和用戶端功能。 本文假設您已在環境中的成員伺服器或網域控制站上安裝網路原則與存取服務角色。 在本指南中，您要設定 RADIUS 以進行 VPN 設定。 在伺服器「而非」VPN 伺服器上安裝網路原則與存取服務角色。

如需有關安裝網路原則與存取服務角色服務 Windows Server 2012 或更新版本的資訊，請參閱[安裝 NAP 健康原則伺服器](https://technet.microsoft.com/library/dd296890.aspx) \(英文\)。 Windows Server 2016 已淘汰 NAP。 如需 NPS 最佳做法的說明 (包括在網域控制站上安裝 NPS 的建議)，請參閱 [NPS 的最佳做法](https://technet.microsoft.com/library/cc771746) \(英文\)。

### <a name="azure-mfa-license"></a>Azure MFA 授權

Azure Multi-Factor Authentication 需要授權，而且可透過 Azure AD Premium、Enterprise Mobility + Security 或 Multi-Factor Authentication 獨立授權提供。 Azure MFA 以使用情況為基礎的授權 (例如每位使用者或每次驗證授權) 與 NPS 擴充功能並不相容。 如需詳細資訊，請參閱[如何取得 Azure Multi-Factor Authentication](concept-mfa-licensing.md)。 若要進行測試，您可以使用試用版訂用帳戶。

### <a name="windows-server-software"></a>Windows Server 軟體

NPS 擴充功能需要 Windows Server 2008 R2 SP1 或更新版本，並安裝網路原則與存取服務角色。 本指南中的所有步驟皆是使用 Windows Server 2016 來執行的。

### <a name="libraries"></a>程式庫

下列程式庫會自動連同 NPS 擴充功能一起安裝：

-   [適用於 Visual Studio 2013 的 Visual C++ 可轉散發套件 (X64)](https://www.microsoft.com/download/details.aspx?id=40784)
-   [適用於 Windows PowerShell 1.1.1660 版本的 Microsoft Azure Active Directory 模組](https://connect.microsoft.com/site1164/Downloads/DownloadDetails.aspx?DownloadID=59185)

如果您還沒有 Microsoft Azure Active Directory PowerShell 模組，系統會透過您在安裝過程中執行的設定指令碼來加以安裝。 因此，如果您尚未安裝此模組，就不必事先安裝。

### <a name="azure-active-directory-synced-with-on-premises-active-directory"></a>與內部部署 Active Directory 同步的 Azure Active Directory 

若要使用 NPS 擴充功能，內部部署使用者必須與 Azure Active Directory 保持同步並啟用 MFA。 本指南假設內部部署使用者已使用 Azure AD Connect 來與 Azure Active Directory 保持同步。 下面會提供為使用者啟用 MFA 的指示。

如需 Azure AD Connect 的相關資訊，請參閱[整合您的內部部署目錄與 Azure Active Directory](../hybrid/whatis-hybrid-identity.md)。 

### <a name="azure-active-directory-guid-id"></a>Azure Active Directory GUID 識別碼 

若要安裝 NPS 擴充功能，您必須知道 Azure Active Directory 的 GUID。 下一節會提供尋找 Azure Active Directory GUID 的指示。

## <a name="configure-radius-for-vpn-connections"></a>設定 RADIUS 的 VPN 連線

如果您已在成員伺服器上安裝 NPS 角色，則必須進行設定以驗證和授權提出 VPN 連線要求的 VPN 用戶端。 

本節假設您已安裝網路原則與存取服務角色，但尚未將它設定用於您的基礎結構中。

> [!NOTE]
> 如果您已擁有運作中的 VPN 伺服器，且使用集中式的 RADIUS 伺服器來進行驗證，則可以略過本節。
>

### <a name="register-server-in-active-directory"></a>在 Active Directory 中註冊伺服器
若要讓 NPS 伺服器在本案例中正常運作，您必須在 Active Directory 中加以註冊。

1. 開啟 [伺服器管理員]。

2. 在 [伺服器管理員] 中選取 [工具]，然後選取 [網路原則伺服器]。 

3. 在 [網路原則伺服器] 主控台中，以滑鼠右鍵按一下 [NPS (本機)]，然後選取 [在 Active Directory 中註冊伺服器]。 選取 [確定] 兩次。

    ![網路原則伺服器](./media/howto-mfa-nps-extension-vpn/image2.png)

4. 讓主控台保持開啟以供下一個程序使用。

### <a name="use-wizard-to-configure-the-radius-server"></a>使用精靈來設定 RADIUS 伺服器
您可以使用標準 (精靈式) 或進階的設定選項來設定 RADIUS 伺服器。 本節假設您使用精靈式的標準設定選項。

1. 在 [網路原則伺服器] 主控台中，選取 [NPS (本機)]。

2. 選取 [標準設定] 底下的 [撥號或 VPN 連線的 RADIUS 伺服器]，然後選取 [設定 VPN 或撥號]。

    ![設定 VPN](./media/howto-mfa-nps-extension-vpn/image3.png)

3. 在 [選取撥號或虛擬私人網路連線類型] 視窗上選取 [虛擬私人網路連線]，然後選取 [下一步]。

    ![虛擬私人網路](./media/howto-mfa-nps-extension-vpn/image4.png)

4. 在 [指定撥號或 VPN 伺服器] 視窗中選取 [新增]。

5. 在 [新增 RADIUS 用戶端] 視窗中，提供好記的名稱，輸入可解析的 VPN 伺服器名稱或 IP 位址，然後輸入共用祕密。 請為此共用祕密使用較長且複雜的形式。 請記錄下來，因為下一節中將會用到。

    ![新增 RADIUS 用戶端](./media/howto-mfa-nps-extension-vpn/image5.png)

6. 選取 [確定]，然後選取 [下一步]。

7. 在 [設定驗證方法] 視窗中，接受預設的選取項目「Microsoft 加密驗證第 2 版 (MS-CHAPv2)」 或選擇另一個選項，然後選取 [下一步]。

    > [!NOTE]
    > 如果您要設定可延伸的驗證通訊協定 (EAP)，則必須使用 Microsoft Challenge-Handshake 驗證通訊協定 (CHAPv2) 或受保護的可延伸驗證通訊協定 (PEAP)。 其他 EAP 皆不受支援。
 
8. 在 [指定使用者群組] 視窗中選取 [新增]，然後選取適當的群組。 如果沒有群組，請保持空白選取狀態，以對所有使用者授與存取權。

    ![[指定使用者群組] 視窗](./media/howto-mfa-nps-extension-vpn/image7.png)

9. 選取 [下一步] 。

10. 在 [指定 IP 篩選器] 視窗中，選取 [下一步]。

11. 在 [指定加密設定] 視窗中接受預設設定，然後選取 [下一步]。

    ![指定加密設定視窗](./media/howto-mfa-nps-extension-vpn/image8.png)

12. 在 [指定領域名稱] 視窗中讓名稱保持空白，接受預設設定，然後選取 [下一步]。

    ![[指定領域名稱] 視窗](./media/howto-mfa-nps-extension-vpn/image9.png)

13. 在 [完成新的撥號或虛擬私人網路連線和 RADIUS 用戶端] 視窗中，選取 [完成]。

    ![[完成新的撥號或虛擬私人網路連線和 RADIUS 用戶端] 視窗](./media/howto-mfa-nps-extension-vpn/image10.png)

### <a name="verify-the-radius-configuration"></a>確認 RADIUS 設定
本節會詳述您使用精靈所建立的設定。

1. 在網路原則伺服器的 [NPS (本機)] 主控台中展開 [RADIUS 用戶端]，然後選取 [RADIUS 用戶端]。

2. 在詳細資料窗格中，以滑鼠右鍵按一下您所建立的 RADIUS 用戶端，然後選取 [屬性]。 RADIUS 用戶端 (VPN 伺服器) 的屬性應該類似如下所示：

    ![VPN 屬性](./media/howto-mfa-nps-extension-vpn/image11.png)

3. 選取 [取消]。

4. 在網路原則伺服器的 [NPS (本機)] 主控台中展開 [原則]，然後選取 [連線要求原則]。 即會顯示如下圖中的 VPN 連線原則：

    ![連線要求](./media/howto-mfa-nps-extension-vpn/image12.png)

5. 在 [原則] 底下選取 [網路原則]。 您應該會看到類似下圖的虛擬私人網路 (VPN) 連線原則：

    ![網路原則](./media/howto-mfa-nps-extension-vpn/image13.png)

## <a name="configure-your-vpn-server-to-use-radius-authentication"></a>將 VPN 伺服器設定為使用 RADIUS 驗證
在本節中，您要將 VPN 伺服器設定為使用 RADIUS 驗證。 本節指示假設您已有運作中的 VPN 伺服器設定，但尚未將 VPN 伺服器設定為使用 RADIUS 驗證。 在設定 VPN 伺服器之後，確認您的設定是否如預期般運作。

> [!NOTE]
> 如果您已擁有運作中的 VPN 伺服器設定，且該設定使用 RADIUS 驗證，則可以略過本節。
>

### <a name="configure-authentication-provider"></a>設定驗證提供者
1. 在 VPN 伺服器上，開啟 [伺服器管理員]。

2. 在 [伺服器管理員] 中選取 [工具]，然後選取 [路由及遠端存取]。

3. 在 [路由及遠端存取] 視窗中，以滑鼠右鍵按一下 [\<伺服器名稱> (本機)]，然後選取 [屬性]。

    ![[路由及遠端存取] 視窗](./media/howto-mfa-nps-extension-vpn/image14.png)
 
4. 在 [\<伺服器名稱> (本機) 屬性] 對話方塊中，按一下 [安全性] 索引標籤。 

5. 在 [安全性] 索引標籤的 [驗證提供者] 底下選取 [RADIUS 驗證]，然後按選取 [設定]。

    ![RADIUS 驗證](./media/howto-mfa-nps-extension-vpn/image15.png)
 
6. 在 [RADIUS 驗證] 視窗中，選取 [新增]。

7. 在 [新增 RADIUS 伺服器] 視窗中，執行下列動作：

    a. 在 [伺服器名稱] 方塊中輸入您在上一節所設定之 RADIUS 伺服器的名稱或 IP 位址。

    b. 在 [共用密碼] 中選取 [變更]，然後輸入您稍早所建立並記錄下來的共用密碼。

    c. 在 [逾時 (秒)] 方塊中，選取 **30** 到 **60** 之間的值。  
    您必須設定逾時值，才能讓第二個驗證要素有足夠的時間完成。
 
    ![[新增 RADIUS 伺服器] 視窗](./media/howto-mfa-nps-extension-vpn/image16.png)
 
8. 選取 [確定] 。

### <a name="test-vpn-connectivity"></a>測試 VPN 連線能力
在本節中，您要確認在您嘗試連線到 VPN 虛擬連接埠時，VPN 用戶端已通過 RADIUS 伺服器的驗證並獲得授權。 本節指示假設您使用 Windows 10 作為 VPN 用戶端。 

> [!NOTE]
> 如果您已將 VPN 用戶端設定為連線到 VPN 伺服器並已儲存各項設定，您可以略過與設定和儲存 VPN 連線物件有關的步驟。
>

1. 在 VPN 用戶端電腦上，選取 [開始] 按鈕，然後選取 [設定] 按鈕。

2. 在 [Windows 設定] 視窗中，選取 [網路和網際網路]。

3. 選取 [VPN]。

4. 選取 [新增 VPN 連線]。

5. 在 [新增 VPN 連線] 視窗的 [VPN 提供者] 方塊中選取 [Windows (內建)]，接著視情況完成剩餘欄位，然後選取 [儲存]。 

    ![[新增 VPN 連線] 視窗](./media/howto-mfa-nps-extension-vpn/image17.png)
 
6. 移至 [控制台]，然後選取 [網路和共用中心]。

7. 選取 [變更介面卡設定]。

    ![變更介面卡設定](./media/howto-mfa-nps-extension-vpn/image18.png)

8. 以滑鼠右鍵按一下 VPN 網路連線，然後選取 [屬性]。 

    ![VPN 網路屬性](./media/howto-mfa-nps-extension-vpn/image19.png)

9. 在 VPN 的 [屬性] 視窗中，選取 [安全性] 索引標籤。 

10. 確定 [安全性] 索引標籤上只選取了 [Microsoft CHAP 第 2 版 (MS-CHAP v2)]，然後選取 [確定]。

    ![[允許這些通訊協定] 選項](./media/howto-mfa-nps-extension-vpn/image20.png)

11. 以滑鼠右鍵按一下 VPN 連線，然後選取 [連線]。

12. 在 [設定] 視窗中，選取 [連線]。  
    如下所示，在 RADIUS 伺服器的安全性記錄中，事件識別碼 6272 會顯示連線成功：

    ![[事件屬性] 視窗](./media/howto-mfa-nps-extension-vpn/image21.png)

## <a name="troubleshooting-radius"></a>針對 RADIUS 進行疑難排解

假設在您將 VPN 伺服器設定為使用集中式的 RADIUS 伺服器來進行驗證和授權之前，您的 VPN 設定就已在運作。 若設定未運作，問題很可能是 RADIUS 伺服器的設定不正確或使用了無效的使用者名稱或密碼所導致。 例如，如果您的使用者名稱中使用替代的 UPN 尾碼，登入嘗試可能會失敗。 使用相同的帳戶名稱可獲得最佳結果。 

若要針對這些問題進行疑難排解，最好是從檢查 RADIUS 伺服器上的安全性事件記錄來著手。 若要節省花在搜尋事件的時間，您可以在事件檢視器中使用以角色為基礎的 [網路原則與存取伺服器] 自訂檢視，如下所示。 「事件識別碼 6273」指出 NPS 拒絕使用者存取的事件。 

![事件檢視器](./media/howto-mfa-nps-extension-vpn/image22.png)
 
## <a name="configure-multi-factor-authentication"></a>設定 Multi-Factor Authentication

如需設定 Multi-Factor Authentication 之使用者的協助，請參閱[如何要求使用者或群組使用雙步驟驗證](howto-mfa-userstates.md)和[對我的帳戶進行雙步驟驗證設定](../user-help/multi-factor-authentication-end-user-first-time.md)文章

## <a name="install-and-configure-the-nps-extension"></a>安裝和設定 NPS 擴充功能

本節會提供指示，以引導您將 VPN 設定為使用 MFA 讓用戶端向 VPN 伺服器進行驗證。

在安裝並設定了 NPS 擴充功能後，由此伺服器負責處理的所有 RADIUS 型用戶端驗證都必須使用 MFA。 如果所有的 VPN 使用者都未在 Azure Multi-Factor Authentication 中註冊，您可以執行下列其中一項：

* 設定另一部 RADIUS 伺服器來驗證尚未設定為使用 MFA 的使用者。 

* 建立登錄項目，來允許受到挑戰的使用者提供第二個驗證要素，但前提是他們必須已在 Azure Multi-Factor Authentication 中註冊。 

在 HKLM\SOFTWARE\Microsoft\AzureMfa 中建立名為「REQUIRE_USER_MATCH」的新字串值，然後將該值設為 *TRUE* 或 *FALSE*。 

![「需要使用者比對」設定](./media/howto-mfa-nps-extension-vpn/image34.png)
 
如果值設為 *TRUE* 或空白，則所有驗證要求都由 MFA 挑戰來決定。 如果值設為 *FALSE*，則只有已在 Azure Multi-Factor Authentication 中註冊的使用者會收到 MFA 挑戰。 在測試或生產環境中，請只在上架期間使用 *FALSE* 設定。

### <a name="obtain-the-azure-active-directory-guid-id"></a>取得 Azure Active Directory GUID 識別碼

在設定 NPS 擴充功能期間，您必須為 Azure AD 租用戶提供系統管理員認證和識別碼。 執行下列動作來取得識別碼：

1. 以 Azure 租用戶的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com)。

2. 在左窗格中選取 [Azure Active Directory] 按鈕。

3. 選取 [屬性] 。

4. 若要複製您的 Azure AD 識別碼，請選取 [複製] 按鈕。
 
    ![Azure AD 識別碼](./media/howto-mfa-nps-extension-vpn/image35.png)

### <a name="install-the-nps-extension"></a>安裝 NPS 擴充功能
NPS 擴充功能必須安裝於已安裝「網路原則與存取服務」角色，且依設計要作為 RADIUS 伺服器的伺服器上。 請「勿」將 NPS 擴充功能安裝在您的遠端桌面伺服器上。

1. 從 [Microsoft 下載中心](https://aka.ms/npsmfa)下載 NPS 擴充功能。 

2. 將安裝程式可執行檔 (*NpsExtnForAzureMfaInstaller.exe*) 複製到 NPS 伺服器。

3. 在 NPS 伺服器上按兩下 **NpsExtnForAzureMfaInstaller.exe**，在系統提示時選取 [執行]。

4. 在 [Azure MFA 安裝程式的 NPS 擴充功能] 視窗中檢閱軟體授權條款，選取 [我同意授權條款和條件] 核取方塊，然後選取 [安裝]。

    ![[Azure MFA 安裝程式的 NPS 擴充功能] 視窗](./media/howto-mfa-nps-extension-vpn/image36.png)
 
5. 在 [Azure MFA 安裝程式的 NPS 擴充功能] 視窗中，選取 [關閉]。  

    ![[安裝成功] 確認視窗](./media/howto-mfa-nps-extension-vpn/image37.png) 
 
### <a name="configure-certificates-for-use-with-the-nps-extension-by-using-a-powershell-script"></a>使用 PowerShell 指令碼來設定要用於 NPS 擴充功能的憑證
為確保通訊安全並提供保證，請設定要供 NPS 擴充功能使用的憑證。 NPS 元件納入了 Windows PowerShell 指令碼，以設定要用於 NPS 的自我簽署憑證。 

此指令碼會執行下列動作：

* 建立自我簽署憑證。
* 讓憑證的公開金鑰與 Azure AD 的服務主體產生關聯。
* 將憑證儲存在本機電腦的存放區。
* 授與網路使用者憑證私密金鑰的存取權。
* 重新啟動 NPS 服務。

如果您想要使用您自己的憑證，就必須讓憑證的公開金鑰與 Azure AD 的服務主體產生關聯，依此類推。

若要使用此指令碼，請為擴充功能提供您的 Azure Active Directory 系統管理認證以及您之前複製的 Azure Active Directory 租用戶識別碼。 在安裝了 NPS 擴充功能的每個 NPS 伺服器上執行此指令碼。

1. 以系統管理理員身分執行 Windows PowerShell。

2. 在 PowerShell 命令提示字元中輸入 **cd "c:\Program Files\Microsoft\AzureMfa\Config"**，然後選取 ENTER 鍵。

3. 在下一個命令提示字元中輸入 **.\AzureMfsNpsExtnConfigSetup.ps1**，然後選取 Enter。 此指令碼會檢查您是否已安裝 Azure AD PowerShell 模組。 如果尚未安裝此模組，指令碼就會為您安裝。
 
    ![PowerShell](./media/howto-mfa-nps-extension-vpn/image38.png)
 
    在指令碼確認您已安裝 PowerShell 模組後，它會顯示 [Azure Active Directory PowerShell 模組] 登入視窗。 

4. 輸入 Azure AD 系統管理員認證和密碼，然後選取 [登入]。 
 
    ![PowerShell 登入視窗](./media/howto-mfa-nps-extension-vpn/image39.png)
 
5. 在命令提示字元中，貼上您先前複製的租用戶識別碼，然後選取 Enter。 

    ![租用戶識別碼](./media/howto-mfa-nps-extension-vpn/image40.png)

    此指令碼會建立自我簽署憑證，並進行其他的設定變更。 輸出與下圖中的內容類似：

    ![自我簽署憑證](./media/howto-mfa-nps-extension-vpn/image41.png)

6. 重新啟動伺服器。

### <a name="verify-the-configuration"></a>驗證組態
若要驗證設定，您必須對 VPN 伺服器建立新的 VPN 連線。 成功輸入主要驗證的認證之後，VPN 連線會先等候次要驗證成功再建立連線，如下所示。 

![Windows 設定 VPN 視窗](./media/howto-mfa-nps-extension-vpn/image42.png)

如果您已成功地使用先前在 Azure MFA 中設定的次要驗證方法來進行驗證，就會連線到資源。 不過，如果次要驗證失敗，系統就會拒絕讓您存取資源。 

在下列範例中，Windows Phone 上的 Microsoft Authenticator 應用程式提供次要驗證：

![驗證帳戶](./media/howto-mfa-nps-extension-vpn/image43.png)

在成功地使用第二種方法進行驗證之後，您就能存取 VPN 伺服器上的虛擬連接埠。 因為您必須使用受信任裝置上的行動裝置應用程式來使用次要驗證方法，登入程序會比僅使用使用者名稱和密碼的組合來得更加安全。

### <a name="view-event-viewer-logs-for-successful-sign-in-events"></a>檢視事件檢視器記錄中的成功登入事件
若要檢視 Windows 事件檢視器記錄中的成功登入事件，請輸入下列 PowerShell 命令來查詢 NPS 伺服器上的 Windows 安全性記錄：

    _Get-WinEvent -Logname Security_ | where {$_.ID -eq '6272'} | FL 

![PowerShell 安全性事件檢視器](./media/howto-mfa-nps-extension-vpn/image44.png)
 
您也可以檢視安全性記錄或網路原則與存取服務自訂檢視，如下所示：

![網路原則伺服器記錄](./media/howto-mfa-nps-extension-vpn/image45.png)

在安裝了 Azure Multi-Factor Authentication NPS 擴充功能的伺服器上，您可以在 *Application and Services Logs\Microsoft\AzureMfa* 找到擴充功能專屬的事件檢視器應用程式記錄。 

    _Get-WinEvent -Logname Security_ | where {$_.ID -eq '6272'} | FL

![事件檢視器的 [事件數目] 窗格](./media/howto-mfa-nps-extension-vpn/image46.png)

## <a name="troubleshooting-guide"></a>疑難排解指南
如果設定未如預期般運作，請確認使用者是否已設為使用 MFA 以開始疑難排解。 讓使用者連線到 [Azure 入口網站](https://portal.azure.com)。 如果使用者收到次要驗證提示而且可以成功地完成驗證，您就可以排除是 MFA 設定不正確的問題。

如果 Azure MFA 對使用者有效，請檢閱相關的事件檢視器記錄。 記錄包括安全性事件記錄、閘道運作記錄，以及上一節中討論的 Azure Multi-Factor Authentication 記錄。 

顯示失敗登入事件 (事件識別碼 6273) 的安全性記錄範例如下：

![顯示失敗登入事件的安全性記錄](./media/howto-mfa-nps-extension-vpn/image47.png)

Azure Multi-Factor Authentication 記錄中的相關事件顯示如下：

![Azure Multi-Factor Authentication 記錄](./media/howto-mfa-nps-extension-vpn/image48.png)

若要執行進階的疑難排解，請參閱安裝了 NPS 服務的 NPS 資料庫格式記錄檔。 記錄檔會建立於 _%SystemRoot%\System32\Logs_ 資料夾，並以逗號分隔文字檔的形式存在。 如需這些記錄檔的說明，請參閱[解譯 NPS 資料庫格式記錄檔](https://technet.microsoft.com/library/cc771748.aspx) \(英文\)。 

這些記錄檔中的項目除非匯入到試算表或資料庫，否則將難以解譯。 您可以找到許多線上的網際網路驗證服務 (IAS) 剖析工具，以協助您解譯記錄檔。 以下是其中一個這類可下載[共享軟體應用程式](http://www.deepsoftware.com/iasviewer)的輸出： 

![共享軟體應用程式](./media/howto-mfa-nps-extension-vpn/image49.png)

如需其他疑難排解，您可以使用通訊協定分析器，例如 Wireshark 或 [Microsoft Message Analyzer](https://technet.microsoft.com/library/jj649776.aspx)。 下圖來自 Wireshark，顯示 VPN 伺服器與 NPS 伺服器之間的 RADIUS 訊息。

![Microsoft Message Analyzer](./media/howto-mfa-nps-extension-vpn/image50.png)

如需詳細資訊，請參閱[將現有的 NPS 基礎結構與 Azure Multi-Factor Authentication 整合](howto-mfa-nps-extension.md)。 

## <a name="next-steps"></a>後續步驟
[取得 Azure Multi-Factor Authentication](concept-mfa-licensing.md)

[使用 RADIUS 的遠端桌面閘道器和 Azure Multi-Factor Authentication Server](howto-mfaserver-nps-rdg.md)

[整合您的內部部署目錄與 Azure Active Directory](../hybrid/whatis-hybrid-identity.md)

