---
title: 如何整合 Azure API 管理與 Azure Application Insights | Microsoft Docs
description: 了解如何在 Azure Application Insights 中從「Azure API 管理」記錄和檢視事件。
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/20/2018
ms.author: apimpm
ms.openlocfilehash: 52e034f9a0c11c2b27888d181304bc16c3369e4a
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49390018"
---
# <a name="how-to-integrate-azure-api-management-with-azure-application-insights"></a>如何整合 Azure API 管理與 Azure Application Insights

「Azure API 管理」可與 Azure Application Insights 輕鬆整合，Azure Application Insights 是一項延伸服務，可供 Web 開發人員在多個平台上建置和管理應用程式。 本指南會逐步解說此整合的每個步驟，並說明降低「API 管理」服務執行個體效能影響的策略。

## <a name="prerequisites"></a>必要條件

若要依照本指南進行操作，您必須具備「Azure API 管理」執行個體。 如果您沒有該執行個體，請先完成此[教學課程](get-started-create-service-instance.md)。

## <a name="create-an-azure-application-insights-instance"></a>建立 Azure Application Insights 執行個體

您必須先建立 Azure Application Insights 執行個體，才能使用此服務。

1. 開啟「Azure 入口網站」，然後瀏覽至 [Application Insights]。  
    ![App Insights 建立](media/api-management-howto-app-insights/apim-app-insights-instance-1.png)  
2. 按一下 [+ 新增]。  
    ![App Insights 建立](media/api-management-howto-app-insights/apim-app-insights-instance-2.png)  
3. 填入表單。 選擇 [一般] 作為 [應用程式類型]。
4. 按一下頁面底部的 [新增] 。

## <a name="create-a-connection-between-azure-application-insights-and-azure-api-management-service-instance"></a>在 Azure Application Insights 與 Azure API 管理服務執行個體之間建立連線

1. 在「Azure 入口網站」中，瀏覽至您的「Azure API 管理服務執行個體」。
2. 從左側功能表中，選取 [Application Insights]。
3. 按一下 [+ 新增]。  
    ![App Insights 記錄器](media/api-management-howto-app-insights/apim-app-insights-logger-1.png)  
4. 選取先前建立的 **Application Insights** 執行個體，然後提供簡短描述。
5. 按一下頁面底部的 [新增] 。
6. 您剛建立了一個含有檢測金鑰的 Azure Application Insights 記錄器。 它現在應該會出現在清單中。  
    ![App Insights 記錄器](media/api-management-howto-app-insights/apim-app-insights-logger-2.png)  

> [!NOTE]
> 在幕後，[記錄器](https://docs.microsoft.com/rest/api/apimanagement/logger/createorupdate)實體會建立於 API 管理執行個體，其中包含 Application Insights 執行個體的檢測金鑰。

## <a name="enable-application-insights-logging-for-your-api"></a>為您的 API 啟用 Application Insights 記錄功能

1. 在「Azure 入口網站」中，瀏覽至您的「Azure API 管理服務執行個體」。
2. 從左側功能表中選取 [API]。
3. 按一下您的 API，在此案例中為 [Demo Conference API]。
4. 從頂端列移至 [設定] 索引標籤。
5. 向下捲動至 [診斷記錄] 區段。  
    ![App Insights 記錄器](media/api-management-howto-app-insights/apim-app-insights-api-1.png)  
6. 選取 [啟用] 方塊。
7. 從 [目的地] 下拉式清單中，選取已連結的記錄器。
8. 輸入 **100** 作為 [取樣 (%)]，然後勾選 [一律記錄錯誤] 核取方塊。
9. 在 [本文的前幾個位元組] 欄位中，輸入 **1024**。
10. 按一下 [檔案] 。

> [!NOTE]
> 在幕後，名為 'applicationinsights' 的[診斷](https://docs.microsoft.com/rest/api/apimanagement/diagnostic/createorupdate)實體會建立於 API 層級。

| 設定名稱                        | 值類型                        | 說明                                                                                                                                                                                                                                                                                                                                      |
|-------------------------------------|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 啟用                              | 布林值                           | 指定是否要啟用此 API 的記錄功能。                                                                                                                                                                                                                                                                                                |
| 目的地                         | Azure Application Insights 記錄器 | 指定要使用的 Azure Application Insights 記錄器                                                                                                                                                                                                                                                                                           |
| 取樣 (%)                        | decimal                           | 值為 0 到 100 (百分比)。 <br/> 指定將記錄至 Azure Application Insights 的要求百分比。 取樣為 0% 時，表示不會記錄任何要求，取樣為 100% 時，則表示會記錄所有要求。 <br/> 此設定可用來降低將要求記錄至 Azure Application Insights 所帶來的效能影響 (請參閱下一節)。 |
| 一律記錄錯誤                   | 布林值                           | 如果選取此設定，則不論 [取樣] 設定為何，都會將所有失敗都記錄至 Azure Application Insights。                                                                                                                                                                                                                  |
| 基本選項：標頭              | list                              | 指定針對要求和回應，將記錄至 Azure Application Insights 的標頭。  預設值：不會記錄任何標頭。                                                                                                                                                                                                             |
| 基本選項：本文的前幾個位元組  | integer                           | 指定針對要求和回應，要記錄至 Azure Application Insights 的本文前幾個位元組數目。  預設值：不記錄本文。                                                                                                                                                                                              |
| 進階選項：前端要求  |                                   | 指定是否要將「前端要求」記錄至 Azure Application Insights 及如何記錄。 「前端要求」是傳入至「Azure API 管理」服務的要求。                                                                                                                                                                        |
| 進階選項：前端回應 |                                   | 指定是否要將「前端回應」記錄至 Azure Application Insights 及如何記錄。 「前端回應」是從「Azure API 管理」服務傳出的回應。                                                                                                                                                                   |
| 進階選項：後端要求   |                                   | 指定是否要將「後端要求」記錄至 Azure Application Insights 及如何記錄。 「後端要求」是從「Azure API 管理」服務傳出的要求。                                                                                                                                                                        |
| 進階選項：後端回應  |                                   | 指定是否要將「後端回應」記錄至 Azure Application Insights 及如何記錄。 「後端回應」是傳入至「Azure API 管理」服務的回應。                                                                                                                                                                       |

> [!NOTE]
> 您可以指定不同層級的記錄器 - 單一 API 記錄器或用於所有 API 的記錄器。
>  
> 當您同時指定兩者時：
> + 如果兩者是不同記錄器，將會同時使用兩者 (多工記錄)，
> + 如果兩者是相同記錄器但設定不同，則用於單一 API (更細微的層級) 的記錄器將會覆寫用於所有 API 的記錄器。

## <a name="what-data-is-added-to-azure-application-insights"></a>哪些資料會新增至 Azure Application Insights

Azure Application Insights 會接收：

+ 每個連入要求 (「前端要求」、「前端回應」) 的「要求」遙測項目，
+ 轉送給後端服務之每個要求 (「後端要求」、「後端回應」) 的「相依性」 遙測項目，
+ 每個失敗要求的「例外狀況」遙測項目。

失敗要求係指有下列情況的要求：

+ 因用戶端連線中斷而失敗，或
+ 觸發了 API 原則的 *on-error* 區段，或
+ 擁有符合 4xx 或 5xx 的回應 HTTP 狀態碼。

## <a name="performance-implications-and-log-sampling"></a>效能影響與記錄取樣

> [!WARNING]
> 視連入要求速率而定，記錄所有事件可能會帶來嚴重的效能影響。

根據內部負載測試，當要求速率超過每秒 1,000 個要求時，啟用此功能造成了輸送量降低 40%-50%。 Azure Application Insights 是設計成使用統計資料分析來評估應用程式效能。 它不是用來作為稽核系統，也不適用於記錄大量 API 的每個個別要求。

您可以透過調整 [取樣] 設定 (請參閱上述步驟) 來操控要記錄的要求數目。 值為 100% 時，表示會記錄所有要求，值為 0% 時，則表示完全不進行記錄。 **取樣**有助於減少遙測量，可有效地防止嚴重的效能衰退，同時仍享有記錄功能的優點。

將要求和回應的標頭與本文略過而不進行記錄，對於緩和效能問題也有正面影響。

## <a name="video"></a>影片

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2pkXv]
>
>

## <a name="next-steps"></a>後續步驟

+ 深入了解 [Azure Application Insights](https://docs.microsoft.com/azure/application-insights/)。
+ 考慮[使用 Azure 事件中樞來進行記錄](api-management-howto-log-event-hubs.md)。
