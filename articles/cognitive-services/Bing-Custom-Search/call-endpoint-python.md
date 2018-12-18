---
title: 快速入門：使用 Python 呼叫端點 - Bing 自訂搜尋
titlesuffix: Azure Cognitive Services
description: 本快速入門說明如何使用 Python 呼叫自訂的 Bing 搜尋端點，以從您的自訂搜尋執行個體要求搜尋結果。
services: cognitive-services
author: brapel
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: quickstart
ms.date: 05/07/2018
ms.author: v-brapel
ms.openlocfilehash: d16b7354b0ef727ba3c670b97105e1bd6f99034e
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48815455"
---
# <a name="quickstart-call-bing-custom-search-endpoint-python"></a>快速入門：呼叫 Bing 自訂搜尋端點 (Python)

此快速入門說明如何使用 Python 呼叫 Bing 自訂搜尋端點，以從您的自訂搜尋執行個體要求搜尋結果。 

## <a name="prerequisites"></a>必要條件

若要完成本快速入門，您需要：

- 已經可以使用的自訂搜尋執行個體。 請參閱[建立您的第一個 Bing 自訂搜尋執行個體](quick-start.md)。
- 已安裝 [Python](https://www.python.org/)。
- 訂用帳戶金鑰。 您可以在啟用[免費試用版](https://azure.microsoft.com/try/cognitive-services/?api=bing-custom-search)時取得訂用帳戶金鑰，或者可以從 Azure 儀表板使用付費訂用帳戶金鑰 (請參閱[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account))。    


## <a name="run-the-code"></a>執行程式碼

若要執行此範例，請遵循下列步驟：

1. 建立程式碼的資料夾。  
  
2. 從管理員命令提示字元或終端機，瀏覽至您剛剛建立的資料夾。  
  
3. 安裝**要求** Python 模組：  
  
    <pre>
    pip install pipenv
    pipenv install requests
    </pre>  
      
4. 在您建立的資料夾中建立名為 BingCustomSearch.py 的檔案，然後將下列程式碼複製到該檔案中。 將 **YOUR-SUBSCRIPTION-KEY** 與 **YOUR-CUSTOM-CONFIG-ID** 取代為您的訂用帳戶金鑰與設定識別碼。  
  
    ``` Python
    import json
    import requests
    
    subscriptionKey = "YOUR-SUBSCRIPTION-KEY"
    customConfigId = "YOUR-CUSTOM-CONFIG-ID"
    searchTerm = "microsoft"
    
    url = 'https://api.cognitive.microsoft.com/bingcustomsearch/v7.0/search?q=' + searchTerm + '&customconfig=' + customConfigId
    r = requests.get(url, headers={'Ocp-Apim-Subscription-Key': subscriptionKey})
    print(r.text)
    ```  
  
7. 使用下列命令來執行程式碼。  
  
    ```
    python BingCustomSearch.py
    ```

## <a name="next-steps"></a>後續步驟
- [設定託管的 UI 體驗](./hosted-ui.md)
- [使用裝飾標記醒目提示文字](./hit-highlighting.md)
- [頁面網頁](./page-webpages.md)
