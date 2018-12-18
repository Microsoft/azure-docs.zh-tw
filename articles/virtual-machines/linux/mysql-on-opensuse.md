---
title: 在 Azure 中的 OpenSUSE VM 上安裝 MySQL | Microsoft Docs
description: 了解如何在 Azure 中的 OpenSUSE Linux 虛擬機器上安裝 MySQL。
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 1594e10e-c314-455a-9efb-a89441de364b
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/11/2018
ms.author: cynthn
ms.openlocfilehash: 838915e7a5fe1d1838c9bd802305e83ec9a773d9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986773"
---
# <a name="install-mysql-on-a-virtual-machine-running-opensuse-linux-in-azure"></a>在 Azure 中執行 OpenSUSE Linux 的虛擬機器上安裝 MySQL

[MySQL](http://www.mysql.com) 是一種很受歡迎的開放原始碼 SQL 資料庫。 本教學課程會示範如何建立執行 OpenSUSE Linux 的虛擬機器，然後安裝 MySQL。


[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，需要 Azure CLI 2.0 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-virtual-machine-running-opensuse-linux"></a>建立執行 OpenSUSE Linux 的虛擬機器

首先，建立資源群組。 在此範例中，資源群組的名稱為 mySQSUSEResourceGroup，並且會建立在「美國東部」區域。

```azurecli-interactive
az group create --name mySQLSUSEResourceGroup --location eastus
```

建立 VM。 在此範例中，VM 的名稱為 myVM，而 VM 的大小為 Standard_D2s_v3，但您應該選擇您認為最適合您工作負載的 [VM 大小](sizes.md)。

```azurecli-interactive
az vm create --resource-group mySQLSUSEResourceGroup \
   --name myVM \
   --image openSUSE-Leap \
   --size Standard_D2s_v3 \
   --generate-ssh-keys
```

您也需要將規則新增至網路安全性群組，以允許 MySQL 之連接埠 3306 的流量。

```azurecli-interactive
az vm open-port --port 3306 --resource-group mySQLSUSEResourceGroup --name myVM
```

## <a name="connect-to-the-vm"></a>連接至 VM

您將使用 SSH 連線至虛擬機器。 在此範例中，虛擬機器的公用 IP 位址是 *10.111.112.113*。 當您建立虛擬機器時，可以在輸出中看見 IP 位址。

```azurecli-interactive  
ssh 10.111.112.113
```

 
## <a name="update-the-vm"></a>更新虛擬機器
 
連線至虛擬機器之後，安裝系統更新和修補程式。 
   
```bash
sudo zypper update
```

依照提示更新您的虛擬機器。

## <a name="install-mysql"></a>安裝 MySQL 


透過 SSH 將 MySQL 安裝在虛擬機器中。 視情況回覆提示。

```bash
sudo zypper install mysql
```
 
將 MySQL 設為在啟動系統啟動。 

```bash
sudo systemctl enable mysql
```
確認已啟用 MySQL。

```bash
systemctl is-enabled mysql
```

這應該會傳回：enabled。

重新啟動伺服器。

```bash
sudo reboot
```


## <a name="mysql-password"></a>MySQL 密碼

安裝之後，MySQL 根密碼預設為空白。 執行 **mysql\_secure\_installation** 指令碼以保護 MySQL。 指令碼會提示您變更 MySQL 根密碼、移除匿名使用者帳戶、停用遠端根登入、移除測試資料庫，以及重新載入權限資料表。 

伺服器重新開機後，再次透過 SSH 連線至 VM。

```azurecli-interactive  
ssh 10.111.112.113
```



```bash
mysql_secure_installation
```

## <a name="sign-in-to-mysql"></a>登入 MySQL

您現在可以登入，並輸入 MySQL 提示。

```bash  
mysql -u root -p
```
這會切換到 MySQL 提示，您可以在該提示中發出 SQL 陳述式，與資料庫進行互動。

現在，建立新的 MySQL 使用者。

```   
CREATE USER 'mysqluser'@'localhost' IDENTIFIED BY 'password';
```
   
行尾的分號 (;) 對結束命令而言十分重要。


## <a name="create-a-database"></a>建立資料庫


建立資料庫，並授與 `mysqluser` 使用者權限。

```   
CREATE DATABASE testdatabase;
GRANT ALL ON testdatabase.* TO 'mysqluser'@'localhost' IDENTIFIED BY 'password';
```
   
只有連線到資料庫的指令碼會使用資料庫使用者名稱和密碼。  資料庫使用者帳戶名稱不一定代表系統上的實際使用者帳戶。

啟用從另一部電腦登入。 在此範例中，允許從中登入的電腦 IP 位址是 10.112.113.114。

```   
GRANT ALL ON testdatabase.* TO 'mysqluser'@'10.112.113.114' IDENTIFIED BY 'password';
```
   
若要結束 MySQL 資料庫管理公用程式，請輸入：

```    
quit
```


## <a name="next-steps"></a>後續步驟
如需 MySQL 的詳細資料，請參閱 [MySQL 文件](http://dev.mysql.com/doc/index-topic.html)。




