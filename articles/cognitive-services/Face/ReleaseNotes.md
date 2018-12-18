---
title: 臉部 API 服務的版本資訊 | Microsoft Docs
titleSuffix: Microsoft Cognitive Services
description: 臉部 API 服務的版本資訊包括各種版本的發行變更歷程記錄。
services: cognitive-services
author: SteveMSFT
manager: corncar
ms.service: cognitive-services
ms.component: face-api
ms.topic: article
ms.date: 03/01/2018
ms.author: sbowles
ms.openlocfilehash: 918b3ea5dbaaa44e4eee1a679354c7becffd4686
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370307"
---
# <a name="face-api-release-notes"></a>臉部 API 版本資訊

本文適用於 Microsoft 臉部 API 服務 1.0 版。

### <a name="release-changes-in-may-2018"></a>2018 年 5 月的發行變更

* 已大幅改善 `gender` 屬性，並已改善 `age`、`glasses`、`facialHair`、`hair`、`makeup` 屬性。 您可以透過 [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) (臉部 - 偵測) 的 `returnFaceAttributes` 參數來使用這些屬性。 

* 在 [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) (臉部 - 偵測)、[FaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) (FaceList - 新增臉部)、[LargeFaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158c10d2de3616c086f2d3) (LargeFaceList - 新增臉部)、[PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) (PersonGroup Person - 新增臉部) 和 [LargePersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adf2a3a7b9412a4d53f42) (LargePersonGroup Person - 新增臉部) 中，輸入影像檔大小限制從 4 MB 提高到 6 MB。

### <a name="release-changes-in-march-2018"></a>2018 年 3 月的發行變更

* 新增百萬規模容器：[LargeFaceList](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc) 和 [LargePersonGroup](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d)。 如需詳細資料，請參閱[如何使用大規模功能](Face-API-How-to-Topics/how-to-use-large-scale.md)。

* 將 [Face - Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) (臉部 - 識別) 的 `maxNumOfCandidatesReturned` 參數從 [1, 5] 提高到 [1, 100]，預設為 10。

### <a name="release-changes-in-may-2017"></a>2017 年 5 月的發行變更

* 在 [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) (臉部 - 偵測) 的 `returnFaceAttributes` 參數中新增 `hair`、`makeup`、`accessory`、`occlusion`、`blur`、`exposure` 和 `noise` 屬性。

* 在 PersonGroup 和 [Face - Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) (臉部 - 識別) 中支援 一萬人。

* [PersonGroup Person - List](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395241) (PersonGroup Person - 列出) 支援分頁，其選擇性參數為：`start` 和 `top`。

* 支援同時對不同的 FaceList 和 PersonGroup 中的不同人員新增/刪除臉部。

### <a name="release-changes-in-march-2017"></a>2017 年 3 月的發行變更
* 在 [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) (臉部 - 偵測) 的 `returnFaceAttributes` 參數中新增 `emotion` 屬性。

* 修正無法使用從 [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) (臉部 - 偵測) 傳回的矩形，重新偵測臉部作為 [FaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) (FaceList - 新增臉部) 和 [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) (PersonGroup Person - 新增臉部) 中的 `targetFace`。

* 修正可偵測的臉部大小，確保它完全介於 36x36 到 4096x4096 像素之間。

### <a name="release-changes-in-november-2016"></a>2016 年 11 月的發行變更
* 新增臉部儲存體標準訂用帳戶，在使用 [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) (PersonGroup Person - 新增臉部) 或 [FaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) (FaceList - 新增臉部) 時，持續儲存更多的臉部，供比對識別及相似度之用。 儲存影像的計費方式為每 1000 張人臉美金 $0.5，並會依使用天數按比例計算。 免費層訂用帳戶的總人數會繼續僅限 1,000 名。

### <a name="release-changes-in-october-2016"></a>2016 年 10 月的發行變更
* 在 [FaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) (FaceList - 新增臉部) 和 [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) (PersonGroup Person - 新增臉部) 中，將 targetFace 中有多個臉部的錯誤訊息，從 'There are more than one face in the image' 變更為 'There is more than one face in the image'。

### <a name="release-changes-in-july-2016"></a>2016 年 7 月的發行變更
* 在 [Face - Verify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a) (臉部 - 驗證) 中支援 Face 對 Person 物件的驗證。

* 在 [Face - Find Similar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) (臉部 - 尋找類似項目) 中新增選擇性 `mode` 參數，讓您可以選取兩個工作模式：`matchPerson` 和 `matchFace`，預設為 `matchPerson`。

* 在 [Face - Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) (臉部 - 識別) 中新增選擇性 `confidenceThreshold` 參數，讓使用者可以設定某個臉部是否屬於 Person 物件的閾值。

* 在 [PersonGroup - List](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248) (PersonGroup - 列出) 中新增選擇性 `start` 和 `top` 參數，讓使用者可以為清單指定起點和 PersonGroup 總數。

### <a name="v10-changes-from-v0"></a>從 V0 變更為 V1.0
* 將服務根端點從 ```https://westus.api.cognitive.microsoft.com/face/v0/``` 更新為 ```https://westus.api.cognitive.microsoft.com/face/v1.0/```。 變更適用於：[Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) (臉部 - 偵測)、[Face - Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) (臉部 - 識別)、[Face - Find Similar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) (臉部 - 尋找類似項目) 和 [Face - Group](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238) (臉部 - 分組)。

* 將可偵測的臉部大小下限更新為 36x36 像素。 小於 36x36 像素的臉部將無法被偵測。

* 臉部 V0 中的 PersonGroup 和 Person 資料已淘汰。 無法使用臉部 V1.0 服務存取這些資料。

* 臉部 API 的 V0 端點已於 2016 年 6 月 30 日淘汰。
