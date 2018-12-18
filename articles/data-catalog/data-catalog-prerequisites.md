---
title: Azure 資料目錄必要條件
description: 了解開始使用 Azure 資料目錄所需的必要條件。
services: data-catalog
author: markingmyname
ms.author: maghan
ms.assetid: ef497a54-dc4d-4820-b5bf-c361b64b964d
ms.service: data-catalog
ms.topic: conceptual
ms.date: 01/18/2018
ms.openlocfilehash: 5d05371d9b948dc2f7d6f834eb9431af80fc6365
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406867"
---
# <a name="azure-data-catalog-prerequisites"></a>Azure 資料目錄必要條件

您必須注意幾件事，才能設定 Azure 資料目錄。 別擔心，這個過程不需要很長時間。

## <a name="azure-subscription"></a>Azure 訂用帳戶
若要設定資料目錄，您必須是 Azure 訂用帳戶的擁有者或共同擁有者。

Azure 訂用帳戶可協助您組織雲端服務資源 (例如 Azure 資料目錄) 的存取權。 訂用帳戶也可協助您控制如何根據資源使用量產生報告、計費及付費。 每一個訂用帳戶可以有個別的計費和付款設定；因此，依照部門、專案及分處等，您可以有不同的訂用帳戶和不同的計劃。 每一個雲端服務都屬於某個訂用帳戶，在設定資料目錄之前，您必須先有訂用帳戶。 若要深入了解，請參閱 [管理帳戶、訂用帳戶及管理角色](../active-directory/users-groups-roles/directory-assign-admin-roles.md)。

## <a name="azure-active-directory"></a>Azure Active Directory
若要設定資料目錄，您必須使用 Azure Active Directory (Azure AD) 使用者帳戶登入。

Azure AD 提供了簡單的方法，讓您的企業無論能輕鬆地管理雲端和內部部署中的身分識別與存取權。 使用者可以使用單一公司帳戶或學校帳戶，以單一登入方法登入任何雲端和內部部署 Web 應用程式。 資料目錄採用 Azure AD 來驗證登入。 若要深入了解，請參閱[什麼是 Azure Active Directory？](../active-directory/fundamentals/active-directory-whatis.md)。

> [!NOTE]
> 透過使用 [Azure 入口網站](http://portal.azure.com/)，您可以使用個人 Microsoft 帳戶或 Azure Active Directory 公司或學校帳戶進行登入。 若要使用 Azure 入口網站或[資料目錄入口網站](http://www.azuredatacatalog.com)設定資料目錄，您必須使用 Azure Active Directory 帳戶而非個人帳戶來登入。
>
>

## <a name="active-directory-policy-configuration"></a>Active Directory 原則組態
您可能會遇到一種情況：您可以登入資料目錄入口網站，但在您嘗試登入資料來源註冊工具時會收到錯誤訊息，導致您無法登入。 僅在您處於公司網路或是從公司網路外部連接時，才會發生此問題行為。

資料來源註冊工具使用「表單驗證」，以根據 Active Directory 驗證使用者認證。 Active Directory 系統管理員必須在全域驗證原則中啟用表單驗證，才會成功登入。

在全域驗證原則中，您可以分別為內部網路和外部網路連線啟用驗證方法，如下圖所示。 如果您連線的網路未啟用表單驗證，可能會發生登入錯誤。

 ![Active Directory 全域驗證原則](./media/data-catalog-prerequisites/global-auth-policy.png)

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱 [設定驗證原則](https://technet.microsoft.com/library/dn486781.aspx)。
