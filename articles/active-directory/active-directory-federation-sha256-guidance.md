---
title: 變更 Office 365 信賴憑證者信任的簽章雜湊演算法 | Microsoft Docs
description: 本頁面提供變更與 Office 365 搭配運作之同盟信任的 SHA 演算法的指導方針
keywords: SHA1,SHA256,O365,同盟,aadconnect,adfs,ad fs,變更 sha,同盟信任,信賴憑證者信任
services: active-directory
documentationcenter: ''
author: anandyadavmsft
manager: mtillman
editor: ''
ms.assetid: cf6880e2-af78-4cc9-91bc-b64de4428bbd
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/31/2016
ms.author: anandy
ms.openlocfilehash: 5352c76d2f41adef8fae29d3f60204fb05546ddc
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46298042"
---
# <a name="change-signature-hash-algorithm-for-office-365-relying-party-trust"></a>變更 Office 365 信賴憑證者信任的簽章雜湊演算法
## <a name="overview"></a>概觀
Active Directory 同盟服務 (AD FS) 會將其權杖簽署到 Microsoft Azure Active Directory 以確保它們無法被竄改。 此簽章可以是以 SHA1 或 SHA256 為基礎。 Azure Active Directory 現在支援以 SHA256 演算法簽署的權杖，建議您將權杖簽署演算法設定為 SHA256 以獲得最高層級的安全性。 本文說明將權杖簽署演算法設定為更安全 SHA256 層級所需的步驟。

>[!NOTE]
>Microsoft 建議使用 SHA256 作為簽署權杖的演算法，因為它比 SHA1 安全，但 SHA1 仍保留為支援的選項。

## <a name="change-the-token-signing-algorithm"></a>變更權杖簽署演算法
使用下列兩個程序之一設定簽章演算法之後，AD FS 會使用 SHA256 簽署 Office 365 信賴憑證者信任的權杖。 您不需要另外進行任何組態變更，而且這項變更不會影響您對 Office 365 或其他 Azure AD 應用程式的存取能力。

### <a name="ad-fs-management-console"></a>AD FS 管理主控台
1. 在主要 AD FS 伺服器上，開啟 [AD FS 管理主控台]。
2. 展開 AD FS 節點，然後按一下 [信賴憑證者信任] 。
3. 在您的 Office 365/Azure 信賴憑證者信任上按一下滑鼠右鍵，然後選取 [屬性] 。
4. 選取 [進階]  索引標籤，然後選取安全雜湊演算法 SHA256。
5. 按一下 [確定]。

![SHA256 簽署演算法--MMC](./media/active-directory-aadconnectfed-sha256guidance/mmc.png)

### <a name="ad-fs-powershell-cmdlets"></a>AD FS PowerShell Cmdlet
1. 在任何 AD FS 伺服器上，以系統管理員權限開啟 PowerShell。
2. 使用 **Set-AdfsRelyingPartyTrust** Cmdlet 來設定安全雜湊演算法。
   
   <code>Set-AdfsRelyingPartyTrust -TargetName 'Microsoft Office 365 Identity Platform' -SignatureAlgorithm 'http://www.w3.org/2001/04/xmldsig-more#rsa-sha256'</code>

## <a name="also-read"></a>另請參閱
* [使用 Azure AD Connect 修復 Office 365 信任](hybrid/how-to-connect-fed-management.md#repairthetrust)

