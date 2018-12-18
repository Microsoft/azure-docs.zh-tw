---
title: 支援可供 Azure Machine Learning 資料準備使用的偵測器 | Microsoft Docs
description: 本文件提供 Azure Machine Learning 資料準備可使用的偵測器完整清單
services: machine-learning
author: euangMS
ms.author: euang
manager: lanceo
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: ''
ms.devlang: ''
ms.topic: article
ms.date: 02/01/2018
ROBOTS: NOINDEX
ms.openlocfilehash: e6af524d7f80104a34831f576a14d2dbf15e384f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987198"
---
# <a name="supported-inspectors-for-the-azure-machine-learning-data-preparation-preview"></a>Azure Machine Learning 資料準備預覽版所支援的偵測器

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


本文件概述此預覽中可用的偵測器集合。

## <a name="the-halo-effect"></a>光暈效果 
某些偵測器可支援光暈效果。 這種效果會使用兩個不同的色彩來立即顯示轉換的視覺化方式變更。 灰色表示最新轉換之前的值，藍色則會顯示目前的值。 這個效果可在選項中加以啟用和停用。

## <a name="graphical-filtering"></a>圖形化篩選 
某些偵測器支援使用偵測器作為編輯器來篩選資料。 使用偵測器做為編輯器牽涉到選取圖形化元素，然後使用偵測器視窗右上方的工具列，來篩選選取的值。 

## <a name="column-statistics"></a>資料行統計資料
針對數值資料行，這個偵測器會提供關於資料行的各種不同統計資料。 統計資料包括下列度量： 
- 最小值
- 下四分位數
- 中間值
- 上四分位數
- 最大值
- 平均值
- 標準差


### <a name="options"></a>選項 
- None

## <a name="histogram"></a>長條圖 
計算並顯示單一數值資料行的長條圖。 預設的貯體數目會使用 Scott 規則來進行計算。 不過，您可以透過選項覆寫此規則。

這個偵測器支援光暈效果。


### <a name="options"></a>選項
- 貯體數目最小值 (即使在檢查預設貯體時也會套用)
- 預設貯體數目 (Scott 的規則) 
- 顯示光暈
- 核心密度繪圖重疊 (高斯核心) 
- 使用對數刻度


### <a name="actions"></a>動作
這個偵測器可透過貯體支援篩選，其可包含單一或複選的貯體。 如先前所述地套用篩選。

## <a name="value-counts"></a>值計數
這個偵測器會顯示目前選取資料行的值頻率表。 預設顯示的是排名前六的值。 不過，您可以將此限制變更為任何數字。 您也可以將顯示的值設定為由下計算，而不是由上。 這個偵測器支援光暈效果。

### <a name="options"></a>選項 
- 頂端數值數目
- 遞減
- 包含 Null/錯誤值
- 顯示光暈
- 使用對數刻度


### <a name="actions"></a>動作 
這個偵測器可透過長條支援篩選，其可包含單一或複選的長條。 如先前所述地套用篩選。

## <a name="box-plot"></a>盒狀圖 
數值資料行的盒鬚圖。

### <a name="options"></a>選項 
- 依資料行分組

## <a name="scatter-plot"></a>散佈圖
兩個數值資料行的散佈圖。 基於效能的考量，資料會採用縮減取樣。 您可以在選項中覆寫取樣大小。

### <a name="options"></a>選項  
- X 軸資料行
- Y 軸資料行
- 取樣大小
- 依資料行分組


## <a name="time-series"></a>時間序列
X 軸上具有時間感知的折線圖。

### <a name="options"></a>選項
- 日期資料行
- 數值資料行
- 取樣大小


### <a name="actions"></a>動作
這個偵測器支援透過按一下和拖曳選取的方法來選取圖形上的範圍，進行篩選。 完成選取之後，如先前所述地套用篩選。


## <a name="map"></a>對應 
具有繪製點的地圖，假設已指定經度和緯度。 必須先選取緯度。

### <a name="options"></a>選項
- 緯度資料行
- 經度資料行
- 開啟叢集
- 依資料行分組


### <a name="actions"></a>動作
這個偵測器支援透過地圖上的點選取範圍進行篩選。 按下 **Ctrl** 鍵，然後按一下並拖曳滑鼠，在點周圍形成方形。 然後套用篩選，如先前所述。

您可以按下地圖左側的 **E** 來快速調整地圖大小，以便只顯示可能的點。


## <a name="pattern-frequency"></a>模式頻率 

這個偵測器會顯示所選字串資料行中之模式的清單。 模式會使用類似規則運算式的語法來表示。 將滑鼠游標暫留在模式上會顯示該模式所表示之值的範例。 除了顯示模式之外，也會顯示以百分比表示的大約覆蓋範圍。

![模式偵測器的影像](media/data-prep-appendix4-supported-inspectors/PatternInspectorProductNumber.png)

### <a name="options"></a>選項
- 頂端數值數目
- 遞減
- 顯示光暈

### <a name="actions"></a>動作
這個偵測器支援依據顯示的模式進行篩選。 按 **Ctrl** 鍵，然後選取模式偵測器中填滿的列。 然後套用篩選，如先前所述。 因為使用者動作的結果，增加了一個「進階篩選」步驟。 您可以透過叫用「進階篩選」步驟的編輯選項，以查看並修改產生的 Python 程式碼。
