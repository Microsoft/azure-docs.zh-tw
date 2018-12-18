---
title: Microsoft Cloud Germany 中的 Azure AD Connect
description: Azure AD Connect 會整合您的內部部署目錄與 Azure Active Directory。 這可讓您為與 Azure AD 整合的 Office 365、Azure 和 SaaS 應用程式提供通用身分識別。
keywords: Azure AD Connect 簡介, Azure AD Connect 概觀, 何謂 Azure AD Connect, 安裝 active directory, 德國, 黑森林
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 2bcb0caf-5d97-46cb-8c32-bda66cc22dad
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 07/12/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: e5377078010e899c41b27ef0ea5248ff4a09df8e
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46301780"
---
# <a name="azure-ad-connect-in-microsoft-cloud-germany---public-preview"></a>Microsoft Cloud Germany 中的 Azure AD Connect - 公開預覽
## <a name="introduction"></a>簡介
Azure AD Connect 可進行內部部署 Active Directory 與 Azure Active Directory 之間的同步處理。
[Microsoft Cloud Germany](https://www.microsoft.com/de-de/cloud/deutschland/default.aspx) 中目前有許多案例必須由操作員來完成。 使用 Microsoft Cloud Germany 時，您必須注意下列事項︰

* 下列 URL 必須在 Proxy 伺服器上開啟，才能成功進行同步處理：
  
  * *.microsoftonline.de
  * *.windows.net
  * * 憑證撤銷清單
* 當您登入 Azure AD 目錄時，必須使用 onmicrosoft.de 網域中的帳戶。

 
## <a name="download"></a>下載
您可以從入口網站中的 [Azure AD Connect] 刀鋒視窗下載 Azure AD Connect。  使用下列指示來尋找 Azure AD Connect 刀鋒視窗。

### <a name="the-azure-ad-connect-blade"></a>Azure AD Connect 刀鋒視窗
登入 Azure 入口網站後，請執行下列動作：

1. 移至瀏覽
2. 選取 Azure Active Directory
3. 然後選取 Azure AD Connect

您應該會看見下列內容：

![Azure AD Connect 刀鋒視窗](./media/reference-connect-germany/germany1.png)

下表描述刀鋒視窗中顯示的功能。

| 標題 | 說明 |
| --- | --- |
| 同步處理狀態 |讓您知道已啟用或停用同步處理。 |
| 上次同步處理 |上次成功完成同步處理。 |
| 同盟網域 |顯示目前設定的同盟網域數目。 |

## <a name="installation"></a>安裝
若要安裝 Azure AD Connect，您可以使用 [這裡](how-to-connect-install-roadmap.md)的文件。

## <a name="advanced-features-and-additional-information"></a>進階功能和其他資訊
如需自訂設定或進階組態的其他資訊和指引，請從 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)著手。  此頁面會提供其他指引的資訊和連結。

