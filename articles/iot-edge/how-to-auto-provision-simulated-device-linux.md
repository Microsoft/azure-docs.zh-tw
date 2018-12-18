---
title: 使用 DPS 自動佈建 Azure IoT Edge 裝置 - Linux | Microsoft Docs
description: 在 Linux VM 上使用模擬的 TPM 來測試 Azure IoT Edge 的裝置佈建
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 06/27/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 9609aab6c70bc0c2755de142023bd26e7417987a
ms.sourcegitcommit: 756f866be058a8223332d91c86139eb7edea80cc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37347592"
---
# <a name="create-and-provision-an-edge-device-with-a-virtual-tpm-on-a-linux-virtual-machine"></a>使用虛擬 TPM 在 Linux 虛擬機器上建立及佈建 Edge 裝置

Azure IoT Edge 裝置可用[裝置佈建服務](../iot-dps/index.yml)來自動佈建，就像未啟用 Edge 的裝置一樣。 如果您不熟悉自動佈建程序，請先檢閱[自動佈建概念](../iot-dps/concepts-auto-provisioning.md)，再繼續作業。 

本文將說明如何藉由下列步驟，在模擬 Edge 裝置上測試自動佈建： 

* 使用保護硬體的模擬信任平台模組 (TPM)，在 Hyper-V 中建立 Linux 虛擬機器 (VM)。
* 建立 IoT 中樞裝置佈建服務 (DPS) 的執行個體。
* 建立裝置的個別註冊
* 安裝 IoT Edge 執行階段，並將裝置連線至 IoT 中樞

本文中的步驟係供測試之用。

## <a name="prerequisites"></a>先決條件

* [已啟用 Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) 的 Windows 開發機器。 本文使用執行 Ubuntu Server VM 的 Windows 10。 
* 使用中的 IoT 中樞。 

## <a name="create-a-linux-virtual-machine-with-a-virtual-tpm"></a>使用虛擬 TPM 建立 Linux 虛擬機器

在本節中，您會在 Hyper-V 上建立具有模擬 TPM 的新 Linux 虛擬機器，以便使用該 TPM 測試 IoT Edge 的自動佈建情形。 

### <a name="create-a-virtual-switch"></a>建立虛擬交換器

虛擬交換器可讓您的虛擬機器連線至實體網路。

1. 在 Windows 機器上開啟 Hyper-V。 

2. 在 [動作] 功能表中，選取 [虛擬交換器管理員]。 

3. 選擇 [外部] 虛擬交換器，然後選取 [建立虛擬交換器]。 

4. 為新的虛擬交換器指定名稱，例如 **EdgeSwitch**。 確定連線類型設定為 [外部網路]，然後選取 [確定]。

5. 快顯視窗會警告您網路連線可能會中斷。 選取 [是] 以繼續進行。 

如果您在建立新的虛擬交換器時看到錯誤，請確定沒有其他交換器正在使用乙太網路配接器，且沒有其他參數使用相同的名稱。 

### <a name="create-virtual-machine"></a>建立虛擬機器

1. 下載要用於虛擬機器的磁碟映像檔，並將其儲存於本機。 例如 [Ubuntu Server](https://www.ubuntu.com/download/server)。 

2. 再次開啟 Hyper-V。 在 [動作] 功能表中，選取 [新增] > [虛擬機器]。

3. 使用下列特定組態完成 [新增虛擬機器精靈]：

   1. **指定世代**：選取 [第 2 代]。
   2. **設定網路功能**：將 [連線] 的值設定為您在上一節中建立的虛擬交換器。 
   3. **安裝選項**：選取 [從可開機映像檔安裝作業系統]，並瀏覽至您在本機儲存的磁碟映像檔。

建立 VM 可能需要幾分鐘的時間。 

### <a name="enable-virtual-tpm"></a>啟用虛擬 TPM

1. VM 建立後，請開啟其設定。 
2. 瀏覽至 [安全性]。 
3. 取消核取 [啟用安全開機]。
4. 核取 [啟用信賴平台模組]。 
5. 按一下 [確定]。  

### <a name="start-the-virtual-machine-and-collect-tpm-data"></a>啟動虛擬機器，並收集 TPM 資料

在虛擬機器中，建立可用來擷取裝置的 [註冊識別碼] 和 [簽署金鑰] 的 C SDK 工具。 

1. 啟動您的 VM 並連線，以完成安裝程序。 

2. 在您的 VM 中，依照[設定 Linux 開發環境](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md#linux)中的步驟安裝並建置 C 的 Azure IoT 裝置 SDK。 

3. 執行下列命令，建置用來擷取裝置佈建資訊的 C SDK 工具。 

   ```bash
   cd azure-iot-sdk-c/cmake
   cmake -Duse_prov_client:BOOL=ON ..
   cd provisioning_client/tools/tpm_device_provision
   make
   sudo ./tpm_device_provision
   ```

3. 複製 [註冊識別碼] 和 [簽署金鑰] 的值。 您可以使用這些值為 DPS 中的裝置建立個別的註冊。 

## <a name="set-up-the-iot-hub-device-provisioning-service"></a>設定 IoT 中樞裝置佈建服務

在 Azure 中建立 IoT 中樞裝置佈建服務的新執行個體，並將其連結至您的 IoT 中樞。 您可以依照[設定 IoT 中樞 DPS](../iot-dps/quick-setup-auto-provision.md) 中的指示操作。

裝置佈建服務開始執行之後，請從 [概觀] 頁面複製 [識別碼範圍] 的值。 當您設定 IoT Edge 執行階段時會用到此值。 

## <a name="create-a-dps-enrollment"></a>建立 DPS 註冊

從您的虛擬機器擷取佈建資訊，並使用該資訊在裝置佈建服務中建立個別的註冊。 

當您在 DPS 中建立註冊時，您有機會宣告**初始裝置對應項狀態**。 在裝置對應項中，您可以根據解決方案中需要的任何計量 (例如區域、環境、位置或裝置類型) 來設定標記，進而將裝置分組。 這些標記會用來建立[自動部署](how-to-deploy-monitor.md)。 


1. 在 [Azure 入口網站](https://portal.azure.com) 中，瀏覽至 IoT 中樞裝置佈建服務的執行個體。 

2. 在 [設定] 下方，選取 [管理註冊]。 

3. 選取 [新增個別註冊]，然後完成下列步驟以設定註冊：  

   1. 針對 [機制]，選取 [TPM]。 
   2. 插入您從虛擬機器複製的 [簽署金鑰] 和 [註冊識別碼]。
   3. 選取 [啟用] 以宣告此虛擬機器是 IoT Edge 裝置。 
   4. 選擇您的裝置所要連接的連結 **IoT 中樞**。 
   5. 視需要提供裝置的識別碼。 您可以使用裝置識別碼，將個別裝置設為模組部署的目標。 
   6. 視需要將標記值新增至 [初始裝置對應項狀態]。 您可以使用標記將裝置群組設定為模組部署的目標。 
   7. 選取 [ **儲存**]。 


## <a name="install-the-iot-edge-runtime"></a>安裝 IoT Edge 執行階段

IoT Edge 執行階段會在所有 IoT Edge 裝置上部署。 其元件會在容器中執行，並可讓您將其他容器部署到裝置，以便您在 Edge 上執行程式碼。 在虛擬機器上安裝 IoT Edge 執行階段。 

開始閱讀您的裝置類型適用的文章之前，請先了解您的 DPS [識別碼範圍] 和裝置的 [註冊識別碼]。 如果您安裝了範例 Ubuntu Server，請使用 **x64** 指示。 請務必將 IoT Edge 執行階段設定為自動佈建，而不是手動佈建。 

* [Linux (x64)](how-to-install-iot-edge-linux.md)
* [Linux (ARM32v7/armhf)](how-to-install-iot-edge-linux-arm.md)

## <a name="give-iot-edge-access-to-the-tpm"></a>為 IoT Edge 指定對 TPM 的存取權

若要讓 IoT Edge 執行階段自動佈建您的裝置，該執行階段必須能夠存取 TPM。 

請使用下列步驟提供 TPM 存取權。 或者，您可以覆寫 systemd 設定讓 *iotedge* 服務能夠以 Root 的形式執行，以達到相同的目的。 

1. 尋找 TPM 硬體模組在您裝置上的檔案路徑，並將其儲存為本機變數。 

   ```bash
   tpm=$(sudo find /sys -name dev -print | fgrep tpm | sed 's/.\{4\}$//')
   ```

2. 建立讓 IoT Edge 執行階段能夠存取 tpm0 的新規則。 

   ```bash
   sudo touch /etc/udev/rules.d/tpmaccess.rules
   ```

3. 開啟規則檔案。 

   ```bash
   sudo nano /etc/udev/rules.d/tpmaccess.rules
   ```

4. 將下列存取資訊複製到規則檔案中。 

   ```input 
   # allow iotedge access to tpm0
   KERNEL=="tpm0", SUBSYSTEM=="tpm", GROUP="iotedge", MODE="0660"
   ```

5. 儲存並結束檔案。 

6. 觸發程序 udev 系統以評估新規則。 

   ```bash
   /bin/udevadm trigger $tpm
   ```

7. 確認規則已成功套用。

   ```bash
   ls -l /dev/tpm0
   ```

   成功的輸出顯示如下：

   ```output
   crw------- 1 root root 10, 224 Jun 28 22:34 /dev/tpm0
   ```

8. 開啟 IoT Edge 執行階段複寫檔案。 

   ```bash
   sudo systemctl edit iotedge.service
   ```

9. 新增下列程式碼，以建立 TPM 環境變數。

   ```input
   [Service]
   Environment=IOTEDGE_USE_TPM_DEVICE=ON
   ```

9. 確認覆寫成功。

   ```bash
   sudo systemctl cat iotedge.service
   ```

   成功的輸出會顯示 **iotedge** 預設服務變數，然後顯示您在 **override.conf** 中設定的環境變數。 

12. 重新載入設定。

   ```bash
   sudo systemctl daemon-reload
   ```

## <a name="restart-the-iot-edge-runtime"></a>重新啟動 IoT Edge 執行階段

重新啟動 IoT Edge 執行階段，使其可取得您對裝置所做的所有組態變更。 

   ```bash
   sudo systemctl restart iotedge
   ```

確認 IoT Edge 執行階段正在執行。 

   ```bash
   sudo systemctl status iotedge
   ```

如果您看到佈建錯誤，可能是因為組態變更尚未生效。 請嘗試重新啟動 IoT Edge 精靈。 

   ```bash
   sudo systemctl daemon-reload
   ```
   
或者，請嘗試重新啟動虛擬機器，以確認變更是否會在重新啟動後生效。 

## <a name="verify-successful-installation"></a>確認安裝成功

如果執行階段順利啟動，您可以移至您的 IoT 中樞，並看到新裝置已自動佈建，且已可執行 IoT Edge 模組。 

檢查 IoT Edge 精靈的狀態。

```cmd/sh
systemctl status iotedge
```

檢查精靈記錄。

```cmd/sh
journalctl -u iotedge --no-pager --no-full
```

列出執行中的模組。

```cmd/sh
iotedge list
```


## <a name="next-steps"></a>後續步驟

佈建新裝置時，裝置佈建服務註冊程序可讓您同時設定裝置識別碼和裝置對應項標記。 您可以使用這些值來鎖定要使用自動裝置管理的個別裝置或裝置群組。 了解如何[使用 Azure 入口網站大規模部署和監視 IoT Edge 模組](how-to-deploy-monitor.md)，或[使用 Azure CLI](how-to-deploy-monitor-cli.md) 執行相同作業
