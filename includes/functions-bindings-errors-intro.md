Azure Functions [觸發程序和繫結](..\articles\azure-functions\functions-triggers-bindings.md)會與各種 Azure 服務通訊。 在與這些服務整合時，您可能會引發錯誤，而這些錯誤是源自基礎 Azure 服務的 API。 當您嘗試使用 REST 或用戶端程式庫，透過您的函式程式碼與其他服務通訊時，也可能會發生錯誤。 若要避免遺失資料並確保函式有正常行為，請務必要處理來自上述任一來源的錯誤。

下列觸發程序有內建的重試支援：

* [Azure Blob 儲存體](../articles/azure-functions/functions-bindings-storage-blob.md)
* [Azure 佇列儲存體](../articles/azure-functions/functions-bindings-storage-queue.md)
* [Azure 服務匯流排 (佇列/主題)](../articles/azure-functions/functions-bindings-service-bus.md)

根據預設，這些觸發程序最多會重試五次。 在第五次重試之後，這些觸發程序就會在特殊的[有害佇列](..\articles\azure-functions\functions-bindings-storage-queue.md#trigger---poison-messages)中寫入一則訊息。 

其他 Functions 觸發程序並未內建可在函式執行期間發生錯誤時進行重試的機制。 為了避免在萬一函式發生錯誤時遺失觸發程序資訊，建議您在函式程式碼中使用 try-catch 區塊，以擷取任何錯誤。 當錯誤發生時，請將觸發程序傳遞至函式的資訊寫入至特殊的「有害」訊息佇列。 這個方法和 [Blob 儲存體觸發程序](..\articles\azure-functions\functions-bindings-storage-blob.md#trigger---poison-blobs)所使用的方法相同。 

如此一來，您就可以擷取可能會因錯誤而遺失的觸發程序事件，並於稍後使用其他函式來重試這些事件，以使用所儲存的資訊處理有害佇列中的訊息。  
