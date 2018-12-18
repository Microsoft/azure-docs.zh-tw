---
title: 使用 Azure Digital Twins (C#) 尋找空閒會議室 | Microsoft Docs
description: 在本快速入門中，您會執行兩個 .NET Core 應用程式範例，將模擬的動作和二氧化碳遙測資料傳送至 Azure Digital Twins 中的空間。 我們的目標是要在計算雲端中的處理能力後，從管理 API 尋找有新鮮空氣的空閒會議室。
author: alinamstanciu
manager: bertvanhoof
ms.service: digital-twins
services: digital-twins
ms.devlang: csharp
ms.topic: quickstart
ms.custom: mvc
ms.date: 10/02/2018
ms.author: alinast
ms.openlocfilehash: e0b47b1322a520ad8b09fd2fe2967e628b5e4e03
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49322711"
---
# <a name="quickstart-find-available-rooms-using-azure-digital-twins"></a>快速入門：使用 Azure Digital Twins 尋找空閒會議室

Azure Digital Twins 服務可讓您重新建立實體環境的數位影像。 然後，您可以收到環境事件通知，並自訂關於通知的回應。 

本快速入門會使用[一組 .NET 範例](https://github.com/Azure-Samples/digital-twins-samples-csharp)將虛構辦公大樓數位化，並示範如何尋找該大樓內的空閒會議室。 使用 Digital Twins，便可將多個感應器關聯至環境。 除了是否有空閒會議室外，您還可以借助於模擬的二氧化碳感應器，來了解空閒會議室的空氣品質是否理想。 其中一個應用程式範例會產生隨機的感應器資料，以協助您將此案例視覺化。

## <a name="prerequisites"></a>必要條件

1. 如果您沒有 Azure 帳戶，請在開始之前[建立免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

1. 您在此快速入門中執行的兩個主控台應用程式是使用 C# 所撰寫的。 您必須在開發電腦上安裝 [.NET Core SDK 2.1.403 版或更新版本](https://www.microsoft.com/net/download)。 如果您已安裝 .NET Core SDK，便可在命令提示字元中執行 `dotnet --version`，來確認開發電腦上目前的 C# 版本。

1. 下載 [C# 專案範例](https://github.com/Azure-Samples/digital-twins-samples-csharp/archive/master.zip)，並將 digital-twins-samples-csharp-master.zip 封存檔解壓縮。 


## <a name="create-a-digital-twins-instance"></a>建立 Digital Twins 執行個體

使用這一節中的步驟，在[入口網站](https://portal.azure.com)中建立新的 Digital Twins 執行個體。

[!INCLUDE [create-digital-twins-portal](../../includes/create-digital-twins-portal.md)]

## <a name="set-permissions-for-your-app"></a>設定應用程式的權限

本節會向 Azure Active Directory (AAD) 註冊應用程式範例，讓此範例可以存取 Digital Twins 執行個體。 如果您已註冊 AAD 應用程式，則可在此範例中重複使用，並確定其已如本節所述完成設定。 

[!INCLUDE [digital-twins-permissions](../../includes/digital-twins-permissions.md)]


## <a name="build-application"></a>建置應用程式

使用下列步驟建置會議室使用情形應用程式：

1. 開啟命令提示字元，並瀏覽至 digital-twins-samples-csharp-master.zip 檔案解壓縮所在的資料夾。
1. 執行 `cd occupancy-quickstart/src`。
1. 執行 `dotnet restore`。
1. 編輯 appSettings.json 以更新下列變數：
    - ClientId：輸入 AAD 應用程式註冊的 [應用程式識別碼]，其記載於上一節。
    - Tenant：輸入 AAD 租用戶的 [目錄識別碼]，其同樣記載於上一節。
    - BaseUrl：Digital Twins 執行個體的 [管理 API] URL，其所用格式如下：`https://yourDigitalTwinsName.yourLocation.azuresmartspaces.net/management/api/v1.0/`。 將此 URL 中的預留位置替換為執行個體在上一節的值。

## <a name="provision-graph"></a>佈建圖形

此步驟會佈建具有數個空間、一部裝置、兩個感應器、一個自訂函式與一個角色指派的 Digital Twins 空間圖形。 空間圖形的佈建會使用 [*provisionSample.yaml*](https://github.com/Azure-Samples/digital-twins-samples-csharp/blob/master/occupancy-quickstart/src/actions/provisionSample.yaml) 檔案來完成。

1. 執行 `dotnet run ProvisionSample`。
    >[!NOTE]
    >我們使用「裝置登入」Azure CLI 工具來對 Azure AD 驗證使用者。 使用者必須輸入指定代碼，才能使用 [Microsoft 登入](https://microsoft.com/devicelogin)頁面來進行驗證。 輸入代碼後，請遵循相關步驟來進行驗證。 此工具每次執行時都會要求使用者進行驗證。
    
    >[!TIP]
    > 如果在執行此步驟時收到下列錯誤，請進行檢查以確定所複製的變數是否正確。 
    > `EXIT: Unexpected error: The input is not a valid Base-64 string ...`


1. 佈建步驟可能需要進行幾分鐘的時間。 其也會在 Digital Twins 執行個體內佈建 IoT 中樞，並執行迴圈直到 IoT 中樞的狀態為 `Running`。

    ![佈建範例][4]

1. 執行結束時，請複製裝置的 `ConnectionString` 以便用於裝置模擬器範例。 您只需要複製下圖所述的字串：

    ![佈建範例][1]

## <a name="send-sensor-data"></a>傳送感應器資料

您可以使用下列步驟來建置及執行模擬器感應器應用程式：

1. 開啟新的命令提示字元，並瀏覽至您下載到 digital-twins-samples-csharp-master 資料夾中的專案。
1. 執行 `cd device-connectivity`。
1. 執行 `dotnet restore`。
1. 編輯 appsettings.json 以使用上方的 `ConnectionString` 來更新 DeviceConnectionString。
1. 執行 `dotnet run` 來開始傳送感應器資料，您應該會看到它正在傳送給 Digital Twins 服務，如下圖所示：

     ![裝置連線能力][2]

1. 讓此模擬器執行，以便您可以使用下一個步驟的動作來並排檢視結果。 此視窗會顯示傳送到 Digital Twins 的模擬感應器資料，而且下一個步驟會透過即時查詢來尋找有新鮮空氣的空閒會議室。

    >[!TIP]
    > 如果在執行此步驟時收到下列錯誤，請進行檢查以確定所複製的 `DeviceConnectionString` 是否正確。  
    > `EXIT: Unexpected error: The input is not a valid Base-64 string ...`

## <a name="find-available-spaces-with-fresh-air"></a>尋找有新鮮空氣的空閒空間

感應器範例會模擬兩個感應器、動作和二氧化碳的隨機資料值。 在我們的範例中，有新鮮空氣的空閒空間定義是會議室內沒有人，且二氧化碳濃度低於 1000 ppm。 如果情況不符，就表示沒有空閒空間，要不然就是空氣品質不良。

1. 開啟用來執行上述佈建步驟的命令提示字元。
1. 執行 `dotnet run GetAvailableAndFreshSpaces`。
1. 查看這個命令提示字元和並排顯示的感應器資料命令提示字元，如下所示。 
1. 一個命令提示字元會每隔 5 秒將模擬的動作和二氧化碳資料傳送至 Digital Twins。 另一個命令會即時讀取圖形，以根據隨機的模擬資料來找出有新鮮空氣的空閒會議室。 它會根據最後一次傳送的感應器資料，近乎即時地顯示下列其中一個情況：
    - 有新鮮空氣的空閒會議室。
    - 會議室有人使用或空中品質不良。

     ![取得有新鮮空氣的空閒空間][3]

若要了解本快速入門所發生的情況以及已呼叫的 API，請使用 digital-twins-samples-csharp 中找到的程式碼工作區專案 (請參閱下面的命令) 來開啟 [Visual Studio Code](https://code.visualstudio.com/Download)。 教學課程會深入探索此程式碼，並告訴您如何修改組態資料，以及呼叫了哪些 API。 若要深入了解管理 API，請瀏覽 Digital Twins Swagger 頁面 `https://yourDigitalTwinsName.yourLocation.azuresmartspaces.net//management/swagger`，若要方便一點，也可以瀏覽 [Digital Twins Swagger](https://docs.westcentralus.azuresmartspaces.net/management/swagger)。 

```
<path>\occupancy-quickstart\src>code ..\..\digital-twins-samples.code-workspace
```

## <a name="clean-up-resources"></a>清除資源

教學課程會詳述如何建置應用程式讓設施管理員提高設施是否有人使用的辨別能力，以及更有效率地管理大樓。

如果您打算繼續進行教學課程，請勿清除在此快速入門中建立的資源。 如果您不打算繼續，請使用下列步驟來刪除本快速入門建立的所有資源：

1. 刪除下載存放庫範例時所建立的資料夾。
1. 從 [Azure 入口網站](http://portal.azure.com)的左側功能表，按一下 [所有資源]，然後選取 Digital Twins 資源。 在 [所有資源] 窗格的頂端，按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

本快速入門會概述一個簡單案例，讓您了解如何尋找有良好工作條件的會議室。 若要更加深入地分析此案例，請繼續進行本教學課程：

> [!div class="nextstepaction"]
> [教學課程：部署 Azure Digital Twins 及設定空間圖形](tutorial-facilities-setup.md)

<!-- Images -->
[1]: media/quickstart-view-occupancy-dotnet/digital-twins-provision-sample.png
[2]: media/quickstart-view-occupancy-dotnet/digital-twins-device-connectivity.png
[3]: media/quickstart-view-occupancy-dotnet/digital-twins-get-available.png
[4]: media/quickstart-view-occupancy-dotnet/digital-twins-provision-sample1.png
