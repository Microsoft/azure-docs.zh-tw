---
title: Azure AD Connect 同步︰如何管理 Azure AD 服務帳戶 | Microsoft Docs
description: 本主題將說明如何還原 Azure AD 服務帳戶。
services: active-directory
keywords: AADSTS70002, AADSTS50054, 如何重設 Azure AD Connect 同步處理連接器服務帳戶的密碼
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 6077043a-27f1-4304-a44b-81dc46620f24
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/12/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 93dcbe5ceac87d84db9638de6d5fe120de8c2d36
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46308307"
---
# <a name="azure-ad-connect-sync-how-to-manage-the-azure-ad-service-account"></a>Azure AD Connect 同步處理︰如何管理 Azure AD 服務帳戶
Azure AD 連接器所使用的服務帳戶應該是免費的服務。 如果您需要重設其認證，則這個主題適合您。 例如，如果全域管理員不小心使用 PowerShell 重設了服務帳戶的密碼。

## <a name="reset-the-credentials"></a>重設認證
如果 Azure AD 連接器上定義的服務帳戶因驗證問題而無法連線至 Azure AD，就可以重設密碼。

1. 登入 Azure AD Connect 同步處理伺服器，並啟動 PowerShell。
2. 執行 `Add-ADSyncAADServiceAccount`。  
   ![PowerShell cmdlet addadsyncaadserviceaccount](./media/how-to-connect-azureadaccount/addadsyncaadserviceaccount.png)
3. 提供 Azure AD 全域管理員認證。

這個 Cmdlet 會重設服務帳戶的密碼，並在 Azure AD 和同步處理引擎中加以更新。

## <a name="known-issues-these-steps-can-solve"></a>這些步驟可以解決的已知問題
本節是客戶回報的錯誤清單，這些錯誤已經透過重設 Azure AD 服務帳戶的認證來修正。

- - -
事件 6900  
伺服器在處理密碼變更通知時發生未預期的錯誤︰  
AADSTS70002︰驗證認證時發生錯誤。 AADSTS50054︰使用舊密碼進行驗證。

- - -
事件 659  
擷取密碼原則同步處理設定時發生錯誤。 Microsoft.IdentityModel.Clients.ActiveDirectory.AdalServiceException：  
AADSTS70002︰驗證認證時發生錯誤。 AADSTS50054︰使用舊密碼進行驗證。

## <a name="next-steps"></a>後續步驟
**概觀主題**

* [Azure AD Connect 同步處理：了解及自訂同步處理](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)

