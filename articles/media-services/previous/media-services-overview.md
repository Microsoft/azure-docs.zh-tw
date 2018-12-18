---
title: Azure 媒體服務概觀 | Microsoft Docs
description: 本主題提供 Azure 媒體服務的概觀
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 10/15/2018
ms.author: juliako
ms.openlocfilehash: cc6cb768e9403a09ea09309e0ffd85ce9012cb41
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49352081"
---
# <a name="azure-media-services-overview"></a>Azure 媒體服務概觀 

> [!div class="op_single_selector" title1="Select the version of Media Services that you are using:"]
> * [第 2 版 - 正式推出](media-services-overview.md)
> * [第 3 版 - 預覽](../latest/media-services-overview.md)

Microsoft Azure 媒體服務 (AMS) 是一個可延伸的雲端式平台，供開發人員建置可擴充的媒體管理和傳遞應用程式。 媒體服務是以 REST API 為基礎，可讓您安全地上傳、儲存、編碼和封裝視訊或音訊內容，以用於隨選和即時資料流傳遞給各種用戶端 (例如電視、電腦和行動裝置)。

您可以建置完全採用媒體服務的端對端工作流程。 您也可以選擇在工作流程的某些部分採用第三方元件。 例如，使用第三方編碼器來進行編碼； 然後使用媒體服務上傳、保護、封裝、傳遞。 您可以選擇即時串流您的內容或隨選傳遞內容。 

## <a name="prerequisites"></a>必要條件

若要開始使用 Azure 媒體服務，您應該具備下列項目：

* 一個 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費試用帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com)。
* Azure 媒體服務帳戶。 如需詳細資訊，請參閱[建立帳戶](media-services-portal-create-account.md)。
* (選擇性) 設定開發環境。 針對開發環境選擇 .NET 或 REST API。 如需詳細資訊，請參閱 [設定環境](media-services-dotnet-how-to-use.md)。

    此外，請了解如何[以程式設計方式連線至 AMS API](media-services-use-aad-auth-to-access-ams-api.md)。
* 已啟動狀態的標準或進階串流端點。  如需詳細資訊，請參閱[管理串流端點](media-services-portal-manage-streaming-endpoints.md)

## <a name="sdks-and-tools"></a>SDK 及工具

若要建置媒體服務解決方案，您可以使用：

* [媒體服務 REST API](https://docs.microsoft.com/rest/api/media/operations/azure-media-services-rest-api-reference)
* 其中一個可用的用戶端 SDK：
    * [Azure Media Services SDK for .NET](https://github.com/Azure/azure-sdk-for-media-services)、
    * [Azure SDK for Java](https://github.com/Azure/azure-sdk-for-java)、
    * [Azure PHP SDK](https://github.com/Azure/azure-sdk-for-php)、
    * [Azure Media Services for Node.js](https://github.com/michelle-becker/node-ams-sdk/blob/master/lib/request.js) (這是非 Microsoft 版本的 Node.js SDK。 它是由社群維護，且目前並沒有 AMS API 的 100% 涵蓋範圍)。
* 現有的工具：
    * [Azure 入口網站](https://portal.azure.com/)
    * [Azure-Media-Services-Explorer](https://github.com/Azure/Azure-Media-Services-Explorer) (Azure 媒體服務總管 (AMSE) 是適用於 Windows 的 Winforms/C# 應用程式)

> [!NOTE]
> 若要取得最新的 Java SDK 版本並開始使用 Java 進行開發，請參閱[開始使用適用於媒體服務的 Java 用戶端 SDK ](https://docs.microsoft.com/azure/media-services/media-services-java-how-to-use)。 <br/>
> 若要下載適用於媒體服務的最新 PHP SDK，請在 [Packagist 存放庫](https://packagist.org/packages/microsoft/windowsazure#v0.5.7)中尋找 0.5.7 版的 Microsoft/WindowAzure 套件。  

## <a name="code-samples"></a>程式碼範例

在 **Azure 程式碼範例**資源庫中找到多個程式碼範例：[Azure 媒體服務程式碼範例](https://azure.microsoft.com/resources/samples/?service=media-services&sort=0)。

## <a name="concepts"></a>概念

如需 Azure 媒體服務概念，請參閱 [概念](media-services-concepts.md)。

## <a name="supported-scenarios-and-availability-of-media-services-across-data-centers"></a>跨資料中心支援的媒體服務情節和可用性

如需詳細資訊，請參閱[跨資料中心的 AMS 功能和服務情節和可用性](scenarios-and-availability.md)。

## <a name="service-level-agreement-sla"></a>服務等級協定 (SLA)

* 對於媒體服務編碼，我們保證 REST API 交易可用性高達 99.9%。
* 對於串流，我們在購買一個標準或近界串流端點時，針對現有的媒體內容，可保證以 99.9% 的可用性成功服務要求。
* 對於即時通道，我們保證執行中的通道至少 99.9% 的時間具有外部連線能力。
* 對於內容保護，我們保證至少 99.9% 的時間將會成功滿足金鑰要求。
* 對於索引子，我們將會在 99.9% 的時間成功服務編碼保留單元處理的索引子工作要求。

如需詳細資訊，請參閱 [Microsoft Azure SLA](https://azure.microsoft.com/support/legal/sla/)。

如需資料中心內可用性的資訊，請參閱[可用性](scenarios-and-availability.md#availability)一節。

## <a name="support"></a>支援

[Azure 支援](https://azure.microsoft.com/support/options/) 提供 Azure 的支援選項，包括媒體服務。

## <a name="provide-feedback"></a>提供意見反應

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
