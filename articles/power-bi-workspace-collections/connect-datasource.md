---
title: 在 Power BI 工作區集合中連線到資料來源 | Microsoft Docs
description: 了解如何在 Power BI 工作區集合中連線到資料來源。
services: power-bi-embedded
author: markingmyname
ROBOTS: NOINDEX
ms.assetid: 2a4caeb3-255d-4215-9554-0ca8e3568c13
ms.service: power-bi-embedded
ms.topic: article
ms.workload: powerbi
ms.date: 09/20/2017
ms.author: maghan
ms.openlocfilehash: c626601d9eae7732779020b153c624f80605b56a
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43051870"
---
# <a name="connect-to-a-data-source"></a>連接到資料來源

使用 **Power BI 工作區集合**，就可將報表內嵌到您自己的應用程式。 當您將 Power BI 報表內嵌到應用程式時，報告會以**匯入**資料複本的方式連接至基礎資料，或使用 **DirectQuery****直接連接**到資料來源。

> [!IMPORTANT]
> Power BI 工作區集合已被取代，只能使用到 2018 年 6 月或您的合約所指出的時間。 建議您進行規劃以移轉至 Power BI Embedded，以免應用程式發生中斷。 如需如何將資料移轉至 Power BI Embedded 的資訊，請參閱[如何將 Power BI 工作區集合的內容移轉至 Power BI Embedded](https://powerbi.microsoft.com/documentation/powerbi-developer-migrate-from-powerbi-embedded/)。

以下是使用 **Import** 和 **DirectQuery** 之間的差異。

| Import | 直接連接 |
| --- | --- |
| 資料表、資料行 *和資料* 匯入或複製到報表的資料集中。 若要查看基礎資料發生的變更，您必須重新整理或再次匯入完整的目前資料集。 |只有 *資料表和資料行* 匯入或複製到報表的資料集中。 一律檢視最新的資料。 |

有了 Power BI 工作區集合，您就可以使用 DirectQuery 搭配雲端資料來源，但目前無法使用內部部署資料來源。

> [!NOTE]
> Power BI 工作區集合現階段不支援內部部署資料閘道。 這表示您無法在 DirectQuery 使用內部部署資料來源。

## <a name="supported-data-sources"></a>支援的資料來源

**DirectQuery**
* Azure SQL Database
* Azure SQL 資料倉儲

**Import**

您可以使用 Power BI Desktop 中所有的可用資料來源進行匯入。 您將**無法**在 Power BI 工作區集合中重新整理該資料。 您必須將 PBIX 檔案的變更上傳到 Power BI 工作區集合。 這是因為沒有閘道器可用。 

## <a name="benefits-of-using-directquery"></a>使用 DirectQuery 的優點

使用 **DirectQuery**有兩項主要優點：

* **DirectQuery** 可讓您對大型資料集建立視覺效果，否則無法第一次就匯入所有資料。
* 基礎資料變更需要重新整理資料，而某些報表如果要顯示目前的資料，還需要大量的資料傳輸，這都使得重新匯入資料不可行。 相較之下， **DirectQuery** 報表一律使用目前的資料。

## <a name="limitations-of-directquery"></a>DirectQuery 的限制

使用 **DirectQuery**有一些限制：

* 所有資料表必須全都來自單一資料庫。
* 如果查詢太複雜，就會發生錯誤。 若要補救錯誤，您必須重構以簡化查詢。 如果查詢必須很複雜，您就需要匯入資料，而不是使用 **DirectQuery**。
* 關聯性篩選限於單向，而非雙向。
* 您無法變更資料行的資料類型。
* 根據預設，限制是置於量值允許的 DAX 運算式中。 請參閱 [DirectQuery 和量值](#measures)。

<a name="measures"/>

## <a name="directquery-and-measures"></a>DirectQuery 和量值
為確保傳送至基礎資料來源的查詢都有可接受的效能，所以將限制加諸於量值之上。 使用 **Power BI Desktop** 時，進階使用者可以選擇 [檔案] > [選項和設定] > [選項]，選擇略過這項限制。 在 [選項] 對話方塊中選擇 [DirectQuery]，並選取選項 [允許在 DirectQuery 模式中量值不受限制]。 選取該選項後，即可使用對量值有效的任何 DAX 運算式。 不過，使用者必須知道，當資料匯入時，某些執行得很好的運算式在 **DirectQuery** 模式中可能會造成後端來源的查詢變慢。 

## <a name="see-also"></a>另請參閱

* [開始使用 Microsoft Power BI 工作區集合](get-started.md)
* [Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-get-the-desktop/)

有其他疑問？ [試用 Power BI 社群](http://community.powerbi.com/)

