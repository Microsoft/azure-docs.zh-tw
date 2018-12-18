---
title: Azure AD Connect 同步處理服務陰影屬性 | Microsoft Docs
description: 描述陰影屬性如何在 Azure AD Connect 同步處理服務中運作。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 62278fa031c28fb32f882e438df0194c4db04efd
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310557"
---
# <a name="azure-ad-connect-sync-service-shadow-attributes"></a>Azure AD Connect 同步處理服務陰影屬性
大部分屬性是以它們在內部部署 Active Directory 中的相同方式，在 Azure AD 中表示。 但是某些屬性有一些特殊處理，而且在 Azure AD 中的屬性值可能不同於 Azure AD Connect 同步處理。

## <a name="introducing-shadow-attributes"></a>簡介陰影屬性
某些屬性在 Azure AD 中有兩種表示法。 會儲存內部部署值和計算值。 這些額外的屬性稱為陰影屬性。 您會在其中看到此行為的兩個最常見屬性是 **userPrincipalName** 和 **proxyAddress**。 這些屬性中有值代表未驗證網域時，就會發生屬性值的變更。 但是，連接中的同步處理引擎會讀取陰影屬性中的值，所以從其角度來看，屬性已由 Azure AD 確認。

您無法使用 Azure 入口網站或 PowerShell 看到陰影屬性。 但是，了解概念可協助您針對特定案例進行疑難排解，其中屬性在內部部署和雲端中有不同的值。

若要進一步了解行為，請參閱來自 Fabrikam 的這個範例：  
![網域](./media/how-to-connect-syncservice-shadow-attributes/domains.png)  
它們在其內部部署 Active Directory 中有多個 UPN 尾碼，但是它們只驗證其中一個。

### <a name="userprincipalname"></a>userPrincipalName
使用者在未驗證網域中具有下列屬性值︰

| 屬性 | 值 |
| --- | --- |
| 內部部署 userPrincipalName | lee.sperry@fabrikam.com |
| Azure AD shadowUserPrincipalName | lee.sperry@fabrikam.com |
| Azure AD userPrincipalName | lee.sperry@fabrikam.onmicrosoft.com |

userPrincipalName 屬性是您使用 PowerShell 時看到的值。

因為實際內部部署屬性值會儲存在 Azure AD 中，當您確認 fabrikam.com 網域時，Azure AD 會以 shadowUserPrincipalName 的值更新 userPrincipalName 屬性。 您不需要同步處理 Azure AD Connect 的任何變更，讓這些值進行更新。

### <a name="proxyaddresses"></a>proxyAddresses
僅包含已驗證網域的相同程序也會針對 proxyAddresses 發生，但是具有某些額外邏輯。 針對已驗證網域的檢查只會對信箱使用者發生。 擁有郵件功能的使用者或連絡人代表另一個 Exchange 組織中的使用者，您可以將 proxyAddresses 中的任何值新增至這些物件。

對於信箱使用者，內部部署或 Exchange Online，只會顯示已驗證網域的值。 它看起來如下所示：

| 屬性 | 值 |
| --- | --- |
| 內部部署 proxyAddresses | SMTP:abbie.spencer@fabrikamonline.com</br>smtp:abbie.spencer@fabrikam.com</br>smtp:abbie@fabrikamonline.com |
| Exchange Online proxyAddresses | SMTP:abbie.spencer@fabrikamonline.com</br>smtp:abbie@fabrikamonline.com</br>SIP:abbie.spencer@fabrikamonline.com |

在此情況下，**smtp:abbie.spencer@fabrikam.com**  已移除，因為該網域尚未驗證。 但是 Exchange 也會新增 **SIP:abbie.spencer@fabrikamonline.com** 。 Fabrikam 不曾使用 Lync/Skype 內部部署，但是 Azure AD 和 Exchange Online 為它進行準備。

proxyAddresses 的此邏輯稱為 **ProxyCalc**。 ProxyCalc 在使用者每次變更時叫用，其時機為︰

- 使用者已指派服務方案，該方案包含 Exchange Online，即使使用者沒有使用 Exchange 的授權。 例如，如果使用者已指派 Office E3 SKU，但是只指派 SharePoint Online。 即使您的信箱仍然是內部部署，也是如此。
- 屬性 msExchRecipientTypeDetails 具有值。
- 您對 proxyAddresses 或 userPrincipalName 進行變更。

ProxyCalc 可能需要一些時間來處理使用者的變更，並且與 Azure AD Connect 匯出程序不是同步的。

> [!NOTE]
> ProxyCalc 邏輯具有本主題中未提及之進階案例的一些額外行為。 本主題是讓您了解行為，而未記載所有內部邏輯。

### <a name="quarantined-attribute-values"></a>隔離的屬性值
有重複的屬性值時，也會使用陰影屬性。 如需詳細資訊，請參閱[重複屬性恢復功能](how-to-connect-syncservice-duplicate-attribute-resiliency.md)。

## <a name="see-also"></a>另請參閱
* [Azure AD Connect 同步處理](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
