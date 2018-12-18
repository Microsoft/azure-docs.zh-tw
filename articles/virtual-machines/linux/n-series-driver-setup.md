---
title: 適用於 Linux 的 Azure N 系列 GPU 驅動程式設定 | Microsoft Docs
description: 如何針對 Azure 中執行 Linux 的 N 系列 VM 設定 NVIDIA GPU 驅動程式
services: virtual-machines-linux
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: d91695d0-64b9-4e6b-84bd-18401eaecdde
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 822261e74f7da941ac89090e5d493c4be18bc307
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47038879"
---
# <a name="install-nvidia-gpu-drivers-on-n-series-vms-running-linux"></a>在執行 Linux 的 N 系列 VM 上安裝 NVIDIA GPU 驅動程式

若要利用 Azure N 系列 VM (執行 Linux) 的 GPU 功能，您必須安裝 NVIDIA GPU 驅動程式。 [NVIDIA GPU 驅動程式擴充功能](../extensions/hpccompute-gpu-linux.md)會在 N 系列 VM 上安裝適當的 NVIDIA CUDA 或 GRID 驅動程式。 使用 Azure 入口網站或者 Azure CLI 或 Azure Resource Manager 範本之類的工具，安裝或管理擴充功能。 如需支援的發佈和部署步驟，請參閱 [NVIDIA GPU 驅動程式擴充功能文件](../extensions/hpccompute-gpu-linux.md)。

如果您選擇手動安裝 GPU 驅動程式，本文提供支援的發佈、驅動程式，以及安裝和驗證步驟。 驅動程式手動設定資訊也適用於 [Windows VM](../windows/n-series-driver-setup.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

如需 N 系列 VM 規格、儲存體容量與磁碟的詳細資料，請參閱 [GPU Linux VM 大小](sizes-gpu.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 

[!INCLUDE [virtual-machines-n-series-linux-support](../../../includes/virtual-machines-n-series-linux-support.md)]

## <a name="install-cuda-drivers-on-n-series-vms"></a>在 N 系列 VM 上安裝 CUDA 驅動程式

以下是從 N 系列 VM 上的 NVIDIA CUDA Toolkit 安裝 CUDA 驅動程式的步驟。 


C 和 C++ 開發人員可以選擇性地安裝完整 Toolkit，以建置 GPU 加速的應用程式。 如需詳細資訊，請參閱 [CUDA 安裝指南](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)。

若要安裝 CUDA 驅動程式，請透過 SSH 連線至每部 VM。 若要確認系統有 CUDA 功能的 GPU，請執行下列命令︰

```bash
lspci | grep -i NVIDIA
```
您會看到類似下列範例 (顯示 NVIDIA Tesla K80 卡) 的輸出︰

![lspci 命令輸出](./media/n-series-driver-setup/lspci.png)

然後執行您的配送映像特有的安裝命令。

### <a name="ubuntu-1604-lts"></a>Ubuntu 16.04 LTS

1. 下載並安裝 CUDA 驅動程式。
  ```bash
  CUDA_REPO_PKG=cuda-repo-ubuntu1604_10.0.130-1_amd64.deb

  wget -O /tmp/${CUDA_REPO_PKG} http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/${CUDA_REPO_PKG} 

  sudo dpkg -i /tmp/${CUDA_REPO_PKG}

  sudo apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub 

  rm -f /tmp/${CUDA_REPO_PKG}

  sudo apt-get update

  sudo apt-get install cuda-drivers

  ```

  安裝可能需要數分鐘的時間。

2. 若要選擇性地安裝完整的 CUDA Toolkit，請輸入︰

  ```bash
  sudo apt-get install cuda
  ```

3. 重新啟動 VM 並繼續確認安裝。

#### <a name="cuda-driver-updates"></a>CUDA 驅動程式更新

我們建議您在部署後定期更新 CUDA 驅動程式。

```bash
sudo apt-get update

sudo apt-get upgrade -y

sudo apt-get dist-upgrade -y

sudo apt-get install cuda-drivers

sudo reboot
```

### <a name="centos-or-red-hat-enterprise-linux-73-or-74"></a>CentOS 或 Red Hat Enterprise Linux 7.3 或 7.4

1. 更新核心 (建議)。 若您選擇不要更新核心，請務必確認 `kernel-devel` 和 `dkms` 版本適合您的核心。

  ```
  sudo yum install kernel kernel-tools kernel-headers kernel-devel
  
  sudo reboot

2. Install the latest [Linux Integration Services for Hyper-V and Azure](https://www.microsoft.com/download/details.aspx?id=55106).

  ```bash
  wget https://aka.ms/lis
 
  tar xvzf lis
 
  cd LISISO
 
  sudo ./install.sh
 
  sudo reboot
  ```
 
3. 重新連線至 VM，並使用下列命令繼續安裝：

  ```bash
  sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

  sudo yum install dkms

  CUDA_REPO_PKG=cuda-repo-rhel7-10.0.130-1.x86_64.rpm

  wget http://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/${CUDA_REPO_PKG} -O /tmp/${CUDA_REPO_PKG}

  sudo rpm -ivh /tmp/${CUDA_REPO_PKG}

  rm -f /tmp/${CUDA_REPO_PKG}

  sudo yum install cuda-drivers
  ```

  安裝可能需要數分鐘的時間。 

4. 若要選擇性地安裝完整的 CUDA Toolkit，請輸入︰

  ```bash
  sudo yum install cuda
  ```

5. 重新啟動 VM 並繼續確認安裝。

### <a name="verify-driver-installation"></a>確認驅動程式安裝

若要查詢 GPU 裝置狀態，請透過 SSH 連線至 VM 並執行與驅動程式一起安裝的 [nvidia-smi](https://developer.nvidia.com/nvidia-system-management-interface) 命令列公用程式。 

如果已安裝驅動程式，您會看到類似以下的輸出。 請注意，除非您正在 VM 上執行 GPU 工作負載，否則 [GPU-Util] 會顯示 0%。 您的驅動程式版本和 GPU 詳細資料可能會與顯示的不同。

![NVIDIA 裝置狀態](./media/n-series-driver-setup/smi.png)

## <a name="rdma-network-connectivity"></a>RDMA 網路連線

可以在支援 RDMA 的 N 系列 VM (例如部署在同一個可用性設定組或 VM 擴展集的單一放置群組中的 NC24r) 上啟用 RDMA 網路連線能力。 RDMA 網路可針對搭配 Intel MPI 5.x 或更新版本執行的應用程式，支援訊息傳遞介面 (MPI) 流量。 其他需求如下：

### <a name="distributions"></a>散發

從 Azure Marketplace 中支援 N 系列 VM 上 RDMA 連線的其中一個映像，部署支援 RDMA 的 N 系列 VM：
  
* **Ubuntu 16.04 LTS** - 設定 VM 上的 RDMA 驅動程式，並向 Intel 註冊以下載 Intel MPI：

  [!INCLUDE [virtual-machines-common-ubuntu-rdma](../../../includes/virtual-machines-common-ubuntu-rdma.md)]

* **CentOS 型 7.4 HPC** - 已在 VM 上安裝 RDMA 驅動程式和 Intel MPI 5.1。

## <a name="install-grid-drivers-on-nv-or-nvv2-series-vms"></a>在 NV 或 NVv2 系列 VM 上安裝 GRID 驅動程式

若要在 NV 或 NVv2 系列 VM 上安裝 NVIDIA GRID 驅動程式，請以 SSH 連線至每個 VM，並遵循您 Linux 發行版本的步驟。 

### <a name="ubuntu-1604-lts"></a>Ubuntu 16.04 LTS

1. 執行 `lspci` 命令。 請確認 NVIDIA M60 卡可以顯示為 PCI 裝置。

2. 安裝更新。

  ```bash
  sudo apt-get update

  sudo apt-get upgrade -y

  sudo apt-get dist-upgrade -y

  sudo apt-get install build-essential ubuntu-desktop -y
  ```
3. 停用與 NVIDIA 驅動程式不相容的 Nouveau 核心驅動程式。 (僅在 NV 或 NVv2 VM 上使用 NVIDIA 驅動程式。)若要這樣做，使用下列內容，在名為 `nouveau.conf` 的 `/etc/modprobe.d ` 中建立檔案︰

  ```
  blacklist nouveau

  blacklist lbm-nouveau
  ```


4. 重新啟動 VM，並重新連線。 結束 X 伺服器：

  ```bash
  sudo systemctl stop lightdm.service
  ```

5. 下載並安裝 GRID 驅動程式：

  ```bash
  wget -O NVIDIA-Linux-x86_64-grid.run https://go.microsoft.com/fwlink/?linkid=849941  

  chmod +x NVIDIA-Linux-x86_64-grid.run

  sudo ./NVIDIA-Linux-x86_64-grid.run
  ``` 

6. 當系統詢問您是否要執行 nvidia-xconfig 公用程式來更新您的 X 組態檔時，選取 [是]。

7. 安裝完成後，請將 /etc/nvidia/gridd.conf.template 複製到位於 /etc/nvidia/ 的新檔案 gridd.conf

  ```bash
  sudo cp /etc/nvidia/gridd.conf.template /etc/nvidia/gridd.conf
  ```

8. 將下面內容新增至 `/etc/nvidia/gridd.conf`：
 
  ```
  IgnoreSP=FALSE
  ```
9. 重新啟動 VM 並繼續確認安裝。


### <a name="centos-or-red-hat-enterprise-linux"></a>CentOS 或 Red Hat Enterprise Linux 

1. 更新核心和 DKMS (建議)。 若您選擇不要更新核心，請務必確認 `kernel-devel` 和 `dkms` 版本適合您的核心。
 
  ```bash  
  sudo yum update
 
  sudo yum install kernel-devel
 
  sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
 
  sudo yum install dkms
  ```

2. 停用與 NVIDIA 驅動程式不相容的 Nouveau 核心驅動程式。 (僅在 NV 或 NV2 VM 上使用 NVIDIA 驅動程式。)若要這樣做，使用下列內容，在名為 `nouveau.conf` 的 `/etc/modprobe.d ` 中建立檔案︰

  ```
  blacklist nouveau

  blacklist lbm-nouveau
  ```
 
3. 重新啟動 VM、重新連線，然後安裝最新的[適用於 Hyper-V 和 Azure 的 Linux 整合服務](https://www.microsoft.com/download/details.aspx?id=55106)。
 
  ```bash
  wget https://aka.ms/lis

  tar xvzf lis

  cd LISISO

  sudo ./install.sh

  sudo reboot

  ```
 
4. 重新連線至 VM 並執行 `lspci` 命令。 請確認 NVIDIA M60 卡可以顯示為 PCI 裝置。
 
5. 下載並安裝 GRID 驅動程式：

  ```bash
  wget -O NVIDIA-Linux-x86_64-grid.run https://go.microsoft.com/fwlink/?linkid=849941  

  chmod +x NVIDIA-Linux-x86_64-grid.run

  sudo ./NVIDIA-Linux-x86_64-grid.run
  ``` 
6. 當系統詢問您是否要執行 nvidia-xconfig 公用程式來更新您的 X 組態檔時，選取 [是]。

7. 安裝完成後，請將 /etc/nvidia/gridd.conf.template 複製到位於 /etc/nvidia/ 的新檔案 gridd.conf
  
  ```bash
  sudo cp /etc/nvidia/gridd.conf.template /etc/nvidia/gridd.conf
  ```
  
8. 將下面內容新增至 `/etc/nvidia/gridd.conf`：
 
  ```
  IgnoreSP=FALSE
  ```
9. 重新啟動 VM 並繼續確認安裝。

### <a name="verify-driver-installation"></a>確認驅動程式安裝


若要查詢 GPU 裝置狀態，請透過 SSH 連線至 VM 並執行與驅動程式一起安裝的 [nvidia-smi](https://developer.nvidia.com/nvidia-system-management-interface) 命令列公用程式。 

如果已安裝驅動程式，您會看到類似以下的輸出。 請注意，除非您正在 VM 上執行 GPU 工作負載，否則 [GPU-Util] 會顯示 0%。 您的驅動程式版本和 GPU 詳細資料可能會與顯示的不同。

![NVIDIA 裝置狀態](./media/n-series-driver-setup/smi-nv.png)
 

### <a name="x11-server"></a>X11 伺服器
如果您需要 X11 伺服器以遠端連線到 NV 或 NVv2 VM，建議使用 [x11vnc](http://www.karlrunge.com/x11vnc/)，因為它允許圖形硬體加速。 M60 裝置的 BusID 必須手動加入至 X11 組態檔 (通常是 `etc/X11/xorg.conf`)。 新增類似下列的 `"Device"` 區段：
 
```
Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    BoardName      "Tesla M60"
    BusID          "PCI:0@your-BusID:0:0"
EndSection
```
 
此外，更新您的 `"Screen"` 區段，即可使用此裝置。
 
可以找到十進位 BusID，方法為執行

```bash
nvidia-xconfig --query-gpu-info | awk '/PCI BusID/{print $4}'
```
 
重新配置或重新啟動 VM 時，可以變更 BusID。 因此，重新啟動 VM 時，建議您建立指令碼來更新 X11 設定中的 BusID。 例如，建立名為 `busidupdate.sh` (或您選擇的其他名稱) 且具有類似下列內容的指令碼：

```bash 
#!/bin/bash
XCONFIG="/etc/X11/xorg.conf"
OLDBUSID=`awk '/BusID/{gsub(/"/, "", $2); print $2}' ${XCONFIG}`
NEWBUSID=`nvidia-xconfig --query-gpu-info | awk '/PCI BusID/{print $4}'`

if [[ "${OLDBUSID}" == "${NEWBUSID}" ]] ; then
        echo "NVIDIA BUSID not changed - nothing to do"
else
        echo "NVIDIA BUSID changed from \"${OLDBUSID}\" to \"${NEWBUSID}\": Updating ${XCONFIG}" 
        sed -e 's|BusID.*|BusID          '\"${NEWBUSID}\"'|' -i ${XCONFIG}
fi
```

接著，在 `/etc/rc.d/rc3.d` 中建立更新指令碼的項目，如此在開機時，會以 root 的身分叫用指令碼。

## <a name="troubleshooting"></a>疑難排解

* 您可以使用 `nvidia-smi` 設定持續性模式，如此當您需要查詢卡片時，命令的輸出更快。 若要設定持續性模式，請執行 `nvidia-smi -pm 1`。 請注意，如果重新啟動 VM，模式設定就會消失。 您一律可以編寫指令碼在啟動時執行模式設定。

## <a name="next-steps"></a>後續步驟

* 若要擷取已安裝 NVIDIA 驅動程式的 Linux VM 映像，請參閱[如何一般化和擷取 Linux 虛擬機器](capture-image.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
