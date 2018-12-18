---
title: 如何使用 Azure 地圖服務搜尋服務來搜尋地址 | Microsoft Docs
description: 了解如何使用 Azure 地圖服務搜尋服務來搜尋地址
author: dsk-2015
ms.author: dkshir
ms.date: 09/11/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: d25142cecd62fc34a08939d5f1915ba9b2cee749
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387424"
---
# <a name="find-an-address-using-the-azure-maps-search-service"></a>使用 Azure 地圖服務搜尋服務來尋找地址

地圖服務搜尋服務是一組 RESTful API，專門針對要搜尋地址、位置、景點、公司清單及其他地理資訊的開發人員所設計。 此服務會將經緯度指派給特定地址、交叉街道、地理功能或感景點 (POI)。 搜尋所傳回的緯度與經度值可用來作為其他地圖服務的參數 (例如路線和流量)。

## <a name="prerequisites"></a>必要條件

若要對地圖服務 API 進行呼叫，您需要有地圖服務帳戶和金鑰。 如需建立帳戶和擷取金鑰的相關資訊，請參閱[如何管理 Azure 地圖服務帳戶和金鑰](how-to-manage-account-keys.md)。

本文使用 [Postman 應用程式](https://www.getpostman.com/apps)來建置 REST 呼叫。 您可以使用您偏好的任何 API 開發環境。

## <a name="using-fuzzy-search"></a>使用模糊搜尋

搜尋服務的預設 API 為[模糊搜尋](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy)，這在您不知道使用者針對搜尋查詢所輸入之內容的情況下很有幫助。 該 API 會將 POI 搜尋和地理編碼結合為標準的「單行搜尋」。 例如，該 API 可以處理任何地址或 POI 權杖之組合的輸入。 API 也可與內容相關的位置進行加權 (lat./lon. pair)，完全受到座標和半徑所限制，或可以更廣泛地執行，不受任何地理偏差錨點影響。

大部分的搜尋查詢預設為 `maxFuzzyLevel=1` 以取得效能並減少不尋常的結果。 可以視需要依據要求覆寫此預設值，方法是傳入查詢參數 `maxFuzzyLevel=2` 或 `3`。

### <a name="search-for-an-address-using-fuzzy-search"></a>使用模糊搜尋來搜尋地址

1. 開啟 Postman 應用程式，按一下 [新增] | [新建]，然後選取 [取得要求]。 輸入**模糊搜尋**的要求名稱，選取集合或資料夾來儲存它，然後按一下 [儲存]。

2. 在 [建立器] 索引標籤上，選取 [取得] HTTP 方法，然後輸入您 API 端點的要求 URL。

    ![模糊搜尋 ](./media/how-to-search-for-address/fuzzy_search_url.png)

    | 參數 | 建議的值 |
    |---------------|------------------------------------------------|
    | HTTP method | GET |
    | 要求 URL | [https://atlas.microsoft.com/search/fuzzy/json?](https://atlas.microsoft.com/search/fuzzy/json?) |
    | Authorization | 無授權 |

    URL 路徑中的 **Json** 屬性會判斷回應格式。 您在整篇文章中使用 json，以方便使用並提升可讀性。 您可以在[地圖服務功能性 API 參考](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy)的＜取得搜尋模糊＞定義中找到可用的回應格式。

3. 按一下 [Params]，然後輸入下列金鑰 / 值組來作為要求 URL 中的查詢或路徑參數：

    ![模糊搜尋 ](./media/how-to-search-for-address/fuzzy_search_params.png)

    | Key | 值 |
    |------------------|-------------------------|
    | api-version | 1.0 |
    | subscription-key | \<您的 Azure 地圖服務金鑰\> |
    | query | 披薩 |

4. 按一下 [傳送]，然後檢視回應本文。

    「pizza」的模稜兩可查詢字串傳回了 10 個[景點](https://docs.microsoft.com/rest/api/maps/search/getsearchpoi#searchpoiresponse) (POI) 結果，其類別分別落在「pizza」和「restaurant」。 每個結果都會傳回街道地址、緯度 / 經度值、檢視連接埠和位置的進入點。
  
    此查詢中的結果會有所不同，未繫結至任何特定的參考位置。 您可以使用 **countrySet** 參數來指定只有您的應用程式需要涵蓋範圍的國家/地區，因為預設行為是要搜尋全世界，可能會傳回不必要的結果。

5. 將下列機碼值組加入 **Params** 區段並按一下 [傳送]：

    | Key | 值 |
    |------------------|-------------------------|
    | countrySet | US |
  
    結果現在會依國家/地區程式碼繫結，此查詢會傳回美國境內的披薩餐廳。
  
    若要提供某個位置的結果，您可以查詢景點，並使用針對模糊搜尋服務之呼叫中所傳回的緯度和經度值。 在此情況下，您使用搜尋服務來傳回「西雅圖太空針塔」的位置，並使用 lat. / lon. 值來導向搜尋。
  
6. 在 Params 中，輸入下列金鑰 / 值組，並按一下 [傳送]：

    ![模糊搜尋 ](./media/how-to-search-for-address/fuzzy_search_latlon.png)
  
    | Key | 值 |
    |-----|------------|
    | lat | 47.620525 |
    | lon | -122.349274 |

## <a name="search-for-address-properties-and-coordinates"></a>搜尋地址內容和座標

您可以將完整或部分的街道地址傳遞至搜尋地址 API，並收到包含詳細地址內容 (例如自治區或分部)，以及緯度和經度中的位置值。

1. 在 Postman 中，按一下 [新要求] | [取得要求]，然後將它命名為**地址搜尋**。
2. 在 [建立器] 索引標籤上，選取 [取得] HTTP 方法、輸入您 API 端點的要求 URL，然後選取授權通訊協定 (如果有的話)。

    ![地址搜尋 ](./media/how-to-search-for-address/address_search_url.png)
  
    | 參數 | 建議的值 |
    |---------------|------------------------------------------------|
    | HTTP method | GET |
    | 要求 URL | [https://atlas.microsoft.com/search/address/json?](https://atlas.microsoft.com/search/address/json?) |
    | Authorization | 無授權 |

3. 按一下 [Params]，然後輸入下列金鑰 / 值組來作為要求 URL 中的查詢或路徑參數：
  
    ![地址搜尋 ](./media/how-to-search-for-address/address_search_params.png)
  
    | Key | 值 |
    |------------------|-------------------------|
    | api-version | 1.0 |
    | subscription-key | \<您的 Azure 地圖服務金鑰\> |
    | query | 400 Broad St, Seattle, WA 98109 |
  
4. 按一下 [傳送]，然後檢視回應本文。
  
    在此情況下，您指定了完整的地址查詢，並在回應主體中接收單一結果。
  
5. 在 Params 中，將查詢字串編輯為下列值：
    ```plaintext
        400 Broad, Seattle
    ```

6. 將下列機碼值組加入 **Params** 區段並按一下 [傳送]：

    | Key | 值 |
    |-----|------------|
    | typeahead | true |

    **typeahead** 旗標會指示地址搜尋 API 將查詢視為部分輸入，並傳回預測值的陣列。

## <a name="search-for-a-street-address-using-reverse-address-search"></a>使用反向地址搜尋來搜尋街道地址

1. 在 Postman 中，按一下 [新要求] | [取得要求]，然後將它命名為**反向地址搜尋**。

2. 在 [建立器] 索引標籤上，選取 [取得] HTTP 方法，然後輸入您 API 端點的要求 URL。
  
    ![反向地址搜尋 URL ](./media/how-to-search-for-address/reverse_address_search_url.png)
  
    | 參數 | 建議的值 |
    |---------------|------------------------------------------------|
    | HTTP method | GET |
    | 要求 URL | [https://atlas.microsoft.com/search/address/reverse/json?](https://atlas.microsoft.com/search/address/reverse/json?) |
    | Authorization | 無授權 |
  
3. 按一下 [Params]，然後輸入下列金鑰 / 值組來作為要求 URL 中的查詢或路徑參數：
  
    ![反向位址搜尋參數 ](./media/how-to-search-for-address/reverse_address_search_params.png)
  
    | Key | 值 |
    |------------------|-------------------------|
    | api-version | 1.0 |
    | subscription-key | \<您的 Azure 地圖服務金鑰\> |
    | query | 47.591180,-122.332700 |
  
4. 按一下 [傳送]，然後檢視回應本文。

    回應會包含關於薩菲柯球場的重要地址資訊。
  
5. 將下列機碼值組加入 **Params** 區段並按一下 [傳送]：

    | Key | 值 |
    |-----|------------|
    | number | true |

    如果 [number](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse#search_getsearchaddressreverse_uri_parameters) 查詢參數隨著要求傳送，回應可能會包含街道邊 (左/右)，以及該號碼的位移位置。
  
6. 將下列機碼值組加入 **Params** 區段並按一下 [傳送]：

    | Key | 值 |
    |-----|------------|
    | returnSpeedLimit | true |
  
    當設定 [returnSpeedLimit](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse#search_getsearchaddressreverse_uri_parameters) 查詢參數時，回應會傳回已張貼的速度限制。

7. 將下列機碼值組加入 **Params** 區段並按一下 [傳送]：

    | Key | 值 |
    |-----|------------|
    | returnRoadUse | true |

    當設定 [returnRoadUse](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse#search_getsearchaddressreverse_uri_parameters) 查詢參數時，回應會傳回街道層級的反向地理代碼道路使用陣列。

8. 將下列機碼值組加入 **Params** 區段並按一下 [傳送]：

    | Key | 值 |
    |-----|------------|
    | roadUse | true |

    您可以使用 [roadUse](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse#search_getsearchaddressreverse_uri_parameters) 查詢參數，將反向地理代碼限制為特定類型的道路使用。
  
## <a name="search-for-the-cross-street-using-reverse-address-cross-street-search"></a>使用反向地址交叉街道搜尋來搜尋交叉街道

1. 在 Postman 中，按一下 [新要求] | [取得要求]，然後將它命名為**反向地址交叉街道搜尋**。

2. 在 [建立器] 索引標籤上，選取 [取得] HTTP 方法，然後輸入您 API 端點的要求 URL。
  
    ![反向地址交叉街道搜尋 ](./media/how-to-search-for-address/reverse_address_search_url.png)
  
    | 參數 | 建議的值 |
    |---------------|------------------------------------------------|
    | HTTP method | GET |
    | 要求 URL | [https://atlas.microsoft.com/search/address/reverse/crossstreet/json?](https://atlas.microsoft.com/search/address/reverse/crossstreet/json?) |
    | Authorization | 無授權 |
  
3. 按一下 [Params]，然後輸入下列金鑰 / 值組來作為要求 URL 中的查詢或路徑參數：
  
    | Key | 值 |
    |------------------|-------------------------|
    | api-version | 1.0 |
    | subscription-key | \<您的 Azure 地圖服務金鑰\> |
    | query | 47.591180,-122.332700 |
  
4. 按一下 [傳送]，然後檢視回應本文。

## <a name="next-steps"></a>後續步驟

- 探索 [Azure 地圖服務搜尋服務](https://docs.microsoft.com/rest/api/maps/search) \(英文\) API 文件。
