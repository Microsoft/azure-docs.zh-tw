---
title: 如何探索 Azure 資料目錄中的資料資產
description: 本文著重在說明如何探索已註冊 Azure 資料目錄的資料資產，包括搜尋、篩選 Azure 資料目錄入口網站，以及使用其結果醒目提示功能。
services: data-catalog
author: markingmyname
ms.author: maghan
ms.assetid: f72ae3a3-6573-4710-89a7-f13555e1968c
ms.service: data-catalog
ms.topic: conceptual
ms.date: 01/18/2018
ms.openlocfilehash: 11216640a4f4f78e0b0de850f32f0ff85bbc9ef2
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407054"
---
# <a name="how-to-discover-data-sources-in-azure-data-catalog"></a>如何探索 Azure 資料目錄中的資料資產
## <a name="introduction"></a>簡介
Azure 資料目錄是完全受控的雲端服務，可作為企業資料來源的註冊和探索系統。 換句話說，資料目錄可於協助人們探索、了解和使用資料來源，並可協助組織從現有的資料獲得更多價值。 在資料目錄註冊資料來源之後，其中繼資料會由此服務編製索引，讓您可以輕鬆地搜尋以探索所需的資料。

## <a name="searching-and-filtering"></a>搜尋和篩選
在資料目錄進行探索會使用兩種主要機制：搜尋和篩選。

搜尋的設計兼具直覺與強大的功能。 根據預設，搜尋字詞會比對目錄中的所有內容，包括使用者提供的註解。

篩選的設計則與搜尋互補。 您可以選取特定的特性，例如專家、資料來源類型、物件類型和標記。 您可以只檢視相符的資料資產，並將搜尋結果限制為相符的資產。

透過結合使用搜尋和篩選，您可以快速巡覽已經在資料目錄註冊的資料來源，以探索所需的資料來源。

## <a name="search-syntax"></a>搜尋語法
雖然預設的任意文字搜尋既簡單又直覺，但您也可以使用資料目錄搜尋語法，更充分地掌控搜尋結果。 資料目錄搜尋支援下列技巧：

| 技巧 | 使用 | 範例 |
| --- | --- | --- |
| 基本搜尋 |使用一或多個搜尋字詞的基本搜尋。 結果包含了任何屬性與一或多個指定字詞相符的所有資產。 |`sales data` |
| 屬性範圍 |只會傳回搜尋字詞與指定屬性相符的資料來源。 |`name:finance` |
| 布林運算子 |使用布林值作業擴大或縮小搜尋。 |`finance NOT corporate` |
| 使用括弧分組 |使用括弧將查詢部分分組，以達到邏輯隔離，尤其是與布林運算子結合時。 |`name:finance AND (tags:Q1 OR tags:Q2)` |
| 比較運算子 |針對具有數值和日期資料類型的屬性使用比較而非相等。 |`modifiedTime > "11/05/2014"` |

如需資料目錄搜尋的詳細資訊，請參閱 [Azure 資料目錄](https://msdn.microsoft.com/library/azure/mt267594.aspx)一文。

## <a name="hit-highlighting"></a>搜尋結果醒目提示
當您檢視搜尋結果時，會醒目顯示符合指定搜尋字詞的任何顯示屬性 (例如資料資產名稱、描述和標記)，讓您更輕鬆地找出指定搜尋傳回指定資料資產的原因。

> [!NOTE]
> 若要關閉結果醒目提示，請使用 [醒目提示] 在資料目錄入口網站中進行切換。
>
>

當您檢視搜尋結果時，即使已啟用醒目提示，不一定能明顯看出為何某個資料資產會包含在其中。 因為預設會搜尋所有屬性，某個資料資產可能因為符合資料行層級的屬性而被傳回。 此外，多位使用者可以使用他們自己的標記和描述為已註冊的資料資產加上註解，因此並非所有的中繼資料都會顯示在搜尋結果清單中。

在預設的並排顯示中，搜尋結果所顯示的每個磚，都含有**檢視搜尋字詞相符項目**圖示，讓您可以快速檢視相符項目的數目和位置，並在需要時移至該位置。

 ![在 Azure 資料目錄入口網站顯示結果醒目提示並搜尋相符項目](./media/data-catalog-how-to-discover/search-matches.png)

## <a name="summary"></a>總結
因為在資料目錄註冊資料來源，可讓您透過將結構化和描述性中繼資料從資料來源複製到目錄服務，所以更容易地探索及了解資料來源。 註冊資料來源之後，您可以使用篩選來探索資料來源，並且在資料目錄入口網站搜尋。

## <a name="next-steps"></a>後續步驟
* 如需如何探索資料來源的逐步詳細資料，請參閱[開始使用 Azure 資料目錄](data-catalog-get-started.md)。
