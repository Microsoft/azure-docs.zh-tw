---
title: 使用 Azure Machine Learning 資料準備 SDK 寫入資料 - Python
description: 了解如何使用 Azure Machine Learning 資料準備 SDK 來載入資料。 您可以在資料流程的任何時間點寫出資料，也可以將資料寫入我們支援的任何位置 (本機檔案系統、Azure Blob 儲存體和 Azure Data Lake Storage)。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: cforbe
author: cforbe
manager: cgronlun
ms.reviewer: jmartens
ms.date: 09/24/2018
ms.openlocfilehash: 81344d388fbba0db034b8adb06adab6797ec2ce1
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166737"
---
# <a name="write-data-using-the-azure-machine-learning-data-prep-sdk"></a>使用 Azure Machine Learning 資料準備 SDK 來寫入資料
您可以在資料流程的任何時間點寫出資料。 這些寫入作業會新增為所產生資料流程的步驟，並會在每次資料流程執行時執行。 資料會寫入多個分割區檔案，以允許平行寫入。

由於管線中的寫入步驟數目沒有任何限制，因此您能輕鬆地新增其他寫入步驟，取得要進行疑難排解或針對其他管線使用的中繼結果。 

每次執行寫入步驟時，就會完整提取資料流程中的資料。 例如，具有三個寫入步驟的資料流程會讀取及處理資料集中的每一筆記錄三次。

## <a name="supported-data-types-and-location"></a>支援的資料類型與位置

支援下列檔案格式
-   符號分隔檔案 (CSV、TSV 等)
-   Parquet 檔案

使用 [Azure Machine Learning 資料準備 Python SDK](https://aka.ms/data-prep-sdk)，您可以將資料寫入到：
+ 本機檔案系統
+ Azure Blob 儲存體
+ Azure Data Lake 儲存體

## <a name="spark-considerations"></a>Spark 考量
在 Spark 中執行資料流程時，您必須寫入空白的資料夾。 嘗試寫入現有資料夾的動作導致失敗。 請確定您的目標資料夾是空的，或針對每次執行使用不同的目標位置，否則寫入將會失敗。

## <a name="monitoring-write-operations"></a>監視寫入作業
為了方便起見，在完成寫入之後，會產生名為 SUCCESS 的標記檔案。 它的存在可協助您識別中繼寫入何時完成，而不需要等待整個管線完成。

## <a name="example-write-code"></a>範例寫入程式碼

針對此範例，一開始先將資料載入資料流程。 我們會以不同的格式重複使用此資料。

```python
import azureml.dataprep as dprep
t = dprep.smart_read_file('./data/fixed_width_file.txt')
t = t.to_number('Column3')
t.head(10)
```   

範例輸出︰
|   |  Column1 |    Column2 | Column3 | Column4  |Column5   | Column6 | Column7 | Column8 | Column9 |
| -------- |  -------- | -------- | -------- |  -------- |  -------- |  -------- |  -------- |  -------- |  -------- |
| 0 |   10000.0 |   99999.0 |   None|       否|     否  |   ENRS    |NaN    |   NaN |   NaN|    
|   1|      10003.0 |   99999.0 |   None|       否|     否  |   ENSO|       NaN|        NaN |NaN|   
|   2|  10010.0|    99999.0|    None|   否| JN| ENJA|   70933.0|    -8667.0 |90.0|
|3| 10013.0|    99999.0|    None|   否| 否| |   NaN|    NaN|    NaN|
|4| 10014.0|    99999.0|    None|   否| 否| ENSO|   59783.0|    5350.0| 500.0|
|5| 10015.0|    99999.0|    None|   否| 否| ENBL|   61383.0|    5867.0| 3270.0|
|6| 10016.0 |99999.0|   None|   否| 否|     |64850.0|   11233.0|    140.0|
|7| 10017.0|    99999.0|    None|   否| 否| ENFR|   59933.0|    2417.0| 480.0|
|8| 10020.0|    99999.0|    None|   否| SV|     |80050.0|   16250.0|    80.0|
|9| 10030.0|    99999.0|    None|   否| SV|     |77000.0|   15500.0|    120.0|

### <a name="delimited-file-example"></a>以符號分隔的檔案範例

在此節中，您可以看到使用 `write_to_csv` 函式寫入以符號分隔之檔案的範例。

```python
# Create a new data flow using `write_to_csv` 
write_t = t.write_to_csv(directory_path=dprep.LocalFileOutput('./test_out/'))

# Run the data flow to begin the write operation.
write_t.run_local()

written_files = dprep.read_csv('./test_out/part-*')
written_files.head(10)
```

範例輸出︰
|   |  Column1 |    Column2 | Column3 | Column4  |Column5   | Column6 | Column7 | Column8 | Column9 |
| -------- |  -------- | -------- | -------- |  -------- |  -------- |  -------- |  -------- |  -------- |  -------- |
| 0 |   10000.0 |   99999.0 |   ERROR |       否|     否  |   ENRS    |ERROR    |   ERROR |   ERROR|    
|   1|      10003.0 |   99999.0 |   ERROR |       否|     否  |   ENSO|       ERROR|        ERROR |ERROR|   
|   2|  10010.0|    99999.0|    ERROR |   否| JN| ENJA|   70933.0|    -8667.0 |90.0|
|3| 10013.0|    99999.0|    ERROR |   否| 否| |   ERROR|    ERROR|    ERROR|
|4| 10014.0|    99999.0|    ERROR |   否| 否| ENSO|   59783.0|    5350.0| 500.0|
|5| 10015.0|    99999.0|    ERROR |   否| 否| ENBL|   61383.0|    5867.0| 3270.0|
|6| 10016.0 |99999.0|   ERROR |   否| 否|     |64850.0|   11233.0|    140.0|
|7| 10017.0|    99999.0|    ERROR |   否| 否| ENFR|   59933.0|    2417.0| 480.0|
|8| 10020.0|    99999.0|    ERROR |   否| SV|     |80050.0|   16250.0|    80.0|
|9| 10030.0|    99999.0|    ERROR |   否| SV|     |77000.0|   15500.0|    120.0|

您可以在上面的輸出中看到由於未正確剖析的數字之故，數值欄中出現數個錯誤。 寫入 CSV 時，這些 Null 值預設會取代為字串 "ERROR"。 

您可以新增參數作為寫入呼叫的一部分，並指定要用來代表 Null 值的字串。 例如︰

```python
write_t = t.write_to_csv(directory_path=dprep.LocalFileOutput('./test_out/'), 
                         error='BadData',
                         na='NA')
write_t.run_local()
written_files = dprep.read_csv('./test_out/part-*')
written_files.head(10)
```

上述程式碼會產生以下輸出：
|   |  Column1 |    Column2 | Column3 | Column4  |Column5   | Column6 | Column7 | Column8 | Column9 |
| -------- |  -------- | -------- | -------- |  -------- |  -------- |  -------- |  -------- |  -------- |  -------- |
| 0 |   10000.0 |   99999.0 |   BadData |       否|     否  |   ENRS    |BadData    |   BadData |   BadData|    
|   1|      10003.0 |   99999.0 |   BadData |       否|     否  |   ENSO|       BadData|        BadData |BadData|   
|   2|  10010.0|    99999.0|    BadData |   否| JN| ENJA|   70933.0|    -8667.0 |90.0|
|3| 10013.0|    99999.0|    BadData |   否| 否| |   BadData|    BadData|    BadData|
|4| 10014.0|    99999.0|    BadData |   否| 否| ENSO|   59783.0|    5350.0| 500.0|
|5| 10015.0|    99999.0|    BadData |   否| 否| ENBL|   61383.0|    5867.0| 3270.0|
|6| 10016.0 |99999.0|   BadData |   否| 否|     |64850.0|   11233.0|    140.0|
|7| 10017.0|    99999.0|    BadData |   否| 否| ENFR|   59933.0|    2417.0| 480.0|
|8| 10020.0|    99999.0|    BadData |   否| SV|     |80050.0|   16250.0|    80.0|
|9| 10030.0|    99999.0|    BadData |   否| SV|     |77000.0|   15500.0|    120.0|


### <a name="parquet-file-example"></a>Parquet 檔案範例

類似於 `write_to_csv`，`write_to_parquet` 函式會傳回具有寫入 Parquet 步驟的新資料流程，該步驟是在資料流程執行時執行。

```python
write_parquet_t = t.write_to_parquet(directory_path=dprep.LocalFileOutput('./test_parquet_out/'),
error='MiscreantData')
```

接下來，您可以執行資料流程，以開始寫入作業。

```
write_parquet_t.run_local()

written_parquet_files = dprep.read_parquet_file('./test_parquet_out/part-*')
written_parquet_files.head(10)
```

上述程式碼會產生以下輸出：
|   |  Column1 |    Column2 | Column3 | Column4  |Column5   | Column6 | Column7 | Column8 | Column9 |
| -------- |  -------- | -------- | -------- |  -------- |  -------- |  -------- |  -------- |  -------- |  -------- |
| 0 |   10000.0 |   99999.0 |   MiscreantData |       否|     否  |   ENRS    |MiscreantData    |   MiscreantData |   MiscreantData|    
|   1|      10003.0 |   99999.0 |   MiscreantData |       否|     否  |   ENSO|       MiscreantData|        MiscreantData |MiscreantData|   
|   2|  10010.0|    99999.0|    MiscreantData |   否| JN| ENJA|   70933.0|    -8667.0 |90.0|
|3| 10013.0|    99999.0|    MiscreantData |   否| 否| |   MiscreantData|    MiscreantData|    MiscreantData|
|4| 10014.0|    99999.0|    MiscreantData |   否| 否| ENSO|   59783.0|    5350.0| 500.0|
|5| 10015.0|    99999.0|    MiscreantData |   否| 否| ENBL|   61383.0|    5867.0| 3270.0|
|6| 10016.0 |99999.0|   MiscreantData |   否| 否|     |64850.0|   11233.0|    140.0|
|7| 10017.0|    99999.0|    MiscreantData |   否| 否| ENFR|   59933.0|    2417.0| 480.0|
|8| 10020.0|    99999.0|    MiscreantData |   否| SV|     |80050.0|   16250.0|    80.0|
|9| 10030.0|    99999.0|    MiscreantData |   否| SV|     |77000.0|   15500.0|    120.0|
