---
title: 開始使用 Azure Multi-Factor Authentication Server | Microsoft Docs
description: 逐步開始進行 Azure MFA Server 內部部署
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: d69addac0647513e00663110ac82eef343b77983
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364613"
---
# <a name="getting-started-with-the-azure-multi-factor-authentication-server"></a>開始使用 Azure Multi-Factor Authentication Server

<center>![MFA 內部部署](./media/howto-mfaserver-deploy/server2.png)</center>

既然我們已經決定要使用內部部署 Multi-Factor Authentication Server，那麼我們就開始著手進行吧。 此頁面探討新伺服器安裝，以及透過內部部署 Active Directory 予以設定。 如果您已經安裝 MFA 伺服器，且想要升級，請參閱[升級至最新的 Azure Multi-factor Authentication Server](howto-mfaserver-deploy-upgrade.md)。 如果您想要尋找僅是安裝 Web 服務的資訊，請參閱[部署 Azure Multi-Factor Authentication Server 行動應用程式 Web 服務](howto-mfaserver-deploy-mobileapp.md)。

## <a name="plan-your-deployment"></a>規劃您的部署

下載 Azure Multi-Factor Authentication Server 之前，請考慮您的負載和高可用性需求為何。 使用此資訊來決定部署的方式和位置。

理想的指導方針：您需要的記憶體數量就是您預期會定期驗證的使用者數目。

| 使用者 | RAM |
| ----- | --- |
| 1-10,000 | 4 GB |
| 10,001-50,000 | 8 GB |
| 50,001-100,000 | 12 GB |
| 100,000-200,001 | 16 GB |
| 200,001+ | 32 GB |

您是否需要設定多部伺服器以取得高可用性或負載平衡？ 使用 Azure MFA Server 設定此組態的方法好幾種。 當您安裝第一個 Azure MFA Server 時，它會變成主要伺服器。 其他所有伺服器都會變成從屬伺服器，而且自動同步處理主要伺服器的使用者和組態。 然後，您可以設定一部主要伺服器，而讓其餘伺服器當作備份，您也可以設定所有伺服器之間的負載平衡。

當主要 Azure MFA Server 離線時，從屬伺服器仍然可以處理雙步驟的驗證要求。 不過，在主要伺服器回到上線狀態或從屬伺服器升階之前，您無法新增使用者，而現有使用者無法更新其設定。

### <a name="prepare-your-environment"></a>準備您的環境

請確定您用於 Azure Multi-Factor Authentication Server 的伺服器符合下列需求：

| Azure Multi-Factor Authentication Server 需求 | 說明 |
|:--- |:--- |
| 硬體 |<li>200 MB 的硬碟空間</li><li>具有 x32 或 x64 功能的處理器</li><li>1 GB 或更高的 RAM</li> |
| 軟體 |<li>Windows Server 2016</li><li>Windows Server 2012 R2</li><li>Windows Server 2012</li><li>Windows Server 2008 R2</li><li>Windows Server 2008，SP1、SP2</li><li>Windows Server 2003 R2</li><li>Windows Server 2003，SP1、SP2</li><li>Windows 10</li><li>Windows 8.1，所有版本</li><li>Windows 8，所有版本</li><li>Windows 7，所有版本</li><li>Windows Vista，所有版本，SP1、SP2</li><li>Microsoft .NET 4.0 Framework</li><li>如果安裝使用者入口網站或 Web 服務 SDK，則為 IIS 7.0 或更高版本</li> |
| 權限 | 網域系統管理員或企業系統管理員以向 Active Directory 註冊 |

### <a name="azure-mfa-server-components"></a>Azure MFA Server 元件

構成 Azure MFA Server 的 Web 元件有三個：

* Web 服務 SDK - 能夠與其他元件通訊，而且安裝於 Azure MFA 應用程式伺服器上
* 使用者入口網站 - 一個 IIS 網站，可讓使用者註冊 Azure Multi-Factor Authentication (MFA) 及維護自己的帳戶。
* 行動裝置應用程式 Web 服務 - 能夠使用行動裝置應用程式 (例如 Microsoft Authenticator 應用程式) 進行雙步驟驗證。

如果伺服器是網際網路對向伺服器，則可將三個元件安裝在相同的伺服器上。 如果拆散這些元件，Web 服務 SDK 會安裝於 Azure MFA 應用程式伺服器上，而使用者入口網站和行動裝置應用程式 Web 服務會安裝於網際網路對向伺服器上。

### <a name="azure-multi-factor-authentication-server-firewall-requirements"></a>Azure Multi-Factor Authentication Server 防火牆需求

每部 MFA Server 都必須要能夠透過連接埠 443 與以下位址進行輸出通訊：

* https://pfd.phonefactor.net
* https://pfd2.phonefactor.net
* https://css.phonefactor.net

如果將輸出防火牆限制在連接埠 443，請開啟以下 IP 位址範圍：

| IP 子網路 | 網路遮罩 | IP 範圍 |
|:---: |:---: |:---: |
| 134.170.116.0/25 |255.255.255.128 |134.170.116.1 – 134.170.116.126 |
| 134.170.165.0/25 |255.255.255.128 |134.170.165.1 – 134.170.165.126 |
| 70.37.154.128/25 |255.255.255.128 |70.37.154.129 – 70.37.154.254 |

如果您不會使用事件確認功能，而且使用者不會使用行動應用程式從公司網路上的裝置進行驗證，您只需要下列範圍︰

| IP 子網路 | 網路遮罩 | IP 範圍 |
|:---: |:---: |:---: |
| 134.170.116.72/29 |255.255.255.248 |134.170.116.72 – 134.170.116.79 |
| 134.170.165.72/29 |255.255.255.248 |134.170.165.72 – 134.170.165.79 |
| 70.37.154.200/29 |255.255.255.248 |70.37.154.201 – 70.37.154.206 |

## <a name="download-the-mfa-server"></a>下載 MFA Server

請遵循下列步驟，以從 Azure 入口網站下載 Azure Multi-Factor Authentication Server：

1. 以系統管理員身分登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [Azure Active Directory] > [MFA Server]。
3. 選取 [伺服器設定]。
4. 選取 [下載] 並遵循下載頁面上的指示，以儲存安裝程式。 

   ![下載 MFA Server](./media/howto-mfaserver-deploy/downloadportal.png)

5. 讓此頁面保持開啟，我們會在執行安裝程式之後參照該頁面。

## <a name="install-and-configure-the-mfa-server"></a>安裝和設定 MFA Server

既然您已下載伺服器，現在可以進行安裝和設定。 請確定要安裝的伺服器符合規劃區段中所列的需求。

1. 按兩下可執行檔。
2. 在 [選取安裝資料夾] 畫面中，請確認資料夾正確，然後按一下 [下一步]。
3. 當安裝完成時，請按一下 [完成]。 組態精靈就會啟動。
4. 在組態精靈歡迎畫面上，核取 [略過使用驗證設定精靈]，然後按一下 [下一步]。 精靈關閉然後伺服器啟動。

   ![雲端](./media/howto-mfaserver-deploy/skip2.png)

5. 回到您下載伺服器的頁面，按一下 [產生啟用認證] 按鈕。 將此資訊複製到 Azure MFA Server 提供的方塊中，然後按一下 [ **啟用**]。

## <a name="send-users-an-email"></a>傳送電子郵件給使用者

為了簡化推出，允許 MFA Server 與您的使用者通訊。 MFA Server 可以傳送電子郵件，通知他們已為其註冊雙步驟驗證。

要傳送什麼電子郵件應取決於您如何為使用者設定雙步驟驗證。 例如，假設您可以從公司目錄匯入電話號碼，則電子郵件中應該包含預設電話號碼，讓使用者知道應該會有什麼。 如果您未匯入電話號碼，或使用者將使用行動裝置應用程式，請對他們傳送可引導他們完成帳戶註冊的電子郵件。 在電子郵件中包含 Azure Multi-factor Authentication 使用者入口網站的超連結。

電子郵件的內容也會隨著已為使用者設定的驗證方法而不同 (電話、簡訊或行動應用程式)。 例如，如果使用者驗證時需要使用 PIN，電子郵件會告訴他們已設定的初始 PIN。 使用者必須在第一次驗證時變更他們的 PIN。

### <a name="configure-email-and-email-templates"></a>設定電子郵件和電子郵件範本

按一下左邊的電子郵件圖示，即可設定用來傳送這些電子郵件的設定。 此頁面可讓您輸入郵件伺服器的 SMTP 資訊，並透過核取 [傳送電子郵件給使用者] 核取方塊來傳送電子郵件。

![MFA Server 電子郵件組態](./media/howto-mfaserver-deploy/email1.png)

在 [電子郵件內容] 索引標籤中，您會看到可供選擇的電子郵件範本。 請根據您是如何設定使用者來執行雙步驟驗證，選擇最適合您使用的範本。

![MFA Server 電子郵件範本](./media/howto-mfaserver-deploy/email2.png)

## <a name="import-users-from-active-directory"></a>從 Active Directory 匯入使用者

現在已安裝伺服器，您需要新增使用者。 您可以選擇手動建立使用者、從 Active Directory 匯入使用者，或設定自動與 Active Directory 同步處理。

### <a name="manual-import-from-active-directory"></a>從 Active Directory 手動匯入

1. 在 Azure MFA Server 的左側選取 [ **使用者**]。
2. 在底部選取 [ **從 Active Directory 匯入**]。
3. 現在您可以搜尋個別使用者，或在 AD 中搜尋含使用者的 OU。 在此情況下，我們會指定使用者的 OU。
4. 反白顯示右側的所有使用者，然後按一下 [匯入]。 您應該會看到指出成功完成作業的快顯視窗。 關閉匯入視窗。

   ![MFA Server 使用者匯入](./media/howto-mfaserver-deploy/import2.png)

### <a name="automated-synchronization-with-active-directory"></a>自動與 Active Directory 同步處理

1. 在 Azure MFA Server 的左側選取 [目錄整合]。
2. 瀏覽至 [同步處理] 索引標籤。
3. 在底部選擇 [新增]
4. 在出現的 [新增同步處理項目] 方塊中選擇 [網域]、[OU] **或**安全性群組、[設定]、[預設方法]，以及此同步處理工作的 [預設語言]，然後按一下 [新增]。
5. 核取標示為 [啟用與 Active Directory 同步處理] 的方塊並選擇介於一分鐘與 24 小時之間的 [同步處理間隔]。

## <a name="how-the-azure-multi-factor-authentication-server-handles-user-data"></a>Azure Multi-Factor Authentication Server 如何處理使用者資料？

當您使用 Multi-Factor Authentication (MFA) Server 內部部署時，使用者的資料會儲存在內部部署伺服器中。 雲端中不會儲存任何持續性的使用者資料。 當使用者執行雙步驟驗證時，MFA Server 會將資料傳送至 Azure MFA 雲端服務，以執行驗證。 當這些驗證要求傳送至雲端服務時，要求和記錄檔中會傳送下列欄位，供客戶的驗證/使用情況報告中使用。 某些欄位是選擇性，可以在 Multi-Factor Authentication Server 內啟用或停用。 從 MFA Server 到 MFA 雲端服務的通訊使用透過連接埠 443 輸出的 SSL/TLS。 這些欄位包括：

* 唯一識別碼 - 使用者名稱或內部的 MFA 伺服器識別碼
* 名字和姓氏 (選擇性)
* 電子郵件地址 (選擇性)
* 電話號碼 - 進行語音通話或簡訊驗證時
* 安全性權杖 - 執行行動應用程式驗證時
* 驗證模式
* 驗證結果
* MFA Server 名稱
* MFA Server IP
* 用戶端 IP – 如果有的話

除了上述欄位，驗證結果 (成功/拒絕) 和任何拒絕的原因也與驗證資料一起儲存，可透過驗證/使用情況報告取得。

## <a name="back-up-and-restore-azure-mfa-server"></a>備份和還原 Azure MFA Server

確定您有完善的備份是任何系統都必須採取的重要步驟。

若要備份 Azure MFA Server，請確定您有一份 **C:\Program Files\Multi-Factor Authentication Server\Data** 資料夾，其中包含 **PhoneFactor.pfdata** 檔案。 

倘若需要還原，請完成下列步驟：

1. 在新的伺服器上重新安裝 Azure MFA Server。
2. 啟動新的 Azure MFA Server。
3. 停止 **MultiFactorAuth** 服務。
4. 以備份的複本覆寫 **PhoneFactor.pfdata**。
5. 啟動 **MultiFactorAuth** 服務。

新的伺服器現已啟動並以原始備份組態和使用者資料進行執行。

## <a name="managing-the-tlsssl-protocols-and-cipher-suites"></a>管理的 TLS/SSL 通訊協定和加密套件

在您已安裝或升級至 MFA Server 8.x 版 或更高版本後，除非您的組織需要較舊的和較弱的加密套件，否則建議您將它停用或移除。 如需如何完成此工作的相關資訊，請參閱[管理 AD FS 的 SSL/TLS 通訊協定和加密套件](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/manage-ssl-protocols-in-ad-fs)一文

## <a name="next-steps"></a>後續步驟

- 針對使用者自助安裝及設定[使用者入口網站](howto-mfaserver-deploy-userportal.md)。
- 搭配 [Active Directory Federation Service](multi-factor-authentication-get-started-adfs.md)、[RADIUS 驗證](howto-mfaserver-dir-radius.md)或 [LDAP 驗證](howto-mfaserver-dir-ldap.md)來安裝和設定 Azure MFA Server。
- 安裝和設定[使用 RADIUS 的遠端桌面閘道和 Azure Multi-Factor Authentication Server](howto-mfaserver-nps-rdg.md)。
- [部署 Azure Multi-Factor Authentication Server 行動裝置應用程式 Web 服務](howto-mfaserver-deploy-mobileapp.md)。
- [使用 Azure Multi-Factor Authentication 與協力廠商 VPN 的進階案例](howto-mfaserver-nps-vpn.md)。
