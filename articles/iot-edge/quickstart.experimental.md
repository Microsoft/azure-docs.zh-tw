---
title: 在 Windows 上模擬 Azure IoT Edge | Microsoft Docs
description: 在 Windows 中的模擬裝置上安裝 Azure IoT Edge 執行階段及部署您的第一個模組
author: kgremban
manager: timlt
ms.author: kgremban
ms.reviewer: elioda
ms.date: 06/08/2018
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: mvc
ms.openlocfilehash: 1a45841564b0c985662e6d2db320111fa27d1e92
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39578172"
---
# <a name="quickstart-deploy-your-first-iot-edge-module-from-the-azure-portal-to-a-windows-device---preview"></a>快速入門：從 Azure 入口網站將您的第一個 IoT Edge 模組部署至 Windows 裝置 - 預覽

Azure IoT Edge 可讓您在裝置上執行分析和資料處理，而不必將所有的資料推送至雲端。 IoT Edge 教學課程說明如何部署不同類型的模組，但首先您需要能夠測試的裝置。 

在此快速入門中，您將了解如何：

1. 建立 IoT 中樞
2. 註冊 IoT Edge 裝置
3. 啟動 IoT Edge 執行階段
4. 部署模組

![教學課程架構][2]

您在本快速入門中部署的模組是一個模擬感應器，會產生溫度、溼度和壓力資料。 其他 Azure IoT Edge 教學課程會以您在此所做的工作為基礎，部署模組來分析模擬資料以產生商業見解。 

>[!NOTE]
>Windows 上的 IoT Edge 執行階段為[公開預覽版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

如果您沒有使用中的 Azure 訂用帳戶，請在開始前建立[免費帳戶][lnk-account]。

## <a name="prerequisites"></a>必要條件

本快速入門假設您使用執行 Windows 的電腦或虛擬機器來模擬 IoT 裝置。 如果您在虛擬機器中執行 Windows，請啟用[巢狀虛擬化][lnk-nested]並配置至少 2GB 的記憶體。 

您要用於 IoT Edge 裝置的電腦必須符合下列先決條件：

1. 請確定您使用的是受支援的 Windows 版本：
   * Windows 10 或更新版本
   * Windows Server 2016 或更新版本
2. 安裝[適用於 Windows 的 Docker][lnk-docker] 並確定它正在執行。
3. 設定 Docker 容器以使用 [Linux 容器](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers)

## <a name="create-an-iot-hub"></a>建立 IoT 中樞

在 Azure 入口網站中建立 IoT 中樞，以開始進行此快速入門。
![建立 IoT 中樞][3]

在資源群組中，建立可用來對您在本快速入門中建立的所有資源進行維護和管理的 IoT 中樞。 請將其命名為易記的名稱，例如 **IoTEdgeResources**。 將快速入門和教學課程的所有資源放同一個群組中，可一併加以管理，並可在完成測試後輕鬆加以移除。 

[!INCLUDE [iot-hub-create-hub](../../includes/iot-hub-create-hub.md)]

## <a name="register-an-iot-edge-device"></a>註冊 IoT Edge 裝置

向新建立的 IoT 中樞註冊 IoT Edge 裝置。
![註冊裝置][4]

[!INCLUDE [iot-edge-register-device](../../includes/iot-edge-register-device.md)]

## <a name="install-and-start-the-iot-edge-runtime"></a>安裝並啟動 IoT Edge 執行階段

在您的 IoT Edge 裝置上安裝並啟動 Azure IoT Edge 執行階段。 
![註冊裝置][5]

IoT Edge 執行階段會在所有 IoT Edge 裝置上部署。 它有三個元件。 **IoT Edge 安全性精靈**會在每次 Edge 裝置開機時啟動，並藉由啟動 IoT Edge 代理程式來啟動該裝置。 **IoT Edge 代理程式**有助於在 IoT Edge 裝置 (包括 IoT Edge 中樞) 上部署及監視模組。 **IoT Edge 中樞**會管理 IoT Edge 裝置上的模組通訊，以及裝置與 IoT 中樞之間的通訊。 

>[!NOTE]
>在安裝指令碼開發期間，本節中的安裝步驟均採手動執行。 

本節中的指示會設定 Linux 容器的 IoT Edge 執行階段。 如果您想要使用 Windows 容器，請參閱[在 Windows 上安裝要用於 Windows 容器的 Azure IoT Edge 執行階段](how-to-install-iot-edge-windows-with-windows.md)。

### <a name="download-and-install-the-iot-edge-service"></a>下載並安裝 IoT Edge 服務

1. 在您的 IoT Edge 裝置上，以系統管理員身分執行 PowerShell。

2. 下載 IoT Edge 服務套件。

   ```powershell
   Invoke-WebRequest https://aka.ms/iotedged-windows-latest -o .\iotedged-windows.zip
   Expand-Archive .\iotedged-windows.zip C:\ProgramData\iotedge -f
   Move-Item c:\ProgramData\iotedge\iotedged-windows\* C:\ProgramData\iotedge\ -Force
   rmdir C:\ProgramData\iotedge\iotedged-windows
   $sysenv = "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment"
   $path = (Get-ItemProperty -Path $sysenv -Name Path).Path + ";C:\ProgramData\iotedge"
   Set-ItemProperty -Path $sysenv -Name Path -Value $path
   ```

3. 安裝 vcruntime。

  ```powershell
  Invoke-WebRequest -useb https://download.microsoft.com/download/0/6/4/064F84EA-D1DB-4EAA-9A5C-CC2F0FF6A638/vc_redist.x64.exe -o vc_redist.exe
  .\vc_redist.exe /quiet /norestart
  ```

4. 建立並啟動 IoT Edge 服務。

   ```powershell
   New-Service -Name "iotedge" -BinaryPathName "C:\ProgramData\iotedge\iotedged.exe -c C:\ProgramData\iotedge\config.yaml"
   Start-Service iotedge
   ```

5. 為 IoT Edge 服務所使用的連接埠新增防火牆例外狀況。

   ```powershell
   New-NetFirewallRule -DisplayName "iotedged allow inbound 15580,15581" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 15580-15581 -Program "C:\programdata\iotedge\iotedged.exe" -InterfaceType Any
   ```

6. 建立名為 **iotedge.reg** 的新檔案，並以文字編輯器加以開啟。 

7. 將下列內容，並儲存檔案。 

   ```input
   Windows Registry Editor Version 5.00
   [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\Application\iotedged]
   "CustomSource"=dword:00000001
   "EventMessageFile"="C:\\ProgramData\\iotedge\\iotedged.exe"
   "TypesSupported"=dword:00000007
   ```

8. 在檔案總管中瀏覽至您的檔案，然後按兩下該檔案將變更匯入至 Windows 登錄。 

### <a name="configure-the-iot-edge-runtime"></a>設定 IoT Edge 執行階段 

使用您在註冊新裝置時所複製的 IoT Edge 裝置連接字串來設定執行階段。 接著，設定執行階段網路。 

1. 開啟位於 `C:\ProgramData\iotedge\config.yaml` 的 IoT Edge 組態檔。 此檔案受到保護，因此請以系統管理員身分執行「記事本」之類的文字編輯器，然後使用該編輯器開啟檔案。 

2. 尋找 [佈建] 區段，並以您從 IoT Edge 裝置詳細資料複製的字串更新 **device_connection_string** 的值。 

3. 在系統管理員 PowerShell 視窗中擷取 IoT Edge 裝置的主機名稱，並複製輸出。 

   ```powershell
   hostname
   ```

4. 在組態檔中，尋找 [Edge 裝置主機名稱] 區段。 請以您從 PowerShell 複製的主機名稱更新 **hostname** 的值。

3. 在系統管理員 PowerShell 視窗中，擷取 IoT Edge 裝置的 IP 位址。 

   ```powershell
   ipconfig
   ```

4. 在輸出的 [vEthernet (DockerNAT)] 區段中複製 [IPv4 位址] 的值。 

5. 建立名為 **IOTEDGE_HOST** 的環境變數，並將 *\<ip_address\>* 取代為 IoT Edge 裝置的 IP 位址。 

  ```powershell
  [Environment]::SetEnvironmentVariable("IOTEDGE_HOST", "http://<ip_address>:15580")
  ```

  在重新啟動時保存環境變數。

  ```powershell
  SETX /M IOTEDGE_HOST "http://<ip_address>:15580"
  ```

6. 在 `config.yaml` 檔案中，尋找 [連接設定] 區段。 以您的 IP 位址和您在上一節中開啟的連接埠更新 **management_uri** 和 **workload_uri** 的值。 請將 **\<GATEWAY_ADDRESS\>** 取代為您所複製的 DockerNAT IP 位址。

   ```yaml
   connect: 
     management_uri: "http://<GATEWAY_ADDRESS>:15580"
     workload_uri: "http://<GATEWAY_ADDRESS>:15581"
   ```

7. 尋找 [接聽設定] 區段，並為 **management_uri** 和 **workload_uri** 新增相同的值。 

   ```yaml
   listen:
     management_uri: "http://<GATEWAY_ADDRESS>:15580"
     workload_uri: "http://<GATEWAY_ADDRESS>:15581"
   ```

8. 尋找 [Moby 容器執行階段設定] 區段，並確認 [網路] 的值已取消註解並設定為 **azure-iot-edge**

   ```yaml
   moby_runtime:
     docker_uri: "npipe://./pipe/docker_engine"
     network: "azure-iot-edge"
   ```
   
9. 儲存組態檔。 

10. 在 PowerShell 中，重新啟動 IoT Edge 服務。

   ```powershell
   Stop-Service iotedge -NoWait
   sleep 5
   Start-Service iotedge
   ```

### <a name="view-the-iot-edge-runtime-status"></a>檢視 IoT Edge 執行階段狀態

確認已成功安裝並設定執行階段。

1. 檢查 IoT Edge 服務的狀態。

   ```powershell
   Get-Service iotedge
   ```

2. 如果您需要對服務進行疑難排解，請擷取服務記錄。 

   ```powershell
   # Displays logs from today, newest at the bottom.

   Get-WinEvent -ea SilentlyContinue `
    -FilterHashtable @{ProviderName= "iotedged";
      LogName = "application"; StartTime = [datetime]::Today} |
    select TimeCreated, Message |
    sort-object @{Expression="TimeCreated";Descending=$false} |
    format-table -autosize -wrap
   ```

3. 檢視所有在 IoT Edge 裝置上執行的模組。 由於服務只是第一次啟動，您應該只會看到 **edgeAgent** 模組正在執行。 EdgeAgent 模組依預設會執行，且有助於安裝及啟動您部署至裝置的任何其他模組。 

   ```powershell
   iotedge list
   ```

   ![檢視裝置上的一個模組](./media/quickstart/iotedge-list-1.png)

## <a name="deploy-a-module"></a>部署模組

從雲端管理您的 Azure IoT Edge 裝置，以部署會將遙測資料傳送到 IoT 中樞的模組。
![註冊裝置][6]

[!INCLUDE [iot-edge-deploy-module](../../includes/iot-edge-deploy-module.md)]

## <a name="view-generated-data"></a>檢視產生的資料

在此快速入門中，您可以建立新的 IoT Edge 裝置，並在其中安裝 IoT Edge 執行階段。 然後，您會使用 Azure 入口網站來推送 IoT Edge 模組，讓其無須變更裝置本身就能在裝置上執行。 在此案例中，您推送的模組會建立可在教學課程中使用的環境資料。 

確認從雲端部署的模組是在 IoT Edge 裝置上執行。 

```powershell
iotedge list
```

   ![在您的裝置上檢視三個模組](./media/quickstart/iotedge-list-2.png)

檢視從 tempSensor 模組傳送至雲端的訊息。 

```powershell
iotedge logs tempSensor -f
```

  ![從您的模組中檢視資料](./media/quickstart/iotedge-logs.png)

您也可以使用[適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit)，來檢視 IoT 中樞所接收的訊息。 

## <a name="clean-up-resources"></a>清除資源

如果您想要繼續進行 IoT Edge 教學課程，您可以使用在本快速入門中註冊和設定的裝置。 否則，您可以刪除您所建立的 Azure 資源，並從裝置中移除 IoT Edge 執行階段。 

### <a name="delete-azure-resources"></a>刪除 Azure 資源

如果您是在新的資源群組中建立虛擬機器和 IoT 中樞，您可以刪除該群組和所有相關聯的資源。 如果該資源群組中有您想要保留的項目，則只要刪除您要清除的個別資源即可。 

若要移除資源群組，請依照下列步驟操作： 

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後按一下 [資源群組]。
2. 在 [依名稱篩選...] 文字方塊中，輸入包含 IoT 中樞的資源群組名稱。 
3. 在結果清單中的資源群組右側，按一下 **...**，然後按一下 [刪除資源群組]。
4. 系統將會要求您確認是否刪除資源。 再次輸入您的資源群組名稱進行確認，然後按一下 [刪除]。 片刻過後，系統便會刪除該資源群組及其所有內含的資源。

### <a name="remove-the-iot-edge-runtime"></a>移除 IoT Edge 執行階段

如果您預計要在未來的測試中使用 IoT Edge 裝置，但想要在未使用時停止讓 tempSensor 模組傳送資料至 IoT 中樞，請使用下列命令停止 IoT Edge 服務。 

   ```powershell
   Stop-Service iotedge -NoWait
   ```

當您準備好再次開始測試時，便可重新啟動服務

   ```powershell
   Start-Service iotedge
   ```

如果您想要從裝置移除這些安裝，請使用下列命令。  

移除 IoT Edge 執行階段。

   ```powershell
   cmd /c sc delete iotedge
   rm -r c:\programdata\iotedge
   ```

IoT Edge 執行階段移除後，它所建立的容器隨即停止，但仍會存在於您的裝置上。 檢視所有容器。

   ```powershell
   docker ps -a
   ```

刪除 IoT Edge 執行階段在您的裝置上建立的容器。 如果您將 tempSensor 容器命名為其他名稱，請變更其名稱。 

   ```powershell
   docker rm -f tempSensor
   docker rm -f edgeHub
   docker rm -f edgeAgent
   ```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已建立新的 IoT Edge 裝置，並使用 Azure IoT Edge 雲端介面將程式碼部署至裝置上。 現在，您已有測試裝置，可產生其環境的相關原始資料。 

您可以繼續進行任何其他教學課程，以了解 Azure IoT Edge 可如何協助您將此資料轉換成 Edge 上的商業見解。

> [!div class="nextstepaction"]
> [使用 Azure 函式篩選感應器資料](tutorial-deploy-function.md)

<!-- Images -->
[2]: ./media/quickstart/install-edge-full.png
[3]: ./media/quickstart/create-iot-hub.png
[4]: ./media/quickstart/register-device.png
[5]: ./media/quickstart/start-runtime.png
[6]: ./media/quickstart/deploy-module.png

<!-- Links -->
[lnk-nested]: https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization
[lnk-docker]: https://docs.docker.com/docker-for-windows/install/ 
[lnk-python]: https://www.python.org/downloads/
[lnk-docker-containers]: https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10#2-switch-to-windows-containers
[lnk-install-iotcore]: how-to-install-iot-core.md
[lnk-account]: https://azure.microsoft.com/free
