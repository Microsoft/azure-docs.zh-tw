---
title: 以 C 將 Linux 裝置佈建到遠端監視 - Azure | Microsoft Docs
description: 描述如何使用以 Linux 上執行的 C 所編寫的應用程式，將裝置連線到遠端監視解決方案加速器。
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 03/14/2018
ms.author: dobett
ms.openlocfilehash: 5d7d6522dc663f13ce40cc638ba90ac4043d435c
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38611407"
---
# <a name="connect-your-device-to-the-remote-monitoring-solution-accelerator-linux"></a>將裝置連線到遠端監視解決方案加速器 (Linux)

[!INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

本教學課程示範如何將實體裝置連線到遠端監視解決方案加速器。

## <a name="create-a-c-client-project-on-linux"></a>在 Linux 上建立 C 用戶端專案

如同大部分在受條件約束裝置上執行的內嵌應用程式，裝置應用程式的用戶端程式碼是以 C 撰寫的。在此教學課程中，您要在執行 Ubuntu (Linux) 的電腦上建置應用程式。

若要完成這些步驟，您需要執行 Ubuntu 版本 15.04 或更新版本的裝置。 繼續之前，使用下列命令在 Ubuntu 裝置上安裝的必要條件封裝：

```sh
sudo apt-get install cmake gcc g++
```

### <a name="install-the-client-libraries-on-your-device"></a>在裝置上安裝用戶端程式庫

Azure IoT 中樞用戶端程式庫可當成封裝來使用，而您可以使用 **apt get** 命令在 Ubuntu 裝置上安裝這類封裝。 完成下列步驟來安裝套件，其中包含 Ubuntu 電腦上的 IoT 中樞用戶端程式庫和標頭檔：

1. 在殼層中，將 AzureIoT 儲存機制新增至電腦：

    ```sh
    sudo add-apt-repository ppa:aziotsdklinux/ppa-azureiot
    sudo apt-get update
    ```

1. 安裝 azure-iot-sdk-c-dev 封裝

    ```sh
    sudo apt-get install -y azure-iot-sdk-c-dev
    ```

### <a name="install-the-parson-json-parser"></a>安裝 Parson JSON 剖析器

IoT 中樞用戶端程式庫會使用 Parson JSON 剖析器來剖析訊息承載。 在您電腦上的適當資料夾中，使用下列命令複製 Parson GitHub 儲存機制︰

```sh
git clone https://github.com/kgabis/parson.git
```

### <a name="prepare-your-project"></a>準備您的專案

在 Ubuntu 電腦上，建立稱為 `remote_monitoring` 的資料夾。 在 `remote_monitoring` 資料夾中：

- 建立 `main.c`、`remote_monitoring.c`、`remote_monitoring.h` 和 `CMakeLists.txt` 四個檔案。
- 建立一個名為 `parson` 的資料夾。

將 `parson.c` 和 `parson.h` 檔案從 Parson 存放庫的本機複本複製到 `remote_monitoring/parson` 資料夾。

在文字編輯器中開啟 `remote_monitoring.c` 檔案。 新增下列 `#include` 陳述式：

```c
#include "iothubtransportmqtt.h"
#include "schemalib.h"
#include "iothub_client.h"
#include "serializer_devicetwin.h"
#include "schemaserializer.h"
#include "azure_c_shared_utility/threadapi.h"
#include "azure_c_shared_utility/platform.h"
#include <string.h>
```

[!INCLUDE [iot-suite-connecting-code](../../includes/iot-suite-connecting-code.md)]

## <a name="add-code-to-run-the-app"></a>新增程式碼以執行應用程式

在文字編輯器中開啟 `remote_monitoring.h` 檔案。 新增下列程式碼：

```c
void remote_monitoring_run(void);
```

在文字編輯器中開啟 `main.c` 檔案。 新增下列程式碼：

```c
#include "remote_monitoring.h"

int main(void)
{
  remote_monitoring_run();

  return 0;
}
```

## <a name="build-and-run-the-application"></a>建置並執行應用程式

下列步驟說明如何使用 *CMake* 建置用戶端應用程式。

1. 在文字編輯器中，開啟 `remote_monitoring` 資料夾中的 **CMakeLists.txt** 檔案。

1. 新增下列指示來定義如何建置用戶端應用程式：

    ```cmake
    macro(compileAsC99)
      if (CMAKE_VERSION VERSION_LESS "3.1")
        if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
          set (CMAKE_C_FLAGS "--std=c99 ${CMAKE_C_FLAGS}")
          set (CMAKE_CXX_FLAGS "--std=c++11 ${CMAKE_CXX_FLAGS}")
        endif()
      else()
        set (CMAKE_C_STANDARD 99)
        set (CMAKE_CXX_STANDARD 11)
      endif()
    endmacro(compileAsC99)

    cmake_minimum_required(VERSION 2.8.11)
    compileAsC99()

    set(AZUREIOT_INC_FOLDER "${CMAKE_SOURCE_DIR}" "${CMAKE_SOURCE_DIR}/parson" "/usr/include/azureiot" "/usr/include/azureiot/inc")

    include_directories(${AZUREIOT_INC_FOLDER})

    set(sample_application_c_files
        ./parson/parson.c
        ./remote_monitoring.c
        ./main.c
    )

    set(sample_application_h_files
        ./parson/parson.h
        ./remote_monitoring.h
    )

    add_executable(sample_app ${sample_application_c_files} ${sample_application_h_files})

    target_link_libraries(sample_app
        serializer
        iothub_client
        iothub_client_mqtt_transport
        aziotsharedutil
        umqtt
        pthread
        curl
        ssl
        crypto
        m
    )
    ```

1. 在 `remote_monitoring` 資料夾中，建立資料夾來儲存 CMake 產生的 make 檔案。 然後執行 **cmake** 和 **make** 命令，如下所示：

    ```sh
    mkdir cmake
    cd cmake
    cmake ../
    make
    ```

1. 執行用戶端應用程式，並將遙測傳送至 IoT 中樞：

    ```sh
    ./sample_app
    ```

[!INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]
