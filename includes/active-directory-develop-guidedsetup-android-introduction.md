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
ms.date: 09/13/2018
ms.author: andret
ms.custom: include file
ms.openlocfilehash: 962cd4598ab3329786b2682bd2f97801c7943fe1
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48843374"
---
# <a name="sign-in-users-and-call-the-microsoft-graph-from-an-android-app"></a>從 Android 應用程式登入使用者並呼叫 Microsoft Graph

在本教學課程中，您將了解如何建置 Android 應用程式，並將它整合到 Microsoft 身分識別平台中。 具體來說，此應用程式會讓使用者登入、取得存取權杖來呼叫 Microsoft Graph API，以及對 Microsoft Graph API 提出基本要求。  

完成本指南後，您的應用程式會接受使用個人Microsoft 帳戶 (包括 outlook.com、live.com 和其他帳戶)，以及採用 Azure Active Directory 的公司或組織所提供的公司或學校帳戶登入。 

## <a name="how-the-sample-app-generated-by-this-guide-works"></a>本指南產生之範例應用程式的運作方式
![這個範例的運作方式](media/active-directory-develop-guidedsetup-android-intro/android-intro.png)

此範例中的應用程式將會登入使用者，並且代表他們取得資料。  此資料會透過需要授權的遠端 API 來存取 (在此情況下為 Microsoft Graph API)，也會受到 Microsoft 身分識別平台保護。 

具體而言： 
* 您的應用程式將啟動網頁來登入使用者。
* 您的應用程式會針對 Microsoft Graph API 發出存取權杖。
* 此存取權杖會包含在 Web API 的 HTTP 要求中。
* 處理 Microsoft Graph 回應。 

此範例會使用適用於 Android 的 Microsoft 驗證程式庫 (MSAL) 進行協調並協助驗證。MSAL 會自動更新權杖、提供裝置上其他應用程式之間的 SSO、協助管理帳戶，以及處理大部分的條件式存取案例。 

## <a name="prerequisites"></a>必要條件
* 本引導式設定使用 Android Studio 3.0。 
* 需要 Android 21 或更新版本 (建議使用 25 以上)。
* 適用於 Android 的這個 MSAL 版本需要 Google Chrome 或使用「自訂索引標籤」的網頁瀏覽器。

## <a name="library"></a>程式庫

本指南使用下列驗證程式庫：

|程式庫|說明|
|---|---|
|[com.microsoft.identity.client (英文)](http://javadoc.io/doc/com.microsoft.identity.client/msal)|Microsoft Authentication Library (MSAL)|
