---
title: 包含檔案
description: 包含檔案
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mtillman
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/17/2018
ms.author: andret
ms.custom: include file
ms.openlocfilehash: b18b6e2fd5bb2071f57033dbe74ef87da028720b
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48843076"
---
## <a name="configure-your-aspnet-web-app-with-the-applications-registration-information"></a>使用應用程式的註冊資訊設定您的 ASP.NET Web 應用程式

在這個步驟中，您將會設定專案使用 SSL，然後使用 SSL URL 設定應用程式的註冊資訊。 設定完成後，請透過 *web.config* 將應用程式的註冊資訊新增到方案。

1.  在「方案總管」中，選取專案並查看 `Properties` 視窗 (如果您沒有看到 [屬性] 視窗，請按 F4)
2.  將 `SSL Enabled` 變更為 `True`
3.  從上面的 `SSL URL` 複製值並貼到此頁面頂端的 `Redirect URL` 欄位中，然後按一下 [更新]：<br/><br/>![專案屬性](media/active-directory-develop-guidedsetup-aspnetwebapp-configure/vsprojectproperties.png)<br />
4.  在位於根資料夾之 `web.config` 檔案中的 `configuration\appSettings` 區段底下新增以下內容：

```xml
<add key="ClientId" value="[Enter the application Id here]" />
<add key="RedirectUri" value="[Enter the Redirect URL here]" />
<add key="Tenant" value="common" />
<add key="Authority" value="https://login.microsoftonline.com/{0}/v2.0" /> 
```
