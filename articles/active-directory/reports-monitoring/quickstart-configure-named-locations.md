---
title: 在 Azure Active Directory 中設定具名位置 | Microsoft Docs
description: 了解如何設定具名位置。
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
ms.assetid: f56e042a-78d5-4ea3-be33-94004f2a0fc3
ms.service: active-directory
ms.workload: identity
ms.component: report-monitor
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 07/23/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: d99f077449529fb4a4a7f124fe1c0263d4113dee
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46363440"
---
# <a name="quickstart-configure-named-locations-in-azure-active-directory"></a>快速入門：在 Azure Active Directory 中設定具名位置

您可以使用具名位置，標記組織中受信任的 IP 位址範圍。 Azure AD 使用具名位置來：
- 偵測[風險事件](concept-risk-events.md)中的誤判。 從受信任位置登入可降低使用者的登入風險。   
- 設定[位置型條件式存取](../conditional-access/location-condition.md)。

在此快速入門中，您將會學到如何在您的環境中設定具名位置。

## <a name="prerequisites"></a>必要條件

若要完成此快速入門，您需要：

* Azure AD 租用戶。 註冊[免費試用](https://azure.microsoft.com/trial/get-started-active-directory/)。 
* 一個使用者，他必須是租用戶端全域系統管理員。
* 在您的組織中建立且可信任的 IP 範圍。 IP 範圍的格式必須為「無類別網域間路由選擇」(CIDR)。

## <a name="configure-named-locations"></a>設定具名位置

1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 在左窗格中，選取 [Azure Active Directory]，然後從 [安全性] 區段選取 [條件式存取]。

    ![[條件式存取] 索引標籤](./media/quickstart-configure-named-locations/entrypoint.png)

3. 在 [條件式存取] 頁面上，選取 [具名位置] 並選取 [新增位置]。

    ![具名位置](./media/quickstart-configure-named-locations/namedlocation.png)

6. 填寫新頁面上的表單。 

    * 在 [名稱] 方塊中，輸入具名位置的名稱。
    * 在 [IP 位址] 方塊中，輸入 CIDR 格式的 IP 範圍。  
    * 按一下頁面底部的 [新增]。
    
    ![[新增] 刀鋒視窗](./media/quickstart-configure-named-locations/61.png)

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱：

- [Azure Active Directory 中的條件式存取](../active-directory-conditional-access-azure-portal.md)。

- [Azure AD 條件式存取中的位置條件](../conditional-access/location-condition.md)

- [Azure AD 入口網站中有風險的登入報告](concept-risky-sign-ins.md)。  
