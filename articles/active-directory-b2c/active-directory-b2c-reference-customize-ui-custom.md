---
title: 使用自訂原則來自訂使用者旅程的 UI | Microsoft Docs
description: 了解 Azure Active Directory B2C 自訂原則。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/25/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: c6882dd56ee96e1fcab5926b77f11ce928bf950b
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37442350"
---
# <a name="customize-the-ui-of-a-user-journey-with-custom-policies"></a>使用自訂原則來自訂使用者旅程的 UI

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

> [!NOTE]
> 本文會進一步說明如何自訂 UI，以及如何使用身分識別體驗架構來對 Azure AD B2C 自訂原則啟用 UI 自訂功能。


順暢的使用者體驗是任何「企業對客戶」解決方案的成功關鍵。 所謂順暢的使用者體驗，不論是在裝置或瀏覽器上，使用者在使用服務時所經歷的旅程，和其使用客戶服務時的旅程沒有差異。

## <a name="understand-the-cors-way-for-ui-customization"></a>了解以 CORS 來自訂 UI 的方法

Azure AD B2C 可讓您在各種頁面上自訂使用者體驗 (UX) 的外觀與風格，並由 Azure AD B2C 使用您的自訂原則來提供和顯示這些自訂。

為了達成該目的，Azure AD B2C 會在取用者的瀏覽器中執行程式碼，並使用現代且標準的方法 ([跨原始資源共用 (CORS)](http://www.w3.org/TR/cors/)) 來載入自訂內容，內容來源則是您在自訂原則中指定為指向 HTML5/CSS 範本的特定 URL。 CORS 機制可讓您從網頁上受限制資源 (如字型) 來源網域以外的其他網域，對該項資源提出要求。

而在傳統的舊式方法中，範本頁面是由解決方案擁有、您只提供有限的文字和影像、版面配置與風格只提供有限的控制能力，因此會導致更多問題而難以實現順暢體驗，相較之下，CORS 方法則支援 HTML5 及 CSS，因此可讓您︰

- 裝載內容，解決方案則會使用用戶端指令碼來插入其控制項。
- 完整控制版面配置與風格的每個像素。

您可以適當地製作 HTML5/CSS 檔案來提供任意數量的內容頁面。

> [!NOTE]
> 為確保安全，系統目前禁止使用 JavaScript 來進行自訂。 若要將 JavaScript 的此一禁制取消，則需要為 Azure AD B2C 租用戶使用自訂的網域名稱。

在每個 HTML5/CSS 範本中，您都需要提供「錨點」元素，以便對應至 HTML 或內容頁面中所需的 `<div id=”api”>` 元素，如以下所述。 Azure AD B2C 要求所有內容頁面都必須有這個特定 div。

```
<!DOCTYPE html>
<html>
  <head>
    <title>Your page content’s tile!</title>
  </head>
  <body>
    <div id="api"></div>
  </body>
</html>
```

頁面的 Azure AD B2C 相關內容會插入這個 div 中，頁面的其餘部分則由您控制。 Azure AD B2C JavaScript 程式碼會提取您的內容，並將 HTML 插入到這個特定的 div 元素。 Azure AD B2C 會適當插入下列控制項︰帳戶選擇器控制項、登入控制項、多因素 (目前為電話式) 控制項和屬性集合控制項。 Azure AD B2C 可確保所有控制項都符合 HTML5 規範且可供存取、所有控制項都可完全自訂樣式，且控制項版本不會倒退。

合併的內容最終會以動態文件的形式向取用者顯示。

若要確保每樣東西都如預期運作，您必須︰

- 確保內容符合 HTML5 規範且可供存取
- 確保內容伺服器已啟用 CORS。
- 透過 HTTPS 提供內容。
- 對所有連結和 CSS 內容使用絕對 URL，例如 https://yourdomain/content。

> [!TIP]
> 若要確認您要用來裝載內容的網站已啟用 CORS 並測試 CORS 要求，您可以使用 http://test-cors.org/ 網站。 由於有此網站，您可以將 CORS 要求傳送到遠端伺服器 (以進行測試，前提是該伺服器支援 CORS)，或將 CORS 要求傳送至測試伺服器 (以瀏覽 CORS 的某些功能)。

> [!TIP]
> http://enable-cors.org/ 網站也會在 CORS 上構成更有用的資源。

由於有此 CORS 式方法，使用者會在您的應用程式與 Azure AD B2C 所提供的頁面之間獲得一致的體驗。

## <a name="create-a-storage-account"></a>建立儲存體帳戶

先決條件是，您必須建立儲存體帳戶。 您需要有 Azure 訂用帳戶才能建立 Azure Blob 儲存體帳戶。 您可以在 [Azure 網站](https://azure.microsoft.com/pricing/free-trial/)上註冊免費試用版。

1. 開啟瀏覽工作階段並瀏覽至 [Azure 入口網站](https://portal.azure.com)。
2. 使用您的系統管理認證來登入。
3. 按一下 [建立資源]  >  [儲存體]  >  [儲存體帳戶]。  [建立儲存體帳戶] 窗格隨即開啟。
4. 在 [名稱] 中，提供儲存體帳戶的名稱，例如 contoso369b2c。 此值稍後會指稱為 storageAccountName。
5. 選擇適當的定價層、資源群組和訂用帳戶。 確定您已核取 [釘選到「開始面板」]  選項。 按一下頁面底部的 [新增] 。
6. 回到「開始面板」，然後按一下您建立的儲存體帳戶。
7. 在 [服務] 區段中，按一下 [Blob]。 [Blob 服務] 窗格隨即開啟。
8. 按一下 [+容器]。
9. 在 [名稱] 中提供容器的名稱，例如 b2c。 此值稍後會指稱為 containerName。
9. 選取 [Blob] 來作為 [存取類型]。 按一下頁面底部的 [新增] 。
10. 您建立的容器將會出現在 [Blob 服務] 窗格的清單中。
11. 關閉 [Blob] 窗格。
12. 在 [儲存體帳戶] 窗格上，按一下**金鑰**圖示。 [存取金鑰] 窗格隨即開啟。  
13. 記下 **key1** 的值。 此值稍後會指稱為 key1。

## <a name="downloading-the-helper-tool"></a>下載協助程式工具

1.  從 [GitHub](https://github.com/azureadquickstarts/b2c-azureblobstorage-client/archive/master.zip) 下載協助程式工具。
2.  將 B2C-AzureBlobStorage-Client-master.zip 檔案儲存在本機電腦上。
3.  在本機磁碟上解壓縮 B2C-AzureBlobStorage-Client-master.zip 檔案的內容，例如在 **UI-Customization-Pack** 資料夾下解壓縮，會在這個資料夾底下建立 *B2C-AzureBlobStorage-Client-master* 資料夾。
4.  開啟該資料夾，並在其中解壓縮 B2CAzureStorageClient.zip 封存檔的內容。

## <a name="upload-the-ui-customization-pack-sample-files"></a>上傳 UI-Customization-Pack 範例檔

1.  使用 Windows 檔案總管，瀏覽至上一節所建立之 UI-Customization-Pack 資料夾底下的 B2C-AzureBlobStorage-Client-master 資料夾。
2.  執行 B2CAzureStorageClient.exe 檔案。 這個程式會將您指定目錄中的所有檔案上傳至您的儲存體帳戶，並允許 CORS 存取這些檔案。
3.  請在提示出現時指定︰a.  您儲存體帳戶 storageAccountName 的名稱，例如 contoso369b2c。
    b.  您的 Azure Blob 儲存體 key1 的主要存取金鑰，例如 contoso369b2c。
    c.  您的儲存體 Blob 儲存體容器 containerName 的名稱，例如 b2c。
    d.  Starter-Pack 範例檔的路徑，例如 ..\B2CTemplates\wingtiptoys。

如果您遵循前述步驟，虛構公司 **wingtiptoys** 的 UI-Customization-Pack 的 HTML5 和 CSS 檔案現在會指向您的儲存體帳戶。  您可以在 Azure 入口網站中開啟相關的容器窗格，以確認該內容已正確上傳。 或者，您也可以從瀏覽器存取頁面來確認該內容已正確上傳。 如需詳細資訊，請參閱 [Azure Active Directory B2C︰用來示範頁面使用者介面 (UI) 自訂功能的協助程式工具](active-directory-b2c-reference-ui-customization-helper-tool.md)。

## <a name="ensure-the-storage-account-has-cors-enabled"></a>確定儲存體帳戶已啟用 CORS

您的端點必須啟用 CORS (跨原始資源共用)，Azure AD B2C 才能載入您的內容。 這是因為您的內容裝載所在的網域，與 Azure AD B2C 用來從中提供頁面的網域不同。

若要確認您要用來裝載內容的儲存體已啟用 CORS，請進行下列步驟︰

1. 開啟瀏覽工作階段，並使用 unified.html 頁面在儲存體帳戶中所在位置的完整 URL `https://<storageAccountName>.blob.core.windows.net/<containerName>/unified.html` 來瀏覽至該頁面。 例如： https://contoso369b2c.blob.core.windows.net/b2c/unified.html。
2. 瀏覽至 http://test-cors.org。這個網站可讓您確認您要使用的頁面已啟用 CORS。  
<!--
![test-cors.org](../../media/active-directory-b2c-customize-ui-of-a-user-journey/test-cors.png)
-->

3. 在 [遠端 URL] 中，輸入 unified.html 內容的完整 URL，然後按一下 [傳送要求]。
4. 確認 [結果] 區段中的輸出包含 XHR status: 200，這表示已啟用 CORS。
<!--
![CORS enabled](../../media/active-directory-b2c-customize-ui-of-a-user-journey/cors-enabled.png)
-->
儲存體帳戶現在應該包含示圖中名為 b2c 的 Blob 容器，其中並包含下列來自 Starter-Pack 的 wingtiptoys 範本。

<!--
![Correctly configured storage account](../../articles/active-directory-b2c/media/active-directory-b2c-reference-customize-ui-custom/storage-account-final.png)
-->

下表說明前述 HTML5 網頁的目的。

| HTML5 範本 | 說明 |
|----------------|-------------|
| phonefactor.html | 此頁面可作為 Multi-Factor Authentication 頁面的範本。 |
| resetpassword.html | 此頁面可作為忘記密碼頁面的範本。 |
| *selfasserted.html* | 此頁面可作為社交帳戶註冊頁面、本機帳戶註冊頁面或本機帳戶登入頁面的範本。 |
| *unified.html* | 此頁面可作為統一之註冊或登入頁面的範本。 |
| *updateprofile.html* | 此頁面可作為設定檔更新頁面的範本。 |

## <a name="add-a-link-to-your-html5css-templates-to-your-user-journey"></a>將 HTML5/CSS 範本的連結新增至使用者旅程

您可以藉由直接編輯自訂原則，將 HTML5/CSS 範本的連結新增至使用者旅程。

要在使用者旅程中使用的自訂 HTML5/CSS 範本，必須在可用於這些使用者旅程的內容定義清單加以指定。 為此，您必須在自訂原則 XML 檔的 <BuildingBlocks> 區段下，宣告選擇性的 <ContentDefinitions> XML 元素。

下表說明 Azure AD B2C 身分識別體驗引擎所能辨識之內容定義識別碼的集合，以及與這些識別碼有關的頁面類型。

| 內容定義識別碼 | 說明 |
|-----------------------|-------------|
| api.error | **錯誤頁面**。 在發生例外狀況或錯誤時，系統會顯示此頁面。 |
| api.idpselections | **識別提供者選取頁面**。 此頁面包含使用者可以在登入期間選擇的識別提供者清單。 這些提供者是企業識別提供者、社交識別提供者 (如 Facebook 和 Google+) 或本機帳戶 (以電子郵件地址或使用者名稱為基礎)。 |
| api.idpselections.signup | **用於註冊的識別提供者選取**。 此頁面包含使用者可以在註冊期間選擇的識別提供者清單。 這些提供者是企業識別提供者、社交識別提供者 (如 Facebook 和 Google+) 或本機帳戶 (以電子郵件地址或使用者名稱為基礎)。 |
| api.localaccountpasswordreset | **忘記密碼頁面**。 此頁面包含表單，使用者必須填寫此表單才能重設其密碼。  |
| api.localaccountsignin | **本機帳戶登入頁面**。 此頁面包含登入表單，使用者必須填寫此表單才能使用以電子郵件地址或使用者名稱為基礎的本機帳戶進行登入。 此表單可以包含文字輸入方塊和密碼輸入方塊。 |
| api.localaccountsignup | **本機帳戶註冊頁面**。 此頁面包含使用者在使用以電子郵件地址或使用者名稱為基礎的本機帳戶註冊時所需填寫的註冊表單。 此表單可以包含不同的輸入控制項，例如文字輸入方塊、密碼輸入方塊、選項按鈕、單選下拉式清單方塊和多選核取方塊。 |
| api.phonefactor | **Multi-Factor Authentication 頁面**。 在此頁面上，使用者可以在註冊或登入期間驗證其電話號碼 (使用文字或語音)。 |
| *api.selfasserted* | **社交帳戶註冊頁面**。 此頁面包含使用者在使用社交識別提供者 (例如 Facebook 或 Google+) 的現有帳戶註冊時所需填寫的註冊表單。 此頁面類似於前述的社交帳戶註冊頁面，但密碼輸入欄位除外。 |
| *api.selfasserted.profileupdate* | **設定檔更新頁面**。 此頁面包含表單，以供使用者用來更新其設定檔。 此頁面類似於前述的社交帳戶註冊頁面，但密碼輸入欄位除外。 |
| *api.signuporsignin* | **統一的註冊或登入頁面**。  此頁面可處理使用者的註冊和登入，這些使用者可使用企業識別提供者、社交識別提供者 (例如 Facebook 或 Google+) 或本機帳戶。

## <a name="next-steps"></a>後續步驟
[參考︰了解自訂原則如何在 B2C 中使用身分識別體驗架構](active-directory-b2c-reference-custom-policies-understanding-contents.md)
