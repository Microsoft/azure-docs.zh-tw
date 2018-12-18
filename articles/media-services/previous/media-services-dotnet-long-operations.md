---
title: 輪詢長時間執行的作業 | Microsoft Docs
description: 本主題說明如何輪詢長時間執行的作業。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: 9a68c4b1-6159-42fe-9439-a3661a90ae03
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/09/2017
ms.author: juliako
ms.openlocfilehash: fa456a2d0bb427af80d67a0b971fe8bd41bb7868
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783077"
---
# <a name="delivering-live-streaming-with-azure-media-services"></a>利用 Azure 媒體服務提供即時資料流

## <a name="overview"></a>概觀

Microsoft Azure 媒體服務提供將要求傳送至媒體服務以啟動作業 (如建立、啟動、停止或刪除頻道) 的 API。 這些作業屬於長時間執行的作業。

Media Services .NET SDK 提供能傳送要求並等候作業完成的 API (API 會在內部依照某些間隔輪詢作業進度). 例如，當您呼叫 channel.Start() 時，方法會在通道啟動後返回。 您也可以使用非同步的 version: await channel.StartAsync() (如需以工作為基礎的非同步模式，請參閱 [TAP](https://msdn.microsoft.com/library/hh873175\(v=vs.110\).aspx))。 傳送作業要求並輪詢狀態，直到作業完成為止的 API 稱為「輪詢方法」。 我們建議豐富型用戶端應用程式和/或可設定狀態的服務使用這些方法 (尤其是非同步版本)。

我們有一些無法等候長時間執行之 http 要求，並想要的手動輪詢作業進度之應用程式的案例。 與無狀態 Web 服務互動之瀏覽器是典型的範例：當瀏覽器要求建立通道時，Web 服務會起始長時間執行的作業，並將作業識別碼傳回瀏覽器。 接著，瀏覽器會要求 Web 服務來根據識別碼取得作業狀態。 Media Services .NET SDK 提供適用於此案例的 API。 這些 API 稱為「非輪詢方法」。
「非輪詢方法」的命名模式如下：SendOperationNameOperation (例如，SendCreateOperation)。 SendOperationNameOperation 方法會傳回 **IOperation** 物件；傳回的物件含有可用來追蹤作業的資訊。 Send*OperationName*OperationAsync 方法會傳回 **Task<IOperation>**。

以下是目前支援非輪詢方法的類別：**通道**、**StreamingEndpoint** 及**程式**。

若要輪詢作業狀態，請針對 **OperationBaseCollection** 類別使用 **GetOperation** 方法。 請使用下列間隔來檢查作業狀態：對於**通道**和 **StreamingEndpoint** 作業，使用 30 秒；對於**程式**作業，使用 10 秒。

## <a name="create-and-configure-a-visual-studio-project"></a>建立和設定 Visual Studio 專案

設定您的開發環境並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)所述。

## <a name="example"></a>範例

以下範例定義名為 **ChannelOperations**的類別。 此類別定義可能是 Web 服務類別定義的起始點。 為了簡單起見，以下範例使用非同步版本的方法。

此範例也示範用戶端如何使用這個類別。

### <a name="channeloperations-class-definition"></a>ChannelOperations class definition

```csharp
using Microsoft.WindowsAzure.MediaServices.Client;
using System;
using System.Collections.Generic;
using System.Configuration;
using System.Net;

/// <summary> 
/// The ChannelOperations class only implements 
/// the Channel’s creation operation. 
/// </summary> 
public class ChannelOperations
{
    // Read values from the App.config file.
    private static readonly string _AADTenantDomain =
        ConfigurationManager.AppSettings["AMSAADTenantDomain"];
    private static readonly string _RESTAPIEndpoint =
        ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
    private static readonly string _AMSClientId =
        ConfigurationManager.AppSettings["AMSClientId"];
    private static readonly string _AMSClientSecret =
        ConfigurationManager.AppSettings["AMSClientSecret"];

    // Field for service context.
    private static CloudMediaContext _context = null;

    public ChannelOperations()
    {
        AzureAdTokenCredentials tokenCredentials = 
            new AzureAdTokenCredentials(_AADTenantDomain,
                new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                AzureEnvironments.AzureCloudEnvironment);

        var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

        _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);
    }

    /// <summary>  
    /// Initiates the creation of a new channel.  
    /// </summary>  
    /// <param name="channelName">Name to be given to the new channel</param>  
    /// <returns>  
    /// Operation Id for the long running operation being executed by Media Services. 
    /// Use this operation Id to poll for the channel creation status. 
    /// </returns> 
    public string StartChannelCreation(string channelName)
    {
        var operation = _context.Channels.SendCreateOperation(
            new ChannelCreationOptions
            {
                Name = channelName,
                Input = CreateChannelInput(),
                Preview = CreateChannelPreview(),
                Output = CreateChannelOutput()
            });

        return operation.Id;
    }

    /// <summary> 
    /// Checks if the operation has been completed. 
    /// If the operation succeeded, the created channel Id is returned in the out parameter.
    /// </summary> 
    /// <param name="operationId">The operation Id.</param> 
    /// <param name="channel">
    /// If the operation succeeded, 
    /// the created channel Id is returned in the out parameter.</param>
    /// <returns>Returns false if the operation is still in progress; otherwise, true.</returns> 
    public bool IsCompleted(string operationId, out string channelId)
    {
        IOperation operation = _context.Operations.GetOperation(operationId);
        bool completed = false;

        channelId = null;

        switch (operation.State)
        {
            case OperationState.Failed:
                // Handle the failure. 
                // For example, throw an exception. 
                // Use the following information in the exception: operationId, operation.ErrorMessage.
                break;
            case OperationState.Succeeded:
                completed = true;
                channelId = operation.TargetEntityId;
                break;
            case OperationState.InProgress:
                completed = false;
                break;
        }
        return completed;
    }

    private static ChannelInput CreateChannelInput()
    {
        return new ChannelInput
        {
            StreamingProtocol = StreamingProtocol.RTMP,
            AccessControl = new ChannelAccessControl
            {
                IPAllowList = new List<IPRange>
                    {
                        new IPRange
                        {
                            Name = "TestChannelInput001",
                            Address = IPAddress.Parse("0.0.0.0"),
                            SubnetPrefixLength = 0
                        }
                    }
            }
        };
    }

    private static ChannelPreview CreateChannelPreview()
    {
        return new ChannelPreview
        {
            AccessControl = new ChannelAccessControl
            {
                IPAllowList = new List<IPRange>
                    {
                        new IPRange
                        {
                            Name = "TestChannelPreview001",
                            Address = IPAddress.Parse("0.0.0.0"),
                            SubnetPrefixLength = 0
                        }
                    }
            }
        };
    }

    private static ChannelOutput CreateChannelOutput()
    {
        return new ChannelOutput
        {
            Hls = new ChannelOutputHls { FragmentsPerSegment = 1 }
        };
    }
}
```

### <a name="the-client-code"></a>The client code

```csharp
ChannelOperations channelOperations = new ChannelOperations();
string opId = channelOperations.StartChannelCreation("MyChannel001");

string channelId = null;
bool isCompleted = false;

while (isCompleted == false)
{
    System.Threading.Thread.Sleep(TimeSpan.FromSeconds(30));
    isCompleted = channelOperations.IsCompleted(opId, out channelId);
}

// If we got here, we should have the newly created channel id.
Console.WriteLine(channelId);
```

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

