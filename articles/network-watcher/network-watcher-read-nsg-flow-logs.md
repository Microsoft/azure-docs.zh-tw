---
title: 讀取 NSG 流量記錄 | Microsoft Docs
description: 本文說明如何剖析 NSG 流量記錄
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/25/2017
ms.author: jdial
ms.openlocfilehash: 63407382762a814ded4529caa109d76e987c9505
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43286439"
---
# <a name="read-nsg-flow-logs"></a>讀取 NSG 流量記錄

了解如何使用 PowerShell 讀取 NSG 流量記錄項目。

NSG 流量記錄會以[區塊 Blob](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) 的形式儲存在儲存體帳戶中。 區塊 Blob 是由較小的區塊組成。 每個記錄都是於每小時產生的個別區塊 Blob。 每小時都會產生新的記錄檔，而每幾分鐘就會以最新的資料更新記錄檔。 您可以透過本文了解讀取部分流量記錄的方式。

## <a name="scenario"></a>案例

在以下案例中，儲存體帳戶中已經存有一個範例流程記錄檔。 您已了解如何選擇性讀取 NSG 流量記錄中最新的事件。 在本文中，您會使用 PowerShell，但本文中所討論的概念並不限於該程式設計語言，且適用於 Azure 儲存體 API 所支援的所有語言。

## <a name="setup"></a>設定

開始之前，您必須在帳戶中的一或多個網路安全性群組上，啟用網路安全性群組流程記錄。 如需有關啟用網路安全性流程記錄的指示，請參閱下列文章︰[網路安全性群組的流程記錄簡介](network-watcher-nsg-flow-logging-overview.md)。

## <a name="retrieve-the-block-list"></a>擷取區塊清單

以下的 PowerShell 會設定查詢 NSG 流程記錄檔 Blob 所需的變數，並會列出 [CloudBlockBlob](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.storage.blob.cloudblockblob?view=azurestorage-8.1.3) \(英文\) 區塊 Blob 內的區塊。 更新指令碼以包含適用於您環境的有效值。

```powershell
function Get-NSGFlowLogBlockList {
    [CmdletBinding()]
    param (
        [string] [Parameter(Mandatory=$true)] $subscriptionId,
        [string] [Parameter(Mandatory=$true)] $NSGResourceGroupName,
        [string] [Parameter(Mandatory=$true)] $NSGName,
        [string] [Parameter(Mandatory=$true)] $storageAccountName,
        [string] [Parameter(Mandatory=$true)] $storageAccountResourceGroup,
        [string] [Parameter(Mandatory=$true)] $macAddress,
        [datetime] [Parameter(Mandatory=$true)] $logTime
    )

    process {
        # Retrieve the primary storage account key to access the NSG logs
        $StorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $storageAccountResourceGroup -Name $storageAccountName).Value[0]

        # Setup a new storage context to be used to query the logs
        $ctx = New-AzureStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

        # Container name used by NSG flow logs
        $ContainerName = "insights-logs-networksecuritygroupflowevent"

        # Name of the blob that contains the NSG flow log
        $BlobName = "resourceId=/SUBSCRIPTIONS/${subscriptionId}/RESOURCEGROUPS/${NSGResourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/${NSGName}/y=$($logTime.Year)/m=$(($logTime).ToString("MM"))/d=$(($logTime).ToString("dd"))/h=$(($logTime).ToString("HH"))/m=00/macAddress=$($macAddress)/PT1H.json"

        # Gets the storage blog
        $Blob = Get-AzureStorageBlob -Context $ctx -Container $ContainerName -Blob $BlobName

        # Gets the block blog of type 'Microsoft.WindowsAzure.Storage.Blob.CloudBlob' from the storage blob
        $CloudBlockBlob = [Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob] $Blob.ICloudBlob

        # Stores the block list in a variable from the block blob.
        $blockList = $CloudBlockBlob.DownloadBlockList()

        # Return the Block List
        $blockList
    }
}
$blockList = Get-NSGFlowLogBlockList -subscriptionId "00000000-0000-0000-0000-000000000000" -NSGResourceGroupName "resourcegroupname" -storageAccountName "storageaccountname" -storageAccountResourceGroup "sa-rg" -macAddress "000D3AF8196E" -logTime "03/07/2018 22:00"
```

`$blockList` 變數會傳回 Blob 中的區塊清單。 每個區塊 Blob 都至少包含兩個區塊。  第一個區塊的長度為 `21` 個位元組，此區塊包含 json 記錄檔的左括號。 另一個區塊是右括號，且長度為 `9` 個位元組。  您可以看到下列範例記錄檔中有七個項目，每個都是個別項目。 記錄檔中的所有新項目都會新增到末端，位於最後一個區塊之前。

```
Name                                         Length Committed
----                                         ------ ---------
ZDk5MTk5N2FkNGE0MmY5MTk5ZWViYjA0YmZhODRhYzY=     12      True
NzQxNDA5MTRhNDUzMGI2M2Y1MDMyOWZlN2QwNDZiYzQ=   2685      True
ODdjM2UyMWY3NzFhZTU3MmVlMmU5MDNlOWEwNWE3YWY=   2586      True
ZDU2MjA3OGQ2ZDU3MjczMWQ4MTRmYWNhYjAzOGJkMTg=   2688      True
ZmM3ZWJjMGQ0ZDA1ODJlOWMyODhlOWE3MDI1MGJhMTc=   2775      True
ZGVkYTc4MzQzNjEyMzlmZWE5MmRiNjc1OWE5OTc0OTQ=   2676      True
ZmY2MjUzYTIwYWIyOGU1OTA2ZDY1OWYzNmY2NmU4ZTY=   2777      True
Mzk1YzQwM2U0ZWY1ZDRhOWFlMTNhYjQ3OGVhYmUzNjk=   2675      True
ZjAyZTliYWE3OTI1YWZmYjFmMWI0MjJhNzMxZTI4MDM=      2      True
```

## <a name="read-the-block-blob"></a>讀取區塊 Blob

接下來，您需要讀取 `$blocklist` 變數以擷取資料。 在這個範例中，我們會逐一查看區塊清單，從每個區塊讀取位元組，並將它們儲存在陣列中。 使用 [DownloadRangeToByteArray](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblob.downloadrangetobytearray?view=azurestorage-8.1.3#Microsoft_WindowsAzure_Storage_Blob_CloudBlob_DownloadRangeToByteArray_System_Byte___System_Int32_System_Nullable_System_Int64__System_Nullable_System_Int64__Microsoft_WindowsAzure_Storage_AccessCondition_Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_Microsoft_WindowsAzure_Storage_OperationContext_) 方法來擷取資料。

```powershell
# Set the size of the byte array to the largest block
$maxvalue = ($blocklist | measure Length -Maximum).Maximum

# Create an array to store values in
$valuearray = @()

# Define the starting index to track the current block being read
$index = 0

# Loop through each block in the block list
for($i=0; $i -lt $blocklist.count; $i++)
{

# Create a byte array object to story the bytes from the block
$downloadArray = New-Object -TypeName byte[] -ArgumentList $maxvalue

# Download the data into the ByteArray, starting with the current index, for the number of bytes in the current block. Index is increased by 3 when reading to remove preceding comma.
$CloudBlockBlob.DownloadRangeToByteArray($downloadArray,0,$index+3,$($blockList[$i].Length-1)) | Out-Null

# Increment the index by adding the current block length to the previous index
$index = $index + $blockList[$i].Length

# Retrieve the string from the byte array

$value = [System.Text.Encoding]::ASCII.GetString($downloadArray)

# Add the log entry to the value array
$valuearray += $value
}
```

現在 `$valuearray` 陣列包含每個區塊的字串值。 若要確認項目，請透過執行 `$valuearray[$valuearray.Length-2]` 取得陣列的第二個到最後一個值。 您不會想要最後一個值，因為它是右括號。

這個值的結果會顯示在下列範例中：

```json
        {
             "time": "2017-06-16T20:59:43.7340000Z",
             "systemId": "5f4d02d3-a7d0-4ed4-9ce8-c0ae9377951c",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/CONTOSORG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/CONTOSONSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_AllowInternetOutBound","flows":[{"mac":"000D3A18077E","flowTuples":["1497646722,10.0.0.4,168.62.32.14,44904,443,T,O,A","1497646722,10.0.0.4,52.240.48.24,45218,443,T,O,A","1497646725,10.
0.0.4,168.62.32.14,44910,443,T,O,A","1497646725,10.0.0.4,52.240.48.24,45224,443,T,O,A","1497646728,10.0.0.4,168.62.32.14,44916,443,T,O,A","1497646728,10.0.0.4,52.240.48.24,45230,443,T,O,A","1497646732,10.0.0.4,168.62.32.14,44922,443,T,O,A","14976
46732,10.0.0.4,52.240.48.24,45236,443,T,O,A","1497646735,10.0.0.4,168.62.32.14,44928,443,T,O,A","1497646735,10.0.0.4,52.240.48.24,45242,443,T,O,A","1497646738,10.0.0.4,168.62.32.14,44934,443,T,O,A","1497646738,10.0.0.4,52.240.48.24,45248,443,T,O,
A","1497646742,10.0.0.4,168.62.32.14,44942,443,T,O,A","1497646742,10.0.0.4,52.240.48.24,45256,443,T,O,A","1497646745,10.0.0.4,168.62.32.14,44948,443,T,O,A","1497646745,10.0.0.4,52.240.48.24,45262,443,T,O,A","1497646749,10.0.0.4,168.62.32.14,44954
,443,T,O,A","1497646749,10.0.0.4,52.240.48.24,45268,443,T,O,A","1497646753,10.0.0.4,168.62.32.14,44960,443,T,O,A","1497646753,10.0.0.4,52.240.48.24,45274,443,T,O,A","1497646756,10.0.0.4,168.62.32.14,44966,443,T,O,A","1497646756,10.0.0.4,52.240.48
.24,45280,443,T,O,A","1497646759,10.0.0.4,168.62.32.14,44972,443,T,O,A","1497646759,10.0.0.4,52.240.48.24,45286,443,T,O,A","1497646763,10.0.0.4,168.62.32.14,44978,443,T,O,A","1497646763,10.0.0.4,52.240.48.24,45292,443,T,O,A","1497646766,10.0.0.4,
168.62.32.14,44984,443,T,O,A","1497646766,10.0.0.4,52.240.48.24,45298,443,T,O,A","1497646769,10.0.0.4,168.62.32.14,44990,443,T,O,A","1497646769,10.0.0.4,52.240.48.24,45304,443,T,O,A","1497646773,10.0.0.4,168.62.32.14,44996,443,T,O,A","1497646773,
10.0.0.4,52.240.48.24,45310,443,T,O,A","1497646776,10.0.0.4,168.62.32.14,45002,443,T,O,A","1497646776,10.0.0.4,52.240.48.24,45316,443,T,O,A","1497646779,10.0.0.4,168.62.32.14,45008,443,T,O,A","1497646779,10.0.0.4,52.240.48.24,45322,443,T,O,A"]}]}
,{"rule":"DefaultRule_DenyAllInBound","flows":[]},{"rule":"UserRule_ssh-rule","flows":[]},{"rule":"UserRule_web-rule","flows":[{"mac":"000D3A18077E","flowTuples":["1497646738,13.82.225.93,10.0.0.4,1180,80,T,I,A","1497646750,13.82.225.93,10.0.0.4,
1184,80,T,I,A","1497646768,13.82.225.93,10.0.0.4,1181,80,T,I,A","1497646780,13.82.225.93,10.0.0.4,1336,80,T,I,A"]}]}]}
        }
```

此案例為如何在不需要剖析整個記錄的情況下，讀取 NSG 流量記錄中項目的範例。 您可以透過使用區塊識別碼，或是透過追蹤儲存在區塊 Blob 中之區塊的長度，來在新項目撰寫至記錄檔時讀取它們。 這可以讓您僅讀取新項目。

## <a name="next-steps"></a>後續步驟

請造訪[使用開放原始碼工具將 Azure 網路監看員 NSG 流量記錄視覺化](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)，以深入了解檢視 NSG 流量記錄的其他方式。

若要深入了解儲存體 Blob，請造訪：[Azure Functions Blob 儲存體繫結](../azure-functions/functions-bindings-storage-blob.md)
