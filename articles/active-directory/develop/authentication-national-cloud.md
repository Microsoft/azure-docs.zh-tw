---
title: 在國家雲端中使用 Azure AD 進行驗證
description: 了解國家雲端的應用程式註冊和驗證端點。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: negoe
ms.reviewer: negoe,andret,saeeda
ms.custom: aaddev
ms.openlocfilehash: 866a86178d66b7b4af069d684e4eb56c12db47ca
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981996"
---
# <a name="national-clouds"></a>國家雲端

國家雲端 (也稱為主權雲端) 是實際隔離的 Azure 執行個體。 這些 Azure 區域的設計目的，是要確保符合地理界限內的資料落地、主權及合規性需求。

包含全球雲端在內，Azure Active Directory 會部署至下列國家雲端：  

- Azure US Government
- Azure Germany
- Azure China 21Vianet

## <a name="app-registration-endpoints"></a>應用程式註冊端點

下表列出用來針對每個國家雲端註冊應用程式的 Azure Active Directory (Azure AD) 端點基底 URL。

| 國家雲端 | Azure AD 入口網站端點
| --- | --- |
| 適用於美國政府的 Azure AD |https://portal.azure.us
|Azure AD Germany |https://portal.microsoftazure.de
|由 21Vianet 營運的 Azure AD China |https://portal.azure.cn
|Azure AD (全球服務)|https://portal.azure.com

## <a name="azure-ad-authentication-endpoints"></a>Azure AD 驗證端點

下表列出 Azure Active Directory (Azure AD) 端點的基底 URL，這些端點可用來取得權杖以呼叫每個國家雲端的 Microsoft Graph。

| 國家雲端 | Azure AD 驗證端點
| --- | --- |
| 適用於美國政府的 Azure AD |`https://login.microsoftonline.us`
|Azure AD Germany| `https://login.microsoftonline.de`
|由 21Vianet 營運的 Azure AD China | `https://login.chinacloudapi.cn`
|Azure AD (全球服務)|`https://login.microsoftonline.com`

使用適當的區域專屬基底 URL，即可形成對 Azure AD 授權或權杖端點的要求。 例如，如果是德國：

- 授權通用端點為 `https://login.microsoftonline.de/common/oauth2/authorize`
- 權杖通用端點為 `https://login.microsoftonline.de/common/oauth2/token` 

對於單一租用戶應用程式，將上述 URL 的 common 取代為租用戶識別碼或名稱，例如 `https://login.microsoftonline.de/contoso.com`

>[!NOTE]
> [Azure AD v2.0 授權]( https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)和權杖端點僅適用於全球服務。 尚未支援國家雲端部署。

## <a name="next-steps"></a>後續步驟

- 深入了解 [Azure Government](https://docs.microsoft.com/azure/azure-government/)
- 深入了解 [Azure China 21Vianet](https://docs.microsoft.com/azure/china/)
- 深入了解 [Azure Germany](https://docs.microsoft.com/azure/germany/)
- 了解 [Azure AD 驗證基本概念](authentication-scenarios.md)
- 深入了解[國家雲端中的 Microsoft Graph 部署](https://developer.microsoft.com/graph/docs/concepts/deployments)