---
title: Azure Application Insights 使用世代 | Microsoft Docs
description: 分析具有共通點的一組使用者、工作階段、事件或作業
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 04/10/2018
ms.reviewer: daviste
ms.author: mbullwin
ms.openlocfilehash: 30ef00fa1bfd2933df5e5c966cb8266d7c598a47
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162163"
---
# <a name="application-insights-cohorts"></a>Application Insights 世代

世代是指具有共通點的一組使用者、工作階段、事件或作業。 在 Azure Application Insights 中，世代會由分析查詢所定義。 如果您必須重複分析一組特定的使用者或事件，世代將可讓您有更多彈性來精準呈現您感興趣的集合。

![世代窗格](.\media\app-insights-usage-cohorts\001.png)

## <a name="cohorts-versus-basic-filters"></a>世代與基本篩選

世代的使用方式類似於篩選器。 但世代的定義是根據自訂分析查詢來建置，因此更具彈性且複雜。 不同於篩選，您可以儲存世代，讓其他小組成員可重複加以使用。

您可以為所有在您應用程式中嘗試使用新功能的使用者定義世代。 您可以將此世代儲存在 Application Insights 資源中。 未來您可以輕鬆地分析這個已儲存的特定使用者群組。

> [!NOTE]
> 世代一經建立後，即可供使用者、工作階段、事件和使用者流程等工具使用。

## <a name="example-engaged-users"></a>範例：參與的使用者

您的小組將參與的使用者定義為在指定月份內使用過您的應用程式五次或更多次的任何人。 在本節中，我們將為這些參與的使用者定義世代。

1. 開啟 [世代] 工具。

2. 選取 [範本庫] 索引標籤。您會看到各種世代的範本集合。

3. 選取 [參與的使用者 -- 依使用天數]。

    此世代有三個參數：
    * **Activities**，可讓您選擇哪些事件和網頁檢視應計為「使用」。
    * **Period**，月份的定義。
    * **UsedAtleastCustom**，為計入為參與的使用者，使用者必須在一段期間內使用某項目的次數。

4. 將 **UsedAtleastCustom** 變更為「**5 天以上**」，並將 [期間] 保留為預設的 28 天。

    ![參與的使用者](.\media\app-insights-usage-cohorts\003.png)

    現在，此世代代表在過去 28 天的五個不同日期中，透過任何自訂事件或網頁檢視傳送的所有使用者識別碼。

5. 選取 [ **儲存**]。

   > [!TIP]
   >  指定您的世代名稱，例如「參與的使用者 (5 天以上)」。 根據您是否要讓有權存取此 Appication Insights 資源的其他人看見此世代，將它儲存到「我的報告」或「共用報告」。

6. 選取 [回到範本庫]。

### <a name="what-can-you-do-by-using-this-cohort"></a>此世代有何功能？

開啟 [使用者] 工具。 在 [顯示] 下拉式方塊中，選擇您已根據**使用者所屬範圍**建立的世代。

此時，「使用者」工具已篩選出下列使用者世代：

![篩選出特定世代的使用者窗格](.\media\app-insights-usage-cohorts\004.png)

有一些重點值得注意：
* 您無法透過一般篩選來建立此集合。 日期邏輯更為進階。
* 您可以使用「使用者」工具中的一般篩選條件進一步篩選此世代。 因此，雖然此世代是以 28 天的時間範圍定義的，您仍可在「使用者」工具中將時間範圍調整為 30、60 或 90 天。

這些篩選可支援更複雜的問題，也就是那些無法透過查詢產生器陳述的問題。 例如，_在過去 28 天中參與的人。這些相同人員在過去 60 天中的行為是如何？_

## <a name="example-events-cohort"></a>範例：事件世代

您也可以建立事件的世代。 在本節中，您會為事件和網頁檢視定義世代。 然後，您會從其他工具了解如何使用這些項目。 此世代可定義一組小組視為_有效使用_的事件，或一組與特定新功能有關的事件。

1. 開啟 [世代] 工具。

2. 選取 [範本庫] 索引標籤。您會看到各種世代的範本集合。

3. 選取 [事件選擇器]。

    ![事件選擇器的螢幕擷取畫面](.\media\app-insights-usage-cohorts\006.png)

4. 在 [活動] 下拉式方塊中，選取要納入世代中的事件。

5. 儲存世代，並為它命名。

## <a name="example-active-users-where-you-modify-a-query"></a>範例：您為其修改查詢的有效使用者

先前的兩個世代是使用下拉式方塊定義的。 但您也可以使用分析查詢來定義世代，以達到整體彈性。 可藉由建立英國的使用者世代，來了解其具體方式。

![逐步使用世代工具的動畫影像](.\media\app-insights-usage-cohorts\cohorts0001.gif)

1. 開啟 [世代] 工具，選取 [範本庫] 索引標籤，然後選取 [空白使用者世代]。

    ![空白使用者世代](.\media\app-insights-usage-cohorts\001.png)

    其中包含三個區段：
    * Markdown 文字區段；您可以在其中為其他小組成員詳細說明世代。

    * 參數區段，可讓您自行設定參數，例如**活動**，以及上述兩個範例中的其他下拉式方塊。

    * 查詢區段，可讓您使用分析查詢定義世代。

    在查詢區段中，您可以[撰寫分析查詢](/azure/kusto/query)。 查詢會選取特定的幾列來說明您要定義的世代。 然後，「世代」工具會隱含地在查詢中加入 “| summarize by user_Id” 子句。 您可以在查詢下方的資料表中預覽此資料，以確定您的查詢會傳回結果。

    > [!NOTE]
    > 如果未看見查詢，請嘗試調高區段的高度，以呈現查詢。 此區段的開頭會以動畫 .gif 說明調整大小的行為。

2. 複製以下文字並貼到查詢編輯器中：

    ```KQL
    union customEvents, pageViews
    | where client_CountryOrRegion == "United Kingdom"
    ```

3. 選取 [執行查詢]。 如果您沒有看到使用者識別碼出現在資料表中，請變更為其中應用程式有使用者的國家/地區。

4. 儲存世代並為其命名。

## <a name="frequently-asked-questions"></a>常見問題集

_我已從特定國家/地區定義使用者世代。當我將「使用者」工具中的此世代，與直接設定該國家/地區的篩選條件進行比較時，我看到不同的結果。原因為何？_

世代與篩選並不相同。 假設您有來自英國的使用者世代 (如前面範例所定義)，並將其結果與設定「國家或地區 = 英國」篩選條件相比較。

* 當使用者在目前時間範圍內從英國傳送了一個或多個事件時，世代版本會顯示這些使用者的所有事件。 如果您依國家或區域劃分，您可能會看到許多國家和區域。
* 篩選版本只會顯示來自英國的事件。 但如果您依國家或區域劃分，您將只會看到英國。

## <a name="learn-more"></a>深入了解
- [分析查詢語言](https://go.microsoft.com/fwlink/?linkid=856587)
- [使用者、工作階段、事件](app-insights-usage-segmentation.md)
- [使用者流程](app-insights-usage-flows.md)
- [使用量概觀](app-insights-usage-overview.md)