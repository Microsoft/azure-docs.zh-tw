---
title: 適用於 Azure Machine Learning 的 Azure 資料來源精靈 | Microsoft Docs
description: 說明 AML Workbench 的資料來源精靈
services: machine-learning
author: cforbe
ms.author: cforbe
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/07/2017
ROBOTS: NOINDEX
ms.openlocfilehash: a3aeb021d5d3726a31b62d66efc44dc29f3dded0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46959643"
---
# <a name="data-source-wizard"></a>資料來源精靈 #

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



資料來源精靈是一種快速且簡單的方式，不需程式碼，就能將資料集帶入 Azure ML Workbench。 您也可以在其中選取資料集適用的取樣策略，以及每個資料行的資料類型。 

## <a name="step-1-trigger-the-data-source-wizard"></a>步驟 1：觸發資料來源精靈 ## 

使用資料來源精靈，將資料帶入專案。 選取資料檢視中搜尋方塊旁邊的 [+] 按鈕，然後選擇 [新增資料來源]。 

![新增資料來源](media/data-source-wizard/add-data-source.png)

## <a name="step-2-select-where-data-is-stored"></a>步驟 2：選取資料的儲存位置 ##
首先，指定資料目前存在的方式。 它可以儲存於一般檔案或目錄、parquet 檔案、Excel 檔案或資料庫中。 如需詳細資訊，請參閱[支援的資料來源](data-prep-appendix2-supported-data-sources.md)。

![步驟 1](media/data-source-wizard/step1.png)

## <a name="step-3-select-data-file"></a>步驟 3：選取資料檔案 ##
針對檔案/目錄，指定檔案路徑。 從下拉式清單中選擇資料的位置，它可能是本機檔案路徑或 Azure Blob 儲存體。 

使用下列方法指定路徑：輸入路徑或按一下 [瀏覽...] 按鈕來瀏覽該路徑。 您可以瀏覽目錄或是一或多個檔案。

按一下 [完成] 以接受其餘步驟的預設值，或 [下一步] 繼續進行下一個步驟。


![步驟 4](media/data-source-wizard/step2.png)

## <a name="step-4-choose-file-parameters"></a>步驟 4：選擇檔案參數 ##

[資料來源精靈] 可以自動偵測檔案類型、分隔符號和編碼方式。 它也會顯示資料的外觀範例。 您也可以手動變更這其中任何參數。 

![步驟 5](media/data-source-wizard/step3.png)

## <a name="step-5-set-data-types-for-columns"></a>步驟 5：設定資料行的資料類型 ##

[資料來源精靈] 會自動偵測資料集資料行的資料類型。 如果它遺漏其中一個，或者您想要強制執行某個資料類型，您可以手動變更資料類型。 [樣本輸出資料] 資料行會顯示資料的外觀範例。

對於「資料準備」推斷會包含日期的資料行，系統可能會提示您選取日期格式中的月份和日期順序。 例如，1/2/2013 可能代表年 1 月 2 日 (為此，請選取 [日-月]) 或 2 月 1 日 (請選取 [月-日])。

![步驟 6](media/data-source-wizard/step4.png)

## <a name="step-6-choose-sampling-strategy-for-data"></a>步驟 6：選擇資料的取樣策略 ##

您可以為資料集指定一或多個取樣策略，並選擇一個作為使用中的策略。 預設值是載入前 10000 個資料列。 您可以按一下工具列中的 [編輯] 按鈕或按一下 [+新增] 來新增策略，藉以編輯此樣本。 以下是目前支援的策略

-     前幾個資料列數目
-     隨機的資料列數目
-     隨機的資料列百分比
-     完整檔案

您可以視需要指定任意個取樣策略，但在準備資料時，只能將一個設定為使用中。 您可以藉由選取任一個策略，然後按一下工具列中的 [設定為使用中]，將該策略設定為使用中。

根據資料的來源處而定，可能不支援某些取樣策略。 如需取樣的詳細資訊，請參閱[這份文件](data-prep-user-guide.md)中關於取樣的小節。 

![步驟 6](media/data-source-wizard/step5.png)

## <a name="step-7-path-column-handling"></a>步驟 7：路徑資料行處理 ##

如果檔案路徑包含重要資料，您可以選擇包含它以作為資料集中的第一個資料行。 如果您想要帶入多個檔案，此選項非常有用。 否則，您可以選擇不包含它。

![步驟 7](media/data-source-wizard/step6.png)

按一下 [完成] 之後，即會將新的資料來源新增至專案。 您可以在資料檢視的 [資料來源] 群組中找到它，或作為 [檔案檢視] 中的 .dsource 檔案。
