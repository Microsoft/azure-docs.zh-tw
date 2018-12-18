---
title: 如何使用 Azure 地圖服務的地圖控制項 | Microsoft Docs
description: 了解如何使用 Azure 地圖服務的地圖控制項用戶端 Javascript 程式庫。
author: dsk-2015
ms.author: dkshir
ms.date: 09/05/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: 45890b4bd474c010b2b086be0405b79d340aeebd
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45603154"
---
# <a name="use-the-azure-maps-map-control"></a>使用 Azure 地圖服務的地圖控制項

地圖控制項用戶端 Javascript 程式庫可讓您將地圖和內嵌的 Azure 地圖服務功能轉譯到 Web 或行動裝置應用程式中。

## <a name="create-a-new-map-in-a-web-page"></a>在網頁中建立新的地圖

您可以使用地圖控制項用戶端 Javascript 程式庫，在網頁中內嵌地圖。

1. 建立新的檔案並將它命名為 **MapSearch.html**。

2. 將 Azure 地圖服務樣式表和指令碼來源參考新增新檔案的 `<head>` 元素：

    ```html
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/css/atlas.min.css?api-version=1" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/js/atlas.min.js?api-version=1"></script>
    ```

3. 若要在瀏覽器中轉譯新地圖，請在 `<style>` 元素中新增 **#map** 參考：

    ```html
    <style>
        #map {
            width: 100%;
            height: 100%;
        }
    </style>
    ```

4. 若要初始化地圖控制項，請在 html 主體中定義新的區段並建立指令碼。 在指令碼中使用您自己的 Azure 地圖服務帳戶金鑰。 如果您需要建立帳戶或尋找金鑰，請參閱[如何管理 Azure 地圖服務帳戶與金鑰](how-to-manage-account-keys.md)。

    ```html
    <div id="map">
        <script>
            var MapsAccountKey = "<_your account key_>";
            var map = new atlas.Map("map", {
                "subscription-key": MapsAccountKey,
                center: [-122.33263,47.59093],
                zoom: 12
            });
        </script>
    </div>
    ```

5. 在網頁瀏覽器中開啟此檔案，並檢視轉譯的地圖。

## <a name="next-steps"></a>後續步驟

了解如何使用完整的範例建立地圖：

> [!div class="nextstepaction"]
> [建立地圖](map-create.md)

了解如何設定地圖的樣式：

> [!div class="nextstepaction"]
> [選擇地圖樣式](choose-map-style.md)
