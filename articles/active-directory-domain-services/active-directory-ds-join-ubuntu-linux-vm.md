---
title: Azure Active Directory Domain Services：將 Ubuntu VM 加入受控網域 | Microsoft Docs
description: 將 Ubuntu Linux 虛擬機器加入 Azure AD Domain Services
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 804438c4-51a1-497d-8ccc-5be775980203
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/22/2018
ms.author: maheshu
ms.openlocfilehash: 645e1eaedf3832b384a174d9f9ede5ea835047cd
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39502962"
---
# <a name="join-an-ubuntu-virtual-machine-in-azure-to-a-managed-domain"></a>在 Azure 中將 Ubuntu 虛擬機器加入受控網域
本文說明如何將 Ubuntu Linux 虛擬機器加入 Azure AD Domain Services 受控網域。

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="before-you-begin"></a>開始之前
若要執行本文中所列的工作，您需要︰  
1. 有效的 **Azure 訂用帳戶**。
2. **Azure AD 目錄** - 與內部部署目錄或僅限雲端的目錄同步處理。
3. **Azure AD 網域服務** 必須已針對 Azure AD 目錄啟用。 如果還沒有啟用，請按照 [入門指南](active-directory-ds-getting-started.md)所述的所有工作來進行。
4. 確保您尚未將受控網域的 IP 位址設定為虛擬網路的 DNS 伺服器。 如需詳細資訊，請參閱[如何更新 Azure 虛擬網路的 DNS 設定](active-directory-ds-getting-started-dns.md)
5. 完成[將密碼同步處理至您的 Azure AD Domain Services 受控網域](active-directory-ds-getting-started-password-sync.md)所需的步驟。


## <a name="provision-an-ubuntu-linux-virtual-machine"></a>佈建 Ubuntu Linux 虛擬機器
使用下列其中一個方法，在 Azure 中佈建 Ubuntu Linux 虛擬機器：
* [Azure 入口網站](../virtual-machines/linux/quick-create-portal.md)
* [Azure CLI](../virtual-machines/linux/quick-create-cli.md)
* [Azure PowerShell](../virtual-machines/linux/quick-create-powershell.md)

> [!IMPORTANT]
> * 將虛擬機器部署至**已在啟用 Azure AD Domain Services 的相同虛擬網路**。
> * 挑選與您已啟用 Azure AD Domain Services 的子網路**不同的子網路**。
>


## <a name="connect-remotely-to-the-ubuntu-linux-virtual-machine"></a>從遠端連線至 Ubuntu Linux 虛擬機器
Ubuntu 虛擬機器已佈建在 Azure 中。 下一個工作是使用佈建 VM 時所建立的本機系統管理員帳戶，從遠端連線至虛擬機器。

依照[如何登入執行 Linux 的虛擬機器](../virtual-machines/linux/mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)一文中的指示操作。


## <a name="configure-the-hosts-file-on-the-linux-virtual-machine"></a>在 Linux 虛擬機器上設定主機檔案
在您的 SSH 終端機中，編輯 /etc/hosts 檔案並更新您的電腦 IP 位址和主機名稱。

```
sudo vi /etc/hosts
```

在主機檔案中，輸入下列值：

```
127.0.0.1 contoso-ubuntu.contoso100.com contoso-ubuntu
```
在這裡，'contoso100.com' 為受控網域的 DNS 網域名稱。 'contoso-ubuntu' 是您要加入至受控網域之 Ubuntu 虛擬機器的主機名稱。


## <a name="install-required-packages-on-the-linux-virtual-machine"></a>在 Linux 虛擬機器上安裝必要封裝
接下來，在虛擬機器上安裝加入網域所需的套件。 執行下列步驟：

1.  在您的 SSH 終端機中，輸入下列命令，從儲存機制下載封裝清單。 此命令會更新套件清單，以取得最新版本套件和其相依性的資訊。

    ```
    sudo apt-get update
    ```

2. 輸入下列命令以安裝必要的套件。
    ```
      sudo apt-get install krb5-user samba sssd sssd-tools libnss-sss libpam-sss ntp ntpdate realmd adcli
    ```

3. 在 Kerberos 安裝期間，您會看到粉紅色畫面。 'krb5-user' 套件的安裝會提示輸入領域名稱 (以全部大寫)。 安裝會寫入 /etc/krb5.conf 的 [realm] 和 [domain_realm] 區段中。

    > [!TIP]
    > 如果受控網域的名稱為 contoso100.com，請輸入 CONTOSO100.COM 做為領域。 請記住，必須以全部大寫指定領域名稱。
    >
    >


## <a name="configure-the-ntp-network-time-protocol-settings-on-the-linux-virtual-machine"></a>在 Linux 虛擬機器上設定 NTP (網路時間通訊協定)
Ubuntu VM 的日期和時間必須與受控網域同步處理。 在 /etc/ntp.conf 檔案中新增受控網域的 NTP 主機名稱。

```
sudo vi /etc/ntp.conf
```

在 ntp.conf 檔案中，輸入下列值並儲存檔案：

```
server contoso100.com
```
在這裡，'contoso100.com' 為受控網域的 DNS 網域名稱。

現在，將 Ubuntu VM 的日期和時間與 NTP 伺服器同步處理，然後啟動 NTP 服務：

```
sudo systemctl stop ntp
sudo ntpdate contoso100.com
sudo systemctl start ntp
```


## <a name="join-the-linux-virtual-machine-to-the-managed-domain"></a>將 Linux 虛擬機器加入受控網域
既然 Linux 虛擬機器上已安裝必要的封裝，下一個工作是將虛擬機器加入受控網域。

1. 探索 AAD 網域服務受控網域。 在 SSH 終端機中輸入下列命令：

    ```
    sudo realm discover CONTOSO100.COM
    ```

   > [!NOTE]
   > **疑難排解：** 如果 [領域探索] 找不到受控網域：
     * 確定可從虛擬機器觸達網域 (請嘗試 ping)。
     * 檢查虛擬機器已確實部署到有提供受控網域的相同虛擬網路上。
     * 查看您是否已更新虛擬網路的 DNS 伺服器設定，以指向受控網域的網域控制站。
   >

2. 初始化 Kerberos。 在 SSH 終端機中輸入下列命令：

    > [!TIP]
    > * 請確定您是指定屬於 'AAD DC Administrators' 群組的使用者。
    > * 以大寫字母指定網域名稱，否則 kinit 會失敗。
    >

    ```
    kinit bob@CONTOSO100.COM
    ```

3. 將電腦加入網域。 在 SSH 終端機中輸入下列命令：

    > [!TIP]
    > 使用您在前面步驟中指定的相同使用者帳戶 ('kinit')。
    >

    ```
    sudo realm join --verbose CONTOSO100.COM -U 'bob@CONTOSO100.COM' --install=/
    ```

當電腦成功加入受控網域時，您應該會收到訊息 (「已成功在領域中註冊電腦」)。


## <a name="update-the-sssd-configuration-and-restart-the-service"></a>更新 SSSD 設定並重新啟動服務
1. 在 SSH 終端機中輸入下列命令。 開啟 sssd.conf 檔案並進行下列變更
    ```
    sudo vi /etc/sssd/sssd.conf
    ```

2. 將 **use_fully_qualified_names = True** 行註解化，並儲存檔案。
    ```
    # use_fully_qualified_names = True
    ```

3. 重新啟動 SSSD 服務。
    ```
    sudo service sssd restart
    ```


## <a name="configure-automatic-home-directory-creation"></a>設定自動主目錄建立
若要啟用主目錄的自動建立，在使用者登入之後，請您 PuTTY 終端機中輸入下列命令：
```
sudo vi /etc/pam.d/common-session
```

在此檔案的 'session optional pam_sss.so' 下新增以下這行，並儲存檔案：
```
session required pam_mkhomedir.so skel=/etc/skel/ umask=0077
```


## <a name="verify-domain-join"></a>確認加入網域
確認電腦是否已成功加入受控網域。 連接到使用不同的 SSH 連線加入網域的 Ubuntu VM。 使用網域使用者帳戶，然後查看使用者帳戶是否解析正確。

1. 在 SSH 終端機中輸入下列命令，以使用 SSH 連線到加入網域的 Ubuntu 虛擬機器。 使用屬於受控網域的網域帳戶 (例如，在此例中為 'bob@CONTOSO100.COM')。
    ```
    ssh -l bob@CONTOSO100.COM contoso-ubuntu.contoso100.com
    ```

2. 在 SSH 終端機中輸入下列命令，以查看是否已正確初始化主目錄。
    ```
    pwd
    ```

3. 在 SSH 終端機中輸入下列命令，以查看是否會正確解析群組成員資格。
    ```
    id
    ```


## <a name="grant-the-aad-dc-administrators-group-sudo-privileges"></a>授與 'AAD DC Administrators' 群組 sudo 權限
您可以在 Ubuntu VM 上對 ' AAD DC Administrators' 群組的成員授與系統管理權限。 sudo 檔案位於 /etc/sudoers。 在 sudoers 中新增的 AD 群組成員可以執行 sudo。

1. 在您的 SSH 終端機，確保您使用 superuser 權限登入。 您可以使用建立 VM 時指定的本機系統管理員帳戶。 執行以下命令：
    ```
    sudo vi /etc/sudoers
    ```

2. 將下列項目新增至 /etc/sudoers 檔案，並將它儲存：
    ```
    # Add 'AAD DC Administrators' group members as admins.
    %AAD\ DC\ Administrators ALL=(ALL) NOPASSWD:ALL
    ```

3. 您現在可以以 ' AAD DC Administrators' 群組成員的身分登入，並且應該具備 VM 上的系統管理權限。


## <a name="troubleshooting-domain-join"></a>針對加入網域進行疑難排解
請參閱 [針對加入網域進行疑難排解](active-directory-ds-admin-guide-join-windows-vm-portal.md#troubleshoot-joining-a-domain) 一文。


## <a name="related-content"></a>相關內容
* [Azure AD Domain Services - 入門指南](active-directory-ds-getting-started.md)
* [將 Windows Server 虛擬機器加入 Azure Active Directory Domain Services 受控網域](active-directory-ds-admin-guide-join-windows-vm.md)
* [如何登入執行 Linux 的虛擬機器](../virtual-machines/linux/mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
