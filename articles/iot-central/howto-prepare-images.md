---
title: 上傳影像至您的 Azure IoT Central 應用程式 | Microsoft Docs
description: 了解如何以建置者的身分準備及上傳影像至您的 Azure IoT Central 應用程式。
author: tbhagwat3
ms.author: tanmayb
ms.date: 04/16/2018
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: 7fd9c8ed5559b00bc755e3f04c768dceeb487562
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34628016"
---
# <a name="prepare-and-upload-images-to-your-azure-iot-central-application"></a>準備及上傳影像至您的 Azure IoT Central 應用程式

本文說明如何以建置者的身分，透過上傳自訂影像來自訂您的 Microsoft Azure IoT Central 應用程式。 例如，您可以使用裝置的圖片來自訂裝置儀表板。

## <a name="before-you-begin"></a>開始之前

若要完成這篇文章中的步驟，您需要下列項目︰

1. Azure IoT Central 應用程式。 如需詳細資訊，請參閱[建立 Azure IoT Central 應用程式](howto-create-application.md)。
1. 適用於對影像檔案進行縮放及調整大小的工具。

## <a name="choose-where-to-use-custom-images"></a>選擇要使用自訂影像的位置

您可以將自訂影像新增至 Azure IoT Central 應用程式中的下列位置：

* [應用程式管理員] 頁面

    ![[應用程式管理員] 頁面上的影像](media/howto-prepare-images/applicationmanager.png)

* 首頁

    ![首頁上的影像](media/howto-prepare-images/homepage.png)

* 裝置範本

    ![裝置範本上的影像](media/howto-prepare-images/devicetemplate.png)

* 裝置儀表板上的磚

    ![裝置磚上的影像](media/howto-prepare-images/devicetile.png)

* 裝置集合儀表板上的磚

    ![裝置集合磚上的影像](media/howto-prepare-images/devicesettile.png)

## <a name="prepare-the-images"></a>準備影像

在上述四個位置中，您可以使用 PNG、GIF 或 JPEG 影像。

下表摘要說明您可以使用的影像大小：

| 位置 | 大小 |
| -------- | ------ |
| **應用程式管理員** | 268x160 px |
| 裝置範本 | 64x64 px |
| 首頁和儀表板磚 | 最小的磚大小為 200x200 px，較大的磚則可以是由多個小磚組合而成的正方形或矩形。 例如 200x400 px、400x200 px 或 400x400 px |

如需在應用程式中取得最佳顯示效果，您應該建立符合上表所述尺寸的影像。

## <a name="upload-the-images"></a>上傳影像

下列各節說明如何上傳影像以用於不同位置：

### <a name="application-manager"></a>應用程式管理員

若要上傳影像以用於 [應用程式管理員] 上，請瀏覽至 [管理] 區段的 [應用程式設定] 頁面。 您必須具有系統管理員的身分以完成此工作：

![上傳應用程式影像](media/howto-prepare-images/uploadapplicationmanager.png)

按一下上傳影像的圖示，並選擇要從本機電腦上傳的檔案。

### <a name="home-page"></a>首頁

若要上傳影像以用於首頁，請瀏覽至您應用程式的 [首頁] 並開啟設計模式。 您必須具有建置者的身分以完成此工作：

![上傳首頁影像](media/howto-prepare-images/uploadhomepage.png)

按一下上傳影像的圖示，並選擇要從本機電腦上傳的檔案。

在影像上傳之後，您可以在設計模式開啟的情況下調整它的大小。

### <a name="device-template"></a>裝置範本

若要上傳影像以用於裝置範本，請瀏覽至 [Device Explorer]，選擇裝置範本並選擇裝置，然後開啟設計模式。 您必須具有建置者的身分以完成此工作：

![上傳裝置範本影像](media/howto-prepare-images/uploaddevicetemplate.png)

按一下上傳影像的圖示，並選擇要從本機電腦上傳的檔案。

### <a name="device-dashboard"></a>裝置儀表板

若要上傳影像以用於裝置儀表板，請瀏覽至 [Device Explorer]，選擇裝置範本，然後選擇裝置。 然後選擇 [儀表板] 頁面並開啟設計模式。 您必須具有建置者的身分以完成此工作：

![上傳裝置儀表板影像](media/howto-prepare-images/uploaddevicedashboard.png)

按一下上傳影像的圖示，並選擇要從本機電腦上傳的檔案。

在影像上傳之後，您可以在 [設計模式] 開啟的情況下調整它的大小及位置。

### <a name="device-set-dashboard"></a>裝置集合儀表板

若要上傳影像以用於裝置集合儀表板，請瀏覽至 [裝置集合]，選擇裝置集合，然後選擇裝置。 然後選擇 [儀表板] 頁面並開啟 [設計模式]：

![上傳裝置集合儀表板影像](media/howto-prepare-images/uploaddevicesetdashboard.png)

按一下上傳影像的圖示，並選擇要從本機電腦上傳的檔案。

在影像上傳之後，您可以在設計模式開啟的情況下調整它的大小及位置。

## <a name="next-steps"></a>後續步驟

您現在已了解如何準備及上傳影像至 Azure IoT Central 應用程式，以下是建議的後續步驟：

> [!div class="nextstepaction"]
> [在 Azure IoT Central 應用程式中管理裝置](howto-manage-devices.md)