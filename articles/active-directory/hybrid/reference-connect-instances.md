---
title: Azure AD Connect︰同步服務執行個體 | Microsoft Docs
description: 本頁記載 Azure AD 執行個體的特殊考量。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: f340ea11-8ff5-4ae6-b09d-e939c76355a3
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/18/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: a41b236182c18a83b6c83a38fd8420a013313d56
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310367"
---
# <a name="azure-ad-connect-special-considerations-for-instances"></a>Azure AD Connect：執行個體的特殊考量
Azure AD Connect 最常搭配 Azure AD 和 Office 365 的全球執行個體。 但還有其他執行個體，而它們有不同的 URL 需求和其他特殊考量。

## <a name="microsoft-cloud-germany"></a>Microsoft Cloud Germany
[Microsoft Cloud Germany](http://www.microsoft.de/cloud-deutschland) 是由德國資料信託所運作的最高雲端。

| 要在 Proxy 伺服器中開啟的 URL |
| --- |
| \*.microsoftonline.de |
| \*.windows.net |
| +憑證撤銷清單 |

當您登入 Azure AD 租用戶時，必須使用 onmicrosoft.de 網域中的帳戶。

Microsoft Cloud Germany 目前沒有的功能︰

* **密碼回寫**適用於 Azure AD Connect 1.1.570.0 預覽版本和更新的版本。
* 無法使用其他 Azure AD Premium 服務。

## <a name="microsoft-azure-government-cloud"></a>Microsoft Azure Government 雲端
[Microsoft Azure Government 雲端](https://azure.microsoft.com/features/gov/) 是用於美國政府的雲端。

此雲端是以舊版的 DirSync 提供支援。 從 Azure AD Connect 組建 1.1.180 起，即支援新一代的雲端。 這一代使用僅限美國的端點，而且有不同的 URL 清單要在您的 Proxy 伺服器中開啟。

| 要在 Proxy 伺服器中開啟的 URL |
| --- |
| \*.microsoftonline.com |
| \*.microsoftonline.us |
| \*.windows.net (自動 Azure AD 政府租用戶偵測所需的項目) |
| \*.gov.us.microsoftonline.com |
| +憑證撤銷清單 |

> [!NOTE]
> 到 AAD Connect 版本 1.1.647.0，在登錄中設定 AzureInstance 值不再需要 *.windows.net 必須在您的 Proxy 伺服器中開啟。

Microsoft Azure Government 雲端目前沒有的功能︰

* **密碼回寫**適用於 Azure AD Connect 1.1.570.0 預覽版本和更新的版本。
* 無法使用其他 Azure AD Premium 服務。

## <a name="next-steps"></a>後續步驟
深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
