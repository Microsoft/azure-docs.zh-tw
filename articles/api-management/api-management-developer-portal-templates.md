---
title: 使用範本自訂 API 管理開發人員入口網站 - Azure | Microsoft Docs
description: 了解如何使用範本自訂 Azure API 管理開發人員入口網站。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: a195675b-f7d0-4fc9-90bf-860e6f17ccf7
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/09/2017
ms.author: apimpm
ms.openlocfilehash: 2bdb8c30ffa630f85d666f3a16ce2b9dcdab6492
ms.sourcegitcommit: 059dae3d8a0e716adc95ad2296843a45745a415d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/09/2018
ms.locfileid: "29118219"
---
# <a name="how-to-customize-the-azure-api-management-developer-portal-using-templates"></a>如何使用範本自訂 Azure API 管理開發人員入口網站。

在 Azure API 管理中自訂開發人員入口網站的基本方式有三種：

* [編輯靜態頁面和頁面配置元素的內容][modify-content-layout]
* [更新用於開發人員入口網站上頁面元素的樣式][customize-styles]
* [修改網站所產生來用於網頁的範本][portal-templates] (本指南會說明)

範本可用來自訂系統產生之開發人員入口網站網頁的內容 (例如 API 文件、產品、使用者驗證等)。 使用 [DotLiquid](http://dotliquidmarkup.org/) 語法及一組提供的當地語系化字串資源、圖示和頁面控制項，您可以依照您的想法自由靈活地設定頁面內容。

## <a name="developer-portal-templates-overview"></a>開發人員入口網站範本概觀
當您以系統管理員身分登入時，可從 [開發人員入口網站] 編輯範本。 若要到達該處，請開啟 Azure 入口網站，然後從 API 管理執行個體的服務工具列按一下 [開發人員入口網站]。

若要存取開發人員入口網站範本，請按一下左側的自訂圖示，顯示 [自訂] 功能表，然後按一下 [範本] 。

![開發人員入口網站範本][api-management-customize-menu]

[範本] 清單會顯示數個範本類別，涵蓋開發人員入口網站的不同頁面。 每個範本各不相同，但是編輯和發佈變更的步驟是一樣的。 若要編輯範本，請按一下範本的名稱。

![開發人員入口網站範本][api-management-templates-menu]

按一下範本即會將您帶到可使用該範本自訂的開發人員入口網站頁面。 這個範例會顯示**產品清單**範本。 **產品清單** 範本控制的畫面區域會以紅色矩形表示。 

![產品清單範本][api-management-developer-portal-templates-overview]

有些範本，像是 **使用者設定檔** 範本，自訂的是同一頁面的不同部分。 

![使用者設定檔範本][api-management-user-profile-templates]

每個開發人員入口網站範本的編輯器都會在頁面底部顯示兩個區段。 左側顯示範本的編輯窗格，右側顯示範本的資料模型。 

範本編輯窗格包含的標記，可控制開發人員入口網站中對應頁面的外觀和行為。 範本中的標記會使用 [DotLiquid](http://dotliquidmarkup.org/) 語法。 常用的 DotLiquid 編輯器是 [DotLiquid for Designers](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Designers)。 在編輯期間對範本進行的任何變更都會即時顯示在瀏覽器中，但您的客戶要到您[儲存](#to-save-a-template)和[發佈](#to-publish-a-template)範本後才看得到。

![範本標記][api-management-template]

[範本資料]  窗格可為能在特定範本中使用的實體，提供有關資料模型的指南。 它提供這份指南的方法是顯示開發人員入口網站中目前顯示的即時資料。 您可以按一下 [範本資料]  窗格右上角的矩形，展開範本窗格。

![範本資料模型][api-management-template-data]

上述範例在開發人員入口網站中顯示了兩項產品，擷取自 [範本資料] 窗格顯示的資料，如下列範例所示：

```json
{
    "Paging": {
        "Page": 1,
        "PageSize": 10,
        "TotalItemCount": 2,
        "ShowAll": false,
        "PageCount": 1
    },
    "Filtering": {
        "Pattern": null,
        "Placeholder": "Search products"
    },
    "Products": [
        {
            "Id": "56ec64c380ed850042060001",
            "Title": "Starter",
            "Description": "Subscribers will be able to run 5 calls/minute up to a maximum of 100 calls/week.",
            "Terms": "",
            "ProductState": 1,
            "AllowMultipleSubscriptions": false,
            "MultipleSubscriptionsCount": 1
        },
        {
            "Id": "56ec64c380ed850042060002",
            "Title": "Unlimited",
            "Description": "Subscribers have completely unlimited access to the API. Administrator approval is required.",
            "Terms": null,
            "ProductState": 1,
            "AllowMultipleSubscriptions": false,
            "MultipleSubscriptionsCount": 1
        }
    ]
}
```

**產品清單** 範本中的標記會處理資料，逐一查看產品集合，顯示每項個別產品的資訊和連結，來提供所需的輸出。 請注意標記中的 `<search-control>` 和 `<page-control>` 元素。 這些控制項會顯示頁面的搜尋和分頁控制項。 `ProductsStrings|PageTitleProducts` 是當地語系化的字串參考，其中包含頁面的 `h2` 標頭文字。 如需字串資源、頁面控制項和開發人員入口網站範本可用圖示的清單，請參閱 [API 管理開發人員入口網站範本參考](api-management-developer-portal-templates-reference.md)。

```html
<search-control></search-control>
<div class="row">
    <div class="col-md-9">
        <h2>{% localized "ProductsStrings|PageTitleProducts" %}</h2>
    </div>
</div>
<div class="row">
    <div class="col-md-12">
    {% if products.size > 0 %}
    <ul class="list-unstyled">
    {% for product in products %}
        <li>
            <h3><a href="/products/{{product.id}}">{{product.title}}</a></h3>
            {{product.description}}
        </li>    
    {% endfor %}
    </ul>
    <paging-control></paging-control>
    {% else %}
    {% localized "CommonResources|NoItemsToDisplay" %}
    {% endif %}
    </div>
</div>
```

## <a name="to-save-a-template"></a>儲存範本
若要儲存範本，請按一下範本編輯器的 [儲存]。

![儲存範本][api-management-save-template]

儲存的變更不會即時顯示在開發人員入口網站中，要發佈後才會顯示。

## <a name="to-publish-a-template"></a>發佈範本
儲存的範本可以個別或一起發佈。 若要發佈個別的範本，請按一下範本編輯器的 [發佈]。

![發佈範本][api-management-publish-template]

按一下 [是]  確認，並讓範本即時顯示在開發人員入口網站中。

![確認發佈][api-management-publish-template-confirm]

若要發佈目前所有尚未發佈的範本版本，請按一下範本清單的 [發佈]。 未發佈的範本會在範本名稱後面標記星號。 本例中要發佈**產品清單**和**產品**範本。

![發佈範本][api-management-publish-templates]

請按一下 [Publish customizations]\(發佈自訂)  確認。

![確認發佈][api-management-publish-customizations]

新發佈的範本在開發人員入口網站中會立即生效。

## <a name="to-revert-a-template-to-the-previous-version"></a>將範本還原成先前的版本
若要將範本還原為先前發佈的版本，請按一下範本編輯器的 [還原]。

![還原範本][api-management-revert-template]

按一下 [ **是** ] 以確認。

![Confirm][api-management-revert-template-confirm]

還原作業一完成，先前發佈的範本版本就會立即顯示在開發人員入口網站中。

## <a name="to-restore-a-template-to-the-default-version"></a>將範本還原成預設的版本
將範本還原成預設版本的程序有兩個步驟。 首先必須還原範本，然後一定要發佈還原的版本。

若要還原單一範本的預設版本，請按一下範本編輯器的 [還原]。

![還原範本][api-management-reset-template]

按一下 [ **是** ] 以確認。

![確認][api-management-reset-template-confirm]

若要還原所有範本的預設版本，請按一下範本清單的 [Restore default templates]\(還原預設範本)  。

![還原範本][api-management-restore-templates]

已還原的範本必須個別發佈，或依照 [發佈範本](#to-publish-a-template)的步驟一次全部發佈。

## <a name="next-steps"></a>後續步驟
如需開發人員入口網站範本、字串資源、圖示和頁面控制項的參考資訊，請參閱 [API 管理開發人員入口網站範本參考](api-management-developer-portal-templates-reference.md)。

[modify-content-layout]: api-management-modify-content-layout.md
[customize-styles]: api-management-customize-styles.md
[portal-templates]: api-management-developer-portal-templates.md

[api-management-customize-menu]: ./media/api-management-developer-portal-templates/api-management-customize-menu.png
[api-management-templates-menu]: ./media/api-management-developer-portal-templates/api-management-templates-menu.png
[api-management-developer-portal-templates-overview]: ./media/api-management-developer-portal-templates/api-management-developer-portal-templates-overview.png
[api-management-template]: ./media/api-management-developer-portal-templates/api-management-template.png
[api-management-template-data]: ./media/api-management-developer-portal-templates/api-management-template-data.png
[api-management-developer-portal-menu]: ./media/api-management-developer-portal-templates/api-management-developer-portal-menu.png
[api-management-management-console]: ./media/api-management-developer-portal-templates/api-management-management-console.png
[api-management-browse]: ./media/api-management-developer-portal-templates/api-management-browse.png
[api-management-user-profile-templates]: ./media/api-management-developer-portal-templates/api-management-user-profile-templates.png
[api-management-save-template]: ./media/api-management-developer-portal-templates/api-management-save-template.png
[api-management-publish-template]: ./media/api-management-developer-portal-templates/api-management-publish-template.png
[api-management-publish-template-confirm]: ./media/api-management-developer-portal-templates/api-management-publish-template-confirm.png
[api-management-publish-templates]: ./media/api-management-developer-portal-templates/api-management-publish-templates.png
[api-management-publish-customizations]: ./media/api-management-developer-portal-templates/api-management-publish-customizations.png
[api-management-revert-template]: ./media/api-management-developer-portal-templates/api-management-revert-template.png
[api-management-revert-template-confirm]: ./media/api-management-developer-portal-templates/api-management-revert-template-confirm.png
[api-management-reset-template]: ./media/api-management-developer-portal-templates/api-management-reset-template.png
[api-management-reset-template-confirm]: ./media/api-management-developer-portal-templates/api-management-reset-template-confirm.png
[api-management-restore-templates]: ./media/api-management-developer-portal-templates/api-management-restore-templates.png







