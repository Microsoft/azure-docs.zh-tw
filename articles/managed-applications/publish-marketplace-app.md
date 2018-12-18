---
title: Marketplace 中 Azure 受控應用程式 | Microsoft Docs
description: 描述可透過 Marketplace 取得的 Azure 受控應用程式。
services: managed-applications
author: tfitzmac
manager: timlt
ms.service: managed-applications
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.date: 07/10/2018
ms.author: tomfitz
ms.openlocfilehash: 8f35bda8c6925bdc10097ac6d180f5998bd5cf1d
ms.sourcegitcommit: df50934d52b0b227d7d796e2522f1fd7c6393478
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38989780"
---
# <a name="azure-managed-applications-in-the-marketplace"></a>Marketplace 中 Azure 受控應用程式

廠商可以使用 Azure 受控應用程式，將其解決方案提供給所有 Azure Marketplace 客戶。 這些廠商可以包含受控服務提供者 (MSP)、獨立軟體廠商 (ISV) 和系統整合者 (SI)。 受控應用程式能減少客戶的維護與服務額外負荷。 廠商可以透過 Marketplace 銷售基礎結構和軟體。 他們可以將服務和操作支援附加至受控應用程式。 如需詳細資訊，請參閱[受控應用程式概觀](overview.md)。

這篇文章說明您如何將應用程式發佈至市集，並將它廣泛提供給客戶。

## <a name="prerequisites-for-publishing-a-managed-application"></a>發行受控應用程式的必要條件

若要完成這篇文章，您必須具備受控應用程式定義的 .zip 檔案。 如需詳細資訊，請參閱[建立服務類別目錄應用程式](publish-service-catalog-app.md)。

此外，還有數個商務必要條件。 如下：

* 您的公司或其子公司必須位於 Marketplace 支援銷售的國家/地區。
* 您的產品授權，必須與 Marketplace 支援的計費模式相容。
* 以合乎商業行為的方式為客戶提供技術支援。 支援可以為免費、付費，或透過社群支援。
* 為您的軟體和任何第三方軟體相依性進行授權。
* 為了使您的供應項目可列於 Marketplace 和 Azure 入口網站中，請提供符合標準的內容。
* 同意 Azure Marketplace 參與原則和發行者合約中的條款。
* 同意遵守使用條款、Microsoft 隱私權聲明以及 Microsoft Azure 認證方案合約。

## <a name="become-a-publisher"></a>成為發行者

若要成為 Azure Marketplace 中的發行者，您必須：

1. 建立 Microsoft ID - 使用屬於貴公司網域，但不屬於單一個人的電子郵件地址，建立您的 Microsoft 帳戶。 此電子郵件地址用於 Microsoft 開發人員中心和雲端合作夥伴入口網站。 如需詳細資訊，請參閱 [Azure Marketplace 發行者指南](https://aka.ms/sellerguide)。
1. 提交 [Azure Marketplace 提名表單](https://aka.ms/ampnomination) - 針對**您想要發行的解決方案？**，選取 [受控應用程式]。 一旦提交表單，Marketplace 上架小組會檢閱應用程式並驗證要求。 核准程序可能需要一到三天。 您的提名核准時，您會收到促銷代碼，以撤銷開發人員中心的註冊費用。 如果您**未**完成 Marketplace 提名表單，系統會要求您支付 $99 的註冊費用。
1. 在[開發人員中心](http://dev.windows.com/registration?accountprogram=azure)註冊 - Microsoft 會驗證您的組織是有效的法律實體，具有註冊所在國家/地區的有效統一編號。 核准程序可能需要 5 到 10 天。 若要避免註冊費用，使用您在提名程序電子郵件中收到的促銷代碼。 如需詳細資訊，請參閱 [Azure Marketplace 發行者指南](https://aka.ms/sellerguide)。
1. 登入 [Cloud Partner 入口網站](https://cloudpartner.azure.com) - 在發行者設定檔中，將您的開發人員中心帳戶與 Marketplace 發行者設定檔產生關聯。 如需詳細資訊，請參閱 [Azure Marketplace 發行者指南](https://aka.ms/sellerguide)。

## <a name="create-a-new-azure-application-offer"></a>建立新的 Azure 應用程式供應項目

建立您的合作夥伴入口網站帳戶之後，您已準備好建立您的受控應用程式供應項目。

### <a name="set-up-an-offer"></a>設定供應項目

受控應用程式的供應項目對應至發行者提供的產品分類供應項目。 如果您有想要在 Marketplace 中提供的新應用程式類型，可以將它設定為新的供應項目。 供應項目 SKU 的集合。 每個供應項目會以個別實體出現在 Marketplace。

1. 登入 [Cloud Partner 入口網站](https://cloudpartner.azure.com/)。

1. 在左側瀏覽窗格中，選取 [+ 新增供應項目] > **[Azure 應用程式]**。

1. 在 [編輯器] 檢視中，您會看見所需的表單。 本文稍後將說明每個表單。

## <a name="offer-settings-form"></a>供應項目設定表單

[供應項目設定] 表單的欄位是：

* **供應項目 ID** - 此唯一識別碼可識別發行者設定檔內的供應項目。 此識別碼會顯示在產品的 URL，Resource Manager 範本和計費報告中。 此識別碼只能包含小寫英數字元或連字號 (-)。 識別碼不能以連字號結尾。 最多不能超過 50 個字元。 供應項目上架後，此欄位便會鎖住。
* **發行者 ID**：使用此下拉式表單，選擇您想要在哪個發行者設定檔之下發佈此供應項目。 供應項目上架後，此欄位便會鎖住。
* **名稱**：供應項目的這個顯示名稱會出現在 Marketplace 和入口網站中。 它最多不能超過 50 個字元。 包含產品的可辨識品牌名稱。 請勿在此包含您的公司名稱，除非這是行銷方式。 如果您在自己的網站行銷此供應項目，請確認名稱與您網站顯示的名稱相同。

完成時，選取 [儲存] 儲存您的進度。

## <a name="skus-form"></a>SKU 表單

下一個步驟是為您的供應項目新增 SKU。

SKU 是供應項目的最小購買單位。 您可以使用相同產品類別 (供應項目) 內的 SKU 來區分：

* 所支援的不同功能
* 供應項目為受控或非受控
* 支援的計費模型

SKU 會顯示在 Marketplace 中的父供應項目底下。 它會在 Azure 入口網站中顯示為自己的可購買實體。

1. 選取 [SKU] > [新增 SKU]。

1. 輸入 [SKU 識別碼]。 SKU 識別碼是在供應項目內 SKU 的唯一識別碼。 此識別碼會顯示在產品的 URL，Resource Manager 範本和計費報告中。 此識別碼只能包含小寫英數字元或連字號 (-)。 此識別碼不能以連字號結尾，且最多不能超過 50 個字元。 供應項目上架後，此欄位便會鎖住。 您可以在一個供應項目內建立多個 SKU。 您預計發佈的每個映像都需要 SKU。

1. 填寫下列表單上的 [SKU 詳細資料] 區段：

   填寫下列欄位：

   * **標題**：輸入此 SKU 的標題。 此標題顯示在此項目的資源庫中。
   * **摘要**：輸入此 SKU 的簡短摘要。 此文字會出現在標題底下。
   * **描述**：提供有關 SKU 的詳細描述。
   * **SKU 類型**：允許的值包括 [受控應用程式] 和 [解決方案範本]。 此案例中，選取 [受控應用程式]。
   * **國家/地區可用性**：選取可使用受控應用程式的國家/地區。
   * **價格**：提供應用程式的管理價格。 先選取可用的國家，再設定價格。

1. 新增套件。 填寫下列表單上的 [套件詳細資料] 區段：

   填寫下列欄位：

   * **版本**：輸入您上傳的套件版本。 其格式應該是 `{number}.{number}.{number}{number}`。
   * **套件檔案 (.zip)**：此套件包含壓縮成 .zip 套件的兩個必要檔案。 其中一個檔案為資源管理員範本，此範本定義要與受控應用程式一起部署的資源。 另一個檔案會針對透過入口網站部署受控應用程式的取用者，定義[使用者介面](create-uidefinition-overview.md)。 在使用者介面中，您可以指定讓取用者提供參數值的項目。
   * **PrincipalId**：這個屬性是使用者、使用者群組或應用程式的 Azure Active Directory (Azure AD) 識別碼，其擁有者被授與客戶訂用帳戶中資源的存取權。 角色定義描述權限。
   * **角色定義**：這個屬性是 Azure AD 提供的所有內建角色型存取控制 (RBAC) 角色清單。 您可以選取最適合用於代表客戶管理資源的角色。
   * **原則設定**：將 [Azure 原則](../azure-policy/azure-policy-introduction.md)套用至您的受控應用程式，以指定已部署解決方案的合規性需求。 從可用的選項中，選取要套用的原則。 針對 [原則參數]，提供包含參數值的 JSON 字串。 如需了解原則定義和參數值格式，請參閱 [Azure 原則範例](../azure-policy/json-samples.md)。

您可以新增數個授權。 建議您建立 AD 使用者群組並且在 **PrincipalId** 中指定其識別碼。 如此一來，您可以將更多使用者新增至使用者群組，而不需要更新 SKU。

如需 RBAC 的詳細資訊，請參閱[在 Azure 入口網站中開始使用 RBAC](../role-based-access-control/overview.md)。

## <a name="marketplace-form"></a>Marketplace 表單

Marketplace 表單會要求 [Azure Marketplace](https://azuremarketplace.microsoft.com) 以及 [Azure 入口網站](https://portal.azure.com/)上顯示的欄位。

### <a name="preview-subscription-ids"></a>預覽訂用帳戶識別碼

輸入 Azure 訂用帳戶識別碼清單，這些訂用帳戶可以在供應項目發佈之後予以存取。 您可以使用這些允許的訂用帳戶，在供應項目上架前測試預覽的供應項目。 您可以在合作夥伴入口網站中將多達 100 個訂用帳戶編入允許清單中。

### <a name="suggested-categories"></a>建議的類別

從清單中選擇與您的供應項目最有關聯的五種類別。 這些類別用於將您的供應項目對應至 [Azure Marketplace](https://azuremarketplace.microsoft.com) 和 [Azure 入口網站](https://portal.azure.com/)中提供的產品類別。

#### <a name="azure-marketplace"></a>Azure Marketplace

受控應用程式摘要會顯示下列欄位：

![Marketplace 摘要](./media/publish-marketplace-app/publishvm10.png)

受控應用程式的 [概觀] 索引標籤會顯示下列欄位：

![Marketplace 概觀](./media/publish-marketplace-app/publishvm11.png)

受控應用程式的 [方案 + 價格] 索引標籤會顯示下列欄位：

![Marketplace 方案](./media/publish-marketplace-app/publishvm15.png)

#### <a name="azure-portal"></a>Azure 入口網站

受控應用程式摘要會顯示下列欄位：

![入口網站摘要](./media/publish-marketplace-app/publishvm12.png)

受控應用程式的概觀會顯示下列欄位：

![入口網站概觀](./media/publish-marketplace-app/publishvm13.png)

#### <a name="logo-guidelines"></a>標誌指導方針

您在 Cloud Partner 入口網站中上傳的所有標誌都應該遵循下列指導方針︰

*   Azure 設計具有簡單的調色盤。 限制標誌上的主要和次要色彩數目。
*   入口網站的佈景主題色彩是白色與黑色。 請勿使用這些色彩作為標誌的背景色彩。 請使用可讓標誌在入口網站突顯出來的色彩。 建議您使用簡單的主要色彩。 如果您使用透明背景，請確定標誌和文字不是白色、黑色或藍色。
*   請不要在標誌上使用漸層背景。
*   請勿在標誌上放置文字 (甚至是公司或品牌名稱)。 標誌的外觀與風格應該是「一般」，且避免使用漸層。
*   確定標誌不會自動縮放。

#### <a name="hero-logo"></a>主圖標誌

主圖標誌是選擇性的。 發行者可以選擇不上傳主圖標誌。 上傳主圖圖示之後，便無法刪除。 屆時合作夥伴必須遵循主圖圖示的 Marketplace 指導方針。

主圖標誌圖示應該遵循下列指導方針：

*   發行者顯示名稱、方案標題和供應項目完整摘要會以白色顯示。 因此，請勿使用淺色作為主圖圖示的背景。 主圖圖示不得使用黑色、白色和透明背景。
*   供應項目列出來之後，元素會以程式設計方式內嵌在主圖標誌內。 內嵌的元素包括發佈者顯示名稱、方案標題、供應項目完整摘要、[建立] 按鈕。 因此，請勿在設計主圖標誌時輸入任何文字。 在右邊留下空白空間，因為系統會以程式設計方式文字放入該空間。 右側的文字空白空間應該是 415 x 100 像素。 它是從左邊位移 370 像素。

    ![主圖標誌範例](./media/publish-marketplace-app/publishvm14.png)

## <a name="support-form"></a>支援表單

在 [支援] 表單中填寫貴公司的支援連絡人。 此資訊可能是工程連絡人和客戶支援連絡人。

## <a name="publish-an-offer"></a>發佈供應項目

填妥所有區段之後，請選取 [發佈] 開始讓客戶可使用供應項目的程序。

## <a name="next-steps"></a>後續步驟

* 如需受控應用程式的簡介，請參閱[受控應用程式概觀](overview.md)。
* 如需發佈 Service Catalog 受控應用程式的資訊，請參閱[建立和發佈 Service Catalog 受控應用程式](publish-service-catalog-app.md)。