---
title: 使用 Azure Data Factory 從 SAP BW 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 SAP Business Warehouse 複製到支援的接收資料存放區。
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
ms.date: 08/07/2018
ms.author: jingwang
ms.openlocfilehash: 52bbf93d73af281f3959e056a4d5b959e7286cb5
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39590325"
---
# <a name="copy-data-from-sap-business-warehouse-using-azure-data-factory"></a>使用 Azure Data Factory 從 SAP Business Warehouse 複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-sap-business-warehouse-connector.md)
> * [目前的版本](connector-sap-business-warehouse.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 SAP Business Warehouse (BW) 複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 SAP Business Warehouse 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 SAP Business Warehouse 連接器支援：

- SAP Business Warehouse **7.x 版**.
- 使用 MDX 查詢從 **InfoCube 和 QueryCube** (包括 BEx 查詢) 複製資料。
- 使用基本驗證來複製資料。

## <a name="prerequisites"></a>必要條件

若要使用這個 SAP Business Warehouse 連接器，您必須：

- 設定一個「自我裝載 Integration Runtime」。 如需詳細資料，請參閱[自我裝載 Integration Runtime](create-self-hosted-integration-runtime.md) 一文。
- 在 Integration Runtime 電腦上安裝 **SAP NetWeaver 程式庫**。 您可以從 SAP 系統管理員那裡取得 SAP Netweaver 程式庫，或直接從 [SAP 軟體下載中心](https://support.sap.com/swdc)取得。 搜尋 **SAP 附註 #1025361** 以取得最新版本的下載位置。 請確定您選取與 Integration Runtime 安裝項相符的 **64 位元** SAP NetWeaver 程式庫。 然後根據 SAP 附註，安裝 SAP NetWeaver RFC SDK 中包含的所有檔案。 SAP NetWeaver 程式庫也隨附於 SAP 用戶端工具安裝。

>[!TIP]
>若要對 SAP BW 的連線問題進行疑難排解，請確定：
>- 所有從 NetWeaver RFC SDK 解壓縮的所有相依性程式庫都位於 %windir%\system32 資料夾中。 其中通常包含 icudt34.dll、icuin34.dll、icuuc34.dll、libicudecnumber.dll、librfc32.dll、libsapucum.dll、sapcrypto.dll、sapcryto_old.dll、sapnwrfc.dll。
>- 用來連線至 SAP 伺服器所需的連接埠會在自我裝載的 IR 機器上啟用，這通常是連接埠 3300 和 3201。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 SAP Business Warehouse 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 SAP Business Warehouse (BW) 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為：**SapBw** | 是 |
| 伺服器 | SAP BW 執行個體所在之伺服器的名稱。 | 是 |
| systemNumber | SAP BW 系統的系統編號。<br/>允許的值：以字串表示的二位數十進位數字。 | 是 |
| clientId | SAP W 系統中用戶端的用戶端識別碼。<br/>允許的值：以字串表示的三位數十進位數字。 | 是 |
| userName | 能夠存取 SAP 伺服器的使用者名稱。 | 是 |
| password | 使用者的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 如[必要條件](#prerequisites)所述，必須要有一個「自我裝載 Integration Runtime」。 |是 |

**範例：**

```json
{
    "name": "SapBwLinkedService",
    "properties": {
        "type": "SapBw",
        "typeProperties": {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client id>",
            "userName": "<SAP user>",
            "password": {
                "type": "SecureString",
                "value": "<Password for SAP user>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 SAP BW 資料集所支援的屬性清單。

若要從 SAP BW 複製資料，請將資料集的類型屬性設定為 **RelationalTable**。 針對 RelationalTable 類型的 SAP BW 資料集，不支援任何類型特定的屬性。

**範例：**

```json
{
    "name": "SAPBWDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<SAP BW linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 SAP BW 來源所支援的屬性清單。

### <a name="sap-bw-as-source"></a>SAP BW 作為來源

若要從 SAP BW 複製資料，請將複製活動中的來源類型設定為 **RelationalSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**RelationalSource** | 是 |
| query | 指定 MDX 查詢從 SAP BW 執行個體讀取資料。 | 是 |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromSAPBW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP BW input dataset name>",
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
                "type": "RelationalSource",
                "query": "<MDX query for SAP BW>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-sap-bw"></a>SAP BW 的資料類型對應

從 SAP BW 複製資料時，會使用下列從 SAP BW 資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| SAP BW 資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| ACCP | int |
| CHAR | 字串 |
| CLNT | 字串 |
| CURR | 十進位 |
| CUKY | 字串 |
| DEC | 十進位 |
| FLTP | 兩倍 |
| INT1 | Byte |
| INT2 | Int16 |
| INT4 | int |
| LANG | 字串 |
| LCHR | 字串 |
| LRAW | Byte[] |
| PREC | Int16 |
| QUAN | 十進位 |
| RAW | Byte[] |
| RAWSTRING | Byte[] |
| STRING | 字串 |
| 單位 | 字串 |
| DATS | 字串 |
| NUMC | 字串 |
| TIMS | 字串 |


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
