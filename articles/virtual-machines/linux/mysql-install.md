---
title: 設定 Azure 中 Linux VM 上的 MySQL| Microsoft Docs
description: 了解如何在 Azure 中的 Linux 虛擬機器 (Ubuntu 或 Red Hat 系列 OS) 上安裝 MySQL 堆疊
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: 153bae7c-897b-46b3-bd86-192a6efb94fa
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 02/01/2016
ms.author: cynthn
ms.openlocfilehash: c8043064ac1df40eaa31ae56e9ec31c0152e0130
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37933530"
---
# <a name="how-to-install-mysql-on-azure"></a>如何在 Azure 上安裝 MySQL
本文將示範如何在執行 Linux 的 Azure 虛擬機器上安裝和設定 MySQL

[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

## <a name="install-mysql-on-your-virtual-machine"></a>在虛擬機器上安裝 MySQL
> [!NOTE]
> 您必須已經具有執行 Linux 的 Microsoft Azure 虛擬機器，才能完成本教學課程。 請參閱 [Azure Linux VM 教學課程](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)，建立並設定以 `mysqlnode` 為 VM 名稱，`azureuser` 為使用者的 Linux VM，然後再繼續進行下一步。
> 
> 

在此情況下，請使用連接埠 3306 作為 MySQL 連接埠。  

連接到您透過 putty 建立的 Linux VM。 如果這是您第一次使用 Azure Linux VM，請在[這裡](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)看看如何使用 putty 連接到 Linux VM。

本文的範例將使用儲存機制封裝安裝 MySQL5.6。 事實上，MySQL5.6 的效能改良功能比 MySQL5.5 更多。  詳細資訊請參閱 [這裡](http://www.mysqlperformanceblog.com/2013/02/18/is-mysql-5-6-slower-than-mysql-5-5/)。

### <a name="how-to-install-mysql56-on-ubuntu"></a>如何在 Ubuntu 上安裝 MySQL5.6
以下範例將使用 Azure 中採用 Ubuntu 的 Linux VM。

* 步驟 1：將 MySQL Server 5.6 參數安裝至 `root` 使用者：
  
            #[azureuser@mysqlnode:~]sudo su -
  
    安裝 mysql-server 5.6：
  
            #[root@mysqlnode ~]# apt-get update
            #[root@mysqlnode ~]# apt-get -y install mysql-server-5.6
  
    在安裝期間，會出現一個快顯對話方塊視窗，要求您設定下方的 MySQL 根密碼，請在這裡設定密碼。
  
    ![image](./media/mysql-install/virtual-machines-linux-install-mysql-p1.png)

    再次輸入密碼以進行確認。

    ![映像](./media/mysql-install/virtual-machines-linux-install-mysql-p2.png)

* 步驟 2：登入 MySQL Server
  
    MySQL Server 安裝完成之後，MySQL 服務便會自動啟動 。 您可以使用 `root` 使用者的身分登入 MySQL Server。
    使用下列命令以登入和輸入密碼。
  
             #[root@mysqlnode ~]# mysql -uroot -p
* 步驟 3：管理執行中的 MySQL 服務
  
    (a) 取得 MySQL 服務的狀態
  
             #[root@mysqlnode ~]# service mysql status
  
    (b) 啟動 MySQL 服務
  
             #[root@mysqlnode ~]# service mysql start
  
    (b) 停止 MySQL 服務
  
             #[root@mysqlnode ~]# service mysql stop
  
    (d) 重新啟動 MySQL 服務
  
             #[root@mysqlnode ~]# service mysql restart

### <a name="how-to-install-mysql-on-red-hat-os-family-like-centos-oracle-linux"></a>如何在 Redhat OS 系列 (例如 CentOS、Oracle Linux) 上安裝 MySQL
以下範例將使用執行 CentOS 或 Oracle Linux 的 Linux VM。

* 步驟 1：將 MySQL Yum 存放庫參數新增至 `root` 使用者：
  
            #[azureuser@mysqlnode:~]sudo su -
  
    下載並安裝 MySQL 發行套件：
  
            #[root@mysqlnode ~]# wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
            #[root@mysqlnode ~]# yum localinstall -y mysql-community-release-el6-5.noarch.rpm
* 步驟 2：編輯下方檔案，以允許 MySQL 儲存機制下載 MySQL5.6 套件檔案。
  
            #[root@mysqlnode ~]# vim /etc/yum.repos.d/mysql-community.repo
  
    將此檔案的每個值更新為下列值：
  
        \# *Enable to use MySQL 5.6*
  
        [mysql56-community]
        name=MySQL 5.6 Community Server
  
        baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/
  
        enabled=1
  
        gpgcheck=1
  
        gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
* 步驟 3：從 MySQL 存放庫安裝 MySQL 安裝 MySQL：
  
           #[root@mysqlnode ~]#yum install mysql-community-server
  
    將安裝 MySQL RPM 封裝和所有相關的封裝。
* 步驟 4：管理執行中的 MySQL 服務
  
    (a) 查看 MySQL 伺服器的服務狀態：
  
           #[root@mysqlnode ~]#service mysqld status
  
    (b) 查看 MySQL Server 的預設連接埠是否正在執行：
  
           #[root@mysqlnode ~]#netstat  –tunlp|grep 3306

    (c) 啟動 MySQL 伺服器：

           #[root@mysqlnode ~]#service mysqld start

    (d) 停止 MySQL 伺服器：

           #[root@mysqlnode ~]#service mysqld stop

    (e) 將 MySQL 設為在啟動系統啟動：

           #[root@mysqlnode ~]#chkconfig mysqld on


### <a name="how-to-install-mysql-on-suse-linux"></a>如何在 SUSE Linux 上安裝 MySQL
以下範例將使用執行 OpenSUSE 的 Linux VM。

* 步驟 1：下載及安裝 MySQL Server
  
    透過下列命令切換至 `root` 使用者：  
  
           #sudo su -
  
    下載及安裝 MySQL Server 封裝：
  
           #[root@mysqlnode ~]# zypper update
  
           #[root@mysqlnode ~]# zypper install mysql-server mysql-devel mysql
* 步驟 2：管理執行中的 MySQL 服務
  
    (a) 查看 MySQL 伺服器的狀態：
  
           #[root@mysqlnode ~]# rcmysql status
  
    (b) 查看 MySQL 伺服器的預設連接埠：
  
           #[root@mysqlnode ~]# netstat  –tunlp|grep 3306

    (c) 啟動 MySQL 伺服器：

           #[root@mysqlnode ~]# rcmysql start

    (d) 停止 MySQL 伺服器：

           #[root@mysqlnode ~]# rcmysql stop

    (e) 將 MySQL 設為在啟動系統啟動：

           #[root@mysqlnode ~]# insserv mysql

### <a name="next-step"></a>後續步驟
按一下 [這裡](https://www.mysql.com/)，了解更多 MySQL 使用方式和相關資訊。

