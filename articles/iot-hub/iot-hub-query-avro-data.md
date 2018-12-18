---
title: 使用 Azure Data Lake Analytics 查詢 Avro 資料 | Microsoft Docs
description: 使用訊息本文屬性將裝置遙測資料路由至 Blob 儲存體，並查詢寫入至 Blob 儲存體的 Avro 格式資料。
author: ash2017
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: asrastog
ms.openlocfilehash: a5b8ce8cd753ee294a8d61ba8a3dfed872f0f31a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956316"
---
# <a name="query-avro-data-by-using-azure-data-lake-analytics"></a>使用 Azure Data Lake Analytics 查詢 Avro 資料

本文討論如何查詢 Avro 資料，以有效率地將訊息從 Azure IoT 中樞路由至 Azure 服務。 [訊息路由](iot-hub-devguide-messages-d2c.md)可讓您使用豐富的查詢，根據訊息屬性、訊息本文、裝置對應項標記與裝置對應項屬性來篩選資料。 若要深入了解訊息路由中的查詢功能，請參閱有關訊息路由查詢語法的文章。 
<!--[Message Routing Query Syntax](iot-hub-devguide-routing-query-syntax.md). I don't have this article yet. -->

過往的挑戰是，當 Azure IoT 中樞將訊息路由至 Azure Blob 儲存體時，IoT 中樞會以 Avro 格式寫入內容，而其中同時包含訊息本文屬性和訊息屬性。 IoT 中樞僅支援以 Avro 資料格式將資料寫入至 Blob 儲存體，而此格式不會用於任何其他端點。 如需詳細資訊，請參閱有關使用 Azure 儲存體容器的文章。 雖然 Avro 格式很適合用來保存資料和訊息，但難以用來查詢資料。 相較之下，JSON 或 CSV 格式則非常適合用來查詢資料。

<!-- https://review.docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-messages-d2c?branch=pr-en-us-51566#azure-blob-storage  NEW LINK FOR 'WHEN USING STORAGE CONTAINERS' -->

若要因應非關聯式巨量資料的需求和格式並克服這項挑戰，您可以使用多種巨量資料模式來轉換和調整資料。 Azure Data Lake Analytics 即為其中一種模式，屬於「依查詢量付費」模式，這是此文章的焦點。 雖然您可以在 Hadoop 或其他解決方案中輕鬆地執行查詢，但對於這種「依查詢量付費」的方法，使用 Data Lake Analytics 通常會適合得多。 

U-SQL 中的 Avro 有一個「擷取程式」。 如需詳細資訊，請參閱 [U-SQL Avro 範例](https://github.com/Azure/usql/tree/master/Examples/AvroExamples)。

## <a name="query-and-export-avro-data-to-a-csv-file"></a>查詢 Avro 資料並將其匯出至 CSV 檔案
在本節中，您會查詢 Avro 資料，並將其匯出至 Azure Blob 儲存體中的 CSV 檔案，但您也可以輕鬆地將資料放置在其他存放庫或資料存放區。

1. 設定 Azure IoT 中樞，使其使用訊息本文中的屬性選取訊息，將資料路由至 Azure Blob 儲存體端點。

   ![[自訂端點] 區段](./media/iot-hub-query-avro-data/query-avro-data-1a.png)

   ![路由規則](./media/iot-hub-query-avro-data/query-avro-data-1b.png)

   如需設定路由與自訂端點的詳細資訊，請參閱 [IoT 中樞的訊息路由](iot-hub-create-through-portal.md#message-routing-for-an-iot-hub)。

2. 確定您的裝置具有編碼、內容類型和屬性或訊息本文中所需的資料，如產品文件所說明。 在 Device Explorer 中檢視這些屬性時 (如此處所示)，您可以確認這些屬性是否已正確設定。

   ![[事件中樞資料] 窗格](./media/iot-hub-query-avro-data/query-avro-data-2.png)

3. 設定 Azure Data Lake Store 執行個體和 Data Lake Analytics 執行個體。 Azure IoT 中樞不會路由至 Data Lake Store 執行個體，但 Data Lake Analytics 執行個體需要一個。

   ![Data Lake Store 和 Data Lake Analytics 執行個體](./media/iot-hub-query-avro-data/query-avro-data-3.png)

4. 在 Data Lake Analytics 中，將 Azure Blob 儲存體設定為另一個存放區，即 Azure IoT 中樞將資料路由到的相同 Blob 儲存體。

   ![[資料來源] 窗格](./media/iot-hub-query-avro-data/query-avro-data-4.png)
 
5. 如 [U-SQL Avro 範例](https://github.com/Azure/usql/tree/master/Examples/AvroExamples)所討論，您需要四個 DLL 檔案。 請將這些檔案上傳至 Data Lake Store 執行個體中的位置。

   ![四個已上傳的 DLL 檔案](./media/iot-hub-query-avro-data/query-avro-data-5.png)

6. 在 Visual Studio 中，建立 U-SQL 專案。
 
   !建立 U-SQL 專案](./media/iot-hub-query-avro-data/query-avro-data-6.png)

7. 將下列指令碼的內容貼到新建立的檔案中。 修改醒目提示的三個部分：Data Lake Analytics 帳戶、相關 DLL 檔案路徑，和儲存體帳戶的正確路徑。
    
   ![要修改的三個部分](./media/iot-hub-query-avro-data/query-avro-data-7a.png)

   簡易輸出至 CSV 檔案的實際 U-SQL 指令碼：
    
    ```sql
        DROP ASSEMBLY IF EXISTS [Avro];
        CREATE ASSEMBLY [Avro] FROM @"/Assemblies/Avro/Avro.dll";
        DROP ASSEMBLY IF EXISTS [Microsoft.Analytics.Samples.Formats];
        CREATE ASSEMBLY [Microsoft.Analytics.Samples.Formats] FROM @"/Assemblies/Avro/Microsoft.Analytics.Samples.Formats.dll";
        DROP ASSEMBLY IF EXISTS [Newtonsoft.Json];
        CREATE ASSEMBLY [Newtonsoft.Json] FROM @"/Assemblies/Avro/Newtonsoft.Json.dll";
        DROP ASSEMBLY IF EXISTS [log4net];
        CREATE ASSEMBLY [log4net] FROM @"/Assemblies/Avro/log4net.dll";

        REFERENCE ASSEMBLY [Newtonsoft.Json];
        REFERENCE ASSEMBLY [log4net];
        REFERENCE ASSEMBLY [Avro];
        REFERENCE ASSEMBLY [Microsoft.Analytics.Samples.Formats];

        // Blob container storage account filenames, with any path
        DECLARE @input_file string = @"wasb://hottubrawdata@kevinsayazstorage/kevinsayIoT/{*}/{*}/{*}/{*}/{*}/{*}";
        DECLARE @output_file string = @"/output/output.csv";

        @rs =
        EXTRACT
        EnqueuedTimeUtc string,
        Body byte[]
        FROM @input_file

        USING new Microsoft.Analytics.Samples.Formats.ApacheAvro.AvroExtractor(@"
        {
            ""type"":""record"",
            ""name"":""Message"",
            ""namespace"":""Microsoft.Azure.Devices"",
            ""fields"":
           [{
                ""name"":""EnqueuedTimeUtc"",
                ""type"":""string""
            },
            {
                ""name"":""Properties"",
                ""type"":
                {
                    ""type"":""map"",
                    ""values"":""string""
                }
            },
            {
                ""name"":""SystemProperties"",
                ""type"":
                {
                    ""type"":""map"",
                    ""values"":""string""
                }
            },
            {
                ""name"":""Body"",
                ""type"":[""null"",""bytes""]
            }]
        }"
        );

        @cnt =
        SELECT EnqueuedTimeUtc AS time, Encoding.UTF8.GetString(Body) AS jsonmessage
        FROM @rs;

        OUTPUT @cnt TO @output_file USING Outputters.Text(); 
    ```    

    Data Lake Analytics 花了五分鐘的時間執行下列指令碼，此指令碼限制在 10 個分析單位，並已處理 177 個檔案。 下圖所示的 CSV 檔案輸出中會顯示結果：
    
    ![輸出至 CSV 檔案的結果](./media/iot-hub-query-avro-data/query-avro-data-7b.png)

    ![輸出轉換為 CSV 檔案](./media/iot-hub-query-avro-data/query-avro-data-7c.png)

    若要剖析 JSON，請接著執行步驟 8。
    
8. 大部分的 IoT 訊息皆為 JSON 檔案格式。 藉由新增以下幾行，即可將訊息剖析至 JSON 檔案，以供新增 WHERE 子句，並只輸出所需的資料。

    ```sql
       @jsonify = 
         SELECT Microsoft.Analytics.Samples.Formats.Json.JsonFunctions.JsonTuple(Encoding.UTF8.GetString(Body)) 
           AS message FROM @rs;
    
        /*
        @cnt =
            SELECT EnqueuedTimeUtc AS time, Encoding.UTF8.GetString(Body) AS jsonmessage
            FROM @rs;
        
        OUTPUT @cnt TO @output_file USING Outputters.Text();
        */
        
        @cnt =
            SELECT message["message"] AS iotmessage,
                   message["event"] AS msgevent,
                   message["object"] AS msgobject,
                   message["status"] AS msgstatus,
                   message["host"] AS msghost
            FROM @jsonify;
            
        OUTPUT @cnt TO @output_file USING Outputters.Text();
    ```

    輸出會為 `SELECT` 命令中的每個項目顯示一個資料行。 
    
    ![為每個項目顯示一個資料行的輸出](./media/iot-hub-query-avro-data/query-avro-data-8.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何查詢 Avro 資料，以有效率地將訊息從 Azure IoT 中樞路由至 Azure 服務。

如需會使用 IoT 中樞的完整端對端解決方案範例，請參閱 [Azure IoT 解決方案加速器文件](../iot-accelerators/index.yml)。

若要深入了解如何使用 IoT 中樞開發解決方案，請參閱 [IoT 中樞開發人員指南](iot-hub-devguide.md)。

若要深入了解 IoT 中樞的訊息路由，請參閱[使用 IoT 中樞傳送和接收訊息](iot-hub-devguide-messaging.md)。
