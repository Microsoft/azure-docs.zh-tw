---
title: 如何在 Linux 上安裝 Azure IoT Edge | Microsoft Docs
description: Linux 上的 Azure IoT Edge 安裝指示
author: kgremban
manager: timlt
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 08/27/2018
ms.author: kgremban
ms.openlocfilehash: f515ddc58a9f9b434e40f44ca7cc50d738ed69f0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993103"
---
# <a name="install-the-azure-iot-edge-runtime-on-linux-x64"></a>在 Linux (x64) 上安裝 Azure IoT Edge 執行階段

Azure IoT Edge 執行階段可將裝置變成 IoT Edge 裝置。 此執行階段可以部署在像 Raspberry Pi 一樣小或像工業伺服器一樣大的裝置上。 利用 IoT Edge 執行階段設定裝置之後，您就可以開始從雲端將商務邏輯部署給它。 

若要深入了解 IoT Edge 執行階段的運作方式，以及會包含哪些元件，請參閱[了解 Azure IoT Edge 執行階段及其架構](iot-edge-runtime.md)。

本文列出在 Linux x64 (Intel/AMD) Edge 裝置上安裝 Azure IoT Edge 執行階段的步驟。 如需目前支援的 AMD64 作業系統清單，請參閱 [Azure IoT Edge 支援](support.md#operating-systems)。 

>[!NOTE]
>Linux 軟體存放庫中的套件受限於每個套件中的授權條款 (/usr/share/doc/package-name)。 在使用套件之前，請先閱讀授權條款。 安裝及使用套件即表示接受這些授權條款。 如果您不同意授權條款，請勿使用套件。

## <a name="register-microsoft-key-and-software-repository-feed"></a>註冊 Microsoft 金鑰和軟體存放庫摘要

根據您的作業系統選擇適當的指令碼，為裝置進行 IoT edge 執行階段安裝的準備工作。 

### <a name="ubuntu-1604"></a>Ubuntu 16.04

```bash
# Install repository configuration
curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > ./microsoft-prod.list
sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/

# Install Microsoft GPG public key
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
```

### <a name="ubuntu-1804"></a>Ubuntu 18.04

```bash
# Install repository configuration
curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list > ./microsoft-prod.list
sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/

# Install Microsoft GPG public key
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/

# Perform apt upgrade
sudo apt-get upgrade
```

## <a name="install-the-container-runtime"></a>安裝容器執行階段 

Azure IoT Edge 會依賴 [OCI 相容][lnk-oci]的容器執行階段。 針對生產案例，強烈建議您使用下方所提供的 [Moby 型][lnk-moby]引擎。 它是 Azure IoT Edge 正式支援的唯一容器引擎。 Docker CE/EE 容器映像與 Moby 執行階段相容。

更新 apt-get。

```bash
sudo apt-get update
```

安裝 Moby 引擎。 

```bash
sudo apt-get install moby-engine
```

安裝 Moby 命令列介面 (CLI)。 CLI 對於開發相當有用，但是對於生產部署則是選擇性的。

```bash
sudo apt-get install moby-cli
```

## <a name="install-the-azure-iot-edge-security-daemon"></a>安裝 Azure IoT Edge 安全性精靈

**IoT Edge 安全性精靈**提供及維護 Edge 裝置的安全性標準。 此精靈會在每次開機時啟動，並藉由啟動 IoT Edge 執行階段讓裝置進入啟動程序。 

如果 **iothsmlib** 尚不存在，安裝命令也會安裝其標準版本。

更新 apt-get。

```bash
sudo apt-get update
```

安裝安全性精靈。 套件會安裝在 `/etc/iotedge/`。

```bash
sudo apt-get install iotedge
```

## <a name="configure-the-azure-iot-edge-security-daemon"></a>設定 Azure IoT Edge 安全性精靈

設定 IoT Edge 執行階段，以連結您的實體裝置與 Azure IoT 中樞中存在的裝置身分識別。 

您可以使用 `/etc/iotedge/config.yaml` 上的組態檔來設定精靈。 此檔案預設有防寫保護，您可能需要提高的權限才能加以編輯。

單一 IoT Edge 裝置可以使用 IoT 中樞所提供的裝置連接字串以手動方式佈建。 或者，您可以使用裝置佈建服務來自動佈建裝置，這在您有許多裝置要佈建時將很有幫助。 請根據您選擇的佈建方式，選擇適當的安裝指令碼。 

### <a name="option-1-manual-provisioning"></a>選項 1：手動佈建

若要手動佈建裝置，您需要提供[裝置連接字串][lnk-dcs]，在您的 IoT 中樞註冊新裝置即可建立該字串。


開啟組態檔。 

```bash
sudo nano /etc/iotedge/config.yaml
```

尋找檔案的佈建區段，然後取消註解**手動**佈建模式。 以來自 IoT Edge 裝置的連接字串更新 **device_connection_string** 的值。

   ```yaml
   provisioning:
     source: "manual"
     device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"
  
   # provisioning: 
   #   source: "dps"
   #   global_endpoint: "https://global.azure-devices-provisioning.net"
   #   scope_id: "{scope_id}"
   #   registration_id: "{registration_id}"
   ```

儲存並關閉檔案。 

   `CTRL + X`、`Y`, `Enter`

在組態檔中輸入佈建資訊之後，請重新啟動精靈：

```bash
sudo systemctl restart iotedge
```

### <a name="option-2-automatic-provisioning"></a>選項 2：自動佈建

若要自動佈建裝置，[請設定裝置佈建服務並擷取您的裝置註冊識別碼][lnk-dps] (DPS)。 自動佈建只適用於具有信賴平台模組 (TPM) 晶片的裝置。 例如，Raspberry Pi 裝置預設未隨附 TPM。 

開啟組態檔。 

```bash
sudo nano /etc/iotedge/config.yaml
```

尋找檔案的佈建區段，然後取消註解 **dps** 佈建模式。 請將 **scope_id** 和 **registration_id** 的值更新為 IoT 中樞裝置佈建服務和具有 TPM 的 IoT Edge 裝置中的值。 

   ```yaml
   # provisioning:
   #   source: "manual"
   #   device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"
  
   provisioning: 
     source: "dps"
     global_endpoint: "https://global.azure-devices-provisioning.net"
     scope_id: "{scope_id}"
     registration_id: "{registration_id}"
   ```

儲存並關閉檔案。 

   `CTRL + X`、`Y`, `Enter`

在組態檔中輸入佈建資訊之後，請重新啟動精靈：

```bash
sudo systemctl restart iotedge
```

## <a name="verify-successful-installation"></a>確認安裝成功

如果您在上一節中使用**手動設定**步驟，IoT Edge 執行階段應會在您的裝置上成功佈建並執行。 如果您使用**自動設定**步驟，則必須完成一些額外的步驟，讓執行階段可代表您向您的 IoT 中樞註冊裝置。 如需後續步驟，請參閱[在 Linux 虛擬機器上建立及佈建模擬 TPM Edge 裝置](how-to-auto-provision-simulated-device-linux.md#give-iot-edge-access-to-the-tpm)。

您可以使用以下項目，檢查 IoT Edge 精靈的狀態：

```bash
systemctl status iotedge
```

使用以下項目檢查精靈記錄：

```bash
journalctl -u iotedge --no-pager --no-full
```

並使用以下項目列出執行中的模組：

```bash
sudo iotedge list
```

## <a name="tips-and-suggestions"></a>祕訣與建議

您必須要有提高的權限才能執行 `iotedge` 命令。 安裝執行階段之後，請登出您的電腦並重新登入，以自動更新您的權限。 在那之前，請在任何 `iotedge` 命令前面使用 **sudo**。

在資源受限的裝置上，強烈建議將 OptimizeForPerformance 環境變數設定為 false，如[疑難排解指南][lnk-trouble]中的指示所述。

如果您的網路具有 Proxy 伺服器，請遵循[設定您的 IoT Edge 裝置以透過 Proxy 伺服器進行通訊](how-to-configure-proxy-support.md)中的步驟。

## <a name="next-steps"></a>後續步驟

如果您有 Edge 執行階段是否正確安裝的問題，請參閱[疑難排解][lnk-trouble]頁面。

<!-- Links -->
[lnk-dcs]: how-to-register-device-portal.md
[lnk-dps]: how-to-auto-provision-simulated-device-linux.md
[lnk-oci]: https://www.opencontainers.org/
[lnk-moby]: https://mobyproject.org/
[lnk-trouble]: troubleshoot.md
