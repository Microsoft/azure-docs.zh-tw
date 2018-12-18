---
title: 視覺化 Power BI 儀表板中的 Azure IoT Central 資料 | Microsoft Docs
description: 您可以使用 Azure IoT Central Analytics Power BI 解決方案範本來視覺化及分析 IoT Central 資料。
ms.service: iot-central
services: iot-central
author: viv-liu
ms.author: viviali
ms.date: 07/16/2018
ms.topic: conceptual
ms.openlocfilehash: 6f8bed8aba43d77cdfad49f4fe62a4c7aa2c5ce3
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46368064"
---
# <a name="visualize-and-analyze-your-azure-iot-central-data-in-a-power-bi-dashboard"></a>視覺化與分析 Power BI 儀表板中的 Azure IoT Central 資料

*本主題適用於系統管理員。*

![Power BI 解決方案範本管線](media/howto-connect-powerbi/iot-continuous-data-export.png)

您可以使用 Azure IoT Central Analytics Power BI 解決方案範本來建立功能強大的 Power BI 儀表板，以監視 IoT 裝置的效能。 在 Power BI 儀表板中，您可以：
- 追蹤裝置在一段時間內傳送的資料量
- 在遙測、狀態和事件之間比較資料量
- 識別報告許多量測的裝置
- 觀察裝置量測的歷程記錄趨勢
- 識別傳送許多重大事件的問題裝置

此解決方案範本會設定管道，以便從[連續資料匯出](howto-export-data.md)中取得 Azure Blob 儲存體帳戶中的資料。 此資料會流至 Azure Functions、Azure Data Factory 和 Azure SQL Database，以便處理和轉換資料。 此輸出可在 Power BI 報告中進行視覺化與分析，您可以 PBIX 檔案格式下載該報告。 所有這些資源都會建立在您的 Azure 訂用帳戶中，因此您可以根據需求來自訂每個元件。 此解決方案範本完全是開放原始碼架構，您可以瀏覽 [GitHub 存放庫](https://aka.ms/iotcentralgithubpowerbisolutiontemplate)來深入了解此架構及擴充解決方案。

## <a name="get-the-azure-iot-central-analytics-solution-templatehttpsakamsiotcentralpowerbisolutiontemplate-from-microsoft-appsource"></a>從 Microsoft AppSource 取得 [Azure IoT Central Analytics 解決方案範本](https://aka.ms/iotcentralpowerbisolutiontemplate)

## <a name="prerequisites"></a>必要條件
設定範本需要下列項目：
- Azure 訂用帳戶的存取權
- 使用[連續資料匯出](howto-export-data.md)從 IoT Central 應用程式匯出的資料。 我們建議您開啟量測、裝置和裝置範本串流，以充分利用 Power BI 儀表板。
- Power BI Desktop (最新版)
- Power BI Pro (如果您想與其他人共用儀表板)

## <a name="reports"></a>報告

系統會自動產生兩份報告。 

第一份報告會顯示裝置所報告的量測歷程記錄檢視，並細分不同類型的量測，以及已送出最多量測的裝置。

![Power BI 報告第一頁](media/howto-connect-powerbi/template-page1-hasdata.PNG)

第二份報告會深入探討到事件，並顯示錯誤和警告報告的歷程記錄檢視。 其中也會顯示哪些裝置報告的事件總數最多，尤其是錯誤事件和警告事件。

![Power BI 報告第二頁](media/howto-connect-powerbi/template-page2-hasdata.PNG)

## <a name="resources"></a>資源

造訪 AppSource 以取得 [Azure IoT Central Analytics 解決方案範本](https://aka.ms/iotcentralpowerbisolutiontemplate)。

請瀏覽 [GitHub 存放庫](https://aka.ms/iotcentralgithubpowerbisolutiontemplate)，以深入了解架構並擴充解決方案。

## <a name="next-steps"></a>後續步驟

現在您已了解如何將 Power BI 中的資料視覺化，以下是建議執行的下一個步驟：

> [!div class="nextstepaction"]
> [如何管理裝置](howto-manage-devices.md)
