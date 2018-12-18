---
title: 開始使用 Speech Devices SDK
description: 開始使用 Speech Devices SDK 的先決條件和指示。
services: cognitive-services
author: v-jerkin
ms.service: cognitive-services
ms.technology: speech
ms.topic: article
ms.date: 05/18/2018
ms.author: v-jerkin
ms.openlocfilehash: 463a015b7c01dafc5b30de56b95fa0510ffb98e4
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42424364"
---
# <a name="get-started-with-the-speech-devices-sdk"></a>開始使用 Speech Devices SDK

本文說明如何設定開發電腦和語音裝置開發套件，以使用 Speech Devices SDK 開發支援語音功能的裝置。 接著，您將建置範例應用程式並將它部署到裝置。 

範例應用程式的原始程式碼包含在 Speech Devices SDK 中，而且您也可以[在 GitHub 上找到](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK)。

## <a name="prerequisites"></a>必要條件

在使用 Speech Devices SDK 開始開發之前，請收集您需要的資訊和軟體。

* 從 [Roobo](http://ddk.roobo.com/) 取得開發套件。 套件提供線性或圓形麥克風陣列設定；根據您的需求選擇正確的設定。

    |開發套件設定|喇叭位置|
    |-----------------------------|------------|
    |圓形|來自裝置的任何方向|
    |線性|裝置正面|

* 從 Speech Devices SDK 的[下載網站](https://shares.datatransfer.microsoft.com/)取得最新版本的 Speech Devices SDK，包括 Android 範例應用程式。 將 ZIP 檔案解壓縮至本機資料夾 (例如 `C:\SDSDK`)。

* 在您的電腦上安裝 [Android Studio](https://developer.android.com/studio/) 和 [Vysor](http://vysor.io/download/)。

* 取得語音服務[訂用帳戶金鑰](get-started.md)。 您可以取得 30 天免費試用版，或從 Azure 儀表板取得金鑰。

* 如果您希望使用語音服務的意圖辨識，請訂閱[語言理解服務](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/) (LUIS) 並[取得訂用帳戶金鑰](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/azureibizasubscription)。 

    您可以[建立簡單的 LUIS 模型](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/)或使用可以從 Speech Devices SDK [下載網站](https://shares.datatransfer.microsoft.com/)取得的範例 LUIS 模型 `LUIS-example.json`。 透過按一下 [匯入新的應用程式] 並選擇 JSON 檔案，將模型的 JSON 檔案上傳至 [LUIS 入口網站](https://www.luis.ai/home)。

## <a name="set-up-the-development-kit"></a>設定開發套件

1. 啟動使用迷你 USB 纜線連接到電腦或電源轉接器的開發套件。 上方面板底下應該會亮起綠色電源指示燈。

1. 使用第二個迷你 USB 纜線將開發套件連接到電腦。

    ![連接開發套件](media/speech-devices-sdk/qsg-1.png)

1. 適當地擺放您的開發套件。

    |開發套件設定|方向|
    |-----------------------------|------------|
    |圓形|直立，麥克風朝向天花板|
    |線性|側邊，麥克風朝向您 (如下所示)|

    ![線性開發套件方向](media/speech-devices-sdk/qsg-2.png)

1. 安裝憑證和喚醒字 (關鍵字) 表檔案，並設定音效裝置的權限。 在命令視窗中輸入下列命令。

    > [!NOTE]
    > 這些命令會使用 Android Debug Bridge `adb.exe`，它是 Android Studio 安裝的一部分。 您可以在 `C:\Users\[user name]\AppData\Local\Android\Sdk\platform-tools` 中找到此工具。 您可以將此目錄加入至您的路徑，以便更方便地叫用 `adb`。 否則，您必須在叫用 `adb` 的每個命令中指定安裝 `adb.exe` 的完整路徑。

    ```
    adb push C:\SDSDK\Android-Sample-Release\scripts\roobo_setup.sh /data/ 
    adb shell
    cd /data/ 
    chmod 777 roobo_setup.sh
    ./roobo_setup.sh
    exit
    ```

    > [!TIP]
    > 將電腦的麥克風和喇叭靜音。 這樣，您便能夠確定您正在使用開發套件的麥克風，而且不會不小心使用電腦的音訊觸發裝置。
    
1.  在您的電腦上啟動 Vysor。

    ![Vysor](media/speech-devices-sdk/qsg-3.png)

1.  您的裝置應該會列在 [選擇裝置] 下。 按一下 [檢視] 旁邊的按鈕。 
 
1.  依序按一下資料夾圖示、[設定]、[WLAN]，即可連線到您的無線網路。

    ![Vysor WLAN](media/speech-devices-sdk/qsg-4.png)
 
 > [!NOTE]
 > 如果貴公司有關於將裝置連線到 wifi 系統的原則，您就需要取得 Mac 位址並連絡 IT 部門，了解如何連線到 wifi 系統。 若要尋找開發套件的 Mac 位址，請按一下開發套件桌面上的檔案資料夾圖示，然後按一下 [設定]、搜尋「Mac 位址」，按一下 [Mac 位址] 以進入 [進階 WLAN]，記下底端找到的 Mac 位址。 此外，有些公司可能有裝置可連線到 wifi 系統的時間長度限制。 您可能需要在特定天數之後，將開發套件向 wifi 系統的註冊延長。  
 
 
   ![Vysor 檔案資料夾](media/speech-devices-sdk/qsg-10.png)
   
   ![VM Mac 位址](media/speech-devices-sdk/qsg-11.png)
   
   
 > 如果您想要將喇叭連結到開發套件，您可以將它連線到音訊線路輸出。您也應該選擇高品質的 3.5mm 喇叭。
 
   ![Vysor 音訊](media/speech-devices-sdk/qsg-14.png)
 
## <a name="run-a-sample-application"></a>執行範例應用程式

若要執行 Roobo 測試並驗證開發套件設定，請建置並安裝範例應用程式。

1.  啟動 Android Studio。

1.  選擇以開啟現有 Android Studio 專案。

    ![Android Studio - 開啟現有專案](media/speech-devices-sdk/qsg-5.png)
 
1.  瀏覽至 `C:\SDSDK\Android-Sample-Release\example`，然後按一下 [確定] 開啟範例專案。
 
1.  將您的語音訂用帳戶金鑰加入至原始程式碼中。 如果您想試用意圖辨識，也請加入您的[語言理解服務](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/)訂用帳戶金鑰和應用程式識別碼。 

    您的金鑰和應用程式資訊放在來源檔案 `MainActivity.java` 中的下列幾行。

    ```java
    // Subscription
    private static final String SpeechSubscriptionKey = "[your speech key]";
    private static final String SpeechRegion = "westus";
    private static final String LuisSubscriptionKey = "[your LUIS key]";
    private static final String LuisRegion = "westus2.api.cognitive.microsoft.com";
    private static final String LuisAppId = "[your LUIS app id]"
    ```

1. 預設喚醒字 (關鍵字) 是「電腦」。  如果需要，您可以嘗試其他提供的喚醒字，例如「機器」和「小幫手」。 可以在「關鍵字」資料夾中的 Speech Devices SDK 中找到這些替代字的資源檔。 例如，`C:\SDSDK\Android-Sample-Release\keyword\Computer` 包含用於「電腦」的檔案。

    您也可以[建立自訂的喚醒字](speech-devices-sdk-create-kws.md)。

    安裝所需的喚醒字：
 
    * 透過在命令視窗中執行下列命令，在裝置上的 \data\ 資料夾中建立一個 `keyword` 資料夾。

        ```
        adb shell
        cd /data
        mkdir keyword
        exit
        ```

    * 將檔案 `kws.table`、`kws_g.fst`、`kws_k.fst` 和 `words_kw.txt`) 複製到裝置的 \data\keyword\ 資料夾。 在命令視窗中執行下列命令。 如果您已建立[自訂喚醒字](speech-devices-sdk-create-kws.md)，從網路產生的 kws.table 檔案將位於與 `kws.table`， `kws_g.fst``kws_k.fst` 和 `words_kw.txt` 檔案相同的目錄中。 請使用 adb 推送 C:\SDSDK\Android-Sample-Release\keyword\[wake_word_name]\kws.table /data/keyword 命令，將 kws.table 檔案改為推送至開發套件。

        ```
        adb push C:\SDSDK\Android-Sample-Release\keyword\kws.table /data/keyword
        adb push C:\SDSDK\Android-Sample-Release\keyword\Computer\kws_g.fst /data/keyword
        adb push C:\SDSDK\Android-Sample-Release\keyword\Computer\kws_k.fst /data/keyword
        adb push C:\SDSDK\Android-Sample-Release\keyword\Computer\words_kw.txt /data/keyword
        ```
    
    * 在範例應用程式中參考這些檔案。 在 `MainActivity.java` 中尋找下列幾行。 確定指定的關鍵字是您正在使用的關鍵字，並且該路徑指向您推送至裝置的 `kws.table` 檔案。
        
        ```java
        private static final String Keyword = "Computer";
        private static final String KeywordModel = "/data/keyword/kws.table";
        ```

        > [!NOTE]
        > 在您自己的程式碼中，可以使用 `kws.table` 檔案建立關鍵字模型執行個體並啟動辨識，如下所示。
        >
        > ```java
        > KeywordRecognitionModel km = KeywordRecognitionModel.fromFile(KeywordModel);
        > final Task<?> task = reco.startKeywordRecognitionAsync(km);
        > ```

1.  更新包含麥克風陣列幾何設定的下列各行。

    ```java
    private static final String DeviceGeometry = "Circular6+1";
    private static final String SelectedGeometry = "Circular6+1";
    ```

    |變數|意義|可用值|
    |--------|-------|----------------|
    |`DeviceGeometry`|實體麥克風設定|`Circular6+1` 適用於圓形開發套件|
    ||| `Linear4` 適用於線性開發套件|
    |`SelectedGeometry`|軟體麥克風設定|`Circular6+1` 適用於使用所有麥克風的圓形開發套件|
    |||`Circular3+1` 適用於使用四個麥克風的圓形開發套件|
    |||`Linear4` 適用於使用所有麥克風的線性開發套件|
    |||`Linear2` 適用於使用兩個麥克風的線性開發套件|


1.  透過從 [執行] 功能表中選擇 [執行 'app'] 來建置應用程式。 [選取部署目標] 對話方塊隨即出現。 選擇您的裝置，然後按一下 [確定] 將應用程式部署到裝置。

    ![選取部署目標](media/speech-devices-sdk/qsg-7.png)
 
1.  Speech Devices SDK 範例應用程式隨即啟動，並顯示如下所示的選項。

    ![範例語音裝置應用程式](media/speech-devices-sdk/qsg-8.png)

1. 請試玩看看！

## <a name="troubleshooting"></a>疑難排解

如果使用語音服務時發生憑證失敗，請確定裝置有正確的日期和時間。 移至 [設定]，按一下 [系統] 下方的 [日期和時間]，然後**選取時區**作為您目前的時區。 讓 [自動日期和時間] 保持 [開啟] 狀態。 當您看到開發套件的時間符合您的 PC 時間時，您將得知開發套件會連線到網際網路。 

 ![Vysor 檔案資料夾](media/speech-devices-sdk/qsg-12.png)
 
 ![Vysor 檔案資料夾](media/speech-devices-sdk/qsg-13.png)

如需更多開發資訊，請參閱 Roobo 的[開發指南](http://dwn.roo.bo/server_upload/ddk/ROOBO%20Dev%20Kit-User%20Guide.pdf)。

Roobo 提供了一種工具，可將所有的音訊擷取到快閃記憶體中，這有助於針對音訊問題進行疑難排解。 針對每個開發套件設定提供了工具版本。 在 [Roobo 網站](http://ddk.roobo.com/)選擇您的裝置，然後按一下頁面底部的 [ROOBO Tools] \(ROOBO 工具\) 連結。
