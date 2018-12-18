---
title: Azure VM 的特定 RDP 錯誤訊息 | Microsoft Docs
description: 了解在 Azure 中嘗試針對 Windows 虛擬機器使用遠端桌面連線時，可能會接收到的特定錯誤訊息
keywords: 遠端桌面錯誤、遠端桌面連線錯誤、無法連接到 VM、遠端桌面疑難排解
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: jeconnoc
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.assetid: 5feb1d64-ee6f-4907-949a-a7cffcbc6153
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: 7d4deb0618b0e0802306e344fea6d259c0597fe2
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411175"
---
# <a name="troubleshooting-specific-rdp-error-messages-to-a-windows-vm-in-azure"></a>針對 Azure 中 Windows VM 的特定 RDP 錯誤訊息進行疑難排解
您在 Azure 中針對 Windows 虛擬機器 (VM) 使用遠端桌面連線時，可能會接收到特定錯誤訊息。 本文將詳述一些較常發生的錯誤訊息，並提供解決它們的疑難排解步驟。 如果您無法使用 RDP 連線到 VM，但沒有遇到特定錯誤訊息，請參閱[遠端桌面的疑難排解指南](troubleshoot-rdp-connection.md)。

如需特定錯誤訊息的資訊，請參閱下列內容：

* [遠端工作階段中斷，因為沒有提供授權的遠端桌面授權伺服器可以使用](#rdplicense)。
* [遠端桌面找不到電腦「名稱」](#rdpname)。
* [發生驗證錯誤。無法連絡本機安全性授權](#rdpauth)。
* [Windows 安全性錯誤：您的認證無法運作](#wincred)。
* [這部電腦無法連線到遠端電腦](#rdpconnect)。

<a id="rdplicense"></a>

## <a name="the-remote-session-was-disconnected-because-there-are-no-remote-desktop-license-servers-available-to-provide-a-license"></a>遠端工作階段中斷，因為沒有提供授權的遠端桌面授權伺服器可以使用。
原因：遠端桌面角色的 120 天授權寬限期已過期，您需要安裝授權。

作為因應措施，請從入口網站儲存 RDP 檔案的本機複本，並在 PowerShell 命令提示字元執行該命令來連接。 此步驟只會停用該連線的授權：

        mstsc <File name>.RDP /admin

如果您並非真的需要多於兩個對 VM 的同步遠端桌面連線，則您可以使用伺服器管理員來移除遠端桌面伺服器角色。

如需詳細資訊，請參閱部落格文章： [Azure VM fails with "No Remote Desktop License Servers available" (Azure VM 因「沒有可用的遠端桌面授權伺服器」而失敗)](https://blogs.msdn.microsoft.com/mast/2014/01/21/rdp-to-azure-vm-fails-with-no-remote-desktop-license-servers-available/)。

<a id="rdpname"></a>

## <a name="remote-desktop-cant-find-the-computer-name"></a>遠端桌面找不到電腦「名稱」。
原因：您電腦上的遠端桌面用戶端無法解析此 RDP 檔案設定中的電腦名稱。

可能的解決方案：

* 如果您位於組織內部網路，請確保您的電腦能存取 Proxy 伺服器，並能將 HTTPS 流量傳送給 Proxy 伺服器。
* 如果您使用本機儲存的 RDP 檔案，請嘗試使用入口網站所產生的檔案。 此步驟可確保您擁有虛擬機器的正確 DNS 名稱，或是 VM 的雲端服務和端點連接埠。 以下是由入口網站產生的 RDP 檔案範例：
  
        full address:s:tailspin-azdatatier.cloudapp.net:55919
        prompt for credentials:i:1

此 RDP 檔案的位址部分包含︰

* 雲端服務的完整網域名稱，其中包含 VM (本例中為 "tailspin-azdatatier.cloudapp.net")。
* 遠端桌面流量 (55919) 端點的外部 TCP 連接埠。

<a id="rdpauth"></a>

## <a name="an-authentication-error-has-occurred-the-local-security-authority-cannot-be-contacted"></a>發生驗證錯誤。 無法連絡本機安全性授權。
原因：目標 VM 在認證的使用者名稱部分找不到安全性授權。

當您的使用者名稱格式為 *SecurityAuthority*\\*UserName* (範例：CORP\User1) 時，*SecurityAuthority* 部分會是 VM 的電腦名稱 (針對本機安全性授權) 或 Active Directory 網域名稱。

可能的解決方案：

* 如果帳戶是 VM 的本機帳戶，請確認 VM 名稱拼寫正確。
* 如果該帳戶位於 Active Directory 網域，請檢查網域名稱的拼字。
* 如果該帳戶是 Active Directory 網域帳戶且網域名稱的拼字正確，請確認該網域內可以使用網域控制站。 其為包含網域控制站的 Azure 虛擬網路中常見的問題，網域控制站無法使用的原因是其尚未啟動為因應措施。 您可以使用本機系統管理員帳戶做為因應措施，而非網域帳戶。

<a id="wincred"></a>

## <a name="windows-security-error-your-credentials-did-not-work"></a>Windows 安全性錯誤：您的認證無法運作。
原因：目標 VM 無法驗證您的帳戶名稱和密碼。

以 Windows 為基礎的電腦可以驗證本機帳戶或網域帳戶之認證。

* 如果是本機帳戶，請使用 *ComputerName*\\*UserName* 語法 (範例：SQL1\Admin4798)。
* 針對網域帳戶，請使用 *DomainName*\\*UserName* 語法 (範例：CONTOSO\peterodman)。

如果您在新的 Active Directory 樹系將 VM 提升為網域控制站，您用來登入的本機系統管理員帳戶會轉換為對等的帳戶，在新樹系和網域中使用相同的密碼。 本機帳戶隨即刪除。

例如，如果您以本機帳戶 DC1\DCAdmin 登入，然後將此虛擬機器提升為 corp.contoso.com 網域中新樹系的網域控制器，則會刪除 DC1\DCAdmin 本機帳戶，並且以相同密碼建立一個新的網域帳戶 (CORP\DCAdmin)。

請確保帳戶名稱為虛擬機器可驗證為有效帳戶的名稱，且密碼正確。

如果您需要變更本機系統管理員帳戶的密碼，請參閱 [如何重設 Windows 虛擬機器的密碼或遠端桌面服務](reset-rdp.md)。

<a id="rdpconnect"></a>

## <a name="this-computer-cant-connect-to-the-remote-computer"></a>這部電腦無法連線到遠端電腦。
原因：用來連接的帳戶並沒有遠端桌面的登入權限。

每部 Windows 電腦都有遠端桌面使用者本機群組，其中包含能夠遠端登入的帳戶和群組。 本機系統管理員群組成員也有權限，即使這些帳戶未列在遠端桌面使用者本機群組中。 對於加入網域的機器，本機系統管理員群組也包含此網域的網域系統管理員。

請確保您用於連接的帳戶具有遠端桌面登入權限。 因應措施是使用網域或本機系統管理員帳戶透過遠端桌面進行連接。 若要將所需帳戶新增至「遠端桌面」使用者本機群組，請使用 Microsoft Management Console 嵌入式管理單元 ([系統工具] > [本機使用者和群組] > [群組] > [遠端桌面使用者])。

## <a name="next-steps"></a>後續步驟
如果您在使用 RDP 進行連線時沒有發生上述錯誤，而是遇到未知的問題，請參閱[遠端桌面的疑難排解指南](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

* 如需存取在 VM 上執行之應用程式的疑難排解步驟，請參閱[針對在 Azure VM 上執行之應用程式的存取進行疑難排解](../linux/troubleshoot-app-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
* 如果您在 Azure 中使用安全殼層 (SSH) 連線到 Linux VM 時遇到問題，請參閱[針對 Azure 中對 Linux VM 的 SSH 連線進行疑難排解](../linux/troubleshoot-ssh-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

