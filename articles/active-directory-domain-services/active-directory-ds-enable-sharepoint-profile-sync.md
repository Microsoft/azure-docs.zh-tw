---
title: Azure Active Directory Domain Services：啟用對 SharePoint User Profile Service 的支援 | Microsoft Docs
description: 設定 Azure Active Directory Domain Services 受控網域以支援 SharePoint Server 的設定檔同步處理
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/22/2018
ms.author: maheshu
ms.openlocfilehash: c25fca2f3645a0397a999cec7552de15f20fb6be
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39502070"
---
# <a name="configure-a-managed-domain-to-support-profile-synchronization-for-sharepoint-server"></a>設定受控網域以支援 SharePoint Server 的設定檔同步處理
SharePoint Server 包含用來同步處理使用者設定檔的 User Profile Service。 若要設定 User Profile Service，必須在 Active Directory 網域授與適當的權限。 如需詳細資訊，請參閱[授與 Active Directory Domain Services 權限以供 SharePoint Server 2013 中的設定檔同步處理使用](https://technet.microsoft.com/library/hh296982.aspx)。

本文說明如何設定 Azure AD Domain Services 受控網域，以部署「SharePoint Server 使用者設定檔同步處理」服務。

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="the-aad-dc-service-accounts-group"></a>'AAD DC Service Accounts' 群組
受控網域上的 'Users' 組織單位內有一個名為 '**AAD DC Service Accounts**' 的安全性群組。 您可以在 [Active Directory 使用者和電腦] MMC 嵌入式管理單元中的受控網域上，看到這個群組。

![AAD DC Service Accounts 安全性群組](./media/active-directory-domain-services-admin-guide/aad-dc-service-accounts.png)

下列權限會委派給此安全性群組的成員：
- 受控網域之根 DSE 上的「複寫目錄變更」權限。
- 受控網域之組態命名內容 (cn=組態容器) 上的「複寫目錄變更」權限。

此安全性群組也是內建群組 **Pre-Windows 2000 Compatible Access** 的成員。

![AAD DC Service Accounts 安全性群組](./media/active-directory-domain-services-admin-guide/aad-dc-service-accounts-properties.png)


## <a name="enable-your-managed-domain-to-support-sharepoint-server-user-profile-sync"></a>可讓您的受控網域支援 SharePoint Server 使用者設定檔同步處理
您可以將用於 SharePoint 使用者設定檔同步處理的服務帳戶新增到 **AAD DC Service Accounts** 群組。 如此一來，同步處理帳戶就會獲得可將變更複寫到目錄的適當權限。 此組態步驟可讓 SharePoint Server 使用者設定檔同步處理正確運作。

![AAD DC Service Accounts - 新增成員](./media/active-directory-domain-services-admin-guide/aad-dc-service-accounts-add-member.png)

![AAD DC Service Accounts - 新增成員](./media/active-directory-domain-services-admin-guide/aad-dc-service-accounts-add-member2.png)

## <a name="related-content"></a>相關內容
* [技術參考 - 授與 Active Directory Domain Services 權限以供 SharePoint Server 2013 中的設定檔同步處理使用](https://technet.microsoft.com/library/hh296982.aspx)
