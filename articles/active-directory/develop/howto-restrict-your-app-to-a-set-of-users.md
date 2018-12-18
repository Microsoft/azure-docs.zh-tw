---
title: 如何將註冊 Azure Active Directory 的應用程式限制為僅供一組使用者存取
description: 了解如何將已在 Azure AD 中註冊的應用程式限制為僅供一組選取的使用者存取。
services: active-directory
documentationcenter: ''
author: kalyankrishna1
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: kkrishna
ms.reviewer: ''
ms.custom: aaddev
ms.openlocfilehash: 575677cd3ace4d5d4948b1296d923b7703c3f20b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46968300"
---
# <a name="how-to-restrict-your-app-to-a-set-of-users"></a>操作說明：將應用程式限制為僅供一組使用者存取

根據預設，在 Azure Active Directory (Azure AD) 租用戶中註冊的應用程式可供成功通過驗證的所有租用戶使用者使用。

同樣地，在[多租用戶](howto-convert-app-to-be-multi-tenant.md)應用程式的案例中，於此應用程式佈建所在的 Azure AD 租用戶中，使用者一旦成功驗證其各自的租用戶後，都能夠存取此應用程式。

租用戶系統管理員和開發人員通常需要將應用程式限制為僅供一組特定的使用者存取。 開發人員可以使用角色型存取控制 (RBAC) 等熱門的授權模式來完成相同限制，但此方法會讓開發人員這一方必須進行大量的工作。

Azure AD 可讓租用戶系統管理員和開發人員將應用程式限制為僅供租用戶中的一組特定使用者或安全性群組存取。

## <a name="supported-app-configurations"></a>支援的應用程式設定

將應用程式限制為僅供租用戶中一組特定使用者或安全性群組存取的選項，適用於下列類型的應用程式︰

- 設定了同盟單一登入與 SAML 型驗證的應用程式
- 使用 Azure AD 預先驗證的應用程式 Proxy 應用程式
- 直接建置在 Azure AD 應用程式平台上，且會在使用者或系統管理員同意該應用程式之後，使用 OAuth 2.0/OpenID Connect 驗證的應用程式。

     > [!NOTE]
     > 這項功能只適用於 Web 應用程式/Web API 和企業應用程式。 註冊為[原生](quickstart-v1-integrate-apps-with-azure-ad.md)的應用程式無法限制為僅供租用戶中的一組使用者或安全性群組存取。

## <a name="update-the-app-to-enable-user-assignment"></a>更新應用程式以啟用使用者指派

1. 前往 [**Azure 入口網站**](https://portal.azure.com/)並以**全域系統管理員**身分登入。
1. 在頂端列中，選取已登入的帳戶。 
1. 在 [目錄] 底下，選取要在其中註冊應用程式的 Azure AD 租用戶。
1. 在左側的導覽中，選取 [Azure Active Directory]。 如果導覽窗格中沒有 Azure Active Directory，請遵循下列步驟：

    1. 在左側主導覽功能表的頂端選取 [所有服務]。
    1. 在篩選搜尋方塊中輸入 **Azure Active Directory**，然後從結果中選取 [Azure Active Directory] 項目。

1. 在 [Azure Active Directory] 窗格中，選取 [Azure Active Directory] 左側導覽功能表中的 [企業應用程式]。
1. 選取 [所有應用程式]，以檢視所有應用程式的清單。

     如果看不到您希望在此顯示的應用程式，請使用 [所有應用程式] 清單頂端的各種篩選條件來限制清單，或向下捲動清單以找出應用程式。

1. 從清單中選取想要作為使用者或安全性群組指派目標的應用程式。
1. 在應用程式的 [概觀] 頁面中，從應用程式的左側導覽功能表中選取 [屬性]。
1. 找出 [需要使用者指派嗎？] 設定，並將它設定為 [是]。 當此選項設定為 [是] 時，必須先將使用者指派給此應用程式，使用者才能存取此應用程式。
1. 選取 [儲存] 以儲存此設定變更。

## <a name="assign-users-and-groups-to-the-app"></a>將使用者和群組指派給應用程式

在設定完應用程式以啟用使用者指派後，您便可繼續作業並將使用者和群組指派給應用程式。

1. 選取應用程式左側導覽功能表中的 [使用者和群組] 窗格。
1. 在 [使用者和群組] 清單頂端，選取 [新增使用者] 按鈕，以開啟 [新增指派] 窗格。
1. 選取 [新增指派] 窗格中的 [使用者] 選取器。 

     隨即會顯示使用者和安全性群組清單，以及用來搜尋並找出特定使用者或群組的文字方塊。 此畫面可讓您一次選取多個使用者和群組。

1. 在您選好使用者和群組後，請按底部的 [選取] 按鈕，以移至下一個部分。
1. 按底部的 [指派] 按鈕，即可將使用者和群組指派給應用程式。 
1. 確認您新增的使用者和群組有顯示在更新後的 [使用者和群組] 清單中。

