---
title: REST API：Azure Data Lake Storage Gen1 的檔案系統作業 | Microsoft Docs
description: 使用 WebHDFS REST API 執行 Azure Data Lake Storage Gen1 上的檔案系統作業
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 62ecf3b1983853629f6bc5fd594231188aa67bcd
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391601"
---
# <a name="filesystem-operations-on-azure-data-lake-storage-gen1-using-rest-api"></a>使用 REST API 在 Azure Data Lake Storage Gen1 上進行檔案系統作業
> [!div class="op_single_selector"]
> * [.NET SDK](data-lake-store-data-operations-net-sdk.md)
> * [Java SDK](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
> 

在本文中，您將了解如何使用 WebHDFS REST API 和 Data Lake Storage Gen1 REST API 執行 Azure Data Lake Storage Gen1 上的檔案系統作業。 如需有關如何使用 REST API 在 Data Lake Storage Gen1 上執行帳戶管理作業的指示，請參閱[使用 REST API 在 Data Lake Storage Gen1 上進行帳戶管理作業](data-lake-store-get-started-rest-api.md)。

## <a name="prerequisites"></a>必要條件
* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

* **Azure Data Lake Storage Gen1 帳戶**。 請遵循[使用 Azure 入口網站開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md) 的指示。

* **[cURL](http://curl.haxx.se/)**。 本文使用 cURL 示範如何對 Data Lake Storage Gen1 帳戶呼叫 REST API。

## <a name="how-do-i-authenticate-using-azure-active-directory"></a>如何使用 Azure Active Directory 驗證？
您可以使用兩種方法來使用 Azure Active Directory 進行驗證。

* 如需讓應用程式進行使用者驗證 (互動式)，請參閱[使用 .NET SDK 向 Data Lake Storage Gen1 進行使用者驗證](data-lake-store-end-user-authenticate-rest-api.md)。
* 如需讓應用程式進行服務對服務驗證 (非互動式)，請參閱[使用 .NET SDK 向 Data Lake Storage Gen1 進行服務對服務驗證](data-lake-store-service-to-service-authenticate-rest-api.md)。


## <a name="create-folders"></a>建立資料夾
這項作業以 [這裡](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Make_a_Directory)定義的 WebHDFS REST API 呼叫為基礎。

使用下列 cURL 命令。 以您的 Data Lake Storage Gen1 帳戶名稱取代 **\<yourstorename>**。

    curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -d "" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/?op=MKDIRS'

在前述命令中，以您稍早擷取的授權權杖取代 \<`REDACTED`\>。 此命令會在 Data Lake Storage Gen1 帳戶的根資料夾下建立名為 **mytempdir** 的目錄。

如果作業順利完成，您應該會看到類似下列程式碼片段的回應︰

    {"boolean":true}

## <a name="list-folders"></a>列出資料夾
這項作業以 [這裡](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#List_a_Directory)定義的 WebHDFS REST API 呼叫為基礎。

使用下列 cURL 命令。 以您的 Data Lake Storage Gen1 帳戶名稱取代 **\<yourstorename>**。

    curl -i -X GET -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/?op=LISTSTATUS'

在前述命令中，以您稍早擷取的授權權杖取代 \<`REDACTED`\>。

如果作業順利完成，您應該會看到類似下列程式碼片段的回應︰

    {
    "FileStatuses": {
        "FileStatus": [{
            "length": 0,
            "pathSuffix": "mytempdir",
            "type": "DIRECTORY",
            "blockSize": 268435456,
            "accessTime": 1458324719512,
            "modificationTime": 1458324719512,
            "replication": 0,
            "permission": "777",
            "owner": "<GUID>",
            "group": "<GUID>"
        }]
    }
    }

## <a name="upload-data"></a>上傳資料
這項作業以 [這裡](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Create_and_Write_to_a_File)定義的 WebHDFS REST API 呼叫為基礎。

使用下列 cURL 命令。 以您的 Data Lake Storage Gen1 帳戶名稱取代 **\<yourstorename>**。

    curl -i -X PUT -L -T 'C:\temp\list.txt' -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/list.txt?op=CREATE'

在前述語法中，**-T** 參數是您要上傳檔案的位置。

輸出大致如下列程式碼片段：
   
    HTTP/1.1 307 Temporary Redirect
    ...
    Location: https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/list.txt?op=CREATE&write=true
    ...
    Content-Length: 0

    HTTP/1.1 100 Continue

    HTTP/1.1 201 Created
    ...

## <a name="read-data"></a>讀取資料
這項作業以 [這裡](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Open_and_Read_a_File)定義的 WebHDFS REST API 呼叫為基礎。

從 Data Lake Storage Gen1 帳戶讀取資料是兩個步驟的程序。

* 您要先針對端點 `https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN`提交 GET 要求。 這個呼叫會傳回要提交下一個 GET 要求的目標位置。
* 接下來您要針對端點 `https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN&read=true`提交 GET 要求。 這個呼叫會顯示檔案的內容。

不過，由於第一和第二個步驟之間的輸入參數沒有任何差異，您可以使用 `-L` 參數來提交第一個要求。 `-L` 選項基本上會將兩個要求結合為一個，並且讓 cURL 在新的位置重做要求。 最後會顯示所有要求呼叫的輸出，如下列程式碼片段所示。 以您的 Data Lake Storage Gen1 帳戶名稱取代 **\<yourstorename>**。

    curl -i -L GET -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN'

您應該會看到如下列程式碼片段的輸出：

    HTTP/1.1 307 Temporary Redirect
    ...
    Location: https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/somerandomfile.txt?op=OPEN&read=true
    ...

    HTTP/1.1 200 OK
    ...

    Hello, Data Lake Store user!

## <a name="rename-a-file"></a>重新命名檔案
這項作業以 [這裡](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Rename_a_FileDirectory)定義的 WebHDFS REST API 呼叫為基礎。

請使用下列 cURL 命令重新命名檔案。 以您的 Data Lake Storage Gen1 帳戶名稱取代 **\<yourstorename>**。

    curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -d "" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=RENAME&destination=/mytempdir/myinputfile1.txt'

您應該會看到如下列程式碼片段的輸出：

    HTTP/1.1 200 OK
    ...

    {"boolean":true}

## <a name="delete-a-file"></a>刪除檔案
這項作業以 [這裡](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Delete_a_FileDirectory)定義的 WebHDFS REST API 呼叫為基礎。

請使用下列 cURL 命令刪除檔案。 以您的 Data Lake Storage Gen1 帳戶名稱取代 **\<yourstorename>**。

    curl -i -X DELETE -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile1.txt?op=DELETE'

您應該會看到如下的輸出：

    HTTP/1.1 200 OK
    ...

    {"boolean":true}

## <a name="next-steps"></a>後續步驟
* [使用 REST API 對 Data Lake Storage Gen1 進行帳戶管理作業](data-lake-store-get-started-rest-api.md)。

## <a name="see-also"></a>另請參閱
* [Azure Data Lake Storage Gen1 REST API 參考資料](https://docs.microsoft.com/rest/api/datalakestore/)
* [與 Azure Data Lake Storage Gen1 相容的開放原始碼巨量資料應用程式](data-lake-store-compatible-oss-other-applications.md)

