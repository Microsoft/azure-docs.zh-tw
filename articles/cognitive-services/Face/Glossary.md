---
title: 臉部 API 服務詞彙 | Microsoft Docs
titleSuffix: Microsoft Cognitive Services
description: 此詞彙說明使用臉部 API 服務時可能遇到的字詞。
services: cognitive-services
author: SteveMSFT
manager: corncar
ms.service: cognitive-services
ms.component: face-api
ms.topic: article
ms.date: 03/01/2018
ms.author: sbowles
ms.openlocfilehash: ec3068c013be9f2fa4ff34b1c24596e46e7c5d05
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368206"
---
# <a name="glossary"></a>詞彙

## <a name="a"></a>A

#### <a name="Attributes"></a>屬性

屬性在[偵測](#Detection-Face-Detection)結果中為選擇性，例如[年齡](#Age-Attribute)、[性別](#Gender-Attribute)、[頭部姿勢](#Head-Pose-Attribute)、[臉部毛髮](#Facial-Hair-Attribute)、[微笑](#Smile-Attribute)。
從[偵測](#Detection-Face-Detection) API 即可取得這些屬性，方法是指定查詢參數：returnFaceAttributes。 屬性提供除了[臉部識別碼](#Face-ID)和[矩形](#Face-Rectangle)之外，所選[臉部](#Face)的額外相關資訊。

如需更多詳細資料，請參閱[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)(英文\) 指南。

#### <a name="Age-Attribute"></a>年齡 (屬性)

年齡是其中一個描述特定臉部年齡的[屬性](#Attributes)。 年齡屬性在[偵測](#Detection-Face-Detection)結果中為選擇性，並可透過指定 returnFaceAttributes 參數，以[偵測](#Detection-Face-Detection)要求加以控制。

如需更多詳細資料，請參閱[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)(英文\) 指南。

## <a name="b"></a>b

## <a name="c"></a>C

#### <a name="Candidate"></a>候選項目

候選項目基本上就是[識別](#Identification)結果 (例如偵測中識別的人員和信賴等級)。 候選項目會由 [PersonID](#Person-ID) 和[信賴度](#Confidence)代表，表示人員的識別信賴度相當高。

如需更多詳細資料，請參閱[臉部 - 識別](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) \(英文\) 指南。

#### <a name="Confidence"></a>信賴度

信賴度是一個以數值揭露[臉部](#Face)或[人員](#Person)之間相似度的度量 – 可在[識別](#Identification)和[驗證](#Verification)中用來指出所搜尋、識別及驗證之結果的相似度。

如需更多詳細資料，請參考下列指南：[臉部 - 尋找相似項目](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) \(英文\)、[臉部 - 識別](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) \(英文\)、[臉部 - 驗證](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a) \(英文\)。

## <a name="d"></a>D

#### <a name="Detection-Face-Detection"></a>偵測/臉部偵測

臉部偵測是在影像中找出臉孔的動作。 使用者可以上傳影像，或在要求中指定影像 URL。 所偵測到的臉部會與表示「臉部 API」中唯一身分識別的[臉部識別碼](#Face-ID)一起傳回。 矩形會以像素指出影像中的臉部位置，以及每個臉部的選擇性[屬性](#Attributes)，例如[年齡](#Age-Attribute)、[性別](#Gender-Attribute)、[頭部姿勢](#Head-Pose-Attribute)、[臉部毛髮](#Facial-Hair-Attribute)及[微笑](#Smile-Attribute)。

如需更多詳細資料，請參閱[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)(英文\) 指南。

## <a name="e"></a>E

## <a name="f"></a>F

#### <a name="Face"></a>臉部

臉部是從「臉部 API」所衍生結果的統一字詞，與所偵測到的臉部相關。 臉部最後會由統一的身分識別 ([臉部識別碼](#Face-ID))、影像中的指定區域 ([臉部矩形](#Face-Rectangle)) 及其他臉部相關[屬性](#Face-Attributes-Facial-Attributes) (例如[年齡](#Age-Attribute)、[性別](#Gender-Attribute)、[特徵點](#Face-Landmarks-Facial-Landmarks)及[頭部姿勢](#Head-Pose-Attribute)) 代表。 此外，也可以從[偵測](#Detection-Face-Detection)傳回臉部。

如需更多詳細資料，請參閱[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)(英文\) 指南。

#### <a name="Face-API"></a>臉部 API

「臉部 API」是一個雲端式 API，提供最先進的臉部偵測和辨識演算法。 「臉部 API」的主要功能可以分成兩個類別：臉部[屬性](#Face-Attributes-Facial-Attributes)[偵測](#Detection-Face-Detection)，以及臉部[辨識](#Recognition)。

如需更多詳細資料，請參考下列指南：[臉部 API 概觀](./Overview.md)、[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) \(英文\)、[臉部 - 尋找相似項目](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) \(英文\)、[臉部 - 分組](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238) \(英文\)、[臉部 - 識別](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) \(英文\)、[臉部 - 驗證](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a) \(英文\)。

#### <a name="Face-Attributes-Facial-Attributes"></a>臉部屬性

請參閱[屬性](#Attributes)。

#### <a name="Face-ID"></a>臉部識別碼

「臉部識別碼」衍生自[偵測](#Detection-Face-Detection)結果，其中一個字串代表[臉部 API](#Face-API) 中的一個[臉部](#Face)。

如需更多詳細資料，請參閱[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)(英文\) 指南。

#### <a name="Face-Landmarks-Facial-Landmarks"></a>臉部特徵點

特徵點在[偵測](#Detection-Face-Detection)結果中為選擇性；這些是語意上的臉部點，例如眼睛、鼻子及嘴巴 (如下圖所示)。 您可以透過布林值數字 returnFaceLandmarks，以[偵測](#Detection-Face-Detection)要求控制特徵點。 如果將 returnFaceLandmarks 設定為 true，傳回的臉部就會包含特徵點屬性。

如需更多詳細資料，請參閱[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)(英文\) 指南。

![HowToDetectFace](./Images/landmarks.1.jpg)

#### <a name="Face-Rectangle"></a>臉部矩形

臉部矩形衍生自[偵測](#Detection-Face-Detection)結果，這是影像中以像素為單位的直向矩形 (left、top、width、height)。 除了寬和高之外，[臉部](#Face)的左上角 (left、top) 分別以 X 和 Y 軸指出臉部大小。

如需更多詳細資料，請參閱[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)(英文\) 指南。

#### <a name="Facial-Hair-Attribute"></a>臉部毛髮 (屬性)

臉部毛髮是其中一個用來描述可用臉孔臉部毛髮長度的[屬性](#Attributes)。 臉部毛髮屬性在[偵測](#Detection-Face-Detection)結果中為選擇性，並可透過 returnFaceAttributes，以[偵測](#Detection-Face-Detection)要求加以控制。 如果 returnFaceAttributes 包含 'facialHair'，傳回的臉部就會包含臉部毛髮屬性。

如需更多詳細資料，請參閱[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)(英文\) 指南。

#### <a name="FaceList"></a>FaceList

FaceList 是 [PersistedFace](#PersistedFace) 的集合，也是[尋找相似項目](#Find-Similar)的單位。 FaceList 隨附 [FaceList 識別碼](#FaceList-ID)以及其他屬性，例如[名稱](#Name)和[使用者資料](#UserData-User-Data)。

如需更多詳細資料，請參考下列指南：[FaceList - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524b) \(英文\)、[FaceList - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524c) \(英文\)。

#### <a name="FaceList-ID"></a>FaceList 識別碼

「FaceList 識別碼」是使用者提供的字串，用來作為 [FaceList](#FaceList) 的識別碼。 「FaceList 識別碼」在訂用帳戶內必須是唯一的。

如需更多詳細資料，請參考下列指南：[FaceList - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524b) \(英文\)、[FaceList - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524c) \(英文\)。

#### <a name="Find-Similar"></a>尋找類似項目

此 API 可用來根據臉部集合，搜尋/查詢相似的臉部。 查詢臉部和臉部集合在要求中會以[臉部識別碼](#Face-ID)或 [FceList 識別碼](#FaceList-ID)/[LargeFaceList 識別碼](#LargeFaceList-ID)表示。 傳回的結果是所搜尋到的相似臉部，以[臉部識別碼](#Face-ID)或 [PersistedFace 識別碼](#PersistedFace-ID)代表。

如需更多詳細資料，請參考下列指南：[臉部 - 尋找相似項目](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) \(英文\)、[LargeFaceList - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc) \(英文\)、[FaceList - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524b) \(英文\)。

## <a name="g"></a>G

#### <a name="Gender-Attribute"></a>性別 (屬性)

性別是其中一個用來描述可用臉孔性別的[屬性](#Attributes)。 性別屬性在[偵測](#Detection-Face-Detection)結果中為選擇性，並可透過 returnFaceAttributes，以[偵測](#Detection-Face-Detection)要求加以控制。 如果 returnFaceAttributes 包含 'gender'，傳回的臉部就會包含性別屬性。

如需更多詳細資料，請參閱[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)(英文\) 指南。

#### <a name="Grouping"></a>分組

臉部分組是根據臉部相似性進行的臉部集合分組。 在要求中，會以臉部識別碼集合表示臉部集合。 進行分組後，相似的臉部會一起組成[群組](#Groups)，與任何其他臉部都不相似的臉部則會合併成一個散亂群組。 分組結果中最多只會有一個[散亂群組](#Messy-Group)。

如需更多詳細資料，請參閱[臉部 - 分組](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238) \(英文\) 指南。

#### <a name="Groups"></a>群組

群組衍生自[分組](#Grouping)結果。 每個群組皆包含一個相似臉部的集合，其中臉部會以[臉部識別碼](#Face-ID)表示。

如需更多詳細資料，請參閱[臉部 - 分組](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238) \(英文\) 指南。

## <a name="h"></a>H

#### <a name="Head-Pose-Attribute"></a>頭部姿勢 (屬性)

頭部姿勢是其中一個根據轉動、傾斜、繞 Y 軸旋轉角度，代表 3D 空間中臉部方向的[屬性](#Attributes)，如下圖所示。 轉動和繞 Y 軸旋轉的值範圍是 [-180, 180] 和 [-90, 90] 度。 在目前的版本中，從偵測傳回的傾斜值一律為 0。 頭部姿勢屬性在[偵測](#Detection-Face-Detection)結果中為選擇性，並可透過 returnFaceAttributes 參數，以[偵測](#Detection-Face-Detection)要求加以控制。 如果 returnFaceAttributes 參數包含 'headPose'，傳回的臉部就會包含頭部姿勢屬性。

如需更多詳細資料，請參閱[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)(英文\) 指南。

![GlossaryHeadPose](./Images/headpose.1.jpg)

## <a name="i"></a>I

#### <a name="Identification"></a>識別

識別是用來從 LargePersonGroup/PersonGroup 中識別一或多張臉。
[PersonGroup](#PersonGroup)/[LargePersonGroup](#LargePersonGroup)是[人員](#Person)的集合。
臉部和 LargePersonGroup/PersonGroup 在結果中會分別由[臉部識別碼](#Face-ID)和 [LargePersonGroup 識別碼](#LargePersonGroup-ID)/[PersonGroup 識別碼](#PersonGroup-ID)代表。
識別的結果會是[候選項目](#Candidate)，由[人員](#Person)搭配信賴度來代表。
輸入中若有多個臉部，將會個別考量，且每個臉部都會有自己的識別結果。

> [!NOTE]
> 進行識別之前，應該先將 LargePersonGroup/PersonGroup 成功定型。 如果未將 LargePersonGroup/PersonGroup 定型，或是定型[狀態](#Status-Train)未顯示為 'succeeded' (亦即顯示為 'running'、'failed' 或 'timeout')，要求回應就會是 400。
> 

如需更多詳細資料，請參考下列指南：[臉部 - 識別](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) \(英文\)、[LargePersonGroup 人員 - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adcba3a7b9412a4d53f40) \(英文\)、[LargePersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d) \(英文\)、[LargePersonGroup - 定型](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599ae2d16ac60f11b48b5aa4) \(英文\)、[PersonGroup 人員 - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) \(英文\)、[PersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) \(英文\)、[PersonGroup - 定型](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249) \(英文\)。

#### <a name="Is-Identical"></a>IsIdentical

IsIdentical 是[驗證](#Verification)結果的布林值欄位，用來表示兩張臉是否屬於同一個人。

如需更多詳細資料，請參閱[臉部 - 驗證](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a) \(英文\) 指南。

## <a name="j"></a>J

## <a name="k"></a>K

## <a name="l"></a>L

#### <a name="landmarks"></a>特徵點

請參閱[臉部特徵點](#Face-Landmarks-Facial-Landmarks)。

#### <a name="LargeFaceList"></a>LargeFaceList

LargeFaceList 是 [PersistedFace](#PersistedFace) 的集合，也是[尋找相似項目](#Find-Similar)的單位。 LargeFaceList 隨附 [LargeFaceList 識別碼](#LargeFaceList-ID)以及其他屬性，例如[名稱](#Name)和[使用者資料](#UserData-User-Data)。

如需更多詳細資料，請參考下列指南：[LargeFaceList - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc) \(英文\)、[LargeFaceList - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a15827cd2de3616c086f2ce) \(英文\)、[LargeFaceList - 列出臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158db4d2de3616c086f2d6) \(英文\)。

#### <a name="LargeFaceList-ID"></a>LargeFaceList 識別碼

「LargeFaceList 識別碼」是使用者提供的字串，用來作為 [LargeFaceList](#LargeFaceList) 的識別碼。 「LargeFaceList 識別碼」在訂用帳戶內必須是唯一的。

如需更多詳細資料，請參考下列指南：[LargeFaceList - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc) \(英文\)、[LargeFaceList - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a15827cd2de3616c086f2ce) \(英文\)。

#### <a name="LargePersonGroup"></a>LargePersonGroup

LargePersonGroup 是[人員](#Person)的集合，也是[識別](#Identification)的單位。 LargePersonGroup 隨附 [LargePersonGroup 識別碼](#LargePersonGroup-ID)以及其他屬性，例如[名稱](#Name)和[使用者資料](#UserData-User-Data)。

如需更多詳細資料，請參考下列指南：[LargePersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d) \(英文\)、[LargePersonGroup - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acebb6ac60f11b48b5a9e) \(英文\)、[LargePersonGroup 人員 - 列出](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adda06ac60f11b48b5aa1) \(英文\)。

#### <a name="LargePersonGroup-ID"></a>LargePersonGroup 識別碼

「LargePersonGroup 識別碼」是使用者提供的字串，用來作為 [LargePersonGroup](#LargePersonGroup) 的識別碼。 「LargePersonGroup 識別碼」在訂用帳戶內必須是唯一的。

如需更多詳細資料，請參考下列指南：[LargePersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d) \(英文\)、[LargePersonGroup - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acebb6ac60f11b48b5a9e) \(英文\)。

## <a name="m"></a>M

#### <a name="Messy-Group"></a>散亂群組

散亂群組衍生自[分組](#Grouping)結果；其中包含與任何其他臉部都不相似的臉。 散亂群組中的每張臉都會以[臉部識別碼](#Face-ID)表示。

如需更多詳細資料，請參閱[臉部 - 分組](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238) \(英文\) 指南。

## <a name="n"></a>N

#### <a name="name-person"></a>名稱 (人員)

名稱是[人員](#Person)的使用者易記描述性字串。 與[人員識別碼](#Person-ID)不同，人員的名稱在群組中可以重複。

如需更多詳細資料，請參考下列指南：[LargePersonGroup 人員 - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adcba3a7b9412a4d53f40) \(英文\)、[LargePersonGroup 人員 - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599add376ac60f11b48b5aa0) \(英文\)、[PersonGroup 人員 - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) \(英文\)、[PersonGroup 人員 - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523f) \(英文\)。

#### <a name="Name"></a>名稱 (LargePersonGroup/PersonGroup)

名稱也是 [LargePersonGroup](#LargePersonGroup)/[PersonGroup](#PersonGroup) 的使用者易記描述性字串。 與 [LargePersonGroup 識別碼](#LargePersonGroup-ID)/[PersonGroup 識別碼](#PersonGroup-ID) 不同，LargePersonGroups/PersonGroups 的名稱在訂用帳戶中可以重複。

如需更多詳細資料，請參考下列指南：[LargePersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d) \(英文\)、[LargePersonGroup - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acebb6ac60f11b48b5a9e) \(英文\)、[PersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) \(英文\)、[PersonGroup - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395246) \(英文\)。

## <a name="o"></a>O

## <a name="p"></a>P

#### <a name="PersistedFace"></a>PersistedFace

PersistedFace 是「臉部 API」中的資料結構。 PersistedFace 隨附 [PersistedFace 識別碼](#PersistedFace-ID)以及其他屬性，例如[名稱](#Name)和[使用者資料](#UserData-User-Data)。

如需更多詳細資料，請參考下列指南：[LargeFaceList - 新增臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158c10d2de3616c086f2d3) \(英文\)、[FaceList - 新增臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) \(英文\)、[LargePersonGroup 人員 - 新增臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adf2a3a7b9412a4d53f42) \(英文\)、[PersonGroup 人員 - 新增臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) \(英文\)。

#### <a name="Person-ID"></a>人員識別碼

「人員識別碼」會在成功建立 [PersistedFace](#PersistedFace) 時產生。 系統會建立一個要在[臉部 API](#Face-API) 中代表這張臉的字串。

如需更多詳細資料，請參考下列指南：[LargeFaceList - 新增臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158c10d2de3616c086f2d3) \(英文\)、[FaceList - 新增臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) \(英文\)、[LargePersonGroup 人員 - 新增臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adf2a3a7b9412a4d53f42) \(英文\)、[PersonGroup 人員 - 新增臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) \(英文\)。

#### <a name="Person"></a>人員

人員是「臉部 API」中所管理的一個資料結構。 人員隨附[人員識別碼](#Person-ID)以及其他屬性，例如[名稱](#Name)、[PersistedFace](#PersistedFace) 的集合和[使用者資料](#UserData-User-Data)。

如需更多詳細資料，請參考下列指南：[LargePersonGroup 人員 - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adcba3a7b9412a4d53f40) \(英文\)、[LargePersonGroup 人員 - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599add376ac60f11b48b5aa0) \(英文\)、[PersonGroup 人員 - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) \(英文\)、[PersonGroup 人員 - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523f) \(英文\)。

#### <a name="Person-ID"></a>人員識別碼

「人員識別碼」會在成功建立[人員](#Person)時產生。 系統會建立一個要在[臉部 API](#Face-API) 中代表這個人員的字串。

如需更多詳細資料，請參考下列指南：[LargePersonGroup 人員 - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adcba3a7b9412a4d53f40) \(英文\)、[LargePersonGroup 人員 - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599add376ac60f11b48b5aa0) \(英文\)、[PersonGroup 人員 - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) \(英文\)、[PersonGroup 人員 - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523f) \(英文\)。

#### <a name="PersonGroup"></a>PersonGroup

PersonGroup 是[人員](#Person)的集合，也是[識別](#Identification)的單位。 PersonGroup 隨附 [PersonGroup 識別碼](#PersonGroup-ID)以及其他屬性，例如[名稱](#Name)和[使用者資料](#UserData-User-Data)。

如需更多詳細資料，請參考下列指南：[PersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) \(英文\)、[PersonGroup - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395246) \(英文\)、[PersonGroup 人員 - 列出](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395241) \(英文\)。

#### <a name="PersonGroup-ID"></a>PersonGroup 識別碼

「PersonGroup 識別碼」是使用者提供的字串，用來作為 [PersonGroup](#PersonGroup) 的識別碼。 群組識別碼在訂用帳戶內必須是唯一的。

如需更多詳細資料，請參考下列指南：[PersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) \(英文\)、[PersonGroup - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395246) \(英文\)。

#### <a name="pose-attribute"></a>姿勢 (屬性)

請參閱[頭部姿勢](#Head-Pose-Attribute)。

## <a name="q"></a>Q

## <a name="r"></a>R

#### <a name="Recognition"></a>辨識

辨識是一個熱門的臉部技術應用領域，例如[尋找相似項目](#Find-Similar)、[分組](#Grouping)、[識別](#Identification)、[驗證兩張臉是否相同](#Verification)。

如需更多詳細資料，請參考下列指南：[臉部 - 尋找相似項目](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) \(英文\)、[臉部 - 分組](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238) \(英文\)、[臉部 - 識別](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) \(英文\)、 [臉部 - 驗證](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a) \(英文\)。

#### <a name="rectangle-face"></a>矩形 (臉部)

請參閱[臉部矩形](#Face-Rectangle)。

## <a name="s"></a>S

#### <a name="Smile-Attribute"></a>微笑 (屬性)

微笑是其中一個用來描述可用臉孔微笑表情的[屬性](#Attributes)。 微笑屬性在[偵測](#Detection-Face-Detection)結果中為選擇性，並可透過 returnFaceAttributes，以[偵測](#Detection-Face-Detection)要求加以控制。 如果 returnFaceAttributes 包含 'smile'，傳回的臉部就會包含微笑屬性。

如需更多詳細資料，請參閱[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)(英文\) 指南。

#### <a name="similar-face-searching"></a>相似臉部搜尋

請參閱[尋找類似項目](#Find-Similar)。

#### <a name="Status-Train"></a>狀態 (定型)

狀態是一個用來描述 [LargeFaceList/LargePersonGroups/PersonGroups 定型](#Train)程序的字串，包括 'notstarted'、'running'、'succeeded'、'failed'。

如需更多詳細資料，請參考下列指南：[LargeFaceList - 定型](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158422d2de3616c086f2d1) \(英文\)、[LargePersonGroup - 定型](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599ae2d16ac60f11b48b5aa4) \(英文\)、[PersonGroup - 定型](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249) \(英文\)。

#### <a name="subscription-key"></a>訂用帳戶金鑰

訂用帳戶金鑰是一個字串，您必須將其指定為查詢字串參數，才能叫用任何「臉部 API」。 在您登入「Microsoft 認知服務」入口網站之後，即可在 [我的訂閱] 頁面中找到訂用帳戶金鑰。 每個訂用帳戶會有兩個關聯的金鑰：一個主要金鑰和一個次要金鑰。 兩者皆可用來以相同方式叫用 API。 您必須確保訂用帳戶金鑰安全，而且也隨時可從 [我的訂閱] 頁面重新產生訂用帳戶金鑰。

## <a name="t"></a>T

#### <a name="Train"></a>定型 (LargeFaceList/LargePersonGroup/PersonGroup)

此 API 可用來對 [LargeFaceList](#LargeFaceList)/[LargePersonGroup](#LargePersonGroup)/[PersonGroup](#PersonGroup) 進行前處理，以確保[尋找相似項目](#Find-Similar)/[識別](#Identification)效能。 如果未進行定型或[定型狀態](#Status-Train)未顯示為成功，此 PersonGroup 的識別結果就會是失敗。

如需更多詳細資料，請參考下列指南：[LargeFaceList - 定型](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158422d2de3616c086f2d1) \(英文\)、[LargePersonGroup - 定型](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599ae2d16ac60f11b48b5aa4) \(英文\)、[PersonGroup - 定型](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249) \(英文\)、 [臉部 - 識別](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) \(英文\)。

## <a name="u"></a>U

#### <a name="UserData-User-Data"></a>UserData/使用者資料

使用者資料是與[人員](#Person)及 [PersonGroup](#PersonGroup)/[LargePersonGroup](#LargePersonGroup) 相關的額外資訊。 使用者資料是由使用者所設定，可讓資料更容易被使用、了解及記住。

如需更多詳細資料，請參考下列指南：[LargePersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d) \(英文\)、[LargePersonGroup - 更新](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acfc83a7b9412a4d53f3f) \(英文\)、[LargePersonGroup 人員 - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adcba3a7b9412a4d53f40) \(英文\)、[LargePersonGroup 人員 - 更新](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599ade043a7b9412a4d53f41) \(英文\)、[PersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) \(英文\)、[PersonGroup - 更新](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524a) \(英文\)、[PersonGroup 人員 - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) \(英文\)、[PersonGroup 人員 - 更新](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395242) \(英文\)。

## <a name="v"></a>V

#### <a name="Verification"></a>驗證

此 API 可用來驗證兩張臉是否相同。 兩張臉在要求中都會由臉部識別碼代表。 驗證的結果會包含一個布林值欄位 ([isIdentical](#Is-Identical))，如果為 true，即表示相同，還包含一個數字欄位 ([confidence](#Confidence))，可指出信賴度等級。

如需更多詳細資料，請參閱[臉部 - 驗證](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a) \(英文\) 指南。

## <a name="w"></a>W

## <a name="x"></a>X

## <a name="y"></a>Y

## <a name="z"></a>Z
