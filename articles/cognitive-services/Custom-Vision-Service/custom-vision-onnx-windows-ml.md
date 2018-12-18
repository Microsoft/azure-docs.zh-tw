---
title: 教學課程：搭配 Windows ML 使用 ONNX 模型 - 自訂視覺服務
titlesuffix: Azure Cognitive Services
description: 了解如何建立 Windows UWP 應用程式來使用從 Azure 認知服務匯出的 ONNX 模型。
services: cognitive-services
author: larryfr
manager: cgronlun
ms.service: cognitive-services
ms.component: custom-vision
ms.topic: tutorial
ms.date: 10/16/2018
ms.author: larryfr
ms.openlocfilehash: f9ffbdd34c6fe7e8fc0f5e8ad8a49a3ad922f88a
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49362957"
---
# <a name="tutorial-use-an-onnx-model-from-custom-vision-with-windows-ml-preview"></a>教學課程：搭配使用自訂視覺中的 ONNX 模型與 Windows ML (預覽)

了解如何搭配使用從自訂視覺匯出的 ONNX 模型與 Windows ML (預覽)。

本文件中的資訊會示範如何搭配使用從自訂視覺服務匯出的 ONNX 檔案與 Windows ML。 我們會提供 Windows UWP 的範例應用程式。 範例中包含可辨識的已定型模型。 也會提供如何使用自有模型來搭配此範例的步驟。

> [!div class="checklist"]
> * 關於範例應用程式
> * 取得範例程式碼
> * 執行範例
> * 使用自有模型

## <a name="prerequisites"></a>必要條件

* Windows 10 組建 17738 或更新版本

* 適用於組建 17738 或更新版本的 Windows SDK

* 啟用__通用 Windows 平台開發__工作負載的 Visual Studio 2017 15.7 版或更新版本。

* 已啟用開發人員模式。 如需詳細資訊，請參閱[啟用您的裝置以用於開發](https://docs.microsoft.com/windows/uwp/get-started/enable-your-device-for-development)文件。

## <a name="about-the-example-app"></a>關於範例應用程式

應用程式是一般的 Windows UWP 應用程式。 它可讓您選取電腦中的映像，然後將它傳送到模型。 模型所傳回的標籤和分數會顯示在映像旁邊。

## <a name="get-the-example-code"></a>取得範例程式碼

範例應用程式可在 [https://github.com/Azure-Samples/cognitive-services-onnx12-customvision-sample/](https://github.com/Azure-Samples/cognitive-services-onnx12-customvision-sample/) 取得。

## <a name="run-the-example"></a>執行範例

1. 使用 `F5` 鍵從 Visual Studio 中啟動應用程式。 系統可能會提示您啟用開發人員模式。 如需詳細資訊，請參閱[啟用您的裝置以用於開發](https://docs.microsoft.com/windows/uwp/get-started/enable-your-device-for-development)文件。

2. 當應用程式啟動時，使用此按鈕來選取映像進行評分。

## <a name="use-your-own-model"></a>使用自有模型

若要使用您自己的模型，請使用下列步驟：

1. 使用自訂視覺服務[建立及訓練](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/getting-started-build-a-classifier)分類器。 若要匯出模型，請選取 __精簡__網域，例如 [一般 (精簡)]。 若要匯出現有的分類器，請選取右上方的齒輪圖示，將網域轉換為精簡。 在 [設定] 中，選擇精簡模型，儲存及訓練您的專案。  

1. 移至 [效能] 索引標籤，以[匯出您的模型](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/export-your-model)。選取使用精簡網域訓練的反覆項目，[匯出] 按鈕就會出現。 選取 [匯出]、[ONNX]、[ONNX1.2]，然後選取 [匯出]。 檔案準備就緒後，請選取 [下載] 按鈕。

1. 將 ONNX 檔案放入您專案的 __Assets__ 資料夾。 

1. 在 [方案總管] 中，以滑鼠右鍵按一下 Assets 資料夾，然後選取 [新增現有的項目]。 選取 ONNX 檔案。

1. 在 [方案總管] 中，從 Assets 資料夾選取 ONNX 檔案。 變更檔案的下列屬性： 
    
    * __建置動作__ -> __內容__
    * __複製到輸出目錄__ -> __有更新時才複製__

1. 將 `_onnxFileNames` 變數變更為 ONNX 檔案名稱。 此外，將 `ClassLabel` 修改為模型內含的標籤數目。

1. 建置和執行。

1. 按一下按鈕以選取要評估的映像。

## <a name="next-steps"></a>後續步驟

若要探索其他匯出及使用自訂視覺模型的方法，請參閱下列文件：

* [匯出您的模型](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/export-your-model)
* [在 Android 應用程式中使用匯出的 Tensorflow 模型](https://github.com/Azure-Samples/cognitive-services-android-customvision-sample)
* [在 Swift iOS 應用程式中使用匯出的 CoreML 模型](https://go.microsoft.com/fwlink/?linkid=857726)
* [搭配使用 iOS 應用程式中的 CoreML 模型與 Xamarin](https://github.com/xamarin/ios-samples/tree/master/ios11/CoreMLAzureModel)

如需有關如何搭配使用 ONNX 模型與 Windows ML 的詳細資訊，請參閱[使用 Windows ML 將模型整合到您的應用程式中](https://docs.microsoft.com/windows/uwp/machine-learning/integrate-model)文件。
