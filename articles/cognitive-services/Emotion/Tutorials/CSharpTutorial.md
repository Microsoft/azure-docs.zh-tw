---
title: 教學課程：辨識影像中的臉部表情 - 表情 API，C#
titlesuffix: Azure Cognitive Services
description: 探索用來辨識影像中臉部表情的基本 Windows 應用程式。
services: cognitive-services
author: anrothMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: emotion-api
ms.topic: tutorial
ms.date: 01/23/2017
ms.author: anroth
ROBOTS: NOINDEX
ms.openlocfilehash: f3a84a68718fba29e2a4b2fae057e68976119c95
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48237019"
---
# <a name="tutorial-recognize-emotions-on-a-face-in-an-image"></a>教學課程：辨識影像中的臉部表情。

> [!IMPORTANT]
> 表情 API 將於 2019 年 2 月 15 日淘汰。 表情辨識功能現已公開推出，是[臉部 API](https://docs.microsoft.com/azure/cognitive-services/face/) 的一部分。 

探索使用表情 API 來辨識影像中臉部表情的基本 Windows 應用程式。 以下範例可讓您提交影像 URL 或儲存在本機的檔案。 您可以將這個開放原始碼的範例作為範本使用，以使用表情 API 和作為 .NET Framework 之一部分的 WPF (Windows Presentation Foundation)，建置適用於 Windows 的應用程式。

## <a name="Prerequisites">先決條件</a>
#### <a name="platform-requirements"></a>平台需求  
下列範例是使用 [Visual Studio 2015 Community Edition](https://www.visualstudio.com/products/visual-studio-community-vs) 針對.NET Framework 開發。  

#### <a name="subscribe-to-emotion-api-and-get-a-subscription-key"></a>訂閱表情 API 並取得訂用帳戶金鑰  
建立範例之前，您必須先訂閱表情 API，它是 Microsoft 認知服務的一部分。 請參閱[訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)。 本教學課程可以使用主要金鑰或次要金鑰。 請務必遵循最佳做法以保護您的 API 金鑰祕密。  

#### <a name="get-the-client-library-and-example"></a>取得用戶端程式庫和範例  
您可以透過 [SDK](https://www.github.com/microsoft/cognitive-emotion-windows) \(英文\) 來下載表情 API 用戶端程式庫。 下載的 zip 檔案必須解壓縮至您選擇的資料夾，大部分的使用者都是選擇 Visual Studio 2015 資料夾。
## <a name="Step1">步驟 1：開啟範例</a>
1.  啟動 Microsoft Visual Studio 2015，然後按一下 [檔案]，選取 [開啟舊檔]，然後選取 [專案/解決方案]。
2.  瀏覽至儲存已下載表情 API 檔案的資料夾。 按一下 [表情]，按一下 [Windows]，然後按一下 [範例 WPF] 資料夾。
3.  按兩下以開啟名為 **EmotionAPI-WPF-Samples.sln**的 Visual Studio 2015 解決方案 (.sln) 檔案。 這將會在 Visual Studio 中開啟該解決方案。

## <a name="Step2">步驟 2：建置範例</a>
1. 在 [方案總管] 中，以滑鼠右鍵按一下 [參考]，然後選取 [管理 NuGet 套件]。

  ![開啟 NuGet 套件管理員](../Images/EmotionNuget.png)

2.  [NuGet 套件管理員] 視窗隨即開啟。 先選取左上角的 [瀏覽]，然後在搜尋方塊中輸入 "Newtonsoft.Json"，選取 [Newtonsoft.Json] 套件，然後按一下 [安裝]。  

  ![瀏覽至 NuGet 套件](../Images/EmotionNugetBrowse.png)  

3.  按 Ctrl+Shift+B，或按一下功能區功能表的 [建置]，然後選取 [建置方案]。

## <a name="Step3">步驟 3：執行範例</a>
1.  建置完畢後，按 **F5** 或按一下功能區功表上的 [啟動] 以執行範例。
2.  找出具有顯示「請在這裡貼上您的訂用帳戶金鑰來開始」之**文字方塊**的表情 API 視窗。 將您訂用帳戶金鑰貼到文字方塊中，如以下螢幕擷取畫面所示。 您可以按一下 [儲存金鑰] 按鈕，選擇將訂用帳戶金鑰保存在電腦或膝上型電腦上。 當您想要從系統刪除訂用帳戶金鑰時，請按一下 [刪除金鑰] 以將它從電腦或膝上型電腦移除。

  ![表情功能介面](../Images/EmotionKey.png)

3.  在 [選取案例] 下方，按一下以使用 [使用資料流來偵測表情] 案例或 [使用 URL 來偵測表情] 案例，然後依照畫面上的指示執行。 Microsoft 會收到您上傳的影像，並可能會使用它們來改善表情 API 及相關服務。 藉由提交影像，即表示您確認自己已遵守我們的[開發人員管理辦法](https://azure.microsoft.com/support/legal/developer-code-of-conduct/)。
4.  有一些範例影像可供搭配這個範例應用程式使用。 您可以在[臉部 API Github 存放庫](https://github.com/Microsoft/Cognitive-Face-Windows/tree/master/Data) \(英文\) 的 **Data** 資料夾底下找到這些影像。 請注意，這些影像的使用是基於「合理使用」的協議進行授權，這表示這些影像可用於測試本範例，但不得用於重新發行的用途上。

## <a name="Review">檢閱和學習</a>
現在您已有一個可執行的應用程式，讓我們檢閱此範例應用程式如何與 Microsoft 認知服務整合。 這可以讓您更輕鬆地使用 Microsoft 表情 API 繼續建置此應用程式，或是開發屬於自己的應用程式。

此範例應用程式會使用表情 API 用戶端程式庫，它是 Microsoft 表情 API 的精簡 C# 用戶端包裝函式。 當您按照上述步驟建置範例應用程式時，會從 NuGet 套件取得該用戶端程式庫。 您可以在 **Emotion**、**Windows**、**Client Library** 底下標題為 [Client Library](https://github.com/Microsoft/Cognitive-Emotion-Windows/tree/master/ClientLibrary) \(英文\) 的資料夾中，檢閱用戶端程式庫原始程式碼，這是上述[先決條件](#Prerequisites)中所提及已下載檔案存放庫的一部分。

您也可以在 [方案總管] 中了解使用用戶端程式庫程式碼的方式：在 [EmotionAPI WPF_Samples] 底下，展開 **DetectEmotionUsingStreamPage.xaml** 來找出 **DetectEmotionUsingStreamPage.xaml.cs** (用來瀏覽儲存在本機的檔案)，或是展開 **DetectEmotionUsingURLPage.xaml** 來尋找 **DetectEmotionUsingURLPage.xaml.cs** (用來上傳影像 URL)。 按兩下 .xaml.cs 檔案，以在 Visual Studio 的新視窗中開啟這些檔案。

為了檢閱表情用戶端程式庫在我們範例應用程式中的使用方式，我們來看看來自 **DetectEmotionUsingStreamPage.xaml.cs** 和 **DetectEmotionUsingURLPage.xaml.cs** 的兩個程式碼片段。 每個檔案都包含註明 “KEY SAMPLE CODE STARTS HERE” 和 “KEY SAMPLE CODE ENDS HERE” 的程式碼註解，以協助您找到以下重新產生的程式碼片段。

表情 API 可接受影像 URL 或二進位影像資料 (具有八位元資料流的形式) 作為輸入。 以下會檢閱這兩個選項。 在這兩種情況下，您需要先找到 using 指示詞，它能讓您使用表情用戶端程式庫。
```csharp

            // -----------------------------------------------------------------------
            // KEY SAMPLE CODE STARTS HERE
            // Use the following namespace for EmotionServiceClient
            // -----------------------------------------------------------------------
            using Microsoft.ProjectOxford.Emotion;
            using Microsoft.ProjectOxford.Emotion.Contract;
            // -----------------------------------------------------------------------
            // KEY SAMPLE CODE ENDS HERE
            // -----------------------------------------------------------------------
```
#### <a name="detectemotionusingurlpagexamlcs"></a>DetectEmotionUsingURLPage.xaml.cs

此程式碼片段示範如何使用用戶端程式庫，將訂用帳戶金鑰和相片 URL 提交給表情 API 服務。

```csharp

            // -----------------------------------------------------------------------
            // KEY SAMPLE CODE STARTS HERE
            // -----------------------------------------------------------------------

            window.Log("EmotionServiceClient is created");

            //
            // Create Project Oxford Emotion API Service client
            //
            EmotionServiceClient emotionServiceClient = new EmotionServiceClient(subscriptionKey);

            window.Log("Calling EmotionServiceClient.RecognizeAsync()...");
            try
            {
                //
                // Detect the emotions in the URL
                //
                Emotion[] emotionResult = await emotionServiceClient.RecognizeAsync(url);
                return emotionResult;
            }
            catch (Exception exception)
            {
                window.Log("Detection failed. Please make sure that you have the right subscription key and proper URL to detect.");
                window.Log(exception.ToString());
                return null;
            }
            // -----------------------------------------------------------------------
            // KEY SAMPLE CODE ENDS HERE
            // -----------------------------------------------------------------------
```
#### <a name="detectemotionusingstreampagexamlcs"></a>DetectEmotionUsingStreamPage.xaml.cs

以下示範如何將訂用帳戶金鑰和儲存在本機的影像提交至表情 API。


```csharp


            // -----------------------------------------------------------------------
            // KEY SAMPLE CODE STARTS HERE
            // -----------------------------------------------------------------------

            //
            // Create Project Oxford Emotion API Service client
            //
            EmotionServiceClient emotionServiceClient = new EmotionServiceClient(subscriptionKey);

            window.Log("Calling EmotionServiceClient.RecognizeAsync()...");
            try
            {
                Emotion[] emotionResult;
                using (Stream imageFileStream = File.OpenRead(imageFilePath))
                {
                    //
                    // Detect the emotions in the URL
                    //
                    emotionResult = await emotionServiceClient.RecognizeAsync(imageFileStream);
                    return emotionResult;
                }
            }
            catch (Exception exception)
            {
                window.Log(exception.ToString());
                return null;
            }
            // -----------------------------------------------------------------------
            // KEY SAMPLE CODE ENDS HERE
            // -----------------------------------------------------------------------
```
<!--
## <a name="Related">Related Topics</a>
[Emotion API Overview](.)
-->
