---
title: 使用適用於預測的 Azure Machine Learning 套件，建置和部署預測模型。
description: 了解如何使用適用於預測的 Azure Machine Learning 套件，建置、定型、測試和部署預測模型。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: mattcon
author: matthewconners
ms.date: 07/13/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 06613ed1eac43ebe865666f85235de74903b1d5c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46953583"
---
# <a name="build-and-deploy-forecasting-models-with-azure-machine-learning"></a>使用 Azure Machine Learning 建置和部署預測模型

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

在本文中，了解如何使用**適用於預測的 Azure Machine Learning 套件** (AMLPF)，快速建置和部署預測模型。 工作流程如下︰

1. 載入並探索資料
2. 建立功能
3. 定型並選取最佳模型
4. 部署模型，然後取用 Web 服務

如需每個模組和類別的完整轉換器與模型清單以及詳細參考，請參閱[套件參考文件](https://aka.ms/aml-packages/forecasting)。

## <a name="prerequisites"></a>必要條件

1. 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

1. 您必須設定並安裝下列帳戶和應用程式：
   - Azure Machine Learning 測試帳戶 
   - Azure Machine Learning 模型管理帳戶
   - 已安裝 Azure Machine Learning Workbench 

 如果還未建立或安裝這三者，請依照 [Azure Machine Learning 快速入門和 Workbench 安裝](../desktop-workbench/quickstart-installation.md)一文來進行。

1. 您必須安裝適用於預測的 Azure Machine Learning 套件。 在此了解如何[安裝此套件](https://aka.ms/aml-packages/forecasting) \(英文\)。

## <a name="sample-data-and-jupyter-notebook"></a>範例資料和 Jupyter Notebook

### <a name="sample-workflow"></a>範例工作流程 
此範例會依照下列工作流程進行：
 
1. **內嵌資料**：載入資料集，並將其轉換成 TimeSeriesDataFrame。 此資料框架是由適用於預測的 Azure Machine Learning 套件所提供的時間序列資料結構，本文中稱為 **AMLPF**。

2. **建立功能**：使用 AMLPF 所提供的各種功能化轉換器建立功能。

3. **定型並選取最佳模型**：比較各種單一變量時間序列模型和機器學習模型的效能。 

4. **部署模型**：透過 Azure Machine Learning Workbench，將定型的模型管線部署為 Web 服務，以供其他人取用。

### <a name="get-the-jupyter-notebook"></a>取得 Jupyter Notebook

下載 Notebook 以自行執行此處所述的程式碼範例。

> [!div class="nextstepaction"]
> [取得 Jupyter Notebook](https://aka.ms/aml-packages/forecasting/notebooks/sales_forecasting) \(英文\)

### <a name="explore-the-sample-data"></a>瀏覽範例資料

下列程式碼範例中的機器學習預測範例依賴[芝加哥大學的 Dominick's Finer Foods 資料集](https://research.chicagobooth.edu/kilts/marketing-databases/dominicks)預測柳橙汁銷售量。 Dominick 是芝加哥大都會區中的連鎖雜貨店。

### <a name="import-any-dependencies-for-this-sample"></a>匯入此範例的任何相依性

您必須為下列程式碼範例匯入這些相依性：


```python
import pandas as pd
import numpy as np
import math
import pkg_resources
from datetime import timedelta
import matplotlib
matplotlib.use('agg')
%matplotlib inline
from matplotlib import pyplot as plt

from sklearn.linear_model import Lasso, ElasticNet
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.neighbors import KNeighborsRegressor

from ftk import TimeSeriesDataFrame, ForecastDataFrame, AzureMLForecastPipeline
from ftk.ts_utils import last_n_periods_split

from ftk.transforms import TimeSeriesImputer, TimeIndexFeaturizer, DropColumns
from ftk.transforms.grain_index_featurizer import GrainIndexFeaturizer
from ftk.models import Arima, SeasonalNaive, Naive, RegressionForecaster, ETS, BestOfForecaster
from ftk.models.forecaster_union import ForecasterUnion
from ftk.model_selection import TSGridSearchCV, RollingOriginValidator

from azuremltkbase.deployment import AMLSettings
from ftk.operationalization.forecast_webservice_factory import ForecastWebserviceFactory
from ftk.operationalization import ScoreContext

from ftk.data import get_a_year_of_daily_weather_data
print('imports done')
```

    imports done
    

## <a name="load-data-and-explore"></a>載入資料並探索

此程式碼片段顯示從原始資料集開始的一般程序，在此案例中為[來自 Dominick's Finer Foods 的資料](https://research.chicagobooth.edu/kilts/marketing-databases/dominicks)。  您也可以使用方便的函式 [load_dominicks_oj_data](https://docs.microsoft.com/python/api/ftk.data.dominicks_oj.load_dominicks_oj_data)。


```python
# Load the data into a pandas DataFrame
csv_path = pkg_resources.resource_filename('ftk', 'data/dominicks_oj/dominicks_oj.csv')
whole_df = pd.read_csv(csv_path, low_memory = False)
whole_df.head()
```




<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store</th>
      <th>brand</th>
      <th>week</th>
      <th>logmove</th>
      <th>feat</th>
      <th>price</th>
      <th>AGE60</th>
      <th>EDUC</th>
      <th>ETHNIC</th>
      <th>INCOME</th>
      <th>HHLARGE</th>
      <th>WORKWOM</th>
      <th>HVAL150</th>
      <th>SSTRDIST</th>
      <th>SSTRVOL</th>
      <th>CPDIST5</th>
      <th>CPWVOL5</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>tropicana</td>
      <td>40</td>
      <td>9.02</td>
      <td>0</td>
      <td>3.87</td>
      <td>0.23</td>
      <td>0.25</td>
      <td>0.11</td>
      <td>10.55</td>
      <td>0.10</td>
      <td>0.30</td>
      <td>0.46</td>
      <td>2.11</td>
      <td>1.14</td>
      <td>1.93</td>
      <td>0.38</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>tropicana</td>
      <td>46</td>
      <td>8.72</td>
      <td>0</td>
      <td>3.87</td>
      <td>0.23</td>
      <td>0.25</td>
      <td>0.11</td>
      <td>10.55</td>
      <td>0.10</td>
      <td>0.30</td>
      <td>0.46</td>
      <td>2.11</td>
      <td>1.14</td>
      <td>1.93</td>
      <td>0.38</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>tropicana</td>
      <td>47</td>
      <td>8.25</td>
      <td>0</td>
      <td>3.87</td>
      <td>0.23</td>
      <td>0.25</td>
      <td>0.11</td>
      <td>10.55</td>
      <td>0.10</td>
      <td>0.30</td>
      <td>0.46</td>
      <td>2.11</td>
      <td>1.14</td>
      <td>1.93</td>
      <td>0.38</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>tropicana</td>
      <td>48</td>
      <td>8.99</td>
      <td>0</td>
      <td>3.87</td>
      <td>0.23</td>
      <td>0.25</td>
      <td>0.11</td>
      <td>10.55</td>
      <td>0.10</td>
      <td>0.30</td>
      <td>0.46</td>
      <td>2.11</td>
      <td>1.14</td>
      <td>1.93</td>
      <td>0.38</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>tropicana</td>
      <td>50</td>
      <td>9.09</td>
      <td>0</td>
      <td>3.87</td>
      <td>0.23</td>
      <td>0.25</td>
      <td>0.11</td>
      <td>10.55</td>
      <td>0.10</td>
      <td>0.30</td>
      <td>0.46</td>
      <td>2.11</td>
      <td>1.14</td>
      <td>1.93</td>
      <td>0.38</td>
    </tr>
  </tbody>
</table>



資料是依 brand 和 store，由每週的銷售量所組成。 售出數量的對數位於 _logmove_ 欄。 此資料也包含一些客戶人口統計功能。 

若要建構時間序列的模型，您必須從這個資料框架擷取下列元素： 
+ 日期/時間軸 
+ 要預測的銷售數量


```python
# The sales are contained in the 'logmove' column. 
# Values are logarithmic, so exponentiate and round them to get quantity sold
def expround(x):
    return math.floor(math.exp(x) + 0.5)
whole_df['Quantity'] = whole_df['logmove'].apply(expround)

# The time axis is in the 'week' column
# This is the week offset from the week of 1989-09-07 through 1989-09-13 inclusive
# Create new datetime columns containing the start and end of each week period
weekZeroStart = pd.to_datetime('1989-09-07 00:00:00')
weekZeroEnd = pd.to_datetime('1989-09-13 23:59:59')
whole_df['WeekFirstDay'] = whole_df['week'].apply(lambda n: weekZeroStart + timedelta(weeks=n))
whole_df['WeekLastDay'] = whole_df['week'].apply(lambda n: weekZeroEnd + timedelta(weeks=n))
whole_df[['store','brand','WeekLastDay','Quantity']].head()
```




<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store</th>
      <th>brand</th>
      <th>WeekLastDay</th>
      <th>數量</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>tropicana</td>
      <td>1990-06-20 23:59:59</td>
      <td>8256</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>tropicana</td>
      <td>1990-08-01 23:59:59</td>
      <td>6144</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>tropicana</td>
      <td>1990-08-08 23:59:59</td>
      <td>3840</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>tropicana</td>
      <td>1990-08-15 23:59:59</td>
      <td>8000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>tropicana</td>
      <td>1990-08-29 23:59:59</td>
      <td>8896</td>
    </tr>
  </tbody>
</table>




```python
nseries = whole_df.groupby(['store', 'brand']).ngroups
print('{} time series in the data frame.'.format(nseries))
```

    249 time series in the data frame.
    

此資料在資料框架中包含大約 250 種不同的 store 及 brand 組合。 每個組合都會定義其本身的銷售時間序列。 

您可以使用 [TimeSeriesDataFrame](https://docs.microsoft.com/python/api/ftk.dataframe_ts.timeseriesdataframe?view=azure-ml-py-latest) 類別，以方便在單一資料結構中使用「粒紋」建立多個序列的模型。 粒紋是由 `store` 和 `brand` 欄所指定。

「粒紋」和「群組」之間的差異是，粒紋在真實世界中永遠具有實質上的意義，但群組則不一定。 如果使用者認為此群組可協助改善模型效能，內部套件函式就會使用群組，從多個時間序列建置單一模型。 根據預設，群組設定為等於粒紋，而且系統會為每個粒紋建置單一模型。 


```python
# Create a TimeSeriesDataFrame
# Use end of period as the time index
# Store and brand combinations label the grain 
# i.e. there is one time series for each unique pair of store and grain
whole_tsdf = TimeSeriesDataFrame(whole_df, 
                                 grain_colnames=['store', 'brand'],
                                 time_colname='WeekLastDay', 
                                 ts_value_colname='Quantity',
                                 group_colnames='store')

whole_tsdf[['Quantity']].head()
```




<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>數量</th>
    </tr>
    <tr>
      <th>WeekLastDay</th>
      <th>store</th>
      <th>brand</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1990-06-20 23:59:59</th>
      <th>2</th>
      <th>tropicana</th>
      <td>8256</td>
    </tr>
    <tr>
      <th>1990-08-01 23:59:59</th>
      <th>2</th>
      <th>tropicana</th>
      <td>6144</td>
    </tr>
    <tr>
      <th>1990-08-08 23:59:59</th>
      <th>2</th>
      <th>tropicana</th>
      <td>3840</td>
    </tr>
    <tr>
      <th>1990-08-15 23:59:59</th>
      <th>2</th>
      <th>tropicana</th>
      <td>8000</td>
    </tr>
    <tr>
      <th>1990-08-29 23:59:59</th>
      <th>2</th>
      <th>tropicana</th>
      <td>8896</td>
    </tr>
  </tbody>
</table>



在 TimeSeriesDataFrame 表示法中，時間軸和粒紋現在已納入資料框架索引，並可讓您輕鬆存取 pandas 日期時間配量功能。


```python
# sort so we can slice
whole_tsdf.sort_index(inplace=True)

# Get sales of dominick's brand orange juice from store 2 during summer 1990
whole_tsdf.loc[pd.IndexSlice['1990-06':'1990-09', 2, 'dominicks'], ['Quantity']]
```




<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>數量</th>
    </tr>
    <tr>
      <th>WeekLastDay</th>
      <th>store</th>
      <th>brand</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1990-06-20 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <td>10560</td>
    </tr>
    <tr>
      <th>1990-08-01 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <td>8000</td>
    </tr>
    <tr>
      <th>1990-08-08 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <td>6848</td>
    </tr>
    <tr>
      <th>1990-08-15 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <td>2880</td>
    </tr>
    <tr>
      <th>1990-08-29 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <td>1600</td>
    </tr>
    <tr>
      <th>1990-09-05 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <td>25344</td>
    </tr>
    <tr>
      <th>1990-09-12 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <td>10752</td>
    </tr>
    <tr>
      <th>1990-09-19 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <td>6656</td>
    </tr>
    <tr>
      <th>1990-09-26 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <td>6592</td>
    </tr>
  </tbody>
</table>



[TimeSeriesDataFrame.ts_report](https://docs.microsoft.com/python/api/ftk.dataframe_ts.timeseriesdataframe?view=azure-ml-py-latest#ts-report) 函式會產生時間序列資料框架的完整報告。 此報告同時包含一般資料描述，以及時間序列資料專用的統計資料。 


```python
whole_tsdf.ts_report()
```

    --------------------------------  Data Overview  ---------------------------------
    <class 'ftk.time_series_data_frame.TimeSeriesDataFrame'>
    MultiIndex: 28947 entries, (1990-06-20 23:59:59, 2, dominicks) to (1992-10-07 23:59:59, 137, tropicana)
    Data columns (total 17 columns):
    week            28947 non-null int64
    logmove         28947 non-null float64
    feat            28947 non-null int64
    price           28947 non-null float64
    AGE60           28947 non-null float64
    EDUC            28947 non-null float64
    ETHNIC          28947 non-null float64
    INCOME          28947 non-null float64
    HHLARGE         28947 non-null float64
    WORKWOM         28947 non-null float64
    HVAL150         28947 non-null float64
    SSTRDIST        28947 non-null float64
    SSTRVOL         28947 non-null float64
    CPDIST5         28947 non-null float64
    CPWVOL5         28947 non-null float64
    Quantity        28947 non-null int64
    WeekFirstDay    28947 non-null datetime64[ns]
    dtypes: datetime64[ns](1), float64(13), int64(3)
    memory usage: 3.8+ MB
    --------------------------  Numerical Variable Summary  --------------------------
              week  logmove     feat    price    AGE60     EDUC   ETHNIC   INCOME  \
    count 28947.00 28947.00 28947.00 28947.00 28947.00 28947.00 28947.00 28947.00   
    mean    100.46     9.17     0.24     2.28     0.17     0.23     0.16    10.62   
    std      34.69     1.02     0.43     0.65     0.06     0.11     0.19     0.28   
    min      40.00     4.16     0.00     0.52     0.06     0.05     0.02     9.87   
    25%      70.00     8.49     0.00     1.79     0.12     0.15     0.04    10.46   
    50%     101.00     9.03     0.00     2.17     0.17     0.23     0.07    10.64   
    75%     130.00     9.76     0.00     2.73     0.21     0.28     0.19    10.80   
    max     160.00    13.48     1.00     3.87     0.31     0.53     1.00    11.24   
    
           HHLARGE  WORKWOM  HVAL150  SSTRDIST  SSTRVOL  CPDIST5  CPWVOL5  \
    count 28947.00 28947.00 28947.00  28947.00 28947.00 28947.00 28947.00   
    mean      0.12     0.36     0.34      5.10     1.21     2.12     0.44   
    std       0.03     0.05     0.24      3.47     0.53     0.73     0.22   
    min       0.01     0.24     0.00      0.13     0.40     0.77     0.09   
    25%       0.10     0.31     0.12      2.77     0.73     1.63     0.27   
    50%       0.11     0.36     0.35      4.65     1.12     1.96     0.38   
    75%       0.14     0.40     0.53      6.65     1.54     2.53     0.56   
    max       0.22     0.47     0.92     17.86     2.57     4.11     1.14   
    
           Quantity  
    count  28947.00  
    mean   17312.21  
    std    27477.66  
    min       64.00  
    25%     4864.00  
    50%     8384.00  
    75%    17408.00  
    max   716416.00  
    ------------------------  Non-Numerical Variable Summary  -----------------------
                   WeekFirstDay
    count                 28947
    unique                  121
    top     1992-03-12 00:00:00
    freq                    249
    first   1990-06-14 00:00:00
    last    1992-10-01 00:00:00
    ------------------------------  Time Series Summary  -----------------------------
    Number of time series                 249
    Minimum time                    1990-06-20 23:59:59
    Maximum time                    1992-10-07 23:59:59
    
    Inferred frequencies
    Number of ['store', 'brand']s with frequency W-WED     249
    Use get_frequency_dict() method to explore ['store', 'brand']s with unusual frequency and clean data
    
    Detected seasonalities
    Number of ['store', 'brand']s with seasonality 1         190
    Number of ['store', 'brand']s with seasonality 15        15
    Number of ['store', 'brand']s with seasonality 14        11
    Number of ['store', 'brand']s with seasonality 7         9
    Number of ['store', 'brand']s with seasonality 6         8
    Number of ['store', 'brand']s with seasonality 8         5
    Number of ['store', 'brand']s with seasonality 2         4
    Number of ['store', 'brand']s with seasonality 23        2
    Number of ['store', 'brand']s with seasonality 3         1
    Number of ['store', 'brand']s with seasonality 11        1
    Number of ['store', 'brand']s with seasonality 12        1
    Number of ['store', 'brand']s with seasonality 13        1
    Number of ['store', 'brand']s with seasonality 47        1
    Use get_seasonality_dict() method to explore ['store', 'brand']s with unusual seasonality and clean data
    -----------------------------  Value Column Summary  -----------------------------
    Value column                        Quantity
    Percentage of missing values        0.00
    Percentage of zero values           0.00
    Mean coefficient of variation       31688.52
    Median coefficient of variation     24000.20
    Minimum coefficient of variation    ['store', 'brand'] (48, 'tropicana'): 4475.53
    Maximum coefficient of variation    ['store', 'brand'] (111, 'dominicks'): 193333.55
    ------------------------------  Correlation Matrix  ------------------------------
        week  logmove  feat  price  AGE60  EDUC  ETHNIC  INCOME  HHLARGE  WORKWOM  \
    0   1.00     0.10  0.04  -0.21  -0.01  0.01    0.00    0.00     0.01    -0.00   
    1   0.10     1.00  0.54  -0.43   0.09  0.00    0.06   -0.04    -0.06    -0.08   
    2   0.04     0.54  1.00  -0.29  -0.00  0.00    0.00   -0.00    -0.00     0.00   
    3  -0.21    -0.43 -0.29   1.00   0.04  0.02    0.04   -0.03    -0.04    -0.02   
    4  -0.01     0.09 -0.00   0.04   1.00 -0.31   -0.09   -0.15    -0.32    -0.63   
    5   0.01     0.00  0.00   0.02  -0.31  1.00   -0.34    0.66    -0.39     0.56   
    6   0.00     0.06  0.00   0.04  -0.09 -0.34    1.00   -0.72     0.25    -0.29   
    7   0.00    -0.04 -0.00  -0.03  -0.15  0.66   -0.72    1.00    -0.08     0.40   
    8   0.01    -0.06 -0.00  -0.04  -0.32 -0.39    0.25   -0.08     1.00    -0.28   
    9  -0.00    -0.08  0.00  -0.02  -0.63  0.56   -0.29    0.40    -0.28     1.00   
    10  0.01     0.02  0.00   0.04  -0.11  0.89   -0.42    0.64    -0.48     0.45   
    11  0.01    -0.00  0.00   0.08   0.07 -0.12    0.54   -0.41     0.06    -0.19   
    12 -0.01    -0.09 -0.00   0.03  -0.05 -0.13    0.23   -0.26     0.04    -0.06   
    13 -0.01     0.02 -0.00  -0.06   0.09 -0.20   -0.22    0.21     0.19    -0.13   
    14 -0.00    -0.12 -0.00  -0.01  -0.09  0.28   -0.38    0.36    -0.20     0.37   
    15  0.03     0.76  0.47  -0.36   0.08 -0.04    0.11   -0.08    -0.00    -0.10   
    
        HVAL150  SSTRDIST  SSTRVOL  CPDIST5  CPWVOL5  Quantity  
    0      0.01      0.01    -0.01    -0.01    -0.00      0.03  
    1      0.02     -0.00    -0.09     0.02    -0.12      0.76  
    2      0.00      0.00    -0.00    -0.00    -0.00      0.47  
    3      0.04      0.08     0.03    -0.06    -0.01     -0.36  
    4     -0.11      0.07    -0.05     0.09    -0.09      0.08  
    5      0.89     -0.12    -0.13    -0.20     0.28     -0.04  
    6     -0.42      0.54     0.23    -0.22    -0.38      0.11  
    7      0.64     -0.41    -0.26     0.21     0.36     -0.08  
    8     -0.48      0.06     0.04     0.19    -0.20     -0.00  
    9      0.45     -0.19    -0.06    -0.13     0.37     -0.10  
    10     1.00     -0.17    -0.24    -0.22     0.27     -0.04  
    11    -0.17      1.00     0.17    -0.11    -0.40      0.06  
    12    -0.24      0.17     1.00    -0.05     0.36     -0.02  
    13    -0.22     -0.11    -0.05     1.00     0.02     -0.00  
    14     0.27     -0.40     0.36     0.02     1.00     -0.11  
    15    -0.04      0.06    -0.02    -0.00    -0.11      1.00  
    You may call TimeSeriesDataFrame.plot_scatter_matrix() to get a correlation matrix plot. However, this
    is not recommended if your data is big.
    



![png](./media/how-to-build-deploy-forecast-models/output_15_1.png)



![png](./media/how-to-build-deploy-forecast-models/output_15_2.png)



![png](./media/how-to-build-deploy-forecast-models/output_15_3.png)



![png](./media/how-to-build-deploy-forecast-models/output_15_4.png)



![png](./media/how-to-build-deploy-forecast-models/output_15_5.png)



![png](./media/how-to-build-deploy-forecast-models/output_15_6.png)


## <a name="integrate-with-external-data"></a>與外部資料整合

將外部資料整合為用於預測的額外功能有時候很有用。 在此程式碼範例中，您可以將具有相關外部資料的 TimeSeriesDataFrame 加入 weather。


```python
# Load weather data
weather_1990 = get_a_year_of_daily_weather_data(year=1990)
weather_1991 = get_a_year_of_daily_weather_data(year=1991)
weather_1992 = get_a_year_of_daily_weather_data(year=1992)

# Preprocess weather data
weather_all = pd.concat([weather_1990, weather_1991, weather_1992])
weather_all.reset_index(inplace=True)

# Only use a subset of columns
weather_all = weather_all[['TEMP', 'DEWP', 'WDSP', 'PRCP']]

# Compute the WeekLastDay column, in order to merge with sales data
weather_all['WeekLastDay'] = pd.Series(
    weather_all.time_index - weekZeroStart, 
    index=weather_all.time_index).apply(lambda n: weekZeroEnd + timedelta(weeks=math.floor(n.days/7)))

# Resample daily weather data to weekly data
weather_all = weather_all.groupby('WeekLastDay').mean()

# Set WeekLastDay as new time index
weather_all = TimeSeriesDataFrame(weather_all, time_colname='WeekLastDay')

# Merge weather data with sales data
whole_tsdf = whole_tsdf.merge(weather_all, how='left', on='WeekLastDay')
whole_tsdf.head()
```




<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th>week</th>
      <th>logmove</th>
      <th>feat</th>
      <th>price</th>
      <th>AGE60</th>
      <th>EDUC</th>
      <th>ETHNIC</th>
      <th>INCOME</th>
      <th>HHLARGE</th>
      <th>WORKWOM</th>
      <th>...</th>
      <th>SSTRDIST</th>
      <th>SSTRVOL</th>
      <th>CPDIST5</th>
      <th>CPWVOL5</th>
      <th>數量</th>
      <th>WeekFirstDay</th>
      <th>TEMP</th>
      <th>DEWP</th>
      <th>WDSP</th>
      <th>PRCP</th>
    </tr>
    <tr>
      <th>WeekLastDay</th>
      <th>store</th>
      <th>brand</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">1990-06-20 23:59:59</th>
      <th rowspan="3" valign="top">2</th>
      <th>dominicks</th>
      <td>40</td>
      <td>9.26</td>
      <td>1</td>
      <td>1.59</td>
      <td>0.23</td>
      <td>0.25</td>
      <td>0.11</td>
      <td>10.55</td>
      <td>0.10</td>
      <td>0.30</td>
      <td>...</td>
      <td>2.11</td>
      <td>1.14</td>
      <td>1.93</td>
      <td>0.38</td>
      <td>10560</td>
      <td>1990-06-14</td>
      <td>72.00</td>
      <td>61.87</td>
      <td>9.74</td>
      <td>0.19</td>
    </tr>
    <tr>
      <th>minute.maid</th>
      <td>40</td>
      <td>8.41</td>
      <td>0</td>
      <td>3.17</td>
      <td>0.23</td>
      <td>0.25</td>
      <td>0.11</td>
      <td>10.55</td>
      <td>0.10</td>
      <td>0.30</td>
      <td>...</td>
      <td>2.11</td>
      <td>1.14</td>
      <td>1.93</td>
      <td>0.38</td>
      <td>4480</td>
      <td>1990-06-14</td>
      <td>72.00</td>
      <td>61.87</td>
      <td>9.74</td>
      <td>0.19</td>
    </tr>
    <tr>
      <th>tropicana</th>
      <td>40</td>
      <td>9.02</td>
      <td>0</td>
      <td>3.87</td>
      <td>0.23</td>
      <td>0.25</td>
      <td>0.11</td>
      <td>10.55</td>
      <td>0.10</td>
      <td>0.30</td>
      <td>...</td>
      <td>2.11</td>
      <td>1.14</td>
      <td>1.93</td>
      <td>0.38</td>
      <td>8256</td>
      <td>1990-06-14</td>
      <td>72.00</td>
      <td>61.87</td>
      <td>9.74</td>
      <td>0.19</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">5</th>
      <th>dominicks</th>
      <td>40</td>
      <td>7.49</td>
      <td>1</td>
      <td>1.59</td>
      <td>0.12</td>
      <td>0.32</td>
      <td>0.05</td>
      <td>10.92</td>
      <td>0.10</td>
      <td>0.41</td>
      <td>...</td>
      <td>3.80</td>
      <td>0.68</td>
      <td>1.60</td>
      <td>0.74</td>
      <td>1792</td>
      <td>1990-06-14</td>
      <td>72.00</td>
      <td>61.87</td>
      <td>9.74</td>
      <td>0.19</td>
    </tr>
    <tr>
      <th>minute.maid</th>
      <td>40</td>
      <td>8.35</td>
      <td>0</td>
      <td>2.99</td>
      <td>0.12</td>
      <td>0.32</td>
      <td>0.05</td>
      <td>10.92</td>
      <td>0.10</td>
      <td>0.41</td>
      <td>...</td>
      <td>3.80</td>
      <td>0.68</td>
      <td>1.60</td>
      <td>0.74</td>
      <td>4224</td>
      <td>1990-06-14</td>
      <td>72.00</td>
      <td>61.87</td>
      <td>9.74</td>
      <td>0.19</td>
    </tr>
  </tbody>
</table>


## <a name="preprocess-data-and-impute-missing-values"></a>預先處理資料，並插補遺漏的值

從使用 [last_n_periods_split](https://docs.microsoft.com/python/api/ftk.ts_utils?view=azure-ml-py-latest) 公用程式函式，將資料分割成定型集和測試集開始。 產生的測試集包含每個時間序列的最後 40 個觀察值。 


```python
train_tsdf, test_tsdf = last_n_periods_split(whole_tsdf, 40)
```

基本時間序列模型需要連續的時間序列。 請檢查序列是否規則，也就是說，它們是否使用 [check_regularity_by_grain](https://docs.microsoft.com/python/api/ftk.dataframe_ts.timeseriesdataframe?view=azure-ml-py-latest#check-regularity-by-grain) 函式，在固定間隔針對時間索引進行取樣。


```python
ts_regularity = train_tsdf.check_regularity_by_grain()
print(ts_regularity[ts_regularity['regular'] == False])
```

                                              problems  regular
    store brand                                                
    2     dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    5     dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    8     dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    9     dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    12    dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    14    dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    18    dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    21    dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    28    dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    33    dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    ...                                            ...      ...
    119   dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    121   dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    123   dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    126   dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    128   dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    129   dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    130   dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    131   dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    134   dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    137   dominicks    [Irregular datetime gaps exist]    False
          minute.maid  [Irregular datetime gaps exist]    False
          tropicana    [Irregular datetime gaps exist]    False
    
    [213 rows x 2 columns]
    

您可以看到大部分的序列 (249 個序列中有 213 個序列) 都不規則。 需要[插補轉換](https://docs.microsoft.com/python/api/ftk.transforms.ts_imputer.timeseriesimputer?view=azure-ml-py-latest)才能填入遺漏的銷售數量值。 雖然有許多插補選項，但下列範例程式碼會使用線性插補。


```python
# Use a TimeSeriesImputer to linearly interpolate missing values
imputer = TimeSeriesImputer(input_column='Quantity', 
                            option='interpolate',
                            method='linear',
                            freq='W-WED')

train_imputed_tsdf = imputer.transform(train_tsdf)
```

插補程式碼執行之後，所有數列都會有一個規則的頻率：


```python
ts_regularity_imputed = train_imputed_tsdf.check_regularity_by_grain()
print(ts_regularity_imputed[ts_regularity_imputed['regular'] == False])
```

    Empty DataFrame
    Columns: [problems, regular]
    Index: []
    

## <a name="univariate-time-series-models"></a>單一變量時間序列模型

清除資料之後，您就可以開始建立模型了。  從建立三個單一變量模型開始："naive" 模型、"seasonal naive" 模型和 "ARIMA" 模型。
* Naive 預測演算法使用最後一個期間的實際目標變數值作為目前期間的預測值。

* Seasonal Naive 演算法使用上一季相同時間點的實際目標變數值作為目前時間點的預測值。 其中一些範例包括使用去年相同月份的實際值來預測的目前年度的月份；使用昨天相同的時數來預測今天的時數。 

* 指數平滑 (ETS) 演算法會在觀察值變舊時，使用指數上衰退的加權計算過去觀察值的加權平均值，以產生預測。 

* 自動迴歸整合式移動平均 (ARIMA) 演算法會擷取時間序列資料中的自動相互關聯。 如需有關 ARIMA 的詳細資訊，請參閱[此連結](https://en.wikipedia.org/wiki/Autoregressive_integrated_moving_average)

從根據您的資料探索，設定特定模型參數開始。 


```python
oj_series_freq = 'W-WED'
oj_series_seasonality = 52
```

### <a name="initialize-models"></a>初始化模型


```python
# Initialize naive model.
naive_model = Naive(freq=oj_series_freq)

# Initialize seasonal naive model. 
seasonal_naive_model = SeasonalNaive(freq=oj_series_freq, 
                                     seasonality=oj_series_seasonality)

# Initialize ETS model.
ets_model = ETS(freq=oj_series_freq, seasonality=oj_series_seasonality)

# Initialize ARIMA(p,d,q) model.
arima_order = [2, 1, 0]
arima_model = Arima(oj_series_freq, arima_order)
```

### <a name="combine-multiple-models"></a>結合多個模型

[ForecasterUnion](https://docs.microsoft.com/python/api/ftk.models.forecaster_union?view=azure-ml-py-latest) 估算器可讓您結合多個估算器，並對其使用一行程式碼進行調整/預測。


```python
forecaster_union = ForecasterUnion(
    forecaster_list=[('naive', naive_model), ('seasonal_naive', seasonal_naive_model), 
                     ('ets', ets_model), ('arima', arima_model)]) 
```

### <a name="fit-and-predict"></a>調整和預測

AMLPF 中的估算器遵循與 scikit-learn 估算器相同的 API：適用於模型定型的調整方法以及用於產生預測的預測方法。 

**將模型定型**  
這些模型全都是單一變量模型，因此每個資料粒紋都符合一個模型。 使用 AMLPF 時，只要利用單一函式呼叫，就可以調整全部 249 個模型。


```python
forecaster_union_fitted = forecaster_union.fit(train_imputed_tsdf)
```

**根據測試資料預測銷售量**  
與調整方法類似，對 `predict` 函式使用一個呼叫，就可以為測試資料集中的全部 249 個序列建立預測。 


```python
forecaster_union_prediction = forecaster_union_fitted.predict(test_tsdf, retain_feature_column=True)
```

**評估模型效能**   

現在您可以計算測試集上的預測錯誤。 您可以在這裡使用平均絕對百分比誤差 (MAPE)。 MAPE 是相對於實際銷售值的平均絕對百分比誤差。 ```calc_error``` 函式會針對常用的誤差計量，提供幾個內建函式。 您也可以定義自訂錯誤函式來計算 MedianAPE，並將它傳遞至 err_fun 引數。


```python
def calc_median_ape(y_true, y_pred):
    y_true = np.array(y_true).astype(float)
    y_pred = np.array(y_pred).astype(float)
    y_true_rm_na = y_true[~(np.isnan(y_true) | np.isnan(y_pred))]
    y_pred_rm_na = y_pred[~(np.isnan(y_true) | np.isnan(y_pred))]
    y_true = y_true_rm_na
    y_pred = y_pred_rm_na
    if len(y_true) == 0:
        # if there is no entries left after removing na data, return np.nan
        return(np.nan)
    y_true_rm_zero = y_true[y_true != 0]
    y_pred_rm_zero = y_pred[y_true != 0]
    if len(y_true_rm_zero) == 0:
        # if all values are zero, np.nan will be returned.
        return(np.nan)
    ape = np.abs((y_true_rm_zero - y_pred_rm_zero) / y_true_rm_zero) * 100
    median_ape = np.median(ape)
    return median_ape
```


```python
forecaster_union_MAPE = forecaster_union_prediction.calc_error(err_name='MAPE',
                                                               by='ModelName')
forecaster_union_MedianAPE = forecaster_union_prediction.calc_error(err_name='MedianAPE', 
                                                                    err_fun=calc_median_ape,
                                                                    by='ModelName')

univariate_model_errors = forecaster_union_MAPE.merge(forecaster_union_MedianAPE, on='ModelName')
univariate_model_errors
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ModelName</th>
      <th>MAPE</th>
      <th>MedianAPE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>arima</td>
      <td>126.57</td>
      <td>66.49</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ets</td>
      <td>187.89</td>
      <td>75.73</td>
    </tr>
    <tr>
      <th>2</th>
      <td>naive</td>
      <td>103.57</td>
      <td>59.14</td>
    </tr>
    <tr>
      <th>3</th>
      <td>seasonal_naive</td>
      <td>180.54</td>
      <td>65.99</td>
    </tr>
  </tbody>
</table>



## <a name="build-machine-learning-models"></a>建置機器學習服務模型

除了傳統的單一變量模型之外，適用於預測的 Azure Machine Learning 套件也可以讓您建立機器學習模型。

針對這些模型，請從建立功能開始。

### <a name="feature-engineering"></a>特徵設計

**轉換器**   
套件針對時間序列資料預先處理和功能化，提供許多轉換器。 接下來的範例示範一些預先處理和功能化功能。


```python
# DropColumns: Drop columns that should not be included for modeling. `logmove` is the log of the number of 
# units sold, so providing this number would be cheating. `WeekFirstDay` would be 
# redundant since we already have a feature for the last day of the week.
columns_to_drop = ['logmove', 'WeekFirstDay', 'week']
column_dropper = DropColumns(columns_to_drop)

# TimeSeriesImputer: Fill missing values in the features
# First, we need to create a dictionary with key as column names and value as values used to fill missing 
# values for that column. We are going to use the mean to fill missing values for each column.
columns_with_missing_values = train_imputed_tsdf.columns[pd.DataFrame(train_imputed_tsdf).isnull().any()].tolist()
columns_with_missing_values = [c for c in columns_with_missing_values if c not in columns_to_drop]
missing_value_imputation_dictionary = {}
for c in columns_with_missing_values:
    missing_value_imputation_dictionary[c] = train_imputed_tsdf[c].mean()
fillna_imputer = TimeSeriesImputer(option='fillna', 
                                   input_column=columns_with_missing_values,
                                   value=missing_value_imputation_dictionary)

# TimeIndexFeaturizer: extract temporal features from timestamps
time_index_featurizer = TimeIndexFeaturizer(correlation_cutoff=0.1, overwrite_columns=True)

# GrainIndexFeaturizer: create indicator variables for stores and brands
grain_featurizer = GrainIndexFeaturizer(overwrite_columns=True, ts_frequency=oj_series_freq)
```

**管線**   
管線物件可讓您輕鬆儲存一組步驟，以便可以將這些步驟一再重複套用至不同的物件。 此外，也可以將管線物件序列化，讓這些物件可以輕鬆地移植到其他電腦上進行部署。 您可以使用管線，鏈結目前為止所建立的所有轉換器。 


```python
pipeline_ml = AzureMLForecastPipeline([('drop_columns', column_dropper), 
                                       ('fillna_imputer', fillna_imputer),
                                       ('time_index_featurizer', time_index_featurizer),
                                       ('grain_featurizer', grain_featurizer)
                                      ])


train_feature_tsdf = pipeline_ml.fit_transform(train_imputed_tsdf)
test_feature_tsdf = pipeline_ml.transform(test_tsdf)

# Let's get a look at our new feature set
print(train_feature_tsdf.head())
```

    F1 2018-06-14 23:10:03,472 INFO azureml.timeseries - pipeline fit_transform started. 
    F1 2018-06-14 23:10:07,317 INFO azureml.timeseries - pipeline fit_transform finished. Time elapsed 0:00:03.845078
    F1 2018-06-14 23:10:07,317 INFO azureml.timeseries - pipeline transforms started. 
    F1 2018-06-14 23:10:16,499 INFO azureml.timeseries - pipeline transforms finished. Time elapsed 0:00:09.182314
                                           feat  price  AGE60  EDUC  ETHNIC  \
    WeekLastDay         store brand                                           
    1990-06-20 23:59:59 2     dominicks    1.00   1.59   0.23  0.25    0.11   
                              minute.maid  0.00   3.17   0.23  0.25    0.11   
                              tropicana    0.00   3.87   0.23  0.25    0.11   
                        5     dominicks    1.00   1.59   0.12  0.32    0.05   
                              minute.maid  0.00   2.99   0.12  0.32    0.05   
    
                                           INCOME  HHLARGE  WORKWOM  HVAL150  \
    WeekLastDay         store brand                                            
    1990-06-20 23:59:59 2     dominicks     10.55     0.10     0.30     0.46   
                              minute.maid   10.55     0.10     0.30     0.46   
                              tropicana     10.55     0.10     0.30     0.46   
                        5     dominicks     10.92     0.10     0.41     0.54   
                              minute.maid   10.92     0.10     0.41     0.54   
    
                                           SSTRDIST     ...       CPWVOL5  \
    WeekLastDay         store brand                     ...                 
    1990-06-20 23:59:59 2     dominicks        2.11     ...          0.38   
                              minute.maid      2.11     ...          0.38   
                              tropicana        2.11     ...          0.38   
                        5     dominicks        3.80     ...          0.74   
                              minute.maid      3.80     ...          0.74   
    
                                           Quantity  TEMP  DEWP  WDSP  PRCP  year  \
    WeekLastDay         store brand                                                 
    1990-06-20 23:59:59 2     dominicks    10560.00 72.00 61.87  9.74  0.19  1990   
                              minute.maid   4480.00 72.00 61.87  9.74  0.19  1990   
                              tropicana     8256.00 72.00 61.87  9.74  0.19  1990   
                        5     dominicks     1792.00 72.00 61.87  9.74  0.19  1990   
                              minute.maid   4224.00 72.00 61.87  9.74  0.19  1990   
    
                                           day  grain_brand  grain_store  
    WeekLastDay         store brand                                       
    1990-06-20 23:59:59 2     dominicks     20    dominicks            2  
                              minute.maid   20  minute.maid            2  
                              tropicana     20    tropicana            2  
                        5     dominicks     20    dominicks            5  
                              minute.maid   20  minute.maid            5  
    
    [5 rows x 22 columns]
    

 **RegressionForecaster**

[RegressionForecaster](https://docs.microsoft.com/python/api/ftk.models.regression_forecaster.regressionforecaster?view=azure-ml-py-latest) 函式會包裝 sklearn 迴歸估算器，以便針對 TimeSeriesDataFrame 進行定型。 已包裝的預測器還可以將每個群組 (在此案例中為 store) 放入相同的模型。 預測器可以針對被視為類似，而且可以集中在一起的一組序列，學習一種模型。 適用於一組序列的一個模型通常會使用較長序列中的資料來改善短序列的預測。 您可以將這些模型取代為程式庫中支援迴歸的其他任何模型。 


```python
lasso_model = RegressionForecaster(estimator=Lasso(),
                                   make_grain_features=False)
elastic_net_model = RegressionForecaster(estimator=ElasticNet(),
                                         make_grain_features=False)
knn_model = RegressionForecaster(estimator=KNeighborsRegressor(),
                                 make_grain_features=False)
random_forest_model = RegressionForecaster(estimator=RandomForestRegressor(),
                                           make_grain_features=False)
boosted_trees_model = RegressionForecaster(estimator=GradientBoostingRegressor(),
                                           make_grain_features=False)

ml_union = ForecasterUnion(forecaster_list=[
    ('lasso', lasso_model), 
    ('elastic_net', elastic_net_model), 
    ('knn', knn_model), 
    ('random_forest', random_forest_model), 
    ('boosted_trees', boosted_trees_model)
]) 
```


```python
ml_union.fit(train_feature_tsdf, y=train_feature_tsdf.ts_value)
ml_results = ml_union.predict(test_feature_tsdf, retain_feature_column=True)
```


```python
ml_model_MAPE = ml_results.calc_error(err_name='MAPE', by='ModelName')
ml_model_MedianAPE = ml_results.calc_error(err_name='MedianAPE', 
                                           err_fun=calc_median_ape,
                                           by='ModelName')
ml_model_errors = ml_model_MAPE.merge(ml_model_MedianAPE, on='ModelName')
all_errors = pd.concat([univariate_model_errors, ml_model_errors])
all_errors.sort_values('MedianAPE')
```





<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ModelName</th>
      <th>MAPE</th>
      <th>MedianAPE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>random_forest</td>
      <td>78.82</td>
      <td>42.81</td>
    </tr>
    <tr>
      <th>0</th>
      <td>boosted_trees</td>
      <td>78.46</td>
      <td>45.37</td>
    </tr>
    <tr>
      <th>2</th>
      <td>naive</td>
      <td>103.57</td>
      <td>59.14</td>
    </tr>
    <tr>
      <th>2</th>
      <td>knn</td>
      <td>129.85</td>
      <td>65.37</td>
    </tr>
    <tr>
      <th>1</th>
      <td>elastic_net</td>
      <td>125.11</td>
      <td>65.59</td>
    </tr>
    <tr>
      <th>3</th>
      <td>seasonal_naive</td>
      <td>180.54</td>
      <td>65.99</td>
    </tr>
    <tr>
      <th>0</th>
      <td>arima</td>
      <td>126.57</td>
      <td>66.49</td>
    </tr>
    <tr>
      <th>3</th>
      <td>lasso</td>
      <td>112.87</td>
      <td>67.92</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ets</td>
      <td>187.89</td>
      <td>75.73</td>
    </tr>
  </tbody>
</table>



部分機器學習模型可以充分利用新增的功能以及序列之間的相似性，取得更好的預測精確度。

### <a name="cross-validation-parameter-and-model-sweeping"></a>交叉驗證、參數和模型掃掠    

此套件會針對預測應用，遷就一些傳統的機器學習函式。  [RollingOriginValidator](https://docs.microsoft.com/python/api/ftk.model_selection.cross_validation.rollingoriginvalidator?view=azure-ml-py-latest) 會短暫執行交叉驗證，並遵守預測框架中將會知道以及將不會知道的內容。 

在下圖中，每個方塊都代表一個時間點的資料。 藍色方塊代表定型，而橘色方塊則代表每個折疊中的測試。 測試資料必須來自最大定型時間點之後的時間點。 否則，未來的資料會洩漏到定型資料，導致模型評估變成無效。 
![png](./media/how-to-build-deploy-forecast-models/cv_figure.PNG)

**參數掃掠**  
[TSGridSearchCV](https://docs.microsoft.com/python/api/ftk.model_selection.search.tsgridsearchcv?view=azure-ml-py-latest) 類別會徹底搜尋指定的參數值，並使用 `RollingOriginValidator` 來評估參數效能，以找出最佳參數。


```python
# Set up the `RollingOriginValidator` to do 2 folds of rolling origin cross-validation
rollcv = RollingOriginValidator(n_splits=2)
randomforest_model_for_cv = RegressionForecaster(estimator=RandomForestRegressor(),
                                                 make_grain_features=False)

# Set up our parameter grid and feed it to our grid search algorithm
param_grid_rf = {'estimator__n_estimators': np.array([10, 50, 100])}
grid_cv_rf = TSGridSearchCV(randomforest_model_for_cv, param_grid_rf, cv=rollcv)

# fit and predict
randomforest_cv_fitted= grid_cv_rf.fit(train_feature_tsdf, y=train_feature_tsdf.ts_value)
print('Best paramter: {}'.format(randomforest_cv_fitted.best_params_))
```

    Best paramter: {'estimator__n_estimators': 100}
    

**模型掃掠**  
`BestOfForecaster` 類別會從指定模型清單中選取效能最好的模型。 類似於 `TSGridSearchCV`，它也會使用 RollingOriginValidator 來進行交叉驗證和效能評估。  
我們在這裡提供兩個模型，以示範 `BestOfForecaster` 的使用方式


```python
best_of_forecaster = BestOfForecaster(forecaster_list=[('naive', naive_model), 
                                                       ('random_forest', random_forest_model)])
best_of_forecaster_fitted = best_of_forecaster.fit(train_feature_tsdf,
                                                   validator=RollingOriginValidator(n_step=20, max_horizon=40))
best_of_forecaster_prediction = best_of_forecaster_fitted.predict(test_feature_tsdf)
best_of_forecaster_prediction.head()
```




<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>PointForecast</th>
      <th>DistributionForecast</th>
      <th>數量</th>
    </tr>
    <tr>
      <th>WeekLastDay</th>
      <th>store</th>
      <th>brand</th>
      <th>ForecastOriginTime</th>
      <th>ModelName</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1992-01-08 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <th>1992-01-01 23:59:59</th>
      <th>random_forest</th>
      <td>9299.20</td>
      <td>&lt;scipy.stats._distn_infrastructure.rv_frozen o...</td>
      <td>11712.00</td>
    </tr>
    <tr>
      <th>1992-01-15 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <th>1992-01-01 23:59:59</th>
      <th>random_forest</th>
      <td>10259.20</td>
      <td>&lt;scipy.stats._distn_infrastructure.rv_frozen o...</td>
      <td>4032.00</td>
    </tr>
    <tr>
      <th>1992-01-22 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <th>1992-01-01 23:59:59</th>
      <th>random_forest</th>
      <td>6828.80</td>
      <td>&lt;scipy.stats._distn_infrastructure.rv_frozen o...</td>
      <td>6336.00</td>
    </tr>
    <tr>
      <th>1992-01-29 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <th>1992-01-01 23:59:59</th>
      <th>random_forest</th>
      <td>16633.60</td>
      <td>&lt;scipy.stats._distn_infrastructure.rv_frozen o...</td>
      <td>13632.00</td>
    </tr>
    <tr>
      <th>1992-02-05 23:59:59</th>
      <th>2</th>
      <th>dominicks</th>
      <th>1992-01-01 23:59:59</th>
      <th>random_forest</th>
      <td>12774.40</td>
      <td>&lt;scipy.stats._distn_infrastructure.rv_frozen o...</td>
      <td>45120.00</td>
    </tr>
  </tbody>
</table>



**建置最終的管線**   
識別出最佳模型之後，就可以使用所有轉換器與最佳模型建置並調整您的最終管線。 


```python
random_forest_model_final = RegressionForecaster(estimator=RandomForestRegressor(100),make_grain_features=False)
pipeline_ml.add_pipeline_step('random_forest_estimator', random_forest_model_final)
pipeline_ml_fitted = pipeline_ml.fit(train_imputed_tsdf)
final_prediction = pipeline_ml_fitted.predict(test_tsdf)
final_median_ape = final_prediction.calc_error(err_name='MedianAPE', err_fun=calc_median_ape)
print('Median of APE of final pipeline: {0}'.format(final_median_ape))
```

    F1 2018-05-04 11:07:04,108 INFO azureml.timeseries - pipeline fit started. 
    F1 2018-05-04 11:07:43,121 INFO azureml.timeseries - pipeline fit finished. Time elapsed 0:00:39.012880
    F1 2018-05-04 11:07:43,136 INFO azureml.timeseries - pipeline predict started. 
    F1 2018-05-04 11:08:03,564 INFO azureml.timeseries - pipeline predict finished. Time elapsed 0:00:20.428147
    Median of APE of final pipeline: 42.54336821266968
    

## <a name="visualization"></a>視覺效果
`ForecastDataFrame` 類別會提供繪圖函式，以進行視覺化及分析預測結果。 使用您最常與資料搭配使用的圖表。 請參閱以下的範例筆記本，了解所有可用函式的繪圖函式。 

`show_error` 函式會繪製任意資料行所彙總的效能計量。 根據預設，`show_error` 函式會由 `ForecastDataFrame` 的 `grain_colnames` 進行彙總。 利用最佳或最差的效能來識別粒紋/群組通常很有用，特別是當您有大量的時間序列時。 `show_error` 的 `performance_percent` 引數可讓您指定效能間隔，並繪製粒紋/群組的子集錯誤。

利用末 5% 效能來繪製粒紋，也就是前 5 % MedianAPE


```python
fig, ax = best_of_forecaster_prediction.show_error(err_name='MedianAPE', err_fun=calc_median_ape, performance_percent=(0.95, 1))
```

![png](./media/how-to-build-deploy-forecast-models/output_59_0.png)


利用前 5% 效能來繪製粒紋，也就是末 5 % MedianAPE。


```python
fig, ax = best_of_forecaster_prediction.show_error(err_name='MedianAPE', err_fun=calc_median_ape, performance_percent=(0, 0.05))
```


![png](./media/how-to-build-deploy-forecast-models/output_61_0.png)


了解整體效能之後，您可以瀏覽個別的粒紋，特別是效能很差的粒紋。 `plot_forecast_by_grain` 方法可繪製預測與實際的指定粒紋。 在這裡，我們會繪製具有最佳效能的粒紋，以及在 `show_error` 繪圖中探索到具有最差效能的粒紋。


```python
fig_ax = best_of_forecaster_prediction.plot_forecast_by_grain(grains=[(33, 'tropicana'), (128, 'minute.maid')])
```


![png](./media/how-to-build-deploy-forecast-models/output_63_0.png)



![png](./media/how-to-build-deploy-forecast-models/output_63_1.png)



## <a name="additional-notebooks"></a>其他筆記本
如需更深入探討 AMLPF 主要功能，請參閱下列各項 Notebook，其中包含更多詳細資訊和每個功能的範例：  
[關於 TimeSeriesDataFrame 的筆記本](https://azuremlftkrelease.blob.core.windows.net/samples/feature_notebooks/Introduction_to_TimeSeriesDataFrames.ipynb)  
[關於 Data Wrangling 的筆記本](https://azuremlftkrelease.blob.core.windows.net/samples/feature_notebooks/Data_Wrangling_Sample.ipynb)  
[關於 Transformers 的筆記本](https://azuremlftkrelease.blob.core.windows.net/samples/feature_notebooks/Forecast_Package_Transforms.ipynb)  
[關於 Models 模型的筆記本](https://azuremlftkrelease.blob.core.windows.net/samples/feature_notebooks/AMLPF_models_sample_notebook.ipynb)  
[關於 Cross Validation 的筆記本](https://azuremlftkrelease.blob.core.windows.net/samples/feature_notebooks/Time_Series_Cross_Validation.ipynb)  
[關於 Lag Transformer 和 OriginTime 的筆記本](https://azuremlftkrelease.blob.core.windows.net/samples/feature_notebooks/Constructing_Lags_and_Explaining_Origin_Times.ipynb)  
[關於 Plotting Functions 的筆記本](https://azuremlftkrelease.blob.core.windows.net/samples/feature_notebooks/Plotting_Functions_in_AMLPF.ipynb)

## <a name="operationalization"></a>運算化

在本節中，您要將管線部署為 Azure Machine Learning Web 服務，然後將其用於定型和計分。
目前，僅支援以未調整的管線進行部署。 為已部署的 Web 服務計分可重新定型模型，並針對新資料產生預測。

### <a name="set-model-deployment-parameters"></a>設定模型部署參數

將下列參數變更為您自己的值。 請確定您的 Azure Machine Learning 環境、模型管理帳戶和資源群組都位於相同區域。


```python
azure_subscription = '<subscription name>'

# Two deployment modes are supported: 'local' and 'cluster'. 
# 'local' deployment deploys to a local docker container.
# 'cluster' deployment deploys to a Azure Container Service Kubernetes-based cluster
deployment_type = '<deployment mode>'

# The deployment environment name. 
# This could be an existing environment or a new environment to be created automatically.
aml_env_name = '<deployment env name>'

# The resource group that contains the Azure resources related to the AML environment.
aml_env_resource_group = '<env resource group name>'

# The location where the Azure resources related to the AML environment are located at.
aml_env_location = '<env resource location>'

# The AML model management account name. This could be an existing model management account a new model management 
# account to be created automatically. 
model_management_account_name = '<model management account name>'

# The resource group that contains the Azure resources related to the model management account.
model_management_account_resource_group = '<model management account resource group>'

# The location where the Azure resources related to the model management account are located at.
model_management_account_location = '<model management account location>'

# The name of the deployment/web service.
deployment_name = '<web service name>'

# The directory to store deployment related files, such as pipeline pickle file, score script, 
# and conda dependencies file. 
deployment_working_directory = '<local working directory>'
```

### <a name="define-the-azure-machine-learning-environment-and-deployment"></a>定義 Azure Machine Learning 環境和部署


```python
aml_settings = AMLSettings(azure_subscription=azure_subscription,
                     env_name=aml_env_name, 
                     env_resource_group=aml_env_resource_group,
                     env_location=aml_env_location, 
                     model_management_account_name=model_management_account_name, 
                     model_management_account_resource_group=model_management_account_resource_group,
                     model_management_account_location=model_management_account_location,
                     cluster=deployment_type)

random_forest_model_deploy = RegressionForecaster(estimator=RandomForestRegressor(),make_grain_features=False)
pipeline_deploy = AzureMLForecastPipeline([('drop_columns', column_dropper), 
                                           ('fillna_imputer', fillna_imputer),
                                           ('time_index_featurizer', time_index_featurizer),
                                           ('random_forest_estimator', random_forest_model_deploy)
                                          ])

aml_deployment = ForecastWebserviceFactory(deployment_name=deployment_name,
                                           aml_settings=aml_settings, 
                                           pipeline=pipeline_deploy,
                                           deployment_working_directory=deployment_working_directory,
                                           ftk_wheel_loc='https://azuremlftkrelease.blob.core.windows.net/dailyrelease/azuremlftk-0.1.18165.29a1-py3-none-any.whl')
```

### <a name="create-the-web-service"></a>建立 Web 服務


```python
# This step can take 5 to 20 minutes
aml_deployment.deploy()
```

### <a name="score-the-web-service"></a>為 Web 服務計分

若要為小型資料集計分，請使用 [score](https://docs.microsoft.com/python/api/ftk.operationalization.deployment.amlwebservice) 方法，為所有資料提交一個 Web 服務呼叫。


```python
# Need to add empty prediction columns to the validation data frame and create a ForecastDataFrame.
# The scoring API will be updated in later versions to take TimeSeriesDataFrame directly. 
validate_tsdf = test_tsdf.assign(PointForecast=0.0, DistributionForecast=np.nan)
validate_fcast = ForecastDataFrame(validate_tsdf, pred_point='PointForecast', pred_dist='DistributionForecast')

# Define Score Context
score_context = ScoreContext(input_training_data_tsdf=train_imputed_tsdf,
                             input_scoring_data_fcdf=validate_fcast, 
                             pipeline_execution_type='train_predict')

# Get deployed web service
aml_web_service = aml_deployment.get_deployment()

# Score the web service
results = aml_web_service.score(score_context=score_context)
```

若要為大型資料集計分，請使用[平行計分](https://docs.microsoft.com/python/api/ftk.operationalization.deployment.amlwebservice)模式提交多個 Web 服務呼叫，每組資料一個呼叫。


```python
results = aml_web_service.score(score_context=score_context, method='parallel')
```

## <a name="next-steps"></a>後續步驟

在這些文章中，深入了解適用於預測的 Azure Machine Learning 套件：

+ 參閱[套件概觀](https://aka.ms/aml-packages/forecasting) \(英文\)。

+ 探索此套件的[參考文件](https://aka.ms/aml-packages/forecasting)。

+ 了解[適用於 Azure Machine Learning 的其他 Python 套件](reference-python-package-overview.md)。
