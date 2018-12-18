---
title: 將您的自訂視覺服務模型匯出至行動裝置 - 自訂視覺服務 - Azure 認知服務 | Microsoft Docs
description: 了解如何匯出您的模型，用以建立行動應用程式。
services: cognitive-services
author: anrothMSFT
manager: corncar
ms.service: cognitive-services
ms.component: custom-vision
ms.topic: article
ms.date: 05/03/2018
ms.author: anroth
ms.openlocfilehash: ce8f42d6239867dd217cddfc61a27d7835dc9c9b
ms.sourcegitcommit: 680964b75f7fff2f0517b7a0d43e01a9ee3da445
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "35370866"
---
# <a name="export-your-model-for-use-with-mobile-devices"></a>匯出您的模型以用於行動裝置

自訂視覺服務可讓分類器匯出而離線執行。 您可以將匯出的分類器內嵌到應用程式中，並裝置的本機執行以進行即時分類。 

自訂視覺服務支援下列匯出：

* __Android__ 的 __Tensorflow__。
* __iOS11__ 的 __CoreML__。
* __Windows ML__ 的 __ONNX__。
* Windows 或 Linux __容器__。 此容器包含要使用自訂視覺服務 API 的 Tensorflow 模型和服務程式碼。 

> [!IMPORTANT]
> 自訂視覺服務只會匯出__密集__區域。 密集區域所產生的模型會針對行動裝置的即時分類條件約束進行最佳化。 以密集區域建置的分類器精確性可能會略低於具有等量訓練資料的標準區域。
>
> 如需改善分類器的相關資訊，請參閱[改善您的分類器](getting-started-improving-your-classifier.md)文件。

## <a name="convert-to-a-compact-domain"></a>轉換為密集區域

> [!NOTE]
> 您有未設定為密集區域的現有分類器時，才適用本節提供的步驟。
 
若要轉換現有分類器的區域，請使用下列步驟：

1. 在 [自訂視覺](https://customvision.ai) 頁面中選取 __首頁__ 圖示，以檢視您的專案清單。 您也可以使用 [https://customvision.ai/projects](https://customvision.ai/projects) 來檢視您的專案。

    ![首頁圖示和專案清單的影像](./media/export-your-model/projects-list.png)

2. 選取專案，然後選取頁面右上方的 [齒輪] 圖示。

    ![齒輪圖示的影像](./media/export-your-model/gear-icon.png)

3. 在 [區域] 區段中，選取 [密集] 網域。 選取 [儲存變更] 以儲存變更。

    ![區域選取項目的影像](./media/export-your-model/domains.png)

4. 在頁面頂端選取 [訓練]，以使用新的區域進行重新訓練。

## <a name="export-your-model"></a>匯出您的模型

若要在重新訓練之後匯出模型，請使用下列步驟：

1. 移至 [效能] 索引標籤，並選取 [匯出]。 

    ![匯出圖示的影像](./media/export-your-model/export.png)

    > [!TIP]
    > 如果 [匯出] 項目無法使用，則選取的反覆項目不會使用密集區域。 請使用此頁面的 [反覆項目] 區段選取使用密集區域的反覆項目，然後選取 [匯出]。

2. 選取匯出格式，然後選取 [匯出] 以下載模型。

## <a name="next-steps"></a>後續步驟

將您匯出的模型整合到應用程式中。 有數個範例應用程式可供使用：

* [在 iOS 應用程式中使用匯出的 CoreML 模型](https://go.microsoft.com/fwlink/?linkid=857726)的範例可用於使用 Swift 的即時影像分類
* [將匯出的 CoreML 模型用於 Xamarin](https://github.com/xamarin/ios-samples/tree/master/ios11/CoreMLAzureModel)的範例 iOS 應用程式可用於即時影像分類 
* [在 Android 應用程式中使用匯出的 Tensorflow 模型](https://github.com/Azure-Samples/cognitive-services-android-customvision-sample)的範例可用於即時影像分類 
* [將 Tensorflow 模型用於 Windows](https://docs.microsoft.com/en-us/azure/cognitive-services/custom-vision-service/export-model-python)
* [將匯出的 ONNX 模型用於 Windows 機器學習](https://azure.microsoft.com/en-us/resources/samples/cognitive-services-onnx-customvision-sample/)的範例
