---
title: 教學課程：使用 C# 撰寫適用於翻譯工具文字的 WPF 應用程式
titleSuffix: Azure Cognitive Services
description: 在此教學課程中，您將了解如何透過使用 C# 建置 WPF 應用程式，以使用翻譯工具文字 API 來翻譯文字、取得已當地語系化的支援語言清單等等。
services: cognitive-services
author: noellelacharite
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-text
ms.topic: tutorial
ms.date: 07/20/2018
ms.author: nolachar
ms.openlocfilehash: 97660985b275bbe4384acb3fc92be8aaa0b57881
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46123958"
---
# <a name="tutorial-write-a-wpf-application-for-translator-text-using-c35"></a>教學課程：使用 C# 撰寫適用於翻譯工具文字的 WPF 應用程式

在此教學課程中，您將使用翻譯工具文字 API (V3) (Azure 中的「Microsoft 認知服務」組件) 來建置互動式文字翻譯工具。 您將學習如何：

> [!div class="checklist"]
> * 取得服務所支援語言的清單
> * 執行使用者輸入的文字從一個語言到另一個語言的翻譯

此應用程式的特色還包括與另外兩個「Microsoft 認知服務」整合。

|||
|-|-|
|[文字分析](https://azure.microsoft.com/services/cognitive-services/text-analytics/)|用來視需要自動偵測所要翻譯文字的來源語言|
|[Bing 拼字檢查](https://azure.microsoft.com/services/cognitive-services/spell-check/)|針對英文來源文字，用來校正拼字錯誤，讓翻譯更準確

![[執行中的教學課程程式]](media/translator-text-csharp-session.png)

## <a name="prerequisites"></a>必要條件

您將需要 [Visual Studio 2017](https://www.visualstudio.com/downloads/) 以在 Windows 上執行此程式碼。 (可使用免費的 Community Edition)。

您還需要本程式中所使用三項 Azure 服務的訂用帳戶金鑰。 您可以從 Azure 儀表板取得 Translator Text 服務的金鑰。 您可以使用免費定價層，此定價層可讓您每月免費翻譯最多兩百萬個字元。

「文字分析」和「Bing 拼字檢查」都提供免費試用，您可以在[試用認知服務](https://azure.microsoft.com/try/cognitive-services/)上註冊使用。 您也可以透過 Azure 儀表板為上述兩項服務其中之一建立訂用帳戶。 「文字分析」具有免費層。

下方提供此教學課程的原始程式碼。 您的訂用帳戶金鑰必須複製到 `MainWindow.xaml.cs` 之原始程式碼中作為變數 `TEXT_TRANSLATION_API_SUBSCRIPTION_KEY` 等等。

> [!IMPORTANT]
> 多個區域都有提供「文字分析」服務。 此教學課程原始程式碼中的 URI 位於 `westus` 區域，這是供免費試用使用的區域。 如果您在另一個區域中有訂用帳戶，請依據該區域更新此 URI。

## <a name="source-code"></a>原始程式碼

以下是 Microsoft Translator Text API 的原始程式碼。 若要執行此應用程式，請將此原始程式碼複製到 Visual Studio 中新 WPF 專案的適當檔案中。

### <a name="mainwindowxamlcs"></a>MainWindow.xaml.cs

這是提供應用程式功能的程式碼後置檔案。

```csharp
using System;
using System.Windows;
using System.Net;
using System.Net.Http;
using System.IO;
using System.Collections.Generic;
using System.Linq;
using System.Text;

// NOTE: Add assembly references to System.Runtime.Serialization, System.Web, and System.Web.Extensions.

// NOTE: Install the Newtonsoft.Json NuGet package.
using Newtonsoft.Json;

namespace MSTranslatorTextDemo
{
    /// <summary>
    /// This WPF application demonstrates the use of the Microsoft Translator Text API to translate a brief text string from
    /// one language to another. The languages are selected from a drop-down menu. The text of the translation is displayed.
    /// The source language may optionally be automatically detected. English text is spell-checked.
    /// </summary>
    public partial class MainWindow : Window
    {
        // Translator text subscription key from Microsoft Azure dashboard
        const string TEXT_TRANSLATION_API_SUBSCRIPTION_KEY = "ENTER KEY HERE";
        const string TEXT_ANALYTICS_API_SUBSCRIPTION_KEY = "ENTER KEY HERE";
        const string BING_SPELL_CHECK_API_SUBSCRIPTION_KEY = "ENTER KEY HERE";

        public static readonly string TEXT_TRANSLATION_API_ENDPOINT = "https://api.cognitive.microsofttranslator.com/{0}?api-version=3.0";
        const string TEXT_ANALYTICS_API_ENDPOINT = "https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/";
        const string BING_SPELL_CHECK_API_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/spellcheck/";

        private string[] languageCodes;     // array of language codes

        // Dictionary to map language code from friendly name (sorted case-insensitively on language name)
        private SortedDictionary<string, string> languageCodesAndTitles =
            new SortedDictionary<string, string>(Comparer<string>.Create((a, b) => string.Compare(a, b, true)));

        public MainWindow()
        {
            // at least show an error dialog if there's an unexpected error
            AppDomain.CurrentDomain.UnhandledException += new UnhandledExceptionEventHandler(HandleExceptions);

            if (TEXT_TRANSLATION_API_SUBSCRIPTION_KEY.Length != 32
                || TEXT_ANALYTICS_API_SUBSCRIPTION_KEY.Length != 32
                || BING_SPELL_CHECK_API_SUBSCRIPTION_KEY.Length != 32)
            {
                MessageBox.Show("One or more invalid API subscription keys.\n\n" +
                    "Put your keys in the *_API_SUBSCRIPTION_KEY variables in MainWindow.xaml.cs.",
                    "Invalid Subscription Key(s)", MessageBoxButton.OK, MessageBoxImage.Error);
                System.Windows.Application.Current.Shutdown();
            }
            else
            {
                InitializeComponent();          // start the GUI
                GetLanguagesForTranslate();     // get codes and friendly names of languages that can be translated
                PopulateLanguageMenus();        // fill the drop-down language lists
            }
        }

        // Global exception handler to display error message and exit
        private static void HandleExceptions(object sender, UnhandledExceptionEventArgs args)
        {
            Exception e = (Exception)args.ExceptionObject;
            MessageBox.Show("Caught " + e.Message, "Error", MessageBoxButton.OK, MessageBoxImage.Error);
            System.Windows.Application.Current.Shutdown();
        }

        // ***** POPULATE LANGUAGE MENUS
        private void PopulateLanguageMenus()
        {
            // Add option to automatically detect the source language
            FromLanguageComboBox.Items.Add("Detect");

            int count = languageCodesAndTitles.Count;
            foreach (string menuItem in languageCodesAndTitles.Keys)
            {
                FromLanguageComboBox.Items.Add(menuItem);
                ToLanguageComboBox.Items.Add(menuItem);
            }

            // set default languages
            FromLanguageComboBox.SelectedItem = "Detect";
            ToLanguageComboBox.SelectedItem = "English";
        }

        // ***** DETECT LANGUAGE OF TEXT TO BE TRANSLATED
        private string DetectLanguage(string text)
        {
            string uri = TEXT_ANALYTICS_API_ENDPOINT + "languages?numberOfLanguagesToDetect=1";

            // create request to Text Analytics API
            HttpWebRequest detectLanguageWebRequest = (HttpWebRequest)WebRequest.Create(uri);
            detectLanguageWebRequest.Headers.Add("Ocp-Apim-Subscription-Key", TEXT_ANALYTICS_API_SUBSCRIPTION_KEY);
            detectLanguageWebRequest.Method = "POST";

            // create and send body of request
            var serializer = new System.Web.Script.Serialization.JavaScriptSerializer();
            string jsonText = serializer.Serialize(text);

            string body = "{ \"documents\": [ { \"id\": \"0\", \"text\": " + jsonText + "} ] }";
            byte[] data = Encoding.UTF8.GetBytes(body);
            detectLanguageWebRequest.ContentLength = data.Length;

            using (var requestStream = detectLanguageWebRequest.GetRequestStream())
                requestStream.Write(data, 0, data.Length);

            HttpWebResponse response = (HttpWebResponse)detectLanguageWebRequest.GetResponse();

            // read and parse JSON response
            var responseStream = response.GetResponseStream();
            var jsonString = new StreamReader(responseStream, Encoding.GetEncoding("utf-8")).ReadToEnd();
            dynamic jsonResponse = serializer.DeserializeObject(jsonString);

            // fish out the detected language code
            var languageInfo = jsonResponse["documents"][0]["detectedLanguages"][0];
            if (languageInfo["score"] > (decimal)0.5)
                return languageInfo["iso6391Name"];
            else
                return "";
        }

        // ***** CORRECT SPELLING OF TEXT TO BE TRANSLATED
        private string CorrectSpelling(string text)
        {
            string uri = BING_SPELL_CHECK_API_ENDPOINT + "?mode=spell&mkt=en-US";

            // create request to Bing Spell Check API
            HttpWebRequest spellCheckWebRequest = (HttpWebRequest)WebRequest.Create(uri);
            spellCheckWebRequest.Headers.Add("Ocp-Apim-Subscription-Key", BING_SPELL_CHECK_API_SUBSCRIPTION_KEY);
            spellCheckWebRequest.Method = "POST";
            spellCheckWebRequest.ContentType = "application/x-www-form-urlencoded"; // doesn't work without this

            // create and send body of request
            string body = "text=" + System.Web.HttpUtility.UrlEncode(text);
            byte[] data = Encoding.UTF8.GetBytes(body);
            spellCheckWebRequest.ContentLength = data.Length;
            using (var requestStream = spellCheckWebRequest.GetRequestStream())
                requestStream.Write(data, 0, data.Length);
            HttpWebResponse response = (HttpWebResponse)spellCheckWebRequest.GetResponse();

            // read and parse JSON response and get spelling corrections
            var serializer = new System.Web.Script.Serialization.JavaScriptSerializer();
            var responseStream = response.GetResponseStream();
            var jsonString = new StreamReader(responseStream, Encoding.GetEncoding("utf-8")).ReadToEnd();
            dynamic jsonResponse = serializer.DeserializeObject(jsonString);
            var flaggedTokens = jsonResponse["flaggedTokens"];

            // construct sorted dictionary of corrections in reverse order in string (right to left)
            // so that making a correction can't affect later indexes
            var corrections = new SortedDictionary<int, string[]>(Comparer<int>.Create((a, b) => b.CompareTo(a)));
            for (int i = 0; i < flaggedTokens.Length; i++)
            {
                var correction = flaggedTokens[i];
                var suggestion = correction["suggestions"][0];  // consider only first suggestion
                if (suggestion["score"] > (decimal)0.7)         // take it only if highly confident
                    corrections[(int)correction["offset"]] = new string[]   // dict key   = offset
                        { correction["token"], suggestion["suggestion"] };  // dict value = {error, correction}
            }

            // apply the corrections in order from right to left
            foreach (int i in corrections.Keys)
            {
                var oldtext = corrections[i][0];
                var newtext = corrections[i][1];

                // apply capitalization from original text to correction - all caps or initial caps
                if (text.Substring(i, oldtext.Length).All(char.IsUpper)) newtext = newtext.ToUpper();
                else if (char.IsUpper(text[i])) newtext = newtext[0].ToString().ToUpper() + newtext.Substring(1);

                text = text.Substring(0, i) + newtext + text.Substring(i + oldtext.Length);
            }

            return text;
        }

        // ***** GET TRANSLATABLE LANGUAGE CODES
        private void GetLanguagesForTranslate()
        {
            // send request to get supported language codes
            string uri = String.Format(TEXT_TRANSLATION_API_ENDPOINT, "languages") + "&scope=translation";
            WebRequest WebRequest = WebRequest.Create(uri);
            WebRequest.Headers.Add("Ocp-Apim-Subscription-Key", TEXT_TRANSLATION_API_SUBSCRIPTION_KEY);
            WebRequest.Headers.Add("Accept-Language", "en");
            WebResponse response = null;
            // read and parse the JSON response
            response = WebRequest.GetResponse();
            using (var reader = new StreamReader(response.GetResponseStream(), UnicodeEncoding.UTF8))
            {
                var result = JsonConvert.DeserializeObject<Dictionary<string, Dictionary<string, Dictionary<string, string>>>>(reader.ReadToEnd());
                var languages = result["translation"];

                languageCodes = languages.Keys.ToArray();
                foreach (var kv in languages)
                {
                    languageCodesAndTitles.Add(kv.Value["name"], kv.Key);
                }
            }
        }

        // ***** PERFORM TRANSLATION ON BUTTON CLICK
        private async void TranslateButton_Click(object sender, EventArgs e)
        {
            string textToTranslate = TextToTranslate.Text.Trim();

            string fromLanguage = FromLanguageComboBox.SelectedValue.ToString();
            string fromLanguageCode;

            // auto-detect source language if requested
            if (fromLanguage == "Detect")
            {
                fromLanguageCode = DetectLanguage(textToTranslate);
                if (!languageCodes.Contains(fromLanguageCode))
                {
                    MessageBox.Show("The source language could not be detected automatically " +
                        "or is not supported for translation.", "Language detection failed",
                        MessageBoxButton.OK, MessageBoxImage.Error);
                    return;
                }
            }
            else
                fromLanguageCode = languageCodesAndTitles[fromLanguage];

            string toLanguageCode = languageCodesAndTitles[ToLanguageComboBox.SelectedValue.ToString()];

            // spell-check the source text if the source language is English
            if (fromLanguageCode == "en")
            {
                if (textToTranslate.StartsWith("-"))    // don't spell check in this case
                    textToTranslate = textToTranslate.Substring(1);
                else
                {
                    textToTranslate = CorrectSpelling(textToTranslate);
                    TextToTranslate.Text = textToTranslate;     // put corrected text into input field
                }
            }

            // handle null operations: no text or same source/target languages
            if (textToTranslate == "" || fromLanguageCode == toLanguageCode)
            {
                TranslatedTextLabel.Content = textToTranslate;
                return;
            }

            // send HTTP request to perform the translation
            string endpoint = string.Format(TEXT_TRANSLATION_API_ENDPOINT, "translate");
            string uri = string.Format(endpoint + "&from={0}&to={1}", fromLanguageCode, toLanguageCode);

            System.Object[] body = new System.Object[] { new { Text = textToTranslate } };
            var requestBody = JsonConvert.SerializeObject(body);

            using (var client = new HttpClient())
            using (var request = new HttpRequestMessage())
            {
                request.Method = HttpMethod.Post;
                request.RequestUri = new Uri(uri);
                request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
                request.Headers.Add("Ocp-Apim-Subscription-Key", TEXT_TRANSLATION_API_SUBSCRIPTION_KEY);
                request.Headers.Add("X-ClientTraceId", Guid.NewGuid().ToString());

                var response = await client.SendAsync(request);
                var responseBody = await response.Content.ReadAsStringAsync();

                var result = JsonConvert.DeserializeObject<List<Dictionary<string, List<Dictionary<string, string>>>>>(responseBody);
                var translations = result[0]["translations"];
                var translation = translations[0]["text"];

                // Update the translation field
                TranslatedTextLabel.Content = translation;
            }
        }
    }
}
```

### <a name="mainwindowxaml"></a>MainWindow.xaml

此檔案定義了應用程式的使用者介面 (一個 WPF 表單)。 如果您想要設計自己版本的表單，則不需要此 XAML。

```xml
<Window x:Class="MSTranslatorTextDemo.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:MSTranslatorTextDemo"
        mc:Ignorable="d"
        Title="Microsoft Translator" Height="400" Width="700" BorderThickness="0">
    <Grid>
        <Label x:Name="label" Content="Microsoft Translator" HorizontalAlignment="Left" Margin="39,6,0,0" VerticalAlignment="Top" Height="49" FontSize="26.667"/>
        <TextBox x:Name="TextToTranslate" HorizontalAlignment="Left" Height="23" Margin="42,160,0,0" TextWrapping="Wrap" VerticalAlignment="Top" Width="600" FontSize="14" TabIndex="3"/>
        <Label x:Name="EnterTextLabel" Content="Text to translate:" HorizontalAlignment="Left" Margin="40,129,0,0" VerticalAlignment="Top" FontSize="14"/>
        <Label x:Name="toLabel" Content="Translate to:" HorizontalAlignment="Left" Margin="304,58,0,0" VerticalAlignment="Top" FontSize="14"/>

        <Button x:Name="TranslateButton" Content="Translate" HorizontalAlignment="Left" Margin="39,206,0,0" VerticalAlignment="Top" Width="114" Height="31" Click="TranslateButton_Click" FontSize="14" TabIndex="4" IsDefault="True"/>
        <ComboBox x:Name="ToLanguageComboBox"
                HorizontalAlignment="Left"
                Margin="306,88,0,0"
                VerticalAlignment="Top"
                Width="175" FontSize="14" TabIndex="2">

        </ComboBox>
        <Label x:Name="fromLabel" Content="Translate from:" HorizontalAlignment="Left" Margin="40,58,0,0" VerticalAlignment="Top" FontSize="14"/>
        <ComboBox x:Name="FromLanguageComboBox"
            HorizontalAlignment="Left"
            Margin="42,88,0,0"
            VerticalAlignment="Top"
            Width="175" FontSize="14" TabIndex="1"/>
        <Label x:Name="TranslatedTextLabel" Content="Translation appears here" HorizontalAlignment="Left" Margin="39,255,0,0" VerticalAlignment="Top" Width="620" FontSize="14" Height="85" BorderThickness="0"/>
    </Grid>
</Window>
```

## <a name="service-endpoints"></a>服務端點

Microsoft Translator 服務有眾多端點，可提供各式各樣的翻譯功能。 在此教學課程中使用的是：

|||
|-|-|
|`Languages`|傳回 Translator Text API 其他作業目前支援的語言集合。|
|`Translate`|在提供來源文字、來源語言代碼及目標語言代碼之後，便會傳回來源文字的目標語言翻譯。|

## <a name="the-translation-app"></a>翻譯應用程式

翻譯工具應用程式的使用者介面是使用 Windows Presentation Foundation (WPF) 來建置的。 請依據下列步驟，在 Visual Studio 中建立新的 WPF 專案。

* 從 [檔案] 功能表中，選擇 [新增] > [專案]。
* 在 [新增裝案] 視窗中，開啟 [已安裝] > [範本] > [Visual C#]。 可用的專案範本清單會顯示在對話方塊中央。
* 確定在專案範本清單上方的下拉式功能表中已選擇 [.NET Framework 4.5.2]。
* 按一下專案範本清單中的 [WPF 應用程式 (.NET Framework)]。
* 使用對話方塊底部的欄位，提供新專案及其所屬解決方案的名稱。
* 按一下 [確定] 以建立新專案和解決方案。

![[在 Visual Studio 中建立新的 WPF 應用程式]](media/translator-text-csharp-new-project.png)

將對下列 .NET Framework 組件的參考新增至您的專案。

* System.Runtime.Serialization
* System.Web
* System.Web.Extensions

此外，也請將 `Newtonsoft.Json` NuGet 套件安裝至您的專案。

現在，在 [方案總管] 中找出 `MainWindow.xaml`，然後開啟它。 此檔案一開始會是空的。 以下是已完成之使用者介面應有的外觀，其中以藍色的控制項名稱加註。 請將相同名稱用於您使用者介面中的控制項，因為它們也出現在程式碼中。

![[Visual Studio 設計工具中主視窗的加註檢視]](media/translator-text-csharp-xaml.png)

> [!NOTE]
> 此教學課程的原始程式碼包含此表單的 XAML 來源。 您可以在 Visual Studio 中將它貼到您的專案，而無須建置表單。

* `FromLanguageComboBox` *(下拉式方塊)* - 顯示 Microsoft Translator 支援的文字翻譯語言清單。 使用者會選取其來源翻譯語言。
* `ToLanguageComboBox` *(下拉式方塊)* - 顯示與 `FromComboBox` 相同的語言清單，但會用來選取使用者的目標翻譯語言。
* `TextToTranslate` *(文字方塊)* - 使用者會在這裡輸入要翻譯的文字。
* `TranslateButton` *(按鈕)* - 使用者會按一下此按鈕 (或按 Enter 鍵) 來翻譯文字。
* `TranslatedTextLabel` *(標籤)* - 使用者的文字翻譯會顯示在這裡。

如果您要建立自己的表單版本，則不需要「完全按照」此處使用的方式建立此表單。 但請確保語言下拉式清單的寬度足夠，以避免語言名稱被裁斷。

## <a name="the-mainwindow-class"></a>MainWindow 類別

程式碼後置檔案 `MainWindow.xaml.cs` 是存放讓程式執行其任務之程式碼的位置。 工作會在兩個時間進行：

* 當程式啟動且 `MainWindow` 具現化時，它會使用翻譯工具和 API 來擷取語言清單，然後以那些語言填入下拉式功能表中。 此工作只會在每個工作階段開始執行一次。

* 當使用者按一下 [翻譯] 按鈕時，將會擷取使用者的語言選取項目及其輸入的文字，然後呼叫 `Translate` API 以執行翻譯。 您也可以呼叫其他函式來判斷文字的語言，以及在翻譯之前先校正其拼字。

查看類別的開頭處：

```csharp
public partial class MainWindow : Window
{
    // Translator text subscription key from Microsoft Azure dashboard
    const string TEXT_TRANSLATION_API_SUBSCRIPTION_KEY = "ENTER KEY HERE";
    const string TEXT_ANALYTICS_API_SUBSCRIPTION_KEY = "ENTER KEY HERE";
    const string BING_SPELL_CHECK_API_SUBSCRIPTION_KEY = "ENTER KEY HERE";

    public static readonly string TEXT_TRANSLATION_API_ENDPOINT = "https://api.cognitive.microsofttranslator.com/{0}?api-version=3.0";
    const string TEXT_ANALYTICS_API_ENDPOINT = "https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/";
    const string BING_SPELL_CHECK_API_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/spellcheck/";

    private string[] languageCodes;     // array of language codes

    // Dictionary to map language code from friendly name (sorted case-insensitively on language name)
    private SortedDictionary<string, string> languageCodesAndTitles =
        new SortedDictionary<string, string>(Comparer<string>.Create((a, b) => string.Compare(a, b, true)));

    public MainWindow()
    {
        // at least show an error dialog if there's an unexpected error
        AppDomain.CurrentDomain.UnhandledException += new UnhandledExceptionEventHandler(HandleExceptions);

        if (TEXT_TRANSLATION_API_SUBSCRIPTION_KEY.Length != 32
            || TEXT_ANALYTICS_API_SUBSCRIPTION_KEY.Length != 32
            || BING_SPELL_CHECK_API_SUBSCRIPTION_KEY.Length != 32)
        {
            MessageBox.Show("One or more invalid API subscription keys.\n\n" +
                "Put your keys in the *_API_SUBSCRIPTION_KEY variables in MainWindow.xaml.cs.",
                "Invalid Subscription Key(s)", MessageBoxButton.OK, MessageBoxImage.Error);
            System.Windows.Application.Current.Shutdown();
        }
        else
        {
            InitializeComponent();          // start the GUI
            GetLanguagesForTranslate();     // get codes and friendly names of languages that can be translated
            PopulateLanguageMenus();        // fill the drop-down language lists
        }
    }
// more to come
}
```

這裡宣告的兩個成員變數存放了我們語言的相關資訊：

|||
|-|-|
|`languageCodes`<br>字串陣列|快取語言代碼。 Translator 服務使用短代碼 (例如 `en` 代表英文) 來識別語言。|
|`languageCodesAndTitles`<br>SortedDictionary|將使用者介面中的「易記」名稱往回對應至 API 中所使用的短代碼。 依字母順序排序，而不考慮大小寫。|

應用程式執行的第一個程式碼是 `MainWindow` 建構函式。 首先，請先將 `HandleExceptions` 方法設定為全域錯誤處理常式。 如此一來，在有例外狀況未獲處理時，至少會出現錯誤警示。

接著，請會檢查以確定 API 訂用帳戶金鑰的長度全部都正好 32 個字元。 如果不是，則最可能的原因會是「某人」尚未貼入其 API 金鑰。 在此情況下，將會顯示錯誤訊息並結束。(當然，通過此測試並不代表金鑰有效)。

如果金鑰至少長度正確，`InitializeComponent()` 呼叫就會透過尋找、載入主應用程式視窗的 XAML 描述並將其具現化，讓使用者介面開始運作。

最後，請設定語言下拉式功能表。 此工作需要三個不同的方法呼叫，其詳細說明請見後續幾節。

## <a name="get-supported-languages"></a>取得支援的語言

在撰寫這份文件時，Microsoft Translator 服務總共支援 61 種語言，而且可能不時新增更多語言。 因此，最好不要以硬式編碼方式將支援的語言寫在您的程式中。 取而代之的是，向 Translator 服務詢問其支援的語言。 任何一種支援的語言都可翻譯成任何其他支援的語言。

請呼叫 `Languages` API 來取得支援的語言清單。

`Languages` API 會接受一個選擇性 GET 查詢參數 *scope*。 *scope* 的值可以是下列三個其中之一：`translation`、`transliteration` 及 `dictionary`。 此程式碼使用 `translation` 值。

`Languages` API 也會接受一個選擇性 HTTP 標頭 `Accept-Language`。 此標頭的值會決定要以哪一種語言傳回所支援語言的名稱。 此值應該是格式正確的 BCP 47 語言標記。 此程式碼會使用 `en` 值來取得以英文顯示的語言名稱。

`Languages` API 會傳回一個類似以下的 JSON 回應。

```json
{
  "translation": {
    "af": {
      "name": "Afrikaans",
      "nativeName": "Afrikaans",
      "dir": "ltr"
    },
    "ar": {
      "name": "Arabic",
      "nativeName": "العربية",
      "dir": "rtl"
    },
...
}
```

如此即可擷取語言代碼 (例如 `af`) 和語言名稱 (例如 `Afrikaans`)，此程式碼會使用 NewtonSoft.Json 方法 [JsonConvert.DeserializeObject](https://www.newtonsoft.com/json/help/html/M_Newtonsoft_Json_JsonConvert_DeserializeObject__1.htm)。

有了這個背景知識之後，請建立下列方法來擷取語言代碼及其名稱。

```csharp
private void GetLanguagesForTranslate()
{
    // send request to get supported language codes
    string uri = String.Format(TEXT_TRANSLATION_API_ENDPOINT, "languages") + "&scope=translation";
    WebRequest WebRequest = WebRequest.Create(uri);
    WebRequest.Headers.Add("Ocp-Apim-Subscription-Key", TEXT_TRANSLATION_API_SUBSCRIPTION_KEY);
    WebRequest.Headers.Add("Accept-Language", "en");
    WebResponse response = null;
    // read and parse the JSON response
    response = WebRequest.GetResponse();
    using (var reader = new StreamReader(response.GetResponseStream(), UnicodeEncoding.UTF8))
    {
        var result = JsonConvert.DeserializeObject<Dictionary<string, Dictionary<string, Dictionary<string, string>>>>(reader.ReadToEnd());
        var languages = result["translation"];

        languageCodes = languages.Keys.ToArray();
        foreach (var kv in languages)
        {
            languageCodesAndTitles.Add(kv.Value["name"], kv.Key);
        }
    }
}
```

`GetLanguagesForTranslate()` 會先建立 HTTP 要求。 `scope=translation` 查詢字串參數只會要求支援文字翻譯的語言。 「文字翻譯 API」訂用帳戶金鑰會新增至要求標頭。 系統會新增值為 `en` 的 `Accept-Language` 標頭，讓支援的語言以英文傳回。

完成要求後，JSON 回應會進行剖析並轉換為字典，然後語言代碼會新增至 `languageCodes` 成員變數。 系統會逐一查看包含語言代碼和易記語言名稱的索引鍵/值組，然後新增至 `languageCodesAndTitles` 成員變數。 (表單中的下拉式功能表會顯示易記名稱，但需要以代碼來要求翻譯)。

## <a name="populate-the-language-menus"></a>填入語言功能表

大部分的使用者介面都是以 XAML 定義的，因此除了呼叫 `InitializeComponent()` 之外，不需要費太多功夫即可完成設定。 其他唯一需要做的，就是將易記語言名稱新增至 [翻譯來源] 和 [翻譯目標] 下拉式清單，這會在 `PopulateLanguageMenus()` 中完成。

```csharp
private void PopulateLanguageMenus()
{
    // Add option to automatically detect the source language
    FromLanguageComboBox.Items.Add("Detect");

    int count = languageCodesAndTitles.Count;
    foreach (string menuItem in languageCodesAndTitles.Keys)
    {
        FromLanguageComboBox.Items.Add(menuItem);
        ToLanguageComboBox.Items.Add(menuItem);
    }

    // set default languages
    FromLanguageComboBox.SelectedItem = "Detect";
    ToLanguageComboBox.SelectedItem = "English";
}
```

填入功能表相當簡單，只要逐一查看 `languageCodesAndTitles` 字典並將每個索引鍵 (亦即「易記」名稱) 新增至那兩個功能表即可。 填入功能表之後，請將預設的 [目標] 和 [來源] 語言設定為 [偵測] (用來自動偵測語言) 和 [英文]。

> [!TIP]
> 如果功能表沒有預設選取項目，使用者將無須先選擇 [目標] 或 [來源] 語言，便可按一下 [翻譯]。 如果有預設值，便不需要處理這個問題。

現在，`MainWindow` 已初始化，且使用者介面已建立，程式碼會等待使用者按下 [翻譯] 按鈕。

## <a name="perform-translation"></a>執行翻譯

當使用者按一下 [翻譯] 時，WPF 會叫用以下所示的 `TranslateButton_Click()` 事件處理常式。

```csharp
private async void TranslateButton_Click(object sender, EventArgs e)
{
    string textToTranslate = TextToTranslate.Text.Trim();

    string fromLanguage = FromLanguageComboBox.SelectedValue.ToString();
    string fromLanguageCode;

    // auto-detect source language if requested
    if (fromLanguage == "Detect")
    {
        fromLanguageCode = DetectLanguage(textToTranslate);
        if (!languageCodes.Contains(fromLanguageCode))
        {
            MessageBox.Show("The source language could not be detected automatically " +
                "or is not supported for translation.", "Language detection failed",
                MessageBoxButton.OK, MessageBoxImage.Error);
            return;
        }
    }
    else
        fromLanguageCode = languageCodesAndTitles[fromLanguage];

    string toLanguageCode = languageCodesAndTitles[ToLanguageComboBox.SelectedValue.ToString()];

    // spell-check the source text if the source language is English
    if (fromLanguageCode == "en")
    {
        if (textToTranslate.StartsWith("-"))    // don't spell check in this case
            textToTranslate = textToTranslate.Substring(1);
        else
        {
            textToTranslate = CorrectSpelling(textToTranslate);
            TextToTranslate.Text = textToTranslate;     // put corrected text into input field
        }
    }

    // handle null operations: no text or same source/target languages
    if (textToTranslate == "" || fromLanguageCode == toLanguageCode)
    {
        TranslatedTextLabel.Content = textToTranslate;
        return;
    }

    // send HTTP request to perform the translation
    string endpoint = string.Format(TEXT_TRANSLATION_API_ENDPOINT, "translate");
    string uri = string.Format(endpoint + "&from={0}&to={1}", fromLanguageCode, toLanguageCode);

    System.Object[] body = new System.Object[] { new { Text = textToTranslate } };
    var requestBody = JsonConvert.SerializeObject(body);

    using (var client = new HttpClient())
    using (var request = new HttpRequestMessage())
    {
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(uri);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", TEXT_TRANSLATION_API_SUBSCRIPTION_KEY);
        request.Headers.Add("X-ClientTraceId", Guid.NewGuid().ToString());

        var response = await client.SendAsync(request);
        var responseBody = await response.Content.ReadAsStringAsync();

        var result = JsonConvert.DeserializeObject<List<Dictionary<string, List<Dictionary<string, string>>>>>(responseBody);
        var translations = result[0]["translations"];
        var translation = translations[0]["text"];

        // Update the translation field
        TranslatedTextLabel.Content = translation;
    }
}
```

第一個步驟是從表單中擷取 [目標] 和 [來源] 語言，以及使用者已輸入的文字。

如果來源語言已設定為 [偵測]，請呼叫 `DetectLanguage()` 以判斷文字的語言。 此文字可能使用 Translator API 不支援的語言 (可偵測的語言遠多於可翻譯的語言)，或是「文字分析 API」可能無法偵測它。 在此情況下會顯示訊息來通知使用者，並返回而不進行翻譯。

如果來源語言是英文 (不論是指定的，還是偵測到的)，請使用 `CorrectSpelling()` 來檢查文字拼字，並套用任何校正。 校正後的文字會回填到輸入欄位中，讓使用者知道已進行校正。 (使用者可以在要翻譯的文字前加上連字號來抑制拼字校正)。

如果使用者尚未輸入任何文字，或 [目標] 和 [來源] 語言相同，則無須進行任何翻譯，並且可避免要求。

用來執行翻譯要求的程式碼應該看起來都很類似：建置 URI、建立要求、進行傳送，以及剖析回應。 若要顯示文字，請將傳送至 `TranslatedTextLabel` 控制項。

接著，請在 POST 要求的本文中，以序列化 JSON 陣列將文字傳遞給 `Translate` API。 此 JSON 陣列可以包含多段要翻譯的文字，但在此處只需要一段文字。

名為 `X-ClientTraceId` 的 HTTP 標頭是選擇性的。 其值應該是一個 GUID。 當程序未如預期般運作時，用戶端所提供的追蹤識別碼會是用來追蹤要求的有用資訊。 不過，用戶端必須記錄 X-ClientTraceID 的值，此資訊才會有用。 用戶端追蹤識別碼和要求日期可協助 Microsoft 診斷可能發生的問題。

> [!NOTE]
> 此教學課程將焦點放在 Microsoft 翻譯工具服務，因此並不詳細說明 `DetectLanguage()` 和 `CorrectSpelling()` 方法。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Microsoft Translator Text API 參考](https://docs.microsoft.com/azure/cognitive-services/Translator/reference/v3-0-reference)
