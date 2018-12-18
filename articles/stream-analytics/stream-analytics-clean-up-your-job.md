---
title: 清除 Azure 串流分析作業
description: 本文是如何刪除 Azure 串流分析作業的指南。
services: stream-analytics
author: mamccrea
manager: kfile
ms.author: mamccrea
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/22/2018
ms.openlocfilehash: 580d05909ff3c94c982be5353b3b5e86a78fc43f
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38969335"
---
# <a name="clean-up-your-azure-stream-analytics-job"></a>清除 Azure 串流分析作業

Azure 串流分析作業可以透過 Azure 入口網站、Azure PowerShell、適用於 .NET 的 Azure SDK 或 REST API 輕鬆刪除。

>[!NOTE] 
>當您停止串流分析作業時，資料只會保留在輸入和輸出儲存體中，例如事件中樞或 Azure SQL Database。 如果您需要移除 Azure 中的資料，請務必遵循串流分析作業的輸入和輸出資源的移除流程。

## <a name="stop-a-job-in-azure-portal"></a>在 Azure 入口網站中停止作業

1. 登入 [Azure 入口網站](https://portal.azure.com)。 

2. 找到執行中的串流分析作業並加以選取。

3. 在 [串流分析作業] 頁面上，選取 [停止] 以停止作業。 

   ![停止作業](./media/stream-analytics-clean-up-your-job/stop-job.png)


## <a name="delete-a-job-in-azure-portal"></a>在 Azure 入口網站中刪除作業

1. 登入 Azure 入口網站。 

2. 找到現有的串流分析作業並加以選取。

3. 在 [串流分析作業] 頁面上，選取 [刪除] 以刪除作業。 

   ![刪除作業](./media/stream-analytics-clean-up-your-job/delete-job.png)


## <a name="stop-or-delete-a-job-using-powershell"></a>使用 PowerShell 停止或刪除作業

若要使用 PowerShell 停止作業，請使用 [Stop-AzureRmStreamAnalyticsJob](https://docs.microsoft.com/powershell/module/azurerm.streamanalytics/stop-azurermstreamanalyticsjob?view=azurermps-5.7.0) Cmdlet。 若要使用 PowerShell 刪除作業，請使用 [Remove-AzureRmStreamAnalyticsJob](https://docs.microsoft.com/powershell/module/azurerm.streamanalytics/Remove-AzureRmStreamAnalyticsJob?view=azurermps-5.7.0) Cmdlet。

## <a name="stop-or-delete-a-job-using-azure-sdk-for-net"></a>使用適用於 .NET 的 Azure SDK 停止或刪除作業

若要使用適用於 .NET 的 Azure SDK 停止作業，請使用 [StreamingJobsOperationsExtensions.BeginStop](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.streamanalytics.streamingjobsoperationsextensions.beginstop?view=azure-dotnet) 方法。 若要使用適用於 .NET 的 Azure SDK 刪除作業，請使用 [StreamingJobsOperationsExtensions.BeginDelete](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.streamanalytics.streamingjobsoperationsextensions.begindelete?view=azure-dotnet) 方法。

## <a name="stop-or-delete-a-job-using-rest-api"></a>使用 REST API 停止或刪除作業

若要使用 REST API 停止作業，請參閱 [Stop](https://docs.microsoft.com/rest/api/streamanalytics/stream-analytics-job#stop) 方法。 若要使用 REST API 刪除作業，請參閱 [Delete](https://docs.microsoft.com/rest/api/streamanalytics/stream-analytics-job#delete) 方法。