---
title: 部署連線處理站閘道 - Azure | Microsoft Docs
description: 如何在 Windows 或 Linux 上部署閘道，以啟用與連線處理站解決方案加速器的連線。
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 01/17/2018
ms.author: dobett
ms.openlocfilehash: 3a68a4a132302051b04b69cc794f5327a82f7639
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45604046"
---
# <a name="deploy-an-edge-gateway-for-the-connected-factory-solution-accelerator-on-windows-or-linux"></a>在 Windows 或 Linux 上部署連線處理站解決方案加速器的 Edge 閘道

您需要兩個軟體元件，以針對「連線處理站」解決方案加速器部署 Edge 閘道：

- *OPC Proxy* 會建立與連線處理站的連線。 然後 OPC Proxy 會等待在連線處理站解決方案入口網站中所執行整合式 OPC 瀏覽器的命令和控制訊息。

- 「OPC 發行者」會連線到現有的內部部署 OPC UA 伺服器，並將它們的遙測訊息轉送到連線處理站。 您可以使用[適用於 OPC UA 的 OPC 傳統配接器](https://github.com/OPCFoundation/UA-.NETStandard/blob/master/ComIOP/README.md)連線 OPC 傳統裝置。

這兩個元件都是開放程式碼，且以 GitHub 上的來源與 DockerHub 上 Docker 容器的方式提供：

| GitHub | DockerHub |
| ------ | --------- |
| [OPC 發行者](https://github.com/Azure/iot-edge-opc-publisher) | [OPC 發行者](https://hub.docker.com/r/microsoft/iot-edge-opc-publisher/)   |
| [OPC Proxy](https://github.com/Azure/iot-edge-opc-proxy)         | [OPC Proxy](https://hub.docker.com/r/microsoft/iot-edge-opc-proxy/) |

針對各個元件，您都不需要公開 IP 位址或閘道防火牆中的開放輸入連接埠。 OPC Proxy 和 OPC 發行者元件僅使用輸出連接埠 443。

本文中的步驟示範如何在 Windows 或 Linux 上使用 Docker 部署 Edge 閘道。 閘道會啟用與連線處理站解決方案加速器的連線。 您也可以使用沒有連線處理站的元件。

> [!NOTE]
> 這兩個元件可作為 [Azure IoT Edge](https://github.com/Azure/iot-edge) 的模組。

## <a name="choose-a-gateway-device"></a>選擇閘道裝置

如果您還沒有閘道裝置，Microsoft 建議您向合作夥伴購買商業閘道。 如需與連線處理站解決方案相容的閘道裝置清單，請造訪 [Azure IoT 裝置目錄](https://catalog.azureiotsuite.com/?q=opc)。 依照裝置隨附的指示來設定閘道。

或者，使用下列指示手動設定現有閘道裝置。

## <a name="install-and-configure-docker"></a>安裝及設定 Docker

在以 Windows 為基礎的閘道裝置上安裝 [Docker for Windows](https://www.docker.com/docker-windows)，或者使用套件管理員在以 Linux 為基礎的閘道裝置上安裝 Docker。

在 Docker for Windows 安裝期間，選取主機電腦上要與 Docker 共用的磁碟機。 下列螢幕擷取畫面顯示在您的 Windows 系統上共用 **D** 磁碟機，以允許從 Docker 容器內存取主機磁碟機：

![安裝 Docker for Windows](./media/iot-accelerators-connected-factory-gateway-deployment/image1.png)

> [!NOTE]
> 您也可以在安裝 Docker 之後從 [設定] 對話方塊執行這個步驟。 以滑鼠右鍵按一下 Windows 系統匣中的 [Docker] 圖示，然後選擇 [設定]。 如果已將主要的 Windows 更新部署到系統 (例如 Windows Fall Creators Update)，請將磁碟機取消共用，然後重新共用它們，以重新整理存取權限。

如果您使用 Linux，不需要額外設定就可以啟用檔案系統的存取權。

在 Windows 中，在您與 Docker 共用的磁碟機上建立資料夾，在 Linux 中於根檔案系統下建立資料夾。 本逐步解說將此資料夾稱為 `<SharedFolder>`。

當您在 Docker 命令中參考 `<SharedFolder>` 時，務必針對您的作業系統使用正確的語法。 以下是兩個範例，一個用於 Windows，一個用於 Linux：

- 如果您在 Windows 上使用資料夾 `D:\shared` 作為 `<SharedFolder>`，Docker 命令語法是 `d:/shared`。

- 如果您在 Linux 上使用資料夾 `/shared` 作為 `<SharedFolder>`，Docker 命令語法是 `/shared`。

如需詳細資訊，請參閱[使用磁碟區](https://docs.docker.com/engine/admin/volumes/volumes/) Docker 引擎參考。

## <a name="configure-the-opc-components"></a>設定 OPC 元件

安裝 OPC 元件之前，請完成下列步驟來準備您的環境：

1. 若要完成閘道部署，您在連線處理站部署中需要 IoT 中樞的 **iothubowner** 連接字串。 在 [Azure 入口網站](http://portal.azure.com/)中，瀏覽至您在部署連線處理站解決方案時所建立資源群組中的 IoT 中樞。 按一下 [共用存取原則] 以存取 **iothubowner** 連接字串︰

    ![找出 IoT 中樞連接字串](./media/iot-accelerators-connected-factory-gateway-deployment/image2.png)

    複製 [連接字串 – 主要金鑰] 的值。

1. 若要允許 Docker 容器之間的通訊，您需要使用者定義的橋接網路。 若要為您的容器建立橋接網路，請在命令提示字元執行下列命令：

    ```cmd/sh
    docker network create -d bridge iot_edge
    ```

    若要確認已建立 **iot_edge** 橋接網路，請執行下列命令：

    ```cmd/sh
    docker network ls
    ```

    您的 **iot_edge** 橋接網路包含在網路清單中。

若要執行 OPC 發行者，請在命令提示字元執行下列命令：

```cmd/sh
docker run --rm -it -v <SharedFolder>:/docker -v x509certstores:/root/.dotnet/corefx/cryptography/x509stores --network iot_edge --name publisher -h publisher -p 62222:62222 --add-host <OpcServerHostname>:<IpAddressOfOpcServerHostname> microsoft/iot-edge-opc-publisher:2.1.4 publisher "<IoTHubOwnerConnectionString>" --lf /docker/publisher.log.txt --as true --si 1 --ms 0 --tm true --vc true --di 30
```

- [OPC 發行者 GitHub](https://github.com/Azure/iot-edge-opc-publisher) 和 [Docker 執行參考](https://docs.docker.com/engine/reference/run/)提供詳細資訊：

  - 容器名稱前面指定的 Docker 命令列選項 (`microsoft/iot-edge-opc-publisher:2.1.4`)。
  - 容器名稱後面指定之 OPC 發行者命令列參數的意義 (`microsoft/iot-edge-opc-publisher:2.1.4`)。

- `<IoTHubOwnerConnectionString>` 是來自 Azure 入口網站的 **iothubowner** 共用存取原則連接字串。 您在上一個步驟中複製這個連接字串。 只有在第一次執行 OPC 發行者時需要此連接字串。 在後續的執行中應該省略它，因為它有安全性風險。

- 您使用的 `<SharedFolder>` 及其語法會在[安裝及設定 Docker](#install-and-configure-docker) 一節中說明。 OPC 發行者會使用 `<SharedFolder>`，來讀取和寫入 OPC 發行者設定檔、寫入記錄檔，並使這兩個檔案可以在容器之外使用。

- OPC 發行者會從 **publishednodes.json** 檔案讀取其設定，此檔案會從 `<SharedFolder>/docker` 資料夾讀取或寫入該資料夾。 此設定檔會定義 OPC 發行者應該訂閱指定 OPC UA 伺服器上的哪些 OPC UA 節點資料。 **publishednodes.json** 檔案的完整語法會在 GitHub 的 [OPC 發行者](https://github.com/Azure/iot-edge-opc-publisher)頁面上說明。 當您新增閘道時，請在資料夾中放置空的 **publishednodes.json**：

    ```json
    [
    ]
    ```

- 每當 OPC UA 伺服器通知 OPC 發行者有資料變更時，新值會傳送到 IoT 中樞。 根據批次設定，OPC 發行者可能會先累積資料，然後再將資料傳送到批次中的 IoT 中樞。

- Docker 不支援 NetBIOS 名稱解析，僅支援 DNS 名稱解析。 如果您在網路上沒有 DNS 伺服器，您可以使用先前命令列範例中顯示的因應措施。 先前的命令列範例使用 `--add-host` 參數以將項目新增至容器主機檔案。 此項目針對指定 `<OpcServerHostname>` 啟用主機名稱查閱，解析至指定 IP 位址 `<IpAddressOfOpcServerHostname>`。

- OPC UA 會使用 X.509 憑證進行驗證和加密。 您需要將 OPC 發行者憑證放置在您所連線的 OPC UA 伺服器，以確保其信任 OPC 發行者。 OPC 發行者憑證存放區位於 `<SharedFolder>/CertificateStores` 資料夾。 您可以在 `CertificateStores` 資料夾的 `trusted/certs` 資料夾中找到 OPC 發行者憑證。

  設定 OPC UA 伺服器的步驟取決於您使用的裝置。 這些步驟通常會記錄在 OPC UA 伺服器的使用者手冊中。

若要安裝 OPC Proxy，請在命令提示字元執行下列命令：

```cmd/sh
docker run -it --rm -v <SharedFolder>:/mapped --network iot_edge --name proxy --add-host <OpcServerHostname>:<IpAddressOfOpcServerHostname> microsoft/iot-edge-opc-proxy:1.0.4 -i -c "<IoTHubOwnerConnectionString>" -D /mapped/cs.db
```

您只需要在系統上執行安裝一次。

使用下列命令以執行 OPC Proxy：

```cmd/sh
docker run -it --rm -v <SharedFolder>:/mapped --network iot_edge --name proxy --add-host <OpcServerHostname>:<IpAddressOfOpcServerHostname> microsoft/iot-edge-opc-proxy:1.0.4 -D /mapped/cs.db
```

OPC Proxy 會在安裝期間儲存連接字串。 在後續的執行中應該省略連接字串，因為它有安全性風險。

## <a name="enable-your-gateway"></a>啟用您的閘道

完成下列步驟，以在連線處理站解決方案加速器中啟用閘道：

1. 當兩個元件都在執行時，瀏覽至連線處理站解決方案入口網站中的 [連線您自己的 OPC UA 伺服器] 頁面。 只有解決方案的系統管理員才能使用此頁面。 輸入發行者端點 URL (opc.tcp://publisher:62222)，並按一下 [連線]。

1. 建立連線處理站入口網站與 OPC 發行者之間的信任關係。 當您看到憑證警告時，按一下 [繼續]。 接下來，您會看到 OPC 發行者不信任 UA Web 用戶端的錯誤。 若要解決這個錯誤，將 **UA Web 用戶端**憑證從 `<SharedFolder>/CertificateStores/rejected/certs` 資料夾複製到閘道上的 `<SharedFolder>/CertificateStores/trusted/certs` 資料夾。 您不需要重新啟動閘道。

您現在可以從雲端連線到閘道，而且您也準備好在解決方案中新增 OPC UA 伺服器。

## <a name="add-your-own-opc-ua-servers"></a>新增自己的 OPC UA 伺服器

將自己的 OPC UA 伺服器新增至連線處理站解決方案加速器：

1. 瀏覽至連線處理站解決方案入口網站中的 [連線您自己的 OPC UA 伺服器] 頁面。

    1. 啟動您想要連線的 OPC UA 伺服器。 確保可從在容器中執行的 OPC 發行者和 OPC Proxy 連線到您的 OPC UA 伺服器 (請參閱先前有關名稱解析的註解)。
    1. 輸入 OPC UA 伺服器的端點 URL (`opc.tcp://<host>:<port>`)，然後按一下 [連線]。
    1. 在連線設定期間，會在連線處理站入口網站 (OPC UA 用戶端) 和您嘗試連線的 OPC UA 伺服器之間建立信任關係。 在連線處理站儀表板中，您會得到 [無法驗證您所要連線伺服器的憑證] 警告。 當您看到憑證警告時，按一下 [繼續]。
    1. 更難設定的是您正嘗試連線之 OPC UA 伺服器的憑證設定。 對於以電腦為基礎的 OPC UA 伺服器，可能只會在儀表板中顯示一個您可以確認的警告對話方塊。 對於內嵌的 OPC UA 伺服器系統，請參閱 OPC UA 伺服器的文件，以查閱如何完成這項工作。 您可能需要連線處理站入口網站的 OPC UA 用戶端憑證，才能完成這項工作。 系統管理員可以在 [連接您自己的 OPC UA 伺服器] 頁面上下載此憑證：

        ![解決方案入口網站](./media/iot-accelerators-connected-factory-gateway-deployment/image4.png)

1. 瀏覽 OPC UA 伺服器的 OPC UA 節點樹狀結構、以滑鼠右鍵按一下您想要傳送值到連線處理站的 OPC 節點，然後選取 [發行]。

1. 現在會從閘道裝置傳送遙測。 您可以在連線處理站入口網站，位於 [新增處理站] 之下的 [處理站位置] 中檢視遙測。

## <a name="next-steps"></a>後續步驟

若要深入了解連線處理站解決方案加速器的架構，請參閱[連線處理站解決方案加速器逐步解說](iot-accelerators-connected-factory-sample-walkthrough.md)。

了解 [OPC 發行者參考實作](https://docs.microsoft.com/azure/iot-suite/iot-suite-connected-factory-publisher)。