---
title: 已知問題和疑難排解指南 | Microsoft Docs
description: 已知問題清單和協助疑難排解的指南
services: machine-learning
author: hning86
ms.author: haining
manager: mwinkle
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 01/12/2018
ms.openlocfilehash: 8177808fd4d666ea04b1bc097f261c7643931704
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48885034"
---
# <a name="azure-machine-learning-workbench---known-issues-and-troubleshooting-guide"></a>Azure Machine Learning Workbench - 已知問題和疑難排解指南 
本文可協助您尋找和修正使用 Azure Machine Learning Workbench 應用程式過程中遇到的錯誤與失敗。 

## <a name="find-the-workbench-build-number"></a>尋找 Workbench 組建編號
與技術支援小組通訊時，務必提供 Workbench 應用程式的組建編號。 在 Windows 上按一下 [說明] 功能表並選擇 [關於 Azure ML Workbench]，就可以找到組建編號。 在 macOS 上則可以按一下 [Azure ML Workbench] 功能表，然後選擇 [關於 Azure ML Workbench]。

## <a name="machine-learning-msdn-forum"></a>機器學習服務 MSDN 論壇
您可以在我們的 MSDN 論壇張貼問題。 產品小組會主動監視論壇。 論壇 URL 是 [https://aka.ms/aml-forum-service](https://aka.ms/aml-forum-service)。 

## <a name="gather-diagnostics-information"></a>蒐集診斷資訊
當您在尋求協助時，如果能夠提供診斷資訊，有時可能會相當有幫助。 以下是記錄檔的所在位置：

### <a name="installer-log"></a>安裝程式記錄檔
如果您在安裝期間遇到問題，可以在以下位置找到安裝程式記錄檔：

```
# Windows:
%TEMP%\amlinstaller\logs\*

# macOS:
/tmp/amlinstaller/logs/*
```
您可以將這些目錄的內容壓縮，然後傳送給我們進行診斷。

### <a name="workbench-desktop-app-log"></a>Workbench 傳統型應用程式記錄檔
如果您無法登入，或如果 Workbench 傳統型應用程式當機，就可以在以下位置找到記錄檔：
```
# Windows
%APPDATA%\AmlWorkbench

# macOS
~/Library/Application Support/AmlWorkbench
``` 
您可以將這些目錄的內容壓縮，然後傳送給我們進行診斷。

### <a name="experiment-execution-log"></a>實驗執行記錄檔
如果從傳統型應用程式提交時，特定指令碼發生失敗，請嘗試透過 CLI 使用 `az ml experiment submit` 命令來重新提交。 這應該會以 JSON 格式提供您完整的錯誤訊息，最重要的是，它會包含**作業識別碼**值。 請將包含**作業識別碼**的 JSON 檔案傳送給我們，我們將可協助進行診斷。 

如果特定指令碼在提交時成功，但在執行時失敗，它應該會顯示**執行識別碼**來識別該特定執行。 您可以使用下列命令來封裝相關的記錄檔︰

```azurecli
# Create a ZIP file that contains all the diagnostics information
$ az ml experiment diagnostics -r <run_id> -t <target_name>
```

`az ml experiment diagnostics` 命令會在專案根資料夾中產生 `diagnostics.zip` 檔案。 此 ZIP 套件會包含整個專案資料夾 (處於執行時的狀態)，再加上記錄資訊。 將診斷檔案傳送給我們之前，請務必先移除您不想要包含的任何敏感性資訊。

## <a name="send-us-a-frown-or-a-smile"></a>將皺眉 (或笑臉) 傳送給我們

當您使用 Azure ML Workbench 時，您也可以按一下應用程式 shell 左下角的笑臉圖示，將皺眉 (或笑臉) 傳送給我們。 您可以選擇性選擇要包含您的電子郵件地址 (因此我們可以回覆您)，及/或目前狀態的螢幕擷取畫面。 

## <a name="known-service-limits"></a>已知的服務限制
- 允許的專案資料夾大小上限：25 MB。
    >[!NOTE]
    >這項限制不適用於 `.git`、`docs` 和 `outputs` 資料夾。 這些資料夾名稱會區分大小寫。 如果您使用大型檔案，請參閱[保存變更和處理大型檔案](how-to-read-write-files.md)。

- 允許的實驗執行時間上限：7 天

- 執行之後 `outputs` 資料夾中追蹤檔案的大小上限：512 MB
  - 這表示如果指令碼在 outputs 資料夾中產生大於 512 MB 的檔案，就不會在那裡收集該檔案。 如果您使用大型檔案，請參閱[保存變更和處理大型檔案](how-to-read-write-files.md)。

- 透過 SSH 連線至遠端電腦或 Spark 叢集時，不支援 SSH 金鑰。 目前只支援使用者名稱/密碼模式。

- 使用 HDInsight 叢集作為計算目標時，必須使用 Azure Blob 作為主要儲存體。 不支援使用 Azure Data Lake 儲存體。

- Mac 不支援文字叢集轉換。

- 只有在 Windows 和 Linux (在 Docker 容器中) 上支援 RevoScalePy 程式庫。 在 macOS 上不提供支援。

- 從 Workbench 應用程式開啟時，Jupyter Notebook 的大小上限為 5 MB。 您可以使用 'az ml notebook start' 命令以從 CLI 開啟大型筆記本，再清理資料格輸出以縮減檔案大小。

## <a name="cant-update-workbench"></a>無法更新 Workbench
有新的更新可用時，Workbench 應用程式首頁會顯示一則訊息，通知您有關新的更新。 您應會看到更新徽章出現在鈴鐺圖示上應用程式的左下角。 按一下徽章並遵循安裝程式精靈來安裝更新。 

![更新影像](../service/media/known-issues-and-troubleshooting-guide/update.png)

如果您沒有看到通知，請嘗試重新啟動應用程式。 如果在重新啟動之後，您仍然沒看到更新通知，則可能會有幾個原因。

### <a name="you-are-launching-workbench-from-a-pinned-shortcut-on-the-task-bar"></a>您是從工作列上釘選的捷徑啟動 Workbench
您可能已經安裝此更新。 但是，您釘選的捷徑仍然指向磁碟上舊的位元。 您可以瀏覽至 `%localappdata%/AmlWorkbench` 資料夾進行確認，查看那裡是否已安裝最新版本，然後檢查釘選捷徑的屬性，查看它指向何處。 確認之後，只要移除舊的捷徑，從 [開始] 功能表啟動 Workbench，然後選擇性地在工作列上建立新的釘選捷徑。

### <a name="you-installed-workbench-using-the-install-azure-ml-workbench-link-on-a-windows-dsvm"></a>您已使用 Windows DSVM 上的「安裝 Azure ML Workbench」連結來安裝 Workbench
不幸的是，這種情況沒有輕鬆的修正方式。 您必須執行下列步驟來移除已安裝的位元，並下載最新的安裝程式來全新安裝 Workbench： 
   - 移除資料夾 `C:\Users\<Username>\AppData\Local\amlworkbench`
   - 移除指令碼 `C:\dsvm\tools\setup\InstallAMLFromLocal.ps1`
   - 移除可啟動上述指令碼的桌面捷徑
   - 下載安裝程式 https://aka.ms/azureml-wb-msi 並且重新安裝。

## <a name="stuck-at-checking-experimentation-account-screen-after-logging-in"></a>在登入後卡在「正在檢查測試帳戶」畫面
登入之後，Workbench 應用程式可能會卡在空白畫面，並會顯示「正在檢查測試帳戶」訊息及旋轉的滾輪。 若要解決此問題，請採取下列步驟：
1. 關閉應用程式
2. 刪除以下檔案：
  ```
  # on Windows
  %appdata%\AmlWorkbench\AmlWb.settings

  # on macOS
  ~/Library/Application Support/AmlWorkbench/AmlWb.settings
  ```
3. 重新啟動應用程式。

## <a name="cant-delete-experimentation-account"></a>無法刪除測試帳戶
您可以使用 CLI 來刪除測試帳戶，但是必須先刪除子工作區和子工作區內的子專案。 否則，「必須先刪除巢狀資源，才能刪除資源」錯誤將會出現。

```azure-cli
# delete a project
$ az ml project delete -g <resource group name> -a <experimentation account name> -w <workspace name> -n <project name>

# delete a workspace 
$ az ml workspace delete -g <resource group name> -a <experimentation account name> -n <workspace name>

# delete an experimentation account
$ az ml account experimentation delete -g <resource group name> -n <experimentation account name>
```

您也可以從 Workbench 應用程式內刪除專案和工作區。

## <a name="cant-open-file-if-project-is-in-onedrive"></a>如果專案位於 OneDrive 中，則無法開啟檔案
如果您有 Windows 10 Fall Creators Update，而且您的專案建立於 OneDrive 對應的本機資料夾，您可能會發現您無法在 Workbench 中開啟任何檔案。 這是因為 Fall Creators Update 所引進的 Bug 造成 node.js 程式碼在 OneDrive 資料夾中發生失敗。 Windows Update 很快就會修正此 Bug，請不要在 OneDrive 資料夾中建立專案。

## <a name="file-name-too-long-on-windows"></a>檔案名稱在 Windows 上太長
如果您在 Windows 上使用 Workbench，您可能會遇到預設最大 260 個字元的檔案名稱長度限制，這可能呈現為「系統找不到指定的路徑」錯誤。 您可以修改登錄機碼設定，以允許更長的檔案路徑名稱。 檢閱[本文](https://msdn.microsoft.com/library/windows/desktop/aa365247%28v=vs.85%29.aspx?#maxpath)，以取得有關如何設定 _MAX_PATH_ 登錄機碼的詳細資訊。

## <a name="interrupt-cli-execution-output"></a>插斷 CLI 執行輸出
如果您使用 `az ml experiment submit` 或 `az ml notebook start` 來啟動測試執行回合，但想要插斷輸出： 
- 在 Windows 上，請使用鍵盤上的 Ctrl-Break 按鍵組合
- 在 macOS 上，請使用 Ctrl-C。

請注意，這個做法只會插斷 CLI 視窗中的輸出串流， 並不會實際停止執行中的作業。 如果您想要取消進行中的作業，請使用 `az ml experiment cancel -r <run_id> -t <target name>` 命令。

如果您使用鍵盤沒有 Break 鍵的 Windows 電腦，可能的替代按鍵組合包括 Fn-B、Ctrl-Fn-B 或 Fn+Esc。如需特定按鍵組合的相關資訊，請參閱硬體廠商的文件。

## <a name="docker-error-read-connection-refused"></a>Docker 錯誤 "read: connection refused"
對本機 Docker 容器執行時，有時您可能會看到下列錯誤： 
```
Get https://registry-1.docker.io/v2/: 
dial tcp: 
lookup registry-1.docker.io on [::1]:53: read udp [::1]:49385->[::1]:53: 
read: connection refused
```

將 Docker DNS Server 從 `automatic` 變更為固定值 `8.8.8.8`，即可修正此錯誤。

## <a name="remove-vm-execution-error-no-tty-present"></a>移除 VM 執行錯誤 "no tty present"
對遠端 Linux 機器上的本機 Docker 容器執行時，您可能會看到下列錯誤訊息：
```
sudo: no tty present and no askpass program specified.
``` 
如果您使用 Azure 入口網站來變更 Ubuntu Linux VM 的根密碼，可能會發生這種情形。 

Azure Machine Learning Workbench 需要無密碼 sudoers 存取權，才能在遠端主機上執行。 若要這麼做，最簡單的方式是使用 _visudo_ 來編輯下列檔案 (如果此檔案不存在，您可加以建立)：

```
$ sudo visudo -f /etc/sudoers
```

>[!IMPORTANT]
>務必使用 _visudo_ (而非另一個命令) 編輯此檔案。 _visudo_ 語法會自動檢查所有 sudo 組態檔，而無法產生正確語法的 sudoers 檔案可能讓您無法存取 sudo。

在檔案結尾處插入下列這一行︰

```
username ALL=(ALL) NOPASSWD:ALL
```

其中 _username_ 是 Azure Machine Learning Workbench 將用來登入遠端主機的名稱。

這一行必須放在 #includedir"/ etc/sudoers.d" 之後，否則它可能會被另一個規則所覆寫。

如果您有更複雜的 sudo 組態，建議您查閱 Ubuntu 的 sudo 文件，網址如下： https://help.ubuntu.com/community/Sudoers

如果您未在 Azure 中使用以 Ubuntu 為基礎的 Linux VM 作為執行目標，也會發生上述錯誤。 我們只支援以 Ubuntu 為基礎的 Linux VM 進行遠端執行。 

## <a name="vm-disk-is-full"></a>VM 磁碟已滿
依預設，當您在 Azure 中建立新的 Linux VM 時，您可取得 30 GB 磁碟以供作業系統使用。 Docker 引擎預設會使用相同的磁碟來提取映像和執行容器。 這可能會填滿作業系統磁碟，而您會在發生此狀況時看到「VM 磁碟已滿」錯誤。

快速修正方法是移除您不再使用的所有 Docker 映像。 下列 Docker 命令可執行該作業。 (當然您必須透過 SSH 連至 VM，以便從 bash shell 執行 Docker 命令。)

```
$ docker system prune -a
```

您也可以新增資料磁碟並將 Docker 引擎設定為使用資料磁碟來儲存映像。 以下說明[如何新增資料磁碟](https://docs.microsoft.com/azure/virtual-machines/linux/add-disk)。 您可以接著[變更 Docker 儲存映像的位置](https://forums.docker.com/t/how-do-i-change-the-docker-image-installation-directory/1169)。

或者，您可以展開作業系統磁碟，但不必碰觸 Docker 引擎組態。 以下說明[如何展開作業系統磁碟](https://docs.microsoft.com/azure/virtual-machines/linux/expand-disks)。

```azure-cli
# Deallocate VM (stopping will not work)
$ az vm deallocate --resource-group myResourceGroup  --name myVM

# Get VM's Disc Name
az disk list --resource-group myResourceGroup --query '[*].{Name:name,Gb:diskSizeGb,Tier:accountType}' --output table

# Update Disc Size using above name
$ az disk update --resource-group myResourceGroup --name myVMdisc --size-gb 250
    
# Start VM    
$ az vm start --resource-group myResourceGroup  --name myVM
```

## <a name="sharing-c-drive-on-windows"></a>在 Windows 上共用 C 磁碟機
如果您在 Windows 上的本機 Docker 容器中執行，將 `aml_config` 底下 `docker.compute` 檔案中的 `sharedVolumes` 設定為 `true` 可以改善執行效能。 不過，您必須在 _Docker for Windows 工具_ 中共用 C 磁碟機。 如果您不能共用 C 磁碟機，請嘗試下列秘訣：

* 使用 [檔案總管] 來檢查 C 磁碟機上的共用
* 開啟網路介面卡設定，並為 vEthernet 解除安裝/重新安裝 [File and Printer Sharing for Microsoft Networks]
* 開啟 Docker 設定並在 Docker 設定內共用 C 磁碟機
* 變更 Windows 密碼會影響共用。 開啟 [檔案總管]、重新共用 C 磁碟機，然後輸入新密碼。
* 使用 Docker 嘗試共用您的 C 磁碟機時，可能也會遇到防火牆問題。 這篇 [Stack Overflow 文章](http://stackoverflow.com/questions/42203488/settings-to-windows-firewall-to-allow-docker-for-windows-to-share-drive/43904051) \(英文\) 提供有用的資訊。
* 使用網域認證共用 C 磁碟機時，若所在網路無法連線網域控制站 (例如，家用網路、公用 WiFi 等)，則共用可能會停止運作。 如需詳細資訊，請參閱[這篇文章](https://blogs.msdn.microsoft.com/stevelasker/2016/06/14/configuring-docker-for-windows-volumes/) \(英文\)。

您也可以在 `docker.compute` 檔案中將 `sharedVolumne` 設定為`false`，以很少的效能成本避開共用問題。

## <a name="wipe-clean-workbench-installation"></a>徹底抹除 Workbench 安裝
您通常不需要執行這項操作。 但如果您必須徹底抹除安裝，步驟如下：

- 在 Windows 上：
  - 首先，請務必使用 [控制台] 的 [新增或移除程式] 小程式移除 Azure Machine Learning Workbench 應用程式項目。  
  - 接下來，您可以下載及執行以下任一個指令碼：
    - [Windows 命令列指令碼](https://github.com/Azure/MachineLearning-Scripts/blob/master/cleanup/cleanup_win.cmd)。
    - [Windows PowerShell 指令碼](https://github.com/Azure/MachineLearning-Scripts/blob/master/cleanup/cleanup_win.ps1)。 (您可能需要先在提高權限的 PowerShell 視窗中執行 `Set-ExecutionPolicy Unrestricted`，之後才能執行指令碼。)
- 在 macOS 上：
  - 只要下載及執行 [macOS Bash 殼層指令碼](https://github.com/Azure/MachineLearning-Scripts/blob/master/cleanup/cleanup_mac.sh)即可。

## <a name="azure-ml-using-a-different-python-location-than-the-azure-ml-installed-python-environment"></a>使用與已安裝 Azure ML 的 Python 環境不同 Python 位置的 Azure ML
由於 Azure Machine Learning Workbench 中最近的變更，所以使用者可能注意到本機執行可能不再指向 Azure ML Workbench 所安裝的 Python 環境。 如果使用者在其電腦上安裝了另一個 Python 環境，並將 "Python" 路徑設定為指向該環境，就可能發生此情況。 若要使用已安裝 Azure ML Workbench 的 Python 環境，請遵循下列步驟：
- 移至專案根目錄底下 aml_config 資料夾中的 local.compute 檔案。
- 變更 "pythonLocation" 變數，以指向已安裝 Azure ML Workbench 之 Python 環境的實體路徑。 有兩種方式可以取得此路徑：
    - 您可以在 %localappdata%\AmlWorkbench\python\python.exe 中找到 Azure ML Python 位置
    - 您可以從 Azure ML Workbench 開啟 cmd、在命令提示字元上輸入 python、匯入 sys.exe、執行 sys.executable，然後從該處取得該路徑。 



## <a name="some-useful-docker-commands"></a>一些實用的 Docker 命令

以下是一些實用的 Docker 命令：

```sh
# display all running containers
$ docker ps

# dislplay all containers (running or stopped)
$ docke ps -a

# display all images
$ docker images

# show Docker logs of a container
$ docker logs <container_id>

# create a new container and launch into a bash shell
$ docker run <image_id> /bin/bash

# launch into a bash shell on a running container
$ docker exec -it <container_id> /bin/bash

# stop an running container
$ docker stop <container_id>

# delete a container
$ docker rm <container_id>

# delete an image
$ docker rmi <image_id>

# delete all unussed Docker images 
$ docker system prune -a

```
