---
title: 變更預設的 Blob 路徑 | Microsoft Docs
description: 了解如何設定 Azure 函式來重新命名 blob 檔案路徑
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 01/16/2018
ms.author: alkohli
ms.openlocfilehash: f73d9dcedee5165af752b9e10fb70de860e8e98b
ms.sourcegitcommit: 7edfa9fbed0f9e274209cec6456bf4a689a4c1a6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2018
ms.locfileid: "27862395"
---
# <a name="change-a-blob-path-from-the-default-path"></a>變更預設的 blob 路徑

StorSimple 資料管理員服務在轉換資料時，預設會將經過轉換的 blob 放在您於建立目標存放庫期間所指定的儲存體容器內。 當 blob 抵達此位置時，建議您將這些 blob 移到其他位置。 本文說明如何設定 Azure 函式來重新命名預設的 blob 檔案路徑，並從此將 blob 移到其他位置。

## <a name="prerequisites"></a>先決條件

確定 StorSimple 資料管理員服務中已有正確設定的作業定義。

## <a name="create-an-azure-function"></a>建立 Azure 函式

若要建立 Azure 函式，請執行下列步驟：

1. 移至 [Azure 入口網站](http://portal.azure.com/)。

2. 按一下 [+ 建立資源]。 在 [搜尋] 方塊中輸入**函式應用程式**，然後按 **Enter** 鍵。 在顯示的應用程式清單中選取並按一下 [函式應用程式]。

    ![在 [搜尋] 方塊中輸入「函式應用程式」](./media/storsimple-data-manager-change-default-blob-path/search-function-app.png)

3. 按一下頁面底部的 [新增] 。

    ![函式應用程式視窗的 [建立] 按鈕](./media/storsimple-data-manager-change-default-blob-path/create-function-app.png)

4. 在 [函式應用程式] 設定刀鋒視窗中，執行下列步驟︰

    1. 提供唯一的 [應用程式名稱]。
    2. 從下拉式清單中選取 [訂用帳戶]。 這必須是與 StorSimple 資料管理員服務相關聯的訂用帳戶。
    3. 選取 [建立新的] 資源群組。
    4. 在 [主控方案] 下拉式清單中選取 [取用方案]。
    5. 指定用來執行函式的位置。 建議您讓 StorSimple 資料管理員服務以及與作業定義相關聯的儲存體帳戶位於相同區域中。
    6. 選取現有的儲存體帳戶或建立新的儲存體帳戶。 儲存體帳戶會在內部針對函式使用。

        ![輸入新的函式應用程式組態資料](./media/storsimple-data-manager-change-default-blob-path/function-app-parameters.png)

    7. 按一下頁面底部的 [新增] 。 函式應用程式隨即建立。
     
        ![已建立的函式應用程式](./media/storsimple-data-manager-change-default-blob-path/function-app-created.png)

5. 選取 [函式]，然後按一下 [+ 新增函式]。

    ![按一下 [+ 新增函式]](./media/storsimple-data-manager-change-default-blob-path/create-new-function.png)

6. 選取 [C#] 作為語言。 在範本圖格的陣列中，選取 [QueueTrigger-CSharp] 圖格中的 [C#]。

7. 在 [佇列觸發程序] 中︰

    1. 輸入函式的 [名稱]。
    2. 在 [佇列名稱] 方塊中，輸入資料轉換作業定義名稱。
    3. 在 [儲存體帳戶連線] 底下，按一下 [新增]。 從儲存體帳戶清單中，選取與作業定義相關聯的帳戶。 記下連線名稱 (已醒目提示)。 稍後在 Azure 函式中需要此名稱。

        ![建立新的 C# 函式](./media/storsimple-data-manager-change-default-blob-path/new-function-parameters.png)

    4. 按一下頁面底部的 [新增] 。 **函式**隨即建立。

     
10. 在 [函式] 視窗中，執行 .csx 檔案。

    ![建立新的 C# 函式](./media/storsimple-data-manager-change-default-blob-path/new-function-run-csx.png)
    
    請執行下列步驟：

    1. 貼上下列程式碼：

        ```
        using System;
        using System.Configuration;
        using Microsoft.WindowsAzure.Storage.Blob;
        using Microsoft.WindowsAzure.Storage.Queue;
        using Microsoft.WindowsAzure.Storage;
        using System.Collections.Generic;
        using System.Linq;

        public static void Run(QueueItem myQueueItem, TraceWriter log)
        {
            CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConfigurationManager.AppSettings["STORAGE_CONNECTIONNAME"]);

            string storageAccUriEndswith = "windows.net/";
            string uri = myQueueItem.TargetLocation.Replace("%20", " ");
            log.Info($"Blob Uri: {uri}");

            // Remove storage account uri string
            uri = uri.Substring(uri.IndexOf(storageAccUriEndswith) + storageAccUriEndswith.Length);

            string containerName = uri.Substring(0, uri.IndexOf("/")); 

            // Remove container name string
            uri = uri.Substring(containerName.Length + 1);

            // Current blob path
            string blobName = uri; 

            string volumeName = uri.Substring(containerName.Length + 1);
            volumeName = uri.Substring(0, uri.IndexOf("/"));

            // Remove volume name string
            uri = uri.Substring(volumeName.Length + 1);

            string newContainerName = uri.Substring(0, uri.IndexOf("/")).ToLower();
            string newBlobName = uri.Substring(newContainerName.Length + 1);

            log.Info($"Container name: {containerName}");
            log.Info($"Volume name: {volumeName}");
            log.Info($"New container name: {newContainerName}");

            log.Info($"Blob name: {blobName}");
            log.Info($"New blob name: {newBlobName}");

            // Create the blob client.
            CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

            // Container reference
            CloudBlobContainer container = blobClient.GetContainerReference(containerName);
            CloudBlobContainer newContainer = blobClient.GetContainerReference(newContainerName);
            newContainer.CreateIfNotExists();

            if(!container.Exists())
            {
                log.Info($"Container - {containerName} not exists");
                return;
            }

            if(!newContainer.Exists())
            {
                log.Info($"Container - {newContainerName} not exists");
                return;
            }

            CloudBlockBlob blob = container.GetBlockBlobReference(blobName);
            if (!blob.Exists())
            {
                // Skip to copy the blob to new container, if source blob doesn't exist
                log.Info($"The specified blob does not exist.");
                log.Info($"Blob Uri: {blob.Uri}");
                return;
            }

            CloudBlockBlob blobCopy = newContainer.GetBlockBlobReference(newBlobName);
            if (!blobCopy.Exists())
            {
                blobCopy.StartCopy(blob);
                // Delete old blob, after copy to new container
                blob.DeleteIfExists();
                log.Info($"Blob file path renamed completed successfully");
            }
            else
            {
                log.Info($"Blob file path renamed already done");
                // Delete old blob, if already exists.
                blob.DeleteIfExists();
            }
        }

        public class QueueItem
        {
            public string SourceLocation {get;set;}
            public long SizeInBytes {get;set;}
            public string Status {get;set;}
            public string JobID {get;set;}
            public string TargetLocation {get; set;}
        }

        ```

    2. 將第 11 行中的 **STORAGE_CONNECTIONNAME** 替換為您的儲存體帳戶連線 (請參閱步驟 7c)。

        ![複製儲存體連線名稱](./media/storsimple-data-manager-change-default-blob-path/new-function-storage-connection-name.png)

    3. [儲存] 函式。

        ![儲存函式](./media/storsimple-data-manager-change-default-blob-path/save-function.png)

12. 若要完成此函式，請執行下列步驟，多新增一個檔案︰

    1. 按一下 [檢視檔案]。

       ![[檢視檔案] 連結](./media/storsimple-data-manager-change-default-blob-path/view-files.png)

    2. 按一下 [+ 新增]。
        
        ![[檢視檔案] 連結](./media/storsimple-data-manager-change-default-blob-path/new-function-add-file.png)
    
    3. 輸入 **project.json**，然後按 **Enter** 鍵。 在 **project.json** 檔案中貼上下列程式碼：

        ```
        {
        "frameworks": {
            "net46":{
            "dependencies": {
                "windowsazure.storage": "8.1.1"
            }
            }
        }
        }

        ```

    
    4. 按一下 [檔案] 。

        ![[檢視檔案] 連結](./media/storsimple-data-manager-change-default-blob-path/new-function-project-json.png)

您已建立 Azure 函式。 每次資料轉換作業產生新的 blob 時就會觸發此函式。

## <a name="next-steps"></a>後續步驟

[使用 StorSimple Data Manager UI 來轉換資料](storsimple-data-manager-ui.md)
