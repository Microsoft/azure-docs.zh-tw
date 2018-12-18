---
title: 在 Visual Studio 中使用已連接服務加入 Azure Active Directory
description: 使用 Visual Studio 的 [加入已連接服務] 對話方塊加入 Azure Active Directory
services: active-directory
author: ghogen
manager: douge
ms.assetid: f599de6b-e369-436f-9cdc-48a0165684cb
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.devlang: multiple
ms.topic: conceptual
ms.date: 03/12/2018
ms.author: ghogen
ms.openlocfilehash: 9b61362525962c5e01d7fef6d317cf9861dfd0a3
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46977888"
---
# <a name="adding-an-azure-active-directory-by-using-connected-services-in-visual-studio"></a>在 Visual Studio 中使用已連接服務加入 Azure Active Directory

藉由使用 Azure Active Directory (Azure AD)，您便可以針對 ASP.NET MVC Web 應用程式支援「單一登入」(SSO)，或在 Web API 服務中支援「Active Directory 驗證」。 在使用 Azure AD 驗證的情況下，您的使用者可以從 Azure Active Directory 使用其帳戶來連線到您的 Web 應用程式。 透過 Web API 的 Azure AD 驗證優點包括從 Web 應用程式公開 API 時的資料安全性會增強。 有了 Azure AD，您不必以各自的帳戶和使用者管理作業來管理個別的驗證系統。

這篇文章和其附屬文件提供使用適用於 Active Directory 之 Visual Studio 連線服務的詳細資料。 此功能適用於 Visual Studio 2017 和 Visual Studio 2015。

目前，Active Directory 連線服務不支援 ASP.NET Core 應用程式。

## <a name="prerequisites"></a>必要條件

- Azure 帳戶：如果您沒有 Azure 帳戶，您可以[申請免費試用](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F)，或是[啟用您的 Visual Studio 訂閱者權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F)。
- **Visual Studio 2015** 或更新版本。 [立即下載 Visual Studio 2017](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs)。

### <a name="connect-to-azure-active-directory-using-the-connected-services-dialog"></a>使用 [已連接服務] 對話方塊來連接到 Azure Active Directory

1. 在 Visual Studio 中，建立或開啟 ASP.NET MVC 專案或 ASP.NET Web API 專案。 您可以使用 MVC、Web API、單頁應用程式、Azure API 應用程式、Azure 行動應用程式及 Azure 行動服務範本。

1. 選取 [專案 > 新增已連線的服務...] 功能表命令，或按兩下在方案總管中專案底下找到的 [已連線的服務] 節點。

1. 在 [已連接服務] 頁面上，選取 [使用 Azure Active Directory 進行驗證]。

    ![[已連接服務] 頁面](./media/vs-azure-active-directory/connected-services-add-active-directory.png)

1. 在 [簡介]  頁面上，選取 [下一步] 。 如果您在此頁面上看到錯誤，請參閱[使用 Azure Active Directory 連線服務來診斷錯誤](vs-active-directory-error.md)。

    ![[簡介] 頁面](./media/vs-azure-active-directory/configure-azure-ad-wizard-1.png)

1. 在 [單一登入] 頁面上，從 [網域] 下拉式清單中選取網域。 清單包含 Visual Studio [帳戶設定] 對話方塊 (**檔案 > 帳戶設定...**) 中所列之帳戶可存取的所有網域。或者，如果您找不到所要尋找的網域 (例如 `mydomain.onmicrosoft.com`)，則可以輸入網域名稱。 您可以選擇可建立 Azure Active Directory 應用程式的選項，或是使用來自現有 Azure Active Directory 應用程式的設定。 完成時，選取 [下一步]。

    ![[單一登入] 頁面](./media/vs-azure-active-directory/configure-azure-ad-wizard-2.png)

1. 在 [目錄存取] 頁面上，選取想要的 [讀取目錄資料] 選項。 開發人員通常會包含此選項。

    ![[目錄存取] 頁面](./media/vs-azure-active-directory/configure-azure-ad-wizard-3.png)

1. 選取 [完成] 開始修改專案，以啟用 Azure AD 驗證。 Visual Studio 會顯示這段期間的進度：

    ![Active Directory 連線服務進度](./media/vs-azure-active-directory/active-directory-connected-service-output.png)

1. 處理程序完成時，Visual Studio 會在瀏覽器中開啟下列其中一篇文章 (視您的專案類型而定)：

    - [開始進行 .NET MVC 專案](vs-active-directory-dotnet-getting-started.md)
    - [開始進行 WebApI 專案](vs-active-directory-webapi-getting-started.md)

1. 您也可以在 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)上看到 Active Directory 網域。

## <a name="how-your-project-is-modified"></a>您的專案修改方式

當您新增連線服務精靈時，Visual Studio 會將 Azure Active Directory 和關聯的參考新增到您的專案。 您專案中的組態檔和程式碼也會進行修改，以加入 Azure AD 支援。 Visual Studio 所做的特定修改視專案類型而定。 請參閱以下文章中的詳細資訊：

- [我的 .NET MVC 專案發生什麼情形？](vs-active-directory-dotnet-what-happened.md)
- [我的 Web API 專案發生什麼情形？](vs-active-directory-webapi-what-happened.md)

## <a name="next-steps"></a>後續步驟

- [Azure Active Directory 的驗證案例](authentication-scenarios.md)
- [將「使用 Microsoft 登入」新增至 ASP.NET Web 應用程式](quickstart-v1-aspnet-webapp.md)
