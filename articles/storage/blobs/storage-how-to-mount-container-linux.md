---
title: 如何在 Linux 上將 Azure Blob 儲存體掛接為檔案系統 | Microsoft Docs
description: 在 Linux 上使用 FUSE 掛接 Azure Blob 儲存體容器
services: storage
author: seguler
ms.service: storage
ms.topic: article
ms.date: 05/10/2018
ms.author: seguler
ms.openlocfilehash: 9964aa4d263e0b75eb59b4e1434a9b3f0aac6ea1
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39400181"
---
# <a name="how-to-mount-blob-storage-as-a-file-system-with-blobfuse"></a>如何使用 Blobfuse 將 Blob 儲存體掛接為檔案系統

## <a name="overview"></a>概觀
[Blobfuse](https://github.com/Azure/azure-storage-fuse) 是 Azure Blob 儲存體的虛擬檔案系統驅動程式，能讓您透過 Linux 檔案系統存取儲存體帳戶中現有的區塊 Blob 資料。 Azure Blob 儲存體是物件儲存體服務，因此沒有階層式的命名空間。 透過使用虛擬目錄配置，再將正斜線 '/' 當做分隔符號，Blobfuse 遂能提供此種命名空間。  

本指南示範如何使用 Blobfuse，以及如何在 Linux 上掛接 Blob 儲存體容器並存取資料。 若要深入了解 Blobfuse，請閱讀 [Blobfuse 存放庫](https://github.com/Azure/azure-storage-fuse)中的詳細資料。

> [!WARNING]
> Blobfuse 不保證 100% 合乎 POSIX 的規範，因為它只會將要求轉譯成 [Blob REST API](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api)。 例如，重新命名作業在 POSIX 中不可部分完成，不過在 Blobfuse 中並非如此。
> 如需原生檔案系統與 Blobfuse 之間差異的完整清單，請造訪 [Blobfuse 原始程式碼存放庫](https://github.com/azure/azure-storage-fuse)。
> 

## <a name="install-blobfuse-on-linux"></a>在 Linux 上安裝 Blobfuse
Blobfuse 二進位檔案位於 [Microsoft 的 Linux 軟體存放庫](https://docs.microsoft.com/windows-server/administration/Linux-Package-Repository-for-Microsoft-Software)中。 若要安裝 Blobfuse，請設定以下任一個存放庫。

### <a name="configure-the-microsoft-package-repository"></a>設定 Microsoft 封裝存放庫
設定 [Microsoft 產品的 Linux 封裝存放庫](https://docs.microsoft.com/windows-server/administration/Linux-Package-Repository-for-Microsoft-Software)。

以 Enterprise Linux 6 發行版為例：
```bash
sudo rpm -Uvh https://packages.microsoft.com/config/rhel/6/packages-microsoft-prod.rpm
```

同樣將 URL 變更為 `.../rhel/7/...`，以指向 Enterprise Linux 7 發行版。

以 Ubuntu 14.04 做為另一個範例：
```bash
wget https://packages.microsoft.com/config/ubuntu/14.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
```

同樣將 URL 變更為 `.../ubuntu/16.04/...`，以指向 Ubuntu 16.04 發行版。

### <a name="install-blobfuse"></a>安裝 Blobfuse

Ubuntu/Debian 發行版：
```bash
sudo apt-get install blobfuse
```

Enterprise Linux 發行版：
```bash
sudo yum install blobfuse
```

## <a name="prepare-for-mounting"></a>準備進行掛接
Blobfuse 在檔案系統中需有暫存路徑，才能緩衝及快取任何開啟的檔案，如此有助於提供近似原生的效能。 對於暫存路徑，請選擇效能最高的磁碟，或使用 Ramdisk 來獲得最佳效能。 

> [!NOTE]
> Blobfuse 會將所有開啟的檔案內容存放在暫存路徑。 請確認磁碟空間足夠容納所有開啟的檔案。 
> 

### <a name="optional-use-a-ramdisk-for-the-temporary-path"></a>(選擇性) 將 Ramdisk 當做暫存路徑
以下範例會建立 16 GB 的 Ramdisk，並建立 Blobfuse 的目錄。 請根據需求選擇大小。 Ramdisk 允許 Blobfuse 開啟大小達 16 GB 的檔案。 
```bash
sudo mount -t tmpfs -o size=16g tmpfs /mnt/ramdisk
sudo mkdir /mnt/ramdisk/blobfusetmp
sudo chown <youruser> /mnt/ramdisk/blobfusetmp
```

### <a name="use-an-ssd-for-temporary-path"></a>將 SSD 當做暫存路徑
在 Azure 中，您可以使用虛擬機器上的暫時磁碟 (SSD) 來當做低延遲的 Blobfuse 緩衝區。 在 Ubuntu 發行版中，此處的暫時磁碟會掛接於 '/mnt'，在 Red Hat 和 CentOS 發行版中則掛接於 '/mnt/resource/'。

請確認使用者擁有暫存路徑的存取權限：
```bash
sudo mkdir /mnt/resource/blobfusetmp
sudo chown <youruser> /mnt/resource/blobfusetmp
```

### <a name="configure-your-storage-account-credentials"></a>設定儲存體帳戶認證
Blobfuse 要求您使用以下格式將認證儲存在文字檔案中： 

```
accountName myaccount
accountKey myaccesskey==
containerName mycontainer
```

建立檔案後，請務必限制該檔案的存取權限，以禁止其他使用者讀取。
```bash
chmod 700 fuse_connection.cfg
```

### <a name="create-an-empty-directory-for-mounting"></a>建立空白的掛接目錄
```bash
mkdir ~/mycontainer
```

## <a name="mount"></a>掛接

> [!NOTE]
> 如需完整的掛接選項清單，請參閱 [Blobfuse 存放庫](https://github.com/Azure/azure-storage-fuse#mount-options)。  
> 

若要掛接 Blobfuse，請讓使用者執行以下命令。 該命令會將 '/path/to/fuse_connection.cfg' 指定的容器掛接到 '/mycontainer' 位置。

```bash
blobfuse ~/mycontainer --tmp-path=/mnt/resource/blobfusetmp  --config-file=/path/to/fuse_connection.cfg -o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120
```

您現在應該可以透過一般的檔案系統 API 存取區塊 Blob 了。 請注意，唯有掛接使用者才能存取掛接的目錄，如此可確保存取安全。 如果您想要允許所有使用者存取，請透過 ```-o allow_other``` 選項掛接。 

```bash
cd ~/mycontainer
mkdir test
echo "hello world" > test/blob.txt
```

## <a name="next-steps"></a>後續步驟

* [Blobfuse 首頁](https://github.com/Azure/azure-storage-fuse#blobfuse)
* [回報 Blobfuse 問題](https://github.com/Azure/azure-storage-fuse/issues) 

