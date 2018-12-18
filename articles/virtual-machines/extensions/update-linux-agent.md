---
title: 從 GitHub 更新 Azure Linux 代理程式 | Microsoft Docs
description: 了解如何更新 Azure 中 Linux VM 的 Azure Linux 代理程式
services: virtual-machines-linux
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: f1f19300-987d-4f29-9393-9aba866f049c
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 08/02/2017
ms.author: roiyz
ms.openlocfilehash: 0926859037c358aaf94ec51d9614cc9c02629e7f
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451930"
---
# <a name="how-to-update-the-azure-linux-agent-on-a-vm"></a>如何更新虛擬機器上的 Azure Linux 代理程式

若要更新 Azure 中 Linux VM 的 [Azure Linux 代理程式](https://github.com/Azure/WALinuxAgent) ，您必須準備：

- 在 Azure 中執行的 Linux VM。
- 對該 Linux VM 的 SSH 連線。

需一律先檢查 Linux 散發版本儲存機制中的封裝。 可用的封裝有可能不是最新的版本，啟用自動更新可確保 Linux 代理程式一律更新為最新版本。 如果使用封裝管理員安裝時碰到問題，請向散發版本廠商尋求支援。

## <a name="minimum-virtual-machine-agent-support-in-azure"></a>在 Azure 中支援的最小虛擬機器代理程式
在繼續之前，確認[在 Azure 中針對虛擬機器代理程式的最小版本支援](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support)。

## <a name="updating-the-azure-linux-agent"></a>更新 Azure Linux 代理程式

## <a name="ubuntu"></a>Ubuntu

#### <a name="check-your-current-package-version"></a>檢查目前的封裝版本

```bash
apt list --installed | grep walinuxagent
```

#### <a name="update-package-cache"></a>更新封裝快取

```bash
sudo apt-get -qq update
```

#### <a name="install-the-latest-package-version"></a>安裝最新版本的封裝

```bash
sudo apt-get install walinuxagent
```

#### <a name="ensure-auto-update-is-enabled"></a>確定已啟用自動更新

首先，請檢查是否已啟用：

```bash
cat /etc/waagent.conf
```

尋找「AutoUpdate.Enabled」。 如果看到此輸出結果，則表示已啟用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要啟用執行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

### <a name="restart-the-waagent-service"></a>重新啟動 waagent 服務

#### <a name="restart-agent-for-1404"></a>重新啟動 14.04 的代理程式

```bash
initctl restart walinuxagent
```

#### <a name="restart-agent-for-1604--1704"></a>重新啟動 16.04 / 17.04 的代理程式

```bash
systemctl restart walinuxagent.service
```

## <a name="debian"></a>Debian

### <a name="debian-7-wheezy"></a>Debian 7 “Wheezy”

#### <a name="check-your-current-package-version"></a>檢查目前的封裝版本

```bash
dpkg -l | grep waagent
```

#### <a name="update-package-cache"></a>更新封裝快取

```bash
sudo apt-get -qq update
```

#### <a name="install-the-latest-package-version"></a>安裝最新版本的封裝

```bash
sudo apt-get install waagent
```

#### <a name="enable-agent-auto-update"></a>啟用代理程式自動更新
這一版本的 Debian 沒有高於或等於 2.0.16 的版本，因此無法使用自動更新。 上述命令的輸出結果會顯示封裝是否為最新的狀態。

### <a name="debian-8-jessie--debian-9-stretch"></a>Debian 8 “Jessie” / Debian 9 “Stretch”

#### <a name="check-your-current-package-version"></a>檢查目前的封裝版本

```bash
apt list --installed | grep waagent
```

#### <a name="update-package-cache"></a>更新封裝快取

```bash
sudo apt-get -qq update
```

#### <a name="install-the-latest-package-version"></a>安裝最新版本的封裝

```bash
sudo apt-get install waagent
```
#### <a name="ensure-auto-update-is-enabled"></a>確定已啟用自動更新 

首先，請檢查是否已啟用：

```bash
cat /etc/waagent.conf
```

尋找「AutoUpdate.Enabled」。 如果看到此輸出結果，則表示已啟用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要啟用執行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

### <a name="restart-the-waagent-service"></a>重新啟動 waagent 服務

```
sudo systemctl restart walinuxagent.service
```

## <a name="red-hat--centos"></a>Red Hat / CentOS

### <a name="rhelcentos-6"></a>RHEL/CentOS 6

#### <a name="check-your-current-package-version"></a>檢查目前的封裝版本

```bash
sudo yum list WALinuxAgent
```

#### <a name="check-available-updates"></a>檢查可用的更新

```bash
sudo yum check-update WALinuxAgent
```

#### <a name="install-the-latest-package-version"></a>安裝最新版本的封裝

```bash
sudo yum install WALinuxAgent
```

#### <a name="ensure-auto-update-is-enabled"></a>確定已啟用自動更新 

首先，請檢查是否已啟用：

```bash
cat /etc/waagent.conf
```

尋找「AutoUpdate.Enabled」。 如果看到此輸出結果，則表示已啟用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要啟用執行：

```bash
sudo sed -i 's/\# AutoUpdate.Enabled=y/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

### <a name="restart-the-waagent-service"></a>重新啟動 waagent 服務

```
sudo service waagent restart
```

### <a name="rhelcentos-7"></a>RHEL/CentOS 7

#### <a name="check-your-current-package-version"></a>檢查目前的封裝版本

```bash
sudo yum list WALinuxAgent
```

#### <a name="check-available-updates"></a>檢查可用的更新

```bash
sudo yum check-update WALinuxAgent
```

#### <a name="install-the-latest-package-version"></a>安裝最新版本的封裝

```bash
sudo yum install WALinuxAgent  
```

#### <a name="ensure-auto-update-is-enabled"></a>確定已啟用自動更新 

首先，請檢查是否已啟用：

```bash
cat /etc/waagent.conf
```

尋找「AutoUpdate.Enabled」。 如果看到此輸出結果，則表示已啟用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要啟用執行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

### <a name="restart-the-waagent-service"></a>重新啟動 waagent 服務

```bash
sudo systemctl restart waagent.service
```

## <a name="suse-sles"></a>SUSE SLES

### <a name="suse-sles-11-sp4"></a>SUSE SLES 11 SP4

#### <a name="check-your-current-package-version"></a>檢查目前的封裝版本

```bash
zypper info python-azure-agent
```

#### <a name="check-available-updates"></a>檢查可用的更新

上述的輸出結果會顯示封裝是否為最新的狀態。

#### <a name="install-the-latest-package-version"></a>安裝最新版本的封裝

```bash
sudo zypper install python-azure-agent
```

#### <a name="ensure-auto-update-is-enabled"></a>確定已啟用自動更新 

首先，請檢查是否已啟用：

```bash
cat /etc/waagent.conf
```

尋找「AutoUpdate.Enabled」。 如果看到此輸出結果，則表示已啟用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要啟用執行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

### <a name="restart-the-waagent-service"></a>重新啟動 waagent 服務

```bash
sudo /etc/init.d/waagent restart
```

### <a name="suse-sles-12-sp2"></a>SUSE SLES 12 SP2

#### <a name="check-your-current-package-version"></a>檢查目前的封裝版本

```bash
zypper info python-azure-agent
```

#### <a name="check-available-updates"></a>檢查可用的更新

在上述的輸出結果中，會顯示封裝是否為最新的狀態。

#### <a name="install-the-latest-package-version"></a>安裝最新版本的封裝

```bash
sudo zypper install python-azure-agent
```

#### <a name="ensure-auto-update-is-enabled"></a>確定已啟用自動更新 

首先，請檢查是否已啟用：

```bash
cat /etc/waagent.conf
```

尋找「AutoUpdate.Enabled」。 如果看到此輸出結果，則表示已啟用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要啟用執行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

### <a name="restart-the-waagent-service"></a>重新啟動 waagent 服務

```bash
sudo systemctl restart waagent.service
```

## <a name="oracle-6-and-7"></a>Oracle 6 和 7

針對 Oracle Linux，請確定已啟用 `Addons` 儲存機制。 視您的情況，編輯檔案 `/etc/yum.repos.d/public-yum-ol6.repo`(Oracle Linux 6) 或 `/etc/yum.repos.d/public-yum-ol7.repo`(Oracle Linux)，將此檔案中 **[ol6_addons]** 或 **[ol7_addons]** 底下的 `enabled=0` 一行變更為 `enabled=1`。

接下來，若要安裝最新版的 Azure Linux 代理程式，請輸入：

```bash
sudo yum install WALinuxAgent
```

如果找不到附加元件儲存機制，只需根據您的 Oracle Linux 版本，將這幾行加入 .repo 檔案結尾處︰

Oracle Linux 6 虛擬機器︰

```sh
[ol6_addons]
name=Add-Ons for Oracle Linux $releasever ($basearch)
baseurl=http://public-yum.oracle.com/repo/OracleLinux/OL6/addons/x86_64
gpgkey=http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6
gpgcheck=1
enabled=1
```

Oracle Linux 7 虛擬機器︰

```sh
[ol7_addons]
name=Oracle Linux $releasever Add ons ($basearch)
baseurl=http://public-yum.oracle.com/repo/OracleLinux/OL7/addons/$basearch/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
gpgcheck=1
enabled=1
```

然後輸入：

```bash
sudo yum update WALinuxAgent
```

通常這就是您需要執行的所有步驟，但如果因為其他原因造成您必須直接從 https://github.com 安裝，請執行以下步驟。


## <a name="update-the-linux-agent-when-no-agent-package-exists-for-distribution"></a>若沒有代理程式封裝可發佈，請更新 Linux 代理程式

在命令列輸入 `sudo yum install wget` 來安裝 wget (有一些發行版本預設為不安裝，例如 Red Hat、CentOS、Oracle Linux 6.4 和 6.5 版)。

### <a name="1-download-the-latest-version"></a>1.下載最新版本
在網頁中開啟 [Github 中的 Azure Linux 代理程式版本](https://github.com/Azure/WALinuxAgent/releases) ，然後查明最新的版本號碼。 (您可以輸入 `waagent --version`，即可找到目前的版本 )。

#### <a name="for-version-22x-or-later-type"></a>針對 2.2.x 版本或較新版本，請輸入：
```bash
wget https://github.com/Azure/WALinuxAgent/archive/v2.2.x.zip
unzip v2.2.x.zip
cd WALinuxAgent-2.2.x
```

下列一行使用 2.2.0 版做為範例：

```bash
wget https://github.com/Azure/WALinuxAgent/archive/v2.2.14.zip
unzip v2.2.14.zip  
cd WALinuxAgent-2.2.14
```

### <a name="2-install-the-azure-linux-agent"></a>2.安裝 Azure Linux 代理程式

#### <a name="for-version-22x-use"></a>針對 2.2.x 版本，請使用：
您可能需要先安裝套件 `setuptools` ，請參閱 [這裡](https://pypi.python.org/pypi/setuptools)。 然後，執行：

```bash
sudo python setup.py install
```

#### <a name="ensure-auto-update-is-enabled"></a>確定已啟用自動更新

首先，請檢查是否已啟用：

```bash
cat /etc/waagent.conf
```

尋找「AutoUpdate.Enabled」。 如果看到此輸出結果，則表示已啟用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要啟用執行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

### <a name="3-restart-the-waagent-service"></a>3.重新啟動 waagent 服務
針對大多數的 Linux 散發版本：

```bash
sudo service waagent restart
```

針對 Ubuntu，使用：

```bash
sudo service walinuxagent restart
```

針對 CoreOS，請使用：

```bash
sudo systemctl restart waagent
```

### <a name="4-confirm-the-azure-linux-agent-version"></a>4.確認 Azure Linux 代理程式版本
    
```bash
waagent -version
```

針對 CoreOS，上述命令可能無效。

您會看到 Azure Linux 代理程式版本已更新為新的版本。

如需有關「Azure Linux 代理程式」的詳細資訊，請參閱 [Azure Linux 代理程式讀我檔案](https://github.com/Azure/WALinuxAgent)。
