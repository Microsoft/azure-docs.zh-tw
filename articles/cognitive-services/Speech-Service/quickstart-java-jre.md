---
title: 快速入門：如何以 Java 辨識語音 (Windows 或 Linux)
titleSuffix: Microsoft Cognitive Services
description: 了解如何以 Java 辨識語音 (Windows 或 Linux)
services: cognitive-services
author: fmegen
ms.service: cognitive-services
ms.component: Speech
ms.topic: quickstart
ms.date: 10/12/2018
ms.author: fmegen
ms.openlocfilehash: 1f0c19524ebd59dcf0377cc173cd780d656ba447
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339191"
---
# <a name="quickstart-recognize-speech-in-java-on-windows-or-linux-by-using-the-speech-sdk"></a>快速入門：使用語音 SDK 在 Windows 或 Linux 上以 Java 辨識語音

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

在本文中，您會使用[語音 SDK](speech-sdk.md)，建立 Java 主控台應用程式。 您從電腦的麥克風即時將語音轉譯為文字。 此應用程式是使用語音 SDK Maven 套件，以及 64 位元 Windows 或 Ubuntu Linux 16.04 上的 Eclipse Java IDE (v4.8) 來建置。 它會在 64 位元 Java 8 Runtime Environment (JRE) 上執行。

> [!NOTE]
> 針對語音裝置 SDK 和 Roobo 裝置，請參閱[語音裝置 SDK](speech-devices-sdk.md)。

## <a name="prerequisites"></a>必要條件

您需要語音服務訂用帳戶金鑰，才能完成本快速入門。 您可以免費取得一個金鑰。 如需詳細資訊，請參閱[免費試用語音服務](get-started.md)。


## <a name="create-and-configure-project"></a>建立及設定專案

如果您使用 Ubuntu 16.04，啟動 Eclipse 之前，請執行下列命令，確定已安裝必要的套件。

  ```sh
  sudo apt-get update
  sudo apt-get install build-essential libssl1.0.0 libcurl3 libasound2 wget
  ```

1. 啟動 Eclipse。

1. 在 Eclipse Launcher 中，於 [工作區] 欄位中輸入新工作區目錄的名稱。 然後選取 [啟動]。

   ![Eclipse Launcher 的螢幕擷取畫面](media/sdk/qs-java-jre-01-create-new-eclipse-workspace.png)

1. 不久之後，Eclipse IDE 的主要視窗隨即出現。 如果出現 [歡迎使用] 畫面，請加以關閉。

1. 從 Eclipse 功能表列中，選擇 [檔案] > [新增] > [專案] 以建立新專案。

1. [新增專案]  對話方塊隨即出現。 選取 [Java 專案]，然後選取 [下一步]。

   ![已醒目提示 [Java 專案] 的 [新增專案] 對話方塊螢幕擷取畫面](media/sdk/qs-java-jre-02-select-wizard.png)

1. [新增 Java 專案] 精靈隨即啟動。 在 [專案名稱] 欄位中，輸入 **quickstart**，然後選擇 [JavaSE-1.8] 作為執行環境。 選取 [完成]。

   ![[新增 Java 專案] 精靈的螢幕擷取畫面](media/sdk/qs-java-jre-03-create-java-project.png)

1. 如果出現 [是否要開啟相關的透視圖?] 視窗，請選取 [開啟透視圖]。

1. 在 [套件總管] 中，以滑鼠右鍵按一下 **quickstart** 專案。 從操作功能表中選擇 [設定] > [轉換成 Maven 專案]。

   ![套件總管的螢幕擷取畫面](media/sdk/qs-java-jre-04-convert-to-maven-project.png)

1. [建立新的 POM] 視窗隨即出現。 在 [群組識別碼] 欄位中輸入 **com.microsoft.cognitiveservices.speech.samples**，並且在 [成品識別碼] 欄位中輸入 **quickstart**。 然後選取 [完成]。

   ![建立新的 POM 的螢幕擷取畫面](media/sdk/qs-java-jre-05-configure-maven-pom.png)

1. 開啟 **pom.xml** 檔案並加以編輯。

   * 在檔案結尾的 `</project>` 結尾標記之前，建立 `repositories` 元素並讓其參考語音 SDK 的 Maven 存放庫，如下所示：

     [!code-xml[POM Repositories](~/samples-cognitive-services-speech-sdk/quickstart/java-jre/pom.xml#repositories)]

  * 此外，再新增 `dependencies` 元素，並以 1.0.1 版語音 SDK 作為相依性：

     [!code-xml[POM Dependencies](~/samples-cognitive-services-speech-sdk/quickstart/java-jre/pom.xml#dependencies)]

   * 儲存變更。

## <a name="add-sample-code"></a>新增範例程式碼

1. 若要將新的空白類別新增到您的 Java 專案，請選取 [檔案] > [新增] > [類別]。

1. 在 [新增 Java 類別] 視窗中，於 [套件] 欄位中輸入 **speechsdk.quickstart**，並在 [名稱] 欄位中輸入 **Main**。

   ![[新增 Java 類別] 視窗的螢幕擷取畫面](media/sdk/qs-java-jre-06-create-main-java.png)

1. 使用下列程式碼片段來取代 `Main.java` 中的所有程式碼：

   [!code-java[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/java-jre/src/speechsdk/quickstart/Main.java#code)]

1. 將字串 `YourSubscriptionKey` 取代為您的訂用帳戶金鑰。

1. 以與您的訂用帳戶 (例如，免費試用訂用帳戶的 `westus`) 相關聯的[區域](regions.md)取代 `YourServiceRegion` 字串。

1. 儲存專案的變更。

## <a name="build-and-run-the-app"></a>建置並執行應用程式

按 F11 鍵，或選取 [執行] > [偵錯]。
系統將會辨識接下來 15 秒來自您麥克風的語音輸入，並記錄在主控台視窗中。

![成功辨識後主控台輸出的螢幕擷取畫面](media/sdk/qs-java-jre-07-console-output.png)

[!INCLUDE [Download this sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]
在 `quickstart/java-jre` 資料夾中尋找此範例。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用適用於 Java 的語音 SDK 從語音辨識意圖](how-to-recognize-intents-from-speech-java.md)

## <a name="see-also"></a>另請參閱

- [轉譯語音](how-to-translate-speech-csharp.md)
- [自訂原音模型](how-to-customize-acoustic-models.md)
- [自訂語言模型](how-to-customize-language-model.md)
