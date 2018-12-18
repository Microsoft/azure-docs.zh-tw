---
title: 與 DMZ 搭配使用的 Azure 範例應用程式 | Microsoft Docs
description: 在建立 DMZ 來測試流量案例後部署這個簡單的 Web 應用程式
services: virtual-network
documentationcenter: na
author: tracsman
manager: rossort
editor: ''
ms.assetid: 60340ab7-b82b-40e0-bd87-83e41fe4519c
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/03/2017
ms.author: jonor
ms.openlocfilehash: 8506238e41c5d9dac8d76d729d4919b30a0528b9
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23124346"
---
# <a name="sample-application-for-use-with-dmzs"></a>與 DMZ 搭配使用的範例應用程式
[返回 [安全性界限最佳作法] 頁面][HOME]

這些 PowerShell 指令碼可在 IIS01 和 AppVM01 伺服器本機上執行，用來安裝和設定一個簡單的 Web 應用程式，以顯示一個來自前端 IIS01 伺服器而內容來自後端 AppVM01 的 html 頁面。

此應用程式提供簡單的測試環境，可測試許多 DMZ 範例，以及端點、NSG、UDR 及防火牆規則的變更對流量的影響。

## <a name="firewall-rule-to-allow-icmp"></a>允許 ICMP 的防火牆規則
這個簡單的 PowerShell 陳述式可以在任何 Windows VM 上執行，以允許 ICMP (Ping) 流量。 此防火牆更新藉由允許 ping 通訊協定通過 Windows 防火牆，讓您能夠更輕鬆進行測試和疑難排解 (ICMP 在大多數 Linux 散發版本上預設為開啟)。

```PowerShell
# Turn On ICMPv4
New-NetFirewallRule -Name Allow_ICMPv4 -DisplayName "Allow ICMPv4" `
    -Protocol ICMPv4 -Enabled True -Profile Any -Action Allow
```

如果您使用下列指令碼，則這個新增的防火牆規則是第一個陳述式。

## <a name="iis01---web-application-installation-script"></a>IIS01 - Web 應用程式安裝指令碼
此指令碼會：

1. 在本機伺服器 Windows 防火牆開啟 IMCPv4 (Ping) 以方便測試
2. 安裝 IIS 和 .Net Framework v4.5
3. 建立 ASP.NET 網頁和 Web.config 檔案
4. 變更預設應用程式集區以方便存取檔案
5. 將匿名使用者設定為您的系統管理員帳戶和密碼

在透過遠端桌面 (RDP) 存取 IIS01 時，這個 PowerShell 指令碼應該會在本機執行。

```PowerShell
# IIS Server Post Build Config Script
# Get Admin Account and Password
    Write-Host "Please enter the admin account information used to create this VM:" -ForegroundColor Cyan
    $theAdmin = Read-Host -Prompt "The Admin Account Name (no domain or machine name)"
    $thePassword = Read-Host -Prompt "The Admin Password"

# Turn On ICMPv4
    Write-Host "Creating ICMP Rule in Windows Firewall" -ForegroundColor Cyan
    New-NetFirewallRule -Name Allow_ICMPv4 -DisplayName "Allow ICMPv4" -Protocol ICMPv4 -Enabled True -Profile Any -Action Allow

# Install IIS
    Write-Host "Installing IIS and .Net 4.5, this can take some time, like 15+ minutes..." -ForegroundColor Cyan
    add-windowsfeature Web-Server, Web-WebServer, Web-Common-Http, Web-Default-Doc, Web-Dir-Browsing, Web-Http-Errors, Web-Static-Content, Web-Health, Web-Http-Logging, Web-Performance, Web-Stat-Compression, Web-Security, Web-Filtering, Web-App-Dev, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Net-Ext, Web-Net-Ext45, Web-Asp-Net45, Web-Mgmt-Tools, Web-Mgmt-Console

# Create Web App Pages
    Write-Host "Creating Web page and Web.Config file" -ForegroundColor Cyan
    $MainPage = '<%@ Page Language="vb" AutoEventWireup="false" %>
    <%@ Import Namespace="System.IO" %>
    <script language="vb" runat="server">
        Protected Sub Page_Load(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.Load
            Dim FILENAME As String = "\\10.0.2.5\WebShare\Rand.txt"
            Dim objStreamReader As StreamReader
            objStreamReader = File.OpenText(FILENAME)
            Dim contents As String = objStreamReader.ReadToEnd()
            lblOutput.Text = contents
            objStreamReader.Close()
            lblTime.Text = Now()
        End Sub
    </script>

    <!DOCTYPE html>
    <html xmlns="http://www.w3.org/1999/xhtml">
    <head runat="server">
        <title>DMZ Example App</title>
    </head>
    <body style="font-family: Optima,Segoe,Segoe UI,Candara,Calibri,Arial,sans-serif;">
      <form id="frmMain" runat="server">
        <div>
          <h1>Looks like you made it!</h1>
          This is a page from the inside (a web server on a private network),<br />
          and it is making its way to the outside! (If you are viewing this from the internet)<br />
          <br />
          The following sections show:
          <ul style="margin-top: 0px;">
            <li> Local Server Time - Shows if this page is or isnt cached anywhere</li>
            <li> File Output - Shows that the web server is reaching AppVM01 on the backend subnet and successfully returning content</li>
            <li> Image from the Internet - Doesnt really show anything, but it made me happy to see this when the app worked</li>
          </ul>
          <div style="border: 2px solid #8AC007; border-radius: 25px; padding: 20px; margin: 10px; width: 650px;">
            <b>Local Web Server Time</b>: <asp:Label runat="server" ID="lblTime" /></div>
          <div style="border: 2px solid #8AC007; border-radius: 25px; padding: 20px; margin: 10px; width: 650px;">
            <b>File Output from AppVM01</b>: <asp:Label runat="server" ID="lblOutput" /></div>
          <div style="border: 2px solid #8AC007; border-radius: 25px; padding: 20px; margin: 10px; width: 650px;">
            <b>Image File Linked from the Internet</b>:<br />
            <br />
            <img src="http://sd.keepcalm-o-matic.co.uk/i/keep-calm-you-made-it-7.png" alt="You made it!" width="150" length="175"/></div>
        </div>
      </form>
    </body>
    </html>'

    $WebConfig ='<?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <system.web>
        <compilation debug="true" strict="false" explicit="true" targetFramework="4.5" />
        <httpRuntime targetFramework="4.5" />
        <identity impersonate="true" />
        <customErrors mode="Off"/>
      </system.web>
      <system.webServer>
        <defaultDocument>
          <files>
            <add value="Home.aspx" />
          </files>
        </defaultDocument>
      </system.webServer>
    </configuration>'

    $MainPage | Out-File -FilePath "C:\inetpub\wwwroot\Home.aspx" -Encoding ascii
    $WebConfig | Out-File -FilePath "C:\inetpub\wwwroot\Web.config" -Encoding ascii

# Set App Pool to Clasic Pipeline to remote file access will work easier
    Write-Host "Updaing IIS Settings" -ForegroundColor Cyan
    c:\windows\system32\inetsrv\appcmd.exe set app "Default Web Site/" /applicationPool:".NET v4.5 Classic"
    c:\windows\system32\inetsrv\appcmd.exe set config "Default Web Site/" /section:system.webServer/security/authentication/anonymousAuthentication /userName:$theAdmin /password:$thePassword /commit:apphost

# Make sure the IIS settings take
    Write-Host "Restarting the W3SVC" -ForegroundColor Cyan
    Restart-Service -Name W3SVC

    Write-Host
    Write-Host "Web App Creation Successfull!" -ForegroundColor Green
    Write-Host
```

## <a name="appvm01---file-server-installation-script"></a>AppVM01 - 檔案伺服器安裝指令碼
此指令碼會設定這個簡單應用程式的後端。 此指令碼會：

1. 在防火牆開啟 IMCPv4 (Ping) 以方便測試
2. 建立網站的目錄
3. 建立網頁所要遠端存取的文字檔
4. 將目錄和檔案的權限設為匿名以允許存取
5. 關閉 IE 增強式安全性以允許更輕鬆地從這部伺服器瀏覽 

> [!IMPORTANT]
> **最佳做法**：永遠不要關閉實際執行伺服器上的「IE 增強式安全性」，而且從實際執行伺服器上網通常不是個好主意。 此外，最好不要開啟檔案共用來供匿名存取，這裡這樣做是為了簡單起見。
> 
> 

在透過遠端桌面 (RDP) 存取 AppVM01 時，這個 PowerShell 指令碼應該會在本機執行。 PowerShell 必須以系統管理員身分執行，才能成功執行。

```PowerShell
# AppVM01 Server Post Build Config Script
# PowerShell must be run as Administrator for Net Share commands to work

# Turn On ICMPv4
    New-NetFirewallRule -Name Allow_ICMPv4 -DisplayName "Allow ICMPv4" -Protocol ICMPv4 -Enabled True -Profile Any -Action Allow

# Create Directory
    New-Item "C:\WebShare" -ItemType Directory

# Write out Rand.txt
    $FileContent = "Hello, I'm the contents of a remote file on AppVM01."
    $FileContent | Out-File -FilePath "C:\WebShare\Rand.txt" -Encoding ascii

# Set Permissions on share
    $Acl = Get-Acl "C:\WebShare"
    $AccessRule = New-Object system.security.accesscontrol.filesystemaccessrule("Everyone","ReadAndExecute, Synchronize","ContainerInherit, ObjectInherit","InheritOnly","Allow")
    $Acl.SetAccessRule($AccessRule)
    Set-Acl "C:\WebShare" $Acl

# Create network share
    Net Share WebShare=C:\WebShare "/grant:Everyone,READ"

# Turn Off IE Enhanced Security Configuration for Admins
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0

    Write-Host
    Write-Host "File Server Set up Successfull!" -ForegroundColor Green
    Write-Host
```

## <a name="dns01---dns-server-installation-script"></a>DNS01 - DNS 伺服器安裝指令碼
此範例應用程式中未包含任何要設定 DNS 伺服器的指令碼。 如果防火牆規則、NSG 或 UDR 的測試需要包含 DNS 流量，就必須手動安裝 DNS01 伺服器。 這兩個範例的「網路組態」XML 檔案和「Resource Manager 範本」都包含 DNS01 作為主要 DNS 伺服器，而由「層級 3」裝載的公用 DNS 伺服器則作為備份 DNS 伺服器。 「層級 3」DNS 伺服器會是非本機流量所使用的實際 DNS 伺服器，而如果未安裝 DNS01，則不會發生區域網路 DNS。

## <a name="next-steps"></a>後續步驟
* 在 IIS 伺服器上執行 IIS01 指令碼
* 在 AppVM01 上執行檔案伺服器指令碼
* 瀏覽至 IIS01 上的公用 IP 以驗證您的組建

<!--Link References-->
[HOME]: ../best-practices-network-security.md
