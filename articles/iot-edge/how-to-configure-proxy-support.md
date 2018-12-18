---
title: 針對網路 Proxy 設定 Azure IoT Edge 裝置 | Microsoft Docs
description: 如何設定 Azure IoT Edge 執行階段及任何網際網路對應的 IoT Edge 模組，以透過 Proxy 伺服器進行通訊。
author: kgremban
manager: ''
ms.author: kgremban
ms.date: 09/24/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 6e6a1d2f758cabca41ac405a01de1f0d8bfd0a7b
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47037451"
---
# <a name="configure-an-iot-edge-device-to-communicate-through-a-proxy-server"></a>設定 IoT Edge 裝置以透過 Proxy 伺服器進行通訊

IoT Edge 裝置會傳送 HTTPS 要求以和 IoT 中樞通訊。 如果您的裝置是連線至使用 Proxy 伺服器的網路，便必須設定 IoT Edge 執行階段以透過伺服器進行通訊。 如果 IoT Edge 模組做出沒有透過 Edge 中樞進行路由的 HTTP 或 HTTPS 要求，則 Proxy 伺服器也可能會影響這些個別的 IoT Edge 模組。 

設定 IoT Edge 裝置以搭配 Proxy 伺服器運作的基本步驟如下： 

1. 在裝置上安裝 IoT Edge 執行階段。 
2. 設定您裝置上的 Docker 精靈及 IoT Edge 精靈以使用 Proxy 伺服器。
3. 設定您裝置上 config.yaml 中的 edgeAgent 屬性。
4. 設定部署資訊清單中 IoT Edge 執行階段和其他 IoT Edge 模組的環境變數。 

## <a name="install-the-runtime"></a>安裝執行階段

若您正於 Linux 裝置上安裝 IoT Edge 執行階段，請設定套件管理員以 Proxy 伺服器存取安裝套件。 例如，[設定 apt-get 以使用 http-proxy](https://help.ubuntu.com/community/AptGet/Howto/#Setting_up_apt-get_to_use_a_http-proxy) \(英文\)。 設定您的套件管理員之後，請依照[在 Linux (ARM32v7/armhf) 上安裝 Azure IoT Edge 執行階段](how-to-install-iot-edge-linux-arm.md)或[在 Linux (x64) 上安裝 Azure IoT Edge 執行階段](how-to-install-iot-edge-linux.md)中的指示執行。 

若您正於 Windows 裝置上安裝 IoT Edge 執行階段，您必須透過 Proxy 伺服器存取安裝套件。 您可以在 Windows 設定中設定 Proxy 資訊，或直接在安裝指令碼中包含您的 Proxy 資訊。 下列 PowerShell 指令碼是使用 `-proxy` 引數的 Wndows 安裝範例：

```powershell
. {Invoke-WebRequest -proxy <proxy URL> -useb aka.ms/iotedge-win} | Invoke-Expression; `
Install-SecurityDaemon -Manual -ContainerOs Windows
```

如需詳細資訊與安裝選項，請參閱[在 Windows 上安裝 Azure IoT Edge 執行階段以搭配 Windows 容器使用](how-to-install-iot-edge-windows-with-windows.md)或[在 Windows 上安裝 Azure IoT Edge 執行階段以搭配 Linux 容器使用](how-to-install-iot-edge-windows-with-linux.md)。

安裝 IoT Edge 執行階段之後，請使用下一節中的指示以您的 Proxy 資訊設定它。 

## <a name="configure-the-daemons"></a>設定精靈

您需對在 IoT Edge 裝置上執行的 Docker 和 IoT Edge 精靈進行設定，以使用 Proxy 伺服器。 Docker 精靈會發出 Web 要求，以從容器登錄提取容器映像。 IoT Edge 精靈會發出 Web 要求，以和 IoT 中樞通訊。

### <a name="docker-daemon"></a>Docker 精靈

請參考 Docker 文件以搭配環境變數設定 Docker 精靈。 大部分的容器登錄 (包括 Docker Hub 和 Azure Container Registry) 皆支援 HTTPS 要求，因此您應該設定 **HTTPS_PROXY**變數。 如果您是從不支援傳輸層安全性 (TLS) 的登錄提取映像，則應該設定 **HTTP_PROXY**。 

請選擇適用於您 Docker 版本的文章： 

* [Docker](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)
* [Docker for Windows](https://docs.docker.com/docker-for-windows/#proxies)

### <a name="iot-edge-daemon"></a>IoT Edge 精靈

IoT Edge 精靈的設定方式與 Docker 精靈類似。 IoT Edge 傳送至 IoT 中樞的所有要求皆是使用 HTTPS。 請根據您的作業系統，使用下列相對應的步驟來針對服務設定環境變數。 

#### <a name="linux"></a>Linux

在終端機中開啟編輯器以設定 IoT 精靈。 

```bash
sudo systemctl edit iotedge
```

輸入下列文字，並將 **\<proxy URL>** 取代為您 Proxy 伺服器的位址和連接埠。 然後儲存並結束。 

```text
[Service]
Environment="https_proxy=<proxy URL>"
```

重新整理服務管理員以取得適用於 iotedge 的新設定。

```bash
sudo systemctl daemon-reload
```

重新啟動 IoT Edge 來讓變更生效。

```bash
sudo systemctl restart iotedge
```

確認您的環境變數已經建立，並已載入新的設定。 

```bash
systemctl show --property=Environment iotedge
```

#### <a name="windows"></a>Windows

以系統管理員身分開啟 PowerShell 視窗，並執行下列命令以搭配新的環境變數編輯登錄。 將 **\<proxy url>** 取代為您 Proxy 伺服器的位址和連接埠。 

```powershell
reg add HKLM\SYSTEM\CurrentControlSet\Services\iotedge /v Environment /t REG_MULTI_SZ /d https_proxy=<proxy URL>
```

重新啟動 IoT Edge 來讓變更生效。

```powershell
Restart-Service iotedge
```

## <a name="configure-the-edge-agent"></a>設定 Edge 代理程式

Edge 代理程式是在所有 IoT Edge 裝置上皆應第一個啟動的模組。 它會根據 IoT Edge config.yaml 檔案中的資訊首次啟動。 Edge 代理程式接著會連線至 IoT 中樞以擷取部署資訊清單，該資訊清單會宣告有哪些其他模組應部署於裝置上。

在您的 IoT Edge 裝置上開啟 config.yaml 檔案。 在 Linux 系統上，此檔案位於 **/etc/iotedge/config.yaml**。 在 Windows 系統上，此檔案位於 **C:\ProgramData\iotedge\config.yaml**。 該設定檔是受保護的，因此您需要系統管理權限以開啟它。 在 Linux 系統上，這表示您需要先使用 `sudo` 命令，才能在您偏好的文字編輯器中開啟該檔案。 在 Windows 上，這表示您需要先以系統管理員身分開啟如記事本之類的文字編輯器，然後再開啟該檔案。 

在 config.yaml 檔案中，找到 **Edge Agent module spec** 區段。 Edge 代理程式定義包含 **env** 參數，可供您於該處新增環境變數。 

![edgeAgent 定義](./media/how-to-configure-proxy-support/edgeagent-unedited.png)

移除作為 env 參數之預留位置的大括號，然後將新的變數加入至新的一行上。 請記得 YAML 中的縮排為兩個空格。 

```yaml
https_proxy: "<proxy URL>"
```

IoT Edge 執行階段預設會使用 AMQP 來與 IoT 中樞通訊。 某些 Proxy 伺服器會封鎖 AMQP 連接埠。 針對那種情況，您也必須設定 edgeAgent 以使用 AMQP，而非 WebSocket。 新增第二個環境變數。

```yaml
UpstreamProtocol: "AmqpWs"
```

![具環境變數的 edgeAgent 定義](./media/how-to-configure-proxy-support/edgeagent-edited.png)

儲存對 config.yaml 所做的變更，然後關閉編輯器。 重新啟動 IoT Edge 來讓變更生效。 

* Linux： 

   ```bash
   sudo systemctl restart iotedge
   ```

* Windows:

   ```powershell
   Restart-Service iotedge
   ```

## <a name="configure-deployment-manifests"></a>設定部署資訊清單  

在您的 IoT Edge 裝置已設定為能搭配您的 Proxy 伺服器運作之後，您也需要在所有未來的部署資訊清單中宣告環境變數。 兩個執行階段模組 (edgeAgent 和 edgeHub) 皆一律應設定 Proxy 伺服器，以維持與 IoT 中樞之間的通訊。 您可以設定任何 IoT Edge 模組以透過 Proxy 伺服器進行通訊，但這對所有會透過 edgeHub 路由其訊息的模組，或是僅會與裝置上其他模組進行通訊的模組而言並不是必要的。 

您可以使用 Azure 入口網站建立部署資訊清單，或是編輯 JSON 檔案來手動建立。 

### <a name="azure-portal"></a>Azure 入口網站

當您使用 [設定模組] 精靈來針對 IoT Edge 裝置建立部署時，每個模組都會有 [環境變數] 區段可供您用來設定 Proxy 伺服器連線。 

若要設定 Edge 代理程式和 Edge 中樞模組，請在精靈的第一個步驟中選取 [設定進階 Edge 執行階段設定]。 

![設定進階 Edge 執行階段設定](./media/how-to-configure-proxy-support/configure-runtime.png)

將 **https_proxy** 環境變數同時新增至 Edge 代理程式和 Edge 中樞模組定義之中。 如果您已在 IoT Edge 裝置上的 config.yaml 檔案中包含 **UpstreamProtocol** 環境變數，請同時將該環境變數新增至 Edge 代理程式模組定義。 

![設定環境變數](./media/how-to-configure-proxy-support/edgehub-environmentvar.png)

所有其他新增至部署資訊清單的模組都會遵循相同的模式。 在您設定模組名稱和映像的頁面中，會有一個環境變數區段。

### <a name="json-deployment-manifest-files"></a>JSON 部署資訊清單檔案

如果您是使用 Visual Studio Code 中的範本，或是手動建立 JSON 檔案來針對 IoT Edge 裝置建立部署，您可以將環境變數直接新增至每個模組定義中。 

使用下列 JSON 格式： 

```json
"env": {
    "https_proxy": {
        "value": "<proxy URL>"
    }
}
```

在包含環境變數的情況下，您的模組定義看起來應該會和下列 edgeHub 範例類似：

```json
"edgeHub": {
    "type": "docker",
    "settings": {
        "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
        "createOptions": ""
    },
    "env": {
        "https_proxy": {
            "value": "https://proxy.example.com:3128"
        }
    },
    "status": "running",
    "restartPolicy": "always"
}
```

如果您已在 IoT Edge 裝置上的 config.yaml 檔案中包含 **UpstreamProtocol** 環境變數，請同時將該環境變數新增至 Edge 代理程式模組定義。 

```json
"env": {
    "https_proxy": {
        "value": "<proxy URL"
    },
    "UpstreamProtocol": {
        "value": "AmqpWs"
    }
}
```

## <a name="next-steps"></a>後續步驟

深入了解 [IoT Edge 執行階段](iot-edge-runtime.md)的角色。

使用 [Azure IoT Edge 的常見問題和解決方案](troubleshoot.md)來對安裝和設定錯誤進行疑難排解

