---
title: 使用 .NET 的 Azure 儲存體範例 | Microsoft Docs
description: 檢視、下載及執行 Azure 儲存體的範例程式碼和應用程式。 探索使用 .NET 儲存體用戶端程式庫之 Blob、佇列、資料表和檔案的入門範例。
services: storage
author: seguler
ms.service: storage
ms.devlang: dotnet
ms.topic: article
ms.date: 01/12/2017
ms.author: seguler
ms.component: common
ms.openlocfilehash: 663df77a7fb574f05cfaa9378dff53ca5db21c49
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39526104"
---
# <a name="azure-storage-samples-using-net"></a>使用 .NET 的 Azure 儲存體範例

## <a name="net-sample-index"></a>.NET 範例索引

下表提供我們的範例儲存機制和每個範例所涵蓋案例的概觀。 按一下連結即可檢視 GitHub 中對應的範例程式碼。

<table style="font-size:90%"><thead><tr><th style="font-size:110%">端點</th><th style="font-size:110%">案例</th><th style="font-size:110%">範例程式碼</th></tr></thead><tbody> 
<tr> 
<td rowspan="16"><b>Blob</b></td>
<td>附加 Blob</td> 
<td><a href="https://msdn.microsoft.com/library/microsoft.windowsazure.storage.blob.cloudblobcontainer.getappendblobreference.aspx">CloudBlobContainer.GetAppendBlobReference 方法範例 (英文)</a></td> 
</tr> 
<tr> 
<td>區塊 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blobs-dotnet-webapp/blob/master/WebApp-Storage-DotNet/Controllers/HomeController.cs">Azure Blob Storage Photo Gallery Web Application (Azure Blob 儲存體影像中心 Web 應用程式)</a></td>
</tr> 
<tr> 
<td>用戶端加密</td>
<td><a href="https://github.com/Azure/azure-storage-net/blob/master/Samples/GettingStarted/EncryptionSamples/BlobGettingStarted/Program.cs">Blob 加密範例</a></td>
</tr> 
<tr> 
<td>複製 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">開始使用 Blob</a></td>
</tr> 
<tr> 
<td>建立容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blobs-dotnet-webapp/blob/master/WebApp-Storage-DotNet/Controllers/HomeController.cs">Azure Blob Storage Photo Gallery Web Application (Azure Blob 儲存體影像中心 Web 應用程式)</a></td>
</tr> 
<tr> 
<td>刪除 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blobs-dotnet-webapp/blob/master/WebApp-Storage-DotNet/Controllers/HomeController.cs">Azure Blob Storage Photo Gallery Web Application (Azure Blob 儲存體影像中心 Web 應用程式)</a></td>
</tr> 
<tr> 
<td>刪除容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">開始使用 Blob</a></td>
</tr> 
<tr> 
<td>Blob 中繼資料/屬性/統計資料</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">開始使用 Blob</a></td>
</tr> 
<tr> 
<td>容器 ACL/中繼資料/屬性</td>
<td><a href="https://github.com/Azure-Samples/storage-blobs-dotnet-webapp/blob/master/WebApp-Storage-DotNet/Controllers/HomeController.cs">Azure Blob Storage Photo Gallery Web Application (Azure Blob 儲存體影像中心 Web 應用程式)</a></td>
</tr> 
<tr> 
<td>取得頁面範圍</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">開始使用 Blob</a></td>
</tr> 
<tr> 
<td>租用 Blob/容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">開始使用 Blob</a></td>
</tr> 
<tr> 
<td>列出 Blob/容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/GettingStarted.cs">開始使用 Blob</a></td>
</tr> 
<tr> 
<td>分頁 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/GettingStarted.cs">開始使用 Blob</a></td>
</tr>
<tr> 
<td>SAS</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">開始使用 Blob</a></td>
</tr>   
<tr> 
<td>服務屬性</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">開始使用 Blob</a></td>
</tr>           
<tr> 
<td>快照 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-back-up-with-incremental-snapshots/blob/master/Program.cs">使用增量快照備份 Azure 虛擬機器磁碟</a></td>
</tr> 
<tr> 
<td rowspan="9"><b>檔案</b></td>
<td>建立共用/目錄/檔案</td> 
<td><a href="https://github.com/Azure/azure-storage-net/blob/master/Samples/GettingStarted/VisualStudioQuickStarts/DataFileStorage/Program.cs">Azure 儲存體 .NET 檔案儲存體範例</a></td> 
</tr>
<tr> 
<td>刪除共用/目錄/檔案</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/master/FileStorage/GettingStarted.cs">Getting Started with Azure File Service in .NET (開始使用 .NET 中的 Azure 檔案服務)</a></td> 
</tr> 
<tr> 
<td>目錄屬性/中繼資料</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/9f12304b2f5f5472a1c87c1e21be4af5661ac043/FileStorage/Advanced.cs">Azure 儲存體 .NET 檔案儲存體範例</a></td> 
</tr> 
<tr> 
<td>下載檔案</td> 
<td><a href="https://github.com/Azure/azure-storage-net/blob/master/Samples/GettingStarted/VisualStudioQuickStarts/DataFileStorage/Program.cs">Azure 儲存體 .NET 檔案儲存體範例</a></td> 
</tr> 
<tr> 
<td>檔案屬性/中繼資料/計量</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/9f12304b2f5f5472a1c87c1e21be4af5661ac043/FileStorage/Advanced.cs">Azure 儲存體 .NET 檔案儲存體範例</a></td> 
</tr> 
<tr> 
<td>檔案服務屬性</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/9f12304b2f5f5472a1c87c1e21be4af5661ac043/FileStorage/Advanced.cs">Azure 儲存體 .NET 檔案儲存體範例</a></td> 
</tr> 
<tr> 
<td>列出目錄和檔案</td> 
<td><a href="https://github.com/Azure/azure-storage-net/blob/master/Samples/GettingStarted/VisualStudioQuickStarts/DataFileStorage/Program.cs">Azure 儲存體 .NET 檔案儲存體範例</a></td> 
</tr>
<tr> 
<td>列出共用</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/9f12304b2f5f5472a1c87c1e21be4af5661ac043/FileStorage/Advanced.cs">Azure 儲存體 .NET 檔案儲存體範例</a></td> 
</tr>
<tr> 
<td>共用屬性/中繼資料/統計資料</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/9f12304b2f5f5472a1c87c1e21be4af5661ac043/FileStorage/Advanced.cs">Azure 儲存體 .NET 檔案儲存體範例</a></td> 
</tr>
<tr> 
<td rowspan="8"><b>佇列</b></td>
<td>新增訊息</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/GettingStarted.cs">Getting Started with Azure Queue Service in .NET (開始使用 .NET 中的 Azure 佇列服務)</a></td> 
</tr> 
<tr> 
<td>用戶端加密</td> 
<td><a href="https://github.com/Azure/azure-storage-net/blob/master/Samples/GettingStarted/EncryptionSamples/QueueGettingStarted/Program.cs">Azure 儲存體 .NET 佇列用戶端加密</a></td> 
</tr> 
<tr> 
<td>建立佇列</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/GettingStarted.cs">Getting Started with Azure Queue Service in .NET (開始使用 .NET 中的 Azure 佇列服務)</a></td> 
</tr> 
<tr> 
<td>刪除訊息/佇列</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/GettingStarted.cs">Getting Started with Azure Queue Service in .NET (開始使用 .NET 中的 Azure 佇列服務)</a></td> 
</tr> 
<tr> 
<td>查看訊息</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/GettingStarted.cs">Getting Started with Azure Queue Service in .NET (開始使用 .NET 中的 Azure 佇列服務)</a></td> 
</tr> 
<tr> 
<td>佇列 ACL/中繼資料/統計資料</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/Advanced.cs">Getting Started with Azure Queue Service in .NET (開始使用 .NET 中的 Azure 佇列服務)</a></td> 
</tr> 
<tr> 
<td>佇列服務屬性</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/Advanced.cs">Getting Started with Azure Queue Service in .NET (開始使用 .NET 中的 Azure 佇列服務)</a></td> 
</tr> 
<tr> 
<td>更新訊息</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/GettingStarted.cs">Getting Started with Azure Queue Service in .NET (開始使用 .NET 中的 Azure 佇列服務)</a></td> 
</tr> 
<tr> 
<td rowspan="7"><b>資料表</b></td>
<td>建立資料表</td> 
<td><a href="https://code.msdn.microsoft.com/Managing-Concurrency-using-56018114/sourcecode?fileId=123913&pathId=50196262">使用 Azure 儲存體管理並行存取 - 範例應用程式</a></td> 
</tr> 
<tr> 
<td>刪除實體/資料表</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-dotnet-getting-started/blob/master/TableStorage/BasicSamples.cs">在 .NET 中開始使用 Azure 表格儲存體</a></td> 
</tr> 
<tr> 
<td>插入/合併/取代實體</td> 
<td><a href="https://code.msdn.microsoft.com/Managing-Concurrency-using-56018114/sourcecode?fileId=123913&pathId=50196262">使用 Azure 儲存體管理並行存取 - 範例應用程式</a></td> 
</tr> 
<tr> 
<td>查詢實體</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-dotnet-getting-started/blob/master/TableStorage/BasicSamples.cs">在 .NET 中開始使用 Azure 表格儲存體</a></td> 
</tr> 
<tr> 
<td>查詢資料表</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-dotnet-getting-started/blob/master/TableStorage/BasicSamples.cs">在 .NET 中開始使用 Azure 表格儲存體</a></td> 
</tr> 
<tr> 
<td>資料表 ACL/屬性</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-dotnet-getting-started/blob/master/TableStorage/AdvancedSamples.cs">在 .NET 中開始使用 Azure 表格儲存體</a></td> 
</tr> 
<tr> 
<td>更新實體</td> 
<td><a href="https://code.msdn.microsoft.com/Managing-Concurrency-using-56018114/sourcecode?fileId=123913&pathId=50196262">使用 Azure 儲存體管理並行存取 - 範例應用程式</a></td> 
</tr> 
</tbody> 
</table>
<br/>

## <a name="azure-code-samples-library"></a>Azure 程式碼範例程式庫

若要檢視完整的範例程式庫，請移至 [Azure 程式碼範例](https://azure.microsoft.com/resources/samples/?service=storage)程式庫，其中包含您可以下載並在本機執行的「Azure 儲存體」範例。 程式碼範例程式庫會提供 .zip 格式的範例程式碼。 或者，您可以瀏覽並複製每個範例的 GitHub 儲存機制。

[!INCLUDE [storage-dotnet-samples-include](../../../includes/storage-dotnet-samples-include.md)]

## <a name="getting-started-guides"></a>入門指南

如果您要尋找有關如何安裝和開始使用「Azure 儲存體用戶端程式庫」的指示，請查看下列指南。

* [Getting Started with Azure Blob Service in .NET (開始使用 .NET 中的 Azure Blob 服務)](../blobs/storage-dotnet-how-to-use-blobs.md)
* [Getting Started with Azure Queue Service in .NET (開始使用 .NET 中的 Azure 佇列服務)](../storage-dotnet-how-to-use-queues.md)
* [Getting Started with Azure Table Service in .NET (開始使用 .NET 中的 Azure 表格服務)](../../cosmos-db/table-storage-how-to-use-dotnet.md)
* [Getting Started with Azure File Service in .NET (開始使用 .NET 中的 Azure 檔案服務)](../storage-dotnet-how-to-use-files.md)

## <a name="next-steps"></a>後續步驟

如需其他語言的範例相關資訊︰

* Java：[使用 Java 的 Azure 儲存體範例](storage-samples-java.md)
* 所有其他語言︰[Azure 儲存體範例](../storage-samples.md)
