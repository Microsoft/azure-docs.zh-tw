---
title: Power BI 工作區集合常見問題集
description: Power BI 工作區集合相關的常見問題。
services: power-bi-embedded
author: markingmyname
ROBOTS: NOINDEX
ms.assetid: 1475ed4f-fc84-4865-b243-e8a47d8bda59
ms.service: power-bi-embedded
ms.topic: article
ms.workload: powerbi
ms.date: 09/25/2017
ms.author: maghan
ms.openlocfilehash: e6e2c0b82f306e399454c1672c56e2933e3f1158
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43040860"
---
# <a name="power-bi-workspace-collections-faq"></a>Power BI 工作區集合常見問題集

> [!IMPORTANT]
> Power BI 工作區集合已被取代，只能使用到 2018 年 6 月或您的合約所指出的時間。 建議您進行規劃以移轉至 Power BI Embedded，以免應用程式發生中斷。 如需如何將資料移轉至 Power BI Embedded 的資訊，請參閱[如何將 Power BI 工作區集合的內容移轉至 Power BI Embedded](https://powerbi.microsoft.com/documentation/powerbi-developer-migrate-from-powerbi-embedded/)。

## <a name="what-is-microsoft-power-bi-workspace-collections"></a>什麼是 Microsoft Power BI 工作區集合？
Power BI 工作區集合是一項 Azure 服務，可讓應用程式開發人員在面對客戶的應用程式中，內嵌令人讚嘆、具全面互動性的報表和視覺效果，而不需花費時間與金錢從頭開始自行建置控制項。 我們現在有 Power BI 工作區集合 (具有 SLA) 適用於全球的 9 個資料中心。 我們也在服務中提供增強功能，例如，透過 Power BI 工作區集合中的資料列層級安全性 (RLS) 支援資料安全性以進行進階篩選。 我們也已簡化並更新 Power BI 工作區集合定價模式。

## <a name="who-would-want-to-use-microsoft-power-bi-workspace-collections-and-why"></a>誰會想要以及為何要使用 Microsoft Power BI 工作區集合？
Microsoft Power BI 工作區集合是針對想要跨任何裝置為使用者提供令人讚嘆且互動式的資料視覺效果體驗，但不想自行建置視覺效果體驗的應用程式開發人員所設計。 使用 Power BI 工作區集合，開發人員就可以使用「直接查詢」持續提供最新的檢視。 開發人員也可以使用 Azure Resource Manager API 和 Power BI API，以程式設計方式部署、管理及自動化 Power BI。 和 Power BI 的所有項目一樣，內嵌服務會自動調整以符合您應用程式的使用狀況和需求。 Power BI 工作區集合服務具有依據使用情況計費基礎的定價模式特色。

## <a name="how-does-power-bi-workspace-collections-relate-to-the-power-bi-service"></a>Power BI 工作區集合與 Power BI 服務之間的關聯為何？
Power BI 工作區集合和 Power BI 服務是不同的供應項目。 Power BI 工作區集合具有以使用情況為基礎的計費模式和透過 Azure 入口網站部署等特色，並且是針對讓 ISV 能夠在應用程式中內嵌資料視覺效果以供其客戶使用所設計的。 Power BI 服務是透過 O365 入口網站計費和部署，為獨立的一般目的 BI 供應項目，主要目標鎖定在企業內部用途。 若要深入了解 Power BI 服務，請參閱 [www.powerbi.com](https://powerbi.microsoft.com)。

## <a name="how-does-power-bi-workspace-collections-improve-my-app"></a>Power BI 工作區集合如何改善我的應用程式？
當您在應用程式中利用絢麗、互動式資料視覺效果直接告知使用者決策時，應用程式明顯變得更加強大。 Power BI 工作區集合可讓您使用互動式、永遠最新及豐富的資料視覺效果來增強您的應用程式，讓您可以在任何裝置上輕鬆提高應用程式使用率、使用者滿意度和忠誠度，以及提供內容相關分析。

## <a name="are-there-any-rules-or-restrictions-about-how-i-can-use-power-bi-workspace-collections-in-my-app"></a>有沒有任何關於如何在應用程式中使用 Power BI 工作區集合的規則或限制？
Power BI 工作區集合的提供對象是您提供給協力廠商使用的應用程式。 若想使用 Power BI 工作區集合服務建立內部商務應用程式，您的所有內部使用者皆須具備 Power BI Pro USL，且您的組織除了 Power BI Pro USL 費用之外，將還須支付 Power BI 工作區集合服務的使用費用。 為了避免同時產生內部應用程式的 Power BI Pro USL 費用及Power BI 工作區集合使用費用，Power BI 服務為 Power BI USL 持有者免費在 Power BI 工作區集合之外另提供自身的內容內嵌功能 (dev.powerbi.com)。

## <a name="can-power-bi-workspace-collections-be-used-to-create-internal-applications"></a>能否使用 Power BI 工作區集合來建立內部應用程式？
不行，Power BI 工作區集合僅供外部使用者使用，不應該在內部商務應用程式內使用。 若要將 Power BI 內容內嵌為供內部商務應用程式使用，則您應該使用 Power BI 服務，而使用該內容的所有使用者皆必須具備有效的 Power BI Free 或 Power BI Pro 使用者訂用帳戶授權。 如需如何使用 Power BI 服務整合內部應用程式的詳細資訊，請前往 [https://dev.powerbi.com](https://dev.powerbi.com)。

## <a name="is-this-service-available-globally"></a>這是全球都可使用的服務嗎？
Power BI 工作區集合服務現在已可在大部分的資料中心內使用。 您一律可以在 [這裡](https://azure.microsoft.com/status/)檢查最新的可用性。

## <a name="what-is-the-available-sla-for-the-service"></a>此服務可用的 SLA 是？
使用 Azure 標準 SLA 的 Power BI 工作區集合。 如需詳細資訊，請參閱 [服務等級協定](https://azure.microsoft.com/support/legal/sla/) 。

## <a name="what-is-a-report-session-and-how-is-it-billed"></a>什麼是報告工作階段？如何計費？
工作階段是指使用者與 Power BI 工作區集合報表間的一組互動。 工作階段會在每次為使用者顯示 Power BI 工作區集合報表時起始，而訂用帳戶持有者須支付工作階段費用。 工作階段以統一費率計費，獨立於報表中的視覺元素數目或報表內容重新整理的頻率。 工作階段會在使用者關閉報告或是工作階段逾時一小時後結束。

## <a name="do-you-offer-any-tools-or-guidance-to-help-me-estimate-how-many-renderssession-i-should-expect-how-will-i-know-how-many-renders-have-been-completed"></a>是否有提供任何工具或指導方針以協助我預估應該預期會有多少轉譯/工作階段？ 如何知道已經完成多少轉譯？
Azure 入口網站會針對您的訂用帳戶提供已經執行多少轉譯/報表工作階段的相關計費詳細資料。

## <a name="do-i-need-a-power-bi-subscription-in-order-to-develop-applications-with-power-bi-workspace-collections-how-do-i-get-started"></a>是否需要有 Power BI 訂用帳戶，才能使用 Power BI 工作區集合來開發應用程式？ 如何開始使用？
身為應用程式開發人員，您不需要有 Power BI 訂用帳戶就能夠建立您想要在應用程式中使用的報表與視覺效果。 您需要 Microsoft Azure 訂用帳戶和免費的 Power BI Desktop 應用程式。

請參閱我們的服務文件，以了解如何使用 Power BI 工作區集合服務的相關資訊。

## <a name="i-have-an-azure-subscription-can-i-use-power-bi-workspace-collections-using-my-existing-subscription"></a>我有 Azure 訂用帳戶。 我可以利用現有的訂用帳戶來使用 Power BI 工作區集合嗎？
是。 您可以使用現有的 Azure 訂用帳戶來佈建及使用 Microsoft Power BI 工作區集合服務。

## <a name="does-my-application-end-user-need-a-power-bi-license"></a>我的應用程式的使用者是否需要 Power BI 授權？
否。 您應用程式的使用者不需購買個別的 Power BI 訂用帳戶，就能存取應用程式內資料視覺效果。 在 Power BI 工作區集合模型中，應用程式提供者需透過 Azure 使用計量來為服務付費。 請參閱[定價和授權頁面](http://go.microsoft.com/fwlink/?LinkId=760527)。

## <a name="how-does-user-authentication-work-with-power-bi-workspace-collections"></a>如何使用 Power BI 工作區集合來執行使用者驗證工作？
Power BI 工作區集合服務會使用應用程式權杖進行驗證和授權，而不使用明確的使用者驗證。 在應用程式權杖模型中，您的應用程式會管理使用者的驗證與授權。 然後，必要時，您的應用程式會建立

及傳送應用程式權杖，以告知我們的服務轉譯要求的報表。 雖然您可以使用 Azure AD 來進行使用者驗證與授權，但這項設計不需要您的應用程式這樣做。 您可以在 [這裡](app-token-flow.md)深入了解應用程式權杖。 我們也針對進階安全性篩選案例，在 Power BI 工作區集合中引進了資料列層級安全性功能 (RLS)。

## <a name="what-data-sources-are-currently-supported-with-power-bi-workspace-collections"></a>Power BI 工作區集合目前支援哪些資料來源？
我們將支援透過「直接查詢」使用基本認證來存取雲端資料來源。 這表示目前支援像是 Azure SQL DB 和 Azure SQL DW 這樣的來源。 我們會在未來幾個月內新增對其他資料來源和存取類型的支援。 如需詳細資訊，請參閱[連線至資料來源](connect-datasource.md)。

## <a name="how-does-the-tenancy-model-work-for-power-bi-workspace-collections"></a>租用模型如何與 Power BI 工作區集合搭配使用？
在 Power BI 工作區集合模型中，並未明確要求您的客戶必須是 Azure AD 租用戶。 您可以選擇是否要為您的客戶要求 Azure AD。 因此，您應用程式的架構和基礎結構將決定 Power BI 工作區集合所需的租用模型。

建置應用程式的開發人員/員工需要有 AAD 使用者帳戶，才能透過 Azure 入口網站管理您的 Azure 訂用帳戶和工作區集合。 可讓開發人員匯入報告、修改連接字串，以及讓內嵌 URL 改為使用應用程式權杖進行驗證，因此不需要 AAD 的程式設計 API。

## <a name="where-can-i-learn-more"></a>哪裡可以深入了解？
您可以造訪 [Power BI 工作區集合文件頁面](get-started.md)。 您可以透過造訪 [Power BI 部落格](https://powerbi.microsoft.com/blog/) \(英文\) 或造訪 Power BI 開發人員中心 (dev.powerbi.com)，來取得此服務的相關最新資訊。 您也可以在 [Stackoverflow](http://stackoverflow.com/questions/tagged/powerbi)提出問題。

## <a name="how-do-i-get-started"></a>如何開始使用？
您現在就可以免費開始使用！ 如果您有 Azure 訂用帳戶，現在就可以直接從 Azure 入口網站佈建 Power BI 工作區集合。 您也可以建立[免費的 Azure 帳戶](https://azure.microsoft.com/free/)。 在您佈建 Power BI 工作區集合服務之後，就能輕鬆地直接使用 Power BI REST API，或使用 [GitHub](http://go.microsoft.com/fwlink/?LinkID=746472) \(英文\) 上提供的開發人員 SDK。 也有提供範例說明如何使用開發人員 SDK。

## <a name="see-also"></a>另請參閱

[什麼是 Microsoft Power BI 工作區集合](what-are-power-bi-workspace-collections.md)
[開始使用 Microsoft Power BI 工作區集合](get-started.md)
[以範例作為起始](get-started-sample.md)   
[JavaScript 內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  

有其他疑問？ [試用 Power BI 社群](http://community.powerbi.com/)

