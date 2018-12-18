---
title: 教學課程：翻譯工具語音 API C#
titleSuffix: Azure Cognitive Services
description: 使用翻譯工具語音 API 來即時翻譯文字。
services: cognitive-services
author: v-jerkin
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-speech
ms.topic: tutorial
ms.date: 3/5/2018
ms.author: v-jerkin
ms.openlocfilehash: 6a53eaf2154162ab9ec85a5a75c2cd52962b53a9
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340421"
---
# <a name="tutorial-translator-speech-application-in-c"></a>教學課程：翻譯工具語音應用程式 (C#)

[!INCLUDE [Deprecation note](../../../includes/cognitive-services-translator-speech-deprecation-note.md)]

此教學課程是一個互動式語音翻譯工具的導覽，此工具使用翻譯工具語音 API (Azure 認知服務」的組件)。 您將了解如何：

> [!div class="checklist"]
> * 要求一份服務所支援語言的清單
> * 擷取音訊並將其傳輸至服務
> * 接收語音翻譯並以文字顯示
> * 依選擇播放口語 (文字轉換語音) 版本的翻譯

此應用程式的 Visual Studio 解決方案檔[可在 GitHub 上找到](https://github.com/MicrosoftTranslator/SpeechTranslator)。

## <a name="prerequisites"></a>必要條件

若要進行本教學課程，您需要有任何一版的 Visual Studio 2017，包括 Community Edition。 

Visual Studio 解決方案也會建置應用程式的安裝程式。 您需要 [WiX Toolset](http://wixtoolset.org/) 和 [WiX Toolset Visual Studio 延伸模組](https://marketplace.visualstudio.com/items?itemName=RobMensching.WixToolsetVisualStudio2017Extension)來支援此功能。

您也需要 Translator Speech 服務的訂用帳戶金鑰，從 Microsoft Azure 儀表板即可取得此金鑰。 您可以使用免費定價層，此定價層可讓您每月免費翻譯最多 10 小時的語音。 這個層已足以應付本教學課程的需求。

此外，還需要協力廠商 [JSON.Net 程式庫](https://www.newtonsoft.com/json) (來自 Newtonsoft)。 如果 Visual Studio 選項中 [套件還原] 的兩個核取方塊都已啟用，NuGet 就會自動安裝此組件。

## <a name="trying-the-translation-app"></a>試用翻譯應用程式

在 Visual Studio 中開啟翻譯工具語音方案 (`SpeechTranslator.sln`) 之後，請按 F5 來建置並啟動應用程式。  程式的主視窗隨即出現。

![[「語音翻譯工具」主視窗]](media/speech-translator-main-window.png)

在第一次執行時，從 [設定] 功能表中選擇 [帳戶設定] 功能表，以開啟這裡顯示的視窗。

![[「語音翻譯工具」主視窗]](media/speech-translator-settings-window.png)

將您的翻譯工具語音訂用帳戶金鑰貼到此視窗中，然後按一下 [儲存]。 您的金鑰會在各個執行之間儲存下來。

請返回主視窗，選擇要使用的音訊輸入和輸出裝置，以及 [從] 和 [到] 語言。 如果您想要聽到翻譯的音訊，請務必勾選 [TTS] (文字轉換語音) 選項。 如果您想要在說話時看到推測性的部分翻譯，請啟用 [部分結果] 選項。

最後，按一下 [啟動] 以開始翻譯。 說出您想要翻譯的話，然後看著已辨識的文字和翻譯出現在視窗中。 如果您已啟用 TTS 選項，就會同時聽到翻譯。

## <a name="obtaining-supported-languages"></a>取得支援的語言

在撰寫這份文件時，翻譯工具語音服務支援超過 60 種文字翻譯語言。 支援較少數的語音翻譯語言。 這類語言需要同時支援文字記錄 (語音辨識) 和合成 (文字轉換語音輸出)。

換句話說，若要進行語音翻譯，來源語言必須支援文字記錄。 如果您想要文字結果，則輸出語言可以是任何一種支援文字翻譯的語言。 如果您想要語音輸出，則只能翻譯成支援文字轉換語音的語言。

Microsoft 可能不時新增新語言支援。 因此，您不應該以硬式編碼方式將任何已知的支援語言寫在您的程式中。 取而代之的是，Translator Speech API 會提供一個 Languages 端點，可讓您在執行階段擷取支援的語言。 您可以選擇接收下列一或多個語言清單： 

| | |
|-|-|
|`speech`|支援語音文字記錄的語言。 可以是語音翻譯的來源語言。|
|`text`|支援文字轉換文字翻譯的語言。 當使用文字輸出功能時，可以是語音翻譯的目標語言。|
|`tts`|支援語音合成的語音，每個語音都與一個特定的語言關聯。 當使用文字轉換語音功能時，可以是語音翻譯的目標語言。 一個指定語言可以受多個語音支援。|

Languages 端點不需要訂用帳戶金鑰，且其使用量不會計入您的配額中。 其 URI 是 `https://dev.microsofttranslator.com/languages`，並且會以 JSON 格式傳回其結果。

以下所示 `MainWindow.xaml.cs` 中的 `UpdateLanguageSettingsAsync()` 會呼叫 Languages 端點以取得支援的語言清單。 

```csharp
private async Task UpdateLanguageSettingsAsync()
{
    Uri baseUri = new Uri("https://" + baseUrl);
    string fullUriString = "/Languages?api-version=1.0&scope=text,speech,tts";
    if (MenuItem_Experimental.IsChecked) fullUriString += "&flight=experimental";            
    Uri fullUri = new Uri(baseUri, fullUriString);

    using (HttpClient client = new HttpClient()) //'client' is the var - using statement ensures the dispose method is used even after an exception.
    {
        HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Get, fullUri);

        // get language names for current UI culture:
        request.Headers.Add("Accept-Language", CultureInfo.CurrentUICulture.TwoLetterISOLanguageName);

        // add a client-side trace Id. In case of issues, one can contact support and provide this:
        //string traceId = "SpeechTranslator" + Guid.NewGuid().ToString();
        //request.Headers.Add("X-ClientTraceId", traceId);
        //Debug.Print("TraceId: {0}", traceId);

        client.Timeout = TimeSpan.FromMilliseconds(10000);
        HttpResponseMessage response = await client.SendAsync(request); //make the async call to the web using the client var and passing the built up URI
        response.EnsureSuccessStatusCode(); //causes exception if the return is false

        Debug.Print("Request Id returned: {0}", GetRequestId(response));

        //create dictionaries to hold the language specific data
        spokenLanguages = new Dictionary<string, string>();
        fromLanguages = new Dictionary<string, string>();
        textLanguages = new Dictionary<string, string>();
        isLTR = new Dictionary<string, bool>();
        voices = new Dictionary<string, List<TTsDetail>>();

        JObject jResponse = JObject.Parse(await response.Content.ReadAsStringAsync()); //get the json from the async call with the response var created above, parse it and put it in a var called jResponse - JObject is a newton class

        //Gather the set of TTS voices
        foreach (JProperty jTts in jResponse["tts"])
        {
            JObject ttsDetails = (JObject)jTts.Value;

            string code = jTts.Name;
            string language = ttsDetails["language"].ToString();
            string displayName = ttsDetails["displayName"].ToString();
            string gender = ttsDetails["gender"].ToString();

            if (!voices.ContainsKey(language)) //check dictionary for a specific key value
            {
                voices.Add(language, new List<TTsDetail>()); //add to the dictionary the locale key and a ttsDetail object
            }

            voices[language].Add(new TTsDetail() { Code = code, DisplayName = string.Format("{0} ({1})", displayName, gender) });
        }

        // Gather the set of speech translation languages
        foreach (JProperty jSpeech in jResponse["speech"])
        {
            JObject languageDetails = (JObject)jSpeech.Value;
            string code = jSpeech.Name;
            string simplecode = languageDetails["language"].ToString();
            string displayName = languageDetails["name"].ToString();
            spokenLanguages.Add(code, displayName);
            fromLanguages.Add(code,simplecode);
        }

        spokenLanguages = spokenLanguages.OrderBy(x => x.Value).ToDictionary(x => x.Key, x => x.Value);
        FromLanguage.Items.Clear();
        foreach (var language in spokenLanguages)
        {
            FromLanguage.Items.Add(new ComboBoxItem() { Content = language.Value, Tag = language.Key});
        }

        // Gather the set of text translation languages
        foreach (JProperty jText in jResponse["text"])
        {
            JObject languageDetails = (JObject)jText.Value;
            string code = jText.Name;
            string displayName = languageDetails["name"].ToString();
            textLanguages.Add(code, displayName);

            string direction = languageDetails["dir"].ToString().ToLowerInvariant();
            bool LTR = true;
            if (direction.ToLowerInvariant() == "rtl") LTR = false;
            isLTR.Add(code, LTR);
        }

        textLanguages = textLanguages.OrderBy(x => x.Value).ToDictionary(x => x.Key, x => x.Value);
        ToLanguage.Items.Clear();
        foreach (var language in textLanguages)
        {
            ToLanguage.Items.Add(new ComboBoxItem() { Content = language.Value, Tag = language.Key });
        }

        if (Properties.Settings.Default.FromLanguageIndex >= 0) FromLanguage.SelectedIndex = Properties.Settings.Default.FromLanguageIndex;
        else
        {
            for(int i=0; i < FromLanguage.Items.Count; ++i)
            {
                ComboBoxItem item = (ComboBoxItem)FromLanguage.Items[i];
                if(CultureInfo.CurrentUICulture.Name.Equals((string)item.Tag, StringComparison.OrdinalIgnoreCase))
                {
                    FromLanguage.SelectedIndex = i;
                }
            }
        }
        if (Properties.Settings.Default.ToLanguageIndex >= 0) ToLanguage.SelectedIndex = Properties.Settings.Default.ToLanguageIndex;
        else
        {
            Random rnd = new Random(DateTime.Now.Millisecond);
            ToLanguage.SelectedIndex = (rnd.Next() % textLanguages.Count);
        }
    }
}
```

此方法會先建構對 Languages 端點的 HTTP 要求，以要求所有三個語言清單 (`text`、`speech` 及 `tts`)。

Languages 端點會使用要求的 `Accept-Languages` 標頭來判斷呈現語言名稱的語言。 例如，講英文的人所知的 "German" 語言，在德文中是稱為 "Deutsch"，在西班牙文中則稱為 "Alemán"，而語言清單便會反映這些差異。 系統的預設語言會用於此標頭。

傳送要求且收到 JSON 回應之後，系統就會將回應剖析成內部資料結構。 這些結構會接著用來建構 [原文語言] 和 [目標語言] 功能表。 

由於可用的語音取決於使用者所選擇的「原文語言」，因此尚無法設定 [語音] 功能表。 取而代之的是，會將每個語言的可用語音儲存以供日後使用。 `ToLanguage_SelectionChanged` 處理常式 (在相同的來源檔案中) 稍後會在使用者選擇某個「原文語言」時，藉由呼叫 `UpdateVoiceComboBox()` 來更新 [語音] 功能表。 

如果使用者之前未執行過此應用程式，系統會隨機選取一個「原文語言」，這只是為了好玩。 (功能表設定會在各個工作階段之間儲存下來)。

## <a name="authenticating-requests"></a>驗證要求

若要向 Microsoft Translator Speech 服務進行驗證，您必須將 Azure 訂用帳戶金鑰當作連線要求標頭中 `Ocp-Apim-Subscription-Key` 的值來傳送。

## <a name="translation-overview"></a>翻譯概觀

「翻譯 API」(WebSockets 端點 `wss://dev.microsofttranslator.com/speech/translate`) 可接受將音訊翻譯成單聲道、16 kHz、16 位元帶正負號 WAVE 格式。 此服務會傳回一或多個包含已辨識和翻譯之文字的 JSON 回應。 如果已提出文字轉換語音要求，則會傳送音訊檔案。

使用者會使用 [麥克風/檔案輸入] 功能表來選擇音訊來源。 音訊可能來自音訊裝置 (例如麥克風) 或來自 `.WAV` 檔案。

當使用者按一下 [開始] 按鈕時，就會叫用 `StartListening_Click` 方法。 此事件處理常式會接著呼叫 `Connect()`，以開始進行將音訊傳送給服務 API 端點的程序。 `Connect()` 方法會執行下列工作：


> [!div class="checklist"]
> * 從主視窗取得使用者設定並驗證這些設定
> * 將音訊輸入和輸出資料流初始化
> * 呼叫 `ConnectAsync()` 以處理剩餘工作

`ConnectAsync()` 會接著處理下列工作：

> [!div class="checklist"]
> * 以 `Ocp-Apim-Subscription-Key` 標頭中的 Azure 訂用帳戶金鑰進行驗證
> * 建立 `SpeechClient` 執行個體 (可在 `SpeechClient.cs` 中找到) 以與服務進行通訊
> * 將 `TextMessageDecoder` 和 `BinaryMessageDecoder` 執行個體 (請參閱 `SpeechResponseDecoder.cs`) 初始化以處理回應
> * 透過 `SpeechClient` 執行個體將音訊傳送給 Translator Speech 服務
> * 接收並處理翻譯的結果

`SpeechClient` 的職責較少：

> [!div class="checklist"]
> * 建立與 Translator Speech 服務的 WebSocket 連線
> * 透過通訊端傳送音訊資料及接收回應

## <a name="a-closer-look"></a>查看細節

您現在應該更清楚了解應用程式各個組件如何一起運作來執行翻譯要求。 讓我們看看一些程式碼，將焦點放在相關的組件。

以下是 `Connect()` 的部分版本，當中顯示如何設定音訊資料流：

```csharp
private void Connect()
{
    if (this.currentState != UiState.ReadyToConnect) return;

    Stopwatch watch = Stopwatch.StartNew();
    UpdateUiState(UiState.Connecting);

    // Omitted: code to validate UI settings

    string tag = ((ComboBoxItem)Mic.SelectedItem).Tag as string;
    string audioFileInputPath = null;
    if (tag == "File")
    {
        audioFileInputPath = this.AudioFileInput.Text;
        foreach (string currFile in audioFileInputPath.Split('|'))
        {
            if (!File.Exists(currFile))
            {
                SetMessage(String.Format($"Invalid audio source: selected file {currFile} does not exist."), "", MessageKind.Error);
                UpdateUiState(UiState.ReadyToConnect);
                return;
            }
        }
    }
    bool shouldSuspendInputAudioDuringTTS = this.CutInputAudioCheckBox.IsChecked.HasValue ? this.CutInputAudioCheckBox.IsChecked.Value : false;

    correlationId = Guid.NewGuid().ToString("D").Split('-')[0].ToUpperInvariant();

    // Setup speech translation client options
    SpeechClientOptions options;

    string voicename = "";
    if (this.Voice.SelectedItem != null)
    {
        voicename = ((ComboBoxItem)this.Voice.SelectedItem).Tag.ToString();
    }
    options = new SpeechTranslateClientOptions()
    {
        TranslateFrom = ((ComboBoxItem)this.FromLanguage.SelectedItem).Tag.ToString(),
        TranslateTo = ((ComboBoxItem)this.ToLanguage.SelectedItem).Tag.ToString(),
        Voice = voicename,
    };
    
    options.Hostname = baseUrl;
    options.AuthHeaderKey = "Authorization";
    options.AuthHeaderValue = ""; // set later in ConnectAsync.
    options.ClientAppId = new Guid("EA66703D-90A8-436B-9BD6-7A2707A2AD99");
    options.CorrelationId = this.correlationId;
    options.Features = GetFeatures().ToString().Replace(" ", "");
    options.Profanity = ((SpeechClient.ProfanityFilter)Enum.Parse(typeof(SpeechClient.ProfanityFilter), GetProfanityLevel(), true)).ToString();
    options.Experimental = MenuItem_Experimental.IsChecked;

    // Setup player and recorder but don't start them yet.
    WaveFormat waveFormat = new WaveFormat(16000, 16, 1);

    // WaveProvider for incoming TTS
    // We use a rather large BufferDuration because we need to be able to hold an entire utterance.
    // TTS audio is received in bursts (faster than real-time).
    textToSpeechBytes = 0;
    playerTextToSpeechWaveProvider = new BufferedWaveProvider(waveFormat);
    playerTextToSpeechWaveProvider.BufferDuration = TimeSpan.FromMinutes(5);

    ISampleProvider sampleProvider = null;
    if (audioFileInputPath != null)
    {
        // Setup mixing of audio from input file and from TTS
        playerAudioInputWaveProvider = new BufferedWaveProvider(waveFormat);
        var srce1 = new Pcm16BitToSampleProvider(playerTextToSpeechWaveProvider);
        var srce2 = new Pcm16BitToSampleProvider(playerAudioInputWaveProvider);
        var mixer = new MixingSampleProvider(srce1.WaveFormat);
        mixer.AddMixerInput(srce1);
        mixer.AddMixerInput(srce2);
        sampleProvider = mixer;
    }
    else
    {
        recorder = new WaveIn();
        recorder.DeviceNumber = (int)((ComboBoxItem)Mic.SelectedItem).Tag;
        recorder.WaveFormat = waveFormat;
        recorder.DataAvailable += OnRecorderDataAvailable;
        sampleProvider = playerTextToSpeechWaveProvider.ToSampleProvider();
    }

    if (!batchMode)
    {
        player = new WaveOut();
        player.DeviceNumber = (int)((ComboBoxItem)Speaker.SelectedItem).Tag;
        player.Init(sampleProvider);
    }

    this.audioBytesSent = 0;

    string logAudioFileName = null;
    if (LogSentAudio.IsChecked|| LogReceivedAudio.IsChecked)
    {
        string logAudioPath = System.IO.Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData), Properties.Settings.Default.OutputDirectory);
        try
        {
            Directory.CreateDirectory(logAudioPath);
        }
        catch
        {
            this.AddItemToLog(string.Format("Could not create folder {0}", logAudioPath));
        }

        if (LogSentAudio.IsChecked)
        {
            logAudioFileName = System.IO.Path.Combine(logAudioPath, string.Format("audiosent_{0}.wav", this.correlationId));
        }

        if (LogReceivedAudio.IsChecked)
        {
            string fmt = System.IO.Path.Combine(logAudioPath, string.Format("audiotts_{0}_{{0}}.wav", this.correlationId));
            this.audioReceived = new BinaryMessageDecoder(fmt);
        }
    }
}
```

`Connect()` 的主要部分涉及建立 `SpeechClientOptions` 執行個體 (請參閱 `SpeechClientOptions.cs`) 來保存翻譯選項。 選項包括連線至服務所需的資訊 (例如驗證金鑰和主機名稱)，以及用於翻譯的功能。 這裡的欄位會對應至 [Translator Speech API](https://docs.microsoft.com/azure/cognitive-services/translator-speech/reference) 所公開的標頭欄位和 HTTP 參數。

`Connect()` 也會建立作為要翻譯之語音來源的音訊輸入裝置 (`sampleProvider` 變數) 並將其初始化。 此裝置會是硬體輸入裝置 (例如麥克風) 或包含 WAVE 音訊資料的檔案。

以下是一個 `ConnectAsync()` 方法，此方法會將 `speechClient` 類別具現化，並掛接匿名函式來處理來自服務的文字和二進位回應。

```csharp
private async Task ConnectAsync(SpeechClientOptions options, bool suspendInputAudioDuringTTS)
{
    await ADMAuthenticate(options);
    
    TextMessageDecoder textDecoder;
    
    s2smtClient = new SpeechClient((SpeechTranslateClientOptions)options, CancellationToken.None);
    
    s2smtClient.OnBinaryData += (c, a) => { AddSamplesToPlay(a, suspendInputAudioDuringTTS); };
    s2smtClient.OnEndOfBinaryData += (c, a) => { AddSamplesToPlay(a, suspendInputAudioDuringTTS); };
    s2smtClient.OnTextData += (c, a) => { textDecoder.AppendData(a); lastReceivedPacketTick = DateTime.Now.Ticks; };
    s2smtClient.OnEndOfTextData += (c, a) =>
    {
        textDecoder.AppendData(a);
        lastReceivedPacketTick = DateTime.Now.Ticks;
        textDecoder
            .Decode()
            .ContinueWith(t =>
            {
                if (t.IsFaulted)
                {
                    Log(t.Exception, "E: Failed to decode incoming text message.");
                }
                else
                {
                    object msg = t.Result;
                    if (msg.GetType() == typeof(FinalResultMessage))
                    {
                        // omitted: code to process final binary result
                    }
                    if (msg.GetType() == typeof(PartialResultMessage))
                    {
                        // omitted: code to process partial binary result
                    }
                }
            });
    };
    s2smtClient.Failed += (c, ex) =>
    {
        Log(ex, "E: SpeechTranslation client reported an error.");
    };
    s2smtClient.Disconnected += (c, ea) =>
    {
        SafeInvoke(() =>
        {
            // We only care to react to server disconnect when our state is Connected. 
            if (currentState == UiState.Connected)
            {
                Log("E: Connection has been lost.");
                Log($"E: Errors (if any): \n{string.Join("\n", s2smtClient.Errors)}");
                Disconnect();
            }
        });
    };
    await s2smtClient.Connect();
}
```

進行驗證之後，此方法會建立 `SpeechClient` 執行個體。 `SpeechClient` 類別 (位於 `SpeechClient.cs` 中) 會在收到二進位和文字資料時，叫用事件處理常式。 當連線失敗或中斷時，則會叫用額外的處理常式。

已啟用 TTS 時，二進位資料是服務所傳送的音訊 (文字轉換語音輸出)。 文字資料是口語文字的部分或完整翻譯。 因此，在具現化之後，此方法會掛接函式來處理這些訊息：音訊 (儲存音訊以供日後播放) 和文字 (在視窗中顯示文字)。

## <a name="next-steps"></a>後續步驟

此程式碼範例是一個富含功能、示範 Translator Speech API 使用方式的應用程式。 因此，有為數眾多的移動組件需要了解。 您已經逐步完成最重要的部分。 針對其餘部分，建議您在 Visual Studio 中設定幾個中斷點，並逐步完成翻譯程序。 當您了解範例應用程式時，便已具備在自己的應用程式中使用 Translator Speech 服務的能力。

> [!div class="nextstepaction"]
> [Microsoft Translator Speech API 參考](https://docs.microsoft.com/azure/cognitive-services/translator-speech/reference)
