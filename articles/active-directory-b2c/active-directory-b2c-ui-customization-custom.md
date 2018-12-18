---
title: 在 Azure Active Directory B2C 中使用自訂原則來自訂 UI | Microsoft Docs
description: 了解如何在 Azure AD B2C 中使用自訂原則時自訂使用者介面 (UI)。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/04/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 9908a7cf96c56e414e0a8d7faea0352b60214ea4
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37446158"
---
# <a name="azure-active-directory-b2c-configure-ui-customization-in-a-custom-policy"></a>Azure Active Directory B2C︰在自訂原則中設定 UI 自訂

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

完成本文之後，您將擁有一個具備您的品牌和外觀的註冊和登入自訂原則。 使用 Azure Active Directory B2C (Azure AD B2C)，幾乎可完全掌控對使用者呈現的 HTML 和 CSS 內容。 使用自訂原則時，您會以 XML 設定 UI 自訂，而不是在 Azure 入口網站中使用控制項。 

## <a name="prerequisites"></a>先決條件

開始之前，請完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)。 您應該有一個使用本機帳戶來註冊和登入的有效自訂原則。

## <a name="page-ui-customization"></a>頁面 UI 自訂

您可以使用頁面 UI 自訂功能，自訂任何自訂原則的外觀與風格。 您也可以維護應用程式與 Azure AD B2C 之間的品牌和視覺一致性。

運作方式如下：Azure AD B2C 會在客戶的瀏覽器中執行程式碼並使用稱為[跨原始資源共用 (CORS)](http://www.w3.org/TR/cors/) 的新式方法。 首先，在自訂原則中使用自訂 HTML 內容指定 URL。 Azure AD B2C 會合併 UI 元素與從您 URL 載入的 HTML 內容，然後對客戶顯示此頁面。

## <a name="create-your-html5-content"></a>建立您的 HTML5 內容

建立 HTML 內容並在標題中顯示您的產品品牌名稱。

1. 複製下列 HTML 程式碼片段。 它是語式正確的 HTML5，在 *\<body\>* 內標籤內有一個稱為 *\<div id="api"\>\</div\>* 的空元素。 這個元素指出要插入 Azure AD B2C 內容的地方。

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>My Product Brand Name</title>
   </head>
   <body>
       <div id="api"></div>
   </body>
   </html>
   ```

   >[!NOTE]
   >為確保安全，系統目前禁止使用 JavaScript 來進行自訂。

2. 在文字編輯器中貼上所複製的程式碼片段，然後將檔案儲存為 customize-ui.html。

## <a name="create-an-azure-blob-storage-account"></a>建立 Azure Blob 儲存體帳戶

>[!NOTE]
> 在本文中，我們使用 Azure Blob 儲存體來裝載內容。 您可以選擇在 Web 伺服器上裝載您的內容，但必須[在 Web 伺服器上啟用 CORS](https://enable-cors.org/server.html)。

若要在 Blob 儲存體中裝載此 HTML 內容，請執行下列作業：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在 [中樞] 功能表上，選取 [新增] > [儲存體] > [儲存體帳戶]。
3. 輸入儲存體帳戶的 [名稱]。
4. [部署模型] 可保持為 [資源管理員]。
5. 將 [帳戶類型] 變更為 [Blob 儲存體]。
6. [效能] 可保持為 [標準]。
7. [複寫] 可保持為 [RA-GRS]。
8. [存取層] 可保持為 [經常性存取]。
9. [儲存體服務加密] 可保持為 [已停用]。
10. 選取儲存體帳戶的 [訂用帳戶]。
11. 建立**資源群組**，或選取現有的資源群組。
12. 選取儲存體帳戶的 [地理位置]。
13. 按一下 [建立]  建立儲存體帳戶。  
    部署完成後，[儲存體帳戶] 刀鋒視窗會自動開啟。

## <a name="create-a-container"></a>建立容器

若要在 Blob 儲存體中建立公用容器，請執行下列作業：

1. 按一下 [概觀] 索引標籤。
2. 按一下 [容器]。
3. 在 [名稱] 中，輸入 **$root**。
4. 將 [存取類型] 設定為 [Blob]。
5. 按一下 **$root** 以開啟新的容器。
6. 按一下 [上傳] 。
7. 按一下 [選取檔案] 旁的資料夾圖示。
8. 移至 **customize-ui.html**，這是您稍早在[頁面 UI 自訂](#the-page-ui-customization-feature)一節中建立的檔案。
9. 按一下 [上傳] 。
10. 選取您上傳的 customize-ui.html blob。
11. 在 **URL** 旁邊，按一下 [複製]。
12. 在瀏覽器中，貼上所複製的 URL，然後移至網站。 如果無法存取此網站，請確定容器的存取類型設定為 **blob**。

## <a name="configure-cors"></a>設定 CORS

執行下列作業，針對跨原始資源共用設定 Blob 儲存體：

>[!NOTE]
>想要使用我們的範例 HTML 和 CSS 內容來嘗試 UI 自訂功能嗎？ 我們已提供[簡單的協助程式工具](active-directory-b2c-reference-ui-customization-helper-tool.md)，讓您上傳我們的範例內容，並在您的 Blob 儲存體帳戶上設定。 如果您使用此工具，請直接跳到[修改註冊或登入自訂原則](#modify-your-sign-up-or-sign-in-custom-policy)。

1. 在 [設定] 下的 [儲存體] 刀鋒視窗中，開啟 [CORS]。
2. 按一下 [新增] 。
3. 針對 [允許的來源]，輸入星號 (\*)。
4. 在 [允許的動詞命令] 下拉式清單中，同時選取 **GET** 和 **OPTIONS**。
5. 針對 [允許的標頭]，輸入星號 (\*)。
6. 針對 [公開的標頭]，輸入星號 (\*)。
7. 針對 [最長使用期限 (秒)]，輸入 **200**。
8. 按一下 [新增] 。

## <a name="test-cors"></a>測試 CORS

執行下列作業來驗證您已準備就緒：

1. 移至 [www.test-cors.org](http://www.test-cors.org/) 網站，然後在 [遠端 URL] 方塊中貼上 URL。
2. 按一下 [傳送要求]。  
    如果您收到錯誤，請確定您的 [CORS 設定](#configure-cors)正確無誤。 您也可能需要清除瀏覽器快取，或按 Ctrl+Shift+P 來開啟 InPrivate 瀏覽工作階段。

## <a name="modify-your-sign-up-or-sign-in-custom-policy"></a>修改您的註冊或登入自訂原則

在最上層的 \<TrustFrameworkPolicy\> 標籤中，您應該會發現 \<BuildingBlocks\> 標籤。 在 \<BuildingBlocks\> 標籤內，藉由複製下列範例來新增 \<ContentDefinitions\> 標籤。 使用您的儲存體帳戶名稱來取代 *your_storage_account*。

  ```xml
  <BuildingBlocks>
    <ContentDefinitions>
      <ContentDefinition Id="api.idpselections">
        <LoadUri>https://{your_storage_account}.blob.core.windows.net/customize-ui.html</LoadUri>
        <DataUri>urn:com:microsoft:aad:b2c:elements:idpselection:1.0.0</DataUri>
      </ContentDefinition>
    </ContentDefinitions>
  </BuildingBlocks>
  ```

## <a name="upload-your-updated-custom-policy"></a>上傳更新的自訂原則

1. 在 [Azure 入口網站](https://portal.azure.com)中，[切換至您的 Azure AD B2C 租用戶環境](active-directory-b2c-navigate-to-b2c-context.md)，然後開啟 [Azure AD B2C] 刀鋒視窗。
2. 按一下 [所有原則]。
3. 按一下 [上傳原則]。
4. 上傳 `SignUpOrSignin.xml` 與您先前新增的 \<ContentDefinitions\> 標籤。

## <a name="test-the-custom-policy-by-using-run-now"></a>使用 [立即執行] 測試自訂原則

1. 在 [Azure AD B2C] 刀鋒視窗上，移至 [所有原則]。
2. 選取您上傳的自訂原則，按一下 [立即執行] 按鈕。
3. 您應該可以使用電子郵件地址註冊。

## <a name="reference"></a>參考

您可以在此找到 UI 自訂的範例範本：

```
git clone https://github.com/azureadquickstarts/b2c-azureblobstorage-client
```

Sample_templates/wingtip 資料夾包含下列 HTML 檔案：

| HTML5 範本 | 說明 |
|----------------|-------------|
| phonefactor.html | 使用此檔案作為多重要素驗證頁面的範本。 |
| resetpassword.html | 使用此檔案作為忘記密碼頁面的範本。 |
| selfasserted.html | 使用此檔案作為社交帳戶註冊頁面、本機帳戶註冊頁面或本機帳戶登入頁面的範本。 |
| unified.html | 使用此檔案作為統一註冊或登入頁面的範本。 |
| *updateprofile.html* | 使用此檔案作為統一註冊或登入頁面的範本。 |

在[修改註冊或登入自訂原則](#modify-your-sign-up-or-sign-in-custom-policy)區段中，設定 `api.idpselections` 的內容定義。 Azure AD B2C 身分識別體驗架構所能辨識的一組完整內容定義識別碼，而其說明位於下表中：

| 內容定義識別碼 | 說明 | 
|-----------------------|-------------|
| api.error | **錯誤頁面**。 在發生例外狀況或錯誤時，系統會顯示此頁面。 |
| api.idpselections | **識別提供者選取頁面**。 此頁面包含使用者可以在登入期間選擇的識別提供者清單。 這些選項是企業識別提供者、社交識別提供者 (如 Facebook 和 Google+) 或本機帳戶。 |
| api.idpselections.signup | **用於註冊的識別提供者選取**。 此頁面包含使用者可以在註冊期間選擇的識別提供者清單。 這些選項是企業識別提供者、社交識別提供者 (如 Facebook 和 Google+) 或本機帳戶。 |
| api.localaccountpasswordreset | **忘記密碼頁面**。 此頁面包含一份表單，使用者必須填妥此表單才能開始重設密碼。  |
| *api.localaccountsignin* | **本機帳戶登入頁面**。 此頁面包含登入表單，可供使用以電子郵件地址或使用者名稱為基礎的本機帳戶進行登入。 此表單可以包含文字輸入方塊和密碼輸入方塊。 |
| api.localaccountsignup | **本機帳戶註冊頁面**。 此頁面包含登入表單，可供註冊以電子郵件地址或使用者名稱為基礎的本機帳戶。 此表單可以包含各種輸入控制項，例如文字輸入方塊、密碼輸入方塊、選項按鈕、單選下拉式清單方塊和多選核取方塊。 |
| *api.phonefactor* | **Multi-Factor Authentication 頁面**。 在此頁面上，使用者可以在註冊或登入期間驗證其電話號碼 (藉由使用文字或語音)。 |
| api.selfasserted | **社交帳戶註冊頁面**。 此頁面包含使用者在使用社交識別提供者 (例如 Facebook 或 Google+) 的現有帳戶註冊時必須填妥的註冊表單。 此頁面類似於上述的社交帳戶註冊頁面，但密碼輸入欄位除外。 |
| api.selfasserted.profileupdate | **設定檔更新頁面**。 此頁面包含一份表單，以供使用者用來更新其設定檔。 此頁面類似於社交帳戶註冊頁面，但密碼輸入欄位除外。 |
| *api.signuporsignin* | **統一的註冊或登入頁面**。 此頁面可處理使用者的註冊和登入，這些使用者可使用企業識別提供者、社交識別提供者 (例如 Facebook 或 Google+) 或本機帳戶。  |

## <a name="next-steps"></a>後續步驟

如需有關可自訂之 UI 元素的其他資訊，請參閱[內建原則 UI 自訂參考指南](active-directory-b2c-reference-ui-customization.md)。
