---
title: 存取 Teams 中的 Azure AD App Proxy 應用程式 | Microsoft Docs
description: 使用 Azure AD Application Proxy 透過 Microsoft Teams 存取內部部署應用程式。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/05/2017
ms.author: barbkess
ms.reviewer: harshja
ms.custom: it-pro
ms.openlocfilehash: 1269dfb3aec33e781601a1d885004ddf80127160
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39367193"
---
# <a name="access-your-on-premises-applications-through-microsoft-teams"></a>透過 Microsoft Teams 存取內部部署應用程式

Azure Active Directory Application Proxy 可讓您從任何地方單一登入內部部署應用程式。 Microsoft Teams 會在一個地方簡化您的共同作業工作。 將兩者整合在一起，表示使用者可以在任何情況下與其小組成員一起合作。 

使用者可以[使用所以標籤](https://support.office.com/article/Video-Using-Tabs-7350a03e-017a-4a00-a6ae-1c9fe8c497b3?ui=en-US&rs=en-US&ad=US)將雲端應用程式新增至其 Teams 通道，但如果 SharePoint 網站或規劃工具裝載於內部部署環境，該怎麼辦？ Application Proxy 是解決方案。 他們可以使用其一直用來從遠端存取應用程式的相同外部 URL，將透過 Application Proxy 發佈的應用程式新增到其通道。 而且，因為 Application Proxy 會透過 Azure Active Directory 進行驗證，所以使用者可進行單一登入。


## <a name="install-the-application-proxy-connector-and-publish-your-app"></a>安裝 Application Proxy 連接器並發佈您的應用程式

如果您尚未這麼做，請[設定租用戶的 Application Proxy 並安裝連接器](application-proxy-enable.md)。 然後，[發佈內部部署應用程式](application-proxy-publish-azure-portal.md)以供遠端存取。 當您要發佈應用程式時，請記下外部 URL，因為這會用於將應用程式新增至 Teams。

如果您已經發佈應用程式，但不記得其外部 URL，請在 [Azure 入口網站](https://portal.azure.com)中查閱。 登入，然後瀏覽至 [Azure Active Directory] > [企業應用程式] > [所有應用程式] > 選取您的應用程式 > [Application Proxy]。

## <a name="add-your-app-to-teams"></a>將應用程式新增至 Teams

一旦透過 Application Proxy 發佈應用程式，請讓使用者知道，他們可以在其 Teams 通道中直接將該應用程式新增為一個索引標籤，小組中的每個人將能夠使用該應用程式。 請他們遵循下列三個步驟：

1. 瀏覽至您要新增此應用程式的 Teams 通道，，然後選取 **+** 來新增索引標籤。

   ![選取 [新增索引標籤]](./media/application-proxy-integrate-with-teams/add-tab.png)

2. 從索引標籤選項中選取 [網站]。

   ![新增網站](./media/application-proxy-integrate-with-teams/website.png)

3. 指定索引標籤的名稱，並將 URL 設定為 Application Proxy 外部 URL。 

   ![設定索引標籤名稱和 URL](./media/application-proxy-integrate-with-teams/tab-name-url.png)

一旦有小組成員新增索引標籤，該索引標籤就會對通道中的每個人顯示。 任何具有此應用程式存取權的使用者均可透過其用於 Microsoft Teams 的認證來取得單一登入存取。 任何不具此應用程式存取權的使用者會在 Teams 中看到此索引標籤，但是在您提供內部部署應用程式和應用程式之 Azure 入口網站發佈版本的權限以前，都會遭到封鎖。 

## <a name="next-steps"></a>後續步驟

- 了解如何透過 Application Proxy [發佈內部部署 SharePoint 網站](application-proxy-integrate-with-sharepoint-server.md)。
- 將您的應用程式設定為將[自訂網域](application-proxy-configure-custom-domain.md)用於其外部 URL。 
