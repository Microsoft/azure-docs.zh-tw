---
title: Azure AD Connect 同步處理︰啟用 AD 資源回收筒 | Microsoft Docs
description: 本主題建議對 Azure AD Connect 使用 AD 資源回收筒功能。
services: active-directory
keywords: AD 資源回收筒, 意外刪除, 來源錨點
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: afec4207-74f7-4cdd-b13a-574af5223a90
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 378e1d3aab992e9b4e6f2263c26ea4268a43d678
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306157"
---
# <a name="azure-ad-connect-sync-enable-ad-recycle-bin"></a>Azure AD Connect 同步處理︰啟用 AD 資源回收筒
建議您為會同步處理至 Azure AD 的內部部署 Active Directory 啟用 AD 資源回收筒功能。 

如果您不小心刪除內部部署 AD 使用者物件，並使用此功能還原該物件，Azure AD 會還原對應的 Azure AD 使用者物件。  如需 AD 資源回收筒功能的相關資訊，請參閱[還原已刪除之 Active Directory 物件的案例概觀](https://technet.microsoft.com/library/dd379542.aspx)一文。

## <a name="benefits-of-enabling-the-ad-recycle-bin"></a>啟用 AD 資源回收筒的好處
這項功能可協助還原 Azure AD 使用者物件，方法是執行下列動作︰

* 如果您不小心刪除內部部署 AD 使用者物件，系統會在下一個同步處理循環刪除對應的 Azure AD 使用者物件。 根據預設，Azure AD 會讓已刪除的 Azure AD 使用者物件保持 30 天的虛刪除狀態。

* 如果您已啟用內部部署 AD 資源回收筒功能，您可以還原已刪除的內部部署 AD 使用者物件，而不必變更其來源錨點值。 當復原後的內部部署 AD 使用者物件同步處理至 Azure AD 時，Azure AD 會還原對應的虛刪除 Azure AD 使用者物件。 如需來源錨點屬性的相關資訊，請參閱[Azure AD Connect：設計概念](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-design-concepts#sourceanchor)一文。

* 如果您未啟用內部部署 AD 資源回收筒功能，您可能需要建立 AD 使用者物件來取代已刪除的物件。 如果您將 Azure AD Connect 同步處理服務設定為使用系統產生的 AD 屬性 (例如 ObjectGuid) 作為來源錨點屬性，新建立的 AD 使用者物件所具有的來源錨點值不會和已刪除之 AD 使用者物件的值相同。 當新建立的 AD 使用者物件同步處理至 Azure AD 時，Azure AD 會建立新的 Azure AD 使用者物件，而不是還原虛刪除的 Azure AD 使用者物件。

> [!NOTE]
> 根據預設，Azure AD 會讓已刪除的 Azure AD 使用者物件保持 30 天的虛刪除狀態，再將其永久刪除。 不過，系統管理員也可以提早刪除這類物件。 物件在永久刪除後就無法再復原，即使您已啟用內部部署 AD 資源回收筒功能也是一樣。



## <a name="next-steps"></a>後續步驟
**概觀主題**

* [Azure AD Connect 同步處理：了解及自訂同步處理](how-to-connect-sync-whatis.md)

* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)
