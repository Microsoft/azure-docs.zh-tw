---
title: Azure Active Directory Identity Protection 通知| Microsoft Docs
description: 了解通知如何支援您的調查活動。
services: active-directory
keywords: azure active directory identity protection, cloud app discovery, 管理應用程式, 安全性, 風險, 風險層級, 弱點, 安全性原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 65ca79b9-4da1-4d5b-bebd-eda776cc32c7
ms.service: active-directory
ms.component: conditional-access
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/07/2017
ms.author: markvi
ms.reviewer: nigu
ms.openlocfilehash: 0a546acd05246e011fa66abea8a667d0b3513588
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40005893"
---
# <a name="azure-active-directory-identity-protection-notifications"></a>Azure Active Directory Identity Protection 通知

Azure AD Identity Protection 會傳送兩種自動化通知電子郵件，協助您管理使用者風險和風險事件：

- 「偵測到具有風險的使用者」電子郵件
- 每週精選文章電子郵件

本文同時提供您這兩種通知電子郵件的概觀。


## <a name="users-at-risk-detected-email"></a>「偵測到具有風險的使用者」電子郵件

當偵測到具有風險的帳戶時，Azure AD Identity Protection 會產生主旨為**偵測到具有風險的使用者**的電子郵件警示來作為回應。 此電子郵件會包含**[標幟為有風險的使用者](../reports-monitoring/concept-user-at-risk.md)** 報告的連結。 最佳做法是應立即調查具有風險的使用者。

![「偵測到具有風險的使用者」電子郵件](./media/notifications/01.png)


### <a name="configuration"></a>組態

如果您是系統管理員，您可以設定：

- **觸發產生此電子郵件的風險層級** - 此風險層級預設為「高」風險。
- **此電子郵件的收件者** - 收件者預設包含所有「全域管理員」。 「全域管理員」也可以新增其他「全域管理員」、「安全性管理員」、「安全性讀取者」作為收件者。  


若要開啟相關的對話方塊，請在 [Identity Protection] 頁面的 [設定] 區段中，按一下 [警示]。

![「偵測到具有風險的使用者」電子郵件](./media/notifications/05.png)


## <a name="weekly-digest-email"></a>每週精選文章電子郵件

每週精選文章電子郵件包含新風險事件的摘要。  
其中包括：

- 有風險的使用者

- 可疑的活動

- 偵測到的弱點

- Identity Protection 中相關報告的連結

    ![補救](./media/notifications/400.png "補救")

### <a name="configuration"></a>組態

如果您是系統管理員，您可以關閉每週摘要電子郵件的傳送。

![使用者風險](./media/notifications/62.png "使用者風險")

若要開啟相關的對話方塊，請在 [Identity Protection] 頁面的 [設定] 區段中，按一下 [每週摘要]。

![「偵測到具有風險的使用者」電子郵件](./media/notifications/04.png)


## <a name="see-also"></a>另請參閱

- [Azure Active Directory Identity Protection](../active-directory-identityprotection.md)
