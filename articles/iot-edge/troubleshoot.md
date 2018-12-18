---
title: 對 Azure IoT Edge 進行移難排解 | Microsoft Docs
description: 解決 Azure IoT Edge 的常見問題及了解其疑難排解技術
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 06/26/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: a6102a6bc28486c24134bbc172b9e8a7e1a61244
ms.sourcegitcommit: cfff72e240193b5a802532de12651162c31778b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39308032"
---
# <a name="common-issues-and-resolutions-for-azure-iot-edge"></a>Azure IoT Edge 的常見問題和解決方案

如果您在您的環境中執行 Azure IoT Edge 時遇到問題，請使用本文作為疑難排解和解決方案的指引。 

## <a name="standard-diagnostic-steps"></a>標準的診斷步驟 

當您遇到問題時，可透過檢閱容器記錄與裝置的往來訊息，深入了解 IoT Edge 裝置的狀態。 可使用這一節中的命令與工具來收集資訊。 

### <a name="check-the-status-of-the-iot-edge-security-manager-and-its-logs"></a>檢查 IoT Edge 安全性管理員的狀態及其記錄：

在 Linux 上：
- 若要檢視 IoT Edge 安全性管理員的狀態：

   ```bash
   sudo systemctl status iotedge
   ```

- 若要檢視 IoT Edge 安全性管理員的記錄：

    ```bash
    sudo journalctl -u iotedge -f
    ```

- 若要檢視 IoT Edge 安全性管理員更詳細的記錄：

   - 編輯 iotedge 精靈設定：

      ```bash
      sudo systemctl edit iotedge.service
      ```
   
   - 更新以下幾行：
    
      ```
      [Service]
      Environment=IOTEDGE_LOG=edgelet=debug
      ```
    
   - 重新啟動 IoT Edge 安全性精靈：
    
      ```bash
      sudo systemctl cat iotedge.service
      sudo systemctl daemon-reload
      sudo systemctl restart iotedge
      ```

在 Windows 上：
- 若要檢視 IoT Edge 安全性管理員的狀態：

   ```powershell
   Get-Service iotedge
   ```

- 若要檢視 IoT Edge 安全性管理員的記錄：

   ```powershell
   # Displays logs from today, newest at the bottom.
 
   Get-WinEvent -ea SilentlyContinue `
   -FilterHashtable @{ProviderName= "iotedged";
     LogName = "application"; StartTime = [datetime]::Today} |
   select TimeCreated, Message |
   sort-object @{Expression="TimeCreated";Descending=$false} |
   format-table -autosize -wrap
   ```

### <a name="if-the-iot-edge-security-manager-is-not-running-verify-your-yaml-configuration-file"></a>如果 IoT Edge 安全性管理員未執行，請驗證您的 yaml 組態檔

> [!WARNING]
> YAML 檔案不可包含使用 Tab 鍵的縮排。 請改用 2 個空格。

在 Linux 上：

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

在 Windows 上：

   ```cmd
   notepad C:\ProgramData\iotedge\config.yaml
   ```

### <a name="check-container-logs-for-issues"></a>檢查容器記錄的問題

在 IoT Edge 安全性精靈開始執行後，請查看容器的記錄以偵測問題。 從您已部署的容器開始著手，然後查看構成 IoT Edge 執行階段的容器：IoT 代理程式和 IoT 中樞。 IoT 代理程式記錄通常會提供每個容器的生命週期相關資訊。 IoT 中樞記錄會提供傳訊和路由的相關資訊。 

   ```cmd
   iotedge logs <container name>
   ```

### <a name="view-the-messages-going-through-the-edge-hub"></a>檢視通過 Edge 中樞的訊息

檢視通過 Edge 中樞的訊息，以及透過 edgeAgent 和 edgeHub 執行階段容器的詳細記錄來收集裝置屬性更新的深入解析。 若要開啟這些容器上的詳細資訊記錄，請在 yaml 組態檔中設定 `RuntimeLogLevel`。 若要開啟檔案：

在 Linux 上：

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

在 Windows 上：

   ```cmd
   notepad C:\ProgramData\iotedge\config.yaml
   ```

根據預設，`agent` 元素會如下所示：

   ```yaml
   agent:
     name: edgeAgent
     type: docker
     env: {}
     config:
       image: mcr.microsoft.com/azureiotedge-agent:1.0
       auth: {}
   ```

將 `env: {}` 取代為：

> [!WARNING]
> YAML 檔案不可包含使用 Tab 鍵的縮排。 請改用 2 個空格。

   ```yaml
   env:
     RuntimeLogLevel: debug
   ```

儲存檔案並重新啟動 IoT Edge 安全性管理員。

您也可以檢查在 IoT 中樞與 IoT Edge 裝置之間傳送的訊息。 您可以使用 Visual Studio Code 的 [Azure IoT 工具組](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit)擴充功能來檢視這些訊息。 如需詳細指引，請參閱[使用 Azure IoT 進行開發時的便利工具](https://blogs.msdn.microsoft.com/iotdev/2017/09/01/handy-tool-when-you-develop-with-azure-iot/)。

### <a name="restart-containers"></a>重新啟動容器
檢查過記錄和訊息並取得資訊後，您可以嘗試重新啟動容器：

```
iotedge restart <container name>
```

重新啟動 IoT Edge 執行階段容器：

```
iotedge restart edgeAgent && iotedge restart edgeHub
```

### <a name="restart-the-iot-edge-security-manager"></a>重新啟動 IoT Edge 安全性管理員

如果問題仍持續發生，您可以嘗試重新啟動 IoT Edge 安全性管理員。

在 Linux 上：

   ```cmd
   sudo systemctl restart iotedge
   ```

在 Windows 上：

   ```powershell
   Stop-Service iotedge -NoWait
   sleep 5
   Start-Service iotedge
   ```

## <a name="edge-agent-stops-after-about-a-minute"></a>Edge 代理程式會在大約一分鐘後停止

Edge 代理程式會啟動並成功執行約一分鐘，然後停止。 記錄指出 Edge 代理程式會嘗試透過 AMQP 連線到 IoT 中樞，然後大約 30 秒之後嘗試使用 AMQP 透過 WebSocket 連線。 當這些作業失敗後，Edge 代理程式就會退出。 

Edge 代理程式記錄的範例：

```output
2017-11-28 18:46:19 [INF] - Starting module management agent. 
2017-11-28 18:46:19 [INF] - Version - 1.0.7516610 (03c94f85d0833a861a43c669842f0817924911d5) 
2017-11-28 18:46:19 [INF] - Edge agent attempting to connect to IoT Hub via AMQP... 
2017-11-28 18:46:49 [INF] - Edge agent attempting to connect to IoT Hub via AMQP over WebSocket... 
```

### <a name="root-cause"></a>根本原因
主機網路上的網路組態阻止了 Edge 代理程式觸達該網路。 代理程式首先嘗試透過 AMQP (連接埠 5671) 進行連線。 如果此作業失敗，代理程式會嘗試 Websocket (連接埠 443)。

IoT Edge 執行階段會為每個模組設定要在其中通訊的網路。 在 Linux 上，此網路是橋接網路。 在 Windows 上則是使用 NAT。 此問題較常見於使用 Windows 容器 (使用 NAT 網路) 的 Windows 裝置。 

### <a name="resolution"></a>解決方案
確認指派給此橋接器/NAT 網路的 IP 位址能路由至網際網路。 有時候主機上的 VPN 組態會覆寫 IoT Edge 網路。 

## <a name="edge-hub-fails-to-start"></a>Edge 中樞無法啟動

Edge 中樞無法啟動，且將下列訊息輸出至記錄： 

```output
One or more errors occurred. 
(Docker API responded with status code=InternalServerError, response=
{\"message\":\"driver failed programming external connectivity on endpoint edgeHub (6a82e5e994bab5187939049684fb64efe07606d2bb8a4cc5655b2a9bad5f8c80): 
Error starting userland proxy: Bind for 0.0.0.0:443 failed: port is already allocated\"}\n) 
```

### <a name="root-cause"></a>根本原因
在主機機器上的某些其他程序已繫結連接埠 443。 Edge 中樞會對應連接埠 5671 和 443，以在閘道案例中使用。 如果另一個程序已繫結此連接埠，則此連接埠對應會失敗。 

### <a name="resolution"></a>解決方案
尋找並停止正在使用連接埠 443 的程序。 此程序通常是網頁伺服器。

## <a name="edge-agent-cant-access-a-modules-image-403"></a>Edge 代理程式無法存取模組的映像 (403)
容器無法執行，而且 Edge 代理程式記錄顯示 403 錯誤。 

### <a name="root-cause"></a>根本原因
Edge 代理程式沒有存取模組映像的權限。 

### <a name="resolution"></a>解決方案
確定您已在部署資訊清單中正確指定登錄認證

## <a name="iot-edge-security-daemon-fails-with-an-invalid-hostname"></a>IoT Edge 安全性精靈因主機名稱無效而失敗

命令 `sudo journalctl -u iotedge` 失敗，並列印下列訊息： 

```output
Error parsing user input data: invalid hostname. Hostname cannot be empty or greater than 64 characters
```

### <a name="root-cause"></a>根本原因
IoT Edge 執行階段只能支援少於 64 個字元的主機名稱。 如此通常不會造成實體機器發生問題，但當在虛擬機器上設定執行階段時，就可能會產生問題。 在 Azure 中代管的 Windows 虛擬機器自動產生的主機名稱通常很長。 

### <a name="resolution"></a>解決方案
當發現這個錯誤時，可以設定虛擬機器的 DNS 名稱，然後將 DNS 名稱設定為安裝命令中的主機名稱來解決。

1. 在 Azure 入口網站中，瀏覽至虛擬機器的概觀頁面。 
2. 選取 DNS 名稱下的 [設定]。 如果虛擬機器已設定 DNS 名稱，則不需要設定新的名稱。 

   ![設定 DNS 名稱](./media/troubleshoot/configure-dns.png)

3. 提供 **DNS 名稱標籤**的值，並選取 [儲存]。
4. 複製新的 DNS 名稱，格式應該是 **\<DNSnamelabel\>.\<vmlocation\>.cloudapp.azure.com**。
5. 在虛擬機器中，使用下列命令以您的 DNS 名稱設定 IoT Edge 執行階段：

   - 在 Linux 上：

      ```bash
      sudo nano /etc/iotedge/config.yaml
      ```

   - 在 Windows 上：

      ```cmd
      notepad C:\ProgramData\iotedge\config.yaml
      ```

## <a name="stability-issues-on-resource-constrained-devices"></a>資源受限裝置的穩定性問題 
您可能會遇到受限裝置 (例如 Raspberry Pi) 的穩定性問題，尤其在該裝置當作閘道的時候。 徵兆包括 Edge 中樞模組的記憶體不足例外狀況，下游裝置無法連線，或裝置在幾小時後停止傳送遙測訊息。

### <a name="root-cause"></a>根本原因
根據預設，Edge 中樞 (也是邊線執行階段的一部分) 已針對效能最佳化，而且會嘗試配置大量的記憶體。 這不適合用於受限邊緣裝置，而且可能會造成穩定性問題。

### <a name="resolution"></a>解決方案
針對 Edge 中樞，將環境變數 **OptimizeForPerformance** 設定為 **false**。 作法有二：

在 UI 中： 

在入口網站中，從 [裝置詳細資料]->[設定模組]->[設定進階 Edge 執行階段設定]，建立名為 OptimizeForPerformance 的環境變數，該變數會針對 [Edge 中樞] 設定為 false。

![optimizeforperformance][img-optimize-for-perf]

**或**

在部署資訊清單中：

```json
  "edgeHub": {
    "type": "docker",
    "settings": {
      "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
      "createOptions": <snipped>
    },
    "env": {
      "OptimizeForPerformance": {
          "value": "false"
      }
    },
```
## <a name="cant-get-the-iot-edge-daemon-logs-on-windows"></a>無法在 Windows 上取得 IoT Edge 精靈記錄 (daemon log)
如果您在 Windows 上使用 `Get-WinEvent` 時收到 EventLogException，請檢查您的登錄項目。

### <a name="root-cause"></a>根本原因
`Get-WinEvent` PowerShell 命令須依賴存在的登錄項目，才能根據特定 `ProviderName` 尋找記錄。

### <a name="resolution"></a>解決方案
設定 IoT Edge 精靈的登錄項目。 建立具有下列內容的 **iotedge.reg** 檔案，然後按兩下此檔案或使用 `reg import iotedge.reg` 命令將檔案匯入至 Windows 登錄：

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\Application\iotedged]
"CustomSource"=dword:00000001
"EventMessageFile"="C:\\ProgramData\\iotedge\\iotedged.exe"
"TypesSupported"=dword:00000007
```


## <a name="next-steps"></a>後續步驟
您在 IoT Edge 平台中發現到錯誤嗎？ 請[提交問題](https://github.com/Azure/iotedge/issues)，讓我們可以持續進行改善。 

<!-- Images -->
[img-optimize-for-perf]: ./media/troubleshoot/OptimizeForPerformanceFalse.png
