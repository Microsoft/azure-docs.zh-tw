---
title: 在 Azure IoT Central 應用程式中使用裝置集合 | Microsoft Docs
description: 如何以操作員的身分在 Azure IoT Central 應用程式中使用裝置集合。
author: ellenfosborne
ms.author: elfarber
ms.date: 01/21/2018
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: peterpfr
ms.openlocfilehash: 28706ad77f48ae826b621ebdd920d26f3b87178a
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45731969"
---
# <a name="use-device-sets-in-your-azure-iot-central-application"></a>在 Azure IoT Central 應用程式中使用裝置集合

本文說明如何以操作員的身分在 Microsoft Azure IoT Central 應用程式中使用裝置集合。

裝置集合是一份裝置清單，這些裝置因為全都符合某些指定準則而群組在一起。 裝置集合可藉由將裝置分組到較小的邏輯群組，來協助您大規模地對裝置進行管理、視覺化和分析。 例如，您建立了一份西雅圖所有空調裝置的清單，讓位於西雅圖的技師可以找到其負責的所有裝置。 本文說明如何建立和設定裝置集合。

## <a name="create-a-device-set"></a>建立裝置集合

若要建立裝置集合：

1. 在左側導覽功能表上選擇 [裝置集合]。

1. 按一下 [+ 新增]。

    ![新增裝置集合](media/howto-use-device-sets/image1.png)

1. 為裝置集合提供在整個應用程式中獨一無二的名稱。 您也可以新增描述。 裝置集合只能包含來自單一裝置範本的裝置。 選擇要用於此集合的裝置範本。

1. 建立查詢，藉由選取屬性、比較運算子和值來識別裝置集合的裝置。 您可以新增多個查詢，而符合**所有**準則的裝置則會放置在裝置集合中。 能存取應用程式的任何人都能存取您所建立的裝置集合，因此任何人都能檢視、修改或刪除裝置集合。

    ![裝置集合查詢](media/howto-use-device-sets/image2.png)

    > [!NOTE]
    > 裝置集合是動態的查詢。 您每次檢視裝置清單時所看到的裝置可能會不同。 清單內容取決於哪些裝置目前符合查詢準則。

1. 選擇 [儲存]。

## <a name="configure-the-dashboard-for-your-device-set"></a>設定裝置集合的儀表板

建立裝置集合後，就可以設定其 [儀表板]。 [儀表板] 是可供放置影像和連結的首頁。 您也可以新增會列出裝置集合所含裝置的格線。

1. 在左側導覽功能表上選擇 [裝置集合]。

1. 選擇 [儀表板] 索引標籤。

1. 按一下 [編輯範本]。

    ![設計模式開啟](media/howto-use-device-sets/image3.png)

1. 如需如何新增影像的資訊，請參閱[準備及上傳影像至您的 Azure IoT Central 應用程式](howto-prepare-images.md)。

1. 新增連結圖格：
    1. 選擇右窗格上的 [連結]。
    1. 為連結提供**標題**。
    1. 選擇要在按下連結時開啟的 URL。
    1. 為連結提供描述，此描述會顯示在**標題**下面。
    1. 選擇 [儲存]。

        ![儲存連結](media/howto-use-device-sets/image7.png)

    1. 您可以在 [儀表板] 上移動連結圖格或調整其大小。

1. 新增格線。 格線是一份資料表，內含裝置集合中的裝置以及您所選擇的資料行。
    1. 選擇右窗格上的 [格線]。

        ![選擇格線](media/howto-use-device-sets/image8.png)

    1. 為格線提供**標題**。
    1. 藉由選擇 [設定] 按鈕來選取要顯示的資料行。 在跳出的面板中選擇您想要顯示的資料行，並選擇向右箭號來加以選取。
    1. 選擇 [確定]。
    1. 選擇 [儲存]。

        ![儲存格線](media/howto-use-device-sets/image9.png)

    1. 拖放格線以將其放在 [儀表板] 上。

    > [!NOTE]
    > 您可以新增多個影像、連結和格線。
  
    1. 按一下 [完成] 。

    ![設計模式關閉](media/howto-use-device-sets/image10.png)


### <a name="configuring-location-map-in-your-device-sets-dashboard"></a>在裝置集合儀表板中設定位置圖 
您可以在地圖中新增位置圖，以將裝置集合位置視覺化。

若要將位置圖新增到裝置集合儀表板，您必須先在 [裝置範本] 中設定好位置屬性，請參閱[建立位置屬性 (採用 Azure 地圖服務)](howto-set-up-template.md)。


1. 在 [裝置集合儀表板] 上，從資源庫選取 [地圖]。

    ![裝置集合儀表板地圖](media/howto-use-device-sets/LocationMaps1.png)

2. 提供標題，然後選擇您先前在 [裝置屬性] 中設定的位置屬性。
3. 儲存之後，就會在 [裝置集合] 中看到顯示您裝置位置的地圖磚。
4. 現在，當操作員檢視裝置集合儀表板時，就可以看到您設定的所有磚 (包括位置圖格)，一眼就能看到以視覺化方式呈現的所有裝置位置！ 
    
[!NOTE] 您可以將地圖調整為需要的大小。 按一下地圖中的大頭針，即會顯示裝置資訊、名稱和位置。 您可以按一下快顯視窗，然後開啟裝置屬性頁。  


## <a name="configure-the-list-for-your-device-set"></a>設定裝置集合的清單

建立裝置集合後，就可以設定 [清單]。 [清單] 會在資料表中顯示裝置集合中的所有裝置以及您所選擇的資料行。

1. 在左側導覽功能表上選擇 [裝置集合]。

1. 選擇 [清單] 索引標籤。

1. 選擇 [資料行選項]。

    ![資料行選項](media/howto-use-device-sets/image11.png)

1. 藉由選取您想要顯示的資料行，然後選擇向右箭號來加以選取，以選擇所要顯示的資料行。

    ![選擇資料行](media/howto-use-device-sets/image12.png)

1. 選擇 [確定]。

## <a name="analytics"></a>分析

裝置集合中的分析等同於左側導覽功能表中的 [主要分析] 索引標籤。 您可以在關於[如何建立分析](howto-create-analytics.md)的文章中深入了解分析。

## <a name="next-steps"></a>後續步驟

您現在已了解如何在 Azure IoT Central 應用程式中使用裝置集合，以下是建議的後續步驟：

> [!div class="nextstepaction"]
> [如何建立遙測規則](howto-create-telemetry-rules.md)
