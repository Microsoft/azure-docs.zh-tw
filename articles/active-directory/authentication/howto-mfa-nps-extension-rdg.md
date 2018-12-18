---
title: 使用 MFA NPS 擴充功能來整合遠端桌面閘道 | Microsoft Docs
description: 使用 Microsoft Azure 的網路原則伺服器延伸模組，來整合遠端桌面閘道基礎結構與 Azure MFA
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 7fb69fd52c03300ee554fd903cba1670f2aa2421
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49318600"
---
# <a name="integrate-your-remote-desktop-gateway-infrastructure-using-the-network-policy-server-nps-extension-and-azure-ad"></a>使用網路原則伺服器 (NPS) 擴充功能和 Azure AD 整合遠端桌面閘道基礎結構

本文提供使用 Microsoft Azure 的網路原則伺服器 (NPS) 擴充功能來整合遠端桌面閘道基礎結構與 Azure Multi-Factor Authentication (MFA) 的詳細資訊。 

Azure 的網路原則伺服器 (NPS) 擴充功能可讓客戶使用 Azure 以雲端為基礎的 [Multi-Factor Authentication (MFA)](multi-factor-authentication.md) 來保護遠端驗證撥入使用者服務 (RADIUS) 用戶端驗證。 此解決方案提供雙步驟驗證，以便為使用者登入和交易增加第二層安全性。

本文提供逐步指示，引導您使用 Azure 的 NPS 擴充功能來整合 NPS 基礎結構與 Azure MFA。 這可為嘗試登入遠端桌面閘道的使用者能夠安全地進行驗證。 

網路原則與存取服務 (NPS) 為組織提供執行下列作業的能力：
* 定義中央位置以供管理和控制網路要求，方法是指出誰可以連線、每天的哪些時段允許連線、連線的持續時間，以及用戶端在連線時必須使用的安全性層級等等。 組織不必在每個 VPN 或遠端桌面 (RD) 閘道伺服器上指定這些原則，而是在中央位置一次指定這些原則。 RADIUS 通訊協定可提供集中式的驗證、授權和計量 (AAA)。 
* 建立並強制執行網路存取保護 (NAP) 用戶端健康原則，以判斷要讓裝置不受限制還是受限制地存取網路資源。
* 提供強制驗證和授權的方法，以供存取具有 802.1x 功能的無線存取點及乙太網路交換器。    

組織通常會使用 NPS (RADIUS) 來簡化和集中管理 VPN 原則。 不過，許多組織也會使用 NPS 來簡化和集中管理 RD 桌面連線授權原則 (RD CAP)。 

組織也可以整合 NPS 與 Azure MFA 來增強安全性，並提供高層級的合規性。 這有助於確保使用者建立雙步驟驗證來登入遠端桌面閘道。 使用者若要獲得存取權，就必須提供其使用者名稱/密碼的組合以及使用者所掌握的資訊。 這項資訊必須能夠讓人信任且無法輕易複製，例如行動電話號碼、室內電話號碼、行動裝置上的應用程式等等。

在 Azure 的 NPS 擴充功能推出前，想要對整合式的 NPS 與 Azure MFA 環境實作雙步驟驗證的客戶，必須在內部部署環境中另外設定及維護一個 MFA Server，如[使用 RADIUS 的遠端桌面閘道和 Azure Multi-Factor Authentication Server](howto-mfaserver-nps-rdg.md)所述。

隨著 Azure 的 NPS 擴充功能推出，組織現在可以選擇要部署內部部署型 MFA 解決方案還是雲端型 MFA 解決方案，以保護 RADIUS 用戶端驗證。

## <a name="authentication-flow"></a>驗證流程

對於要取得透過遠端桌面閘道存取網路資源之權限的使用者，他們必須符合在一個 RD 連線授權原則 (RD CAP) 和一個 RD 資源授權原則 (RD RAP) 中所指定的條件。 RD CAP 指定誰獲得授權連線到 RD 閘道。 RD RAP 指定允許使用者透過 RD 閘道連線的網路資源，例如遠端桌面或遠端應用程式。 

您可以將 RD 閘道設定為對 RD CAP 使用中央原則存放區。 RD RAP 無法使用中央原則，因為它們會在 RD 閘道上進行處理。 另一部當作中央原則存放區之 NPS 伺服器的 RADIUS 用戶端，即為設定要為對 RD CAP 使用中央原則存放區的 RD 閘道範例之一。

當 Azure 的 NPS 擴充功能與 NPS 和遠端桌面閘道整合時，成功的驗證流程如下所示：

1. 遠端桌面閘道伺服器會收到遠端桌面使用者要連線到某項資源 (例如遠端桌面工作階段) 的驗證要求。 作為 RADIUS 用戶端的遠端桌面閘道伺服器，會將此要求轉換為 RADIUS Access-Request 訊息，並將此訊息傳送至 NPS 擴充功能安裝所在的 RADIUS (NPS) 伺服器。 
1. 使用者名稱和密碼組合會在 Active Directory 中進行驗證，且使用者已經過驗證。
1. 如果 NPS 連線要求與網路原則中所指定的所有條件 (例如，一天當中的時間或群組成員資格限制) 皆能符合，NPS 擴充功能就會觸發要求，以便使用 Azure MFA 進行第二項驗證。 
1. Azure MFA 會與 Azure AD 通訊、擷取使用者的詳細資料，並使用使用者所設定的方法執行第二項驗證 (簡訊、行動裝置應用程式等等)。 
1. 成功通過 MFA 挑戰後，Azure MFA 會將結果告知 NPS 擴充功能。
1. 安裝擴充功能的 NPS 伺服器會將 RD CAP 原則的 RADIUS Access-Accept 訊息傳送至遠端桌面閘道伺服器。
1. 使用者便取得透過 RD 閘道存取要求之網路資源的權限。

## <a name="prerequisites"></a>必要條件
本節會詳述在整合 Azure MFA 與遠端桌面閘道之前所需具備的必要條件。 開始之前，您必須先具備下列必要條件。  

* 遠端桌面服務 (RDS) 基礎結構
* Azure MFA 授權
* Windows Server 軟體
* 網路原則與存取服務 (NPS) 角色
* 與內部部署 Active Directory 同步的 Azure Active Directory
* Azure Active Directory GUID 識別碼

### <a name="remote-desktop-services-rds-infrastructure"></a>遠端桌面服務 (RDS) 基礎結構
您必須備妥中運作中的遠端桌面服務 (RDS) 基礎結構。 如果您未這麼做，您可以使用下列快速入門範本，在 Azure 中快速建立此基礎結構：[建立遠端桌面工作階段集合部署](https://github.com/Azure/azure-quickstart-templates/tree/ad20c78b36d8e1246f96bb0e7a8741db481f957f/rds-deployment)。 

如果您想要以手動方式快速建立內部部署 RDS 基礎結構以供測試，請遵循下列步驟來部署一個。 
**深入了解**：[使用 Azure 快速入門部署 RDS](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/rds-in-azure)和[基本 RDS 基礎結構部署](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/rds-deploy-infrastructure)。 

### <a name="azure-mfa-license"></a>Azure MFA 授權
需要 Azure MFA 的授權，此授權可透過 Azure AD Premium、Enterprise Mobility plus Security (EMS) 或 MFA 訂用帳戶來獲得。 Azure MFA 以使用情況為基礎的授權 (例如每位使用者或每次驗證授權) 與 NPS 擴充功能並不相容。 如需詳細資訊，請參閱[如何取得 Azure Multi-Factor Authentication](concept-mfa-licensing.md)。 若要進行測試，您可以使用試用版訂用帳戶。 

### <a name="windows-server-software"></a>Windows Server 軟體
NPS 擴充功能需要安裝了 NPS 角色服務的 Windows Server 2008 R2 SP1 或更新版本。 這一節中的所有步驟均使用 Windows Server 2016 來執行。

### <a name="network-policy-and-access-services-nps-role"></a>網路原則與存取服務 (NPS) 角色
NPS 角色服務可提供 RADIUS 伺服器和用戶端功能，以及網路存取原則健康情況服務。 此角色必須安裝於您基礎結構中的至少兩部電腦上：遠端桌面閘道和另一個成員伺服器或網域控制站。 根據預設，此角色已存在於設定為遠端桌面閘道的電腦上。  您也必須至少在另一部電腦 (例如網域控制站或成員伺服器) 上安裝 NPS 角色。

如需有關安裝 NPS 角色服務 Windows Server 2012 或更舊版本的資訊，請參閱[安裝 NAP 健康原則伺服器](https://technet.microsoft.com/library/dd296890.aspx)。 如需 NPS 最佳做法的說明 (包括在網域控制站上安裝 NPS 的建議)，請參閱 [NPS 的最佳做法](https://technet.microsoft.com/library/cc771746)。

### <a name="azure-active-directory-synched-with-on-premises-active-directory"></a>與內部部署 Active Directory 同步的 Azure Active Directory
若要使用 NPS 擴充功能，內部部署使用者必須與 Azure AD 保持同步並啟用 MFA。 這一節假設內部部署使用者已使用 AD Connect 來與 Azure AD 保持同步。 如需 Azure AD Connect 的相關資訊，請參閱[整合您的內部部署目錄與 Azure Active Directory](../hybrid/whatis-hybrid-identity.md)。 

### <a name="azure-active-directory-guid-id"></a>Azure Active Directory GUID 識別碼
若要安裝 NPS 擴充功能，您必須知道 Azure AD 的 GUID。 下面提供尋找 Azure AD GUID 的指示。

## <a name="configure-multi-factor-authentication"></a>設定 Multi-Factor Authentication 
這一節提供用於整合 Azure MFA 與遠端桌面閘道的指示。 身為管理員的您必須先設定 Azure MFA 服務，使用者才能自行註冊其多因素裝置或應用程式。

請遵循[開始在雲端使用 Azure Multi-factor Authentication](howto-mfa-getstarted.md)中的步驟，以對 Azure AD 使用者啟用 MFA。 

### <a name="configure-accounts-for-two-step-verification"></a>為帳戶設定雙步驟驗證
在帳戶啟用 MFA 之後，您便無法登入 MFA 原則所控管的資源，除非您成功地設定受信任的裝置來作為第二個驗證要素，並使用雙步驟驗證通過驗證。

遵循 [Azure Multi-factor Authentication 什麼適合我？](../user-help/multi-factor-authentication-end-user.md)中的步驟，了解及正確設定您的裝置，以便透過您的使用者帳戶進行 MFA。

## <a name="install-and-configure-nps-extension"></a>安裝和設定 NPS 擴充功能
本節會提供指示，引導您將 RDS 基礎結構設定為使用 Azure MFA 讓用戶端向遠端桌面閘道進行驗證。

### <a name="acquire-azure-active-directory-guid-id"></a>取得 Azure Active Directory GUID 識別碼

在設定 NPS 擴充功能期間，您必須為 Azure AD 租用戶提供管理員認證和 Azure AD 識別碼。 下列步驟說明如何取得租用戶識別碼。

1. 以 Azure 租用戶的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com)。
1. 在左方瀏覽窗格中，選取 [Azure Active Directory] 圖示。
1. 選取 [屬性] 。
1. 在 [屬性] 刀鋒視窗中，按一下 [目錄識別碼] 旁邊的 [複製] 圖示 (如下所示)，將識別碼複製到剪貼簿。

 ![properties](./media/howto-mfa-nps-extension-rdg/image1.png)

### <a name="install-the-nps-extension"></a>安裝 NPS 擴充功能
在已安裝網路原則與存取服務 (NPS) 角色的伺服器上安裝 NPS 擴充功能。 這可當作您的設計的 RADIUS 伺服器。 

>[!Important]
> 切勿在您的遠端桌面閘道伺服器上安裝 NPS 擴充功能。
> 

1. 下載 [NPS 擴充功能](https://aka.ms/npsmfa)。 
1. 將安裝程式可執行檔 (NpsExtnForAzureMfaInstaller.exe) 複製到 NPS 伺服器。
1. 在 NPS 伺服器上按兩下 **NpsExtnForAzureMfaInstaller.exe**。 出現提示時，按一下 [執行]。
1. 檢閱 [Azure MFA 安裝程式的 NPS 擴充功能] 對話方塊中的軟體授權條款，勾選 [我同意授權條款和條件]，然後按一下 [安裝]。
 
  ![Azure MFA 設定](./media/howto-mfa-nps-extension-rdg/image2.png)

1. 在 [Azure MFA 安裝程式的 NPS 擴充功能] 對話方塊中，按一下 [關閉]。 

  ![Azure MFA 的 NPS 擴充功能](./media/howto-mfa-nps-extension-rdg/image3.png)

### <a name="configure-certificates-for-use-with-the-nps-extension-using-a-powershell-script"></a>使用 PowerShell 指令碼來設定要用於 NPS 擴充功能的憑證
接下來，您必須設定要供 NPS 擴充功能使用的憑證，以確保通訊安全並提供保證。 NPS 元件納入了 Windows PowerShell 指令碼，以設定要用於 NPS 的自我簽署憑證。 

此指令碼會執行下列動作：

* 建立自我簽署憑證
* 讓憑證的公開金鑰與 Azure AD 的服務主體產生關聯
* 將憑證儲存在本機電腦的存放區
* 將憑證的私密金鑰存取權授與給網路使用者
* 重新啟動網路原則伺服器服務

如果您想要使用您自己的憑證，就必須讓該憑證的公開金鑰與 Azure AD 的服務主體產生關聯，依此類推。

若要使用此指令碼，請為擴充功能提供您的 Azure AD 系統管理認證以及您之前複製的 Azure AD 租用戶識別碼。 在已安裝 NPS 擴充功能的每個 NPS 伺服器上執行此指令碼。 然後執行以下動作：

1. 開啟系統管理 Windows PowerShell 提示字元。
1. 在 PowerShell 提示字元中，輸入 `cd ‘c:\Program Files\Microsoft\AzureMfa\Config’`，然後按 **ENTER** 鍵。
1. 輸入 `.\AzureMfsNpsExtnConfigSetup.ps1`，然後按 **ENTER** 鍵。 此指令碼會檢查您是否已安裝 Azure Active Directory PowerShell 模組。 如果尚未安裝此模組，指令碼就會為您安裝。

  ![Azure AD PowerShell](./media/howto-mfa-nps-extension-rdg/image4.png)
  
1. 在指令碼確認您已安裝 PowerShell 模組後，它會顯示 [Azure Active Directory PowerShell 模組] 對話方塊。 在對話方塊中，輸入您的 Azure 系統管理認證和密碼，然後按一下 [登入]。

  ![開啟 Powershell 帳戶](./media/howto-mfa-nps-extension-rdg/image5.png)

1. 出現提示時，貼上您之前複製到剪貼簿的租用戶識別碼，然後按 **ENTER**。

  ![輸入租用戶識別碼](./media/howto-mfa-nps-extension-rdg/image6.png)

1. 此指令碼會建立自我簽署憑證，並進行其他的設定變更。 其輸出應類似下圖所示。

  ![自我簽署憑證](./media/howto-mfa-nps-extension-rdg/image7.png)

## <a name="configure-nps-components-on-remote-desktop-gateway"></a>設定遠端桌面閘道上的 NPS 元件
在本節中，您可以設定遠端桌面閘道的連線授權原則和其他 RADIUS 設定。

驗證流程要求在遠端桌面閘道與 NPS Server 安裝所在的 NPS 伺服器之間交換 RADIUS 訊息。 這表示您必須在遠端桌面閘道和 NPS 擴充功能安裝所在的 NPS 伺服器上設定 RADIUS 用戶端設定。 

### <a name="configure-remote-desktop-gateway-connection-authorization-policies-to-use-central-store"></a>設定遠端桌面閘道連線授權原則以使用中央存放區
遠端桌面連線授權原則 (RD CAP) 會指定連線至遠端桌面閘道伺服器的需求。 RD CAP 可以儲存在本機 (預設值)，也可以儲存在執行 NPS 的中央 RD CAP 存放區中。 若要設定 Azure MFA 與 RDS 的整合，您必須指定使用中央存放區。

1. 在 RD 閘道伺服器上，開啟 [伺服器管理員]。 
1. 在功能表上，按一下 [工具]，指向 [遠端桌面服務]，然後按一下 [遠端桌面閘道管理員]。

  ![遠端桌面服務問題](./media/howto-mfa-nps-extension-rdg/image8.png)

1. 在 [RD 閘道管理員] 中，以滑鼠右鍵按一下 **\[伺服器名稱\] (本機)**，然後按一下 [屬性]。

  ![伺服器名稱](./media/howto-mfa-nps-extension-rdg/image9.png)

1. 在 [屬性] 對話方塊中，選取 [RD CAP 存放區] 索引標籤。
1. 在 [RD CAP 存放區] 索引標籤上，選取 [執行 NPS 的中央伺服器]。 
1. 在 [輸入執行 NPS 之伺服器的名稱或 IP 位址] 欄位中，輸入您安裝 NPS 擴充功能之伺服器的 IP 位址或伺服器名稱。

  ![輸入名稱或 IP 位址](./media/howto-mfa-nps-extension-rdg/image10.png)
  
1. 按一下 [新增] 。
1. 在 [共用祕密] 對話方塊中，輸入共用祕密，然後按一下 [確定]。 務必記錄此共用祕密並安全地儲存記錄。

 >[!NOTE]
 >共用祕密用於建立 RADIUS 伺服器與用戶端之間的信任。 建立長而複雜的密碼。
 >

 ![共用祕密](./media/howto-mfa-nps-extension-rdg/image11.png)

1. 按一下 [確定] 關閉對話方塊。

### <a name="configure-radius-timeout-value-on-remote-desktop-gateway-nps"></a>在遠端桌面閘道 NPS 上設定 RADIUS 逾時值
若要確保有時間驗證使用者的認證，請執行雙步驟驗證、接收回應，然後回應 RADIUS 訊息，而且一定要調整 RADIUS 逾時值。

1. 在 RD 閘道伺服器上，開啟 [伺服器管理員]。 在功能表中，按一下 [工具]，然後按一下 [網路原則伺服器]。 
1. 在 [NPS (本機)] 主控台中，展開 [RADIUS 用戶端和伺服器]，然後選取 [遠端 RADIUS 伺服器]。

 ![遠端 RADIUS 伺服器](./media/howto-mfa-nps-extension-rdg/image12.png)

1. 在詳細資料窗格中，按兩下 [TS GATEWAY SERVER GROUP]。

 >[!NOTE]
 >設定 NPS 原則的中央伺服器時，已建立此 RADIUS 伺服器群組。 RD 閘道會將 RADIUS 訊息轉送到此伺服器或伺服器群組 (如果群組中超過一個伺服器)。
 >

1. 在 [TS GATEWAY SERVER GROUP 屬性] 對話方塊中，選取您設定用來儲存 RD CAP 之 NPS 伺服器的 IP 位址或名稱，然後按一下 [編輯]。 

 ![TS Gateway Server Group](./media/howto-mfa-nps-extension-rdg/image13.png)

1. 在 [編輯 RADIUS 伺服器] 對話方塊中，選取 [負載平衡] 索引標籤。
1. 在 [負載平衡] 索引標籤的 [要求被丟棄前，無回應的秒數] 欄位中，將預設值從 3 變更為介於 30 與 60 秒之間的值。
1. 在 [伺服器被識別為無法使用時，要求之間的間隔秒數] 欄位中，將預設值 30 秒變更為等於或大於您在上一個步驟中指定的值。

 ![編輯 Radius 伺服器](./media/howto-mfa-nps-extension-rdg/image14.png)

1.  按兩次 [確定] 以關閉對話方塊。

### <a name="verify-connection-request-policies"></a>驗證連線要求原則 
根據預設，當您將 RD 閘道設定為使用連線授權原則的中央原則存放區時，則 RD 閘道會被設定為將 CAP 要求轉送到 NPS 伺服器。 已安裝 Azure MFA 擴充功能的 NPS 伺服器，會處理 RADIUS 存取要求。 下列步驟顯示如何確認預設連線要求原則。 

1. 在 RD 閘道上，於 [NPS (本機)] 主控台中展開 [原則]，然後選取 [連線要求原則]。
1. 按兩下 [TS 閘道授權原則]。
1. 在 [TS GATEWAY AUTHORIZATION POLICY 屬性] 對話方塊中，按一下 [設定] 索引標籤。
1. 在 [設定] 索引標籤上，按一下 [轉送連線要求] 之下的 [驗證]。 RADIUS 用戶端會設定為轉送要求進行驗證。

 ![驗證設定](./media/howto-mfa-nps-extension-rdg/image15.png)
 
1. 按一下 [取消]。 

## <a name="configure-nps-on-the-server-where-the-nps-extension-is-installed"></a>在安裝 NPS 擴充功能的伺服器上設定 NPS
安裝 NPS 擴充功能的 NPS 伺服器必須能夠與遠端桌面閘道上的 NPS 伺服器交換 RADIUS 訊息。 若要啟用此訊息交換，您必須在安裝 NPS 擴充服務的伺服器上設定 NPS 元件。 

### <a name="register-server-in-active-directory"></a>在 Active Directory 中註冊伺服器
若要讓 NPS 伺服器在本案例中正常運作，您必須在 Active Directory 中加以註冊。

1. 在 NPS 伺服器上，開啟 [伺服器管理員]。
1. 在 [伺服器管理員] 中按一下 [工具]，然後按一下 [網路原則伺服器]。 
1. 在 [網路原則伺服器] 主控台中，以滑鼠右鍵按一下 [NPS (本機)]，然後按一下 [在 Active Directory 中註冊伺服器]。 
1. 按 [確定] 兩次。

 ![在 AD 中註冊伺服器](./media/howto-mfa-nps-extension-rdg/image16.png)

1. 讓主控台保持開啟以供下一個程序使用。

### <a name="create-and-configure-radius-client"></a>建立及設定 RADIUS 用戶端 
必須將遠端桌面閘道設定為 NPS 伺服器的 RADIUS 用戶端。 

1. 在安裝 NPS 擴充功能的 NPS 伺服器上，於 [NPS (本機)] 主控台中，以滑鼠右鍵按一下 [RADIUS 用戶端] 並按一下 [新增]。

 ![新增 RADIUS 用戶端](./media/howto-mfa-nps-extension-rdg/image17.png)

1. 在 [新增 RADIUS 用戶端] 對話方塊中，提供易記的名稱 (例如 Gateway)，以及遠端桌面閘道伺服器的 IP 位址或 DNS 名稱。 
1. 在 [共用祕密] 和 [確認共用祕密] 欄位中，輸入您之前使用的相同祕密。

 ![名稱和位址](./media/howto-mfa-nps-extension-rdg/image18.png)

1. 按一下 [確定] 關閉 [新增 RADIUS 用戶端] 對話方塊。

### <a name="configure-network-policy"></a>設定網路原則
請記住，具備 Azure MFA 擴充功能的 NPS 伺服器是連線授權原則 (CAP) 的指定中央原則存放區。 因此，您需要在 NPS 伺服器上實作 CAP，才能授權有效的連線要求。  

1. 在 NPS 伺服器上，開啟 [NPS (本機)] 主控台，展開 [原則]，然後按一下 [網路原則]。
1. 以滑鼠右鍵按一下 [其他存取伺服器的連線]，然後按一下 [複製原則]。 

 ![複製原則](./media/howto-mfa-nps-extension-rdg/image19.png)

1. 以滑鼠右鍵按一下 [複製其他存取伺服器的連線]，然後按一下 [屬性]。

 ![網路屬性](./media/howto-mfa-nps-extension-rdg/image20.png)

1. 在 [複製其他存取伺服器的連線] 對話方塊的 [原則名稱] 中，輸入適當的名稱，例如 _RDG_CAP_。 勾選 [啟用原則]，然後選取 [授與存取權]。 (選擇性) 在 [網路存取伺服器類型] 中選取 [遠端桌面閘道]，您也可以將它保留為 [未指定]。

 ![複製連線](./media/howto-mfa-nps-extension-rdg/image21.png)

1.  按一下 [條件約束] 索引標籤，然後勾選 [允許用戶端沒有交涉驗證方法仍然可以連線]。

 ![允許用戶端連線](./media/howto-mfa-nps-extension-rdg/image22.png)

1. (選擇性) 按一下 [條件] 索引標籤，並新增必須符合才能授權連線的條件，例如，特定 Windows 群組中的成員資格。

 ![條件](./media/howto-mfa-nps-extension-rdg/image23.png)

1. 按一下 [確定]。 當系統提示您檢視對應的說明主題時，請按一下 [否]。
1. 請確定新原則位於清單的頂端，已啟用原則，而且它會授與存取權。

 ![網路原則](./media/howto-mfa-nps-extension-rdg/image24.png)

## <a name="verify-configuration"></a>驗證組態
若要驗證組態，您必須使用適當的 RDP 用戶端登入遠端桌面閘道。 請務必使用您的連線授權原則所允許的帳戶並已啟用 Azure MFA。 

如下圖所示，您可以使用 [遠端桌面 Web 存取] 頁面。

![遠端桌面 Web 存取](./media/howto-mfa-nps-extension-rdg/image25.png)

在成功輸入您的認證進行主要驗證時，[遠端桌面連線] 對話方塊會顯示 [起始遠端連線] 的狀態，如下所示。 

如果您已成功地使用先前在 Azure MFA 中設定的次要驗證方法進行驗證，您就會連線到資源。 不過，如果次要驗證失敗，系統就會拒絕讓您存取資源。 

![起始遠端連線](./media/howto-mfa-nps-extension-rdg/image26.png)

在下列範例中，我們使用了 Windows Phone 上的 Authenticator 應用程式來提供次要驗證。

![帳戶](./media/howto-mfa-nps-extension-rdg/image27.png)

在使用第二種驗證方法驗證成功之後，您便已如往常一樣登入遠端桌面閘道。 不過，因為您必須使用受信任裝置上的行動裝置應用程式來使用次要驗證方法，登入程序會比使用其他方法來得更加安全。

### <a name="view-event-viewer-logs-for-successful-logon-events"></a>檢視事件檢視器記錄來找到成功的登入事件
若要檢視 Windows 事件檢視器記錄中的成功登入事件，您可以發出下列 Windows PowerShell 命令來查詢 Windows 終端機服務和 Windows 安全性記錄。

若要查詢閘道操作記錄 (Event Viewer\Applications and Services Logs\Microsoft\Windows\TerminalServices-Gateway\Operational) 中的成功登入事件，使用下列 PowerShell 命令：

* _Get-WinEvent -Logname Microsoft-Windows-TerminalServices-Gateway/Operational_ | where {$_.ID -eq '300'} | FL 
* 此命令顯示的 Windows 事件顯示使用者符合資源授權原則需求 (RD RAP) 並已取得存取權。

![檢視事件檢視器](./media/howto-mfa-nps-extension-rdg/image28.png)

* _Get-WinEvent -Logname Microsoft-Windows-TerminalServices-Gateway/Operational_ | where {$_.ID -eq '200'} | FL
* 此命令會顯示使用者符合連線授權原則需求時顯示的事件。

![連線授權](./media/howto-mfa-nps-extension-rdg/image29.png)

您也可以檢視此記錄並依據事件識別碼 (300 和 200) 進行篩選。 若要查詢安全性事件檢視器記錄中的成功登入事件，請使用下列命令：

* _Get-WinEvent -Logname Security_ | where {$_.ID -eq '6272'} | FL 
* 此命令可以在中央 NPS 或 RD 閘道伺服器上執行。 

![成功的登入事件](./media/howto-mfa-nps-extension-rdg/image30.png)

您也可以檢視安全性記錄或網路原則與存取服務自訂檢視，如下所示：

![網路原則與存取服務](./media/howto-mfa-nps-extension-rdg/image31.png)

在安裝了 Azure MFA NPS 擴充功能的伺服器上，您可以在 _Application and Services Logs\Microsoft\AzureMfa_ 找到擴充功能專屬的事件檢視器應用程式記錄。 

![事件檢視器應用程式記錄](./media/howto-mfa-nps-extension-rdg/image32.png)

## <a name="troubleshoot-guide"></a>疑難排解指南

如果組態未如預期般運作，確認使用者是否已設為使用 Azure MFA 是疑難排解的起點。 讓使用者連線到 [Azure 入口網站](https://portal.azure.com)。 如果使用者收到次要驗證提示而且可以成功地完成驗證，您就可以排除 Azure MFA 設定不正確的可能性。

如果 Azure MFA 對使用者有效，請檢閱相關的事件記錄。 這些記錄包括安全性事件記錄、閘道運作記錄，以及上一節中討論的 Azure MFA 記錄。 

以下是安全性記錄的輸出範例，其中顯示了失敗登入事件 (事件識別碼 6273)。

![失敗的登入事件](./media/howto-mfa-nps-extension-rdg/image33.png)

以下是來自 Azure MFA 記錄的相關事件：

![Azure MFA 記錄](./media/howto-mfa-nps-extension-rdg/image34.png)

若要執行進階的疑難排解選項，請參閱安裝了 NPS 服務的 NPS 資料庫格式記錄檔。 這些記錄檔會建立於 %SystemRoot%\System32\Logs 資料夾，並以逗號分隔文字檔的形式存在。 

如需這些記錄檔的說明，請參閱[解譯 NPS 資料庫格式記錄檔](https://technet.microsoft.com/library/cc771748.aspx)。 這些記錄檔中的項目若不匯入到試算表或資料庫，將難以解譯。 您可以在線上找到數個 IAS 剖析器來協助您解譯記錄檔。 

下圖顯示其中一個這類可下載[共享軟體應用程式](http://www.deepsoftware.com/iasviewer)的輸出。 

![共享軟體應用程式](./media/howto-mfa-nps-extension-rdg/image35.png)

最後，如需其他疑難排解選項，您可以使用通訊協定分析器，例如 [Microsoft Message Analyzer](https://technet.microsoft.com/library/jj649776.aspx)。 

以下的 Microsoft Message Analyzer 影像顯示依據 RADIUS 通訊協定篩選並包含使用者名稱 **Contoso\AliceC** 的網路流量。

![Microsoft Message Analyzer](./media/howto-mfa-nps-extension-rdg/image36.png)

## <a name="next-steps"></a>後續步驟
[如何取得 Azure Multi-Factor Authentication](concept-mfa-licensing.md)

[使用 RADIUS 的遠端桌面閘道器和 Azure Multi-Factor Authentication Server](howto-mfaserver-nps-rdg.md)

[整合您的內部部署目錄與 Azure Active Directory](../hybrid/whatis-hybrid-identity.md)
