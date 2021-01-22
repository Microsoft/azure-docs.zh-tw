---
title: 'Machine learning 中的差異隱私權 (預覽版) '
titleSuffix: Azure Machine Learning
description: 瞭解何謂差異隱私權，以及如何實行可保留資料隱私權的微分私用系統。
author: luisquintanilla
ms.author: luquinta
ms.date: 01/21/2020
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: responsible-ml
ms.openlocfilehash: 39f4b1a7b9eb1ad7a87097240dd772e4f2dadf17
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98683529"
---
# <a name="what-is-differential-privacy-in-machine-learning-preview"></a>什麼是機器學習服務中的差異隱私權 (預覽版) 

深入瞭解機器學習服務中的差異隱私權，以及其運作方式。

由於組織收集並用於分析的資料量增加，隱私權和安全性也會受到重視。 分析必要資料。 一般來說，用來定型模型的資料越多，結果就越精確。 當使用個人資訊進行這些分析時，資料在整個使用過程中都必須保持隱私。

## <a name="how-differential-privacy-works"></a>差異隱私如何運作

差異隱私權是一組系統和實務，可協助保護個人資料的安全和隱私。

> [!div class="mx-imgBorder"]
> ![差異隱私權機器學習程式](./media/concept-differential-privacy/differential-privacy-machine-learning.jpg)

在傳統的案例中，未經處理資料會儲存在檔案和資料庫中。 當使用者分析資料時，通常會使用未經處理資料。 如此可能會因為侵害個人的隱私權而產生問題。 差異隱私權會嘗試處理此問題，方法是在資料中新增「雜訊」或隨機性，讓使用者無法識別任何個別的資料點。 至少，這類系統會提供合理推諉。 因此，個人的隱私權會受到影響，但對資料的精確度有很大的影響。

在差異隱私系統中，資料是透過稱為 **查詢** 的要求來共用。 當使用者提交資料查詢時，稱為 **隱私權機制** 的作業會將雜訊新增至要求的資料。 隱私權機制會傳回「資料的近似值」，而不是原始資料。 此隱私權保留結果會出現在 **報告** 中。 報告是由兩個部分所組成：實際計算的資料，以及資料建立方式的描述。

## <a name="differential-privacy-metrics"></a>差異隱私權計量

差異隱私權會嘗試防止使用者產生無限數量的報告，避免最終顯示機密資料。 所謂的 **epsilon** 值會測量報告的雜訊或隱私程度。 Epsilon 與雜訊或隱私權具有反向關聯性。 Epsilon 越低，資料的雜訊 (和隱私) 就越多。

Epsilon 值為非負值。 低於 1 的值會提供完整的合理推諉。 大於 1 的任何專案都有更高的風險可能暴露實際資料。 當您實作差異隱私系統時，建議您產生具有介於 0 和 1 之 epsilon 值的報表。

另一個與 epsilon 直接關聯的值是 **delta**。 Delta 是報告並非完全私人之機率的量值。 差異越高，epsilon 就越高。 由於這些值是相互關聯的，因此更常使用 epsilon。

## <a name="limit-queries-with-a-privacy-budget"></a>限制具有隱私權預算的查詢

為確保允許多項查詢的系統隱私權，差異隱私權會定義速率限制。 此限制稱為 **隱私權預算**。 隱私權預算會防止資料透過多個查詢重新建立。 隱私權預算會配置一個 epsilon 額度，通常介於 1 到 3 之間，以限制重新識別的風險。 產生報告時，隱私權預算會追蹤個別報告的 epsilon 值，以及所有報告的彙總。 在隱私權預算用完或耗盡之後，使用者就無法再存取資料。 

## <a name="reliability-of-data"></a>資料可靠性

雖然我們的目標是保留隱私權，但在資料的可用性和可靠性方面也會有所取捨。 在資料分析中，可以將精確度視為取樣錯誤造成之不確定性的量值。 這種不確定性傾向於落在特定界限內。 差異隱私權觀點的 **精確度** 會改為測量資料的可靠性，這會受到隱私權機制帶來的不確定性所影響。 簡言之，較高層級的雜訊或隱私權會轉譯成具有較低的 epsilon、準確度和可靠性的資料。 

## <a name="open-source-differential-privacy-libraries"></a>開放原始碼差異隱私權程式庫

SmartNoise 是一個開放原始碼專案，其中包含用來建立全球微分私用系統的不同元件。 SmartNoise 是由下列最上層元件所組成：

- SmartNoise 核心程式庫
- SmartNoise SDK 程式庫

### <a name="smartnoise-core"></a>SmartNoise 核心

核心程式庫包含下列用來實作差異隱私系統的隱私權機制：

|元件  |描述  |
|---------|---------|
|分析     | 任意計算的圖表描述。 |
|驗證程式     | Rust 程式庫包含一組工具，可用來檢查及衍生要進行差異隱私之分析的必要條件。          |
|執行階段     | 執行分析的媒體。 參考執行階段是以 Rust 撰寫，但執行階段可以根據您的資料需求，使用任何計算架構 (例如 SQL 和 Spark) 來撰寫。        |
|繫結     | 用來建置分析的語言繫結和協助程式程式庫。 目前 SmartNoise 會提供 Python 系結。 |

### <a name="smartnoise-sdk"></a>SmartNoise SDK

系統程式庫提供下列工具和服務，以使用表格式和關聯式資料：

|元件  |描述  |
|---------|---------|
|資料存取     | 可攔截及處理 SQL 查詢並產生報告的程式庫。 此程式庫是在 Python 中實作，並支援下列 ODBC 和 DBAPI 資料來源：<ul><li>PostgreSQL</li><li>SQL Server</li><li>Spark</li><li>Preston</li><li>Pandas</li></ul>|
|服務     | 提供 REST 端點來處理共用資料來源之要求或查詢的執行服務。 此服務的設計目的，是為了允許撰寫差異隱私權模組，以在包含不同差異和 epsilon 值 (也稱為「異質要求」) 的要求上運作。 此參考實作會因為相互關聯資料的查詢產生額外的影響。 |
|評估工具     | 隨機評估工具，可檢查隱私權違規、正確性和偏差。 評估工具支援下列測試： <ul><li>隱私權測試 - 判斷報告是否符合差異隱私權的條件。</li><li>正確性測試 - 測量報告的可靠性是否落在指定 95% 信賴等級的上下限內。</li><li>公用程式測試 - 判斷報告的信賴界限是否足夠接近資料，同時還能將維持最高的隱私權。</li><li>偏差測試 - 測量報告的分佈以進行重複的查詢，確保不會產生不平衡的情況</li></ul> |

## <a name="next-steps"></a>後續步驟

如何在 Azure Machine Learning 中[建立微分私用系統](how-to-differential-privacy.md)。

若要深入瞭解 SmartNoise 的元件，請參閱適用于 [SmartNoise Core](https://github.com/opendifferentialprivacy/smartnoise-core)、 [SmartNoise SDK](https://github.com/opendifferentialprivacy/smartnoise-sdk)和 [SmartNoise 範例](https://github.com/opendifferentialprivacy/smartnoise-samples)的 GitHub 存放庫。