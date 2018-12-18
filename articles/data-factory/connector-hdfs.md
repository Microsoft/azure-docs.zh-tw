---
title: 使用 Azure Data Factory 從 HDFS 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，從雲端或內部部署 HDFS 來源將資料複製到支援的接收資料存放區。
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
ms.date: 04/27/2018
ms.author: jingwang
ms.openlocfilehash: 034c9a321f402bada87290f6aa72fc7e416ef2c6
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37054539"
---
# <a name="copy-data-from-hdfs-using-azure-data-factory"></a>使用 Azure Data Factory 從 HDFS 複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-hdfs-connector.md)
> * [目前的版本](connector-hdfs.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 HDFS 複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 HDFS 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，此 HDFS 連接器支援：

- 使用 **Windows** (Kerberos) 或**匿名**驗證複製檔案。
- 使用 **webhdfs** 通訊協定或**內建 DistCp** 支援複製檔案。
- 依原樣複製檔案，或使用[支援的檔案格式和壓縮轉碼器](supported-file-formats-and-compression-codecs.md)來剖析/產生檔案。

## <a name="prerequisites"></a>先決條件

若要從不可公開存取的 HDFS 複製資料，您需要設定「自我裝載 Integration Runtime」。 如需詳細資料，請參閱[自我裝載 Integration Runtime](concepts-integration-runtime.md) 一文。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 HDFS 專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 HDFS 連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設為：**Hdfs**。 | yes |
| url |到 HDFS 的 URL |yes |
| authenticationType | 允許的值為：**匿名**或 **Windows**。 <br><br> 若要對 HDFS 連接器使用 **Kerberos 驗證**，請參閱[此章節](#use-kerberos-authentication-for-hdfs-connector)來據以設定您的內部部署環境。 |yes |
| userName |Windows 驗證的使用者名稱。 Kerberos 驗證請指定 `<username>@<domain>.com`。 |是 (適用於 Windows 驗證) |
| password |Windows 驗證的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |是 (適用於 Windows 驗證) |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用「自我裝載 Integration Runtime」或 Azure Integration Runtime (如果您的資料存放區是可公開存取的)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

**範例：使用匿名驗證**

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "authenticationType": "Anonymous",
            "userName": "hadoop"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**範例：使用 Windows 驗證**

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "authenticationType": "Windows",
            "userName": "<username>@<domain>.com (for Kerberos auth)",
            "password": {
                "type": "SecureString",
                "value": "<password>"
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

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 HDFS 資料集所支援的屬性清單。

若要從 HDFS 複製資料，請將資料集的類型屬性設定為 **FileShare**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**FileShare** |yes |
| folderPath | 資料夾的路徑。 不支援萬用字元篩選。 例如：資料夾/子資料夾/ |yes |
| fileName |  在指定 "folderPath" 之下檔案的**名稱或萬用字元篩選**。 若未指定此屬性的值，資料集就會指向資料夾中的所有檔案。 <br/><br/>針對篩選，允許的萬用字元為：`*` (符合零或多個字元) 和 `?` (符合零或單一字元)。<br/>- 範例 1：`"fileName": "*.csv"`<br/>- 範例 2：`"fileName": "???20180427.txt"`<br/>如果實際檔案名稱內有萬用字元或逸出字元 `^`，請使用此逸出字元來逸出。 |否 |
| format | 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。<br/><br/>如果您想要以特定格式來剖析檔案，以下是支援的檔案格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](supported-file-formats-and-compression-codecs.md#text-format)、[Json 格式](supported-file-formats-and-compression-codecs.md#json-format)、[Avro 格式](supported-file-formats-and-compression-codecs.md#avro-format)、[Orc 格式](supported-file-formats-and-compression-codecs.md#orc-format)和 [Parquet 格式](supported-file-formats-and-compression-codecs.md#parquet-format)章節。 |否 (僅適用於二進位複製案例) |
| compression | 指定此資料的壓縮類型和層級。 如需詳細資訊，請參閱[支援的檔案格式和壓縮轉碼器](supported-file-formats-and-compression-codecs.md#compression-support)。<br/>支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。<br/>支援的層級為：**Optimal** 和 **Fastest**。 |否 |

>[!TIP]
>若要複製資料夾下的所有檔案，請只指定 **folderPath**。<br>若要複製指定名稱的單一檔案，請以資料夾部分指定 **folderPath**，並以檔案名稱指定 **fileName**。<br>若要複製資料夾下的檔案子集，請以資料夾部分指定 **folderPath**，並以萬用字元篩選指定 **fileName**。

**範例：**

```json
{
    "name": "HDFSDataset",
    "properties": {
        "type": "FileShare",
        "linkedServiceName":{
            "referenceName": "<HDFS linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "folder/subfolder/",
            "fileName": "myfile.csv.gz",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 HDFS 來源所支援的屬性清單。

### <a name="hdfs-as-source"></a>HDFS 作為來源

若要從 HDFS 複製資料，請將複製活動中的來源類型設定為 **HdfsSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**HdfsSource** |yes |
| 遞迴 | 表示是否從子資料夾，或只有從指定的資料夾，以遞迴方式讀取資料。 請注意，當 recursive 設定為 true，而接收器為檔案型存放區時，系統不會在接收器複製/建立空資料夾/子資料夾。<br/>允許的值為：**true** (預設值)、**false** | 否 |
| distcpSettings | 使用 HDFS DistCp 時的屬性群組。 | 否 |
| resourceManagerEndpoint | Yarn ResourceManager 端點 | 若使用 DistCp 則為「是」 |
| tempScriptPath | 資料夾路徑，用來儲存暫存 DistCp 命令指令碼。 指令碼檔案是由 Data Factory 產生，並會在複製作業完成後移除。 | 若使用 DistCp 則為「是」 |
| distcpOptions | 對於 DistCp 命令提供的其他選項。 | 否 |

**範例：使用 UNLOAD 的複製活動之中的 HDFS 來源**

```json
"source": {
    "type": "HdfsSource",
    "distcpSettings": {
        "resourceManagerEndpoint": "resourcemanagerendpoint:8088",
        "tempScriptPath": "/usr/hadoop/tempscript",
        "distcpOptions": "-m 100"
    }
}
```

參閱下一節，深入了解如何使用 DistCp 從 HDFS 有效率地複製資料。

## <a name="use-distcp-to-copy-data-from-hdfs"></a>使用 DistCp 從 HDFS 複製資料

[DistCp](https://hadoop.apache.org/docs/current3/hadoop-distcp/DistCp.html) 是 Hadoop 原生命令列工具，可用來執行 Hadoop 叢集中的分散式複製。 執行 Distcp 命令時，它會先列出將複製的所有檔案，並將數個對應工作建立到 Hadoop 叢集中，而且每個對應工作會執行從來源到接收的二進位複製。

複製活動支援使用 DistCp 將檔案依原樣複製到 Azure Blob (包括[分段複製](copy-activity-performance.md)或 Azure Data Lake Store)；在此情況下，它會充分利用叢集的功用，而不會在自我裝載整合執行階段上執行。 尤其是，如果您的叢集相當強大，可提供更好的複製輸送量。 根據 Azure Data Factory 中的設定而定，複製活動會自動建構 distcp 命令，並提交至 Hadoop 叢集，然後監視複製狀態。

### <a name="prerequsites"></a>先決條件

若要使用 DistCp 從 HDFS 將檔案依原樣複製到 Azure Blob (包括分段複製) 或 Azure Data Lake Store，請確定 Hadoop 叢集符合下列需求：

1. MapReduce 和 Yarn 服務已啟用。
2. Yarn 是 2.5 或更高版本。
3. HDFS 伺服器與目標資料存放區 (Azure Blob 或 Azure Data Lake Store) 整合：

    - 自從 Hadoop 2.7 開始即原生支援 Azure Blob FileSystem。 您只需要在 Hadoop env config 中指定 jar 路徑。
    - Azure Data Lake Store FileSystem 是從 Hadoop 3.0.0-alpha1 開始封裝。 如果 Hadoop 叢集低於該版本，您需要從[這裡](https://hadoop.apache.org/releases.html)手動將 ADLS 相關 jar 封裝 (azure-datalake-store.jar) 匯入至叢集，並在 Hadoop env config 中指定 jar 路徑。

4. 準備 HDFS 中的暫存資料夾。 此暫存資料夾是用來儲存 DistCp 殼層指令碼，因此會佔用 KB 層級的空間。
5. 確定 HDFS 連結服務中提供的使用者帳戶擁有下列權限：a) 提交 Yarn 中的應用程式；b) 建立子資料夾、讀取/寫入上層暫存資料夾下的檔案。

### <a name="configurations"></a>組態

以下是使用 DistCp 從 HDFS 將資料複製到 Azure Blob 的複製活動組態範例：

**範例：**

```json
"activities":[
    {
        "name": "CopyFromHDFSToBlob",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "HDFSDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "BlobDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "HdfsSource",
                "distcpSettings": {
                    "resourceManagerEndpoint": "resourcemanagerendpoint:8088",
                    "tempScriptPath": "/usr/hadoop/tempscript",
                    "distcpOptions": "-strategy dynamic -map 100"
                }
            },
            "sink": {
                "type": "BlobSink"
            }
        }
    }
]
```

## <a name="use-kerberos-authentication-for-hdfs-connector"></a>對 HDFS 連接器使用 Kerberos 驗證

有兩種選項可以設定內部部署環境，以便在 HDFS 連接器中使用 Kerberos 驗證。 您可以選擇較適合您案例的選項。
* 選項 1：[加入 Kerberos 領域中的自我裝載整合執行階段機器](#kerberos-join-realm)
* 選項 2：[啟用 Windows 網域和 Kerberos 領域之間的相互信任](#kerberos-mutual-trust)

### <a name="kerberos-join-realm"></a>選項 1：加入 Kerberos 領域中的自我裝載整合執行階段機器

#### <a name="requirements"></a>需求

* 自我裝載整合執行階段機器必須加入 Kerberos 領域，且不可加入任何 Windows 網域。

#### <a name="how-to-configure"></a>如何設定

**在自我裝載整合執行階段機器上：**

1.  執行 **Ksetup** 公用程式來設定 Kerberos KDC 伺服器和領域。

    電腦必須設定為工作群組的成員，因為 Kerberos 領域與 Windows 網域不同。 若要進行此操作，您可以設定 Kerberos 領域並新增 KDC 伺服器，如下所示。 視需要以您自己的個別領域取代 *REALM.COM*。

            C:> Ksetup /setdomain REALM.COM
            C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>

    在執行這 2 個命令後**重新啟動**電腦。

2.  使用 **Ksetup** 命令確認組態。 輸出應該會像這樣：

            C:> Ksetup
            default realm = REALM.COM (external)
            REALM.com:
                kdc = <your_kdc_server_address>

**在 Azure Data Factory 中：**

* 使用 **Windows 驗證**以及您用來連線到 HDFS 資料來源的 Kerberos 主體名稱和密碼，來設定 HDFS 連接器。 檢查組態詳細資料上的 [HDFS 連結服務屬性](#linked-service-properties)區段。

### <a name="kerberos-mutual-trust"></a>選項 2：啟用 Windows 網域和 Kerberos 領域之間的相互信任

#### <a name="requirements"></a>需求

*   自我裝載整合執行階段機器必須加入 Windows 網域。
*   您需要可更新網域控制站設定的權限。

#### <a name="how-to-configure"></a>如何設定

> [!NOTE]
> 視需要，以您自己的個別領域和網域控制站取代下列教學課程中的 REALM.COM 和 AD.COM。

**在 KDC 伺服器上︰**

1.  編輯 **krb5.conf** 檔案中的 KDC 組態，讓 KDC 信任參考下列組態範本的 Windows 網域。 根據預設，此組態位於 **/etc/krb5.conf**。

            [logging]
             default = FILE:/var/log/krb5libs.log
             kdc = FILE:/var/log/krb5kdc.log
             admin_server = FILE:/var/log/kadmind.log

            [libdefaults]
             default_realm = REALM.COM
             dns_lookup_realm = false
             dns_lookup_kdc = false
             ticket_lifetime = 24h
             renew_lifetime = 7d
             forwardable = true

            [realms]
             REALM.COM = {
              kdc = node.REALM.COM
              admin_server = node.REALM.COM
             }
            AD.COM = {
             kdc = windc.ad.com
             admin_server = windc.ad.com
            }

            [domain_realm]
             .REALM.COM = REALM.COM
             REALM.COM = REALM.COM
             .ad.com = AD.COM
             ad.com = AD.COM

            [capaths]
             AD.COM = {
              REALM.COM = .
             }

  設定之後**重新啟動** KDC 服務。

2.  使用下列命令，在 KDC 伺服器中準備名為 **krbtgt/REALM.COM@AD.COM** 的主體︰

            Kadmin> addprinc krbtgt/REALM.COM@AD.COM

3.  在 **hadoop.security.auth_to_local** HDFS 服務組態檔中，新增 `RULE:[1:$1@$0](.*@AD.COM)s/@.*//`。

**在網域控制站上：**

1.  執行下列 **Ksetup** 命令來新增領域項目：

            C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
            C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM

2.  建立 Windows 網域到 Kerberos 領域的信任關係。 [password] 是主體 **krbtgt/REALM.COM@AD.COM** 的密碼。

            C:> netdom trust REALM.COM /Domain: AD.COM /add /realm /passwordt:[password]

3.  選取 Kerberos 所使用的加密演算法。

    1. 移至 [伺服器管理員] > [群組原則管理] > [網域] > [群組原則物件] > [預設或作用中的網域原則]，然後進行編輯。

    2. 在 [群組原則管理編輯器] 快顯視窗中，移至 [電腦設定] > [原則] > [Windows 設定] > [安全性設定] > [本機原則] > [安全性選項]，並設定 [網路安全性: 設定 Kerberos 允許的加密類型]。

    3. 選取您想要在連線至 KDC 時使用的加密演算法。 一般來說，您可以直接選取所有選項。

        ![設定 Kerberos 的加密類型](media/connector-hdfs/config-encryption-types-for-kerberos.png)

    4. 使用 **Ksetup** 命令，指定要用於特定 REALM 的加密演算法。

                C:> ksetup /SetEncTypeAttr REALM.COM DES-CBC-CRC DES-CBC-MD5 RC4-HMAC-MD5 AES128-CTS-HMAC-SHA1-96 AES256-CTS-HMAC-SHA1-96

4.  建立網域帳戶與 Kerberos 主體之間的對應，以便在 Windows 網域中使用 Kerberos 主體。

    1. 啟動 [系統管理工具] > [Active Directory 使用者和電腦]。

    2. 按一下 [檢視] > [進階功能] 來設定進階功能。

    3. 找到您要用以建立對應的帳戶，然後按一下滑鼠右鍵以檢視 [名稱對應] > 按一下 [Kerberos 名稱] 索引標籤。

    4. 從領域中新增主體。

        ![對應安全性身分識別](media/connector-hdfs/map-security-identity.png)

**在自我裝載整合執行階段機器上：**

* 執行下列 **Ksetup** 命令來新增領域項目。

            C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
            C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM

**在 Azure Data Factory 中：**

* 使用 **Windows 驗證**以及您用來連線到 HDFS 資料來源的網域帳戶或 Kerberos 主體，來設定 HDFS 連接器。 檢查組態詳細資料上的 [HDFS 連結服務屬性](#linked-service-properties)區段。


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
