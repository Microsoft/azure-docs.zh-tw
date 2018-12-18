---
title: 整合遠端監視解決方案與 Azure Data Lake Store | Microsoft Docs
description: 了解如何使用 Azure 串流分析作業來整合遠端監視解決方案與 Azure Data Lake Store。
author: philmea
manager: timlt
ms.author: philmea
ms.date: 04/29/2018
ms.topic: conceptual
ms.service: iot-accelerators
services: iot-accelerators
ms.openlocfilehash: a918866ff5e206ea4d2dedde2711424924a478fe
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39188059"
---
# <a name="integrate-the-remote-monitoring-solution-with-azure-data-lake-store"></a>整合遠端監視解決方案與 Azure Data Lake Store

您可能會有超出遠端監視解決方案中所提供功能的進階分析需求。 Azure Data Lake Store 最適合此用途，因為它可以儲存來自龐大且互異資料集的資料，並與 Azure Data Lake Analytics 整合以提供即時分析。

在本操作說明中，您會使用 Azure 串流分析作業將來自遠端監視解決方案 IoT 中樞的資料串流至 Azure Data Lake Store。

## <a name="prerequisites"></a>必要條件

若要完成本操作說明，您需要下列各項：

* [部署遠端監視解決方案加速器](quickstart-remote-monitoring-deploy.md)。
  * 遠端監視解決方案會將要在本文中使用的 IoT 中樞與 Azure 串流分析作業部署到您的 Azure 訂用帳戶。
* [部署 Azure Data Lake Store](../data-lake-store/data-lake-store-get-started-portal.md)
  * 您的 Data Lake Store 應該要部署到與您遠端監視解決方案相同的區域。
  * 在您的帳戶中[建立資料夾](../data-lake-store/data-lake-store-get-started-portal.md#createfolder)，並將它命名為 "streaming"。

## <a name="create-a-consumer-group"></a>建立取用者群組

在遠端監視解決方案的 IoT 中樞中建立專屬的取用者群組。 串流分析作業將使用此群組來將資料串流至您的 Data Lake Store。

> [!NOTE]
> 應用程式會使用取用者群組從 Azure IoT 中樞提取資料。 您應該針對每五個輸出取用者建立一個新的取用者群組。 您最多可以建立 32 個取用者群組。

1. 登入 Azure 入口網站。

1. 在 Azure 入口網站中，按一下 [Cloud Shell] 按鈕。

    ![入口網站啟動圖示](./media/iot-accelerators-integrate-data-lake/portal-launch-icon.png)

1. 執行下列命令以建立新的取用者群組：

```azurecli-interactive
az iot hub consumer-group create --hub-name contoso-rm30263 --name streamanalyticsjob --resource-group contoso-rm
```

> [!NOTE]
> 使用您遠端監視解決方案的資源群組及 IoT 中樞名稱。

## <a name="create-stream-analytics-job"></a>建立串流分析作業

建立 Azure 串流分析作業以將來自 IoT 中樞的資料串流至您的 Azure Data Lake Store。

1. 按一下 [建立資源]，從 Marketplace 選取 [物聯網]，然後按一下 [串流分析作業]。

    ![新增串流分析作業](./media/iot-accelerators-integrate-data-lake/new-stream-analytics-job.png)

1. 輸入作業名稱，並選取適當的訂用帳戶和資源群組。

1. 選取接近您 Data Lake Store 的區域或是位於相同區域中的位置。 我們在此處會使用 [美國東部]。

1. 請務必讓主控環境保留為預設的 [雲端]。

1. 按一下頁面底部的 [新增] 。

    ![建立串流分析作業](./media/iot-accelerators-integrate-data-lake/create-stream-analytics-job.png)

## <a name="configure-the-stream-analytics-job"></a>設定串流分析作業

1. 在您的遠端監視解決方案資源群組中，移至該**串流分析作業**。

1. 在 [概觀] 頁面上，按一下 [輸入]。

    ![[概觀] 頁面](./media/iot-accelerators-integrate-data-lake/stream-analytics-overview.png)

1. 按一下 [新增串流輸入]，然後從下拉式清單中選取 [IoT 中樞]。

    ![新增輸入](./media/iot-accelerators-integrate-data-lake/stream-analytics-add-input.png)

1. 在 [新的輸入] 索引標籤上，輸入 **IoTHub** 作為 [輸入別名]。

1. 從 [取用者群組] 下拉式清單中，選取您稍早建立的取用者群組。 我們在此處是使用 [streamanalyticsjob]。

    ![選取輸入](./media/iot-accelerators-integrate-data-lake/stream-analytics-new-input.png)

1. 按一下 [檔案] 。

1. 在 [概觀] 頁面上，按一下 [輸出]。

    ![新增 Data Lake Store](./media/iot-accelerators-integrate-data-lake/stream-analytics-overview-2.png)

1. 按一下 [新增]，然後從下拉式清單中選取 [Data Lake Store]。

    ![新增輸出](./media/iot-accelerators-integrate-data-lake/stream-analytics-output.png)

1. 在 [新的輸出] 索引標籤中，輸入 **DataLakeStore** 作為 [輸出別名]。

1. 選取您在上一個步驟中建立的 Data Lake Store 帳戶，並提供資料夾結構以將資料串流至存放區。

1. 在 [路徑前置詞模式] 欄位中，輸入 **/streaming/{date}/{time}**。 保留 YYYY/MM/DD 的預設日期格式及 HH 的時間格式。

    ![提供資料夾結構](./media/iot-accelerators-integrate-data-lake/stream-analytics-new-output.png)

1. 按一下 [授權]。

    您必須以 Data Lake Store 授權，以提供串流分析作業寫入檔案系統的存取權。

    ![針對 Data Lake Store 授權串流分析](./media/iot-accelerators-integrate-data-lake/stream-analytics-out-authorize.png)

    您將會看到快顯視窗，而且在關閉該快顯視窗之後，[授權] 按鈕會在完成授權之後呈現灰色。

    > [!NOTE]
    > 如果您在快顯視窗中看見錯誤，請以無痕模式開啟新的瀏覽器視窗，然後再試一次。

1. 按一下 [檔案] 。

## <a name="edit-the-stream-analytics-query"></a>編輯串流分析查詢

Azure 串流分析會使用類似 SQL 的查詢語言，來指定要串流資料的輸入來源，視需要轉換該資料，然後將它輸出至各種不同的儲存體或處理目的地。

1. 在 [概觀] 索引標籤上，按一下 [編輯查詢]。

    ![Edit Query](./media/iot-accelerators-integrate-data-lake/stream-analytics-edit-query.png)

1. 在查詢編輯器中，使用您先前所定義的值來取代 [YourOutputAlias] 和 [YourInputAlias] 預留位置。

    ```sql
    SELECT
        *, System.Timestamp as time
    INTO
        DataLakeStore
    FROM
        IoTHub
    ```

    ![串流分析查詢](./media/iot-accelerators-integrate-data-lake/stream-analytics-query.png)

1. 按一下 [檔案] 。
1. 按一下 [是] 以接受變更。

## <a name="start-the-stream-analytics-job"></a>啟動串流分析工作

1. 在 [概觀] 索引標籤上，按一下 [啟動]。

    ![啟動串流分析作業](./media/iot-accelerators-integrate-data-lake/stream-analytics-start.png)

1. 在 [啟動工作] 索引標籤上，按一下 [自訂]。

1. 將自訂時間往前設定數個小時，以收集從裝置開始進行串流之後的資料。

1. 按一下 [啟動] 。

    ![挑選自訂日期](./media/iot-accelerators-integrate-data-lake/stream-analytics-start-custom.png)

    等候作業進入執行狀態；如果您看到錯誤，則可能是來自您的查詢，請務必確認語法正確。

    ![工作正在執行](./media/iot-accelerators-integrate-data-lake/stream-analytics-running.png)

    串流作業將開始從您的 IoT 中樞讀取資料，並將資料儲存於您的 Data Lake Store 中。 可能需要幾分鐘的時間，資料才會開始出現在您的 Data Lake Store 中。

## <a name="explore-the-streaming-data"></a>瀏覽串流資料

1. 前往您的 Data Lake Store。

1. 在 [概觀] 索引標籤上，按一下 [資料總管]。

1. 在 [資料總管] 中，向下切入至 **/streaming** 資料夾。 您將看到使用 YYYY/MM/DD/HH 格式建立的資料夾。

    ![探索串流資料](./media/iot-accelerators-integrate-data-lake/data-lake-store-data-explorer.png)

    您將會看到以每小時一個檔案的頻率產生的 json 檔案。

    ![探索串流資料](./media/iot-accelerators-integrate-data-lake/data-lake-store-file-preview.png)

## <a name="next-steps"></a>後續步驟

Azure Data Lake Analytics 可用來在您的 Data Lake Store 資料集上執行巨量資料分析。 請參閱 [Data Lake Analytics 文件](https://docs.microsoft.com/azure/data-lake-analytics)以深入了解。
