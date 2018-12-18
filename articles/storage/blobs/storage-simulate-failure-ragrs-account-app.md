---
title: 模擬在 Azure 中存取讀取權限備援儲存體時失敗 | Microsoft Docs
description: 模擬存取讀取權限異地備援儲存體時發生錯誤
services: storage
author: tamram
ms.service: storage
ms.topic: tutorial
ms.date: 12/23/2017
ms.author: tamram
ms.openlocfilehash: 84ced8a529c2e717dc3e5888466d9a2e1e7e928a
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47180938"
---
# <a name="tutorial-simulate-a-failure-in-accessing-read-access-redundant-storage"></a>教學課程：模擬存取讀取權限備援儲存體時失敗

本教學課程是一個系列的第二部分。  在本教學課程中，您可以使用 [Fiddler](#simulate-a-failure-with-fiddler) 或[靜態路由](#simulate-a-failure-with-an-invalid-static-route)，模擬對您 [read-access geo-redundant](../common/storage-redundancy-grs.md#read-access-geo-redundant-storage) (RA-GRS) 儲存體帳戶的主要端點要求失敗，而從次要端點讀取應用程式的情況。

![應用程式案例](media/storage-simulate-failure-ragrs-account-app/scenario.png)

若要完成本教學課程，您必須先完成上一個儲存體教學課程：[使用 Azure 儲存體讓應用程式資料具有高可用性][previous-tutorial]。

在本系列的第二部分中，您將瞭解如何：

> [!div class="checklist"]
> * 執行和暫停應用程式
> * 使用 [fiddler](#simulate-a-failure-with-fiddler) 或[無效的靜態路由](#simulate-a-failure-with-an-invalid-static-route)模擬失敗 
> * 模擬主要端點還原


## <a name="prerequisites"></a>必要條件

若要使用 Fiddler 模擬失敗： 

* 下載並[安裝 Fiddler](https://www.telerik.com/download/fiddler)

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="simulate-a-failure-with-fiddler"></a>使用 Fiddler 模擬失敗

若要使用 Fiddler 模擬失敗，您必須將要求的失敗回應插入 RA-GRS 儲存體帳戶的主要端點，以模擬失敗。

請遵循下列步驟，以使用 fiddler 模擬失敗和主要端點還原。

### <a name="launch-fiddler"></a>啟動 Fiddler

開啟 Fiddler，選取 [規則] 和 [自訂規則]。

![自訂 Fiddler 規則](media/storage-simulate-failure-ragrs-account-app/figure1.png)

Fiddler ScriptEditor 啟動，顯示 **SampleRules.js** 檔案。 此檔案是用來自訂 Fiddler。 請將下列範例程式碼貼至 `OnBeforeResponse` 函式中。 新程式碼會被轉為註解，以確保其所建立的邏輯不會立即實作。 完成後，選取 [檔案] 和 [儲存] 以儲存變更。

```javascript
    /*
        // Simulate data center failure
        // After it is successfully downloading the blob, pause the code in the sample,
        // uncomment these lines of script, and save the script.
        // It will intercept the (probably successful) responses and send back a 503 error. 
        // When you're ready to stop sending back errors, comment these lines of script out again 
        //     and save the changes.

        if ((oSession.hostname == "contosoragrs.blob.core.windows.net") 
            && (oSession.PathAndQuery.Contains("HelloWorld"))) {
            oSession.responseCode = 503;  
        }
    */
```

![貼上自訂規則](media/storage-simulate-failure-ragrs-account-app/figure2.png)

### <a name="start-and-pause-the-application"></a>啟動和暫停應用程式

在您的 IDE 或文字編輯器中執行應用程式。 應用程式開始從主要端點讀取時，在主控台視窗中按下**任意鍵**以暫停應用程式。

### <a name="simulate-failure"></a>模擬失敗

暫停應用程式後，您現在可以將在先前的步驟中儲存於 Fiddler 的自訂規則取消註解。 此範例程式碼會尋找 RA-GRS 儲存體帳戶要求，若路徑包含映像名稱 `HelloWorld`，則會傳回 `503 - Service Unavailable` 的回應碼。

瀏覽至 Fiddler，然後選取 [規則] -> [自訂規則...]。將下列幾行取消註解，並將 `STORAGEACCOUNTNAME` 取代為您儲存體帳戶的名稱。 選取 [檔案] -> [儲存]，以儲存變更。 

> [!NOTE]
> 如果您在 Linux 上執行範例應用程式，您必須在每次編輯 **CustomRule.js** 檔案後重新啟動 Fiddler，讓 Fiddler 安裝自訂邏輯。 
> 
> 


```javascript
         if ((oSession.hostname == "STORAGEACCOUNTNAME.blob.core.windows.net")
         && (oSession.PathAndQuery.Contains("HelloWorld"))) {
         oSession.responseCode = 503;
         }
```

若要繼續執行應用程式，請按**任意鍵**。

應用程式再次開始執行後，針對主要端點的要求即會開始發生失敗。 應用程式會嘗試重新連線至主要端點 5 次。 超過五次的失敗嘗試閥值後，其會從次要唯讀端點要求映像。 應用程式從次要端點成功取出 20 次映像後，即會嘗試連線至主要端點。 若仍然無法連線至主要端點，則應用程式會繼續從次要端點讀取。 此模式即是在上個教學課程中述及的[斷路器模式](https://docs.microsoft.com/azure/architecture/patterns/circuit-breaker)。

![貼上自訂規則](media/storage-simulate-failure-ragrs-account-app/figure3.png)

### <a name="simulate-primary-endpoint-restoration"></a>模擬主要端點還原

使用上個步驟中設定的 Fiddler 自訂規則集，會使主要端點要求失敗。 若要再次模擬主要端點功能，請移除邏輯以插入 `503` 錯誤。

若要暫停應用程式，請按下**任意鍵**。

瀏覽至 Fiddler，然後選取 [規則] 和 [自訂規則...]。註解或移除 `OnBeforeResponse` 函式中的自訂邏輯，並保留預設函式。 選取 [檔案] 和 [儲存] 以儲存變更。

![移除自訂規則](media/storage-simulate-failure-ragrs-account-app/figure5.png)

完成時，請按下**任意鍵**繼續執行應用程式。 此應用程式會繼續從主要端點讀取，直至達到 999 次讀取為止。

![繼續執行應用程式](media/storage-simulate-failure-ragrs-account-app/figure4.png)


## <a name="simulate-a-failure-with-an-invalid-static-route"></a>使用無效的靜態路由模擬失敗 
您可以建立無效的靜態路由，並將其用於所有對您[讀取權限異地備援](../common/storage-redundancy-grs.md#read-access-geo-redundant-storage) (RA-GRS) 儲存體帳戶之主要端點的要求。 在本教學課程中，會使用本機主機作為將要求路由傳送到儲存體帳戶的閘道。 以本機主機作為閘道，會導致所有對儲存體帳戶主要端點的要求在主機內形成迴圈，進而導致失敗。 請遵循下列步驟，以使用無效的靜態路由模擬失敗和主要端點還原。 

### <a name="start-and-pause-the-application"></a>啟動和暫停應用程式

在您的 IDE 或文字編輯器中執行應用程式。 應用程式開始從主要端點讀取時，在主控台視窗中按下**任意鍵**以暫停應用程式。 

### <a name="simulate-failure"></a>模擬失敗

在應用程式暫停的情況下，在 Windows 上以系統管理員身分啟動命令提示字元，或在 Linux 上以 root 身分執行終端機。 若要取得儲存體帳戶主要端點網域的相關資訊，請在命令提示字元或終端機上輸入下列命令。

```
nslookup STORAGEACCOUNTNAME.blob.core.windows.net
``` 
 使用您的儲存體帳戶名稱取代 `STORAGEACCOUNTNAME`。 將儲存體帳戶的 IP 位址複製到文字編輯器，供後續使用。 若要取得本機主機的 IP 位址，請在 Windows 命令提示字元上輸入 `ipconfig`，或在 Linux 終端機上輸入 `ifconfig`。 

若要為目的地主機新增靜態路由，請在 Windows 命令提示字元或 Linux 終端機上輸入下列命令。 


# <a name="linuxtablinux"></a>[Linux](#tab/linux)

  route add <destination_ip> gw <gateway_ip>

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

  route add <destination_ip> <gateway_ip>

---
 
將 `<destination_ip>` 取代為您的儲存體帳戶 IP 位址，並將 `<gateway_ip>` 取代為您的本機主機 IP 位址。 若要繼續執行應用程式，請按**任意鍵**。

應用程式再次開始執行後，針對主要端點的要求即會開始發生失敗。 應用程式會嘗試重新連線至主要端點 5 次。 超過五次的失敗嘗試閥值後，其會從次要唯讀端點要求映像。 應用程式從次要端點成功取出 20 次映像後，即會嘗試連線至主要端點。 若仍然無法連線至主要端點，則應用程式會繼續從次要端點讀取。 此模式即是在上個教學課程中述及的[斷路器模式](/azure/architecture/patterns/circuit-breaker.md)。

### <a name="simulate-primary-endpoint-restoration"></a>模擬主要端點還原

若要再次模擬主要端點的運作，請從路由表中刪除主要端點的靜態路由。 這可讓所有對主要端點的要求透過預設閘道路由傳送。 

若要刪除目的地主機 (儲存體帳戶) 的靜態路由，請在 Windows 命令提示字元或 Linux 終端機上輸入下列命令。 
 
# <a name="linuxtablinux"></a>[Linux](#tab/linux)

route del <destination_ip> gw <gateway_ip>

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

route delete <destination_ip>

---

按**任意鍵**以繼續執行應用程式。 此應用程式會繼續從主要端點讀取，直至達到 999 次讀取為止。

![繼續執行應用程式](media/storage-simulate-failure-ragrs-account-app/figure4.png)


## <a name="next-steps"></a>後續步驟

在系列的第二部分中，您已了解模擬失敗以測試讀取權限異地備援儲存體的相關資訊，例如如何：

> [!div class="checklist"]
> * 執行和暫停應用程式
> * 使用 [fiddler](#simulate-a-failure-with-fiddler) 或[無效的靜態路由](#simulate-a-failure-with-an-invalid-static-route)模擬失敗 
> * 模擬主要端點還原

請閱讀下列文章，以深入了解 RA-GRS 儲存體的運作方式 (及其相關聯的風險)。

> [!div class="nextstepaction"]
> [使用 RA-GRS 設計 HA 應用程式](../common/storage-designing-ha-apps-with-ragrs.md)

[previous-tutorial]: storage-create-geo-redundant-storage.md