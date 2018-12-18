---
title: Azure Active Directory B2C 中的頁面 UI 自訂協助程式工具 | Microsoft Docs
description: 用來示範 Azure Active Directory B2C 中頁面 UI 自訂功能的協助程式工具。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 01/07/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 1a37a37dbed3b5ef9733f1105444529b4d255bcf
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43336776"
---
# <a name="azure-active-directory-b2c-a-helper-tool-used-to-demonstrate-the-page-user-interface-ui-customization-feature"></a>Azure Active Directory B2C︰用來示範頁面使用者介面 (UI) 自訂功能的協助程式工具
本文可搭配 Azure Active Directory (Azure AD) B2C 中的 [主要 UI 自訂文章](active-directory-b2c-reference-ui-customization.md) 一起閱讀。 以下步驟說明如何使用我們提供的範例 HTML 和 CSS 內容，來練習頁面 UI 自訂功能。

## <a name="get-an-azure-ad-b2c-tenant"></a>取得 Azure AD B2C 租用戶
您需要 [取得 Azure AD B2C 租用戶](active-directory-b2c-get-started.md) (如果沒有的話)，才能自訂任何項目。

## <a name="create-a-sign-up-or-sign-in-policy"></a>建立註冊或登入原則
我們提供的範例內容可用來在[註冊或登入原則](active-directory-b2c-reference-policies.md)自訂兩個頁面：[統一的登入頁面](active-directory-b2c-reference-ui-customization.md)和[自我判斷提示的屬性頁面](active-directory-b2c-reference-ui-customization.md)。 [建立註冊或登入原則](active-directory-b2c-reference-policies.md#create-a-sign-up-or-sign-in-policy)時，新增本機帳戶 (電子郵件地址)、Facebook、Google 和 Microsoft 做為 **身分識別提供者**。 我們的範例 HTML 內容只接受這些 IDP。  如果您想要的話，也可以新增這些 IDP 的子集。

## <a name="register-an-application"></a>註冊應用程式
您需要在 B2C 租用戶中 [註冊應用程式](active-directory-b2c-app-registration.md) ，此應用程式可用於執行原則。 註冊應用程式之後，有幾個選項可用來實際執行註冊原則：

* 建置 [在您的應用程式中註冊與登入取用者](active-directory-b2c-overview.md)的＜開始使用＞一節中所列的其中一個 Azure AD B2C 快速啟動應用程式。
* 使用預先建置的 [Azure AD B2C 遊樂場](https://aadb2cplayground.azurewebsites.net) 應用程式。 如果您選擇使用遊樂場，您必須在 B2C 租用戶中使用**重新導向 URI** `https://aadb2cplayground.azurewebsites.net/` 註冊應用程式。
* 在 [Azure 入口網站](https://portal.azure.com/)中，使用原則的 [立即執行] 按鈕。

## <a name="customize-your-policy"></a>自訂您的原則
若要自訂原則的外觀和風格，您必須先使用 Azure AD B2C 的特定慣例來建立 HTML 和 CSS 檔案。 接著可以將靜態內容上載至公開可用的位置，供 Azure AD B2C 存取。 這可以是您自己專屬的 Web 伺服器、Azure Blob 儲存體、Azure 內容傳遞網路，或任何其他的靜態資源裝載提供者。 唯一的條件是您的內容必須可透過 HTTPS 取得，且可使用 CORS 來存取。 一旦將靜態內容公開在 Web 上，您就可以編輯原則來指向這個位置，讓客戶看到此內容。 [主要 UI 自訂文件](active-directory-b2c-reference-ui-customization.md) 詳細說明 Azure AD B2C 自訂功能的運作方式。

基於本教學課程的目的，我們已經建立一些範例內容並裝載於 Azure Blob 儲存體。 範例內容會以虛構的公司 "Wingtip Toys" 為例，示範如何以非常基本的方式自訂佈景主題。 若要在您自己的原則中試用一下，請遵循下列步驟：

1. 在 [Azure 入口網站](https://portal.azure.com/) 上登入您的租用戶並瀏覽至 B2C 功能刀鋒視窗。
2. 按一下 [註冊或登入原則]，然後按一下您的原則，再按一下 [編輯] (例如 "b2c\_1\_sign\_up\_sign\_in")。
3. 依序按一下 [頁面 UI 自訂] 和 [統一的註冊或登入頁面]。
4. 將 [使用自訂頁面] 開關切換到 [是]。 在 [自訂頁面 URI] 欄位中輸入 `https://wingtiptoysb2c.blob.core.windows.net/b2c/wingtip/unified.html`。 按一下 [確定]。
5. 按一下 [本機帳戶註冊頁面] 。 將 [使用自訂範本] 開關切換到 [是]。 在 [自訂頁面 URI] 欄位中輸入 `https://wingtiptoysb2c.blob.core.windows.net/b2c/wingtip/selfasserted.html`。
6. 針對 [社交帳戶註冊頁面] 重複相同步驟。
   按兩次 [確定]  ，關閉 UI 自訂刀鋒視窗。
7. 按一下 [檔案] 。

現在可以試試您自訂的原則。 您可以視需要使用自己的應用程式或 Azure AD B2C 遊樂場，也可以直接按一下原則刀鋒視窗中的 [立即執行]  命令。 在下拉式清單方塊中選取您的應用程式，並選擇適當的重新導向 URI。 按一下 [立即執行]  按鈕。 新的瀏覽器索引標籤隨即開啟，您可以使用已備妥的新內容，在您的應用程式中體驗一次使用者註冊過程！

## <a name="upload-the-sample-content-to-azure-blob-storage"></a>將範例內容上傳至 Azure Blob 儲存體
如果想要使用 Azure Blob 儲存體來裝載您的網頁內容，您可以建立自己的儲存體帳戶，並使用我們的 B2C 協助程式工具來上傳檔案。

### <a name="create-a-storage-account"></a>建立儲存體帳戶
1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下 [+新增]  >  [資料 + 儲存體]  >  [儲存體帳戶]。 您需要有  Azure 訂用帳戶才能建立 Azure Blob 儲存體帳戶。 您可以在 [Azure 網站](https://azure.microsoft.com/pricing/free-trial/)上註冊免費試用版。
3. 提供儲存體帳戶的 [名稱] \(例如 "contoso")，並針對 [定價層]、[資源群組] 和 [訂用帳戶] 挑選適當的選取項目。 確定您已核取 [釘選到「開始面板」]  選項。 按一下頁面底部的 [新增] 。
4. 回到「開始面板」，然後按一下您剛建立的儲存體帳戶。
5. 在 [摘要] 區段中，按一下 [容器]，然後按一下 [+新增]。
6. 提供容器的 [名稱] \(例如 "b2c")，並選取 [Blob] 作為 [存取類型]。 按一下 [確定]。
7. 您建立的容器將會出現在 [Blob]  刀鋒視窗的清單中。 記下容器的 URL；例如，其看起來應該類似 `https://contoso.blob.core.windows.net/b2c`。 關閉 [Blob]  刀鋒視窗。
8. 在儲存體帳戶刀鋒視窗中，按一下 [金鑰]，並記下 [儲存體帳戶名稱] 和 [主要存取金鑰] 欄位的值。

> [!NOTE]
> [主要存取金鑰] 是重要的安全性認證。
> 
> 

### <a name="download-the-helper-tool-and-sample-files"></a>下載協助程式工具和範例檔案
您可以 [下載 Azure Blob 儲存體協助程式工具和 .zip 檔案格式的範例檔案](https://github.com/azureadquickstarts/b2c-azureblobstorage-client/archive/master.zip) ，或是從 GitHub 加以複製：

```
git clone https://github.com/azureadquickstarts/b2c-azureblobstorage-client
```

此儲存機制包含 `sample_templates\wingtip` 目錄，其中含有範例 HTML、CSS 和影像。 為了讓這些範本參考您自己的 Azure Blob 儲存體帳戶，您必須編輯 HTML 檔案。 開啟 `unified.html` 和 `selfasserted.html`，並以您自己容器的 URL (您在上述步驟中記下的 URL) 來取代任何出現的 `https://localhost`。 您必須使用 HTML 檔案的絕對路徑，因為在此案例中，HTML 將是由 Azure AD 在 `tenantname.b2clogin.com`網域下提供。

### <a name="upload-the-sample-files"></a>上傳範例檔案
在相同的儲存機制中，解壓縮 `B2CAzureStorageClient.zip` 並執行其中的 `B2CAzureStorageClient.exe` 檔案。 這個程式只會將您指定目錄中的所有檔案上傳至您的儲存體帳戶，並允許 CORS 存取這些檔案。 如果您遵循上述步驟操作，HTML 和 CSS 檔案現在會指向您的儲存體帳戶。 請注意您的儲存體帳戶名稱是 `blob.core.windows.net` 前面的部分，例如 `contoso`。 您可以試著在瀏覽器上存取 `https://{storage-account-name}.blob.core.windows.net/{container-name}/wingtip/unified.html` ，確認該內容已正確地上傳。 此外，使用 [http://test-cors.org/](http://test-cors.org/) 來確定該內容現在已啟用 CORS。 (在結果中尋找 "XHR status: 200")。

### <a name="customize-your-policy-again"></a>再次自訂您的原則
既然已經將範例內容上載至您自己的儲存體帳戶，您必須編輯註冊原則來參考它。 重複上述 [自訂您的原則](#customize-your-policy) 一節的步驟，但這次使用您自己的儲存體帳戶 URL。 比方說，`unified.html` 檔案的位置是 `<url-of-your-container>/wingtip/unified.html`。

現在，您可以使用 [立即執行]  按鈕或您自己的應用程式，再次執行您的原則。 結果看起來應該幾乎完全相同 -- 您在這兩個案例中都使用相同的範例 HTML 和 CSS。 不過，您的原則現在會參考您自己的 Azure Blob 儲存體執行個體，而您可以再次自由編輯和上傳檔案。 如需自訂 HTML 和 CSS 的詳細資訊，請參閱 [主要 UI 自訂文章](active-directory-b2c-reference-ui-customization.md)。

