---
title: 在 Azure 入口網站中移轉要求多重要素驗證的傳統原則 | Microsoft Docs
description: 本文示範在 Azure 入口網站中移轉要求多重要素驗證的傳統原則。
services: active-directory
keywords: 應用程式的條件式存取, Azure AD 條件式存取, 安全存取公司資源, 條件式存取原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 8c1d978f-e80b-420e-853a-8bbddc4bcdad
ms.service: active-directory
ms.component: conditional-access
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/13/2018
ms.author: markvi
ms.reviewer: nigu
ms.openlocfilehash: 20a15bf94826df0b058be59ff242c46878ea26dd
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39630070"
---
# <a name="migrate-a-classic-policy-that-requires-multi-factor-authentication-in-the-azure-portal"></a>在 Azure 入口網站中移轉要求多重要素驗證的傳統原則 

本文示範如何移轉要求雲端應用程式進行**多重要素驗證**的傳統原則。 雖然這不是必要條件，但我們建議您先閱讀[在 Azure 入口網站中移轉傳統原則](policy-migration.md)，再開始移轉傳統原則。


 
## <a name="overview"></a>概觀 

本文中的案例示範如何移轉要求雲端應用程式進行**多重要素驗證**的傳統原則。 

![Azure Active Directory](./media/policy-migration/33.png)


移轉程序包含下列步驟：

1. [開啟傳統原則](#open-a-classic-policy)以取得組態設定。
2. 建立新的 Azure AD 條件式存取原則，以取代您的傳統原則。 
3. 停用傳統原則。



## <a name="open-a-classic-policy"></a>開啟傳統原則

1. 在 [Azure 入口網站](https://portal.azure.com)的左側導覽列上，按一下 [Azure Active Directory]。

    ![Azure Active Directory](./media/policy-migration-mfa/01.png)

2. 在 [Azure Active Directory] 頁面的 [管理] 區段中，按一下 [條件式存取]。

    ![條件式存取](./media/policy-migration-mfa/02.png)

3. 在 [管理] 區段中，按一下 [傳統原則 (預覽)]。

    ![傳統原則](./media/policy-migration-mfa/12.png)

4. 在傳統原則清單中，按一下要求雲端應用程式進行**多重要素驗證**的原則。

    ![傳統原則](./media/policy-migration-mfa/13.png)


## <a name="create-a-new-conditional-access-policy"></a>建立新的條件式存取原則


1. 在 [Azure 入口網站](https://portal.azure.com)的左側導覽列上，按一下 [Azure Active Directory]。

    ![Azure Active Directory](./media/policy-migration/01.png)

2. 在 [Azure Active Directory] 頁面的 [管理] 區段中，按一下 [條件式存取]。

    ![條件式存取](./media/policy-migration/02.png)



3. 若要在 [條件式存取] 頁面上開啟 [新增] 頁面，請在頂端的工具列中按一下 [新增]。

    ![條件式存取](./media/policy-migration/03.png)

4. 在 [新增] 頁面的 [名稱] 文字方塊中，鍵入您的原則名稱。

    ![條件式存取](./media/policy-migration/29.png)

5. 在 [指派] 區段中，按一下 [使用者和群組]。

    ![條件式存取](./media/policy-migration/05.png)

    a. 如已選取傳統原則中的所有使用者，請按一下 [所有使用者]。 

    ![條件式存取](./media/policy-migration/35.png)

    b. 如已選取傳統原則中的群組，請按一下 [選取使用者和群組]，然後選取所需的使用者和群組。

    ![條件式存取](./media/policy-migration/36.png)

    c. 如已排除群組，請按一下 [排除] 索引標籤，然後選取所需的使用者和群組。 

    ![條件式存取](./media/policy-migration/37.png)

6. 在 [新增] 頁面上，若要開啟 [雲端應用程式] 頁面，請在 [指派] 區段中，按一下 [雲端應用程式]。

8. 在 [雲端應用程式] 頁面上，執行下列步驟︰

    ![條件式存取](./media/policy-migration/08.png)

    a. 按一下 [選取應用程式]。

    b. 按一下 [選取] 。

    c. 在 [選取] 頁面上，選取您的雲端應用程式，然後按一下 [選取]。

    d. 在 [雲端應用程式] 頁面上，按一下 [完成]。



9. 如已選取 [需要多重要素驗證]：

    ![條件式存取](./media/policy-migration/26.png)

    a. 在 [存取控制] 區段中，按一下 [授與]。

    ![條件式存取](./media/policy-migration/27.png)

    b. 在 [授與] 頁面上，按一下 [授與存取權]，然後按一下 [需要多重要素驗證]。

    c. 按一下 [選取] 。


10. 按一下 [開啟] 以啟用您的原則。

    ![條件式存取](./media/policy-migration/30.png)



## <a name="disable-the-classic-policy"></a>停用傳統原則

若要停用傳統原則，請在 [詳細資料] 檢視中按一下 [停用]。

![傳統原則](./media/policy-migration-mfa/14.png)



## <a name="next-steps"></a>後續步驟

- 如需傳統原則移轉的詳細資訊，請參閱[在 Azure 入口網站中移轉傳統原則](policy-migration.md)。


- 如果您想要知道如何設定條件式存取原則，請參閱[利用 Azure Active Directory 條件式存取來取得特定應用程式的 MFA](app-based-mfa.md)。

- 如果您已準備好設定您環境的條件式存取原則，請參閱 [Azure Active Directory 中條件式存取的最佳做法](best-practices.md)。 
