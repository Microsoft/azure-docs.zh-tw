---
title: 使用 Azure Machine Learning 資料準備 SDK 載入資料 - Python
description: 了解如何使用 Azure Machine Learning 資料準備 SDK 來載入資料。 您可以載入不同類型的輸入資料、指定資料類型與參數，或使用 SDK 智慧型讀取功能來自動偵測檔案類型。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: cforbe
author: cforbe
manager: cgronlun
ms.reviewer: jmartens
ms.date: 09/24/2018
ms.openlocfilehash: 436ff9d318dc311efe27352a8b2ac91cfb5be618
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47221314"
---
#<a name="load-and-read-data-with-azure-machine-learning"></a>使用 Azure Machine Learning 載入和讀取資料

使用 [Azure Machine Learning 資料準備 SDK](https://docs.microsoft.com/python/api/overview/azure/dataprep?view=azure-dataprep-py) 來載入不同類型的輸入資料。 

若要載入資料，您有兩種方法：
+ 指定資料檔案類型和其參數
+ 使用 SDK 智慧讀取功能來自動偵測檔案的類型

## <a name="use-text-line-data"></a>使用文字行資料 
載入資料的其中一種最簡單的方式是以文字行方式讀取它。

以下是範例程式碼：
```python
dataflow = dprep.read_lines(path='./data/text_lines.txt')
dataflow.head(5)
```
||折線圖|
|----|-----|
|0|日期 \|\|  最低溫度 \|\|  最高溫度|
|1|2015-07-1 \|\|  -4.1 \|\|  10.0|
|2|2015-07-2 \|\|  -0.8 \|\|  10.8|
|3|2015-07-3 \|\|  -7.0 \|\|  10.5|
|4|2015-07-4 \|\|  -5.5 \|\|  9.3|

內嵌資料之後，您可以擷取完整資料集的 pandas DataFrame。

以下是範例程式碼：
```python
df = dataflow.to_pandas_dataframe()
df
```
範例輸出：
||折線圖|
|----|-----|
|0|日期 \|\| 最低溫度 \|\| 最高溫度|
|1|2015-07-1\|\| 4.1\|\| 10.0|
|2|2015-07-2\|\| 0.8\|\| 10.8|
|3|2015-07-3\|\| 7.0\|\| 10.5|
|4|2015-07-4\|\| 5.5\|\| 9.3|

## <a name="use-csv-data"></a>使用 CSV 資料
讀取分隔檔時，您可以讓底層執行階段推斷剖析參數 (例如分隔符號、編碼、是否要使用標題等)，而非提供這些參數。 針對此範例，嘗試透過只指定檔案位置來讀取它。 

以下是範例程式碼：
```python
# SAS expires June 16th, 2019
dataflow = dprep.read_csv(path='https://dpreptestfiles.blob.core.windows.net/testfiles/read_csv_duplicate_headers.csv?st=2018-06-15T23%3A01%3A42Z&se=2019-06-16T23%3A01%3A00Z&sp=r&sv=2017-04-17&sr=b&sig=ugQQCmeC2eBamm6ynM7wnI%2BI3TTDTM6z9RPKj4a%2FU6g%3D')
dataflow.head(5)
```

範例輸出：
| |stnam|fipst|leaid|leanm10|ncessch|MAM_MTH00numvalid_1011|
|-----|-------|---------| -------|------|-----|------|-----|
|0||stnam|fipst|leaid|leanm10|ncessch|MAM_MTH00numvalid_1011|
|1|ALABAMA|1|101710|Hale County|10171002158| |
|2|ALABAMA|1|101710|Hale County|10171002162| |
|3|ALABAMA|1|101710|Hale County|10171002156| |
|4|ALABAMA|1|101710|Hale County|10171000588|2|

您可以指定的其中一個參數是要在您讀取的檔案中跳過的行數。 使用下列程式碼來篩掉重複的行。
```python
dataflow = dprep.read_csv(path='https://dpreptestfiles.blob.core.windows.net/testfiles/read_csv_duplicate_headers.csv',
                          skip_rows=1)
dataflow.head(5)
```

範例輸出：
| |stnam|fipst|leaid|leanm10|ncessch|MAM_MTH00numvalid_1011|
|-----|-------|---------| -------|------|-----|------|-----|
|0|ALABAMA|1|101710|Hale County|10171002158|29|
|1|ALABAMA|1|101710|Hale County|10171002162|40 |
|2|ALABAMA|1|101710|Hale County|10171002156| 43|
|3|ALABAMA|1|101710|Hale County|10171000588|2|
|4|ALABAMA|1|101710|Hale County|10171000589|23 |

接著，您可以查看資料行的資料類型。
以下是範例程式碼：
```python
dataflow.head(1).dtypes

stnam                     object
fipst                     object
leaid                     object
leanm10                   object
ncessch                   object
schnam10                  object
MAM_MTH00numvalid_1011    object
dtype: object
```

不幸的是，我們的所有資料行都傳回為字串。 這是因為 Azure Machine Learning 資料準備 SDK 預設不會變更您的資料類型。 我們所讀取的資料來源是文字檔，因此 SDK 會以字串形式讀取有值。 不過針對此範例，我們想要將數值資料行剖析為數字。 若要這樣做，您可以將 inference_arguments 參數設定為 current_culture。

```
dataflow = dprep.read_csv(path='https://dpreptestfiles.blob.core.windows.net/testfiles/read_csv_duplicate_headers.csv',
                          skip_rows=1,
                          inference_arguments=dprep.InferenceArguments.current_culture())
dataflow.head(1).dtypes

stnam                      object
fipst                     float64
leaid                     float64
leanm10                    object
ncessch                   float64
schnam10                   object
ALL_MTH00numvalid_1011    float64
dtype: object
```

已將數個資料行正確偵測為數字，而且其類型已設定為 float64。 內嵌完成之後，您可以擷取完整資料集的 pandas DataFrame，。
以下是範例程式碼：
```python
df = dataflow.to_pandas_dataframe()
df
```

範例輸出：
| |stnam|leanm10|ncessch|MAM_MTH00numvalid_1011|
|-----|-------|---------| -------|------|-----|
|0|ALABAMA|Hale County|1.017100e+10|49.0|
|1|ALABAMA|Hale County|1.017100e+10|40.0|
|2|ALABAMA|Hale County|1.017100e+10|43.0|
|3|ALABAMA|Hale County|1.017100e+10|2.0|
|4|ALABAMA|Hale County|1.017100e+10|23.0|

## <a name="use-excel-data"></a>使用 Excel 資料
Azure Machine Learning 資料準備 SDK 包括可載入 Excel 檔案的 `read_excel` 函式。 以下是範例程式碼：
```python
dataflow = dprep.read_excel(path='./data/excel.xlsx')
dataflow.head(5)
```

範例輸出：
||Column1|Column2|Column3|Column4|Column5|Column6|Column7|Column8|
|------|------|------|-----|------|-----|-------|----|-----|
|0|霍巴|鐵、IVB|60000000.0|已找到|1920.0|http://www.lpi.usra.edu/meteor/metbull.php?cod.. 。 |-19.58333|17.91667|
|1|約克角半島|鐵、IIIAB|58200000.0|已找到|1818.0|http://www.lpi.usra.edu/meteor/metbull.php?cod.. 。 |76.13333|-64.93333|
|2|隕鐵|鐵、IAB-MG|50000000.0|已找到|1576.0|http://www.lpi.usra.edu/meteor/metbull.php?cod.. 。 |-27.46667|-60.58333|
|3|代亞布羅峽谷隕石坑|鐵、IAB-MG|30000000.0|已找到|1891.0|http://www.lpi.usra.edu/meteor/metbull.php?cod.. 。 |35.05000|-111.03333|
|4|新疆鐵隕石|鐵、IIIE|28000000.0|已找到|1898.0|http://www.lpi.usra.edu/meteor/metbull.php?cod.. 。 |47.00000|88.00000|

您已在入 Excel 檔案的第一張工作表。 您可以透過明確指定您要載入的工作表名稱來達到相同的結果。 若要改為載入第二張工作表，您可以提供其名稱做為引數。 例如︰
```python
dataflow = dprep.read_excel(path='./data/excel.xlsx', sheet_name='Sheet2')
dataflow.head(5)
```

範例輸出：
||Column1|Column2|Column3|Column4|Column5|Column6|Column7|Column8|
|------|------|------|-----|------|-----|-------|----|-----|
|0|None|None|None|None|None|None|None|None|None|
|1|None|None|None|None|None|None|None|None|None|
|2|None|None|None|None|None|None|None|None|None|
|3|RANK|標題|Studio|全球|國內 / %|資料行 1|海外 / %|資料行 2|年^|
|4|1|阿凡達|福斯|2788|760.5|0.273|2027.5|0.727|2009^|5|

如您所見，第二張工作表中的表格有標題與三個空列。 您必須相應地修改函式的引數。例如：
```python
dataflow = dprep.read_excel(path='./data/excel.xlsx', sheet_name='Sheet2', use_header=True, skip_rows=3)
df = dataflow.to_pandas_dataframe()
df
```

範例輸出：
||RANK|標題|Studio|全球|國內 / %|資料行 1|海外 / %|資料行 2|年^|
|------|------|------|-----|------|-----|-------|----|-----|-----|
|0|1|阿凡達|福斯|2788|760.5|0.273|2027.5|0.727|2009^|
|1|2|鐵達尼號|派拉蒙影業|2186.8|658.7|0.301|1528.1|0.699|1997^|
|2|3|漫威復仇者聯盟|BV|1518.6|623.4|0.41|895.2|0.59|2012|
|3|4|哈利波特：死神的聖物Ⅱ|華納兄弟影業|1341.5|381|0.284|960.5|0.716|2011|
|4|5|冰雪奇緣|BV|1274.2|400.7|0.314|873.5|0.686|2013|

## <a name="use-fixed-width-data-files"></a>使用固定寬度資料檔案
針對固定寬度檔案，您可以指定位移清單。 第一欄一律假設為從位移 0 開始。例如：
```python
dataflow = dprep.read_fwf('./data/fixed_width_file.txt', offsets=[7, 13, 43, 46, 52, 58, 65, 73])
dataflow.head(5)
```

範例輸出：
||010000|99999|BOGUS 挪威|否|NO_1|ENRS|資料行 7|Column8|Column9|
|------|------|------|-----|------|-----|-------|----|-----|----|
|0|010003|99999|BOGUS 挪威|否|否|ENSO||||
|1|010010|99999|揚馬延|否|JN|ENJA|+70933|-008667|+00090|
|2|010013|99999|ROST|否|否|||||
|3|010014|99999|SOERSTOKKEN|否|否|ENSO|+59783|+005350|+00500|
|4|010015|99999|BRINGELAND|否|否|ENBL|+61383|+005867|+03270|


若檔案中沒有標題，您可以將第一列視為資料。 透過傳遞 `PromoteHeadersMode.NONE` 給標題關鍵字引數，您可以避免標題偵測並取得正確的資料。 例如︰
```python
dataflow = dprep.read_fwf('./data/fixed_width_file.txt',
                          offsets=[7, 13, 43, 46, 52, 58, 65, 73],
                          header=dprep.PromoteHeadersMode.NONE)

df = dataflow.to_pandas_dataframe()
df
```

範例輸出：

||Column1|Column2|Column3|Column4|Column5|Column6|Column7|Column8|Column9|
|------|------|------|-----|------|-----|-------|----|-----|----|
|0|010000|99999|BOGUS 挪威|否|NO_1|ENRS|資料行 7|Column8|Column9|
|1|010003|99999|BOGUS 挪威|否|否|ENSO||||
|2|010010|99999|揚馬延|否|JN|ENJA|+70933|-008667|+00090|
|3|010013|99999|ROST|否|否|||||
|4|010014|99999|SOERSTOKKEN|否|否|ENSO|+59783|+005350|+00500|
|5|010015|99999|BRINGELAND|否|否|ENBL|+61383|+005867|+03270|

## <a name="use-sql-data"></a>使用 SQL 資料
Azure Machine Learning 資料準備 SDK 也可以從 SQL 伺服器載入資料。 目前只支援 Microsoft SQL Server。
若要從 SQL 伺服器讀取資料，請建立包含連線資訊的資料來源物件。 例如︰
```python
secret = dprep.register_secret("[SECRET-USERNAME]", "[SECRET-PASSWORD]")

ds = dprep.MSSQLDataSource(server_name="[SERVER-NAME]",
                           database_name="[DATABASE-NAME]",
                           user_name="[DATABASE-USERNAME]",
                           password=[DATABASE-PASSWORD])
```
如您所見，`MSSQLDataSource` 的密碼參數接受祕密物件。 您能以兩種方式取得祕密物件：
-   向執行引擎註冊祕密與其值。 
-   建立只具有識別碼 (若祕密值已在執行環境中註冊，則很實用) 的祕密。

在您建立資料來源物件之後，您可以繼續讀取資料。 例如︰
```python
dataflow = dprep.read_sql(ds, "SELECT top 100 * FROM [SalesLT].[Product]")
dataflow.head(5)
```

範例輸出：
||ProductID|名稱|ProductNumber|色彩|StandardCost|ListPrice|大小|Weight|ProductCategoryID|ProductModelID|SellStartDate|SellEndDate|DiscontinuedDate|ThumbNailPhoto|ThumbnailPhotoFileName|rowguid|ModifiedDate|
|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|
|0|680|HL Road Frame - Black, 58|FR-R92B-58|黑色|1059.3100|1431.50|58|1016.04|18|6|2002-06-01 00:00:00+00:00|None|None|b'GIF89aP\x001\x00\xf7\x00\x00\x00\x00\x00\x80...|no_image_available_small.gif|43dd68d6-14a4-461f-9069-55309d90ea7e|2008-03-11 |0:01:36.827000+00:00|
|1|706|HL Road Frame - Red, 58|FR-R92R-58|紅色|1059.3100|1431.50|58|1016.04|18|6|2002-06-01 00:00:00+00:00|None|None|b'GIF89aP\x001\x00\xf7\x00\x00\x00\x00\x00\x80...|no_image_available_small.gif|9540ff17-2712-4c90-a3d1-8ce5568b2462|2008-03-11 |10:01:36.827000+00:00|
|2|707|Sport-100 Helmet, Red|HL-U509-R|紅色|13.0863|34.99|None|None|35|33|2005-07-01 00:00:00+00:00|None|None|b'GIF89aP\x001\x00\xf7\x00\x00\x00\x00\x00\x80...|no_image_available_small.gif|2e1ef41a-c08a-4ff6-8ada-bde58b64a712|2008-03-11 |10:01:36.827000+00:00|
|3|708|Sport-100 Helmet, Black|HL-U509|黑色|13.0863|34.99|None|None|35|33|2005-07-01 00:00:00+00:00|None|None|b'GIF89aP\x001\x00\xf7\x00\x00\x00\x00\x00\x80...|no_image_available_small.gif|a25a44fb-c2de-4268-958f-110b8d7621e2|2008-03-11 |10:01:36.827000+00:00|
|4|709|Mountain Bike Socks, M|SO-B909-M|白色|3.3963|9.50|M|None|27|18|2005-07-01 00:00:00+00:00|2006-06-30 00:00:00+00:00|None|b'GIF89aP\x001\x00\xf7\x00\x00\x00\x00\x00\x80...|no_image_available_small.gif|18f95f47-1540-4e02-8f1f-cc1bcb6828d0|2008-03-11 |10:01:36.827000+00:00|

```python
df = dataflow.to_pandas_dataframe()
df.dtypes
```

範例輸出：
```
ProductID                                     int64
Name                                         object
ProductNumber                                object
Color                                        object
StandardCost                                float64
ListPrice                                   float64
Size                                         object
Weight                                      float64
ProductCategoryID                             int64
ProductModelID                                int64
SellStartDate             datetime64[ns, UTC+00:00]
SellEndDate                                  object
DiscontinuedDate                             object
ThumbNailPhoto                               object
ThumbnailPhotoFileName                       object
rowguid                                      object
ModifiedDate              datetime64[ns, UTC+00:00]
dtype: object
```

## <a name="use-azure-data-lake-storage"></a>使用 Azure Data Lake 儲存體
SDK 能以兩種方式取得必要 OAuth 權杖以存取 Azure Data Lake Storage：
-   從使用者 Azure CLI 登入的最近登入工作階段擷取存取權杖
-   使用服務主體 (SP) 與憑證做為祕密

### <a name="use-an-access-token-from-a-recent-azure-cli-session"></a>使用來自最近 Azure CLI 工作階段的存取權杖
在您的本機電腦上，執行下列命令：

> [!NOTE] 
> 若您的使用者帳戶是多個 Azure 租用戶的成員，您必須以 AAD URL 主機名稱格式指定該租用戶。


例如︰
```azurecli
az login
az account show --query tenantId
dataflow = read_csv(path = DataLakeDataSource(path='adl://dpreptestfiles.azuredatalakestore.net/farmers-markets.csv', tenant='microsoft.onmicrosoft.com')) head = dataflow.head(5) head
```
### <a name="create-a-service-principal-with-the-azure-cli"></a>使用 Azure CLI 建立服務主體
您可以使用 Azure CLI 來建立服務主體與對應的憑證。 這個特定服務主體設定為讀者，而且其範圍縮減為僅限 Azure Data Lake Storage 帳戶 'dpreptestfiles'。  例如︰
```azurecli
az account set --subscription "Data Wrangling development"
az ad sp create-for-rbac -n "SP-ADLS-dpreptestfiles" --create-cert --role reader --scopes /subscriptions/35f16a99-532a-4a47-9e93-00305f6c40f2/resourceGroups/dpreptestfiles/providers/Microsoft.DataLakeStore/accounts/dpreptestfiles
```
此命令會發出 `appId` 與憑證檔案的路徑 (通常位於主資料夾)。 .crt 檔案同時包含 PEM 格式的公開憑證與私密金鑰。

擷取指紋：
```
openssl x509 -in adls-dpreptestfiles.crt -noout -fingerprint
```

若要為 Azure Data Lake Storage 檔案系統設定 ACL，請使用使用者或服務主體 (此範例中所使用) 的 objectId。 例如︰
```azurecli
az ad sp show --id "8dd38f34-1fcb-4ff9-accd-7cd60b757174" --query objectId
```

若要設定 Azure Data Lake Storage 檔案系統的 `Read` 和 `Execute` 存取，您必須個別設定資料夾和檔案的 ACL。 這是因為基礎 HDFS ACL 模型不支援繼承。 例如︰
```azurecli
az dls fs access set-entry --account dpreptestfiles --acl-spec "user:e37b9b1f-6a5e-4bee-9def-402b956f4e6f:r-x" --path /
az dls fs access set-entry --account dpreptestfiles --acl-spec "user:e37b9b1f-6a5e-4bee-9def-402b956f4e6f:r--" --path /farmers-markets.csv
```
```
certThumbprint = 'C2:08:9D:9E:D1:74:FC:EB:E9:7E:63:96:37:1C:13:88:5E:B9:2C:84'
certificate = ''
with open('./data/adls-dpreptestfiles.crt', 'rt', encoding='utf-8') as crtFile:
    certificate = crtFile.read()

servicePrincipalAppId = "8dd38f34-1fcb-4ff9-accd-7cd60b757174"
```
### <a name="acquire-an-oauth-access-token"></a>取得 OAuth 存取權杖
使用 `adal` 套件 (透過 `pip install adal`) 在 MSFT 租用戶上建立驗證內容，並取得 OAuth 存取權杖。 針對 ADLS，權杖要求中的資源必須適用於 'https://datalake.azure.net'，這與大部分的其他 Azure 資源不同。

```python
import adal
from azureml.dataprep.api.datasources import DataLakeDataSource

ctx = adal.AuthenticationContext('https://login.microsoftonline.com/microsoft.onmicrosoft.com')
token = ctx.acquire_token_with_client_certificate('https://datalake.azure.net/', servicePrincipalAppId, certificate, certThumbprint)
dataflow = dprep.read_csv(path = DataLakeDataSource(path='adl://dpreptestfiles.azuredatalakestore.net/farmers-markets.csv', accessToken=token['accessToken']))
dataflow.to_pandas_dataframe().head()
```

||FMID|MarketName|網站|street|city|郡/縣|
|----|------|-----|----|----|----|----|
|0|1012063|喀里多尼亞農民市場協會 - 丹維爾|https://sites.google.com/site/caledoniafarmers.. 。 ||丹維爾|喀里多尼亞|
|1|1011871|Stearns Homestead 農民市集|http://Stearnshomestead.com |6975 Ridge Road|帕爾馬|凱霍加郡|
|2|1011878|100 Mile Market|http://www.pfcmarkets.com |507 Harrison St|卡拉馬朱|卡拉馬朱|
|3|1009364|106 南主幹道農民市集|http://thetownofsixmile.wordpress.com/ |106 南主幹道|六英里|||
|4|1010691|第 10 街社區農民市集|http://agrimissouri.com/mo-grown/grodetail.php.. 。 |10th Street and Poplar|拉馬爾|巴頓|

## <a name="use-smart-reading"></a>使用「智慧讀取」

使用 SDK 智慧讀取功能來自動偵測檔案的類型。
