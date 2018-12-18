---
title: 針對 Azure AD Domain Services 中的安全 LDAP (LDAPS) 進行疑難排解 | Microsoft Docs
description: 針對 Azure AD Domain Services 受控網域的安全 LDAP (LDAPS) 進行疑難排解
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 445c60da-e115-447b-841d-96739975bdf6
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/01/2018
ms.author: maheshu
ms.openlocfilehash: 9713a06bbf6a61b316e061cb851721a3554cd632
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39503667"
---
# <a name="troubleshoot-secure-ldap-ldaps-for-an-azure-ad-domain-services-managed-domain"></a>針對 Azure AD Domain Services 受控網域的安全 LDAP (LDAPS) 進行疑難排解

## <a name="connection-issues"></a>連線問題
如果您無法使用安全 LDAP 連線到受控網域：

* 安全 LDAP 憑證的簽發者鏈結在用戶端上必須受信任。 您可以選擇將根憑證授權單位新增到用戶端上受信任的根憑證存放區，以建立信任。
* 確認 LDAP 用戶端 (例如 ldp.exe) 是使用 DNS 名稱來連線至安全 LDAP 端點，而不是使用 IP 位址。
* 檢查 LDAP 用戶端連線到的 DNS 名稱。 它必須解析至受控網域上安全 LDAP 的公用 IP 位址。
* 確認受控網域之安全 LDAP 憑證的「主體」和「主體別名」屬性中有 DNS 名稱。
* 虛擬網路的 NSG 設定必須允許從網際網路至連接埠 636 的流量。 只有當您已啟用透過網際網路的安全 LDAP 存取時，才適用此步驟。


## <a name="need-help"></a>需要協助嗎？
如果您仍然無法使用安全 LDAP 來連線至受控網域，請[連絡產品小組](active-directory-ds-contact-us.md)以取得協助。 請包含下列資訊來協助進一步診斷問題：
* ldp.exe 進行連線及失敗時的螢幕擷取畫面。
* 您的 Azure AD 租用戶識別碼，以及受控網域的 DNS 網域名稱。
* 您嘗試作為繫結身分的確切使用者名稱。


## <a name="related-content"></a>相關內容
* [Azure AD Domain Services - 入門指南](active-directory-ds-getting-started.md)
* [Administer an Azure AD Domain Services managed domain (管理 Azure AD 網域服務受控網域)](active-directory-ds-admin-guide-administer-domain.md)
* [LDAP 查詢基本概念](https://technet.microsoft.com/library/aa996205.aspx)
* [管理 Azure AD Domain Services 受控網域上的群組原則](active-directory-ds-admin-guide-administer-group-policy.md)
* [網路安全性群組](../virtual-network/security-overview.md)
* [建立網路安全性群組](../virtual-network/tutorial-filter-network-traffic.md)
