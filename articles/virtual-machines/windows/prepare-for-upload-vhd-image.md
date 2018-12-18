---
title: 準備要上傳至 Azure 的 Windows VHD | Microsoft Docs
description: 如何在上傳至 Azure 之前準備 Windows VHD 或 VHDX
services: virtual-machines-windows
documentationcenter: ''
author: glimoli
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 7802489d-33ec-4302-82a4-91463d03887a
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: troubleshooting
ms.date: 08/23/2018
ms.author: genli
ms.openlocfilehash: b4787f5b9657afbcedbd3803d6a17af9c8cf9099
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406986"
---
# <a name="prepare-a-windows-vhd-or-vhdx-to-upload-to-azure"></a>準備 Windows VHD 或 VHDX 以上傳至 Azure
將 Windows 虛擬機器 (VM) 從內部部署上傳至 Microsoft Azure 之前，您必須先準備虛擬硬碟 (VHD 或 VHDX)。 Azure 只支援採用 VHD 檔案格式且具有固定大小磁碟的**第 1 代 VM**。 允許的 VHD 大小上限為 1023 GB。 您可以將第 1 代 VM 從 VHDX 檔案系統轉換為 VHD，以及從動態擴充磁碟轉換為固定大小的磁碟。 但您無法變更 VM 的世代。 如需詳細資訊，請參閱[應該在 Hyper-V 中建立第 1 代還是第 2 代的 VM](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v) \(英文\)。

如需 Azure VM 支援原則的詳細資訊，請參閱[適用於 Microsoft Azure 虛擬機器的 Microsoft Server Software 支援](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines) \(機器翻譯\)。

> [!Note]
> 本文中的指示適用於 64 位元版本的 Windows Server 2008 R2 或更新版本的 Windows Server 作業系統。 如需 Azure 中執行 32 位元版本的作業系統相關資訊，請參閱[在 Azure 虛擬機器中對 32 位元作業系統的支援](https://support.microsoft.com/help/4021388/support-for-32-bit-operating-systems-in-azure-virtual-machines) \(機器翻譯\)。

## <a name="convert-the-virtual-disk-to-vhd-and-fixed-size-disk"></a>將虛擬磁碟轉換成 VHD 及固定大小的磁碟 
如果您需要將虛擬磁碟轉換為 Azure 所需的格式，請使用本節中的其中一種方法。 在執行虛擬磁碟轉換程序之前備份 VM ，並確定 Windows VHD 在本機伺服器上正常運作。 先解決 VM 本身的任何錯誤，然後嘗試轉換或上傳至 Azure。

在轉換磁碟之後，建立會使用轉換磁碟的 VM。 啟動並登入 VM 以完成準備上傳 VM。

### <a name="convert-disk-using-hyper-v-manager"></a>使用 HYPER-V 管理員轉換磁碟
1. 開啟 Hyper-V 管理員，然後在左側選取您的本機電腦。 在電腦清單上方的功能表中，按一下 [動作] >  [編輯磁碟]。
2. 在 [尋找虛擬硬碟] 畫面上，尋找並選取您的虛擬磁碟。
3. 在 [選擇動作] 畫面上，接著選取 [轉換] 和 [下一步]。
4. 如果您需要從 VHDX 進行轉換，請選取 [VHD]，然後按 [下一步]。
5. 如果您需要從動態擴充磁碟進行轉換，選取 [固定大小]，然後按 [下一步]。
6. 尋找並選取用以儲存新 VHD 檔案的路徑。
7. 按一下 [完成] 。

>[!NOTE]
>本文中的命令必須以提高權限的 PowerShell 工作階段來執行。

### <a name="convert-disk-by-using-powershell"></a>使用 PowerShell 轉換磁碟
您可以在 Windows PowerShell 中使用 [Convert-VHD](http://technet.microsoft.com/library/hh848454.aspx) 命令來轉換虛擬磁碟。 當您啟動 PowerShell 時，選取 [以系統管理員身分執行]。 

下列範例命令會從 VHDX 轉換至 VHD，以及從動態擴充磁碟轉換至固定大小的磁碟：

```Powershell
Convert-VHD –Path c:\test\MY-VM.vhdx –DestinationPath c:\test\MY-NEW-VM.vhd -VHDType Fixed
```
在這個命令中，使用您想要轉換的虛擬硬碟路徑取代 "-Path" 的值，並使用已轉換磁碟的新路徑和名稱取代 "-DestinationPath" 的值。

### <a name="convert-from-vmware-vmdk-disk-format"></a>從 VMware VMDK 磁碟格式進行轉換
如果您的 Windows VM 映像是 [VMDK 檔案格式](https://en.wikipedia.org/wiki/VMDK)，使用 [Microsoft VM Converter](https://www.microsoft.com/download/details.aspx?id=42497) \(英文\) 將它轉換為 VHD。 如需詳細資訊，請參閱部落格文章：[如何將 VMware VMDK 轉換為 Hyper-V VHD](http://blogs.msdn.com/b/timomta/archive/2015/06/11/how-to-convert-a-vmware-vmdk-to-hyper-v-vhd.aspx) \(英文\)。

## <a name="set-windows-configurations-for-azure"></a>設定適用於 Azure 的 Windows 設定

在您計劃上傳至 Azure 的 VM 上，於下列步驟中，從[提升權限的命令提示字元視窗](https://technet.microsoft.com/library/cc947813.aspx)執行所有命令：

1. 在路由表上移除任何靜態持續路由：
   
   * 若要檢視路由表，在命令提示字元視窗上執行 `route print`。
   * 檢查 [持續路由]  區段。 如果有持續的路由，請使用 **route delete** 命令加以移除。
2. 移除 WinHTTP Proxy：
   
    ```PowerShell
    netsh winhttp reset proxy
    ```
3. 將磁碟 SAN 原則設為 [Onlineall](https://technet.microsoft.com/library/gg252636.aspx)：
   
    ```PowerShell
    diskpart 
    ```
    在開啟的命令提示字元視窗中，輸入下列命令：

     ```DISKPART
    san policy=onlineall
    exit   
    ```

4. 設定適用於 Windows 的國際標準時間 (UTC)，並將 Windows 時間 (w32time) 服務的啟動類型設為 [自動]：
   
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\TimeZoneInformation' -name "RealTimeIsUniversal" 1 -Type DWord

    Set-Service -Name w32time -StartupType Automatic
    ```
5. 將電源設定檔設為 [高效能]：

    ```PowerShell
    powercfg /setactive SCHEME_MIN
    ```

## <a name="check-the-windows-services"></a>檢查 Windows 服務
確定以下的每個 Windows 服務都已設為 **Windows 預設值**。 這些是必須設定來確定 VM 具有連線能力的服務最小數目。 若要重設啟動設定，請執行下列命令：
   
```PowerShell
Set-Service -Name bfe -StartupType Automatic
Set-Service -Name dhcp -StartupType Automatic
Set-Service -Name dnscache -StartupType Automatic
Set-Service -Name IKEEXT -StartupType Automatic
Set-Service -Name iphlpsvc -StartupType Automatic
Set-Service -Name netlogon -StartupType Manual
Set-Service -Name netman -StartupType Manual
Set-Service -Name nsi -StartupType Automatic
Set-Service -Name termService -StartupType Manual
Set-Service -Name MpsSvc -StartupType Automatic
Set-Service -Name RemoteRegistry -StartupType Automatic
```

## <a name="update-remote-desktop-registry-settings"></a>更新遠端桌面登錄設定
確定已針對遠端桌面連線正確設定下列設定：

>[!Note] 
>當您在這些步驟中執行 **Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services -name &lt;物件名稱&gt; &lt;值&gt;** 時，可能會收到一則錯誤訊息。 您可以放心地忽略該錯誤訊息。 它只是表示網域不是透過群組原則物件來推送該設定。
>
>

1. 遠端桌面通訊協定 (RDP) 已啟用：
   
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0 -Type DWord

    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -name "fDenyTSConnections" -Value 0 -Type DWord
    ```
   
2. RDP 連接埠已正確設定 (預設連接埠 3389)：
   
    ```PowerShell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "PortNumber" 3389 -Type DWord
    ```
    當您部署 VM 時，會針對連接埠 3389 建立預設規則。 如果您想要變更連接埠號碼，請在 Azure 中部署 VM 之後進行。

3. 接聽程式正在每個網路介面中進行接聽：
   
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "LanAdapter" 0 -Type DWord
   ```
4. 設定 RDP 連線的網路層級驗證模式：
   
    ```PowerShell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" 1 -Type DWord

    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "SecurityLayer" 1 -Type DWord

    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "fAllowSecProtocolNegotiation" 1 -Type DWord
     ```

5. 設定 Keep-Alive 值：
    
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -name "KeepAliveEnable" 1 -Type DWord
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -name "KeepAliveInterval" 1 -Type DWord
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "KeepAliveTimeout" 1 -Type DWord
    ```
6. 重新連線：
    
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -name "fDisableAutoReconnect" 0 -Type DWord
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "fInheritReconnectSame" 1 -Type DWord
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "fReconnectSame" 0 -Type DWord
    ```
7. 並行連線數目的限制：
    
    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -name "MaxInstanceCount" 4294967295 -Type DWord
    ```
8. 如果有任何自我簽署憑證繫結至 RDP 接聽程式，請移除它們：
    
    ```PowerShell
    Remove-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "SSLCertificateSHA1Hash"
    ```
    這是為了確定您在部署 VM 時，一開始就能連線。 您也可以視需要，在 Azure 中部署 VM 之後的後續階段中檢閱這部分。

9. 如果 VM 將為網域的一部分，請檢查下列所有設定，以確定不會還原先前設定。 以下為必須檢查的原則：
    
    | 目標                                     | 原則                                                                                                                                                       | 值                                                                                    |
    |------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
    | RDP 已啟用                           | 電腦設定\原則\Windows 設定\系統管理範本\元件\遠端桌面服務\遠端桌面工作階段主機\連線         | 允許使用者使用遠端桌面服務從遠端連線                                  |
    | NLA 群組原則                         | 設定\系統管理範本\元件\遠端桌面服務\遠端桌面工作階段主機\安全性                                                    | 透過使用網路層級驗證以要求對遠端連線進行使用者驗證 |
    | Keep Alive 設定                      | 電腦設定\原則\Windows 設定\系統管理範本\Windows 元件\遠端桌面服務\遠端桌面工作階段主機\連線 | 設定 Keep-Alive 連線間隔                                                 |
    | 重新連線設定                       | 電腦設定\原則\Windows 設定\系統管理範本\Windows 元件\遠端桌面服務\遠端桌面工作階段主機\連線 | 自動重新連線                                                                   |
    | 限制連線數目的設定 | 電腦設定\原則\Windows 設定\系統管理範本\Windows 元件\遠端桌面服務\遠端桌面工作階段主機\連線 | 限制連線數目                                                              |

## <a name="configure-windows-firewall-rules"></a>設定 Windows 防火牆規則
1. 在這三個設定檔 (網域、標準和公用) 上開啟 Windows 防火牆：

   ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\DomainProfile' -name "EnableFirewall" -Value 1 -Type DWord
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\PublicProfile' -name "EnableFirewall" -Value 1 -Type DWord
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\Standardprofile' -name "EnableFirewall" -Value 1 -Type DWord
   ```

2. 在 PowerShell 中執行下列命令，以允許 WinRM 透過這三種防火牆設定檔 (網域、私人和公用)，並啟用 PowerShell 遠端服務：
   
   ```PowerShell
    Enable-PSRemoting -force
    netsh advfirewall firewall set rule dir=in name="Windows Remote Management (HTTP-In)" new enable=yes
    netsh advfirewall firewall set rule dir=in name="Windows Remote Management (HTTP-In)" new enable=yes
   ```
3. 啟用下列防火牆規則以允許 RDP 流量：

   ```PowerShell
    netsh advfirewall firewall set rule group="Remote Desktop" new enable=yes
   ```   
4. 啟用檔案及印表機共用規則，讓 VM 可以在虛擬網路內回應 ping 命令：

   ```PowerShell
    netsh advfirewall firewall set rule dir=in name="File and Printer Sharing (Echo Request - ICMPv4-In)" new enable=yes
   ``` 
5. 如果 VM 將為網域的一部分，請檢查下列設定，以確定不會還原先前設定。 以下為必須檢查的 AD 原則：

    | 目標                                 | 原則                                                                                                                                                  | 值                                   |
    |--------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------|
    | 啟用 Windows 防火牆設定檔 | 電腦設定\原則\Windows 設定\系統管理範本\網路\網路連線\Windows 防火牆\網域設定檔\Windows 防火牆   | 保護所有網路連線         |
    | 啟用 RDP                           | 電腦設定\原則\Windows 設定\系統管理範本\網路\網路連線\Windows 防火牆\網域設定檔\Windows 防火牆   | 允許輸入遠端桌面例外狀況 |
    |                                      | 電腦設定\原則\Windows 設定\系統管理範本\網路\網路連線\Windows 防火牆\標準設定檔\Windows 防火牆 | 允許輸入遠端桌面例外狀況 |
    | 啟用 ICMP-V4                       | 電腦設定\原則\Windows 設定\系統管理範本\網路\網路連線\Windows 防火牆\網域設定檔\Windows 防火牆   | 允許 ICMP 例外狀況                   |
    |                                      | 電腦設定\原則\Windows 設定\系統管理範本\網路\網路連線\Windows 防火牆\標準設定檔\Windows 防火牆 | 允許 ICMP 例外狀況                   |

## <a name="verify-vm-is-healthy-secure-and-accessible-with-rdp"></a>確認 VM 處於狀況良好、安全且可使用 RDP 存取 
1. 若要確定磁碟是狀況良好且一致的，在下一個 VM 重新啟動時，執行檢查磁碟作業：

    ```PowerShell
    Chkdsk /f
    ```
    請確定此報告會顯示全新且狀況良好的磁碟。

2. 設定開機組態資料 (BCD) 設定。 

    > [!Note]
    > 確定您是在已提升權限的 PowerShell 視窗上執行這些命令。
   
   ```powershell
    cmd

    bcdedit /set {bootmgr} integrityservices enable
    bcdedit /set {default} device partition=C:
    bcdedit /set {default} integrityservices enable
    bcdedit /set {default} recoveryenabled Off
    bcdedit /set {default} osdevice partition=C:
    bcdedit /set {default} bootstatuspolicy IgnoreAllFailures

    #Enable Serial Console Feature
    bcdedit /set {bootmgr} displaybootmenu yes
    bcdedit /set {bootmgr} timeout 10
    bcdedit /set {bootmgr} bootems yes
    bcdedit /ems {current} ON
    bcdedit /emssettings EMSPORT:1 EMSBAUDRATE:115200

    exit
   ```
3. 傾印記錄檔能幫助您進行 Windows 損毀問題的疑難排解。 啟用傾印記錄檔收集：

    ```powershell
    cmd

    #Setup the Guest OS to collect a kernel dump on an OS crash event
    REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 2 /f
    REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    #Setup the Guest OS to collect user mode dumps on a service crash event
    md c:\Crashdumps
    REG ADD "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting" /v DumpFolder /t REG_EXPAND_SZ /d "c:\CrashDumps" /f
    REG ADD "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting" /v CrashCount /t REG_DWORD /d 10 /f
    REG ADD "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting" /v DumpType /t REG_DWORD /d 2 /f
    sc config WerSvc start= demand

    exit
    
    ```
4. 確認 Windows Management Instrumentation 存放庫是一致的。 若要執行此動作，請執行下列命令：

    ```PowerShell
    winmgmt /verifyrepository
    ```
    如果存放庫損毀，請參閱 [WMI︰存放庫損毀，還是沒有損毀](https://blogs.technet.microsoft.com/askperf/2014/08/08/wmi-repository-corruption-or-not) \(英文\)。

5. 確定沒有任何其他應用程式使用連接埠 3389。 在 Azure 中，此連接埠是由 RDP 服務所使用。 您可以執行 **netstat anob** 來查看 VM 上使用了哪些連接埠：

    ```PowerShell
    netstat -anob
    ```

6. 如果您想要上傳的 Windows VHD 是一個網域控制站，則請遵循這些步驟：

    1. 遵循[這些額外的步驟](https://support.microsoft.com/kb/2904015)來準備磁碟。

    1. 確定您知道 DSRM 密碼，以防您必須在某個時間點於 DSRM 中啟動 VM。 您可能想要參考此連結來設定 [DSRM 密碼](https://technet.microsoft.com/library/cc754363(v=ws.11).aspx) \(英文\)。

7. 確定您知道內建的系統管理員帳戶和密碼。 您可能想要重設目前的本機系統管理員密碼，並確定您可以使用此帳戶，透過 RDP 連線登入 Windows。 此存取權限會受到「允許透過遠端桌面服務登入」群組原則物件所控制。 您可以於下列位置的本機群組原則編輯器中檢視此物件：

    電腦設定\Windows 設定\安全性設定\本機原則\使用者權限指派

8. 請檢查下列 AD 原則，確定您並未封鎖透過 RDP 或來自網路的 RDP 存取：

    - 電腦設定\Windows 設定\安全性設定\本機原則\使用者權限指派\拒絕從網路存取此電腦

    - 電腦設定\Windows 設定\安全性設定\本機原則\使用者權限指派\拒絕從遠端桌面服務登入


9. 檢查下列 AD 原則，確定您未移除下列任何一項所需的存取帳戶：

    - 電腦設定\Windows 設定\安全性設定\本機原則\使用者權限指派\從網路存取此電腦

    下列群組應列在此原則上：

    - 系統管理員
    - 備份操作員
    - 所有人
    - 使用者

10. 重新啟動 VM，以確保 Windows 仍然狀況良好，可使用 RDP 連線來達成。 此時，您可能想要在本機 Hyper-V 中建立 VM，以確定 VM 已完全啟動，然後測試是否可連線到 RDP。

11. 移除任何額外的傳輸驅動程式介面篩選，例如分析 TCP 封包的軟體或額外的防火牆。 您也可以視需要，在 Azure 中部署 VM 之後的後續階段中檢閱這部分。

12. 解除安裝與實體元件或任何其他虛擬化技術相關的所有其他協力廠商軟體和驅動程式。

### <a name="install-windows-updates"></a>安裝 Windows 更新
理想的設定是**具有最新的電腦修補程式等級**。 如果這不可行，請確定已安裝下列更新：

| 元件               | Binary         | Windows 7 SP1、Windows Server 2008 R2 SP1 | Windows 8、Windows Server 2012               | Windows 8.1、Windows Server 2012 R2 | Windows 10 版本 1607、Windows Server 2016 版本 1607 | Windows 10 版本 1703    | Windows 10 版本 1709、Windows Server 2016 版本 1709 | Windows 10 版本 1803、Windows Server 2016 版本 1803 |
|-------------------------|----------------|-------------------------------------------|---------------------------------------------|------------------------------------|---------------------------------------------------------|----------------------------|-------------------------------------------------|-------------------------------------------------|
| 儲存體                 | disk.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17638 / 6.2.9200.21757 - KB3137061 | 6.3.9600.18203 - KB3137061         | -                                                       | -                          | -                                               | -                                               |
|                         | storport.sys   | 6.1.7601.23403 - KB3125574                | 6.2.9200.17188 / 6.2.9200.21306 - KB3018489 | 6.3.9600.18573 - KB4022726         | 10.0.14393.1358 - KB4022715                             | 10.0.15063.332             | -                                               | -                                               |
|                         | ntfs.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17623 / 6.2.9200.21743 - KB3121255 | 6.3.9600.18654 - KB4022726         | 10.0.14393.1198 - KB4022715                             | 10.0.15063.447             | -                                               | -                                               |
|                         | Iologmsg.dll   | 6.1.7601.23403 - KB3125574                | 6.2.9200.16384 - KB2995387                  | -                                  | -                                                       | -                          | -                                               | -                                               |
|                         | Classpnp.sys   | 6.1.7601.23403 - KB3125574                | 6.2.9200.17061 / 6.2.9200.21180 - KB2995387 | 6.3.9600.18334 - KB3172614         | 10.0.14393.953 - KB4022715                              | -                          | -                                               | -                                               |
|                         | Volsnap.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.17047 / 6.2.9200.21165 - KB2975331 | 6.3.9600.18265 - KB3145384         | -                                                       | 10.0.15063.0               | -                                               | -                                               |
|                         | partmgr.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.16681 - KB2877114                  | 6.3.9600.17401 - KB3000850         | 10.0.14393.953 - KB4022715                              | 10.0.15063.0               | -                                               | -                                               |
|                         | volmgr.sys     |                                           |                                             |                                    |                                                         | 10.0.15063.0               | -                                               | -                                               |
|                         | Volmgrx.sys    | 6.1.7601.23403 - KB3125574                | -                                           | -                                  | -                                                       | 10.0.15063.0               | -                                               | -                                               |
|                         | Msiscsi.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.21006 - KB2955163                  | 6.3.9600.18624 - KB4022726         | 10.0.14393.1066 - KB4022715                             | 10.0.15063.447             | -                                               | -                                               |
|                         | Msdsm.sys      | 6.1.7601.23403 - KB3125574                | 6.2.9200.21474 - KB3046101                  | 6.3.9600.18592 - KB4022726         | -                                                       | -                          | -                                               | -                                               |
|                         | Mpio.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.21190 - KB3046101                  | 6.3.9600.18616 - KB4022726         | 10.0.14393.1198 - KB4022715                             | -                          | -                                               | -                                               |
|                         | vmstorfl.sys   | 6.3.9600.18907 - KB4072650                | 6.3.9600.18080 - KB3063109                  | 6.3.9600.18907 - KB4072650         | 10.0.14393.2007 - KB4345418                             | 10.0.15063.850 - KB4345419 | 10.0.16299.371 - KB4345420                      | -                                               |
|                         | Fveapi.dll     | 6.1.7601.23311 - KB3125574                | 6.2.9200.20930 - KB2930244                  | 6.3.9600.18294 - KB3172614         | 10.0.14393.576 - KB4022715                              | -                          | -                                               | -                                               |
|                         | Fveapibase.dll | 6.1.7601.23403 - KB3125574                | 6.2.9200.20930 - KB2930244                  | 6.3.9600.17415 - KB3172614         | 10.0.14393.206 - KB4022715                              | -                          | -                                               | -                                               |
| 網路                 | netvsc.sys     | -                                         | -                                           | -                                  | 10.0.14393.1198 - KB4022715                             | 10.0.15063.250 - KB4020001 | -                                               | -                                               |
|                         | mrxsmb10.sys   | 6.1.7601.23816 - KB4022722                | 6.2.9200.22108 - KB4022724                  | 6.3.9600.18603 - KB4022726         | 10.0.14393.479 - KB4022715                              | 10.0.15063.483             | -                                               | -                                               |
|                         | mrxsmb20.sys   | 6.1.7601.23816 - KB4022722                | 6.2.9200.21548 - KB4022724                  | 6.3.9600.18586 - KB4022726         | 10.0.14393.953 - KB4022715                              | 10.0.15063.483             | -                                               | -                                               |
|                         | mrxsmb.sys     | 6.1.7601.23816 - KB4022722                | 6.2.9200.22074 - KB4022724                  | 6.3.9600.18586 - KB4022726         | 10.0.14393.953 - KB4022715                              | 10.0.15063.0               | -                                               | -                                               |
|                         | tcpip.sys      | 6.1.7601.23761 - KB4022722                | 6.2.9200.22070 - KB4022724                  | 6.3.9600.18478 - KB4022726         | 10.0.14393.1358 - KB4022715                             | 10.0.15063.447             | -                                               | -                                               |
|                         | http.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17285 - KB3042553                  | 6.3.9600.18574 - KB4022726         | 10.0.14393.251 - KB4022715                              | 10.0.15063.483             | -                                               | -                                               |
|                         | vmswitch.sys   | 6.1.7601.23727 - KB4022719                | 6.2.9200.22117 - KB4022724                  | 6.3.9600.18654 - KB4022726         | 10.0.14393.1358 - KB4022715                             | 10.0.15063.138             | -                                               | -                                               |
| 核心                    | ntoskrnl.exe   | 6.1.7601.23807 - KB4022719                | 6.2.9200.22170 - KB4022718                  | 6.3.9600.18696 - KB4022726         | 10.0.14393.1358 - KB4022715                             | 10.0.15063.483             | -                                               | -                                               |
| 遠端桌面服務問題 | rdpcorets.dll  | 6.2.9200.21506 - KB4022719                | 6.2.9200.22104 - KB4022724                  | 6.3.9600.18619 - KB4022726         | 10.0.14393.1198 - KB4022715                             | 10.0.15063.0               | -                                               | -                                               |
|                         | termsrv.dll    | 6.1.7601.23403 - KB3125574                | 6.2.9200.17048 - KB2973501                  | 6.3.9600.17415 - KB3000850         | 10.0.14393.0 - KB4022715                                | 10.0.15063.0               | -                                               | -                                               |
|                         | termdd.sys     | 6.1.7601.23403 - KB3125574                | -                                           | -                                  | -                                                       | -                          | -                                               | -                                               |
|                         | win32k.sys     | 6.1.7601.23807 - KB4022719                | 6.2.9200.22168 - KB4022718                  | 6.3.9600.18698 - KB4022726         | 10.0.14393.594 - KB4022715                              | -                          | -                                               | -                                               |
|                         | rdpdd.dll      | 6.1.7601.23403 - KB3125574                | -                                           | -                                  | -                                                       | -                          | -                                               | -                                               |
|                         | rdpwd.sys      | 6.1.7601.23403 - KB3125574                | -                                           | -                                  | -                                                       | -                          | -                                               | -                                               |
| 安全性                | MS17-010       | KB4012212                                 | KB4012213                                   | KB4012213                          | KB4012606                                               | KB4012606                  | -                                               | -                                               |
|                         |                |                                           | KB4012216                                   |                                    | KB4013198                                               | KB4013198                  | -                                               | -                                               |
|                         |                | KB4012215                                 | KB4012214                                   | KB4012216                          | KB4013429                                               | KB4013429                  | -                                               | -                                               |
|                         |                |                                           | KB4012217                                   |                                    | KB4013429                                               | KB4013429                  | -                                               | -                                               |
|                         | CVE-2018-0886  | KB4103718               | KB4103730                | KB4103725       | KB4103723                                               | KB4103731                  | KB4103727                                       | KB4103721                                       |
|                         |                | KB4103712          | KB4103726          | KB4103715|                                                         |                            |                                                 |                                                 |
       
### 使用 sysprep 的時機 <a id="step23"></a>    

Sysprep 是您可執行來進行 Windows 安裝的程序，將重設系統安裝，且將藉由移除所有個人資料並重設數個元件來提供「全新體驗」。 如果您想要建立一個範本，以部署數個其他具有特定設定的 VM，通常會執行此動作。 這稱為**一般化映像**。

但若您只想從一部磁碟建立一個 VM，就不需使用 sysprep。 在此情況下，您只需從所謂的**特殊化映像**建立 VM 即可。

如需如何從特殊化磁碟建立 VM 的詳細資訊，請參閱：

- [從特殊化磁碟建立 VM](create-vm-specialized.md)
- [從特殊化 VHD 磁碟建立 VM](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/create-vm-specialized-portal?branch=master)

如果您想要建立一般化映像，就必須執行 sysprep。 如需 Sysprep 的詳細資訊，請參閱[如何使用 Sysprep：簡介](http://technet.microsoft.com/library/bb457073.aspx) \(英文\)。 

並非 Windows 電腦上安裝的每個角色或應用程式都支援這個一般化。 因此，在執行此程序之前，請先參閱下列文章，以確定 sysprep 支援該電腦的角色。 如需詳細資訊，請參閱[伺服器角色的 Sysprep 支援](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles) \(英文\)。

### <a name="steps-to-generalize-a-vhd"></a>將 VHD 一般化的步驟

>[!NOTE]
> 當您執行 sysprep.exe 之後 (如下列步驟中所指定)，請關閉 VM，且在您於 Azure 中建立它的映像之前，不要再度開啟它。

1. 登入 Windows VM。
2. 以系統管理員身分執行**命令提示字元**。 
3. 將目錄切換至：**%windir%\system32\sysprep**，然後執行 **sysprep.exe**。
3. 在 [系統準備工具] 對話方塊中，選取 [進入系統全新體驗 (OOBE)]，並確認已勾選 [一般化] 核取方塊。

    ![系統準備工具](media/prepare-for-upload-vhd-image/syspre.png)
4. 在 [關機選項] 中選取 [關機]。
5. 按一下 [確定]。
6. 當 Sysprep 完成時，關閉 VM。 不要使用**重新啟動**來關閉 VM。
7. 現在已準備好上傳 VHD。 如需如何從一般化磁碟建立 VM 的詳細資訊，請參閱[將一般化 VHD 上傳，並使用它在 Azure 中建立新的 VM](sa-upload-generalized.md)。


## <a name="complete-recommended-configurations"></a>完成建議的設定
下列設定不會影響 VHD 上傳。 不過，我們強烈建議您設定它們。

* 安裝 [Azure VM 代理程式](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。 然後您可以啟用 VM 擴充功能。 VM 擴充功能實作了您可能想要與 VM 搭配使用的大部分重要功能，例如重設密碼、設定 RDP 等功能。 如需詳細資訊，請參閱

    - [VM 代理程式與擴充功能 - 第 1 部分](https://azure.microsoft.com/blog/vm-agent-and-extensions-part-1/) \(英文\)
    - [VM 代理程式與擴充功能 - 第 2 部分](https://azure.microsoft.com/blog/vm-agent-and-extensions-part-2/) \(英文\)

*  在 Azure 中建立 VM 之後，我們建議您將分頁檔放在「暫存磁碟機」磁碟區中，以改善效能。 您可以如下方式設定這部分：

    ```PowerShell
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management' -name "PagingFiles" -Value "D:\pagefile"
    ```
如果沒有任何資料磁碟連接到 VM，暫存磁碟機磁碟區的磁碟機代號通常是 "D"。 根據可用的磁碟機數目和您所進行的設定而定，這項指定可能不同。

## <a name="next-steps"></a>後續步驟
* [將 Windows VM 映像上傳至 Azure 供 Resource Manager 部署使用](upload-generalized-managed.md)
* [針對 Azure Windows 虛擬機器啟用問題進行疑難排解](troubleshoot-activation-problems.md)

