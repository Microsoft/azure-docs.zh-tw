---
title: 使用 Machine Learning Studio 中的範例資料集 | Microsoft Docs
description: Machine Learning Studio 中包含的範例模型所使用之資料集的描述。 您可以為您的實驗使用這些範例資料集。
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: 03a0b844-e8a7-4896-996f-d3c7a0db7a50
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/19/2018
ms.openlocfilehash: 7a90a8f05082e2c0731f9f112d3e56ecaf4ea55b
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39398796"
---
# <a name="use-the-sample-datasets-in-azure-machine-learning-studio"></a>在 Azure Machine Learning Studio 中使用範例資料集
[top]: #machine-learning-sample-datasets

在 Azure Machine Learning 中建立新的工作區時，預設會包含一些範例資料集和實驗。 其中許多範例資料集由 [Azure AI 資源庫](http://gallery.cortanaintelligence.com/)中的範例模型使用。 其他則是機器學習中經常使用的各種範例。

其中的部分資料集可在 Azure Blob 儲存體中使用。 下表提供這些資料集的直接連結。 您也可以透過[匯入資料][import-data]模組在實驗中使用這些資料集。

在**儲存的資料集**之下的工作區中可使用這些範例資料集的其餘部分。 您可以在 Machine Learning Studio 的實驗畫布左邊的模組調色盤中找到這個。
您可以將資料集拖曳到您的實驗畫布上，在自己的實驗中使用任一資料集。


[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

<table>

<tr>
  <th>資料集名稱</th>
  <th>資料集說明</th>
</tr>

<tr>
  <td>成人收入普查二進位分類資料集</td>
  <td>
1994 年普查資料的子集，使用年齡 16 歲以上的成年工作者，和 > 100 的調整收入指數。
<p></p>
<b>使用方式：</b>使用人口統計來將人口分類，以預測個人年收入是否超過 5 萬元。
<p></p>
<b>相關研究：</b>Kohavi, R.、Becker, B. (1996 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>機場代碼資料集</td>
  <td>
美國機場代碼。
<p></p>
此資料集包含一列各美國機場的資料，以提供機場 ID 號碼和名稱以及所在的城市和州別。
  </td>
</tr>

<tr>
  <td>汽車價格資料 (原始)</td>
  <td>
依構造和型號分類的汽車相關資訊，包括價格、性能 (例如汽缸數和油耗)，以及保險風險評分。
<p></p>
風險分數一開始與自動價格相關聯。 然後它會在精算師稱為符號化的程序中根據實際風險進行調整。 若值為 +3，表示該汽車屬於高風險，若值為 -3，表示大致而言很安全。
<p></p>
<b>使用方式：</b>使用迴歸或多變量分類，依特性預測風險評分。 
<p></p>
<b>相關研究：</b>Schlimmer, J.C. (1987 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>單車出租 UCI 資料集</td>
  <td>
根據在華盛頓特區維護單車出租網路的 Capital Bikeshare 公司所提供的實際資料而建立的「UCI 單車出租」資料集。
<p></p>
在此資料集中，2011 和 2012 年每一天的每個小時分別有一個資料列，共計 17,379 個資料列。 每小時的單車出租量介於 1 到 977 部之間。

  </td>
</tr>

<tr>
  <td>Bill Gates RGB 影像</td>
  <td>
已轉換為 CSV 資料的公開可用映像檔案。
<p></p>
<strong>使用 K-Means 分群法的色彩量化</strong>模型詳細資料頁面會提供用於轉換映像的程式碼。
  </td>
</tr>

<tr>
  <td>捐血資料</td>
  <td>
台灣省新竹市捐血服務中心的捐血資料庫包含的資料子集。
<p></p>
捐血資料包括前次捐血後經過的月數、頻率、或捐血總次數、前次捐血後經過的時間，以及捐血的數量。
<p></p>
<b>使用方式：</b>目標是透過分類來預測捐血者是否曾在 2007 年 3 月份捐血；1 表示目標期間內有 1 名捐血者，0 表示沒有捐血者。 
<p></p>
<b>相關研究：</b>Yeh, I.C. (2008 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science
<p></p>
Yeh, I-Cheng, Yang, King-Jang, and Ting, Tao-Ming, "Knowledge discovery on RFM model using Bernoulli sequence, "Expert Systems with Applications, 2008, <a href="http://dx.doi.org/10.1016/j.eswa.2008.07.018">http://dx.doi.org/10.1016/j.eswa.2008.07.018</a>
  </td>
</tr>

<tr>
  <td>乳癌資料</td>
  <td>
Oncology Institute 所提供的三個癌症相關資料集之一，常出現在機器學習服務文獻中。 結合診斷資訊與約 300 個生物組織樣本的實驗室分析中的特性。
<p></p>
<b>使用方式：</b>根據 9 個屬性 (有些是線性，有些則為類別) 將癌症類型分類。 
<p></p>
<b>相關研究：</b>Wohlberg, W.H.、Street, W.N. 和 Mangasarian, O.L. (1995 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>乳癌特徵 <td>
資料集包含 10 萬 2 千個 X 光影像可疑區域 (候選項目) 的資訊，以 117 個特徵描述各項。 特徵為專屬資訊，並且資料集建立者 (Siemens Healthcare) 不會揭露其意義。 
  </td>
</tr>

<tr>
  <td>乳癌資訊</td>
  <td>
資料集包含 X 光影像每個可疑區域的其他資訊。 每個範例可提供與乳癌特徵功能集中對應資料列號碼的相關資訊 (例如：標籤、病患識別碼、與整個影像相關的修補座標)。 每個病患有一些範例。 對於有癌症的病患，一些範例是正數，而一些是負數。 對於沒有癌症的病患，所有範例都是負數。 資料集有 10 萬 2 千個範例。 資料集有偏差，0.6% 的點是正數，其餘的是負數。 資料集是由 Siemens Healthcare 提供。
  </td>
</tr>

<tr>
  <td>共用 CRM Appetency 標籤</td>
  <td>
KDD Cup 2009 客戶關係預測挑戰 (<a href="http://www.sigkdd.org/site/2009/files/orange_small_train_appetency.labels">orange_small_train_appetency.labels</a>) 中的標籤。
  </td>
</tr>

<tr>
  <td>共用 CRM 流失標籤</td>
  <td>
KDD Cup 2009 客戶關係預測挑戰 (<a href="http://www.sigkdd.org/site/2009/files/orange_small_train_churn.labels">orange_small_train_churn.labels</a>) 中的標籤。
  </td>
</tr>

<tr>
  <td>共用 CRM 資料集</td>
  <td>
此資料來自 KDD Cup 2009 客戶關係預測挑戰 (<a href="http://www.sigkdd.org/site/2009/files/orange_small_train.data.zip">orange_small_train.data.zip</a>)。
<p></p>
資料集包含來自法國電信公司 Orange 的 50K 個客戶。 每個客戶都有 230 項不具名的特性，其中有 190 項數值特性和 40 項類別特性。 這些特性非常稀疏。
  </td>
</tr>

<tr>
  <td>共用 CRM 向上銷售標籤</td>
  <td>
KDD Cup 2009 客戶關係預測挑戰 (<a href="http://www.sigkdd.org/site/2009/files/orange_large_train_upselling.labels">orange_large_train_upselling.labels</a>) 中的標籤。
  </td>
</tr>

<tr>
  <td>能量效益迴歸資料</td>
  <td>
模擬能量分佈曲線的集合，以 12 種不同的建築形狀為基礎。 組建分為八種功能。 這包括玻璃區域、玻璃區域分佈和方向。
<p></p>
<b>使用方式：</b>使用迴歸或分類，根據兩個實數值回應的其中之一來預測能量效益評等。 就多類別分類而言，會將回應變數進位至最接近的整數。 
<p></p>
<b>相關研究：</b>Xifara, A. 和 Tsanas, A.(2012 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>航班誤點資料</td>
  <td>
美國交通部收集的 TranStats 資料所包含的客機航班準點率資料(<a href="http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time">準點</a>)。
<p></p>
此資料集涵蓋的其間為 2013 年 4 月至 10 月。 上傳至 Azure Machine Learning Studio 之前，資料集已經過下列處理：
<ul>
  <li>資料集已經過篩選，僅涵蓋美國大陸 70 個最繁忙的機場</li>
  <li>取消的航班已標示為誤點達 15 分鐘以上</li>
  <li>已篩選掉更改路徑的航班</li>
  <li>已選取下列資料行：Year、Month、DayofMonth、DayOfWeek、Carrier、OriginAirportID、DestAirportID、CRSDepTime、DepDelay、DepDel15、CRSArrTime、ArrDelay、ArrDel15、Canceled</li>
</ul>
</td>
</tr>

<tr>
  <td>航班準點率 (原始)</td>
  <td>
2011 年 10 月份美國境內飛機航班起降記錄。
<p></p>
<b>使用方式：</b>預測航班誤點。 
<p></p>
<b>相關研究：</b>來自美國運輸部 <a href="http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time">http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time</a>。
  </td>
</tr>

<tr>
  <td>森林火災資料</td>
  <td>
包含天氣資料，例如氣溫和溼度索引以及風速。 資料來自葡萄牙東北部區域，結合森林火災的記錄。
<p></p>
<b>使用方式：</b>這是一項困難的迴歸工作，其目的是要預測森林火災的燃燒區域。 
<p></p>
<b>相關研究：</b>Cortez, P. 和 Morais, A.(2008 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science
<p></p>
[Cortez and Morais, 2007] P. Cortez and A. Morais。 一個使用氣象資料預測森林火災的資料採礦方法。 In J. Neves, M. F. Santos 與 J. Machado 編著，New Trends in Artificial Intelligence (人工智慧的新趨勢)，第 13 屆 EPIA 2007 年會議論文集 - 葡萄牙人工智慧會議，12 月，葡萄牙吉馬良斯，第 512-523 頁，2007 年。 APPIA, ISBN-13 978-989-95618-0-9. 可在下列地方獲取：<a href="http://www.dsi.uminho.pt/~pcortez/fires.pdf">http://www.dsi.uminho.pt/~pcortez/fires.pdf</a>。
  </td>
</tr>

<tr>
  <td>德國信用卡 UCI 資料集</td>
  <td>
UCI Statlog (德國信用卡) 資料集 (<a href="http://archive.ics.uci.edu/ml/datasets/Statlog+(German+Credit+Data)">Statlog+German+Credit+Data</a>)，使用 german.data 檔案。
<p></p>
此資料集會將申請者 (以一組屬性說明) 分類為低或高信用風險。 每個範例代表一名申請者。 共有 20 項特性 (包括數值和類別) 以及一個二進位標籤 (信用風險值)。 高信用風險項目的標籤 = 2，低信用風險項目的標籤 = 1。 將低風險範例誤判為高風險的成本為 1，而將高風險範例誤判為低風險的成本為 5。
  </td>
</tr>

<tr>
  <td>IMDB 影片標題</td>
  <td>
資料集包含 Twitter 推文中分級影片的資訊：IMDB 影片識別碼、影片名稱、內容類型、製作年份。 資料集中有 1 萬 7 千部影片。 該資料集在論文 "S. Dooms, T. De Pessemier and L. Martens" 中推出。 MovieTweetings：收集自 Twitter 的影片分級資料集。 Workshop on Crowdsourcing and Human Computation for Recommender Systems, CrowdRec at RecSys 2013."
  </td>
</tr>

<tr>
  <td>Iris 雙類別資料</td>
  <td>
這或許是型態辨識文獻中最著名的資料庫。 此資料集並不大，內含 50 個範例，說明三個鳶尾花品種的花瓣尺寸。
<p></p>
<b>使用方式：</b>從尺寸預測鳶尾花類型。  
<p></p>
<b>相關研究：</b>Fisher, R.A. (1988 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>影片推文</td>
  <td>
資料集是 Movie Tweetings 資料集的擴充版本。 資料集包含 17 萬個影片的分級，擷取自 Twitter 上結構良好的推文。 每個執行個體代表推文，並且是 Tuple：使用者識別碼、IMDB 影片識別碼、分級、時間戳記、推文的收藏數，以及這個推文的轉推數。 資料集是由 A. Said、S. Dooms、B. Loni 和 D. Tikk 為 Recommender Systems Challenge 2014 提供。
  </td>
</tr>

<tr>
  <td>不同汽車的油耗資料</td>
  <td>
此資料集是根據 Carnegie Mellon University 的 StatLib 圖書館所提供的資料集略做修改而成。 此資料集用於 1983 American Statistical Association Exposition。
<p></p>
這項資料以每加侖的里程數為單位列出各種汽車的油耗。 這也包含汽缸數、引擎排氣量、馬力、總重量和加速等資訊。
<p></p>
<b>使用方式：</b>根據三個多重值離散屬性和五個連續屬性，預測燃料經濟效益。 
<p></p>
<b>相關研究：</b>StatLib Carnegie Mellon University (1993 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>皮馬族印第安人糖尿病二進位分類資料集</td>
  <td>
National Institute of Diabetes and Digestive and Kidney Diseases 資料庫中的資料子集。 此資料集經過篩選，主要著重於皮馬族印第安人遺傳疾病的女性病患。 資料中包含葡萄糖和胰島素水平之類的醫學資料，以及生活方式因素。
<p></p>
<b>使用方式：</b>預測受測者是否罹患糖尿病 (二元分類)。 
<p></p>
<b>相關研究：</b>Sigillito, V.(1990 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml"</a>。 Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>餐廳顧客資料</td>
  <td>
一組關於顧客的中繼資料，包括人口統計和喜好。
<p></p>
<b>使用方式：</b>將此資料集與其他兩個餐廳資料集搭配使用，以使推薦系統定型並進行測試。 
<p></p>
<b>相關研究：</b>Bache, K. 和 Lichman, M.(2013 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science.
  </td>
</tr>

<tr>
  <td>餐廳特色資料</td>
  <td>
一組關於餐廳及其特色的中繼資料，例如食物類型、用餐風格和地點等。
<p></p>
<b>使用方式：</b>將此資料集與其他兩個餐廳資料集搭配使用，以使推薦系統定型並進行測試。 
<p></p>
<b>相關研究：</b>Bache, K. 和 Lichman, M.(2013 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science.
  </td>
</tr>

<tr>
  <td>餐廳評等</td>
  <td>
包含使用者給予餐廳的評等，最低為 0，最高為 2。
<p></p>
<b>使用方式：</b>將此資料集與其他兩個餐廳資料集搭配使用，以使推薦系統定型並進行測試。 
<p></p>
<b>相關研究：</b>Bache, K. 和 Lichman, M.(2013 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science.
  </td>
</tr>

<tr>
  <td>煉鋼多類別資料集</td>
  <td>
此資料集包含試煉鋼鐵的一系列記錄。 其中包含產生的鋼鐵類型實際屬性 (寬度、厚度、類型 (鋼條圈、鋼片等)。
<p></p>
<b>使用方式：</b>預測任兩個數值類別屬性；硬度或強度。 您也可以分析各屬性之間的關聯。
<p></p>
鋼鐵等級會遵循 SAE 和其他組織所定義的一組標準。 您想要尋找特定的「等級」(類別變數)，並了解所需的值。 
<p></p>
<b>相關研究：</b>Sterling, D. 和 Buntine, W.(NA)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information and Computer Science
<p></p>
鋼鐵等級的實用指南可在這裡找到：<a href="https://otk-sitecore-prod-v2-cdn.azureedge.net/-/media/from-sharepoint/documents/product/outokumpu-steel-grades-properties-global-standards.pdf">https://otk-sitecore-prod-v2-cdn.azureedge.net/-/media/from-sharepoint/documents/product/outokumpu-steel-grades-properties-global-standards.pdf</a>
  </td>
</tr>

<tr>
  <td>望遠鏡資料</td>
  <td>
高能伽瑪粒子爆與背景雜訊的記錄，兩者皆使用蒙地卡羅程序來模擬。
<p></p>
模擬的目的是改善地面大氣切倫科夫伽瑪望遠鏡的精確性。 這是使用統計方法區分所要的訊號 (切倫科夫射線波) 與背景雜訊 (高層大氣中的宇宙射線所引發的強子波) 所達成。
<p></p>
這項資料已經過前處理，而建立長軸指向攝相中心的狹長叢集。 這個橢圓的特性 (通常稱為 Hillas 參數) 屬於可用於區別的影像參數。
<p></p>
<b>使用方式：</b>預測某個波的影像代表的是訊號還是背景雜訊。
<p></p>
<b>附註：</b>簡單的分類精確性對這項資料並沒有幫助，因為將背景活動分類為訊號，比將訊號活動分類為背景還要糟。 若要比較不同的分類器，應使用 ROC 圖形。 將背景活動認定為訊號的機率，必須低於下列其中一個臨界值：0.01、0.02、0.05、0.1 或 0.2。
<p></p>
此外請注意，背景活動的數目 (h 表示強子波) 會被低估。 在實際的測量中，h 或雜訊類別會佔活動的多數。 
<p></p>
<b>相關研究：</b>Bock, R.K。 (1995 年)。 UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。 Irvine, CA: University of California, School of Information </td>
</tr>

<tr>
  <td>天氣資料集</td>
  <td>
NOAA 提供的每小時起降天候觀測值 (<a href="http://az754797.vo.msecnd.net/data/WeatherDataset.csv">將 201304 的資料合併至 201310</a>)。
<p></p>
天氣資料涵蓋從機場天候觀測站進行的觀測，涵蓋期間為 2013 年 4 月至 10 月。 上傳至 Azure Machine Learning Studio 之前，資料集已經過下列處理：
<ul>
  <li>天候觀測站識別碼已對應至相對應的機場識別碼</li>
  <li>已篩選掉與 70 個最繁忙的機場沒有關聯的天候觀測站</li>
  <li>Date 資料行已分割為個別的 Year、Month 和 Day 資料行</li>
  <li>已選取下列資料行：AirportID、Year、Month、Day、Time、TimeZone、SkyCondition、Visibility、WeatherType、DryBulbFarenheit、DryBulbCelsius、WetBulbFarenheit、WetBulbCelsius、DewPointFarenheit、DewPointCelsius、RelativeHumidity、WindSpeed、WindDirection、ValueForWindCharacter、StationPressure、PressureTendency、PressureChange、SeaLevelPressure、RecordType、HourlyPrecip、Altimeter</li>
</ul>
  </td>
</tr>

<tr>
  <td>Wikipedia SP 500 資料集</td>
  <td>
資料是從 Wikipedia (<a href="http://www.wikipedia.org/">http://www.wikipedia.org/</a>) 上每家 S&P 500 公司的文章衍生而來 (儲存為 XML 資料)。
<p></p>
上傳至 Azure Machine Learning Studio 之前，資料集已經過下列處理：
<ul>
  <li>擷取每家特定公司的文字內容</li>
  <li>移除 wiki 格式</li>
  <li>移除非英數字元</li>
  <li>將所有文字轉換為小寫</li>
  <li>新增了知名公司類別</li>
</ul>
<p></p>
請注意，對某些公司而言，找不到某篇文章，所以記錄筆數小於 500 筆。
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/direct_marketing.csv">direct_marketing.csv</a></td>
  <td>
資料集含有客戶資料，以及他們對直接郵寄促銷活動回應的相關指示。 每個資料列都代表一個客戶。 資料集包含相關的使用者人口統計和過去行為的九項特徵，以及三個標籤資料行 (瀏覽、轉換和費用)。  瀏覽是二進位資料行，指出客戶在行銷活動之後造訪。 轉換指出客戶購買某個物品。 費用是所花費的金額。  資料集是由 Kevin Hillstrom 為 MineThatData E-Mail Analytics And Data Mining Challenge 提供。
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/lyrl2004_tokens_test.csv">lyrl2004_tokens_test.csv</a></td>
  <td>
RCV1-V2 路透社新聞資料集中測試功能的範例。 資料集有 78 萬 1 千篇新聞文章，以及它們的識別碼 (資料集的第一個資料行)。 每篇文章均執行語彙基元化、停用字詞和詞幹分析。 資料集由 David. D. Lewis  D. 提供。
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/lyrl2004_tokens_train.csv">lyrl2004_tokens_train.csv</a></td>
  <td>
RCV1-V2 路透社新聞資料集中訓練功能的範例。 資料集有 2 萬 3 千篇新聞文章，以及它們的識別碼 (資料集的第一個資料行)。 每篇文章均執行語彙基元化、停用字詞和詞幹分析。 資料集由 David. D. Lewis  D. 提供。
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/network_intrusion_detection.csv">network_intrusion_detection.csv</a><br></td>
  <td>
來自 KDD Cup 1999 知識探索和資料探勘工具競賽 (<a href="http://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html">kddcup99.html</a>) 的資料集。
<p></p>
此資料集已下載並儲存在 Azure Blob 儲存體 (<a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/network_intrusion_detection.csv">network_intrusion_detection.csv</a>) 中，同時包含訓練和測試資料集。 訓練資料集約有 126K 個資料列和 43 個資料行，內含標籤。 有 3 個資料行屬於標籤資訊，而 40 個資料行 (包含數值和字串/類別特性) 可用於訓練模型。 測試資料約有 22.5K 個測試範例 (具有與訓練資料相同的 43 個資料行)。
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/rcv1-v2.topics.qrels.csv">rcv1-v2.topics.qrels.csv</a></td>
  <td>
RCV1-V2 路透社新聞資料集中新聞文章的主題指派。 一篇新聞文章可以指派給數個主題。 每個資料列的格式為 "&lt;topic name&gt; &lt;document id&gt; 1"。 資料集包含 260 萬個主題指派。 資料集由 David. D. Lewis  D. 提供。
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/student_performance.txt">student_performance.txt</a></td>
  <td>
此資料來自 KDD Cup 2010 學員績效評估挑戰 (<a href="http://www.kdd.org/kdd-cup/view/kdd-cup-2010-student-performance-evaluation">student performance evaluation</a>)。 使用的資料為 Algebra_2008_2009 訓練集 (Stamper, J.、Niculescu-Mizil, A.、Ritter, S.、Gordon, G.J.、& Koedinger, K.R. (2010)。 Algebra I 2008-2009。 KDD Cup 2010 教育資料探勘挑戰中的的挑戰資料集。 請在 <a href="http://pslcdatashop.web.cmu.edu/KDDCup/downloads.jsp">downloads.jsp</a> 尋找。
<p></p>
此資料集已下載並儲存在 Azure Blob 儲存體 (<a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/student_performance.txt">student_performance.txt</a>) 中並包含來自學員教學系統的記錄檔。 提供的特性包括問題識別碼及其簡短說明、學員識別碼、時間戳記，以及學員以正確方式解出問題之間的嘗試次數。 原始資料集有 8.9M 的記錄，此資料集已縮減為前 100K 個資料列。 此資料集有 23 個以 Tab 字元分隔的各種資料行：數值、類別和時間戳記。
  </td>
</tr>

</table>


<!-- Module References -->
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
