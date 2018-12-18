---
title: 範例：偵測影像中的人臉 - 臉部 API
titleSuffix: Azure Cognitive Services
description: 使用臉部 API 偵測影像中的人臉。
services: cognitive-services
author: SteveMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: sample
ms.date: 03/01/2018
ms.author: sbowles
ms.openlocfilehash: a4c74ff70a4426abf97562bf997479a91afbf17a
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46124043"
---
# <a name="example-how-to-detect-faces-in-image"></a>範例：如何偵測影像中的人臉

本指南會示範如何偵測影像中的人臉及臉部屬性，例如性別、年齡或所擷取的姿勢。 這些範例是使用「臉部 API」用戶端程式庫以 C# 撰寫的。 

## <a name="concepts"></a>概念

如果您不熟悉本指南中的下列任何概念，請隨時參閱[詞彙](../Glossary.md)中的定義： 

- 臉部偵測
- 臉部特徵點
- 頭部姿勢
- 臉部屬性

## <a name="preparation"></a>準備工作

此範例會示範下列功能： 

- 偵測影像中的人臉，並使用矩形框架標示它們
- 分析瞳孔、鼻子或嘴巴的位置，然後在影像中標示出來
- 分析臉部的頭部姿勢、性別和年齡

為了能夠執行這些功能，您必須準備至少有一張清楚臉部的影像。 

## <a name="step-1-authorize-the-api-call"></a>步驟 1：授權 API 呼叫

每次呼叫臉部 API 時，都需要訂用帳戶金鑰。 這個金鑰必須透過查詢字串參數傳遞，或者在要求標頭中指定。 若要透過查詢字串傳遞訂用帳戶密鑰，請參閱所舉出的[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)要求 URL 範例：

```
https://westus.api.cognitive.microsoft.com/face/v1.0/detect[?returnFaceId][&returnFaceLandmarks][&returnFaceAttributes]
&subscription-key=<Subscription Key>
```

或者，您也可以在 HTTP 要求標頭中指定訂用帳戶金鑰：**ocp-apim-subscription-key: &lt;訂用帳戶金鑰&gt;**。若您使用用戶端程式庫，則會透過 FaceServiceClient 類別的建構函式傳入訂用帳戶金鑰。 例如︰
```CSharp
faceServiceClient = new FaceServiceClient("<Subscription Key>");
```

## <a name="step-2-upload-an-image-to-the-service-and-execute-face-detection"></a>步驟 2：將影像上傳至服務並執行臉部偵測

若要執行臉部偵測，最基本的方式是直接上傳影像。 您可以傳送 "POST" 要求並附上應用程式/八位元資料流內容類型以及從 JPEG 影像讀取的資料，便可搞定一切。 影像大小上限為 4 MB。

使用用戶端程式庫時，藉由傳入資料流物件即可透過上傳方式執行臉部偵測。 請參閱下面的範例：

```CSharp
using (Stream s = File.OpenRead(@"D:\MyPictures\image1.jpg"))
{
    var faces = await faceServiceClient.DetectAsync(s, true, true);
 
    foreach (var face in faces)
    {
        var rect = face.FaceRectangle;
        var landmarks = face.FaceLandmarks;
    }
}
```

請注意，FaceServiceClient 的 DetectAsync 方法是「非同步的」。 呼叫的方法也應該標示為非同步，才能使用 await 子句。
如果影像已在網路上且具有 URL，您也可以藉由提供 URL 來執行臉部偵測。 在這個範例中，要求主體將會是包含 URL 的 JSON 字串。
使用用戶端程式庫時，可以藉由使用 DetectAsync 方法的另一個多載，輕鬆地透過 URL 執行臉部偵測。

```CSharp
string imageUrl = "http://news.microsoft.com/ceo/assets/photos/06_web.jpg";
var faces = await faceServiceClient.DetectAsync(imageUrl, true, true);
 
foreach (var face in faces)
{
    var rect = face.FaceRectangle;
    var landmarks = face.FaceLandmarks;
}
``` 

基本上，和偵測到的臉部一起傳回的 FaceRectangle 屬性，會是臉部上的各個位置 (以像素為單位)。 這個矩形通常會包含眼睛、眉毛、鼻子和嘴巴 – 頭頂、耳朵和下巴則不會包含在內。 如果您裁剪出完整的頭部或中景人像 (附照證件類型的影像)，由於這類影像的臉部區域對某些應用程式來說可能太小，建議您將其矩形臉部框架的區域擴大。 若要更精準地定位臉部，請使用下一節所述且證明有用的特徵點 (定位臉部特徵或臉部方向機制)。

## <a name="step-3-understanding-and-using-face-landmarks"></a>步驟 3：了解和使用臉部特徵點

臉部特徵點是臉部上的一系列細部位置點；通常是臉部器官的位置點，例如瞳孔、兩眥或鼻子。 臉部特徵點是選擇性屬性，可在臉部偵測進行期間進行分析。 您可以在呼叫[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)時對 returnFaceLandmarks 查詢參數傳遞布林值 'true'，也可以對 FaceServiceClient 類別 DetectAsync 方法使用 returnFaceLandmarks 選擇性參數，以便在偵測結果中包括臉部特徵點。

預設會有 27 個預先定義的臉部特徵點。 下圖顯示這 27 個位置點的定義方式：

![HowToDetectFace](../Images/landmarks.1.jpg)

所傳回的位置點是以像素為單位，就和臉部矩形框架一樣。 因此，您可以輕鬆地在影像中標出感興趣的特定位置點。 下列程式碼示範如何擷取鼻子和瞳孔的位置：

```CSharp
var faces = await faceServiceClient.DetectAsync(imageUrl, returnFaceLandmarks:true);
 
foreach (var face in faces)
{
    var rect = face.FaceRectangle;
    var landmarks = face.FaceLandmarks;
 
    double noseX = landmarks.NoseTip.X;
    double noseY = landmarks.NoseTip.Y;
 
    double leftPupilX = landmarks.PupilLeft.X;
    double leftPupilY = landmarks.PupilLeft.Y;
 
    double rightPupilX = landmarks.PupilRight.X;
    double rightPupilY = landmarks.PupilRight.Y;
}
``` 

除了在影像中標出臉部特徵外，臉部特徵點也可用來正確計算臉部的朝向。 例如，我們可以將臉部的朝向定義為從嘴巴中心點到雙眼中心點的向量。 下列程式碼會對此做出詳細說明：

```CSharp
var landmarks = face.FaceLandmarks;
 
var upperLipBottom = landmarks.UpperLipBottom;
var underLipTop = landmarks.UnderLipTop;
 
var centerOfMouth = new Point(
    (upperLipBottom.X + underLipTop.X) / 2,
    (upperLipBottom.Y + underLipTop.Y) / 2);
 
var eyeLeftInner = landmarks.EyeLeftInner;
var eyeRightInner = landmarks.EyeRightInner;
 
var centerOfTwoEyes = new Point(
    (eyeLeftInner.X + eyeRightInner.X) / 2,
    (eyeLeftInner.Y + eyeRightInner.Y) / 2);
 
Vector faceDirection = new Vector(
    centerOfTwoEyes.X - centerOfMouth.X,
    centerOfTwoEyes.Y - centerOfMouth.Y);
``` 

藉由知道臉部的朝向，您就可以旋轉矩形臉部框架以與臉部貼齊。 很顯然地，使用臉部特徵點可以提供更多細節和用處。

## <a name="step-4-using-other-face-attributes"></a>步驟 4：使用其他臉部屬性

除了臉部特徵點外，「臉部 - 偵測」API 也可以分析臉部的數個其他屬性。 這些屬性包括：

- 年齡
- 性別
- 笑容程度
- 臉部毛髮
- 3D 頭部姿勢

這些屬性是使用統計演算法來做出預測的，不一定會 100% 精確。 不過，當您想要以這些屬性來將臉部分類時，這些屬性仍然很有用。 如需各個屬性的詳細資訊，請參閱[詞彙](../Glossary.md)。

以下是在臉部偵測進行期間擷取臉部屬性的簡單範例：

```CSharp
var requiredFaceAttributes = new FaceAttributeType[] {
                FaceAttributeType.Age,
                FaceAttributeType.Gender,
                FaceAttributeType.Smile,
                FaceAttributeType.FacialHair,
                FaceAttributeType.HeadPose,
                FaceAttributeType.Glasses
            };
var faces = await faceServiceClient.DetectAsync(imageUrl,
    returnFaceLandmarks: true,
    returnFaceAttributes: requiredFaceAttributes);

foreach (var face in faces)
{
    var id = face.FaceId;
    var attributes = face.FaceAttributes;
    var age = attributes.Age;
    var gender = attributes.Gender;
    var smile = attributes.Smile;
    var facialHair = attributes.FacialHair;
    var headPose = attributes.HeadPose;
    var glasses = attributes.Glasses;
}
``` 

## <a name="summary"></a>總結

在本指南中，您已了解[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) API 的功能；它會如何偵測本機上傳影像或網路上影像 URL 的臉部；它會如何藉由傳回矩形臉部框架來偵測臉部；以及它也會如何分析臉部特徵點、3D 頭部姿勢和其他臉部屬性。

如需 API 詳細資料的詳細資訊，[請參閱](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)的 API 參考指南。

## <a name="related-topics"></a>相關主題

[如何識別影像中的人臉](HowtoIdentifyFacesinImage.md)
