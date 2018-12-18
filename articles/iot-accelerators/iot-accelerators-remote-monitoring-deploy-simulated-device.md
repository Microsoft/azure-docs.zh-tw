---
title: IoT 部署自訂模擬裝置 - Azure |Microsoft Docs
description: 本操作指南說明如何將自訂模擬裝置部署到遠端監視解決方案加速器。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 08/15/2018
ms.topic: conceptual
ms.openlocfilehash: f073637810e9ed1acdf37b0e541ca3f1d518de2a
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43346691"
---
# <a name="deploy-a-new-simulated-device"></a>部署新的模擬裝置

遠端監視和裝置模擬解決方案加速器都可讓您定義自己的模擬裝置。 本文說明如何將自訂的冷卻器裝置類型和新的燈泡裝置類型部署到遠端監視解決方案加速器。

本文中的步驟假設您已完成[建立及測試新的模擬裝置](iot-accelerators-remote-monitoring-create-simulated-device.md)操作指南指南，並且具有可定義自訂冷卻器和新的燈泡裝置類型的檔案。

本操作指南中的步驟說明如何：

1. 使用 SSH 來存取裝載遠端監視解決方案加速器的虛擬機器檔案系統。

1. 將 Docker 設定為從 Docker 容器以外的位置載入裝置模型。

1. 使用自訂裝置模型檔案來執行遠端監視解決方案加速器。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

若要完成此操作指南中的步驟，您必須具備有效的 Azure 訂用帳戶。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

若要遵循本操作說明指南，您需要：

- [遠端監視解決方案加速器](https://www.azureiotsolutions.com/Accelerators#solutions/types/RM2)的已部署執行個體。
- 用於執行 `ssh` 和 `scp` 命令的本機 **bash** 殼層。 在 Windows 上，輕鬆安裝 **bash** 的方法就是安裝 [git](https://git-scm.com/download/win)。
- 您的自訂裝置模型檔案，例如[建立及測試新的模擬裝置](iot-accelerators-remote-monitoring-create-simulated-device.md)中所述的檔案。

[!INCLUDE [iot-solution-accelerators-access-vm](../../includes/iot-solution-accelerators-access-vm.md)]

## <a name="configure-docker"></a>設定 Docker

在這一節中，您會將 Docker 設定為從虛擬機器 的 **/tmp/devicemodels** 資料夾 (而不是從 Docker 容器內部) 載入裝置模型檔案。 在本機電腦上的 **bash** 殼層中執行這一節中的命令：

在這一節中，您會將 Docker 設定為從虛擬機器 的 **/tmp/devicemodels** 資料夾 (而不是從 Docker 容器內部) 載入裝置模型檔案。 在本機電腦上的 **bash** 殼層中執行這一節中的命令：

1. 使用 SSH，從本機電腦連線到 Azure 中的虛擬機器。 下列命令假設虛擬機器 **vm vikxv** 的公用 IP 位址是 **104.41.128.108** -- 以上一節中虛擬機器的公用 IP 位址取代此值：

   ```sh
    ssh azureuser@104.41.128.108
    ```

    使用您在上一節中設定的密碼，遵循提示來登入虛擬機器。

1. 將裝置模擬服務設定為從容器外部載入裝置模型。 首先開啟 Docker 組態檔：

    ```sh
    sudo nano /app/docker-compose.yml
    ```

    找出 **devicesimulation** 容器的設定，並且編輯**磁碟區**設定，如下列程式碼片段所示：

    ```yml
    devicesimulation:
      image: azureiotpcs/device-simulation-dotnet:1.0.0
      networks:
        - default_net
      depends_on:
        - storageadapter
      environment:
        - PCS_IOTHUB_CONNSTRING
        - PCS_STORAGEADAPTER_WEBSERVICE_URL=http://storageadapter:9022/v1
        - PCS_AUTH_ISSUER
        - PCS_AUTH_AUDIENCE
        - PCS_AUTH_REQUIRED
        - PCS_CORS_WHITELIST
        - PCS_APPLICATION_SECRET
      # How one could mount custom device models
      volumes:
        - /tmp/devicemodels:/app/webservice/data/devicemodels:ro
    ```

    儲存變更。

1. 將容器中的現有裝置模型檔案複製到新位置。 首先，找出裝置模擬容器的容器識別碼：

    ```sh
    docker ps
    ```

    然後將裝置模型檔案複製到虛擬機器中的 **tmp** 資料夾。 下列命令假設容器識別碼為 c378d6878407 -- 以您的裝置模擬容器識別碼取代此值：

    ```sh
    docker cp c378d6878407:/app/webservice/data/devicemodels /tmp
    ```

    讓 **bash** 視窗中的 SSH 工作階段保持開啟狀態。

1. 將自訂裝置模型檔案複製到虛擬機器中。 您已建立自訂裝置模型的機器上，於另一個 **bash** 殼層中執行此命令。 首先，瀏覽至包含您的裝置模型 JSON 檔案的本機資料夾。 下列命令假設虛擬機器的公用 IP 位址是 **104.41.128.108** -- 以您虛擬機器的公用 IP 位址取代此值。 在系統提示時輸入您的虛擬機器密碼：

    ```sh
    scp *json azureuser@104.41.128.108:/tmp/devicemodels
    cd scripts
    scp *js azureuser@104.41.128.108:/tmp/devicemodels/scripts
    ```

1. 重新啟動裝置模擬 Docker 容器，以使用新的裝置模型。 在 **bash** 殼層中執行下列命令 (搭配對虛擬機器開啟的 SSH 工作階段)：

    ```sh
    sudo /app/start.sh
    ```

    如果您要查看執行中 Docker 容器的狀態及其容器識別碼，請使用下列命令：

    ```sh
    docker ps
    ```

    如果您要查看來自裝置模擬容器的記錄，請執行下列命令。 以您裝置模擬容器的識別碼取代此容器識別碼：

    ```sh
    docker logs -f 5d3f3e78822e
    ```

## <a name="run-simulation"></a>執行模擬

您現在可以在遠端監視解決方案中使用自訂裝置模型：

1. 從 [Microsoft Azure IoT 解決方案加速器](https://www.azureiotsolutions.com/Accelerators#dashboard)啟動您的遠端監視儀表板。

1. 使用 [裝置] 頁面來新增模擬裝置。 當您新增模擬裝置時，您的新裝置模型可以供您選擇。

1. 您可以使用儀表板來檢視裝置遙測資料並呼叫裝置方法。

## <a name="clean-up-resources"></a>清除資源

如果您打算進一步探索，請讓遠端監視解決方案加速器維持部署。

如果您不再需要解決方案加速器，可加以選取，然後按一下 [刪除解決方案]，從[已佈建的解決方案](https://www.azureiotsolutions.com/Accelerators#dashboard)頁面中加以刪除。

## <a name="next-steps"></a>後續步驟

本指南已說明如何將自訂裝置模型部署到遠端監視解決方案加速器。 建議的下一步是了解如何[將實體裝置連線到遠端監視解決方案](iot-accelerators-connecting-devices-node.md)。
