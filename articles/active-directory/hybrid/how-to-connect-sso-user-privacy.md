---
title: 使用者隱私權與 Azure AD 無縫單一登入 | Microsoft Docs
description: 本文說明 Azure Active Directory (Azure AD) 無縫 SSO 和 GDPR 合規性。
services: active-directory
keywords: 何謂 Azure AD Connect、GDPR、Azure AD 的必要元件、SSO、單一登入
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/21/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: c7fc6146db93938ed861c6302d2cd8f40b894977
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310387"
---
# <a name="user-privacy-and-azure-ad-seamless-single-sign-on"></a>使用者隱私權與 Azure AD 無縫單一登入

[!INCLUDE [Privacy](../../../includes/gdpr-intro-sentence.md)]

## <a name="overview"></a>概觀


Azure AD 無縫 SSO 會建立下列記錄類型，其中可以包含個人資料： 

- Azure AD Connect 追蹤記錄檔。

有兩種方式可改善無縫 SSO 的使用者隱私權：

1.  在接到要求時擷取使用者的資料，並從安裝中移除該使用者的資料。
2.  確保沒有資料會保留超過 48 小時。

強烈建議您採用第二個選項，因為它比較容易實作和維護。 請針對每個記錄類型參閱下列指示：

### <a name="delete-azure-ad-connect-trace-log-files"></a>刪除 Azure AD Connect 追蹤記錄檔

檢查 **%ProgramData%\AADConnect** 資料夾的內容，並且在安裝或升級 Azure AD Connect 或修改無縫 SSO 組態的 48 小時內，刪除此資料夾的追蹤記錄內容 (**trace-\*.log** 檔案)，因為這個動作可能會建立 GDPR 涵蓋的資料。

>[!IMPORTANT]
>請勿刪除此資料夾中的 **PersistedState.xml** 檔案，因為此檔案是用來維護上一個 Azure AD Connect 安裝的狀態，以及在升級安裝完成時使用。 此檔案從未包含任何個人相關資料，所以不應刪除。

您可以使用 Windows 檔案總管檢閱和刪除這些追蹤記錄檔，或者可以使用下列 PowerShell 指令碼來執行必要的動作：

```
$Files = ((Get-Item -Path "$env:programdata\aadconnect\trace-*.log").VersionInfo).FileName 
 
Foreach ($file in $Files) { 
    {Remove-Item -Path $File -Force} 
}
```

將指令碼儲存在副檔名為 ".PS1" 的檔案中。 視需要執行此指令碼。

若要深入了解相關的 Azure AD Connect GDPR 需求，請參閱[這篇文章](reference-connect-user-privacy.md)。

### <a name="note-about-domain-controller-logs"></a>請注意網域控制站記錄

如果已啟用稽核記錄，這項產品可能會針對您的網域控制站產生安全性記錄。 若要深入了解設定稽核原則，請閱讀[這篇文章](https://technet.microsoft.com/library/dd277403.aspx)。

## <a name="next-steps"></a>後續步驟
* [在信任中心檢閱 Microsoft 隱私權原則](https://www.microsoft.com/trustcenter)
- [**疑難排解**](tshoot-connect-sso.md) - 了解如何解決此功能的常見問題。
- [**UserVoice**](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) - 用於提出新的功能要求。
