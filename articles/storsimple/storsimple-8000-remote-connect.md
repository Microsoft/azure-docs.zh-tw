---
title: 遠端連線至 StorSimple 裝置 | Microsoft Docs
description: 說明如何設定您的裝置以進行遠端管理，以及如何透過 HTTP 或 HTTPS 連線到 Windows PowerShell for StorSimple。
services: storsimple
documentationcenter: ''
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/02/2018
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 9414d9c93fe463910ffa6fce72aada6a0d720464
ms.sourcegitcommit: 79683e67911c3ab14bcae668f7551e57f3095425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/25/2018
ms.locfileid: "28103950"
---
# <a name="connect-remotely-to-your-storsimple-8000-series-device"></a>遠端連線至 StorSimple 8000 系列裝置

## <a name="overview"></a>概觀

您可以透過 Windows PowerShell，從遠端連線到您的裝置。 當您以這種方式連線時，不會看到功能表。 (只有當您使用裝置上的序列主控台進行連線時，才會看到功能表)。使用 Windows PowerShell 遠端連線到特定的 Runspace。 您也可以指定顯示語言。

如需有關如何使用 Windows PowerShell 遠端來管理裝置的詳細資訊，請移至 [使用 Windows PowerShell for StorSimple 管理您的 StorSimple 裝置](storsimple-8000-windows-powershell-administration.md)。

本教學課程說明如何設定您的裝置以進行遠端管理，以及如何連線到 Windows PowerShell for StorSimple。 您可以透過 Windows PowerShell 使用 HTTP 或 HTTPS 從遠端連線。 不過，當您決定如何連線到適用於 StorSimple 的 Windows PowerShell 時，請考慮下列資訊：

* 直接連線至裝置序列主控台是安全的，但透過網路交換器連線至序列主控台則否。 透過網路交換器連線至裝置序列主控台時，請務必注意安全性風險。
* 透過 HTTP 工作階段連線的安全性，比在網路上透過序列主控台連線更高。 雖然這不是最安全的方法，但在受信任的網路上是可接受的做法。
* 透過 HTTP 工作階段連線並使用自我簽署憑證，是最安全且建議的選項。

您可以從遠端連線至 Windows PowerShell 介面。 不過，透過 Windows PowerShell 介面遠端存取您的 StorSimple 裝置依預設不會啟用。 您必須先在裝置上啟用遠端管理，然後在要用來存取裝置的用戶端上啟用。

本文章中所述的步驟是在執行 Windows Server 2012 R2 的主機系統上執行。

## <a name="connect-through-http"></a>透過 HTTP 連線

透過 HTTP 工作階段連線至 Windows PowerShell for StorSimple 的安全性，比透過 StorSimple 裝置的序列主控台連線更高。 雖然這不是最安全的方法，但在受信任的網路上是可接受的做法。

您可以使用 Azure 入口網站或序列主控台來設定遠端管理。 選擇下列程序之一：

* [使用 Azure 入口網站來啟用透過 HTTP 的遠端管理](#use-the-azure-classic-portal-to-enable-remote-management-over-http)
* [使用序列主控台啟用透過 HTTP 的遠端管理](#use-the-serial-console-to-enable-remote-management-over-http)

啟用遠端管理後，使用下列程序準備遠端連線的用戶端。

* [準備遠端連線的用戶端](#prepare-the-client-for-remote-connection)

### <a name="use-the-azure-portal-to-enable-remote-management-over-http"></a>使用 Azure 入口網站來啟用透過 HTTP 的遠端管理

在 Azure 入口網站中執行下列步驟以啟用透過 HTTP 的遠端管理。

#### <a name="to-enable-remote-management-through-the-azure-portal"></a>透過 Azure 入口網站啟用遠端管理

1. 移至您的 StorSimple 裝置管理員服務。 選取 [裝置]，然後選取並按一下您想要設定遠端管理的裝置。 移至 [裝置設定] > [安全性]。
2. 在 [安全性設定] 刀鋒視窗中，按一下 [遠端管理]。
3. 在 [遠端管理] 刀鋒視窗中，將 [啟用遠端管理] 設定為 [是]。
4. 您現在可以選擇使用 HTTP 來連接。 (預設是透過 HTTPS 來連線。)請確定已選取 HTTP。
   
   > [!NOTE]
   > 只有受信任的網路上才接受透過 HTTP 來連接。
   
5. 按一下 [儲存]，系統提示您進行確認時，選取 [是]。

### <a name="use-the-serial-console-to-enable-remote-management-over-http"></a>使用序列主控台啟用透過 HTTP 的遠端管理
在裝置的序列主控台上執行下列步驟以啟用遠端管理。

#### <a name="to-enable-remote-management-through-the-device-serial-console"></a>使用裝置序列主控台啟用遠端管理
1. 在序列主控台的功能表中，選取選項 1。 如需有關如何使用裝置序列主控台的詳細資訊，請移至[透過裝置序列主控台連線到 Windows PowerShell for StorSimple](storsimple-8000-windows-powershell-administration.md#connect-to-windows-powershell-for-storsimple-via-the-device-serial-console)。
2. 在出現提示時輸入： `Enable-HcsRemoteManagement –AllowHttp`
3. 您會看到使用 HTTP 來連線至裝置的安全性漏洞的相關通知。 出現提示時，輸入 **Y**確認。
4. 確認 HTTP 已啟用，做法是輸入: `Get-HcsSystem`
5. 確認 [RemoteManagementMode] 欄位顯示 **HttpsAndHttpEnabled**。下圖顯示 PuTTY 中的這些設定。
   
     ![序列 HTTPS 和 HTTP 已啟用](./media/storsimple-remote-connect/HCS_SerialHttpsAndHttpEnabled.png)

### <a name="prepare-the-client-for-remote-connection"></a>準備遠端連線的用戶端
在用戶端上執行下列步驟以啟用遠端管理。

#### <a name="to-prepare-the-client-for-remote-connection"></a>準備遠端連線的用戶端
1. 以系統管理員的身分開啟 Windows PowerShell工作階段。 如果使用 Windows 10 用戶端，Windows 遠端管理服務預設會設定為手動。 您可能需要輸入下列命令以啟動服務：

    `Start-Service WinRM`
    
2. 輸入下列命令將 StorSimple 裝置的 IP 位址加入用戶端的信任的主機清單：
   
     `Set-Item wsman:\localhost\Client\TrustedHosts <device_ip> -Concatenate -Force`
   
     以您的裝置的 IP 位址取代其中的 <device_ip>，例如： 
   
     `Set-Item wsman:\localhost\Client\TrustedHosts 10.126.173.90 -Concatenate -Force`
3. 輸入下列命令將裝置認證儲存在變數中： 
   
    ```
    $cred = Get-Credential
    ```
    
4. 在出現的對話方塊中：
   
   1. 以此格式輸入使用者名稱：device_ip\SSAdmin。
   2. 輸入以安裝精靈設定裝置時設定的裝置管理員密碼。 預設密碼為 *Password1*。
5. 輸入以下命令在裝置上啟動 Windows PowerShell 工作階段：
   
     `Enter-PSSession -Credential $cred -ConfigurationName SSAdminConsole -ComputerName <device_ip>`
   
   > [!NOTE]
   > 若要建立與 StorSimple 虛擬裝置搭配使用的 Windows PowerShell 工作階段，請附加 `–Port` 參數，並指定您在 StorSimple 虛擬設備遠端處理中設定的公用連接埠。
   
   
此時，您應該有個連線到裝置的使用中遠端 Windows PowerShell 工作階段。
   
![使用 HTTPS 進行 PowerShell 遠端處理](./media/storsimple-remote-connect/HCS_PSRemotingUsingHTTP.png)

## <a name="connect-through-https"></a>透過 HTTP 連線

透過 HTTPS 工作階段連線到 Windows PowerShell for StorSimple，是從遠端連線至 Microsoft Azure StorSimple 裝置最安全且建議的方法。 下列程序說明如何設定序列主控台和用戶端電腦，讓您可以使用 HTTPS 連線到 Windows PowerShell for StorSimple。

您可以使用 Azure 入口網站或序列主控台來設定遠端管理。 選擇下列程序之一：

* [使用 Azure 入口網站來啟用透過 HTTPS 的遠端管理](#use-the-azure-classic-portal-to-enable-remote-management-over-https)
* [使用序列主控台啟用透過 HTTPS 的遠端管理](#use-the-serial-console-to-enable-remote-management-over-https)

啟用遠端管理後，使用下列程序準備遠端管理的主機，並從該遠端主機連線至裝置。

* [準備遠端管理的主機](#prepare-the-host-for-remote-management)
* [從遠端主機連線至裝置](#connect-to-the-device-from-the-remote-host)

### <a name="use-the-azure-portal-to-enable-remote-management-over-https"></a>使用 Azure 入口網站來啟用透過 HTTPS 的遠端管理

在 Azure 入口網站中執行下列步驟以啟用透過 HTTPS 的遠端管理。

#### <a name="to-enable-remote-management-over-https-from-the-azure-portal"></a>從 Azure 入口網站來啟用透過 HTTPS 的遠端管理

1. 移至您的 StorSimple 裝置管理員服務。 選取 [裝置]，然後選取並按一下您想要設定遠端管理的裝置。 移至 [裝置設定] > [安全性]。
2. 在 [安全性設定] 刀鋒視窗中，按一下 [遠端管理]。
3. 將 [啟用遠端管理] 設為 [是]。
4. 您現在可以選擇使用 HTTPS 來連線。 (預設是透過 HTTPS 來連線。)請確定已選取 HTTPS。
5. 按一下 [...]，然後按一下 [下載遠端管理憑證]。 指定要儲存此檔案的位置。 您必須將此憑證安裝在將用來連線到裝置的用戶端或主機電腦上。
6. 按一下 [儲存]，系統提示您進行確認時，按一下 [是]。

### <a name="use-the-serial-console-to-enable-remote-management-over-https"></a>使用序列主控台啟用透過 HTTPS 的遠端管理

在裝置的序列主控台上執行下列步驟以啟用遠端管理。

#### <a name="to-enable-remote-management-through-the-device-serial-console"></a>使用裝置序列主控台啟用遠端管理
1. 在序列主控台的功能表中，選取選項 1。 如需有關如何使用裝置序列主控台的詳細資訊，請移至[透過裝置序列主控台連線到 Windows PowerShell for StorSimple](storsimple-8000-windows-powershell-administration.md#connect-to-windows-powershell-for-storsimple-via-the-device-serial-console)。
2. 在出現提示時輸入：
   
     `Enable-HcsRemoteManagement`
   
    這應該會在您的裝置上啟用 HTTPS。
3. 確認 HTTPS 已啟用，做法是輸入： 
   
     `Get-HcsSystem`
   
    確定 [RemoteManagementMode] 欄位顯示 **HttpsEnabled**。下圖顯示 PuTTY 中的這些設定。
   
     ![序列 HTTPS 已啟用](./media/storsimple-remote-connect/HCS_SerialHttpsEnabled.png)
4. 從 `Get-HcsSystem`的輸出，複製裝置的序號，並儲存供稍後使用。
   
   > [!NOTE]
   > 序號對應至憑證中的 CN 名稱。
   
5. 取得遠端管理憑證，做法是輸入： 
   
     `Get-HcsRemoteManagementCert`
   
    將出現類似以下的憑證。
   
    ![取得遠端管理憑證](./media/storsimple-remote-connect/HCS_GetRemoteManagementCertificate.png)
6. 複製憑證中從 **-----BEGIN CERTIFICATE-----** 到 **-----END CERTIFICATE-----** 的資訊到文字編輯器，如 [記事本]，然後將它儲存為 .cer 檔案。 (在您準備主機時，要將這個檔案複製到您的遠端主機)。
   
   > [!NOTE]
   > 若要產生新的憑證，使用 `Set-HcsRemoteManagementCert` Cmdlet。
   
### <a name="prepare-the-host-for-remote-management"></a>準備遠端管理的主機

若要準備使用 HTTPS 工作階段進行遠端連線的主機電腦，請執行下列程序：

* [將 .cer 檔案匯入至用戶端或遠端主機的根存放區](#to-import-the-certificate-on-the-remote-host)。
* [將裝置序號新增至遠端主機上的主機檔案](#to-add-device-serial-numbers-to-the-remote-host)。

以下說明上述各程序。

#### <a name="to-import-the-certificate-on-the-remote-host"></a>匯入遠端主機上的憑證
1. 以滑鼠右鍵按一下.cer 檔案，選取 [ **安裝憑證**]。 這會啟動 [憑證匯入精靈]。
   
    ![憑證匯入精靈 1](./media/storsimple-remote-connect/HCS_CertificateImportWizard1.png)
2. [存放區位置] 請選取 [本機電腦]，然後按一下 [下一步]。
3. 選取 [將所有憑證放入以下的存放區]，然後按一下 [瀏覽]。 導覽至遠端主機的根存放區，然後按一下 [ **下一步**]。
   
    ![憑證匯入精靈  2](./media/storsimple-remote-connect/HCS_CertificateImportWizard2.png)
4. 按一下 [完成] 。 會出現訊息告訴您匯入成功。
   
    ![憑證匯入精靈  3](./media/storsimple-remote-connect/HCS_CertificateImportWizard3.png)

#### <a name="to-add-device-serial-numbers-to-the-remote-host"></a>將裝置序號新增至遠端主機
1. 以管理員的身分啟動 [記事本]，然後開啟位於 \Windows\System32\Drivers\etc 的主機檔案。
2. 將下列三個項目新增至主機檔案：**DATA 0 IP 位址**、**控制器 0 固定 IP 位址**、**控制器 1 固定 IP 位址**。
3. 輸入您稍早儲存的裝置序號。 對應至 IP 位址，如下圖所示。 對於控制器 0 及控制器 1，在序號 (CN 名稱) 結尾後附加 **Controller0** 和 **Controller1**。
   
    ![將 CN 名稱加入至主機檔案](./media/storsimple-remote-connect/HCS_AddingCNNameToHostsFile.png)
4. 儲存主機檔案。

### <a name="connect-to-the-device-from-the-remote-host"></a>從遠端主機連線至裝置

使用 Windows PowerShell 和 SSL從遠端主機或用戶端進入您的裝置上的 SSAdmin 工作階段。 SSAdmin 工作階段會對應至裝置的[序列主控台](storsimple-8000-windows-powershell-administration.md#connect-to-windows-powershell-for-storsimple-via-the-device-serial-console)功能表中的選項 1。

從您要建立遠端 Windows PowerShell 連線的電腦上執行下列程序。

#### <a name="to-enter-an-ssadmin-session-on-the-device-by-using-windows-powershell-and-ssl"></a>使用 Windows PowerShell 和 SSL進入 SSAdmin 工作階段
1. 以系統管理員的身分開啟 Windows PowerShell工作階段。 如果使用 Windows 10 用戶端，Windows 遠端管理服務預設會設定為手動。 您可能需要輸入下列命令以啟動服務：

    `Start-Service WinRM`

2. 將裝置的 IP 位址新增至用戶端信任的主機，做法是輸入：
   
     `Set-Item wsman:\localhost\Client\TrustedHosts <device_ip> -Concatenate -Force`
   
    其中 <device_ip> 是您的裝置的 IP 位址，例如： 
   
     `Set-Item wsman:\localhost\Client\TrustedHosts 10.126.173.90 -Concatenate -Force`
3. 若要建立新認證，請輸入：
   
     `$cred = New-Object pscredential @("<IP of target device>\SSAdmin", (ConvertTo-SecureString -Force -AsPlainText "<Device Administrator Password>"))`
   
    其中 <目標裝置的 IP> 是您的裝置的 DATA 0 的 IP 位址，例如之前影像中的主機檔案的 **10.126.173.90**。 此外，提供您的裝置的管理員密碼。
4. 建立工作階段，做法是輸入：
   
     `$session = New-PSSession -UseSSL -ComputerName <Serial number of target device> -Credential $cred -ConfigurationName "SSAdminConsole"`
   
    請為 Cmdlet 中的 -ComputerName 參數提供 <目標裝置的序號>。 此序號已對應至您的遠端主機上主機檔案中 DATA 0 的 IP 位址，如下圖所示的 **SHX0991003G44MT** 。
5. 輸入：
   
     `Enter-PSSession $session`
6. 您必須等候幾分鐘，接著便會透過 SSL 經由 HTTPS 連線到您的裝置。 您會看到訊息，指出您已連線到裝置。
   
    ![使用 HTTPS 和 SSL 進行 PowerShell 遠端處理](./media/storsimple-remote-connect/HCS_PSRemotingUsingHTTPSAndSSL.png)

## <a name="next-steps"></a>後續步驟

* 深入了解 [使用 Windows PowerShell 來管理您的 StorSimple 裝置](storsimple-8000-windows-powershell-administration.md)。
* 深入了解[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

