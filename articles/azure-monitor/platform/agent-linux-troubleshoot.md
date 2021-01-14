---
title: 針對 Azure Log Analytics Linux 代理程式 進行疑難排解 | Microsoft Docs
description: 描述 Azure 監視器中適用于 Linux 的 Log Analytics 代理程式最常見問題的徵兆、原因和解決方式。
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/21/2019
ms.openlocfilehash: 26fb70592a75910ae21d327e53569eda12dfea97
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98197365"
---
# <a name="how-to-troubleshoot-issues-with-the-log-analytics-agent-for-linux"></a>如何針對 Log Analytics Linux 代理程式的問題進行疑難排解 

本文提供疑難排解 Log Analytics Linux 代理程式在 Azure 監視器中可能遇到的錯誤，並建議解決這些問題的可能解決方案。

如果這些步驟對您都沒有幫助，還有下列支援管道可供使用：

* 具有頂級支援權益的客戶可以透過[頂級支援](https://premier.microsoft.com/)開啟支援要求。
* 具有 Azure 支援合約的客戶可以在 [Azure 入口網站](https://manage.windowsazure.com/?getsupport=true)開啟支援要求。
* 使用 [OMI 疑難排解指南](https://github.com/Microsoft/omi/blob/master/Unix/doc/diagnose-omi-problems.md)來診斷 OMI 問題。
* 提出 [GitHub 問題](https://github.com/Microsoft/OMS-Agent-for-Linux/issues)。
* 造訪 Log Analytics 意見反應頁面，以審查提交的想法和 bug， [https://aka.ms/opinsightsfeedback](https://aka.ms/opinsightsfeedback) 或提出新的 bug。 

## <a name="log-analytics-troubleshooting-tool"></a>Log Analytics 疑難排解工具

Log Analytics 代理程式 Linux 疑難排解工具是設計用來協助找出並診斷 Log Analytics 代理程式問題的腳本。 它會在安裝時自動隨附于代理程式。 執行工具應該是診斷問題的第一個步驟。

### <a name="how-to-use"></a>如何使用
您可以使用 Log Analytics 代理程式，將下列命令貼到電腦上的終端機視窗中，以執行疑難排解工具： `sudo /opt/microsoft/omsagent/bin/troubleshooter`

### <a name="manual-installation"></a>手動安裝
安裝 Log Analytics 代理程式時，會自動包含疑難排解工具。 但是，如果安裝是以任何方式失敗，也可以遵循下列步驟手動安裝。

1. 將疑難排解員配套複製到您的電腦： `wget https://raw.github.com/microsoft/OMS-Agent-for-Linux/master/source/code/troubleshooter/omsagent_tst.tar.gz`
2. 解壓縮套件組合： `tar -xzvf omsagent_tst.tar.gz`
3. 執行手動安裝： `sudo ./install_tst`

### <a name="scenarios-covered"></a>涵蓋的案例
以下是疑難排解工具所檢查的案例清單：

1. 代理程式狀況不良，心跳無法正常運作
2. 代理程式無法啟動，無法連接到記錄分析服務
3. 代理程式 syslog 無法運作
4. 代理程式具有高 CPU/記憶體使用量
5. 代理程式發生安裝問題
6. 代理程式自訂記錄無法運作
7. 收集代理程式記錄檔

如需詳細資料，請參閱我們的 [Github 檔](https://github.com/microsoft/OMS-Agent-for-Linux/blob/master/docs/Troubleshooting-Tool.md)。

 >[!NOTE]
 >當您遇到問題時，請執行記錄檔收集器工具。 一開始就能提供記錄，讓我們的支援小組能更快地針對您的問題進行疑難排解。

## <a name="purge-and-re-install-the-linux-agent"></a>清除並 Re-Install Linux 代理程式

我們已瞭解代理程式的全新重新安裝將會修正大部分的問題。 事實上，這可能是第一次支援的建議，可讓代理程式從我們的支援小組進入 uncurropted 狀態。 執行疑難排解員、記錄檔收集和嘗試全新重新安裝，將有助於更快速地解決問題。

1. 下載清除腳本：
- `$ wget https://raw.githubusercontent.com/microsoft/OMS-Agent-for-Linux/master/tools/purge_omsagent.sh`
2. 以 sudo 許可權執行 [清除腳本 (]) ：
- `$ sudo sh purge_omsagent.sh`

## <a name="important-log-locations-and-log-collector-tool"></a>重要記錄位置和記錄收集器工具

 檔案 | Path
 ---- | -----
 Log Analytics Linux 代理程式記錄檔 | `/var/opt/microsoft/omsagent/<workspace id>/log/omsagent.log`
 Log Analytics 代理程式組態記錄檔 | `/var/opt/microsoft/omsconfig/omsconfig.log`

 建議您使用記錄收集器工具來擷取重要記錄，以供進行疑難排解，或作為提交 GitHub 問題之前的準備工作。 您可以[在此](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/tools/LogCollector/OMS_Linux_Agent_Log_Collector.md)深入了解此工具及其執行方式。

## <a name="important-configuration-files"></a>重要組態檔案

 類別 | 檔案位置
 ----- | -----
 syslog | `/etc/syslog-ng/syslog-ng.conf`、`/etc/rsyslog.conf` 或 `/etc/rsyslog.d/95-omsagent.conf`
 效能、Nagios、Zabbix、Log Analytics 輸出和一般代理程式 | `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.conf`
 其他組態 | `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.d/*.conf`

 >[!NOTE]
 >如果您是從 Azure 入口網站中的[資料功能表 Log Analytics [進階設定]](./agent-data-sources.md#configuring-data-sources) 為工作區設定了集合，則為效能計數器和 Syslog 編輯組態檔會遭到覆寫。 若要停用所有代理程式的組態，請從 Log Analytics [進階設定] 停用集合，若是單一代理程式，則請執行下列命令：  
> `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/OMS_MetaConfigHelper.py --disable'`

## <a name="installation-error-codes"></a>安裝錯誤碼

| 錯誤碼 | 意義 |
| --- | --- |
| NOT_DEFINED | 未安裝必要的相依性，所以不會安裝 auoms auditd 外掛程式 | auoms 安裝失敗，安裝套件 auditd。 |
| 2 | 提供給殼層組合的選項無效。 請執行 `sudo sh ./omsagent-*.universal*.sh --help` 以了解使用方式 |
| 3 | 未提供任何選項給殼層組合。 請執行 `sudo sh ./omsagent-*.universal*.sh --help` 以了解使用方式。 |
| 4 | 套件類型無效或 Proxy 設定無效；omsagent-rpm.sh 套件只能安裝在以 RPM 為基礎的系統上，至於 omsagent-deb.sh 套件，則只能安裝在以 Debian 為基礎的系統上。 建議您使用來自[最新版本](../learn/quick-collect-linux-computer.md#install-the-agent-for-linux)的通用安裝程式。 也請進行檢閱，以驗證 Proxy 設定。 |
| 5 | 必須以 root 身分執行殼層組合，否則上架期間會傳回 403 錯誤。 請使用 `sudo` 執行命令。 |
| 6 | 套件架構無效，或是在上架期間傳回錯誤 200 錯誤；omsagent-x64.sh 套件只能安裝在 64 位元系統上，而 omsagent-x86.sh 套件則只能安裝在 32 位元系統上。 從[最新版本](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/latest)下載架構的正確套件。 |
| 17 | OMS 套件安裝失敗。 查看命令的輸出中是否有 root 失敗。 |
| 18 | OMSConfig 套件安裝失敗。 查看命令的輸出中是否有 root 失敗。 |
| 19 | OMI 套件安裝失敗。 查看命令的輸出中是否有 root 失敗。 |
| 20 | SCX 套件安裝失敗。 查看命令的輸出中是否有 root 失敗。 |
| 21 | 提供者套件安裝失敗。 查看命令的輸出中是否有 root 失敗。 |
| 22 | 組合套件安裝失敗。 查看命令的輸出中是否有 root 失敗 |
| 23 | 已經安裝 SCX 或 OMI 套件。 使用 `--upgrade` 而非 `--install` 來安裝殼層組合。 |
| 30 | 內部組合錯誤。 提出 [GitHub 問題](https://github.com/Microsoft/OMS-Agent-for-Linux/issues)，並附上輸出中的詳細資料。 |
| 55 | 不支援的 openssl 版本或無法連線至 Azure 監視器或 dpkg 已鎖定或遺失的捲曲程式。 |
| 61 | 遺失 Python ctypes 程式庫。 安裝 Python ctypes 程式庫或套件 (python-ctypes)。 |
| 62 | 遺失 tar 程式，請安裝 tar。 |
| 63 | 遺失 sed 程式，請安裝 sed。 |
| 64 | 遺失 curl 程式，請安裝 curl。 |
| 65 | 遺失 gpg 程式，請安裝 gpg。 |

## <a name="onboarding-error-codes"></a>上架錯誤碼

| 錯誤碼 | 意義 |
| --- | --- |
| 2 | 提供給 omsadmin 指令碼的選項無效。 請執行 `sudo sh /opt/microsoft/omsagent/bin/omsadmin.sh -h` 以了解使用方式。 |
| 3 | 提供給 omsadmin 指令碼的組態無效。 請執行 `sudo sh /opt/microsoft/omsagent/bin/omsadmin.sh -h` 以了解使用方式。 |
| 4 | 提供給 omsadmin 指令碼的 Proxy 無效。 請確認 Proxy，並查看 [HTTP Proxy 的使用文件](log-analytics-agent.md#firewall-requirements)。 |
| 5 | 從 Azure 監視器接收到 403 HTTP 錯誤。 如需詳細資訊，請參閱 omsadmin 指令碼的完整輸出。 |
| 6 | 從 Azure 監視器收到非200的 HTTP 錯誤。 如需詳細資訊，請參閱 omsadmin 指令碼的完整輸出。 |
| 7 | 無法連接到 Azure 監視器。 如需詳細資訊，請參閱 omsadmin 指令碼的完整輸出。 |
| 8 | 上架至 Log Analytics 工作區時發生錯誤。 如需詳細資訊，請參閱 omsadmin 指令碼的完整輸出。 |
| 30 | 內部指令碼錯誤。 提出 [GitHub 問題](https://github.com/Microsoft/OMS-Agent-for-Linux/issues)，並附上輸出中的詳細資料。 |
| 31 | 產生代理程式識別碼時發生錯誤。 提出 [GitHub 問題](https://github.com/Microsoft/OMS-Agent-for-Linux/issues)，並附上輸出中的詳細資料。 |
| 32 | 產生憑證時發生錯誤。 如需詳細資訊，請參閱 omsadmin 指令碼的完整輸出。 |
| 33 | 產生 omsconfig 的中繼組態時發生錯誤。 提出 [GitHub 問題](https://github.com/Microsoft/OMS-Agent-for-Linux/issues)，並附上輸出中的詳細資料。 |
| 34 | 中繼組態產生指令碼不存在。 請使用 `sudo sh /opt/microsoft/omsagent/bin/omsadmin.sh -w <Workspace ID> -s <Workspace Key>` 重試上架。 |

## <a name="enable-debug-logging"></a>啟用偵錯記錄
### <a name="oms-output-plugin-debug"></a>OMS 輸出外掛程式偵錯
 FluentD 允許使用外掛程式特定記錄層級，可讓您為輸入和輸出指定不同的記錄層級。 若要為 OMS 輸出指定不同的記錄層級，請在 `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.conf` 編輯一般代理程式組態。  

 在 OMS 輸出外掛程式的組態檔結尾前面，將 `log_level` 屬性從 `info` 變更為 `debug`：

 ```
 <match oms.** docker.**>
  type out_oms
  log_level debug
  num_threads 5
  buffer_chunk_limit 5m
  buffer_type file
  buffer_path /var/opt/microsoft/omsagent/<workspace id>/state/out_oms*.buffer
  buffer_queue_limit 10
  flush_interval 20s
  retry_limit 10
  retry_wait 30s
</match>
 ```

「偵錯工具記錄」可讓您查看批次上傳至 Azure 監視器依類型、資料項目數目和傳送所花費的時間來分隔：

*啟用偵錯的記錄檔範例︰*

```
Success sending oms.nagios x 1 in 0.14s
Success sending oms.omi x 4 in 0.52s
Success sending oms.syslog.authpriv.info x 1 in 0.91s
```

### <a name="verbose-output"></a>詳細資訊輸出
除了使用 OMS 輸出外掛程式，您也可以將資料項目直接輸出至 `stdout`，其顯示在 Log Analytics Linux 代理程式記錄檔中。

在位於 `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.conf` 的 Log Analytics 一般代理程式組態檔中，於每一行前面新增 `#`，以將 OMS 輸出註解化：

```
#<match oms.** docker.**>
#  type out_oms
#  log_level info
#  num_threads 5
#  buffer_chunk_limit 5m
#  buffer_type file
#  buffer_path /var/opt/microsoft/omsagent/<workspace id>/state/out_oms*.buffer
#  buffer_queue_limit 10
#  flush_interval 20s
#  retry_limit 10
#  retry_wait 30s
#</match>
```

在輸出外掛程式下面，移除每一行開頭的 `#`，以將下列區段取消註解：

```
<match **>
  type stdout
</match>
```

## <a name="issue--unable-to-connect-through-proxy-to-azure-monitor"></a>問題：無法透過 proxy 連接到 Azure 監視器

### <a name="probable-causes"></a>可能的原因
* 上架期間指定的 Proxy 不正確
* Azure 監視器和 Azure 自動化的服務端點不會包含在資料中心的核准清單中 

### <a name="resolution"></a>解決方案
1. Azure 監視器重新上架使用適用于 Linux 的 Log Analytics 代理程式，方法是在啟用選項的情況下使用下列命令 `-v` 。 它允許透過 proxy 連線到 Azure 監視器的代理程式詳細資訊輸出。 
`/opt/microsoft/omsagent/bin/omsadmin.sh -w <Workspace ID> -s <Workspace Key> -p <Proxy Conf> -v`

2. 檢閱[更新 Proxy 設定](agent-manage.md#update-proxy-settings)一節以驗證您是否已正確設定代理程式透過 Proxy 伺服器通訊。    

3. 再次檢查 [Azure 監視器 [網路防火牆需求](log-analytics-agent.md#firewall-requirements) ] 清單中所述的端點是否已正確新增至允許清單。 如果您使用 Azure 自動化，必要的網路設定步驟也會連結在上方。

## <a name="issue-you-receive-a-403-error-when-trying-to-onboard"></a>問題：您在嘗試上架時收到 403 錯誤

### <a name="probable-causes"></a>可能的原因
* Linux 伺服器上的日期與時間不正確 
* 使用的工作區識別碼和工作區金鑰不正確

### <a name="resolution"></a>解決方案

1. 使用命令日期檢查 Linux 伺服器上的時間。 如果時間為自目前時間起的 + /-15 分鐘，則上架失敗。 若要修正此問題，請更新 Linux 伺服器的日期和/或時區。 
2. 確認您已安裝最新版的 Log Analytics Linux 代理程式。  最新版本現在會通知您時間差異是否造成上架失敗。
3. 使用正確的工作區識別碼和工作區金鑰並遵循本文前面的安裝指示重新上架。

## <a name="issue-you-see-a-500-and-404-error-in-the-log-file-right-after-onboarding"></a>問題︰上架後您隨即在記錄檔中看到 500 與 404 錯誤
這是已知第一次將 Linux 資料上傳至 Log Analytics 工作區時會發生的問題。 這不會影響正在傳送的資料或服務體驗。


## <a name="issue-you-see-omiagent-using-100-cpu"></a>問題：您看到使用 100% CPU 的 >omi >omiagent

### <a name="probable-causes"></a>可能的原因
Nss 套件 [v 1.0.3-5](https://centos.pkgs.org/7/centos-x86_64/nss-pem-1.0.3-7.el7.x86_64.rpm.html) 中的回歸會造成嚴重的效能問題，我們看到的是 Redhat/Centos 7.x 散發套件中的許多問題。 若要深入瞭解此問題，請參閱下列檔： [libcurl 中的 Bug 1667121 效能回歸](https://bugzilla.redhat.com/show_bug.cgi?id=1667121)。

效能相關的錯誤不會在任何時間發生，而且難以重現。 如果您遇到這類 >omi >omiagent 的問題，您應該使用腳本 omiHighCPUDiagnostics.sh，這會在超過特定臨界值時，收集 >omi >omiagent 的堆疊追蹤。

1. 下載腳本 <br/>
`wget https://raw.githubusercontent.com/microsoft/OMS-Agent-for-Linux/master/tools/LogCollector/source/omiHighCPUDiagnostics.sh`

2. 以 30% CPU 閾值執行診斷24小時 <br/>
`bash omiHighCPUDiagnostics.sh --runtime-in-min 1440 --cpu-threshold 30`

3. 呼叫堆疊會在 omiagent_trace 檔案中傾印，如果您發現許多捲曲和 NSS 函式呼叫，請遵循下列解決步驟。

### <a name="resolution-step-by-step"></a> (逐步執行) 的解決方案

1. 將 nss-pem 套件升級為 [v 1.0.3-5.el7_6. 1](https://centos.pkgs.org/7/centos-x86_64/nss-pem-1.0.3-7.el7.x86_64.rpm.html)。 <br/>
`sudo yum upgrade nss-pem`

2. 如果無法進行 (升級，則會在 Centos) 上執行，然後將其降級為 7.29.0-46。 如果您不小心執行 "yum update"，則會將捲曲升級到 7.29.0-51，問題就會再次發生。 <br/>
`sudo yum downgrade curl libcurl`

3. 重新開機 OMI： <br/>
`sudo scxadmin -restart`


## <a name="issue-you-are-not-seeing-forwarded-syslog-messages"></a>問題：沒看到轉送的 Syslog 訊息 

### <a name="probable-causes"></a>可能的原因
* 套用到 Linux 伺服器的組態不允許收集傳送的設備和 (或) 記錄檔層級
* Syslog 並未正確地轉送到 Linux 伺服器
* 每秒所轉送的訊息數目太大，以致無法處理 Log Analytics Linux 代理程式的基本組態

### <a name="resolution"></a>解決方案
* 確認 Log Analytics 工作區 (適用於 Syslog) 中的組態具有所有設備和正確的記錄檔層級。 檢閱[在 Azure 入口網站中設定 Syslog 集合](./data-sources-syslog.md#configure-syslog-in-the-azure-portal)
* 確認原生 syslog 傳訊精靈 (`rsyslog`、`syslog-ng`) 能夠接收轉送的訊息
* 檢查 Syslog 伺服器上的防火牆設定，確定不會封鎖訊息
* 使用 `logger` 命令模擬給 Log Analytics 的 Syslog 訊息
  * `logger -p local0.err "This is my test message"`

## <a name="issue-you-are-receiving-errno-address-already-in-use-in-omsagent-log-file"></a>問題：您在 omsagent 記錄檔中收到「位址已在使用中」的錯誤
如果您在 omsagent.log 中看到 `[error]: unexpected error error_class=Errno::EADDRINUSE error=#<Errno::EADDRINUSE: Address already in use - bind(2) for "127.0.0.1" port 25224>`。

### <a name="probable-causes"></a>可能的原因
此錯誤指出 Linux 診斷擴充功能 (LAD) 已隨 Log Analytics Linux VM 擴充功能一起安裝，且它會使用和 omsagent 一樣的連接埠來收集 syslog 資料。

### <a name="resolution"></a>解決方案
1. 以 root 身分執行下列命令 (請注意，25224 只是範例，您在環境中可能會看到 LAD 使用不同的連接埠號碼)：

    ```
    /opt/microsoft/omsagent/bin/configure_syslog.sh configure LAD 25229

    sed -i -e 's/25224/25229/' /etc/opt/microsoft/omsagent/LAD/conf/omsagent.d/syslog.conf
    ```

    然後，您需要編輯正確的 `rsyslogd` 或 `syslog_ng` 組態檔，並變更 LAD 相關組態以寫入至連接埠 25229。

2. 如果 VM 執行 `rsyslogd`，所要修改的檔案則是：`/etc/rsyslog.d/95-omsagent.conf` (若有存在，否則是 `/etc/rsyslog`)。 如果 VM 執行 `syslog_ng`，所要修改的檔案則是：`/etc/syslog-ng/syslog-ng.conf`。
3. 重新啟動 omsagent `sudo /opt/microsoft/omsagent/bin/service_control restart`。
4. 重新啟動 syslog 服務。

## <a name="issue-you-are-unable-to-uninstall-omsagent-using-purge-option"></a>問題：您無法使用清除選項解除安裝 omsagent

### <a name="probable-causes"></a>可能的原因

* 已安裝 Linux 診斷擴充功能
* 安裝了 Linux 診斷擴充功能後又解除安裝，但您仍看到 omsagent 正由 mdsd 使用所以無法移除的錯誤。

### <a name="resolution"></a>解決方案
1. 解除安裝 Linux 診斷擴充功能 (LAD)。
2. 從機器中移除出現在下列位置的 Linux 診斷擴充功能檔案：`/var/lib/waagent/Microsoft.Azure.Diagnostics.LinuxDiagnostic-<version>/` 和 `/var/opt/microsoft/omsagent/LAD/`。

## <a name="issue-you-cannot-see-data-any-nagios-data"></a>問題：您無法看到任何 Nagios 資料 

### <a name="probable-causes"></a>可能的原因
* omsagent 使用者沒有從 Nagios 記錄檔讀取資料的權限
* 未在 omsagent.conf 檔案中將 Nagios 來源和篩選取消註解

### <a name="resolution"></a>解決方案
1. 遵循這些[指示](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#nagios-alerts)，新增 omsagent 使用者以讀取 Nagios 檔案。
2. 在位於 `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.conf` 的 Log Analytics Linux 代理程式一般組態檔中，確定 Nagios 來源和篩選 **都** 已取消註解。

    ```
    <source>
      type tail
      path /var/log/nagios/nagios.log
      format none
      tag oms.nagios
    </source>

    <filter oms.nagios>
      type filter_nagios_log
    </filter>
    ```

## <a name="issue-you-are-not-seeing-any-linux-data"></a>問題︰您沒有看到任何 Linux 資料 

### <a name="probable-causes"></a>可能的原因
* 上架至 Azure 監視器失敗
* 已封鎖與 Azure 監視器的連接
* 虛擬機器已重新啟動
* OMI 套件已手動升級為比 Log Analytics Linux 代理程式套件所安裝版本還新的版本
* OMI 已凍結，正在封鎖 OMS 代理程式
* DSC 資源在 `omsconfig.log` 記錄中記錄了「找不到類別」錯誤
* Log Analytics 的資料代理程式已備份
* DSC 記錄 *目前的設定不存在。執行 Start-DscConfiguration 命令搭配-Path 參數來指定設定檔，並先建立目前的設定。* 在 `omsconfig.log` 記錄檔中，但沒有關於 `PerformRequiredConfigurationChecks` 作業的記錄訊息存在。

### <a name="resolution"></a>解決方案
1. 安裝所有相依性，例如 auditd 套件。
2. 檢查下列檔案是否存在，以確認上架至 Azure 監視器是否成功： `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsadmin.conf` 。  如果不成功，請使用 omsadmin.sh 命令列[指示](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#onboarding-using-the-command-line)重新上架。
4. 如果使用 Proxy，請查看上述的 Proxy 疑難排解步驟。
5. 在某些 Azure 發佈系統中，omid OMI 伺服器精靈未在虛擬機器重新啟動後隨之啟動。 這會導致您看不到 Audit、ChangeTracking 或 UpdateManagement 解決方案相關資料。 因應措施是執行 `sudo /opt/omi/bin/service_control restart` 來手動啟動 omi 伺服器。
6. OMI 套件手動升級為較新版本後，必須手動加以重新啟動，Log Analytics 代理程式才能繼續運作。 在 OMI 伺服器未於升級後自動啟動的某些散發套件中，此為必要步驟。 請執行 `sudo /opt/omi/bin/service_control restart` 來重新啟動 OMI。
* 在某些情況下，OMI 可能會被凍結。 OMS 代理程式可能會進入封鎖狀態，等候 OMI，封鎖所有資料收集。 OMS 代理程式進程將會執行，但不會有任何新的記錄行所證明的活動， (例如) 存在的傳送的心跳 `omsagent.log` 。 重新開機 OMI， `sudo /opt/omi/bin/service_control restart` 以復原代理程式。
7. 如果您在 omsconfig.log 中看到 DSC 資源「找不到類別」錯誤，請執行 `sudo /opt/omi/bin/service_control restart`。
8. 在某些情況下，當 Log Analytics Linux 代理程式無法與 Azure 監視器交談時，代理程式上的資料會備份到完整的緩衝區大小： 50 MB。 應該執行下列命令重新啟動代理程式：`/opt/microsoft/omsagent/bin/service_control restart`。

    >[!NOTE]
    >此問題已在代理程式 1.1.0-28 版和更新版本中修正
    >

* 如果 `omsconfig.log` 記錄檔未指出 `PerformRequiredConfigurationChecks` 作業會在系統上定期執行，則表示 cron 作業/服務可能有問題。 請確定 cron 作業存在於 `/etc/cron.d/OMSConsistencyInvoker` 底下。 如有需要，請執行下列命令來建立 cron 作業：

    ```
    mkdir -p /etc/cron.d/
    echo "*/15 * * * * omsagent /opt/omi/bin/OMSConsistencyInvoker >/dev/null 2>&1" | sudo tee /etc/cron.d/OMSConsistencyInvoker
    ```

    此外，請確定 cron 服務正在執行。 您可以對 Debian、Ubuntu、SUSE 使用 `service cron status` 或對 RHEL、CentOS、Oracle Linux 使用 `service crond status`，來檢查此服務的狀態。 如果服務不存在，您可以使用下列命令來安裝二進位檔並啟動服務：

    **Ubuntu/Debian**

    ```
    # To Install the service binaries
    sudo apt-get install -y cron
    # To start the service
    sudo service cron start
    ```

    **SUSE**

    ```
    # To Install the service binaries
    sudo zypper in cron -y
    # To start the service
    sudo systemctl enable cron
    sudo systemctl start cron
    ```

    **RHEL/CeonOS**

    ```
    # To Install the service binaries
    sudo yum install -y crond
    # To start the service
    sudo service crond start
    ```

    **Oracle Linux**

    ```
    # To Install the service binaries
    sudo yum install -y cronie
    # To start the service
    sudo service crond start
    ```

## <a name="issue-when-configuring-collection-from-the-portal-for-syslog-or-linux-performance-counters-the-settings-are-not-applied"></a>問題：從入口網站設定 Syslog 或 Linux 效能計數器的收集功能時，系統不會套用設定

### <a name="probable-causes"></a>可能的原因
* Log Analytics Linux 代理程式未挑選最新的組態
* 未套用入口網站中經過變更的設定

### <a name="resolution"></a>解決方案
**背景：** `omsconfig` 是 Log Analytics Linux 代理程式組態代理程式，會每隔五分鐘尋找一次新的入口網站端組態。 此組態接著會套用到位於 /etc/opt/microsoft/omsagent/conf/omsagent.conf 的 Log Analytics Linux 代理程式組態檔。

* 在某些情況下，Log Analytics Linux 代理程式組態代理程式可能無法與入口網站組態服務進行通訊，以致未套用最新的組態。
  1. 執行 `dpkg --list omsconfig` 或 `rpm -qi omsconfig` 來確認 `omsconfig` 代理程式已安裝。  若未安裝，請重新安裝最新版的 Log Analytics Linux 代理程式。

  2. 藉由執行 `omsconfig` 下列命令，確認代理程式可以與 Azure 監視器通訊 `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/GetDscConfiguration.py'` 。 此命令會傳回代理程式從服務接收的組態，包括 Syslog 設定、Linux 效能計數器以及自訂記錄。 如果此命令失敗，請執行下列命令 `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/PerformRequiredConfigurationChecks.py'`。 此命令會強制 omsconfig 代理程式與 Azure 監視器，並取得最新的設定。

## <a name="issue-you-are-not-seeing-any-custom-log-data"></a>問題︰您沒有看到任何自訂記錄資料 

### <a name="probable-causes"></a>可能的原因
* 上架到 Azure 監視器失敗。
* 尚未選取 [將下列組態套用至我的 Linux 伺服器] 設定。
* omsconfig 尚未從服務挑選最新的自訂記錄組態。
* Log Analytics Linux 代理程式的使用者 `omsagent` 無法存取自訂記錄，因為沒有權限或找不到記錄。  您可能會看到下列錯誤：
 * `[DATETIME] [warn]: file not found. Continuing without tailing it.`
 * `[DATETIME] [error]: file not accessible by omsagent.`
* 已在 Log Analytics Linux 代理程式 1.1.0-217 版中修正的已知競爭條件問題

### <a name="resolution"></a>解決方案
1. 檢查下列檔案是否存在，確認上架至 Azure 監視器成功： `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsadmin.conf` 。 如果不成功，則：  

  1. 使用 omsadmin.sh 命令列[指示](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#onboarding-using-the-command-line)重新上架。
  2. 在 Azure 入口網站的 [進階設定] 之下，確定已啟用 [將下列組態套用至我的 Linux 伺服器] 設定。  

2. 藉由執行 `omsconfig` 下列命令，確認代理程式可以與 Azure 監視器通訊 `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/GetDscConfiguration.py'` 。  此命令會傳回代理程式從服務接收的組態，包括 Syslog 設定、Linux 效能計數器以及自訂記錄。 如果此命令失敗，請執行下列命令 `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/PerformRequiredConfigurationChecks.py'`。 此命令會強制 omsconfig 代理程式與 Azure 監視器，並取得最新的設定。

**背景：** Log Analytics Linux 代理程式使用者不會以特殊權限使用者 `root` 身分執行，而會以 `omsagent` 使用者身分執行。 在大部分情況下，必須將明確的權限授與給這位使用者，才能讀取特定檔案。 若要授與權限給 `omsagent` 使用者，請執行下列命令︰

1. 將 `omsagent` 使用者新增至特定群組 `sudo usermod -a -G <GROUPNAME> <USERNAME>`
2. 授與必要檔案的通用讀取權限 `sudo chmod -R ugo+rx <FILE DIRECTORY>`

1.1.0-217 版之前的 Log Analytics Linux 代理程式已知有競爭條件問題。 更新為最新的代理程式後，請執行下列命令，以取得最新版的輸出外掛程式 `sudo cp /etc/opt/microsoft/omsagent/sysconf/omsagent.conf /etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.conf`。

## <a name="issue-you-are-trying-to-reonboard-to-a-new-workspace"></a>問題：您嘗試重新上架至新的工作區
在嘗試將代理程式重新上架至新的工作區時，必須先將 Log Analytics 代理程式組態清除，才能重新上架。 若要從代理程式清除舊組態，請使用 `--purge` 執行殼層組合

```
sudo sh ./omsagent-*.universal.x64.sh --purge
```
Or

```
sudo sh ./onboard_agent.sh --purge
```

在使用 `--purge` 選項之後，便可繼續重新上架

## <a name="log-analytics-agent-extension-in-the-azure-portal-is-marked-with-a-failed-state-provisioning-failed"></a>Azure 入口網站中的 Log Analytics 代理程式擴充功能標示了失敗狀態：佈建失敗

### <a name="probable-causes"></a>可能的原因
* Log Analytics 代理程式已從作業系統中移除
* Log Analytics 代理程式服務已關閉、停用或未設定

### <a name="resolution"></a>解決方案 
請執行下列步驟來解決此問題。
1. 從 Azure 入口網站移除擴充功能。
2. 遵循[指示](../learn/quick-collect-linux-computer.md)來安裝代理程式。
3. 執行下列命令來重新啟動代理程式：`sudo /opt/microsoft/omsagent/bin/service_control restart`。
* 等候幾分鐘的時間，然後佈建狀態就會變為 **佈建成功**。


## <a name="issue-the-log-analytics-agent-upgrade-on-demand"></a>問題：Log Analytics 代理程式隨選升級

### <a name="probable-causes"></a>可能的原因

主機上的 Log Analytics 代理程式套件已過時。

### <a name="resolution"></a>解決方案 
請執行下列步驟來解決此問題。

1. 請至[網頁](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/)查看最新版。
2. 下載安裝指令碼 (1.4.2-124 為範例版本)：

    ```
    wget https://github.com/Microsoft/OMS-Agent-for-Linux/releases/download/OMSAgent_GA_v1.4.2-124/omsagent-1.4.2-124.universal.x64.sh
    ```

3. 執行 `sudo sh ./omsagent-*.universal.x64.sh --upgrade` 來升級套件。
