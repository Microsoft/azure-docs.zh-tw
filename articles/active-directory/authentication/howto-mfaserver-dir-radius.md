---
title: RADIUS 驗證和 Azure MFA Server | Microsoft Docs
description: 部署 RADIUS 驗證與 Azure Multi-Factor Authentication Server。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 651035430695c0c5082e443dabd998a196e0eefa
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39158296"
---
# <a name="integrate-radius-authentication-with-azure-multi-factor-authentication-server"></a>將 RADIUS 驗證與 Azure Multi-Factor Authentication Server 整合

RADIUS 是接受驗證要求並處理這些要求的標準通訊協定。 Azure Multi-Factor Authentication Server 可作為 RADIUS 伺服器。 將它插入您的 RADIUS 用戶端 (VPN 應用裝置) 與驗證目標之間，以新增雙步驟驗證。 您的驗證目標可能是 Active Directory、LDAP 目錄或其他 RADIUS 伺服器。 為了讓 Azure Multi-Factor Authentication (MFA) 運作，必須將 Azure MFA Server 設定為能夠與用戶端伺服器和驗證目標進行通訊。 Azure MFA Server 會從 RADIUS 用戶端接收要求、向驗證目標驗證認證、新增 Azure Multi-Factor Authentication，然後將回應傳回給 RADIUS 用戶端。 只有當主要驗證和 Azure Multi-Factor Authentication 都成功時，驗證要求才會成功。

> [!NOTE]
> MFA 伺服器在做為 RADIUS 伺服器時，僅支援 PAP (密碼驗證通訊協定) 和 MSCHAPv2 (Microsoft 的 Challenge-Handshake 驗證通訊協定) RADIUS 通訊協定。  當 MFA Server 是作為另一部 RADIUS 伺服器的 RADIUS Proxy，而該伺服器支援 EAP (可延伸的驗證通訊協定) 這類其他通訊協定時，則也可以使用該通訊協定。
>
> 在此組態中，單向 SMS 和 OATH 權杖沒有作用，因為 MFA Server 無法使用替代通訊協定來起始成功的 RADIUS 挑戰回應。

![Radius 驗證](./media/howto-mfaserver-dir-radius/radius.png)

## <a name="add-a-radius-client"></a>新增 RADIUS 用戶端
若要設定 RADIUS 驗證，請在 Windows 伺服器上安裝 Azure Multi-Factor Authentication Server。 如果您有 Active Directory 環境，此伺服器應該加入網路內的網域。 使用下列程序來設定 Azure Multi-Factor Authentication Server：

1. 在 Azure Multi-Factor Authentication Server 中，按一下左功能表中的 [RADIUS 驗證] 圖示。
2. 選取 [啟用 RADIUS 驗證] 核取方塊。
3. 如果 Azure MFA RADIUS 服務需要在非標準連接埠上接聽 RADIUS 要求，請在 [用戶端] 索引標籤上變更 [驗證連接埠] 和 [帳戶處理連接埠]。
4. 按一下 [新增] 。
5. 輸入將向 Azure Multi-Factor Authentication Server 進行驗證之應用裝置/伺服器的 IP 位址、應用程式名稱 (選擇性)，以及 共用密碼。

  應用程式名稱會出現在報告中，而且可能顯示在簡訊或行動裝置應用程式驗證訊息內。

  共用密碼在 Azure Multi-Factor Authentication Server 和應用裝置/伺服器上必須相同。

6. 如果所有使用者都已經匯入伺服器中，且必須接受多重要素驗證，請選取 [需要進行 Multi-Factor Authentication 使用者比對] 方塊。 如果有大量使用者尚未匯入伺服器或將免除雙步驟驗證，請勿核取此方塊。
7. 如果您想要使用來自行動驗證應用程式的 OATH 密碼作為備份方法，請核取 [啟用遞補 OATH 權杖] 方塊。
8. 按一下 [確定]。

請重複步驟 4 到 8 以視需要新增多個其他 RADIUS 用戶端。

## <a name="configure-your-radius-client"></a>設定 RADIUS 用戶端

1. 按一下 [目標] 索引標籤。
2. 如果 Azure MFA Server 是安裝在 Active Directory 環境中已加入網域的伺服器上，請選取 [Windows 網域]。
3. 如果應該向 LDAP 目錄驗證使用者，請選取 [LDAP 繫結]。

  選取 [目錄整合] 圖示，然後編輯 [設定] 索引標籤上的 LDAP 組態，以便讓伺服器能夠繫結至您的目錄。 如需有關設定 LDAP 的指示，請參閱 [LDAP Proxy 組態指南](howto-mfaserver-dir-ldap.md)。

4. 如果應該向另一部 RADIUS 伺服器驗證使用者，選取 [RADIUS 伺服器]。
5. 按一下 [新增] 以設定 Azure MFA Server 要作為其 Proxy 來處理 RADIUS 要求的伺服器。
6. 在 [新增 RADIUS 伺服器] 對話方塊中，輸入 RADIUS 伺服器的 IP 位址和共用密碼。

  共用密碼在 Azure Multi-Factor Authentication Server 和 RADIUS 伺服器上必須相同。 如果 RADIUS 伺服器使用不同的通訊埠，請變更 [驗證連接埠] 和 [帳戶處理連接埠]。

7. 按一下 [確定]。
8. 在另一部 RADIUS 伺服器中新增 Azure MFA Server 作為 RADIUS 用戶端，如此它才能夠處理從 Azure MFA Server 傳送給它的存取要求。 請使用 Azure Multi-Factor Authentication Server 中設定的相同共用密碼。

重複這些步驟來新增更多 RADIUS 伺服器。 使用 [上移] 和 [下移] 按鈕來設定 Azure MFA Server 應呼叫它們的順序。

您已成功設定 Azure Multi-Factor Authentication Server。 「伺服器」正在設定的連接埠上接聽來自設定的用戶端的 RADIUS 存取要求。   

## <a name="radius-client-configuration"></a>RADIUS 用戶端組態
若要設定 RADIUS 用戶端，請遵循下列指導方針：

* 將您的應用裝置/伺服器設定為透過 RADIUS 向 Azure Multi-Factor Authentication Server (作為 RADIUS 伺服器) 的 IP 位址驗證。
* 使用先前設定的相同共用密碼。
* 將 RADIUS 逾時設定為 30 至 60 秒，以保留足夠的時間來驗證使用者的認證、執行兩步驟驗證、接收其回應，然後回應 RADIUS 存取要求。

## <a name="next-steps"></a>後續步驟

如果您在雲端有 Azure Multi-Factor Authentication，請了解如何[與 RADIUS 驗證整合](howto-mfa-nps-extension.md)。 