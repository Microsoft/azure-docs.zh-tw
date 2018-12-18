---
title: 使用 Azure Data Factory 從 Web 資料表複製資料 | Microsoft Docs
description: 了解 Azure Data Factory 服務的「Web 資料表連接器」，此連接器可讓您將資料從 Web 資料表複製到 Data Factory 所支援作為接收器的資料存放區。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/28/2018
ms.author: jingwang
ms.openlocfilehash: 995bf4586b88671c65077d965b0588de8de74e5c
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37048929"
---
# <a name="copy-data-from-web-table-by-using-azure-data-factory"></a>使用 Azure Data Factory 從 Web 資料表複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-web-table-connector.md)
> * [目前的版本](connector-web-table.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Web 資料表資料庫複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 Web 資料表資料庫複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 Web 資料表連接器支援**從 HTML 頁面擷取資料表內容**。 若要從 HTTP/s 端點擷取資料，請改用 [HTTP 連接器](connector-http.md)。

## <a name="prerequisites"></a>先決條件

若要使用此 Web 資料表連接器，您需要設定「自我裝載 Integration Runtime」。 如需詳細資料，請參閱[自我裝載 Integration Runtime](create-self-hosted-integration-runtime.md) 一文。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Web 資料表連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Web 資料表已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設為： **Web** |yes |
| url | Web 來源的 URL |yes |
| authenticationType | 允許的值為：**Anonymous** (匿名)。 |yes |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 如[必要條件](#prerequisites)所述，必須要有一個「自我裝載 Integration Runtime」。 |yes |

**範例：**

```json
{
    "name": "WebLinkedService",
    "properties": {
        "type": "Web",
        "typeProperties": {
            "url" : "https://en.wikipedia.org/wiki/",
            "authenticationType": "Anonymous"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 Web 資料表資料集所支援的屬性清單。

若要從 Web 資料表複製資料，請將資料集的類型屬性設定為 **WebTable**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**WebTable** | yes |
| path |包含資料表之資源的相對 URL。 |否。 當路徑未指定時，則只會使用在連結服務定義中指定的 URL。 |
| index |資源中資料表的索引。 如需如何取得 HTML 網頁中資料表索引的步驟，請參閱 [取得 HTML 網頁中資料表的索引](#get-index-of-a-table-in-an-html-page) 一節。 |yes |

**範例：**

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "linkedServiceName": {
            "referenceName": "<Web linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Web 資料表來源所支援的屬性清單。

### <a name="web-table-as-source"></a>Web 資料表作為來源

若要從 Web 資料表複製資料，請將複製活動中的來源類型設定為 **WebSource**，不支援任何其他屬性。

**範例：**

```json
"activities":[
    {
        "name": "CopyFromWebTable",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Web table input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "WebSource"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="get-index-of-a-table-in-an-html-page"></a>取得 HTML 網頁中資料表的索引

若要取得您要在[資料集屬性](#dataset-properties)中設定的資料表索引，可以使用 Excel 2016 等軟體 (如下所示) 做為工具：

1. 啟動 **Excel 2016**，然後切換到 [資料] 索引標籤。
2. 按一下工具列上的 [開新查詢]、指向 [從其他來源]，然後按一下 [從 Web]。

    ![Power Query 功能表](./media/copy-data-from-web-table/PowerQuery-Menu.png)
3. 在 [從 Web] 對話方塊中，輸入您要在連結服務 JSON 中使用的 **URL** (例如：https://en.wikipedia.org/wiki/)，以及您為資料集指定的路徑 (例如：AFI%27s_100_Years...100_Movies)，然後按一下 [確定]。

    ![[從 Web] 對話方塊](./media/copy-data-from-web-table/FromWeb-DialogBox.png)

    此範例使用的 URL：https://en.wikipedia.org/wiki/AFI%27s_100_Years...100_Movies
4. 當您看到 [存取 Web 內容] 對話方塊時，選取右側的 **URL**、**驗證方式**，然後按一下 [連線]。

   ![[存取 Web 內容] 對話方塊](./media/copy-data-from-web-table/AccessWebContentDialog.png)
5. 按一下樹狀檢視中的某個**資料表**項目來查看資料表內容，然後按一下底部的 [編輯] 按鈕。  

   ![[導覽器] 對話方塊](./media/copy-data-from-web-table/Navigator-DialogBox.png)
6. 在 [查詢編輯器] 視窗中，按一下工具列上的 [進階編輯器] 按鈕。

    ![[進階編輯器] 按鈕](./media/copy-data-from-web-table/QueryEditor-AdvancedEditorButton.png)
7. 在 [進階編輯器] 對話方塊中，「Source」旁的數字就是索引。

    ![進階編輯器及索引](./media/copy-data-from-web-table/AdvancedEditor-Index.png)

如果您使用的是 Excel 2013，請使用 [Microsoft Power Query for Excel](https://www.microsoft.com/download/details.aspx?id=39379) 來取得索引。 如需詳細資訊，請參閱 [連線至網頁](https://support.office.com/article/Connect-to-a-web-page-Power-Query-b2725d67-c9e8-43e6-a590-c0a175bd64d8) 一文。 如果您使用 [Microsoft Power BI for Desktop](https://powerbi.microsoft.com/desktop/)，步驟就很類似。


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
