---
title: 在 Azure Active Directory B2C 中定義自訂屬性 | Microsoft Docs
description: 在 Azure Active Directory B2C 中定義您應用程式的自訂屬性，以收集您客戶的相關資訊。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 07/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: d5ef77ab0bbf00d4ddbb05b7a38516e3c3e7d800
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38968769"
---
# <a name="define-custom-attributes-in-azure-active-directory-b2c"></a>在 Azure Active Directory B2C 中定義自訂屬性

 每個客戶面向的應用程式對於需要收集的資訊都有不同的需求。 您的 Azure Active Directory (Azure AD) B2C 租用戶隨附一組儲存在屬性中的內建資訊，例如名字、姓氏、城市及郵遞區號。 Azure AD B2C 可讓您擴充儲存在每個客戶帳戶上的屬性組合。 
 
 您可以在 [Azure 入口網站](https://portal.azure.com/) 中建立自訂屬性，並在您的註冊原則、註冊或登入原則，或設定檔編輯原則中使用這些屬性。 您也可以使用 [Azure AD Graph API](active-directory-b2c-devquickstarts-graph-dotnet.md)讀取和寫入這些屬性。 Azure AD B2C 中的自訂屬性會使用 [Azure AD Graph API 目錄結構描述擴充](https://msdn.microsoft.com/library/azure/ad/graph/howto/azure-ad-graph-api-directory-schema-extensions)。

## <a name="create-a-custom-attribute"></a>建立自訂屬性

1. 以 Azure AD B2C 租用戶的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在 Azure 入口網站的右上角切換到您的 Azure AD B2C 租用戶，確定您使用的目錄包含該租用戶。 選取您的訂用帳戶資訊，然後選取 [切換目錄]。 

    ![切換為您的 Azure AD B2C 租用戶](./media/active-directory-b2c-reference-custom-attr/switch-directories.png)

    選擇包含您租用戶的目錄。

    ![選取目錄](./media/active-directory-b2c-reference-custom-attr/select-directory.png)

3. 選擇 Azure 入口網站左上角的 [所有服務]，搜尋並選取 [Azure AD B2C]。
4. 選取 [使用者屬性]，然後選取 [新增]。
5. 提供自訂屬性的**名稱** (例如 "ShoeSize")
6. 選擇 [資料類型]。 只有**字串**、**布林值**，以及 **Int** 可供使用。
7. 選擇性地輸入 [描述]，以供參考。 
8. 按一下頁面底部的 [新增] 。

[使用者屬性] 清單現已提供自訂屬性，您可用於原則中。 只有在第一次用於任何原則時才會建立自訂屬性，而不是在您將它加入 [使用者屬性] 清單時建立。

## <a name="use-a-custom-attribute-in-your-policy"></a>在您的原則中使用自訂屬性

1. 在 Azure AD B2C 租用戶中，選取 [註冊或登入原則]。
2. 選取您的原則 (例如「B2C_1_SignupSignin」) 以開啟之。 
3. 按一下 [編輯]。
4. 選取 [註冊屬性]，然後選取自訂屬性 (例如「ShoeSize」)。 按一下 [確定]。
5. 選取 [應用程式宣告]，然後選取自訂屬性。 按一下 [確定]。
6. 按一下 [檔案] 。

您可以使用原則上的**立即執行**功能，以驗證客戶體驗。 現在您應該會在註冊期間收集的屬性清單中看到 **ShoeSize** ，其亦會在傳回至您應用程式的權杖中出現。

