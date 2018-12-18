---
title: 建立 Linux VHD 並上傳至 Azure | Microsoft Docs
description: 使用傳統部署模型來建立及上傳包含 Linux 作業系統的 Azure 虛擬硬碟 (VHD)。
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-service-management
ROBOTS: NOINDEX
ms.assetid: 8058ff98-db03-4309-9bf4-69842bd64dd4
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 11/28/2016
ms.author: cynthn
ms.openlocfilehash: cdbe6aa5683ecf9d8bdaf6bbf9503ddc455f03ee
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37928262"
---
# <a name="creating-and-uploading-a-virtual-hard-disk-that-contains-the-linux-operating-system"></a>建立及上傳包含 Linux 作業系統的虛擬硬碟
> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 您也可以[使用 Azure Resource Manager 上傳自訂磁碟映像](../upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

本文說明如何建立及上傳虛擬硬碟 (VHD)，以便用它做為您自己的映像，在 Azure 中建立虛擬機器。 了解如何準備作業系統，以便使用它根據該映像建立多部虛擬機器。 


## <a name="prerequisites"></a>先決條件
本文假設您具有下列項目：

* **以 .vhd 檔案安裝的 Linux 作業系統** - 您已經以 VHD 格式將 [Azure 背書的 Linux 散發套件](../endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (或參閱[非背書散發套件的資訊](../create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)) 安裝到虛擬磁碟。 有多項工具可用來建立 VM 和 VHD：
  * 安裝和設定 [QEMU](https://en.wikibooks.org/wiki/QEMU/Installing_QEMU) 或 [KVM](http://www.linux-kvm.org/page/RunningKVM)，並小心使用 VHD 做為您的映像格式。 如有需要，您可以使用 `qemu-img convert` 來[轉換映像](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats)。
  * 您也可以在 [Windows 10](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install) 或 [Windows Server 2012/2012 R2](https://technet.microsoft.com/library/hh846766.aspx) 上使用 Hyper-V。

> [!NOTE]
> Azure 不支援較新的 VHDX 格式。 當您建立 VM 時，請指定 VHD 做為格式。 如有需要，您可以使用 [`qemu-img convert`](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats) 或 [`Convert-VHD`](https://technet.microsoft.com/library/hh848454.aspx) PowerShell Cmdlet 將 VHDX 磁碟轉換為 VHD。 此外，Azure 不支援上傳動態 VHD，因此您必須將此類磁碟轉換成靜態 VHD 再上傳。 您可以在上傳至 Azure 的期間使用 [適用於 GO 的 Azure VHD 公用程式](https://github.com/Microsoft/azure-vhd-utils-for-go) 之類的工具來轉換動態磁碟。

* **Azure 命令列介面** - 安裝最新的 [Azure 命令列介面](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2) 來上傳 VHD。

<a id="prepimage"> </a>

## <a name="step-1-prepare-the-image-to-be-uploaded"></a>步驟 1：準備要上傳的映像
Azure 支援各種 Linux 散發套件 (請參閱 [背書的散發套件](../endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json))。 下列文章會逐步引導您了解如何準備 Azure 上支援的各種 Linux 散發套件： 完成下列指南中的步驟並已有一個準備好可上傳到 Azure 的 VHD 檔案之後，請回到這裡：

* **[CentOS 型散發套件](../create-upload-centos.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Debian Linux](../debian-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Oracle Linux](../oracle-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Red Hat Enterprise Linux](../redhat-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[SLES 和 openSUSE](../suse-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Ubuntu](../create-upload-ubuntu.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[其他：非背書散發套件](../create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**

> [!NOTE]
> 只有使用其中一個背書散發套件搭配[經 Azure 背書之配送映像上的 Linux](../endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中＜支援的版本＞底下指定的組態詳細資料時，Azure 平台 SLA 才適用於執行 Linux OS 的虛擬機器。 Azure 映像庫中的所有 Linux 散發套件，皆為使用必要組態的背書散發套件。
> 
> 

如需有關為 Azure 準備 Linux 映像的更多一般秘訣，另請參閱 **[Linux 安裝注意事項](../create-upload-generic.md#general-linux-installation-notes)**。

<a id="connect"> </a>

## <a name="step-2-prepare-the-connection-to-azure"></a>步驟 2：準備與 Azure 的連接
確定您是在傳統部署模型中使用 Azure CLI (`azure config mode asm`)，然後登入您的帳戶︰

```azurecli
azure login
```


<a id="upload"> </a>

## <a name="step-3-upload-the-image-to-azure"></a>步驟 3：將映像上傳至 Azure
您需要一個可供上傳 VHD 檔案的儲存體帳戶。 您可以選取現有的儲存體帳戶或[建立新帳戶](../../../storage/common/storage-create-storage-account.md)。

使用 Azure CLI 上傳映像，方法是使用下列命令：

```azurecli
azure vm image create <ImageName> `
    --blob-url <BlobStorageURL>/<YourImagesFolder>/<VHDName> `
    --os Linux <PathToVHDFile>
```

在上述範例中︰

* **BlobStorageURL** 是您打算使用的儲存體帳戶 URL
* **YourImagesFolder** 是 Blob 儲存體內您要用來儲存映像的容器
* **VHDName** 是入口網站中用來識別虛擬硬碟的顯示標籤。
* **PathToVHDFile** 是 .vhd 檔案在您電腦上的完整路徑和名稱。

以下顯示完整的範例：

```azurecli
azure vm image create myImage `
    --blob-url https://mystorage.blob.core.windows.net/vhds/myimage.vhd `
    --os Linux /home/ahmet/myimage.vhd
```

## <a name="step-4-create-a-vm-from-the-image"></a>步驟 4：從映像建立 VM
您使用 `azure vm create` 以和一般 VM 相同的方式建立 VM。 指定您在前一個步驟中提供給映像的名稱。 在下列範例中，我們會使用在前一個步驟中提供的 **myImage** 映像名稱：

```azurecli
azure vm create --userName ops --password P@ssw0rd! --vm-size Small --ssh `
    --location "West US" "myDeployedVM" myImage
```

若要建立您自己的 VM，請提供您自己的使用者名稱 + 密碼、位置、DNS 名稱，以及映像名稱。

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱 [Azure 傳統部署模型的 Azure CLI 參考](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)。

[Step 1: Prepare the image to be uploaded]:#prepimage
[Step 2: Prepare the connection to Azure]:#connect
[Step 3: Upload the image to Azure]:#upload
