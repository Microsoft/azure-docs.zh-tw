---
title: 使用 LDAP (LDAPS) 繫結到 Azure AD Domain Services 受控網域 | Microsoft Docs
description: 使用 LDAP (LDAPS) 繫結到 Azure AD Domain Services 受控網域
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 6871374a-0300-4275-9a45-a39a52c65ae4
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/01/2018
ms.author: maheshu
ms.openlocfilehash: 9728d42710ce44226363ea4954d83fcc3efbfb75
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39502947"
---
# <a name="bind-to-an-azure-ad-domain-services-managed-domain-using-secure-ldap-ldaps"></a>使用 LDAP (LDAPS) 繫結到 Azure AD Domain Services 受控網域

## <a name="before-you-begin"></a>開始之前
完成[工作 4 - 設定 DNS 以從網際網路存取受控網域](active-directory-ds-ldaps-configure-dns.md)。


## <a name="task-5-bind-to-the-managed-domain-over-ldap-using-ldpexe"></a>工作 5：使用 LDP.exe 透過 LDAP 繫結到受控網域
您可以使用「遠端伺服器管理工具」套件隨附的 LDP.exe 工具，來透過 LDAP 繫結和搜尋。

首先，開啟 LDP 並連線到受控網域。 按一下功能表中的 [連線]，然後按一下 [連線...]。 指定受控網域的 DNS 網域名稱。 指定連線用的連接埠。 若是 LDAP 連線，請使用連接埠 389。 針對 LDAPS 連線，請使用連接埠 636。 按一下 [確定] 按鈕以連線至受控網域。

接著繫結至受控網域。 按一下功能表中的 [連線]，然後按一下 [繫結...]。 提供屬於「AAD DC 系統管理員」群組之使用者帳戶的認證。

選取功能表中的 [檢視]，然後選取 [樹狀]。 「基準 DN」欄位保持空白，然後按一下 [確定]。 瀏覽至要搜尋的容器，以滑鼠右鍵按一下該容器，然後選取 [搜尋]。

> [!TIP]
> - 從 Azure AD 同步的使用者和群組會儲存在 **AADDC 使用者**容器中。 此容器的搜尋路徑看起來如同 ```CN=AADDC\ Users,DC=CONTOSO100,DC=COM```。
> - 已加入受控網域的電腦之電腦帳戶會儲存在 **AADDC 電腦**容器中。 此容器的搜尋路徑看起來如同 ```CN=AADDC\ Computers,DC=CONTOSO100,DC=COM```。
>
>

更多資訊 - [LDAP 查詢基本概念](https://technet.microsoft.com/library/aa996205.aspx)


## <a name="task-6-lock-down-secure-ldap-access-to-your-managed-domain-over-the-internet"></a>工作 6：限制透過網際網路存取受控網域時只能使用安全 LDAP 存取
> [!NOTE]
> 如果您尚未限制透過網際網路存取受控網域時只能使用安全 LDAP 存取，請略過此設定工作。
>
>

開始此工作之前，請先確定您已完成[工作 3](active-directory-ds-admin-guide-configure-secure-ldap-enable-ldaps.md) 中所述的步驟。

當您限制透過網際網路存取受控網域時只能使用 LDAPS 存取時，會產生安全性威脅。 受控網域可以從用於安全 LDAP 之連接埠的網際網路 (也就是連接埠 636) 存取。 您可以選擇將受控網域存取限制為特定已知 IP 位址。 建立網路安全性群組 (NSG)，並將它與您已啟用 Azure AD Domain Services 的子網路產生關聯。

以下表格中的範例 NSG 會鎖定透過網際網路的安全 LDAP 存取。 NSG 中的規則只允許來自一組指定之 IP 位址，透過 TCP 連接埠 636 的連入安全 LDAP 存取。 預設 'DenyAll' 規則適用於來自網際網路的所有其他輸入流量。 允許從指定的 IP 位址透過網際網路之 LDAPS 存取的 NSG 規則，其優先順序高於 DenyAll NSG 規則。

![透過網際網路之安全 LDAP 存取的範例 NSG](./media/active-directory-domain-services-admin-guide/secure-ldap-sample-nsg.png)

**詳細資訊** - [網路安全性群組](../virtual-network/security-overview.md)。


## <a name="related-content"></a>相關內容
* [Azure AD Domain Services - 入門指南](active-directory-ds-getting-started.md)
* [Administer an Azure AD Domain Services managed domain (管理 Azure AD 網域服務受控網域)](active-directory-ds-admin-guide-administer-domain.md)
* [LDAP 查詢基本概念](https://technet.microsoft.com/library/aa996205.aspx)
* [管理 Azure AD Domain Services 受控網域上的群組原則](active-directory-ds-admin-guide-administer-group-policy.md)
* [網路安全性群組](../virtual-network/security-overview.md)
* [建立網路安全性群組](../virtual-network/tutorial-filter-network-traffic.md)


## <a name="next-step"></a>後續步驟
[針對受控網域上的安全 LDAP 進行疑難排解](active-directory-ds-ldaps-troubleshoot.md)
