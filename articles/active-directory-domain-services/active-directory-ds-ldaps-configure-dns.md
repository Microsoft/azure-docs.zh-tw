---
title: 將 DNS 設定為使用 LDAPS 透過網際網路存取受控網域 | Microsoft Docs
description: 將 DNS 設定為使用 LDAPS 透過網際網路存取 Azure AD Domain Services 受控網域
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: a47f0f3e-2578-422a-a421-034f66de38f5
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/01/2018
ms.author: maheshu
ms.openlocfilehash: 669e0392cb77434c372c9af3c4d467d19cff8abd
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39501729"
---
# <a name="configure-dns-to-access-an-azure-ad-domain-services-managed-domain-using-secure-ldap-ldaps"></a>將 DNS 設定為使用安全的 LDAP (LDAPS) 存取 Azure AD Domain Services 受控網域

## <a name="before-you-begin"></a>開始之前
完成[工作 3：使用 Azure 入口網站為受控網域啟用安全 LDAP](active-directory-ds-admin-guide-configure-secure-ldap-enable-ldaps.md)

## <a name="task-4-configure-dns-to-access-the-managed-domain-from-the-internet"></a>工作 4：設定 DNS 以從網際網路存取受控網域
> [!TIP]
> **選擇性工作** - 如果您不打算使用 LDAPS 來透過網際網路存取受控網域，請略過這項設定工作。
>
>

開始這項工作之前，請先確定您已完成[工作 3](active-directory-ds-admin-guide-configure-secure-ldap-enable-ldaps.md)中所述的步驟。

啟用透過網際網路的安全 LDAP 存取後，您需要更新 DNS 以便用戶端電腦可以找到此受控網域。 您可在 [屬性] 索引標籤上 [可透過 LDAPS 存取的外部 IP 位址] 中看到外部 IP 位址。

請設定外部 DNS 提供者，讓受控網域的 DNS 名稱 (例如 'ldaps.contoso100.com') 指向這個外部 IP 位址。 例如，建立下列 DNS 項目︰

    ldaps.contoso100.com  -> 52.165.38.113

就這麼簡單！ 您現在已準備好可使用安全 LDAP 透過網際網路連線到受控網域。

> [!WARNING]
> 請記住，用戶端電腦必須信任 LDAPS 憑證的簽發者，才能成功使用 LDAPS 連線到受控網域。 如果您使用公開的受信任憑證授權單位，您不需要採取任何動作，因為用戶端電腦會信任這些憑證簽發者。 如果您使用自我簽署憑證，在用戶端電腦的受信任憑證存放區中安裝自我簽署憑證的公開部分。
>
>

## <a name="next-step"></a>後續步驟
[工作 5：繫結至受控網域，並鎖定安全 LDAP 存取](active-directory-ds-ldaps-bind-lockdown.md)
