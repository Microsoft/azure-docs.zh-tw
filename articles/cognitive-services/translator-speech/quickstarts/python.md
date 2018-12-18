---
title: 快速入門：翻譯工具語音 API Python
titlesuffix: Azure Cognitive Services
description: 取得資訊和程式碼範例，以協助您快速開始使用翻譯工具語音 API。
services: cognitive-services
author: v-jaswel
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-speech
ms.topic: quickstart
ms.date: 07/17/2018
ms.author: v-jaswel
ms.openlocfilehash: f6386fbb54ab78def289f5eee9f957bb7a11ff65
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49341207"
---
# <a name="quickstart-translator-speech-api-with-python"></a>快速入門：翻譯工具語音 API (使用 Python)
<a name="HOLTop"></a>

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-translator-speech-deprecation-note.md)]

此文章說明如何使用翻譯工具語音 API 來翻譯 .wav 檔案中所說的話語。

## <a name="prerequisites"></a>必要條件

您將需要有 [Python 3.x](https://www.python.org/downloads/)，才能執行此程式碼。

您將需要安裝適用於 Python 的 [websocket-client 套件](https://pypi.python.org/pypi/websocket-client)。

您將需要將一個名為 "speak.wav" 的 .wav 檔案，放在與您從下方程式碼編譯之可執行檔相同的資料夾中。 這個 .wav 檔案應該採用標準 PCM 16 位元 16 kHz 單聲道格式。 

您必須擁有具備 **Microsoft Translator Speech API** 的[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。 您將需要一個來自 [Azure 儀表板](https://portal.azure.com/#create/Microsoft.CognitiveServices)的付費訂用帳戶金鑰。

## <a name="translate-speech"></a>翻譯語音

以下程式碼會將語音從一種語言翻譯成另一種語言。

1. 在您最愛的 IDE 中建立新的 Python 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```python
# -*- coding: utf-8 -*-

# To install this package, run:
#   pip install websocket-client
import websocket

# **********************************************
# *** Update or verify the following values. ***
# **********************************************

# Replace the subscriptionKey string value with your valid subscription key.
key = 'ENTER KEY HERE'

host = 'wss://dev.microsofttranslator.com'
path = '/speech/translate';
params = '?api-version=1.0&from=en-US&to=it-IT&features=texttospeech&voice=it-IT-Elsa'
uri = host + path + params

input_file = 'speak.wav'
output_file = 'speak2.wav'

output = bytearray ()

def on_open (client):
    print ("Connected.")

# r = read. b = binary.
    with open (input_file, mode='rb') as file:
        data = file.read()

    print ("Sending audio.")
    client.send (data, websocket.ABNF.OPCODE_BINARY)
# Make sure the audio file is followed by silence.
# This lets the service know that the audio input is finished.
    print ("Sending silence.")
    client.send (bytearray (32000), websocket.ABNF.OPCODE_BINARY)

def on_data (client, message, message_type, is_last):
    global output
    if (websocket.ABNF.OPCODE_TEXT == message_type):
        print ("Received text data.")
        print (message)
# For some reason, we receive the data as type websocket.ABNF.OPCODE_CONT.
    elif (websocket.ABNF.OPCODE_BINARY == message_type or websocket.ABNF.OPCODE_CONT == message_type):
        print ("Received binary data.")
        print ("Is last? " + str(is_last))
        output = output + message
        if (True == is_last):
# w = write. b = binary.
            with open (output_file, mode='wb') as file:
                file.write (output)
                print ("Wrote data to output file.")
            client.close ()
    else:
        print ("Received data of type: " + str (message_type))

def on_error (client, error):
    print ("Connection error: " + str (error))

def on_close (client):
    print ("Connection closed.")

client = websocket.WebSocketApp(
    uri,
    header=[
        'Ocp-Apim-Subscription-Key: ' + key
    ],
    on_open=on_open,
    on_data=on_data,
    on_error=on_error,
    on_close=on_close
)

print ("Connecting...")
client.run_forever()
```

**翻譯語音回應**

成功的結果是會建立名為 "speak2.wav" 的檔案。 此檔案包含 "speak.wav" 中所說話語的翻譯。

[回到頁首](#HOLTop)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Translator Speech 教學課程](../tutorial-translator-speech-csharp.md)

## <a name="see-also"></a>另請參閱 

[Translator Speech 概觀](../overview.md)
[API 參考](https://docs.microsoft.com/azure/cognitive-services/translator-speech/reference)
