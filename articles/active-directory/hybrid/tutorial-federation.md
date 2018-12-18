---
title: 教學課程：將單一 AD 樹系環境與 Azure 建立同盟 | Microsoft Docs
description: 示範如何使用同盟來設定混合式身分識別環境。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/16/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 2a869e8d504e4f3557caa7925b89e8c184718af9
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306077"
---
# <a name="tutorial-federate-a-single-ad-forest-environment-to-the-cloud"></a>教學課程：將單一 AD 樹系環境與雲端建立同盟

![建立](media/tutorial-federation/diagram.png)

下列教學課程將逐步說明如何使用同盟來建立混合式身分識別環境。  此環境可用於測試或更熟悉混合式身分識別的運作。

## <a name="prerequisites"></a>先決條件
下列是完成此教學課程的先決條件
- 已安裝 [Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/hyper-v-technology-overview) 的電腦。  建議您在 [Windows 10](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/supported-guest-os) 或 [Windows Server 2016](https://docs.microsoft.com/windows-server/virtualization/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows) 電腦上執行此作業。
- [Azure 訂用帳戶](https://azure.microsoft.com/free)
- - [外部網路介面卡](https://docs.microsoft.com/virtualization/hyper-v-on-windows/quick-start/connect-to-network)，以允許虛擬機器與網際網路通訊。
- 一份 Windows Server 2016
- 可驗證的[自訂網域](../../active-directory/fundamentals/add-custom-domain.md)

> [!NOTE]
> 此教學課程使用 PowerShell 指令碼，因此您可以在最快的時間內建立教學課程環境。  每個指令碼都使用在指令碼開頭處宣告的變數。  您可以且應該變更變數以反映您的環境。
>
>用來在安裝 Azure AD Connect 之前建立一般 Active Directory 環境的指令碼。  它們在所有指令碼中都通用。
>
> 您可以在 Github 上的[這裡](https://github.com/billmath/tutorial-phs)用於此教學課程中的 PowerShell 指令碼。

## <a name="create-a-virtual-machine"></a>建立虛擬機器
我們必須執行以讓我們的混合式身分識別環境開始執行的第一步是建立將做為我們的內部部署 Active Directory 伺服器使用的虛擬機器。  

>[!NOTE]
>若您從未在主機電腦上的 PowerShell 中執行過指令碼，您將必須在 PowerShell 中執行 `Set-ExecutionPolicy remotesigned` 並回答是，才能執行指令碼。

執行下列動作：

1. 以系統管理員身分開啟 PowerShell ISE。
2. 執行下列指令碼。

```powershell
#Declare variables
$VMName = 'DC1'
$Switch = 'External'
$InstallMedia = 'D:\ISO\en_windows_server_2016_updated_feb_2018_x64_dvd_11636692.iso'
$Path = 'D:\VM'
$VHDPath = 'D:\VM\DC1\DC1.vhdx'
$VHDSize = '64424509440'

#Create New Virtual Machine
New-VM -Name $VMName -MemoryStartupBytes 16GB -BootDevice VHD -Path $Path -NewVHDPath $VHDPath -NewVHDSizeBytes $VHDSize  -Generation 2 -Switch $Switch  

#Set the memory to be non-dynamic
Set-VMMemory $VMName -DynamicMemoryEnabled $false

#Add DVD Drive to Virtual Machine
Add-VMDvdDrive -VMName $VMName -ControllerNumber 0 -ControllerLocation 1 -Path $InstallMedia

#Mount Installation Media
$DVDDrive = Get-VMDvdDrive -VMName $VMName

#Configure Virtual Machine to Boot from DVD
Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive 
```

## <a name="complete-the-operating-system-deployment"></a>完成作業系統部署
若要完成虛擬機器建立作業，您必須完成作業系統安裝。

1. Hyper-V Manager，按兩下虛擬機器
2. 按一下 [啟動] 按鈕。
3.  系統將提示您「按任意鍵以從 CD 或 DVD 開機」。 按任意鍵以繼續。
4. 在 Windows Server 啟動畫面上選取您的語言，然後按一下 [下一步]。
5. 按一下 [立即安裝] 。
6. 輸入您的授權金鑰，然後按一下 [下一步]。
7. 按一下 [我接受授權條款]，然後按一下 [下一步]。
8. 選取 [自訂: 只安裝 Windows (進階)]
9. 按一下 [下一步]
10. 安裝完成之後，重新啟動虛擬機器，登入並執行 Windows 更新，以確定 VM 是最新的。  安裝最新的更新。

## <a name="install-active-directory-pre-requisites"></a>安裝 Active Directory 先決條件
既然我們已啟動虛擬機器，我們必須在安裝 Active Directory 之前執行一些動作。  也就是說，我們必須重新命名虛擬機器、設定靜態 IP 位址與 DNS 資訊，然後安裝遠端伺服器管理工具。   執行下列動作：

1. 以系統管理員身分開啟 PowerShell ISE。
2. 執行 `Set-ExecutionPolicy remotesigned` 並回答全部皆是 [A]。  按 Enter。
3. 執行下列指令碼。

```powershell
#Declare variables
$ipaddress = "10.0.1.117" 
$ipprefix = "24" 
$ipgw = "10.0.1.1" 
$ipdns = "10.0.1.117"
$ipdns2 = "8.8.8.8" 
$ipif = (Get-NetAdapter).ifIndex 
$featureLogPath = "c:\poshlog\featurelog.txt" 
$newname = "DC1"
$addsTools = "RSAT-AD-Tools" 

#Set static IP address
New-NetIPAddress -IPAddress $ipaddress -PrefixLength $ipprefix -InterfaceIndex $ipif -DefaultGateway $ipgw 

# Set the DNS servers
Set-DnsClientServerAddress -InterfaceIndex $ipif -ServerAddresses ($ipdns, $ipdns2)

#Rename the computer 
Rename-Computer -NewName $newname -force 

#Install features 
New-Item $featureLogPath -ItemType file -Force 
Add-WindowsFeature $addsTools 
Get-WindowsFeature | Where installed >>$featureLogPath 

#Restart the computer 
Restart-Computer
```

## <a name="create-a-windows-server-ad-environment"></a>建立 Windows Server AD 環境
既然我們已建立 VM 並將它重新命名且設定靜態 IP 位址，我們可以繼續安裝並設定 Active Directory Domain Services。  執行下列動作：

1. 以系統管理員身分開啟 PowerShell ISE。
2. 執行下列指令碼。

```powershell 
#Declare variables
$DatabasePath = "c:\windows\NTDS"
$DomainMode = "WinThreshold"
$DomainName = "contoso.com"
$DomaninNetBIOSName = "CONTOSO"
$ForestMode = "WinThreshold"
$LogPath = "c:\windows\NTDS"
$SysVolPath = "c:\windows\SYSVOL"
$featureLogPath = "c:\poshlog\featurelog.txt" 
$Password = "Pass1w0rd"

#Install AD DS, DNS and GPMC 
start-job -Name addFeature -ScriptBlock { 
Add-WindowsFeature -Name "ad-domain-services" -IncludeAllSubFeature -IncludeManagementTools 
Add-WindowsFeature -Name "dns" -IncludeAllSubFeature -IncludeManagementTools 
Add-WindowsFeature -Name "gpmc" -IncludeAllSubFeature -IncludeManagementTools } 
Wait-Job -Name addFeature 
Get-WindowsFeature | Where installed >>$featureLogPath

#Create New AD Forest
Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath $DatabasePath -DomainMode $DomainMode -DomainName $DomainName -SafeModeAdministratorPassword $Password -DomainNetbiosName $DomainNetBIOSName -ForestMode $ForestMode -InstallDns:$true -LogPath $LogPath -NoRebootOnCompletion:$false -SysvolPath $SysVolPath -Force:$true
```

## <a name="create-a-windows-server-ad-user"></a>建立 Windows Server AD 使用者
既然我們已經有 Active Directory 環境，我們需要測試帳戶。  此帳戶將在我們的內部部署 AD 環境建立，然後同步到 Azure AD。  執行下列動作：

1. 以系統管理員身分開啟 PowerShell ISE。
2. 執行下列指令碼。

```powershell 
#Declare variables
$Givenname = "Allie"
$Surname = "McCray"
$Displayname = "Allie McCray"
$Name = "amccray"
$Password = "Pass1w0rd"
$Identity = "CN=ammccray,CN=Users,DC=contoso,DC=com"
$SecureString = ConvertTo-SecureString $Password -AsPlainText -Force


#Create the user
New-ADUser -Name $Name -GivenName $Givenname -Surname $Surname -DisplayName $Displayname -AccountPassword $SecureString

#Set the password to never expire
Set-ADUser -Identity $Identity -PasswordNeverExpires $true -ChangePasswordAtLogon $false -Enabled $true
```

## <a name="create-a-certificate-for-ad-fs"></a>為 AD FS 建立憑證
現在我們將建立將由 AD FS 使用的 SSL 憑證。  這將會是自我簽署憑證，而且僅供測試用途使用。  Microsoft 建議您不要在生產環境中使用自我簽署憑證。 執行下列動作：

1. 以系統管理員身分開啟 PowerShell ISE。
2. 執行下列指令碼。

```powershell 
#Declare variables
$DNSname = "adfs.contoso.com"
$Location = "cert:\LocalMachine\My"

#Create certificate
New-SelfSignedCertificate -DnsName $DNSname -CertStoreLocation $Location
```

## <a name="create-an-azure-ad-tenant"></a>建立 Azure AD 租用戶
現在我們必須建立 Azure AD 租用戶，以便我們可以將使用者同步到雲端。  若要建立新的 Azure AD 租用戶，請執行下列動作。

1. 瀏覽到 [Azure 入口網站](https://portal.azure.com)並使用具有 Azure 訂用帳戶的帳戶登入。
2. 選取**加號圖示 (+)** 並搜尋 **Azure Active Directory**。
3. 在搜尋結果中選取 [Azure Active Directory]。
4. 選取 [建立] 。</br>
![建立](media/tutorial-password-hash-sync/create1.png)</br>
5. 提供**組織名稱**與**初始網域名稱**。 然後選取 [建立]。 這將會建立您的目錄。
6. 完成此動作之後，請按一下**這裡**以管理目錄。

## <a name="create-a-global-administrator-in-azure-ad"></a>在 Azure AD 中建立全域系統管理員
既然我們已經有 Azure AD 租用戶，我們將建立全域系統管理員帳戶。  此帳戶會用來在 Azure AD Connect 安裝期間建立 Azure AD Connector 帳戶。  Azure AD Connector 帳戶會用來將資訊寫入到 Azure AD。   若要建立全域系統管理員帳戶，請執行下列動作。

1.  在 [管理] 底下選取 [使用者]。</br>
![建立](media/tutorial-password-hash-sync/gadmin1.png)</br>
2.  選取 [所有使用者 ]，然後選取 [+ 新增使用者]。
3.  提供此使用者的名稱與使用者名稱。 這將成為您租用戶的全域系統管理員。 您也可以將 [目錄角色] 變更為 [全域系統管理員]。 您也可以顯示臨時密碼。 完成之後，請選取 [建立]。</br>
![建立](media/tutorial-password-hash-sync/gadmin2.png)</br>
4. 完成此動作之後，請開啟新的網頁瀏覽器並使用新的全域系統管理員帳戶與臨時密碼登入 myapps.microsoft.com。
5. 將全域系統管理員的密碼變更為您可以記住的密碼。

## <a name="add-the-custom-domain-name-to-your-directory"></a>在目錄中新增自訂網域名稱
既然我們已經有租用戶與全域系統管理員，我們必須新增我們的自訂網域，以便 Azure 可以驗證該網域。  執行下列動作：

1. 返回 [Azure 入口網站](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview)，並關閉 [所有使用者] 刀鋒視窗。
2. 在左側選取 [自訂網域名稱]。
3. 選取 [新增自訂網域]。</br>
![同盟](media/tutorial-federation/custom1.png)</br>
4. 在 [自訂網域名稱] 上，於方塊中輸入您的自訂網域名稱，然後按一下 [新增網域]。
5. 在自訂網域名稱畫面上，我們將提供 TXT 或 MX 資訊給您。  此資訊必須新增到網域註冊機構中您的網域下的 DNS 資訊中。  因此，您必須移至您的網域註冊機構，然後將 TXT 或 MX 資訊輸入到您網域的 DNS 設定中。  這將可允許 Azure 驗證您的網域。  Azure 最多可能需要 24 小時才能驗證該網域。  如需詳細資訊，請參閱[新增自訂網域](../../active-directory/fundamentals/add-custom-domain.md)文件。</br>
![同盟](media/tutorial-federation/custom2.png)</br>
6. 若要確定系統會驗證它，請按一下 [驗證] 按鈕。</br>
![同盟](media/tutorial-federation/custom3.png)</br>

## <a name="download-and-install-azure-ad-connect"></a>下載並安裝 Azure AD Connect
現在您可以下載並安裝 Azure AD Connect。  安裝之後，我們將會執行快速安裝。  執行下列動作：

1. 下載 [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594)
2. 瀏覽並按兩下 **AzureADConnect.msi**。
3. 在 [歡迎] 畫面上，選取同意授權條款的方塊，然後按一下 [繼續] 。  
4. 在 [快速設定] 畫面上，按一下 [自訂]。  
5. 在 [安裝所需的元件] 畫面上。 按一下 [安裝]。  
6. 在 [使用者登入] 畫面上，選取 [與 AD FS 同盟]，然後按一下 [下一步]。
![同盟](media/tutorial-federation/fed1.png)

1. 在 [連線到 Azure AD] 畫面上，輸入我們在上面的步驟中建立的全域系統管理員使用者名稱與密碼，然後按一下 [下一步]。
2. 在 [連線您的目錄] 畫面上，按一下 [新增目錄]。  接著，選取 [建立新的 AD 帳戶] 並輸入 contoso\Administrator 使用者名稱與密碼，然後按一下 [確定]。
3. 按一下 [下一步]。
4. 在 [Azure AD 登入設定] 畫面上，選取 [不將所有 UPN 尾碼比對至已驗證網域就繼續]，然後按一下 [下一步]。
5. 在 [網域與 OU 篩選] 畫面上，按一下 [下一步]。
6. 在 [專門識別您的使用者] 畫面上，按一下 [下一步]。
7. 在 [篩選使用者和裝置] 畫面上，按一下 [下一步]。
8. 在 [選用功能] 畫面上，按一下 [下一步]。
9. 在 [網域系統管理員認證] 頁面上，輸入 contoso\Administrator 使用者名稱與密碼，然後按一下 [下一步]。
10. 在 [AD FS 伺服器陣列] 畫面上，確定已選取 [設定新的 AD FS 伺服器陣列]。
11. 選取 [使用安裝在同盟伺服器上的憑證]，然後按一下[瀏覽]。
12. 在搜尋方塊中輸入 DC1，然後在找到它之後選取它。  按一下 [確定]。
13. 從 [憑證檔案] 下拉式清單中，選取 [adfs.contoso.com] 我們在上面建立的憑證。  按一下 [下一步]。
![同盟](media/tutorial-federation/fed2.png)

1. 在 [AD FS 伺服器] 畫面上，按一下 [瀏覽] 並在搜尋方塊中輸入 DC1，然後在找到它之後選取它。  按一下 [確定]。  按一下 [下一步]。
![同盟](media/tutorial-federation/fed3.png)

1. 在 [Web 應用程式 Proxy 伺服器] 畫面上，按一下 [下一步]。
2. 在 [AD FS 服務帳戶] 畫面上，輸入 contoso\Administrator 使用者名稱與密碼，然後按一下 [下一步]。
3. 在 [Azure AD 網域] 畫面上，從下拉式清單選取您的已驗證自訂網域，然後按一下 [下一步]。
4. 在 [準備好設定] 畫面中，按一下 [安裝]。
5. 當安裝完成時，按一下 [結束]。
6. 安裝完成之後，請先登出並重新登入，再使用 Synchronization Service Manager 或「同步處理規則編輯器」。


## <a name="verify-users-are-created-and-synchronization-is-occurring"></a>確認已建立使用者且已進行同步
我們現在將會確認內部部署目錄中的使用者已同步，而且現在存在於外部 Azure AD 租用戶中。  這可能需要花幾個小時才能完成。  若要確認使用者是否已同步，請執行下列動作。


1. 瀏覽到 [Azure 入口網站](https://portal.azure.com)並使用具有 Azure 訂用帳戶的帳戶登入。
2. 選取左邊的 [Azure Active Directory]。
3. 在 [管理] 底下選取 [使用者]。
4. 確認您看到新使用者存在於我們的租用戶 ![同步](media/tutorial-password-hash-sync/synch1.png)

## <a name="test-signing-in-with-one-of-our-users"></a>使用我們的其中一個使用者來測試登入

1.  瀏覽到 [http://myapps.microsoft.com](http://myapps.microsoft.com)
2. 使用我們在我們的新租用戶中建立的使用者來登入。  您必須使用下列格式來登入：(user@domain.onmicrosoft.com)。 使用該使用者在內部部署用來登入的密碼來登入。
![驗證](media/tutorial-password-hash-sync/verify1.png)

您現在已成功設定混合式身分識別環境，可用來測試及熟悉 Azure 的功能。

## <a name="next-steps"></a>後續步驟

- [硬體與先決條件](how-to-connect-install-prerequisites.md) 
- [自訂設定](how-to-connect-install-custom.md)
- [Azure AD Connect 和同盟](how-to-connect-fed-whatis.md)

