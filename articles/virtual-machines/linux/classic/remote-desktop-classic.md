---
title: 從遠端桌面連接至 Linux VM | Microsoft Docs
description: 了解如何安裝和設定遠端桌面，以連接至傳統部署模型的 Microsoft Azure Linux VM
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-service-management
ROBOTS: NOINDEX
ms.assetid: 34348659-ddb7-41da-82d6-b5885859e7e4
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 05/30/2017
ms.author: cynthn
ms.openlocfilehash: 5e68774c3edb7d82fef388c593a6b96c52857be6
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37927735"
---
# <a name="using-remote-desktop-to-connect-to-a-microsoft-azure-linux-vm"></a>使用遠端桌面連接至 Microsoft Azure Linux VM
> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 如需本文已更新的 Resource Manager 版本，請參閱[這裡](../use-remote-desktop.md)。

## <a name="overview"></a>概觀
RDP (遠端桌面通訊協定) 是用於 Windows 的專屬通訊協定。 我們要如何使用 RDP 從遠端連接到 Linux VM (虛擬機器)？

本指南會為您提供答案！ 它會協助您在 Microsoft Azure Linux VM 上安裝及設定 xrdp，這會讓您能夠從 Windows 電腦使用遠端桌面來連接它。 在本指南中我們會使用執行 Ubuntu 或 OpenSUSE 的 Linux VM 做為範例。

xrdp 工具是一個開放原始碼 RDP 伺服器，可讓您從 Windows 電腦使用遠端桌面連接 Linux 伺服器。 RDP 擁有比 VNC (虛擬網路運算) 更好的效能。 VNC 使用 JPEG 品質圖形進行轉譯，而且速度可能很慢，RDP 的速度很快，且圖形品質非常清晰。

> [!NOTE]
> 您必須已經有執行 Linux 的 Microsoft Azure VM。 若要建立並設定 Linux VM，請參閱 [Azure Linux VM 教學課程](createportal-classic.md)。
> 
> 

## <a name="create-an-endpoint-for-remote-desktop"></a>建立遠端桌面的端點
我們將對本文件中的遠端桌面使用預設端點 3389。將 3389 端點設定為 Linux VM 的 `Remote Desktop`，如下所示：

![映像](./media/remote-desktop/endpoint-for-linux-server.png)

如果您不知道如何為 VM 設定端點，請參閱[本指南](setup-endpoints.md)。

## <a name="install-gnome-desktop"></a>安裝 Gnome 桌面
透過 `putty` 連接到 Linux VM，然後安裝 `Gnome Desktop`。

針對 Ubuntu，使用：

```bash
sudo apt-get update
sudo apt-get install ubuntu-desktop
```

針對 OpenSUSE，使用︰

```bash
sudo zypper install gnome-session
```

## <a name="install-xrdp"></a>安裝 xrdp
針對 Ubuntu，使用：

```bash
sudo apt-get install xrdp
```

針對 OpenSUSE，使用︰

> [!NOTE]
> 在以下命令中，用您正在使用的版本更新 OpenSUSE 版本。 下方是 `OpenSUSE 13.2` 的範例。
> 
> 

```bash
sudo zypper in http://download.opensuse.org/repositories/X11:/RemoteDesktop/openSUSE_13.2/x86_64/xrdp-0.9.0git.1401423964-2.1.x86_64.rpm
sudo zypper install tigervnc xorg-x11-Xvnc xterm remmina-plugin-vnc
```

## <a name="start-xrdp-and-set-xdrp-service-at-boot-up"></a>在開機時啟動 xrdp 並設定 xdrp 服務
針對 OpenSUSE，使用︰

```bash
sudo systemctl start xrdp
sudo systemctl enable xrdp
```

對於 Ubuntu，安裝之後會啟動 xrdp，並在開機時自動啟用。

## <a name="using-xfce-if-you-are-using-an-ubuntu-version-later-than-ubuntu-1204lts"></a>如果您使用比 Ubuntu 12.04LTS 更新的 Ubuntu 版本則使用 xfce
由於 xrdp 目前的版本不支援比 Ubuntu 12.04LTS 更新的 Ubuntu 版本 Gnome 桌面，因此我們將改用 `xfce` 桌面。

若要安裝 `xfce`，請使用此命令：

```bash
sudo apt-get install xubuntu-desktop
```

然後使用此命令啟用 `xfce`：

```bash
echo xfce4-session >~/.xsession
```

編輯組態檔 `/etc/xrdp/startwm.sh`：

```bash
sudo vi /etc/xrdp/startwm.sh   
```

在 `/etc/X11/Xsession` 一行前面加入 `xfce4-session` 這一行。

若要重新啟動 xrdp 服務，請使用此：

```bash
sudo service xrdp restart
```

## <a name="connect-your-linux-vm-from-a-windows-machine"></a>從 Windows 電腦連接 Linux VM
在 Windows 電腦中啟動遠端桌面用戶端，並輸入您的 Linux VM DNS 名稱。 或在 Azure 入口網站中移至您 VM 的儀表板，並按一下 `Connect` 來連接 Linux VM。 這時您會看到登入視窗：

![映像](./media/remote-desktop/no2.png)

請使用您 Linux VM 的使用者名稱和密碼登入。

## <a name="next-steps"></a>後續步驟
如需使用 xrdp 的相關詳細資訊，請參閱 [http://www.xrdp.org/](http://www.xrdp.org/)。
