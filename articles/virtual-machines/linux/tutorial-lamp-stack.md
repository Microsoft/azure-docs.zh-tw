---
title: 教學課程 - 在 Azure 的 Linux 虛擬機器上部署 LAMP | Microsoft Docs
description: 在本教學課程中，您會了解如何在 Azure 中的 Linux 虛擬機器上安裝 LAMP 堆疊。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 6c12603a-e391-4d3e-acce-442dd7ebb2fe
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 11/27/2017
ms.author: danlep
ms.openlocfilehash: 5caed6fba607cb93a6168bded7531bc8bf63b9da
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970680"
---
# <a name="tutorial-install-a-lamp-web-server-on-a-linux-virtual-machine-in-azure"></a>教學課程：在 Azure 中的 Linux 虛擬機器上安裝 LAMP 網頁伺服器

本文會逐步引導您在 Azure 中的 Ubuntu VM 上部署 Apache 網頁伺服器、MySQL 和 PHP (LAMP 堆疊)。 如果您偏好使用 NGINX 網頁伺服器，請參閱 [LEMP 堆疊](tutorial-lemp-stack.md)教學課程。 若要查看作用中的 LAMP 伺服器，您可以選擇安裝及設定 WordPress 網站。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立 Ubuntu VM (LAMP 堆疊中的 'L')
> * 針對 Web 流量開啟連接埠 80
> * 安裝 Apache、MySQL 和 PHP
> * 驗證安裝和設定
> * 在 LAMP 伺服器上安裝 WordPress

此安裝程式適用於快速測試或概念證明。 如需 LAMP 堆疊的詳細資訊 (包括適用於生產環境的建議)，請參閱 [Ubuntu 文件](https://help.ubuntu.com/community/ApacheMySQLPHP)。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

[!INCLUDE [virtual-machines-linux-tutorial-stack-intro.md](../../../includes/virtual-machines-linux-tutorial-stack-intro.md)]

## <a name="install-apache-mysql-and-php"></a>安裝 Apache、MySQL 和 PHP

執行下列命令以更新 Ubuntu 套件來源，並安裝 Apache、MySQL 和 PHP。 請注意命令結尾的插入號 (^)，其為 `lamp-server^` 套件名稱的一部分。 


```bash
sudo apt update && sudo apt install lamp-server^
```


系統會提示您安裝套件和其他相依性。 出現提示時，請為 MySQL 設定根密碼，然後按 [Enter] 以繼續。 按照其餘的提示來進行。 此程序會安裝使用 PHP 搭配 MySQL 時所需的基本必要 PHP 擴充功能。 

![MySQL 根密碼頁面][1]

## <a name="verify-installation-and-configuration"></a>驗證安裝和設定


### <a name="apache"></a>Apache

使用下列命令檢查 Apache 的版本：
```bash
apache2 -v
```

安裝 Apache 後，且連接埠 80 對您的 VM 狀態為開啟，即可立即從網際網路存取網頁伺服器。 若要檢視 Apache2 Ubuntu 預設網頁，請開啟網頁瀏覽器，並輸入 VM 的公用 IP 位址。 使用您在透過 SSH 連線到 VM 時所使用的公用 IP 位址：

![Apache 預設網頁][3]


### <a name="mysql"></a>MySQL

使用下列命令檢查 MySQL 的版本 (請注意 `V` 參數是大寫)：

```bash
mysql -V
```

若要協助保護 MySQL 的安裝，請執行 `mysql_secure_installation` 指令碼。 如果您只要設定臨時伺服器，則可略過此步驟。

```bash
mysql_secure_installation
```

輸入 MySQL 的根密碼，並為您的環境設定安全性設定。

如果您想要試用 MySQL 功能 (建立 MySQL 資料庫、新增使用者或變更組態設定)，請登入 MySQL。 您不需要進行這個步驟也能完成本教學課程。

```bash
mysql -u root -p
```

完成後，輸入 `\q` 以結束 mysql 提示字元。

### <a name="php"></a>PHP

使用下列命令檢查 PHP 的版本：

```bash
php -v
```

如果您想要進一步測試，請建立要在瀏覽器中檢視的快速 PHP 資訊網頁。 下列命令會建立 PHP 資訊網頁：

```bash
sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
```

現在您可以檢查您所建立的 PHP 資訊網頁。 開啟瀏覽器並前往 `http://yourPublicIPAddress/info.php`。 替換為您 VM 的公用 IP 位址。 該頁面看起來應該類似下圖。

![PHP 資訊網頁][2]

[!INCLUDE [virtual-machines-linux-tutorial-wordpress.md](../../../includes/virtual-machines-linux-tutorial-wordpress.md)]


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已在 Azure 中部署 LAMP 伺服器。 您已了解如何︰

> [!div class="checklist"]
> * 建立 Ubuntu VM
> * 針對 Web 流量開啟連接埠 80
> * 安裝 Apache、MySQL 和 PHP
> * 驗證安裝和設定
> * 在 LAMP 伺服器上安裝 WordPress

前進到下一個教學課程，以了解如何使用 SSL 憑證保護 Web 伺服器。

> [!div class="nextstepaction"]
> [使用 SSL 保護網路伺服器](tutorial-secure-web-server.md)

[1]: ./media/tutorial-lamp-stack/configmysqlpassword-small.png
[2]: ./media/tutorial-lamp-stack/phpsuccesspage.png
[3]: ./media/tutorial-lamp-stack/apachesuccesspage.png