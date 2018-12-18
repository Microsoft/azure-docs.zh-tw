---
title: 將裝置中的檔案上傳至使用 Python 的 Azure IoT 中樞 | Microsoft Docs
description: 如何使用適用於 Python 的 Azure IoT 裝置 SDK 將檔案從裝置上傳至雲端。 上傳的檔案會儲存在 Azure 儲存體 blob 容器中。
author: kgremban
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 03/05/2018
ms.author: kgremban
ms.openlocfilehash: eb5e7ce608f434bd880baae4d6780dd5038099f8
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39187211"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub"></a>使用 IoT 中樞將檔案從裝置上傳至雲端

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

本教學課程會遵循如何使用 [IoT 中樞的檔案上傳功能](iot-hub-devguide-file-upload.md)，將檔案上傳到 [Azure blob 儲存體](../storage/index.yml)。 本教學課程說明如何：

- 安全地提供儲存體容器來上傳檔案。
- 使用 Python 用戶端透過 IoT 中樞上傳檔案。

[開始使用 IoT 中樞](quickstart-send-telemetry-node.md)教學課程示範「IoT 中樞」的基本裝置到雲端傳訊功能。 不過，在某些情況下，您無法輕易地將裝置傳送的資料對應到 IoT 中樞接受且相對較小的裝置到雲端訊息。 當您需要從裝置上傳檔案時，您仍然可以使用安全可靠的 IoT 中樞。

> [!NOTE]
> IoT 中樞 Python SDK 目前僅支援上傳諸如 **.txt** 檔案等以字元為基礎的檔案。

在本教學課程結尾，您會執行 Python 主控台應用程式：

* **FileUpload.py**，其會使用 Python 裝置 SDK 將檔案上傳至儲存體。

> [!NOTE]
> IoT 中樞透過 Azure IoT 裝置 SDK 來支援許多裝置平台和語言 (包括 C、.NET、Javascript、Python 和 Java)。 如需如何將您的裝置連接到 Azure IoT 中樞的逐步指示，請參閱 [Azure IoT 開發人員中心]。

若要完成此教學課程，您需要下列項目：

* [Python 2.x 或 3.x][lnk-python-download]。 請務必使用安裝程式所需的 32 位元或 64 位元安裝。 在安裝期間出現系統提示時，務必將 Python 新增至平台特有的環境變數。 如果您使用 Python 2.x，則可能需要[安裝或升級 *pip*(Python 套件管理系統][lnk-install-pip])。
* 如果您使用 Windows 作業系統，則 [Visual c + + 可轉散發套件][lnk-visual-c-redist] 允許使用 Python 的原生 DLL。
* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶](http://azure.microsoft.com/pricing/free-trial/)。)


[!INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity-portal.md)]

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]


## <a name="upload-a-file-from-a-device-app"></a>從裝置應用程式上傳檔案

在本節中，您可以建立裝置應用程式來將檔案上傳到 IoT 中樞。

1. 在命令提示字元上執行下列命令，以安裝 **azure-iothub-device-client** 套件：

    ```cmd/sh
    pip install azure-iothub-device-client
    ```

1. 使用文字編輯器，在工作資料夾中建立 **FileUpload.py** 檔案。

1. 在 **FileUpload.py** 檔案開頭處新增下列 `import` 陳述式和變數。 將 `deviceConnectionString` 取代為您 IoT 中樞裝置的連接字串：

    ```python
    import time
    import sys
    import iothub_client
    import os
    from iothub_client import IoTHubClient, IoTHubClientError, IoTHubTransportProvider, IoTHubClientResult, IoTHubError

    CONNECTION_STRING = "[Device Connection String]"
    PROTOCOL = IoTHubTransportProvider.HTTP

    PATHTOFILE = "[Full path to file]"
    FILENAME = "[File name on storage after upload]"
    ```

1. 建立 **upload_blob** 函式的回呼：

    ```python
    def blob_upload_conf_callback(result, user_context):
        if str(result) == 'OK':
            print ( "...file uploaded successfully." )
        else:
            print ( "...file upload callback returned: " + str(result) )
    ```

1. 新增下列程式碼以連線用戶端並上傳檔案。 另包含 `main` 常式：

    ```python
    def iothub_file_upload_sample_run():
        try:
            print ( "IoT Hub file upload sample, press Ctrl-C to exit" )

            client = IoTHubClient(CONNECTION_STRING, PROTOCOL)

            f = open(PATHTOFILE, "r")
            content = f.read()

            client.upload_blob_async(FILENAME, content, len(content), blob_upload_conf_callback, 0)

            print ( "" )
            print ( "File upload initiated..." )

            while True:
                time.sleep(30)

        except IoTHubError as iothub_error:
            print ( "Unexpected error %s from IoTHub" % iothub_error )
            return
        except KeyboardInterrupt:
            print ( "IoTHubClient sample stopped" )
        except:
            print ( "generic error" )

    if __name__ == '__main__':
        print ( "Simulating a file upload using the Azure IoT Hub Device SDK for Python" )
        print ( "    Protocol %s" % PROTOCOL )
        print ( "    Connection string=%s" % CONNECTION_STRING )

        iothub_file_upload_sample_run()
    ```

1. 儲存並關閉 **UploadFile.py** 檔案。

1. 將範例文字檔複製到工作資料夾，並將它重新命名為 `sample.txt`。

    > [!NOTE]
    > IoT 中樞 Python SDK 目前僅支援上傳諸如 **.txt** 檔案等以字元為基礎的檔案。


## <a name="run-the-application"></a>執行應用程式

現在您已經準備好執行應用程式。

1. 在工作資料夾的命令提示字元中，執行下列命令：

    ```cmd/sh
    python FileUpload.py
    ```

1. 下列螢幕擷取畫面顯示 **FileUpload** 應用程式的輸出：

    ![simulated-device 應用程式的輸出](./media/iot-hub-python-python-file-upload/1.png)

1. 您可以使用入口網站檢視您所設定之儲存體容器中的上傳檔案：

    ![上傳的檔案](./media/iot-hub-python-python-file-upload/2.png)


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已學到如何使用 IoT 中樞的檔案上傳功能來簡化從裝置上傳檔案。 您可以利用下列文章繼續探索 IoT 中樞功能和案例：

* [以程式設計方式建立 IoT 中樞][lnk-create-hub]
* [C SDK 簡介][lnk-c-sdk]
* [Azure IoT SDK][lnk-sdks]

<!-- Links -->
[Azure IoT 開發人員中心]: http://azure.microsoft.com/develop/iot

[lnk-create-hub]: iot-hub-rm-template-powershell.md
[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md
[lnk-python-download]: https://www.python.org/downloads/
[lnk-visual-c-redist]: http://www.microsoft.com/download/confirmation.aspx?id=48145
[lnk-install-pip]: https://pip.pypa.io/en/stable/installing/
