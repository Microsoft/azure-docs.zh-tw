---
title: Azure Active Directory B2C 中的擴充功能應用程式 | Microsoft Docs
description: 還原 b2c-extensions-app。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 9/06/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: ad3d459b1211d2777f57169f3ee896d2ab5618bc
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37442819"
---
# <a name="azure-ad-b2c-extensions-app"></a>Azure AD B2C：擴充功能應用程式

建立 Azure AD B2C 目錄時，會自動在新的目錄內建立名為 `b2c-extensions-app. Do not modify. Used by AADB2C for storing user data.` 的應用程式。 此應用程式稱為 **b2c-extensions-app**，會顯示在應用程式註冊中。 它是由 Azure AD B2C 服務用來儲存使用者和自訂屬性的相關資訊。 如果將應用程式刪除，Azure AD B2C 就無法正確運作，並會影響您的生產環境。

> [!IMPORTANT]
> 除非您打算立即刪除租用戶，否則請勿刪除 b2c-extensions-app。 若將應用程式刪除超過 30 天，使用者資訊就會永久遺失。

## <a name="verifying-that-the-extensions-app-is-present"></a>正在驗證擴充功能應用程式是否存在

若要驗證 b2c-extensions-app 是否存在：

1. 在您的 Azure AD B2C 租用戶中，按一下左側導覽功能表中的 [所有服務]。
1. 搜尋並開啟 [應用程式註冊]。
1. 尋找以 **b2c-extensions-app** 開頭的應用程式

## <a name="recover-the-extensions-app"></a>復原擴充功能應用程式

如果您不小心刪除 b2c-extensions-app，會有 30 天可進行復原。 您可以使用圖形 API 來還原應用程式：

1. 瀏覽至 [https://graphexplorer.azurewebsites.net/](https://graphexplorer.azurewebsites.net/)。
1. 針對您需要還原已刪除應用程式的 Azure AD B2C 目錄，以全域系統管理員身分登入站台。 此全域系統管理員必須有類似下列的電子郵件地址：`username@{yourTenant}.onmicrosoft.com`。
1. 針對 URL `https://graph.windows.net/myorganization/deletedApplications` 發出 HTTP GET，其中 api-version=1.6。 這項作業會列出過去 30 天內所有已刪除的應用程式。
1. 在名稱開頭為 'b2c-extension-app' 的清單中尋找應用程式，並複製其 `objectid` 屬性值。
1. 針對 URL `https://graph.windows.net/myorganization/deletedApplications/{OBJECTID}/restore` 發出 HTTP POST。 從上一個步驟將 URL 的 `{OBJECTID}` 部分取代為 `objectid`。 

您現在應該能夠在 Azure 入口網站中[看到還原的應用程式](#verifying-that-the-extensions-app-is-present)。

> [!NOTE]
> 只能還原過去 30 天內刪除的應用程式。 如果已經超過 30 天，資料將會永久遺失。 如需協助，請提出支援票證。