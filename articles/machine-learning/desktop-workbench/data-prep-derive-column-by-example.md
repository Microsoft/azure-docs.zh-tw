---
title: 使用 Azure Machine Learning Workbench 進行衍生資料行實例化轉換
description: 「衍生資料行實例化」轉換的參考文件
services: machine-learning
author: ranvijaykumar
ms.author: ranku
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc, reference
ms.topic: article
ms.date: 09/14/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 311fd39792274ef01b1b03fdf8252eb7ac93c922
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978846"
---
# <a name="derive-column-by-example-transformation"></a>衍生資料行實例化轉換

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



**衍生資料行實例化**轉換可讓使用者使用衍生結果的提供範例，建立一或多個現有資料行的衍生項目。 衍生項目可以是支援的字串、日期和數字轉換的任何組合。 

支援下列字串、日期和數字轉換：

**字串轉換：** 

子字串包括智慧型擷取數字和日期、串連、案例操作、對應常數值。

**日期轉換：** 

日期格式變更、擷取日期組件、對應時間對時間量化。

日期轉換相當通用，但有一些值得注意的限制：
* 未支援時區。
* 某些常見格式未受支援：
    * ISO 8601 年度週格式 (例如 “2009-W53-7”) 
    * Unix Epoch 時間。
* 所有的格式皆區分大小寫 (尤其是「4am」無法辨識，只能辨識「4AM」)。

**數字轉換：** 

四捨五入、無條件捨去，無條件進位、量化、以零或空格填補、除以或乘以 1000 次方。

**複合轉換：** 

字串、數字或日期轉換的任何組合。

## <a name="how-to-use-this-transformation"></a>如何使用此轉換

若要執行此轉換，請遵循下列步驟：
1. 選取要從中衍生值的一或多個資料行。 
2. 從 [轉換] 功能表選取 [衍生資料行實例化]。 或者，以滑鼠右鍵按一下任一選取資料行的標頭，並從操作功能表選取 [衍生資料行實例化]。 「轉換編輯器」隨即開啟，並在最右邊選取的資料行旁邊新增一個新資料行。 可以根據資料行標頭中的核取方塊來識別選取的資料行。 可以使用資料行標頭中的核取方塊來新增和移除選擇的資料行。
3. 針對資料列輸入*輸出*的範例，然後按 Enter。 此時，Workbench 會分析輸入資料行和提供的輸出，以合成可將給定輸入轉換為輸出的程式。 合成的程式會針對資料格中的所有資料列來執行。 對於模稜兩可和複雜的情況，可能會需要多個範例。 根據您是處於「基本模式」或「進階模式」，可透過不同方式提供多個範例。
4. 檢閱輸出，然後按一下 [確定] 以接受轉換。

### <a name="transform-editor-basic-mode"></a>轉換編輯器：基本模式

「基本模式」提供在資料格中的內嵌編輯體驗。 您可以巡巡覽至感興趣的資料格，並輸入值，以提供輸出的範例。 

Workbench 會分析資料，並嘗試找出使用者應該檢閱的邊緣案例。 分析資料時，**分析資料**會顯示在「轉換編輯器」的標頭中。 完成其中一個分析時，**無建議**或**檢閱下一個建議的資料列**會顯示在標頭中。 您可以按一下 [檢閱下一個建議的資料列]，以覽整個邊緣案例。 如果資料列的值不正確，您應該鍵入正確的值作為附加範例。 

### <a name="transform-editor-advanced-mode"></a>轉換編輯器：進階模式

「進階模式」針對依範例衍生資料行提供更豐富的體驗。 所有範例都顯示在一個地方。 您也可以按一下 [顯示建立的範例]，檢閱某處的所有邊緣案例。 

在進階模式中，您可以按兩下格線中的資料列，以新增任何資料列作為範例資料列。 會複製其中一個資料列作為範例資料列，您也可以編輯來源資料行中的資料，以製作綜合範例。 如此一來，您可以新增範例資料中目前不存在的案例。

使用者可按一下「轉換編輯器」中的連結，以在**基本模式**和**進階模式**之間切換。

### <a name="transform-editor-send-feedback"></a>轉換編輯器：傳送意見反應

按一下 [傳送意見反應] 連結會開啟 [意見反應] 對話方塊，其中會顯示具有使用者所提供範例的意見方塊。 使用者應該檢閱意見方塊的內容並提供更多詳細資料以協助我們了解問題。 若使用者不想要與 Microsoft 分享資料，使用者在按下 [傳送意見反應] 按鈕之前應該先刪除預先填寫的範例資料。 

### <a name="editing-existing-transformation"></a>編輯現有的轉換

使用者可選取「轉換步驟」中的 [編輯] 選項，以編輯現有的**衍生資料行實例化**轉換。 按一下 [編輯]，可在**進階模式**中開啟「轉換編輯器」，並顯示建立轉換期間提供的所有範例。

## <a name="examples-of-string-transformations-by-example"></a>字串轉換實例化的範例


>[!NOTE] 
>**粗體**值表示為了完成顯示之資料集中的轉換而提供的範例。


### <a name="s1-extracting-file-names-from-file-paths"></a>S1. 從檔案路徑擷取檔案名稱

此案例所需要的範例數目：2

|輸入|輸出|
|:-----|:-----|
|C:\Python35\Tools\pynche\TypeinViewer.py|**TypeinViewer.py**|
|C:\Python35\Tools\pynche\webcolors.txt|webcolors.txt|
|C:\Python35\Tools\pynche\websafe.txt|websafe.txt|
|C:\Python35\Tools\pynche\X\rgb.txt|rgb.txt|
|C:\Python35\Tools\pynche\X\xlicense.txt|xlicense.txt|
|C:\Python35\Tools\Scripts\2to3.py|2to3.py|
|C:\Python35\Tools\Scripts\analyze_dxp.py|**analyze_dxp.py**|
|C:\Python35\Tools\Scripts\byext.py|byext.py|
|C:\Python35\Tools\Scripts\byteyears.py|byteyears.py|
|C:\Python35\Tools\Scripts\checkappend.py|checkappend.py|

### <a name="s2-case-manipulation-during-string-extraction"></a>S2. 字串擷取期間操作案例

此案例所需要的範例數目：3

|輸入|輸出|
|:-----|:-----|
|REINDEER CT & DEAD END;  NEW HANOVER; Station 332; 2015-12-10 \@ 17:10:52;|**New Hanover**|
|BRIAR PATH & WHITEMARSH LN;  HATFIELD TOWNSHIP; Station 345; 2015-12-10 \@ 17:29:21;|Hatfield Township|
|HAWS AVE; NORRISTOWN; 2015-12-10 \@ 14:39:21-Station:STA27;|**Norristown**|
|AIRY ST & SWEDE ST;  NORRISTOWN; Station 308A; 2015-12-10 \@ 16:47:36;|**Norristown**|
|CHERRYWOOD CT & DEAD END;  LOWER POTTSGROVE; Station 329; 2015-12-10 \@ 16:56:52;|Lower Pottsgrove|
|CANNON AVE & W 9TH ST;  LANSDALE; Station 345; 2015-12-10 \@ 15:39:04;|Lansdale|
|LAUREL AVE & OAKDALE AVE;  HORSHAM; Station 352; 2015-12-10 \@ 16:46:48;|Horsham|
|COLLEGEVILLE RD & LYWISKI RD;  SKIPPACK; Station 336; 2015-12-10 \@ 16:17:05;|Skippack|
|MAIN ST & OLD SUMNEYTOWN PIKE;  LOWER SALFORD; Station 344; 2015-12-10 \@ 16:51:42;|Lower Salford|
|BLUEROUTE  & RAMP I476 NB TO CHEMICAL RD; PLYMOUTH; 2015-12-10 \@ 17:35:41;|Plymouth|
|RT202 PKWY & KNAPP RD; MONTGOMERY; 2015-12-10 \@ 17:33:50;|Montgomery|
|BROOK RD & COLWELL LN; PLYMOUTH; 2015-12-10 \@ 16:32:10;|Plymouth|

### <a name="s3-date-format-manipulation-during-string-extraction"></a>S3. 字串擷取期間的日期格式操作

此案例所需要的範例數目：1

|輸入|輸出|
|:-----|:-----|
|MONTGOMERY AVE & WOODSIDE RD;  LOWER MERION; Station 313; 2015-12-11 \@ 04:11:35;|**12 Nov 2015 4AM**|
|DREYCOTT LN & W LANCASTER AVE;  LOWER MERION; Station 313; 2015-12-11 \@ 01:29:52;|12 Nov 2015 1AM|
|E LEVERING MILL RD & CONSHOHOCKEN STATE RD; LOWER MERION; 2015-12-11 \@ 07:29:58;|12 Nov 2015 7AM|
|PENN VALLEY RD & MANOR RD;  LOWER MERION; Station 313; 2015-12-10 \@ 20:53:30;|12 Oct 2015 8PM|
|BELMONT AVE & OVERHILL RD; LOWER MERION; 2015-12-10 \@ 23:02:27;|12 Oct 2015 11PM|
|W MONTGOMERY AVE & PENNSWOOD RD; LOWER MERION; 2015-12-10 \@ 19:25:22;|12 Oct 2015 7PM|
|ROSEMONT AVE & DEAD END;  LOWER MERION; Station 313; 2015-12-10 \@ 18:43:07;|12 Oct 2015 6PM|
|AVIGNON DR & DEAD END; LOWER MERION; 2015-12-10 \@ 20:01:29-Station:STA24;|12 Oct 2015 8PM|

### <a name="s4-concatenating-strings"></a>S4. 串連字串

此案例所需要的範例數目：1

>[!NOTE] 
>在此範例中，特殊字元表示輸出資料行中的空格。

|名字|中間名縮寫|姓氏|輸出|
|:-----|:-----|:-----|:-----|
|Laquanda||Lohmann|Laquanda··Lohmann|
|Claudio|具有使用 |Chew|**Claudio·A·Chew**|
|Sarah-Jane|S|Smith|Sarah-Jane·S·Smith|
|Brandi||Blumenthal|Brandi··Blumenthal|
|Jesusita|R|Journey|Jesusita·R·Journey|
|Hermina||Hults|Hermina··Hults|
|Anne-Marie|W|Jones|Anne-Marie·W·Jones|
|Rico||Ropp|Rico··Ropp|
|Lauren-May||Fullmer|Lauren-May··Fullmer|
|Marc|T|Maine|Marc·T·Maine|
|Angie||Adelman|Angie··Adelman|
|John-Paul||Smith|John-Paul··Smith|
|Song|W|Staller|Song·W·Staller|
|Jill||Jefferies|Jill··Jefferies|
|Ruby-Grace|M|Simmons|Ruby-Grace·M·Simmons|

### <a name="s5-generating-initials"></a>S5. 產生縮寫

此案例所需要的範例數目：2

|完整名稱|輸出|
|:-----|:-----|
|Laquanda Lohmann|**L.L.**|
|Claudio Chew|C.C.|
|Sarah-Jane Smith|S.S.|
|Brandi Blumenthal|B.B.|
|Jesusita Journey|J.J.|
|Hermina Hults|H.H.|
|Anne-Marie Jones|A.J.|
|Rico Ropp|R.R.|
|Lauren-May Fullmer|L.F.|
|Marc Maine|M.M.|
|Angie Adelman|A.A.|
|John-Paul Smith|**J.S.**|
|Song Staller|S.S.|
|Jill Jefferies|J.J.|
|Ruby-Grace Simmons|R.S.|


### <a name="s6-mapping-constant-values"></a>S6. 對應常數值

此案例所需要的範例數目：3

|管理性別|輸出|
|:-----|:-----:|
|男性|**0**|
|女性|**1**|
|不明|**2**|
|女性|1|
|女性|1|
|男性|0|
|不明|2|
|男性|0|
|女性|1|

## <a name="examples-of-number-transformations-by-example"></a>數字轉換實例化範例

>[!NOTE] 
>**粗體**值表示為了完成顯示之資料集中的轉換而提供的範例。


### <a name="n1-rounding-to-nearest-10"></a>N1. 四捨五入至最接近 10

此案例所需要的範例數目：1

|輸入|輸出|
|-----:|-----:|
|112|**110**|
|117|120|
|11112|11110|
|11119|11120|
|548|550|

### <a name="n2-rounding-down-to-nearest-10"></a>N2. 無條件捨去至最接近 10

此案例所需要的範例數目：2

|輸入|輸出|
|-----:|-----:|
|112|**110**|
|117|**110**|
|11112|11110|
|11119|11110|
|548|540|

### <a name="n3-rounding-to-nearest-005"></a>N3. 四捨五入至最接近 0.05

此案例所需要的範例數目：2

|輸入|輸出|
|-----:|-----:|
|-75.5812935|**-75.60**|
|-75.2646799|-75.25|
|-75.3519752|-75.35|
|-75.343513|**-75.35**|
|-75.6033497|-75.60|
|-75.283245|-75.30|

### <a name="n4-binning"></a>N4. 量化

此案例所需要的範例數目：1

|輸入|輸出|
|-----:|:-----:|
|20.16|**20-25**|
|14.32|10-15|
|5.44|5-10|
|3.84|0-5|
|3.73|0-5|
|7.36|5-10|

### <a name="n5-scaling-by-1000"></a>N5. 將規模調整為 1000 倍

此案例所需要的範例數目：1

|輸入|輸出|
|-----:|-----:|
|-243|**-243000**|
|-12.5|-12500|
|-2345.23292|-2345232.92|
|-1202.3433|-1202343.3|
|1202.3433|1202343.3|

### <a name="n6-padding"></a>N6. 填補

此案例所需要的範例數目：1

|代碼|輸出|
|-----:|-----:|
|5828|**05828**|
|44130|44130|
|49007|49007|
|29682|29682|
|4759|04759|
|10029|10029|
|7204|07204|

## <a name="examples-of-date-transformations-by-example"></a>日期轉換實例化範例

>[!NOTE] 
>**粗體**值表示為了完成顯示之資料集中的轉換而提供的範例。


### <a name="d1-extracting-date-parts"></a>D1. 擷取日期組件

這些日期組件是使用相同資料集上不同的實例化轉換來擷取的。 粗體字串表示在各自轉換中提供的範例。

|Datetime|Weekday|日期|月|Year|小時|分鐘|秒|
|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|
|31-Jan-2031 05:54:18|**Fri**|**31**|**Jan**|**2031**|**5**|**54**|**18**|
|17-Jan-1990 13:32:01|Wed|17|Jan|1990|13|32|01|
|14-Feb-2034 05:36:07|Tue|14|Feb|2034|5|36|07|
|14-Mar-2002 13:16:16|Thu|14|Mar|2002|13|16|16|
|21-Jan-1985 05:44:43|Mon|21|Jan|1985|5|44|**43**|
|16-Aug-1985 01:11:56|Fri|16|Aug|1985|1|11|56|
|20-Dec-2033 18:36:29|Tue|20|Dec|2033|18|36|29|
|16-Jul-1984 10:21:59|Mon|16|Jul|1984|10|21|59|
|13-Jan-2038 10:59:36|Wed|13|Jan|2038|10|59|36|
|14-Aug-1982 15:13:54|Sat|14|Aug|1982|15|13|54|
|22-Nov-2030 08:18:08|Fri|22|Nov|2030|8|18|08|
|21-Oct-1997 08:42:58|Tue|21|Oct|1997|8|42|58|
|28-Nov-2006 14:19:15|Tue|28|Nov|2006|14|19|15|
|29-Apr-2031 04:59:45|Tue|29|Apr|2031|4|59|45|
|29-Jan-2032 02:38:36|Thu|29|Jan|2032|2|38|36|
|11-May-2028 15:31:52|Thu|11|May|2028|15|31|52|
|15-Jul-1977 12:45:39|Fri|15|Jul|1977|12|45|39|
|27-Jan-2029 05:55:41|Sat|27|Jan|2029|5|55|41|
|03-Mar-2024 10:17:49|Sun|3|Mar|2024|10|17|49|
|14-Apr-2010 00:23:13|Wed|14|Apr|2010|0|23|13|

### <a name="d2-formatting-dates"></a>D2. 格式化日期

這些日期格式化是使用相同資料集上不同的實例化轉換來完成的。 粗體字串表示在各自轉換中提供的範例。

|Datetime|Format1|Format2|Format3|Format4|Format5|
|-----:|-----:|-----:|-----:|-----:|-----:|
|31-Jan-2031 05:54:18|**1/31/2031**|**Friday, January 31, 2031**|**01312031 5:54**|**31/1/2031 5:54 AM**|**Q1 2031**|
|17-Jan-1990 13:32:01|1/17/1990|Wednesday, January 17, 1990|01171990 13:32|17/1/1990 1:32 PM|Q1 1990|
|14-Feb-2034 05:36:07|2/14/2034|Tuesday, February 14, 2034|02142034 5:36|14/2/2034 5:36 AM|Q1 2034
|14-Mar-2002 13:16:16|3/14/2002|Thursday, March 14, 2002|03142002 13:16|14/3/2002 1:16 PM|Q1 2002
|21-Jan-1985 05:44:43|1/21/1985|Monday, January 21, 1985|01211985 5:44|21/1/1985 5:44 AM|Q1 1985
|16-Aug-1985 01:11:56|8/16/1985|Friday, August 16, 1985|08161985 1:11|16/8/1985 1:11 AM|Q3 1985
|20-Dec-2033 18:36:29|12/20/2033|Tuesday, December 20, 2033|12202033 18:36|20/12/2033 6:36 PM|Q4 2033
|16-Jul-1984 10:21:59|7/16/1984|Monday, July 16, 1984|07161984 10:21|16/7/1984 10:21 AM|Q3 1984
|13-Jan-2038 10:59:36|1/13/2038|Wednesday, January 13, 2038|01132038 10:59|13/1/2038 10:59 AM|Q1 2038
|14-Aug-1982 15:13:54|8/14/1982|Saturday, August 14, 1982|08141982 15:13|14/8/1982 3:13 PM|Q3 1982
|22-Nov-2030 08:18:08|11/22/2030|Friday, November 22, 2030|11222030 8:18|22/11/2030 8:18 AM|Q4 2030
|21-Oct-1997 08:42:58|10/21/1997|Tuesday, October 21, 1997|10211997 8:42|21/10/1997 8:42 AM|Q4 1997
|28-Nov-2006 14:19:15|11/28/2006|Tuesday, November 28, 2006|11282006 14:19|28/11/2006 2:19 PM|Q4 2006
|29-Apr-2031 04:59:45|4/29/2031|Tuesday, April 29, 2031|04292031 4:59|29/4/2031 4:59 AM|Q2 2031
|29-Jan-2032 02:38:36|1/29/2032|Thursday, January 29, 2032|01292032 2:38|29/1/2032 2:38 AM|Q1 2032
|11-May-2028 15:31:52|5/11/2028|Thursday, May 11, 2028|05112028 15:31|11/5/2028 3:31 PM|Q2 2028
|15-Jul-1977 12:45:39|7/15/1977|Friday, July 15, 1977|07151977 12:45|15/7/1977 12:45 PM|Q3 1977
|27-Jan-2029 05:55:41|1/27/2029|Saturday, January 27, 2029|01272029 5:55|27/1/2029 5:55 AM|Q1 2029
|03-Mar-2024 10:17:49|3/3/2024|Sunday, March 3, 2024|03032024 10:17|3/3/2024 10:17 AM|Q1 2024
|14-Apr-2010 00:23:13|4/14/2010|Wednesday, April 14, 2010|04142010 0:23|14/4/2010 12:23 AM|Q2 2010


### <a name="d3-mapping-time-to-time-periods"></a>D3. 對應時間對時間週期

這些日期時間對期間對應是使用相同資料集上不同的實例化轉換來完成的。 粗體字串表示在各自轉換中提供的範例。

|Datetime|Period(Seconds)|Period(Minutes)|Period(Two Hours)|Period(30 Minutes)|
|-----:|-----:|-----:|-----:|-----:|
|31-Jan-2031 05:54:18|**0-20**|**45-60**|**5AM-7AM**|**5:30-6:00**|
|17-Jan-1990 13:32:01|**0-20**|30-45|1PM-3PM|13:30-14:00|
|14-Feb-2034 05:36:07|0-20|30-45|5AM-7AM|5:30-6:00|
|14-Mar-2002 13:16:16|0-20|15-30|1PM-3PM|13:00-13:30|
|21-Jan-1985 05:44:43|40-60|30-45|5AM-7AM|5:30-6:00|
|16-Aug-1985 01:11:56|40-60|0-15|1AM-3AM|1:00-1:30|
|20-Dec-2033 18:36:29|20-40|30-45|5PM-7PM|18:30-19:00|
|16-Jul-1984 10:21:59|40-60|15-30|9AM-11AM|10:00-10:30|
|13-Jan-2038 10:59:36|20-40|45-60|9AM-11AM|10:30-11:00|
|14-Aug-1982 15:13:54|40-60|0-15|3PM-5PM|15:00-15:30|
|22-Nov-2030 08:18:08|0-20|15-30|7AM-9AM|8:00-8:30|
|21-Oct-1997 08:42:58|40-60|30-45|7AM-9AM|8:30-9:00|
|28-Nov-2006 14:19:15|0-20|15-30|1PM-3PM|14:00-14:30|
|29-Apr-2031 04:59:45|40-60|45-60|3AM-5AM|4:30-5:00|
|29-Jan-2032 02:38:36|20-40|30-45|1AM-3AM|2:30-3:00|
|11-May-2028 15:31:52|40-60|30-45|3PM-5PM|15:30-16:00|
|15-Jul-1977 12:45:39|20-40|45-60|11AM-1PM|12:30-13:00|
|27-Jan-2029 05:55:41|40-60|45-60|5AM-7AM|5:30-6:00|
|03-Mar-2024 10:17:49|40-60|15-30|9AM-11AM|10:00-10:30|
|14-Apr-2010 00:23:13|0-20|15-30|11PM-1AM|0:00-0:30|

## <a name="examples-of-composite-transformations-by-example"></a>複合轉換實例化範例

|tripduration|starttime|start station id|start station latitude|start station longitude|usertype|欄|
|-----:|-----:|-----:|-----:|-----:|-----:|-----:|
|61|2016-01-08 16:09:32|107|42.3625|-71.08822|用戶|**用戶在 2016 年 1 月 8 日下午 4 點左右，從經緯度 (42.363,-71.088) 的 107 號車站，取用自行車。路程持續時間為 61 分鐘**|
|61|2016-01-17 09:28:10|74|42.373268|-71.118579|Customer|客戶在 2016 年 1 月 17 日上午 9 點左右，從經緯度 (42.373,-71.119) 的 74 號車站取用自行車。 路程持續時間為 61 分鐘|
|62|2016-01-25 08:10:26|176|42.386748020450561|-71.119018793106079|用戶|用戶在 2016 年 1 月 25 日上午 8 點左右，從經緯度 (42.387,-71.119) 的 176 號車站，取用自行車。 路程持續時間為 62 分鐘|
|63|2016-01-08 10:10:29|107|42.3625|-71.08822|用戶|用戶在 2016 年 1 月 8 日上午 10 點左右，從經緯度 (42.363,-71.088) 的 107 號車站，取用自行車。 路程持續時間為 63 分鐘|
|64|2016-01-15 19:42:08|68|42.36507|-71.1031|用戶|用戶在 2016 年 1 月 15 日下午 7 點左右，從經緯度 (42.365,-71.103) 的 68 號車站，取用自行車。 路程持續時間為 64 分鐘|
|64|2016-01-22 18:16:13|115|42.387995|-71.119084|用戶|用戶在 2016 年 1 月 22 日下午 6 點左右，從經緯度 (42.388,-71.119) 的 115 號車站，取用自行車。 路程持續時間為 64 分鐘|
|68|2016-01-18 09:51:52|178|42.359573201090441|-71.101294755935669|用戶|用戶在 2016 年 1 月 18 日上午 9 點左右，從經緯度 (42.360,-71.101) 的 178 號車站，取用自行車。 路程持續時間為 68 分鐘|
|69|2016-01-14 08:57:55|176|42.386748020450561|-71.119018793106079|用戶|用戶在 2016 年 1 月 14 日上午 8 點左右，從經緯度 (42.387,-71.119) 的 176 號車站，取用自行車。 路程持續時間為 69 分鐘|
|69|2016-01-13 22:12:55|141|42.363560158429884|-71.08216792345047|用戶|用戶在 2016 年 1 月 13 日下午 10 點左右，從經緯度 (42.364,-71.082) 的 141 號車站，取用自行車。 路程持續時間為 69 分鐘|
|69|2016-01-15 08:13:09|176|42.386748020450561|-71.119018793106079|用戶|用戶在 2016 年 1 月 15 日上午 8 點左右，從經緯度 (42.387,-71.119) 的 176 號車站，取用自行車。 路程持續時間為 69 分鐘|


## <a name="technical-notes"></a>技術說明

### <a name="conditional-transformations"></a>條件轉換
在某些情況下，找不到滿足所提供範例的單一轉換。 在這種情況下，「衍生資料行實例化轉換」會嘗試根據某些模式來分組輸入，並了解每個群組的個別轉換。 我們稱之為**條件式轉換**。 **條件式轉換**僅針對具有單一輸入資料行的轉換進行嘗試。 

### <a name="reference"></a>參考
關於字串轉換實例化技術的詳細資訊，可在[此發佈](https://www.microsoft.com/research/publication/automating-string-processing-spreadsheets-using-input-output-examples/)中找到。
