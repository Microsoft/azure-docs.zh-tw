---
title: 在 Azure Machine Learning 中撰寫自訂 R 模組 | Microsoft Docs
description: 在 Azure Machine Learning 中撰寫自訂 R 模組的快速入門。
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: 6cbc628a-7e60-42ce-9f90-20aaea7ba630
ms.service: machine-learning
ms.component: studio
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 11/29/2017
ms.openlocfilehash: 555672df5b0b86858d460ff7606bc6ca23f4f103
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34834350"
---
# <a name="author-custom-r-modules-in-azure-machine-learning"></a>在 Azure Machine Learning 中撰寫自訂 R 模組
本主題描述如何在 Azure Machine Learning 中撰寫及部署自訂 R 模組。 它說明什麼是自訂 R 模組，以及使用哪些檔案定義這些模組； 並說明如何在 Machine Learning 工作區中建構這些用來定義模組的檔案，以及如何註冊模組以進行部署。 接著，詳細說明用於自訂模組定義中的元素和屬性。 此外，也討論如何使用輔助功能和檔案以及多個輸出。 

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="what-is-a-custom-r-module"></a>什麼是自訂 R 模組？
**自訂模組** 是一種使用者定義的模組，可上傳至您的工作區，並在 Azure Machine Learning 實驗時執行。 **自訂 R 模組** 是執行使用者定義之 R 函數的自訂模組。 **R** 是一種適用於統計運算和圖形的程式設計語言，由實作演算法的統計學家和資料科學家廣泛使用。 目前，R 是自訂模組中唯一支援的語言，但未來版本將新增更多語言的支援。

自訂模組在 Azure Machine Learning 中為 **第一等狀態** ，也就是說，它們的用法就像其他任何模組一樣。 它們可以與已發行實驗或視覺效果中包含的其他模組一起執行。 您可以控制模組實作的演算法、要使用的輸入與輸出連接埠、模型參數，以及其他多種執行階段行為。 包含自訂模組的實驗也可以發佈至 Azure AI 資源庫以輕鬆共用。

## <a name="files-in-a-custom-r-module"></a>自訂 R 模組中的檔案
自訂 R 模組是由至少包含兩個檔案的 .zip 檔案所定義：

* 一個實作這個模組所公開之 R 函數的 **原始程式檔**
* 一個描述自訂模組介面的 **XML 定義檔**

其他輔助檔案也可以包含在提供可從自訂模組存取的功能的 .zip 檔案中。 此選項會在快速入門範例之後的參考章節 **XML 定義檔中的元素**的**引數**部分討論。

## <a name="quickstart-example-define-package-and-register-a-custom-r-module"></a>快速入門範例：定義、封裝及註冊自訂 R 模組
這個範例說明如何建構自訂 R 模組所需的檔案、將檔案封裝成 zip 檔案，然後在您的 Machine Learning 工作區中註冊模組。 您可以從 [下載 CustomAddRows.zip 檔案](http://go.microsoft.com/fwlink/?LinkID=524916&clcid=0x409)下載範例 zip 封裝和範例檔案。

## <a name="the-source-file"></a>來源檔案
請考慮**自訂新增資料列**模組的範例，這個模組會修改從兩個資料集 (資料框架) 串連資料列 (觀察) 時所使用的**新增資料列**模組標準實作。 標準的**新增資料列**模組使用 `rbind` 演算法，將第二個輸入資料集的資料列附加到第一個輸入資料集的結尾。 自訂的 `CustomAddRows` 函式同樣會接受兩個資料集，但還會接受一個布林交換參數做為額外的輸入。 如果交換參數設為 **FALSE**，它會傳回與標準實作相同的資料集。 但是，如果交換參數為 **TRUE**，函式會將第一個輸入資料集的資料列改為附加到第二個資料集的結尾。 R `CustomAddRows` 函式由 **自訂新增資料列** 模組所公開，包含此函式的 CustomAddRows.R 檔案案包含下列 R 程式碼。

    CustomAddRows <- function(dataset1, dataset2, swap=FALSE) 
    {
        if (swap)
        {
            return (rbind(dataset2, dataset1));
        }
        else
        {
            return (rbind(dataset1, dataset2));
        } 
    } 

### <a name="the-xml-definition-file"></a>XML 定義檔
若要公開這個 `CustomAddRows` 函數做為 Azure Machine Learning 模組，必須建立 XML 定義檔以指定 **自訂 [加入資料列]** 模組的外觀及運作方式。 

    <!-- Defined a module using an R Script -->
    <Module name="Custom Add Rows">
        <Owner>Microsoft Corporation</Owner>
        <Description>Appends one dataset to another. Dataset 2 is concatenated to Dataset 1 when Swap is FALSE, and vice versa when Swap is TRUE.</Description>

    <!-- Specify the base language, script file and R function to use for this module. -->        
        <Language name="R" 
         sourceFile="CustomAddRows.R" 
         entryPoint="CustomAddRows" />  

    <!-- Define module input and output ports -->
    <!-- Note: The values of the id attributes in the Input and Arg elements must match the parameter names in the R Function CustomAddRows defined in CustomAddRows.R. -->
        <Ports>
            <Input id="dataset1" name="Dataset 1" type="DataTable">
                <Description>First input dataset</Description>
            </Input>
            <Input id="dataset2" name="Dataset 2" type="DataTable">
                <Description>Second input dataset</Description>
            </Input>
            <Output id="dataset" name="Dataset" type="DataTable">
                <Description>The combined dataset</Description>
            </Output>
        </Ports>

    <!-- Define module parameters -->
        <Arguments>
            <Arg id="swap" name="Swap" type="bool" >
                <Description>Swap input datasets.</Description>
            </Arg>
        </Arguments>
    </Module>


這一點很重要，請注意 XML 檔案中之 **Input** 和 **Arg** 元素的 **id** 屬性值必須完全符合在 CustomAddRows.R 檔案中 R 程式碼的函式參數名稱 (在本例中為 *dataset1*、*dataset2* 和 *swap*)。 同樣地，**Language** 元素的 **entryPoint** 屬性值必須完全符合 R 指令碼中的函式名稱 (在本例中為 *CustomAddRows*)。 

相反地，**Output** 元素的 **id** 屬性不會對應至 R 指令碼中的任何變數。 如果需要多個輸入，請直接從 R 函式傳回清單，其中包含的結果會依照 **Output** 元素在 XML 檔案中宣告的*相同順序*來排列。

### <a name="package-and-register-the-module"></a>封裝並註冊模組
將這兩個檔案另存為 *CustomAddRows.R* 和 *CustomAddRows.xml*，然後一起壓縮成 *CustomAddRows.zip* 檔案。

若要在 Machine Learning 工作區中註冊這兩個檔案，請移至 Machine Learning Studio 中的工作區，按一下底部的 [+ 新增] 按鈕，然後選擇 [模組] -> [從 ZIP 封裝]，以上傳新的**新增資料列**模組。

![上傳 Zip 檔案](./media/custom-r-modules/upload-from-zip-package.png)

[自訂新增資料列]  模組現在已經準備好，可供機器學習服務實驗存取。

## <a name="elements-in-the-xml-definition-file"></a>引數
### <a name="module-elements"></a>Module 元素
**Module** 元素可用來定義 XML 檔案中的自訂模組。 您可以在一個 XML 檔案中，使用多個 **Module** 元素來定義多個模組。 工作區中的每個模組都必須有唯一的名稱。 使用與現有自訂模組相同的名稱來註冊自訂模組，會以這個新模組取代現有的模組。 不過，您可以使用與現有 Azure Machine Learning 模組相同的名稱來註冊自訂模組。 如果您這麼做，它們會出現在模組選擇區的 [自訂]  類別中。

    <Module name="Custom Add Rows" isDeterministic="false"> 
        <Owner>Microsoft Corporation</Owner>
        <Description>Appends one dataset to another...</Description>/> 


在 **Module** 元素內，您可以指定兩個額外的選擇性元素︰

* **Owner** 元素，內嵌在模組中  
* **Description** 元素，是顯示在模組快速說明中的文字，也是當滑鼠停留在機器學習服務 UI 中的模組上時所顯示的文字。

Module 元素中的字元限制規則：

* **Module** 元素中的 **name** 屬性值長度不能超過 64 個字元。 
* **Description** 元素的內容長度不能超過 128 個字元。
* **Owner** 元素的內容長度不能超過 32 個字元。

模組的結果可能具決定性或不具決定性。** 依預設，所有模組都視為具決定性。 也就是說，如果提供一組不變的參數和資料，每次 RAND 或函式執行時模組都應該傳回相同的結果。 在這個行為下，只有當參數或輸入資料有所變更，Azure Machine Learning Studio 才會重新執行標示為具決定性的模組。 傳回快取的結果也會讓實驗的執行速度加快許多。

也有不具決定性的函式，例如 RAND 或傳回目前日期或時間的函式。 如果您的模組使用不具決定性的函式，您可以將選擇性屬性 **isDeterministic** 設為 **FALSE**，藉此方式指定模組是不具決定性。 這可確保每次執行實驗時，都會重新執行模組，即使模組輸入和參數未變更亦然。 

### <a name="language-definition"></a>語言定義
XML 定義檔中的 **Language** 元素可用來指定自訂模組的語言。 目前，R 是唯一支援的語言。 **sourceFile** 屬性值必須是包含執行模組時所呼叫之函數的 R 檔案名稱。 這個檔案必須是 zip 封裝的一部分。 **entryPoint** 屬性值是所要呼叫之函數的名稱，必須符合原始程式檔中定義的有效函數。

    <Language name="R" sourceFile="CustomAddRows.R" entryPoint="CustomAddRows" />


### <a name="ports"></a>連接埠
自訂模組的輸入和輸出連接埠是在 XML 定義檔之 **Ports** 區段的子元素中所指定。 這些元素的順序會決定使用者所經歷的配置 (UX)。 列於 XML 檔案之 **Ports** 元素中的第一個子 **input** 或 **output**，成為機器學習服務 UX 中最左側的輸入連接埠。
每個輸入和輸出連接埠可能會有一個選擇性 **Description** 子元素，以指定當滑鼠游標停留在機器學習服務 UI 中的連接埠上時所顯示的文字。

**連接埠規則**：

* **輸入和輸出連接埠** 數目上限各為 8 個。

### <a name="input-elements"></a>Input 元素
輸入連接埠可讓您將資料傳遞至 R 函式和工作區。 輸入連接埠支援的 **資料類型** 如下所示： 

**DataTable：** 這個類型會當做 data.frame 傳遞至 R 函數。 事實上，機器學習服務支援之所有與 **DataTable** 相容的類型 (例如 CSV 檔案或 ARFF 檔案)，都會自動轉換成 data.frame。 

        <Input id="dataset1" name="Input 1" type="DataTable" isOptional="false">
            <Description>Input Dataset 1</Description>
           </Input>

與每個 **DataTable** 輸入連接埠相關聯的 **id** 屬性都必須有唯一值，而且這個值必須符合其在 R 函數中的對應具名參數。
未當做實驗輸入傳遞的選擇性 **DataTable** 連接埠，會將 **NULL** 值傳遞至 R 函式，而且如果未連接輸入，則會忽略選擇性 zip 連接埠。 **isOptional** 屬性對於 **DataTable** 和 **Zip** 類型都是選擇性的，而且預設為 *false*。

**Zip：** 自訂模組可以接受 zip 檔案做為輸入。 這個輸入會解壓縮到您函數的 R 工作目錄中

        <Input id="zippedData" name="Zip Input" type="Zip" IsOptional="false">
            <Description>Zip files to be extracted to the R working directory.</Description>
           </Input>

對於自訂 R 模組， Zip 連接埠的 id 不一定要符合 R 函式的任何參數。 這是因為 zip 檔案會自動解壓縮到 R 工作目錄。

**輸入規則：**

* **Input** 元素的 **id** 屬性值必須是有效的 R 變數名稱。
* **Input** 元素的 **id** 屬性值長度不能超過 64 個字元。
* **Input** 元素的 **name** 屬性值長度不能超過 64 個字元。
* **Description** 元素的內容長度不能超過 128 個字元。
* **Input** 元素的 **type** 屬性值必須是 *Zip* 或 *DataTable*。
* 不需要 **Input** 元素的 **isOptional** 屬性值 (而且未指定時預設為 *false*)；如有指定，則必須為 *true* 或 *false*。

### <a name="output-elements"></a>Output 元素
**標準輸出連接埠：** 輸出連接埠會對應至 R 函數中的傳回值，後續模組可以接著使用這些值。 *DataTable* 是目前唯一支援的標準輸出連接埠類型。 (即將推出 *Learners* 和 *Transforms* 的支援。)*DataTable* 輸出的定義如下：

    <Output id="dataset" name="Dataset" type="DataTable">
        <Description>Combined dataset</Description>
    </Output>

對於自訂 R 模組中的輸出， **id** 屬性值不一定要對應至 R 指令碼中的任何項目，但它必須是唯一的。 對於單一模組輸出，R 函數中的傳回值必須是 *data.frame*。 若要輸出多個支援的資料類型的物件，必須在 XML 定義檔中指定適當的輸出連接埠，而且必須以清單形式傳回物件。 輸出物件會從左到右指派給輸出連接埠，以反映物件置於傳回清單中的順序。

例如，如果您要修改**自訂新增資料列**模組，使其除了新增的資料集 *dataset* 之外，還要輸出兩個原始資料集 *dataset1* 和 *dataset2* (依照從左到右的順序：*dataset*、*dataset1*、*dataset2*)，請在 CustomAddRows.xml 檔案中定義輸出連接埠，如下所示：

    <Ports> 
        <Output id="dataset" name="Dataset Out" type="DataTable"> 
            <Description>New Dataset</Description> 
        </Output> 
        <Output id="dataset1_out" name="Dataset 1 Out" type="DataTable"> 
            <Description>First Dataset</Description> 
        </Output> 
        <Output id="dataset2_out" name="Dataset 2 Out" type="DataTable"> 
            <Description>Second Dataset</Description> 
        </Output> 
        <Input id="dataset1" name="Dataset 1" type="DataTable"> 
            <Description>First Input Table</Description>
        </Input> 
        <Input id="dataset2" name="Dataset 2" type="DataTable"> 
            <Description>Second Input Table</Description> 
        </Input> 
    </Ports> 


然後在清單中，以 ‘CustomAddRows.R’ 中的正確順序，傳回物件清單：

    CustomAddRows <- function(dataset1, dataset2, swap=FALSE) { 
        if (swap) { dataset <- rbind(dataset2, dataset1)) } 
        else { dataset <- rbind(dataset1, dataset2)) 
        } 
    return (list(dataset, dataset1, dataset2)) 
    } 

**視覺效果輸出：** 您也可以指定 *Visualization*類型的輸出連接埠，以顯示 R 圖形裝置的輸出和主控台輸出。 這個連接埠不是 R 函式輸出的一部分，而且不會干擾其他輸出連接埠類型的順序。 若要將視覺效果連接埠新增至自訂模組，請針對其 **type** 屬性新增 *Visualization* 值的 **Output** 元素：

    <Output id="deviceOutput" name="View Port" type="Visualization">
      <Description>View the R console graphics device output.</Description>
    </Output>

**輸出規則：**

* **Output** 元素的 **id** 屬性值必須是有效的 R 變數名稱。
* **Output** 元素的 **id** 屬性值長度不能超過 32 個字元。
* **Output** 元素的 **name** 屬性值長度不能超過 64 個字元。
* **Output** 元素的 **type** 屬性值必須是 *Visualization*。

### <a name="arguments"></a>引數
您可以透過 **Arguments** 元素中定義的模組參數，將其他資料傳遞至 R 函數。 選取模組時，這些參數會出現在 Machine Learning UI 最右側的屬性窗格中。 引數可以是任何支援的類型，或者您可以視需要建立自訂列舉。 類似於 **Ports** 元素，**Arguments** 元素可以有選擇性 **Description** 元素，以指定當滑鼠停留在參數名稱上時所顯示的文字。
您可以將模組的選擇性屬性 (例如 defaultValue、minValue 和 maxValue) 新增到任何引數，做為 **Properties** 元素的屬性。 **Properties** 元素的有效屬性取決於引數類型，下一節將說明這些有效屬性和支援的引數類型。 **isOptional** 屬性設定為 **"true"** 的引數不需要使用者輸入值。 如果未提供值給引數，則不會傳遞引數給進入點函式。 選擇性的進入點函式引數必須由函式明確處理，例如在進入點函式定義中指派 NULL 預設值。 選擇性引數將只會強制執行其他引數限制 (也就是最小值或最大值，如果使用者提供值)。
如同輸入和輸出，每個參數都必須有與其相關聯的 id 值。 在我們的快速入門範例中，相關聯的 id/參數是 *swap*。

### <a name="arg-element"></a>Arg 元素
模組參數是使用 XML 定義檔之 **Arguments** 區段的 **Arg** 子元素所定義。 如同在 **Ports** 區段中的子元素，**Arguments** 區段中的參數順序會定義 UX 中遇到的配置。 參數會依照其在 XML 檔案中定義的相同順序，由上而下顯示在 UI 中。 Machine Learning 所支援的參數類型列示於此。 

**int** ：整數 (32 位元) 類型的參數。

    <Arg id="intValue1" name="Int Param" type="int">
        <Properties min="0" max="100" default="0" />
        <Description>Integer Parameter</Description>
    </Arg>


* *選擇性屬性*：**min**、**max**、**default** 和 **isOptional**

**double** ：double 類型的參數。

    <Arg id="doubleValue1" name="Double Param" type="double">
        <Properties min="0.000" max="0.999" default="0.3" />
        <Description>Double Parameter</Description>
    </Arg>


* *選擇性屬性*：**min**、**max**、**default** 和 **isOptional**

**bool** ：在 UX 中以核取方塊代表的布林值參數。

    <Arg id="boolValue1" name="Boolean Param" type="bool">
        <Properties default="true" />
        <Description>Boolean Parameter</Description>
    </Arg>



* *選擇性屬性*： **default** - 如果未設定，則為 false

**string**：標準字串

    <Arg id="stringValue1" name="My string Param" type="string">
        <Properties isOptional="true" />
        <Description>String Parameter 1</Description>
    </Arg>    

* *選擇性屬性*：**default** 和 **isOptional**

**ColumnPicker**：資料行選取參數。 此類型在 UX 中會轉譯成資料行選擇器。 此處使用 **Property** 元素來指定所要選取之資料行的連接埠 id，其中目標連接埠類型必須是 *DataTable*。 資料行選取的結果會傳遞至 R 函式，做為包含所選取資料行名稱的字串清單。 

        <Arg id="colset" name="Column set" type="ColumnPicker">      
          <Properties portId="datasetIn1" allowedTypes="Numeric" default="NumericAll"/>
          <Description>Column set</Description>
        </Arg>


* *必要屬性*：**portId** - 符合類型為 *DataTable* 之 Input 元素的 id。
* *選擇性屬性*：
  
  * **allowedTypes** - 篩選您可以從中挑選的資料行類型。 有效值包含： 
    
    * 數值
    * BOOLEAN
    * 類別
    * 字串
    * 標籤
    * 功能
    * 分數
    * 全部
  * **default** - 資料行選擇器的有效預設選取項目包括： 
    
    * None
    * NumericFeature
    * NumericLabel
    * NumericScore
    * NumericAll
    * BooleanFeature
    * BooleanLabel
    * BooleanScore
    * BooleanAll
    * CategoricalFeature
    * CategoricalLabel
    * CategoricalScore
    * CategoricalAll
    * StringFeature
    * StringLabel
    * StringScore
    * StringAll
    * AllLabel
    * AllFeature
    * AllScore
    * 全部

**DropDown**：使用者指定的列舉 (下拉式) 清單。 下拉式清單項目是在 **Properties** 元素中透過 **Item** 元素所指定。 每個 **Item** 的 **id** 必須是唯一且有效的 R 變數。 **Item** 的 **name** 的值可以同時做為您看到的文字以及傳遞至 R 函式的值。

    <Arg id="color" name="Color" type="DropDown">
      <Properties default="red">
        <Item id="red" name="Red Value"/>
        <Item id="green" name="Green Value"/>
        <Item id="blue" name="Blue Value"/>
      </Properties>
      <Description>Select a color.</Description>
    </Arg>    

* *選擇性屬性*：
  * **default** - 預設屬性的值必須對應至其中一個 **Item** 元素的 id 值。

### <a name="auxiliary-files"></a>輔助檔案
放在自訂模組 ZIP 檔案中的所有檔案在執行期間都可供使用。 所有存在的目錄結構皆會保留。 也就是說，檔案的獲得在本機和在 Azure Machine Learning 執行中的運作相同。 

> [!NOTE]
> 請注意，所有檔案都會解壓縮到 ‘src’ 目錄中，因此所有路徑應該都有 ‘src/’ 前置詞。
> 
> 

例如，假設您要先移除資料集中包含 NA 的所有資料列，並移除所有重複的資料列，然後才將該資料集輸出到 CustomAddRows 中，而且您已經撰寫一個在 RemoveDupNARows.R 檔案中執行該操作的 R 函數：

    RemoveDupNARows <- function(dataFrame) {
        #Remove Duplicate Rows:
        dataFrame <- unique(dataFrame)
        #Remove Rows with NAs:
        finalDataFrame <- dataFrame[complete.cases(dataFrame),]
        return(finalDataFrame)
    }
您可以使用 CustomAddRows 函數獲得 removeDupNARows.R 輔助檔案：

    CustomAddRows <- function(dataset1, dataset2, swap=FALSE) {
        source("src/RemoveDupNARows.R")
            if (swap) { 
                dataset <- rbind(dataset2, dataset1))
             } else { 
                  dataset <- rbind(dataset1, dataset2)) 
             } 
        dataset <- removeDupNARows(dataset)
        return (dataset)
    }

接著，上傳包含 ‘CustomAddRows.R’、‘CustomAddRows.xml’ 和 ‘RemoveDupNARows.R’ 的 zip 檔案，做為自訂 R 模組。

## <a name="execution-environment"></a>執行環境
R 指令碼的執行環境使用與 **Execute R Script** 模組相同的 R 版本，並且可以使用相同的預設封裝。 您也可以將其他 R 封裝納入自訂模組 zip 封裝，以將其新增至自訂模組。 只要在您的 R 指令碼中載入它們，就像在您自己的 R 環境中一樣。 

**執行環境的限制** 包括：

* 非持續性檔案系統：執行自訂模組時所撰寫的檔案無法在相同模組的多次執行間保留。
* 無法存取網路

