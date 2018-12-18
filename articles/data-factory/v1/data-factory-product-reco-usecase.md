---
title: Data Factory 使用案例 - 產品建議
description: 了解使用 Azure Data Factory 以及其他服務所實作的使用案例。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 6f1523c7-46c3-4b8d-9ed6-b847ae5ec4ae
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 3328a7c80a05a48b3c04ad5492a7eafe62059681
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34622460"
---
# <a name="use-case---product-recommendations"></a>使用案例 - 產品建議
Azure Data Factory 是許多服務之一，可用來實作解決方案加速器的 Cortana Intelligence 套件。  請參閱 [Cortana Intelligence 套件](http://www.microsoft.com/cortanaanalytics) 頁面以了解這個套件的詳細資料。 在本文中，我們描述 Azure 使用者已經使用 Azure Data Factory 和其他 Cortana Intelligence 元件服務解決與實作的常見使用案例。

## <a name="scenario"></a>案例
線上零售商通常會想要向客戶呈現他們最有興趣、也因此最可能購買的產品，藉以誘使客戶購買產品。 若要達成此目的，線上零售商需要使用該特定使用者的個人化產品建議來自訂使用者的線上體驗。 這些個人化建議將會依據其目前和過去的購物行為資料、產品資訊、新引進的品牌以及產品和客戶區隔資料來提出。  此外，它們可以根據所有使用者的結合整體使用行為分析，提供使用者產品建議。

這些零售商的目標是要最佳化使用者的點選-銷售轉換並獲得更高的銷售收益。  他們透過根據客戶興趣和動作提供關聯式、以行為為基礎的產品建議來達成此轉換。 對於此使用案例，我們使用線上零售商做為想要最佳化客戶的企業範例。 但是這些原則適用於所有想要建立客戶及產品和服務之關聯，並利用個人化產品建議強化其客戶購買體驗的所有企業。

## <a name="challenges"></a>挑戰
在嘗試實作這種類型的使用案例時，線上零售商面臨許多挑戰。 

首先，必須從多個資料來源 (包括內部部署和雲端) 擷取不同大小和形狀的資料。 此資料包括使用者瀏覽線上零售網站時的產品資料、過去的客戶行為資料和使用者資料。 

第二，必須合理及準確計算和預測個人化的產品建議。 除了產品、品牌和客戶行為及瀏覽器資料外，線上零售商也需要包含客戶關於過往購買的意見反應，藉以考量決定使用者的最佳產品建議。 

第三，這些建議必須可立即提供給使用者，以提供順暢的瀏覽和購買體驗，並提供最新且最有關聯的建議。 

最後，零售商需要追蹤整體的向上銷售及交叉銷售點擊轉換銷售成功等情況，以測量其方法的效率並調整未來的建議。

## <a name="solution-overview"></a>解決方案概觀
此範例使用案例已被實際的 Azure 使用者透過使用 Azure Data Factory 及其他 Cortana Intelligence 元件服務 (包括 [HDInsight](https://azure.microsoft.com/services/hdinsight/) 和 [Power BI](https://powerbi.microsoft.com/)) 來解決並實作。

線上零售商使用 Azure Blob 存放區、內部部署 SQL Server、Azure SQL DB 和關聯式資料市集做為整個工作流程的資料儲存體選項。  Blob 存放區包含客戶資訊、客戶行為資料和產品資訊資料。 產品資訊資料包含 SQL 資料倉儲中的產品品牌資訊和產品目錄預存內部部署。 

所有資料都會組合並提供給產品建議系統，使用者在網站上瀏覽目錄中的產品時，可根據客戶興趣和動作提供個人化建議。 根據和所有使用者無關的整體網站使用模式，客戶也會看到與其正在查看之產品相關的產品。

![使用案例圖](./media/data-factory-product-reco-usecase/diagram-1.png)

線上零售商網站每天都會產生數 GB 的未經處理 web 記錄檔，做為半結構化的檔案。 未經處理的 web 記錄檔和客戶及產品目錄資訊會定期擷取到 Azure Blob 儲存體，使用 Data Factory 的全域部署資料移動做為服務。 一天的未經處理記錄檔會在 Blob 儲存體中分割 (根據年和月) 以進行長期儲存。  [Azure HDInsight](https://azure.microsoft.com/services/hdinsight/) 可用來分割 Blob 存放區中的未經處理記錄檔，並使用 Hive 和 Pig 指令碼大規模處理擷取的記錄檔。 接著會處理分割的 Web 記錄檔資料，以擷取需要的輸入，讓機器學習服務建議系統產生個人化的產品建議。

在此範例中，用於機器學習的建議系統是來自 [Apache Mahout](http://mahout.apache.org/)的開放原始碼機器學習建議平台。  任何 [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) 或自訂模型皆可套用至案例。  Mahout 模型可用來根據整體使用模式預測網站上項目之間的相似性，並根據個別使用者產生個人化的建議。

最後，個人化產品建議的結果集會移到關聯式資料市集供零售商網站使用。  結果集也可由另一個應用程式直接從 Blob 儲存體存取，或移至其他存放區供其他客戶及使用案例使用。

## <a name="benefits"></a>優點
藉由最佳化其產品建議策略並與業務目標對齊，解決方案可符合線上零售商的商務和行銷目標。 此外，他們能夠以有效率、可靠且符合成本效益的方式作業化和管理產品建議工作流程。 此方法讓他們能夠輕鬆地根據銷售點擊轉換成功的測量結果更新其模型並微調其有效性。 藉由使用 Azure Data Factory，他們可以放棄其耗時且昂貴的手動雲端資源管理，移至隨選雲端資源管理。 因此，他們能夠節省時間和金錢並減少解決方案部署的時間。 資料歷程檢視和作業服務健全狀況變得容易視覺化，且 Azure 入口網站也提供利用直覺式 Data Factory 監視和管理 UI 進行的疑難排解。 其解決方案可以立即排程與管理，如此可以可靠地產生完成的資料並將其傳送給使用者，並可自動管理資料和處理相依性而不需人力介入。

藉由提供此個人化的購物體驗，線上零售商可建立更具競爭力、更吸引人的客戶體驗，並因此增加銷售量和整體的客戶滿意度。

