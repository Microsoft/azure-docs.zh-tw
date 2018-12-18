---
title: 使用 System Center Configuration Manager 自動進行適用於 Azure Site Recovery 的行動服務安裝 | Microsoft Docs
description: 本文協助您使用 System Center Configuration Manager 自動進行行動服務安裝。
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: ramamill
ms.openlocfilehash: 6aff17908d220557720776a2d7884adc28b5deee
ms.sourcegitcommit: baed5a8884cb998138787a6ecfff46de07b8473d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "37917164"
---
# <a name="automate-mobility-service-installation-with-system-center-configuration-manager"></a>使用 System Center Configuration Manager 自動進行行動服務安裝

行動服務會安裝在您想要使用 [Azure Site Recovery](site-recovery-overview.md) 複寫到 Azure 的 VMware VM 和實體伺服器上

本文提供範例說明如何使用 System Center Configuration Manager，在 VMware VM 上部署 Azure Site Recovery 行動服務。 使用 Configuration Manager 之類的軟體部署工具有下列優點：

* 在您規劃的軟體更新維護期間，排程全新安裝和升級
* 同時大規模部署至數百部伺服器

本文使用 System Center Configuration Manager 2012 R2 示範部署活動。 我們假設您使用版本 **9.9.4510.1** 或更高版本的行動服務。

或者，您可以使用 [Azure Automation DSC](vmware-azure-mobility-deploy-automation-dsc.md) 自動進行行動服務安裝。

## <a name="prerequisites"></a>必要條件

1. 環境中已部署的軟體部署工具，例如 Configuration Manager。
2. 您應該建立兩個[裝置集合](https://technet.microsoft.com/library/gg682169.aspx)，一個用於所有 **Windows 伺服器**，另一個用於所有 **Linux 伺服器** (都是您想要使用 Site Recovery 保護的伺服器)。
3. 已在復原服務保存庫中註冊的組態伺服器。
4. 組態管理員機器可存取的安全網路檔案共用 (SMB 共用)。

## <a name="deploy-on-windows-machines"></a>在 Windows 機器上部署
> [!NOTE]
> 本文假設設定伺服器的 IP 位址為 192.168.3.121，且安全網路檔案共用是 \\\ContosoSecureFS\MobilityServiceInstallers。

### <a name="prepare-for-deployment"></a>準備部署
1. 在網路共用上建立資料夾，並命名為 **MobSvcWindows**。
2. 登入設定伺服器，然後將系統管理命令提示字元開啟。
3. 執行下列命令來產生複雜密碼檔案：

    `cd %ProgramData%\ASR\home\svsystems\bin`

    `genpassphrase.exe -v > MobSvc.passphrase`
4. 將 **MobSvc.passphrase** 檔案複製到網路共用上的 **MobSvcWindows** 資料夾。
5. 執行下列命令，以瀏覽至設定伺服器上的安裝程式存放庫：

   `cd %ProgramData%\ASR\home\svsystems\pushinstallsvc\repository`

6. 將 **Microsoft-ASR\_UA\_*version*\_Windows\_GA\_*date*\_Release.exe** 複製到網路共用上的 **MobSvcWindows** 資料夾。
7. 複製下列程式碼，儲存為 **install.bat** 放在 **MobSvcWindows** 資料夾中。

   > [!NOTE]
   > 將此指令碼中的 [CSIP] 預留位置取代為設定伺服器 IP 位址的實際值。

```DOS
Time /t >> C:\Temp\logfile.log
REM ==================================================
REM ==== Clean up the folders ========================
RMDIR /S /q %temp%\MobSvc
MKDIR %Temp%\MobSvc
MKDIR C:\Temp
REM ==================================================

REM ==== Copy new files ==============================
COPY M*.* %Temp%\MobSvc
CD %Temp%\MobSvc
REN Micro*.exe MobSvcInstaller.exe
REM ==================================================

REM ==== Extract the installer =======================
MobSvcInstaller.exe /q /x:%Temp%\MobSvc\Extracted
REM ==== Wait 10s for extraction to complete =========
TIMEOUT /t 10
REM =================================================

REM ==== Perform installation =======================
REM =================================================

CD %Temp%\MobSvc\Extracted
whoami >> C:\Temp\logfile.log
SET PRODKEY=HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
REG QUERY %PRODKEY%\{275197FC-14FD-4560-A5EB-38217F80CBD1}
IF NOT %ERRORLEVEL% EQU 0 (
    echo "Product is not installed. Goto INSTALL." >> C:\Temp\logfile.log
    GOTO :INSTALL
) ELSE (
    echo "Product is installed." >> C:\Temp\logfile.log

    echo "Checking for Post-install action status." >> C:\Temp\logfile.log
    GOTO :POSTINSTALLCHECK
)

:POSTINSTALLCHECK
    REG QUERY "HKLM\SOFTWARE\Wow6432Node\InMage Systems\Installed Products\5" /v "PostInstallActions" | Find "Succeeded"
    If %ERRORLEVEL% EQU 0 (
        echo "Post-install actions succeeded. Checking for Configuration status." >> C:\Temp\logfile.log
        GOTO :CONFIGURATIONCHECK
    ) ELSE (
        echo "Post-install actions didn't succeed. Goto INSTALL." >> C:\Temp\logfile.log
        GOTO :INSTALL
    )

:CONFIGURATIONCHECK
    REG QUERY "HKLM\SOFTWARE\Wow6432Node\InMage Systems\Installed Products\5" /v "AgentConfigurationStatus" | Find "Succeeded"
    If %ERRORLEVEL% EQU 0 (
        echo "Configuration has succeeded. Goto UPGRADE." >> C:\Temp\logfile.log
        GOTO :UPGRADE
    ) ELSE (
        echo "Configuration didn't succeed. Goto CONFIGURE." >> C:\Temp\logfile.log
        GOTO :CONFIGURE
    )


:INSTALL
    echo "Perform installation." >> C:\Temp\logfile.log
    UnifiedAgent.exe /Role MS /InstallLocation "C:\Program Files (x86)\Microsoft Azure Site Recovery" /Platform "VmWare" /Silent
    IF %ERRORLEVEL% EQU 0 (
        echo "Installation has succeeded." >> C:\Temp\logfile.log
        (GOTO :CONFIGURE)
    ) ELSE (
        echo "Installation has failed." >> C:\Temp\logfile.log
        GOTO :ENDSCRIPT
    )

:CONFIGURE
    echo "Perform configuration." >> C:\Temp\logfile.log
    cd "C:\Program Files (x86)\Microsoft Azure Site Recovery\agent"
    UnifiedAgentConfigurator.exe  /CSEndPoint "[CSIP]" /PassphraseFilePath %Temp%\MobSvc\MobSvc.passphrase
    IF %ERRORLEVEL% EQU 0 (
        echo "Configuration has succeeded." >> C:\Temp\logfile.log
    ) ELSE (
        echo "Configuration has failed." >> C:\Temp\logfile.log
    )
    GOTO :ENDSCRIPT

:UPGRADE
    echo "Perform upgrade." >> C:\Temp\logfile.log
    UnifiedAgent.exe /Platform "VmWare" /Silent
    IF %ERRORLEVEL% EQU 0 (
        echo "Upgrade has succeeded." >> C:\Temp\logfile.log
    ) ELSE (
        echo "Upgrade has failed." >> C:\Temp\logfile.log
    )
    GOTO :ENDSCRIPT

:ENDSCRIPT
    echo "End of script." >> C:\Temp\logfile.log


```

### <a name="create-a-package"></a>建立套件

1. 登入您的 Configuration Manager 主控台。
2. 瀏覽至 [軟體程式庫] > [應用程式管理] > [套件]。
3. 以滑鼠右鍵按一下 [套件]，然後選取 [建立套件]。
4. 提供 [名稱]、[描述]、[製造商]、[語言] 和 [版本] 的值。
5. 選取 [此套件包含來源檔案] 核取方塊。
6. 按一下 [瀏覽]，選取儲存安裝程式的網路共用 (\\\ContosoSecureFS\MobilityServiceInstaller\MobSvcWindows)。

  ![建立套件和程式精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/create_sccm_package.png)

7. 在 [選擇您要建立的程式類型] 頁面上，選取 [標準程式]，然後按 [下一步]。

  ![建立套件和程式精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm-standard-program.png)

8. 在 [指定此標準程式的相關資訊] 頁面上，提供下列輸入，然後按 [下一步]。 (其他輸入可以使用其預設值。)

  | **參數名稱** | **值** |
  |--|--|
  | 名稱 | 安裝 Microsoft Azure 行動服務 (Windows) |
  | 命令列 | install.bat |
  | 程式可以執行 | 使用者是否登入 |

  ![建立套件和程式精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm-program-properties.png)

9. 在下一頁，選取目標作業系統。 行動服務可以安裝在 Windows Server 2012 R2、Windows Server 2012 和 Windows Server 2008 R2。

  ![建立套件和程式精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm-program-properties-page2.png)

10. 按兩次 [下一步] 以完成精靈。


> [!NOTE]
> 指令碼支援全新安裝行動服務代理程式，以及升級至已安裝的代理程式。

### <a name="deploy-the-package"></a>部署套件
1. 在 Configuration Manager 主控台，以滑鼠右鍵按一下套件，然後選取 [發佈內容]。
  ![Configuration Manager 主控台的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm_distribute.png)
2. 選取應該將套件複製過去的**[發佈點](https://technet.microsoft.com/library/gg712321.aspx#BKMK_PlanForDistributionPoints)**。
3. 完成精靈。 套件便會開始複寫至指定的發佈點。
4. 完成套件發佈後，以滑鼠右鍵按一下套件，然後選取 [部署]。
  ![Configuration Manager 主控台的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm_deploy.png)
5. 選取您在必要條件一節所建立的 Windows Server 裝置集合，作為部署的目標集合。

  ![部署軟體精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm-select-target-collection.png)

6. 在 [指定內容目的地] 頁面上，選取您的**發佈點**。
7. 在 [指定控制此軟體部署方式的設定] 頁面上，確定目的為**必要**。

  ![部署軟體精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm-deploy-select-purpose.png)

8. 在 [指定此部署的排程] 頁面上指定排程。 如需詳細資訊，請參閱 [排程套件](https://technet.microsoft.com/library/gg682178.aspx)。
9. 根據資料中心的需求，在 [發佈點] 頁面上設定屬性。 然後完成精靈。

> [!TIP]
> 為了避免不必要的重新開機，請排定在每月維護期間或軟體更新期間安裝套件。

您可以使用 Configuration Manager 主控台來監視部署進度。 移至 [監視] > [部署] > *[套件名稱]*。

  ![監視部署的 Configuration Manager 選項螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/report.PNG)

## <a name="deploy-on-linux-machines"></a>在 Linux 機器上部署
> [!NOTE]
> 本文假設設定伺服器的 IP 位址為 192.168.3.121，且安全網路檔案共用是 \\\ContosoSecureFS\MobilityServiceInstallers。

### <a name="prepare-for-deployment"></a>準備部署
1. 在網路共用上建立資料夾，並命名為 **MobSvcLinux**。
2. 登入設定伺服器，然後將系統管理命令提示字元開啟。
3. 執行下列命令來產生複雜密碼檔案：

    `cd %ProgramData%\ASR\home\svsystems\bin`

    `genpassphrase.exe -v > MobSvc.passphrase`
4. 將 **MobSvc.passphrase** 檔案複製到網路共用上的 **MobSvcLinux** 資料夾。
5. 執行命令以瀏覽至設定伺服器上的安裝程式存放庫：

   `cd %ProgramData%\ASR\home\svsystems\pushinstallsvc\repository`

6. 將下列檔案複製到網路共用上的 **MobSvcLinux** 資料夾：
   * Microsoft-ASR\_UA\*RHEL6-64*release.tar.gz
   * Microsoft-ASR\_UA\*RHEL7-64\*release.tar.gz
   * Microsoft-ASR\_UA\*SLES11-SP3-64\*release.tar.gz
   * Microsoft-ASR\_UA\*SLES11-SP4-64\*release.tar.gz
   * Microsoft-ASR\_UA\*OL6-64\*release.tar.gz
   * Microsoft-ASR\_UA\*UBUNTU-14.04-64\*release.tar.gz


7. 複製下列程式碼，儲存為 **install_linux.sh** 放在 **MobSvcLinux** 資料夾中。
   > [!NOTE]
   > 將此指令碼中的 [CSIP] 預留位置取代為設定伺服器 IP 位址的實際值。

```Bash
#!/usr/bin/env bash

rm -rf /tmp/MobSvc
mkdir -p /tmp/MobSvc
INSTALL_DIR='/usr/local/ASR'
VX_VERSION_FILE='/usr/local/.vx_version'

echo "=============================" >> /tmp/MobSvc/sccm.log
echo `date` >> /tmp/MobSvc/sccm.log
echo "=============================" >> /tmp/MobSvc/sccm.log

if [ -f /etc/oracle-release ] && [ -f /etc/redhat-release ]; then
    if grep -q 'Oracle Linux Server release 6.*' /etc/oracle-release; then
        if uname -a | grep -q x86_64; then
            OS="OL6-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *OL6*.tar.gz /tmp/MobSvc
        fi
    fi
elif [ -f /etc/redhat-release ]; then
    if grep -q 'Red Hat Enterprise Linux Server release 6.* (Santiago)' /etc/redhat-release || \
        grep -q 'CentOS Linux release 6.* (Final)' /etc/redhat-release || \
        grep -q 'CentOS release 6.* (Final)' /etc/redhat-release; then
        if uname -a | grep -q x86_64; then
            OS="RHEL6-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *RHEL6*.tar.gz /tmp/MobSvc
        fi
    elif grep -q 'Red Hat Enterprise Linux Server release 7.* (Maipo)' /etc/redhat-release || \
        grep -q 'CentOS Linux release 7.* (Core)' /etc/redhat-release; then
        if uname -a | grep -q x86_64; then
            OS="RHEL7-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *RHEL7*.tar.gz /tmp/MobSvc
                fi
    fi
elif [ -f /etc/SuSE-release ] && grep -q 'VERSION = 11' /etc/SuSE-release; then
    if grep -q "SUSE Linux Enterprise Server 11" /etc/SuSE-release && grep -q 'PATCHLEVEL = 3' /etc/SuSE-release; then
        if uname -a | grep -q x86_64; then
            OS="SLES11-SP3-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *SLES11-SP3*.tar.gz /tmp/MobSvc
        fi
    elif grep -q "SUSE Linux Enterprise Server 11" /etc/SuSE-release && grep -q 'PATCHLEVEL = 4' /etc/SuSE-release; then
        if uname -a | grep -q x86_64; then
            OS="SLES11-SP4-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *SLES11-SP4*.tar.gz /tmp/MobSvc
        fi
    fi
elif [ -f /etc/lsb-release ] ; then
    if grep -q 'DISTRIB_RELEASE=14.04' /etc/lsb-release ; then
       if uname -a | grep -q x86_64; then
           OS="UBUNTU-14.04-64"
           echo $OS >> /tmp/MobSvc/sccm.log
           cp *UBUNTU-14*.tar.gz /tmp/MobSvc
       fi
    fi
else
    exit 1
fi

if [ -z "$OS" ]; then
    exit 1
fi

Install()
{
    echo "Perform Installation." >> /tmp/MobSvc/sccm.log
    ./install -q -d ${INSTALL_DIR} -r MS -v VmWare
    RET_VAL=$?
    echo "Installation Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Installation has succeeded. Proceed to configuration." >> /tmp/MobSvc/sccm.log
        Configure
    else
        echo "Installation has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

Configure()
{
    echo "Perform configuration." >> /tmp/MobSvc/sccm.log
    ${INSTALL_DIR}/Vx/bin/UnifiedAgentConfigurator.sh -i [CSIP] -P MobSvc.passphrase
    RET_VAL=$?
    echo "Configuration Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Configuration has succeeded." >> /tmp/MobSvc/sccm.log
    else
        echo "Configuration has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

Upgrade()
{
    echo "Perform Upgrade." >> /tmp/MobSvc/sccm.log
    ./install -q -v VmWare
    RET_VAL=$?
    echo "Upgrade Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Upgrade has succeeded." >> /tmp/MobSvc/sccm.log
    else
        echo "Upgrade has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

cp MobSvc.passphrase /tmp/MobSvc
cd /tmp/MobSvc

tar -zxvf *.tar.gz

if [ -e ${VX_VERSION_FILE} ]; then
    echo "${VX_VERSION_FILE} exists. Checking for configuration status." >> /tmp/MobSvc/sccm.log
    agent_configuration=$(grep ^AGENT_CONFIGURATION_STATUS "${VX_VERSION_FILE}" | cut -d"=" -f2 | tr -d " ")
    echo "agent_configuration=$agent_configuration" >> /tmp/MobSvc/sccm.log
     if [ "$agent_configuration" == "Succeeded" ]; then
        echo "Agent is already configured. Proceed to Upgrade." >> /tmp/MobSvc/sccm.log
        Upgrade
    else
        echo "Agent is not configured. Proceed to Configure." >> /tmp/MobSvc/sccm.log
        Configure
    fi
else
    Install
fi

cd /tmp

```

### <a name="create-a-package"></a>建立套件

1. 登入您的 Configuration Manager 主控台。
2. 瀏覽至 [軟體程式庫] > [應用程式管理] > [套件]。
3. 以滑鼠右鍵按一下 [套件]，然後選取 [建立套件]。
4. 提供 [名稱]、[描述]、[製造商]、[語言] 和 [版本] 的值。
5. 選取 [此套件包含來源檔案] 核取方塊。
6. 按一下 [瀏覽]，選取儲存安裝程式的網路共用 (\\\ContosoSecureFS\MobilityServiceInstaller\MobSvcLinux)。

  ![建立套件和程式精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/create_sccm_package-linux.png)

7. 在 [選擇您要建立的程式類型] 頁面上，選取 [標準程式]，然後按 [下一步]。

  ![建立套件和程式精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm-standard-program.png)

8. 在 [指定此標準程式的相關資訊] 頁面上，提供下列輸入，然後按 [下一步]。 (其他輸入可以使用其預設值。)

    | **參數名稱** | **值** |
  |--|--|
  | 名稱 | 安裝 Microsoft Azure 行動服務 (Linux) |
  | 命令列 | ./install_linux.sh |
  | 程式可以執行 | 使用者是否登入 |

  ![建立套件和程式精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm-program-properties-linux.png)

9. 在下一個頁面上，選取 [可在任何平台上執行這個程式]。
  ![建立套件和程式精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm-program-properties-page2-linux.png)

10. 按兩次 [下一步] 以完成精靈。

> [!NOTE]
> 指令碼支援全新安裝行動服務代理程式，以及升級至已安裝的代理程式。

### <a name="deploy-the-package"></a>部署套件
1. 在 Configuration Manager 主控台，以滑鼠右鍵按一下套件，然後選取 [發佈內容]。
  ![Configuration Manager 主控台的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm_distribute.png)
2. 選取應該將套件複製過去的**[發佈點](https://technet.microsoft.com/library/gg712321.aspx#BKMK_PlanForDistributionPoints)**。
3. 完成精靈。 套件便會開始複寫至指定的發佈點。
4. 完成套件發佈後，以滑鼠右鍵按一下套件，然後選取 [部署]。
  ![Configuration Manager 主控台的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm_deploy.png)
5. 選取您在必要條件一節所建立的 Linux 伺服器裝置集合，作為部署的目標集合。

  ![部署軟體精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm-select-target-collection-linux.png)

6. 在 [指定內容目的地] 頁面上，選取您的**發佈點**。
7. 在 [指定控制此軟體部署方式的設定] 頁面上，確定目的為**必要**。

  ![部署軟體精靈的螢幕擷取畫面](./media/vmware-azure-mobility-install-configuration-mgr/sccm-deploy-select-purpose.png)

8. 在 [指定此部署的排程] 頁面上指定排程。 如需詳細資訊，請參閱 [排程套件](https://technet.microsoft.com/library/gg682178.aspx)。
9. 根據資料中心的需求，在 [發佈點] 頁面上設定屬性。 然後完成精靈。

行動服務會根據您設定的排程，安裝在 Linux 伺服器裝置集合上。


## <a name="uninstall-the-mobility-service"></a>將行動服務解除安裝
您可以建立 Configuration Manager 套件，將行動服務解除安裝。 若要這樣做，請使用下列指令碼︰

```
Time /t >> C:\logfile.log
REM ==================================================
REM ==== Check if Mob Svc is already installed =======
REM ==== If not installed no operation required ========
REM ==== Else run uninstall command =====================
REM ==== {275197FC-14FD-4560-A5EB-38217F80CBD1} is ====
REM ==== guid for Mob Svc Installer ====================
whoami >> C:\logfile.log
NET START | FIND "InMage Scout Application Service"
IF  %ERRORLEVEL% EQU 1 (GOTO :INSTALL) ELSE GOTO :UNINSTALL
:NOOPERATION
                echo "No Operation Required." >> c:\logfile.log
                GOTO :ENDSCRIPT
:UNINSTALL
                echo "Uninstall" >> C:\logfile.log
                MsiExec.exe /qn /x {275197FC-14FD-4560-A5EB-38217F80CBD1} /L+*V "C:\ProgramData\ASRSetupLogs\UnifiedAgentMSIUninstall.log"
:ENDSCRIPT

```

## <a name="next-steps"></a>後續步驟
您現在可以對虛擬機器[啟用保護](vmware-azure-enable-replication.md)。
