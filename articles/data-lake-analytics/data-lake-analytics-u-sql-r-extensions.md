---
title: 在 Azure Data Lake Analytics 中使用 R 擴充 U-SQL 指令碼
description: 了解如何使用 Azure Data Lake Analytics 以 U-SQL 指令碼執行 R 程式碼
services: data-lake-analytics
ms.service: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: c1c74e5e-3e4a-41ab-9e3f-e9085da1d315
ms.topic: conceptual
ms.date: 06/20/2017
ms.openlocfilehash: b7e6c4911081d3b83ed99ab7316cb6fd810a0d60
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048720"
---
# <a name="extend-u-sql-scripts-with-r-code-in-azure-data-lake-analytics"></a>在 Azure Data Lake Analytics 中使用 R 程式碼擴充 U-SQL 指令碼

以下範例說明部署 R 程式碼的基本步驟：
* 使用 `REFERENCE ASSEMBLY` 陳述式啟用 U-SQL 指令碼的 R 延伸模組。
* 使用 ` REDUCE` 作業分割索引鍵上的輸入資料。
* U-SQL 的 R 延伸模組有內建的歸納器 (`Extension.R.Reducer`)，會在指派給歸納器的每一個頂點上執行 R 程式碼。 
* 專用具名資料框架的使用情況會相對應地呼叫 `inputFromUSQL` 和 `outputToUSQL ` 以在 USQL 和 R 之間傳遞資料。輸入和輸出資料框架識別碼名稱是固定的 (也就是使用者無法變更輸入和輸出資料框架識別碼的這些預先定義名稱)。

## <a name="embedding-r-code-in-the-u-sql-script"></a>將 R 程式碼內嵌在 U-SQL 指令碼中

您可以使用 `Extension.R.Reducer` 的 command 參數，將 R 程式碼內嵌在 U-SQL 指令碼中。 例如，您可以將 R 指令碼宣告為字串變數，再當作參數傳給歸納器。


    REFERENCE ASSEMBLY [ExtR];
    
    DECLARE @myRScript = @"
    inputFromUSQL$Species = as.factor(inputFromUSQL$Species)
    lm.fit=lm(unclass(Species)~.-Par, data=inputFromUSQL)
    #do not return readonly columns and make sure that the column names are the same in usql and r scripts,
    outputToUSQL=data.frame(summary(lm.fit)$coefficients)
    colnames(outputToUSQL) <- c(""Estimate"", ""StdError"", ""tValue"", ""Pr"")
    outputToUSQL
    ";
    
    @RScriptOutput = REDUCE … USING new Extension.R.Reducer(command:@myRScript, rReturnType:"dataframe");

## <a name="keep-the-r-code-in-a-separate-file-and-reference-it--the-u-sql-script"></a>將 R 程式碼保存在個別檔案中，然後在 U-SQL 指令碼中參考它

下列範例說明更複雜的使用方式。 在此案例中，R 程式碼會部署成「資源」(也就是 U-SQL 指令碼)。

將此 R 程式碼儲存為個別的檔案。

    load("my_model_LM_Iris.rda")
    outputToUSQL=data.frame(predict(lm.fit, inputFromUSQL, interval="confidence")) 

透過 DEPLOY RESOURCE 陳述式，使用 U-SQL 指令碼來部署該 R 指令碼。

    REFERENCE ASSEMBLY [ExtR];

    DEPLOY RESOURCE @"/usqlext/samples/R/RinUSQL_PredictUsingLinearModelasDF.R";
    DEPLOY RESOURCE @"/usqlext/samples/R/my_model_LM_Iris.rda";
    DECLARE @IrisData string = @"/usqlext/samples/R/iris.csv";
    DECLARE @OutputFilePredictions string = @"/my/R/Output/LMPredictionsIris.txt";
    DECLARE @PartitionCount int = 10;

    @InputData =
        EXTRACT 
            SepalLength double,
            SepalWidth double,
            PetalLength double,
            PetalWidth double,
            Species string
        FROM @IrisData
        USING Extractors.Csv();

    @ExtendedData =
        SELECT 
            Extension.R.RandomNumberGenerator.GetRandomNumber(@PartitionCount) AS Par,
            SepalLength,
            SepalWidth,
            PetalLength,
            PetalWidth
        FROM @InputData;

    // Predict Species

    @RScriptOutput = REDUCE @ExtendedData ON Par
        PRODUCE Par, fit double, lwr double, upr double
        READONLY Par
        USING new Extension.R.Reducer(scriptFile:"RinUSQL_PredictUsingLinearModelasDF.R", rReturnType:"dataframe", stringsAsFactors:false);
        OUTPUT @RScriptOutput TO @OutputFilePredictions USING Outputters.Tsv();

## <a name="how-r-integrates-with-u-sql"></a>R 如何與 U-SQL 整合

### <a name="datatypes"></a>資料類型
* 來自 U-SQL 的字串和數值資料行會在 R 資料框架和 U-SQL 之間以現狀進行轉換 [支援的類型：`double``string`、`bool`、`integer`、`byte`]。
* U-SQL 不支援 `Factor` 資料類型。
* `byte[]` 必須序列化為 base64 編碼的 `string`。
* 當 U-SQL 建立 R 輸入資料框架，或是設定歸納器參數 `stringsAsFactors: true` 之後，U-SQL 字串便可以轉換為 R 程式碼的因素。

### <a name="schemas"></a>結構描述
* U-SQL 資料集不能有重複的資料行名稱。
* U-SQL 資料集的資料行名稱必須為字串。
* 資料行名稱在 U-SQL 和 R 指令碼中皆必須為相同。
* 唯讀資料行不能是輸出資料框架的一部分。 因為如果唯讀資料行是 UDO 輸出結構描述的一部分，便會自動插入回 U-SQL 資料表。

### <a name="functional-limitations"></a>功能限制：
* R 引擎無法在相同的程序中具現化兩次。 
* 目前，在以歸納器 UDO 所產生的分割模型進行預測時，U-SQL 不支援使用合併器 UDO。 使用者可以將分割的模型宣告為資源，然後用於 R 指令碼中 (請參閱範例程式碼 `ExtR_PredictUsingLMRawStringReducer.usql`)

### <a name="r-versions"></a>R 版本
僅支援 R 3.2.2。

### <a name="standard-r-modules"></a>標準的 R 模組

    base
    boot
    Class
    Cluster
    codetools
    compiler
    datasets
    doParallel
    doRSR
    foreach
    foreign
    Graphics
    grDevices
    grid
    iterators
    KernSmooth
    lattice
    MASS
    Matrix
    Methods
    mgcv
    nlme
    Nnet
    Parallel
    pkgXMLBuilder
    RevoIOQ
    revoIpe
    RevoMods
    RevoPemaR
    RevoRpeConnector
    RevoRsrConnector
    RevoScaleR
    RevoTreeView
    RevoUtils
    RevoUtilsMath
    Rpart
    RUnit
    spatial
    splines
    Stats
    stats4
    survival
    Tcltk
    Tools
    translations
    utils
    XML

### <a name="input-and-output-size-limitations"></a>輸入和輸出的大小限制
指派給每個頂點的記憶體數量皆有上限。 因為輸入和輸出資料框架必須存在於 R 程式碼的記憶體中，輸入和輸出的大小總和不能超過 500 GB。

### <a name="sample-code"></a>範例程式碼
安裝 U-SQL 進階分析擴充功能之後，您的 Data Lake Store 帳戶中會有更多範例程式碼可用。 其他範例程式碼的路徑如下：`<your_account_address>/usqlext/samples/R`。 

## <a name="deploying-custom-r-modules-with-u-sql"></a>使用 U-SQL 部署自訂 R 模組

首先，建立自訂 R 模組並壓縮它，然後將已壓縮的 R 自訂模組檔案上傳至您的 ADL 存放區。 在範例中，我們會將 magittr_1.5.zip 上傳至我們所使用之 ADLA 帳戶的預設 ADLA 帳戶根目錄。 將模組上傳至 ADL 存放區之後，將它宣告為使用 DEPLOY RESOURCE 以供您的 U-SQL 指令碼使用，然後呼叫 `install.packages` 來安裝它。

    REFERENCE ASSEMBLY [ExtR];
    DEPLOY RESOURCE @"/magrittr_1.5.zip";

    DECLARE @IrisData string =  @"/usqlext/samples/R/iris.csv";
    DECLARE @OutputFileModelSummary string = @"/R/Output/CustomePackages.txt";

    // R script to run
    DECLARE @myRScript = @"
    # install the magrittr package,
    install.packages('magrittr_1.5.zip', repos = NULL),
    # load the magrittr package,
    require(magrittr),
    # demonstrate use of the magrittr package,
    2 %>% sqrt
    ";

    @InputData =
    EXTRACT SepalLength double,
    SepalWidth double,
    PetalLength double,
    PetalWidth double,
    Species string
    FROM @IrisData
    USING Extractors.Csv();

    @ExtendedData =
    SELECT 0 AS Par,
    *
    FROM @InputData;

    @RScriptOutput = REDUCE @ExtendedData ON Par
    PRODUCE Par, RowId int, ROutput string
    READONLY Par
    USING new Extension.R.Reducer(command:@myRScript, rReturnType:"charactermatrix");

    OUTPUT @RScriptOutput TO @OutputFileModelSummary USING Outputters.Tsv();

## <a name="next-steps"></a>後續步驟
* [Microsoft Azure Data Lake Analytics 概觀](data-lake-analytics-overview.md)
* [使用 Data Lake Tools for Visual Studio 開發 U-SQL 指令碼](data-lake-analytics-data-lake-tools-get-started.md)
* [針對 Azure 資料湖分析工作使用 U-SQL 視窗函式](data-lake-analytics-use-window-functions.md)
