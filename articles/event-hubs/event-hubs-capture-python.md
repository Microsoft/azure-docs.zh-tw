---
title: Azure 事件中樞擷取逐步解說 | Microsoft Docs
description: 此範例使用 Azure Python SDK 來示範如何使用事件中樞擷取功能。
services: event-hubs
documentationcenter: ''
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: bdff820c-5b38-4054-a06a-d1de207f01f6
ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/05/2017
ms.author: shvija
ms.openlocfilehash: b38e64891ce1065290d46f0fae2d22e151e03e4d
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40005609"
---
# <a name="event-hubs-capture-walkthrough-python"></a>事件中樞擷取逐步解說︰Python

擷取是 Azure 事件中樞的功能。 您可以使用此功能，將事件中樞內的串流資料自動傳遞給您選擇的 Azure Blob 儲存體帳戶。 此功能可讓您輕鬆地對即時串流資料執行批次處理。 本文說明如何搭配使用事件中樞擷取與 Python。 如需事件中樞擷取的詳細資訊，請參閱[概觀文章](event-hubs-capture-overview.md)。

此範例使用 [Azure Python SDK](https://azure.microsoft.com/develop/python/) 來示範「擷取」功能。 sender.py 程式會以 JSON 格式將模擬的環境遙測傳送至事件中樞。 事件中樞已設定為使用擷取功能，將此資料批次寫入至 Blob 儲存體。 capturereader.py 應用程式會讀取這些 Blob，並為每個裝置建立附加檔案。 此應用程式接著會將資料寫入至 .csv 檔案。

## <a name="what-youll-accomplish"></a>您將完成的作業

1. 使用 Azure 入口網站，建立 Azure Blob 儲存體帳戶和其中所含的 Blob 容器。
2. 使用 Azure 入口網站，建立事件中樞命名空間。
3. 使用 Azure 入口網站，建立已啟用擷取功能的事件中樞。
4. 使用 Python 指令碼，將資料傳送到事件中樞。
5. 使用另一個 Python 指令碼，讀取擷取的檔案並加以處理。

## <a name="prerequisites"></a>必要條件

- Python 2.7.x
- Azure 訂用帳戶
- 作用中的[事件中樞命名空間和事件中樞](event-hubs-create.md)

[!INCLUDE [create-account-note](../../includes/create-account-note.md)]

## <a name="create-an-azure-blob-storage-account"></a>建立 Azure Blob 儲存體帳戶
1. 登入 [Azure 入口網站][Azure portal]。
2. 在入口網站的左窗格中，選取 [新增] > [儲存體] > [儲存體帳戶]。
3. 完成 [建立儲存體帳戶] 窗格中的選項，然後選取 [建立]。
   
   ![[建立儲存體帳戶] 窗格][1]
4. 在看到**部署成功**訊息之後，選取新儲存體帳戶的名稱，並在 [基本功能] 窗格中選取 [Blob]。 當 [Blob 服務] 窗格開啟時，選取頂端的 [+ 容器]。 將容器命名為**擷取**，然後關閉 [Blob 服務] 窗格。
5. 選取左窗格中的 [存取金鑰]，然後複製儲存體帳戶名稱和 **key1** 的值。 將這些值儲存到記事本或一些其他暫存位置。

## <a name="create-a-python-script-to-send-events-to-your-event-hub"></a>建立 Python 指令碼以將事件傳送到事件中樞
1. 開啟您慣用的 Python 編輯器，例如 [Visual Studio 程式碼][Visual Studio Code]。
2. 建立稱為 **sender.py**的指令碼。 此指令碼會將 200 個事件傳送到事件中樞。 這些事件是以 JSON 格式傳送的簡單環境數據。
3. 將下列程式碼貼到 sender.py：
   
   ```python
   import uuid
   import datetime
   import random
   import json
   from azure.servicebus import ServiceBusService
   
   sbs = ServiceBusService(service_namespace='INSERT YOUR NAMESPACE NAME', shared_access_key_name='RootManageSharedAccessKey', shared_access_key_value='INSERT YOUR KEY')
   devices = []
   for x in range(0, 10):
       devices.append(str(uuid.uuid4()))
   
   for y in range(0,20):
       for dev in devices:
           reading = {'id': dev, 'timestamp': str(datetime.datetime.utcnow()), 'uv': random.random(), 'temperature': random.randint(70, 100), 'humidity': random.randint(70, 100)}
           s = json.dumps(reading)
           sbs.send_event('INSERT YOUR EVENT HUB NAME', s)
       print y
   ```
4. 更新上述程式碼，以使用您在建立「事件中樞」命名空間時取得的命名空間名稱、金鑰值及事件中樞名稱。

## <a name="create-a-python-script-to-read-your-capture-files"></a>建立 Python 指令碼來讀取擷取檔案

1. 填妥窗格，然後選取 [建立]。
2. 建立名為 **capturereader.py** 的指令碼。 此指令碼會讀取擷取檔案，並為每個裝置建立檔案以便只寫入該裝置的資料。
3. 將下列程式碼貼到 capturereader.py：
   
   ```python
   import os
   import string
   import json
   import avro.schema
   from avro.datafile import DataFileReader, DataFileWriter
   from avro.io import DatumReader, DatumWriter
   from azure.storage.blob import BlockBlobService
   
   def processBlob(filename):
       reader = DataFileReader(open(filename, 'rb'), DatumReader())
       dict = {}
       for reading in reader:
           parsed_json = json.loads(reading["Body"])
           if not 'id' in parsed_json:
               return
           if not dict.has_key(parsed_json['id']):
               list = []
               dict[parsed_json['id']] = list
           else:
               list = dict[parsed_json['id']]
               list.append(parsed_json)
       reader.close()
       for device in dict.keys():
           deviceFile = open(device + '.csv', "a")
           for r in dict[device]:
               deviceFile.write(", ".join([str(r[x]) for x in r.keys()])+'\n')
   
   def startProcessing(accountName, key, container):
       print 'Processor started using path: ' + os.getcwd()
       block_blob_service = BlockBlobService(account_name=accountName, account_key=key)
       generator = block_blob_service.list_blobs(container)
       for blob in generator:
           #content_length == 508 is an empty file, so only process content_length > 508 (skip empty files)
           if blob.properties.content_length > 508:
               print('Downloaded a non empty blob: ' + blob.name)
               cleanName = string.replace(blob.name, '/', '_')
               block_blob_service.get_blob_to_path(container, blob.name, cleanName)
               processBlob(cleanName)
               os.remove(cleanName)
           block_blob_service.delete_blob(container, blob.name)
   startProcessing('YOUR STORAGE ACCOUNT NAME', 'YOUR KEY', 'capture')
   ```
4. 在 `startProcessing` 的呼叫中，貼上儲存體帳戶名稱和金鑰的適當值。

## <a name="run-the-scripts"></a>執行指令碼
1. 開啟在其路徑中具有 Python 的命令提示字元，並執行下列命令來安裝 Python 必要條件封裝︰
   
   ```
   pip install azure-storage
   pip install azure-servicebus
   pip install avro
   ```
   
   如果您有舊版的 **azure-storage** 或 **azure**，您可能需要使用 **--upgrade** 選項。
   
   您可能也需要執行下列命令 (在大部分系統上並不需要)：
   
   ```
   pip install cryptography
   ```
2. 將目錄變更為儲存了 sender.py 和 capturereader.py 的任何位置，並執行此命令︰
   
   ```
   start python sender.py
   ```
   
   此命令會啟動新的 Python 程序來執行傳送器。
3. 等候擷取執行幾分鐘的時間。 然後在原始命令視窗中輸入下列命令︰
   
   ```
   python capturereader.py
   ```

   此擷取處理器會使用本機目錄從儲存體帳戶/容器下載所有 Blob。 它會處理任何非空白的 Blob，並將結果以 .csv 檔案的形式寫入到本機目錄。

## <a name="next-steps"></a>後續步驟

您可以使用下列連結來深入了解事件中樞：

* [事件中樞擷取概觀][Overview of Event Hubs Capture]
* [使用事件中樞的完整範例應用程式](https://github.com/Azure/azure-event-hubs/tree/master/samples)
* [事件中樞概觀][Event Hubs overview]

[Azure portal]: https://portal.azure.com/
[Overview of Event Hubs Capture]: event-hubs-capture-overview.md
[1]: ./media/event-hubs-archive-python/event-hubs-python1.png
[About Azure storage accounts]:../storage/common/storage-create-storage-account.md
[Visual Studio Code]: https://code.visualstudio.com/
[Event Hubs overview]: event-hubs-what-is-event-hubs.md
