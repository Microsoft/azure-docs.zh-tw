---
title: 使用 Azure Data Factory 從 Phoenix 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 Phoenix 複製到支援的接收資料存放區。
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
ms.date: 09/19/2018
ms.author: jingwang
ms.openlocfilehash: 78e432bf526ad270ae8543ad1be40727ed560d4b
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46367894"
---
# <a name="copy-data-from-phoenix-using-azure-data-factory"></a>使用 Azure Data Factory 從 Phoenix 複製資料 

此文章概述如何使用 Azure Data Factory 中的「複製活動」，從 Phoenix 複製資料。 此文章是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 Phoenix 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

Azure Data Factory 提供的內建驅動程式可啟用連線，因此使用此連接器您不需要手動安裝任何驅動程式。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Phoenix 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Phoenix 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | Type 屬性必須設定為：**Phoenix** | 是 |
| host | Phoenix 伺服器的 IP 位址或主機名稱。 (也就是 192.168.222.160)  | 是 |
| port | Phoenix 伺服器用來接聽用戶端連線的 TCP 連接埠。 預設值為 8765。 如果您連線到 Azure HDInsights，請將連接埠指定為 443。 | 否 |
| httpPath | 對應至 Phoenix 伺服器的部分 URL。 (也就是 /gateway/sandbox/phoenix/version)。 如果使用 HDInsights 叢集，請指定 `/hbasephoenix0`。  | 否 |
| authenticationType | 用來連線到 Phoenix 伺服器的驗證機制。 <br/>允許的值為：**Anonymous**、**UsernameAndPassword**、**WindowsAzureHDInsightService** | 是 |
| username | 用來連線到 Phoenix 伺服器的使用者名稱。  | 否 |
| password | 對應到使用者名稱的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 否 |
| enableSsl | 指定是否使用 SSL 來加密與伺服器的連線。 預設值為 False。  | 否 |
| trustedCertPath | .pem 檔案的完整路徑，其中包含在透過 SSL 連線時，用來驗證伺服器的受信任 CA 憑證。 只有在自我裝載 IR 上使用 SSL 時，才能設定這個屬性。 預設值為隨 IR 安裝的 cacerts.pem 檔案。  | 否 |
| useSystemTrustStore | 指定是否使用來自系統信任存放區或來自指定 PEM 檔案的 CA 憑證。 預設值為 False。  | 否 |
| allowHostNameCNMismatch | 指定在透過 SSL 連線時，是否要求 CA 所核發的 SSL 憑證名稱符合伺服器的主機名稱。 預設值為 False。  | 否 |
| allowSelfSignedServerCert | 指定是否允許來自伺服器的自我簽署憑證。 預設值為 False。  | 否 |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用「自我裝載 Integration Runtime」或 Azure Integration Runtime (如果您的資料存放區是可公開存取的)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

>[!NOTE]
>如果叢集不支援 HDInsight 等連續的工作階段，請明確地在 http 路徑設定結尾新增節點索引，例如，指定 `/hbasephoenix0` 而不是 `/hbasephoenix`。

**範例：**

```json
{
    "name": "PhoenixLinkedService",
    "properties": {
        "type": "Phoenix",
        "typeProperties": {
            "host" : "<cluster>.azurehdinsight.net",
            "port" : "443",
            "httpPath" : "/hbasephoenix0",
            "authenticationType" : "WindowsAzureHDInsightService",
            "username" : "<username>",
            "password": {
                 "type": "SecureString",
                 "value": "<password>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 此節提供 Phoenix 資料集所支援的屬性清單。

若要從 Phoenix 複製資料，請將資料集的 type 屬性設定為 **PhoenixObject**。 在此類型的資料集中，沒有任何其他類型特定的屬性。

**範例**

```json
{
    "name": "PhoenixDataset",
    "properties": {
        "type": "PhoenixObject",
        "linkedServiceName": {
            "referenceName": "<Phoenix linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 此節提供 Phoenix 來源所支援的屬性清單。

### <a name="phoenixsource-as-source"></a>將 PhoenixSource 作為來源

若要從 Phoenix 複製資料，請將複製活動中的來源類型設定為 **PhoenixSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的 type 屬性必須設定為：**PhoenixSource** | 是 |
| query | 使用自訂 SQL 查詢來讀取資料。 例如： `"SELECT * FROM MyTable"` 。 | 是 |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromPhoenix",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Phoenix input dataset name>",
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
                "type": "PhoenixSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
