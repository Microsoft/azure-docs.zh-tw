---
title: Azure AD Connect 同步處理：了解使用者、群組和連絡人 | Microsoft Docs
description: 說明 Azure AD Connect 同步處理中的使用者、群組和連絡人。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 8d204647-213a-4519-bd62-49563c421602
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/15/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 46d0ce28d6381662847917ce83c77780f1bd9e4c
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310577"
---
# <a name="azure-ad-connect-sync-understanding-users-groups-and-contacts"></a>Azure AD Connect 同步處理：了解使用者、群組和連絡人
您可能有幾種不同的原因，而擁有多個 Active Directory 樹系並且具有幾種不同的部署拓撲。 常見的模型包括合併與收購之後的帳戶-資源部署與 GAL 同步處理的樹系。 雖然有單純的模型，但混合模型也同樣常見。 Azure AD Connect 同步處理中的預設組態不會採用任何特定的模型，但是根據在安裝指南中選取使用者比對的方式，可以觀察到不同的行為。

在本主題中，我們將詳細解說預設組態在某些拓撲中的運作方式。 我們將詳細解說組態，以及可用來查看組態的同步處理規則編輯器。

組態假設幾個一般規則：
* 無論我們以何種順序從來源 Active Directory 匯入，最後的結果應一律相同。
* 使用中的帳戶一律會提供登入資訊，包括 **userPrincipalName** 和 **sourceAnchor**。
* 如果找不到使用中的帳戶，除非帳戶是連結的信箱，否則停用的帳戶將會提供 userPrincipalName 和 sourceAnchor。
* 具有連結信箱的帳戶永遠不會用於 userPrincipalName 和 sourceAnchor。 它假設稍後就會找到使用中的帳戶。
* 連絡人物件可能會佈建到 Azure AD 成為連絡人或使用者。 一直要到處理了所有來源 Active Directory 樹系之後，您才會知道物件成為何者。

## <a name="groups"></a>群組
將 Active Directory 的群組同步至 Azure AD 時應該注意的重點：

* Azure AD Connect 會將內建安全性群組從目錄同步作業中排除。

* Azure AD Connect 不支援將[主要群組成員資格](https://technet.microsoft.com/library/cc771489(v=ws.11).aspx)同步至 Azure AD。

* Azure AD Connect 不支援將[動態通訊群組成員資格](https://technet.microsoft.com/library/bb123722(v=exchg.160).aspx)同步至 Azure AD。

* 若要將 Active Directory 群組同步至 Azure AD 做為擁有郵件功能的群組：

    * 如果群組的 *proxyAddress* 屬性是空的，其 *mail* 屬性必須有值

    * 如果群組的 *proxyAddress* 屬性不是空的，則必須包含至少一個 SMTP Proxy 位址值。 這裡有一些範例：
    
      * proxyAddress 屬性值為 *{"X500:/0=contoso.com/ou=users/cn=testgroup"}* 的 Active Directory 群組在 Azure AD 中不會擁有郵件功能。 該群組沒有 SMTP 位址。
      
      * proxyAddress 屬性值為 *{"X500:/0=contoso.com/ou=users/cn=testgroup","SMTP:johndoe@contoso.com"}* 的 Active Directory 群組在 Azure AD 中會擁有郵件功能。
      
      * proxyAddress 屬性值為 *{"X500:/0=contoso.com/ou=users/cn=testgroup", "smtp:johndoe@contoso.com"}* 的 Active Directory 群組在 Azure AD 中也會擁有郵件功能。

## <a name="contacts"></a>連絡人
在合併與收購時使用 GALSync 解決方案橋接兩個或多個 Exchange 樹系之後，常會有多個連絡人代表不同樹系中的某個使用者。 連絡人物件一律從連接器空間使用 mail 屬性加入 Metaverse。 如果已經有具相同郵件地址的連絡人物件或使用者物件，則物件會一起加入。 這設定在規則 **In from AD – Contact Join**中。 另外還有一個名為 **In from AD - Contact Common** 的規則，其屬性流程是使用常數 **Contact** 提供給 Metaverse 屬性 **sourceObjectType**。 此規則的優先順序非常低，因此，如果已將任何使用者物件聯結到同一個 Metaverse 物件，則規則 **In from AD – User Common** 會提供 User 值給這個屬性。 有了這項規則，如果沒有使用者加入，此屬性的值就會是 Contact，如果至少找到了一個使用者，則屬性的值就會是 User。

當佈建物件到 Azure AD 時，如果 Metaverse 屬性 **sourceObjectType** 設為 **Contact**，輸出規則 **Out to AAD - Contact Join** 就會建立連絡人物件。 如果將此屬性設定為 **User**，則規則 **Out to AAD - User Join** 將改為建立使用者物件。
當有多個來源 Active Directory 匯入並同步處理時，可以將物件從 Contact 升級為 User。

例如，在 GALSync 拓撲中，當我們匯入第一個樹系時，我們會在第二個樹系中找到每個人的連絡人物件。 這會在 AAD 連接器中暫存新的連絡人物件。 當我們之後匯入和同步處理第二個樹系時，我們會找到真正的使用者，並將他們加入現有的 Metaverse 物件。 然後我們就可以刪除 AAD 中的連絡人物件，然後改為建立新的使用者物件。

如果您有一個拓撲，其中使用者以連絡人代表，請確定您在安裝指南中選取根據 mail 屬性比對使用者。 如果您選取另一個選項，則您的組態就會與順序有關。 連絡人物件永遠會跟隨著 mail 屬性，但如果在安裝指南中選取了此選項，使用者物件就只會跟隨著 mail 屬性。 如果在匯入使用者物件之前先匯入連絡人物件，您在 Metaverse 中就會得到具有相同 mail 屬性的兩種不同物件。 在匯出到 Azure AD 期間，將會擲回錯誤。 此行為是刻意設計，這樣會指出資料有錯誤，或者在安裝期間未正確識別拓撲。

## <a name="disabled-accounts"></a>停用的帳戶
停用的帳戶也同樣會同步處理到 Azure AD。 停用的帳戶在 Exchange 中通常代表資源，例如會議室。 例外狀況是具有連結信箱的使用者。如先前所述，這些使用者將永遠不會佈建帳戶到 Azure AD。

這個假設是，如果找到停用的使用者帳戶，我們之後就不會找到另一個使用中的帳戶，而物件會使用找到的 userPrincipalName 和 sourceAnchor 佈建到 Azure AD。 如果有另一個使用中的帳戶加入相同的 Metaverse 物件，則會使用其 userPrincipalName 和 sourceAnchor。

## <a name="changing-sourceanchor"></a>變更 sourceAnchor
當物件匯出到 Azure AD 之後，則不允許再變更 sourceAnchor。 當物件匯出之後，Metaverse 屬性 **cloudSourceAnchor** 就會設為 Azure AD 所接受的 **sourceAnchor** 值。 如果 **sourceAnchor** 已變更且不符合 **cloudSourceAnchor**，規則 **Out to AAD - User Join** 將會擲回 **sourceAnchor 屬性已經變更**的錯誤。 在此情況下，必須先更正組態或資料，讓 Metaverse 中再度具有相同的 sourceAnchor，才能再次同步處理物件。

## <a name="additional-resources"></a>其他資源
* [Azure AD Connect 同步處理：自訂同步處理選項](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)

