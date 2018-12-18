---
title: 快速入門：翻譯工具語音 API Node.js
titlesuffix: Azure Cognitive Services
description: 取得資訊和程式碼範例，以協助您快速開始使用翻譯工具語音 API。
services: cognitive-services
author: v-jaswel
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-speech
ms.topic: quickstart
ms.date: 3/5/2018
ms.author: v-jaswel
ms.openlocfilehash: 3db4e242ad630949533823c22a16587daa4f9786
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389695"
---
# <a name="quickstart-translator-speech-api-with-nodejs"></a>快速入門：翻譯工具語音 API (使用 Node.js) 
<a name="HOLTop"></a>

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-translator-speech-deprecation-note.md)]

此文章說明如何使用翻譯工具語音 API 來翻譯 .wav 檔案中所說的話語。

## <a name="prerequisites"></a>必要條件

您需要有 [Node.js 6](https://nodejs.org/en/download/)，才能執行此程式碼。

您需要安裝適用於 Node.js 的 [Websocket 套件](https://www.npmjs.com/package/websocket)。

您需要將一個名為 "speak.wav" 的 .wav 檔案，放在與您從下方程式碼編譯之可執行檔相同的資料夾中。 這個 .wav 檔案應該採用標準 PCM 16 位元 16 kHz 單聲道格式。 您可以從[文字轉換語音 API](https://docs.microsoft.com/azure/cognitive-services/speech-service/rest-apis#text-to-speech) 取得此類 .wav 檔案。

您必須擁有具備 **Microsoft Translator Speech API** 的[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。 您需要一個來自 [Azure 儀表板](https://portal.azure.com/#create/Microsoft.CognitiveServices)的付費訂用帳戶金鑰。

## <a name="translate-speech"></a>翻譯語音

以下程式碼會將語音從一種語言翻譯成另一種語言。

1. 在您最愛的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
/* To install this dependency, run:
npm install websocket
*/
var wsClient = require('websocket').client;
var fs = require('fs');
/* To install this dependency, run:
npm install stream-buffers
*/
var streamBuffers = require('stream-buffers');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace the subscriptionKey string value with your valid subscription key.
let key = 'ENTER KEY HERE';

let host = 'wss://dev.microsofttranslator.com';
let path = '/speech/translate';
let params = '?api-version=1.0&from=en-US&to=it-IT&features=texttospeech&voice=it-IT-Elsa';
let uri = host + path + params;

/* The input .wav file is in PCM 16bit, 16kHz, mono format.
You can obtain such a .wav file using the Text to Speech API. See:
https://docs.microsoft.com/azure/cognitive-services/speech-service/rest-apis#text-to-speech
*/
let input_path = 'speak.wav';

let output_path = 'speak2.wav';

function receive(message) {
    console.log ("Received message of type: " + message.type + ".");

    if (message.type == 'utf8') {
        var result = JSON.parse(message.utf8Data)
        console.log("Response type: " + result.type + ".");
        console.log("Recognized input as: " + result.recognition);
        console.log("Translation: " + result.translation);
    }
    else if (message.type == 'binary') {
/* This is needed to make sure message.binaryData is defined. See:
https://stackoverflow.com/questions/17546953/cant-access-object-property-even-though-it-exists-returns-undefined
*/
        let binary_data = JSON.parse (JSON.stringify (message.binaryData));

        if (binary_data.type == 'Buffer') {
/* Put the binary data in a Buffer - do not write it directly to the file. */
            let buffer = new Buffer(binary_data.data, 'binary');
/* Write the contents of the Buffer to the file. */
            fs.writeFile (output_path, buffer, 'binary', function (error) {
/* fs.writeFile calls the error-handling function even if there is no error, in which case error === null. See:
https://stackoverflow.com/questions/44473868/node-js-fs-writefile-err-returns-null
*/
                if (error !== null) {
                    console.log ("Error: " + error);
                }
            });
        }
    }
}

function send(connection, filename) {
    
    var myReadableStreamBuffer = new streamBuffers.ReadableStreamBuffer({
        frequency: 100,
        chunkSize: 32000
    });

/* Make sure the audio file is followed by silence.
This lets the service know that the audio file is finished.
At 32 bytes per millisecond, this is 100 seconds of silence. */
    myReadableStreamBuffer.put(fs.readFileSync(filename));
    myReadableStreamBuffer.put(new Buffer(3200000));
    myReadableStreamBuffer.stop();

    myReadableStreamBuffer.on('data', function (data) {
        connection.sendBytes(data);
    });

    myReadableStreamBuffer.on('end', function () {
        console.log('Closing connection.');
        connection.close(1000);
    });
}

function connect() {
    var ws = new wsClient();

    ws.on('connectFailed', function (error) {
        console.log('Connection error: ' + error.toString());
    });
                            
    ws.on('connect', function (connection) {
        console.log('Connected.');

        connection.on('message', receive);
        
        connection.on('close', function (reasonCode, description) {
            console.log('Connection closed: ' + reasonCode);
        });

        connection.on('error', function (error) {
            console.log('Connection error: ' + error.toString());
        });
        
        send(connection, input_path);
    });

    ws.connect(uri, null, null, { 'Ocp-Apim-Subscription-Key' : key });
}

connect();
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
