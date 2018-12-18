---
title: Azure Machine Learning 新增功能
description: 本文件會詳細說明 Azure Machine Learning 的更新內容。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: reference
author: hning86
ms.author: haining
ms.date: 03/28/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 08be059cb30c8a7ec4ad24fc4f73f4b569883483
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970612"
---
# <a name="release-notes-in-azure-machine-learning-sept-2017---jun-2018"></a>Azure Machine Learning 版本資訊 2017 年 9 月 - 2018 年 6 月

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 

在此文章中，了解 Azure Machine Learning 過去推出的版本。 


## <a name="2018-05-sprint-5"></a>2018-05 (Sprint 5)

透過此版本的 Azure Machine Learning，您可以：
+ 利用量化版本的 ResNet 50 將影像功能化、根據這些功能為分類器定型，並[將該模型部署到 Azure 上的 FPGA](../service/how-to-deploy-fpga-web-service.md)，以達到超低延遲推斷。

+ 針對下列網域使用[自訂 Azure Machine Learning 套件](../desktop-workbench/reference-python-package-overview.md)快速建置並部署高度精確的機器學習和深入學習模型：
  + [電腦視覺](../desktop-workbench/how-to-build-deploy-image-classification-models.md)
  + [文字分析](../desktop-workbench/how-to-build-deploy-text-classification-models.md)
  + [預測](../desktop-workbench/how-to-build-deploy-forecast-models.md)

## <a name="2018-03-sprint-4"></a>2018-03 (Sprint 4)
**版本號碼**：0.1.1801.24353  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;([尋找您的版本](../desktop-workbench/known-issues-and-troubleshooting-guide.md#find-the-workbench-build-number))


以下的更新有許多是直接來自您的意見反應。 請繼續提供您的意見！

**重要新功能和變更**

- 除了以 remote-docker 為基礎的執行以外，也支援在您自己的環境中，在遠端 Ubnutu VM 上執行原生指令碼。
- Workbench 應用程式中的新環境經驗可讓您建立計算目標，並執行以 CLI 為基礎的經驗以外的組態。
![環境索引標籤](media/azure-machine-learning-release-notes/environment-page.png)
- 可自訂的執行歷程記錄報告![新執行歷程記錄報告圖像](media/azure-machine-learning-release-notes/new-run-history-reports.png)

**詳細的更新**

以下是 Azure Machine Learning 在此 Sprint 中各元件區域的詳細更新清單。

### <a name="workbench-ui"></a>Workbench UI
- 可自訂的執行歷程記錄報告
  - 改善的圖表組態，適用於執行歷程記錄報告
    - 可以變更使用的進入點
    - 可以新增及修改最上層篩選條件![新增篩選條件](media/azure-machine-learning-release-notes/add-filters.jpg)
    - 可以新增或修改 (也可拖放並重新排列) 圖表與統計資料。
    ![建立新的圖表](media/azure-machine-learning-release-notes/configure-charts.png)

  - 適用於執行歷程記錄報告的 CRUD
  - 已將所有現有的執行歷程記錄清單檢視設定移至伺服器端報告，其動作會與在選定的進入點執行的管道類似。

- 環境索引標籤
  - 輕鬆地將新的計算目標和執行組態檔新增到專案中![新的計算的目標](media/azure-machine-learning-release-notes/add-new-environments.png)
  - 使用簡單、以表單為基礎的 UX 來管理與更新組態檔
  - 用來準備執行環境的新按鈕

- 增強側邊欄中檔案清單的效能

### <a name="data-preparation"></a>資料準備 
- Azure Machine Learning Workbench 現在可讓您使用已知的資料行名稱來搜尋資料行。


### <a name="experimentation"></a>測試
- Azure Machine Learning Workbench 現在支援在您自己的 Python 或 Pyspark 環境中執行原生指令碼。 若使用此功能，使用者必須在遠端 VM 上建立並管理自己的環境，並使用 Azure Machine Learning Workbench 在該目標上執行其指令碼。 請參閱[設定 Azure Machine Learning 測試服務](../desktop-workbench/experimentation-service-configuration.md) 

### <a name="model-management"></a>模型管理
- 支援自訂已部署的容器：允許您使用 apt-get 等命令安裝外部程式碼，以自訂容器映像。不再僅限於可安裝 pip 的程式庫。 請參閱[文件](../desktop-workbench/model-management-custom-container.md)以取得更多資訊。
  - 使用 `--docker-file myDockerStepsFilename` 旗標與檔案名稱搭配資訊清單、映像或服務建立命令。
  - 請注意，基礎映像為 Ubuntu，無法修改。
  - 範例命令： 
  
      ```shell
      $ az ml image create -n myimage -m mymodel.pkl -f score.py --docker-file mydockerstepsfile
      ```



## <a name="2018-01-sprint-3"></a>2018-01 (Sprint 3) 
**版本號碼**：0.1.1712.18263  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;([尋找您的版本](../desktop-workbench/known-issues-and-troubleshooting-guide.md#find-the-workbench-build-number))

以下是此 Sprint 中的更新和增強功能。 以下的更新有許多是直接來自使用者的意見反應。 


以下是 Azure Machine Learning 在此 Sprint 中各元件區域的詳細更新清單。

- 更新驗證堆疊會強制啟動時登入和選擇帳戶

### <a name="workbench"></a>Workbench
- 從 [新增/移除程式] 安裝/解除安裝應用程式的能力
- 更新驗證堆疊會強制啟動時登入和選擇帳戶
- 改善 Windows 上的單一登入 (SSO) 體驗
- 隸屬於多個租用戶且有不同認證的使用者，現在將可登入到 Workbench

### <a name="ui"></a>UI
- 一般增強功能和 Bug 修正

### <a name="notebooks"></a>筆記本
- 一般增強功能和 Bug 修正

### <a name="data-preparation"></a>資料準備 
- 改善執行 By Example 轉換時的自動建議
- 改善模式頻率偵測器的演算法
- 執行 By Example 轉換時傳送範例資料和意見反應的能力 ![在衍生資料行轉換時傳送意見反應連結的影像](media/azure-machine-learning-release-notes/SendFeedbackFromDeriveColumn.png)
- Spark 執行階段增強功能
- Scala 已取代 Pyspark
- 修正無法關閉時間序列偵測器不適用資料的問題 
- 修正針對 HDI 執行資料準備的擱置時間

### <a name="model-management-cli-updates"></a>模型管理 CLI 更新 
  - 佈建資源不再需要訂用帳戶的擁有權。 有資源群組的參與者存取權，就足以設定部署環境。
  - 針對免費訂用帳戶啟用本機環境設定 

## <a name="2017-12-sprint-2-qfe"></a>2017-12 (Sprint 2 QFE) 
**版本號碼**：0.1.1711.15323  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;([尋找您的版本](../desktop-workbench/known-issues-and-troubleshooting-guide.md#find-the-workbench-build-number))

這是 QFE (快速檢修) 版本，也就是次要版本。 此版本解決了幾個遙測問題，並可協助產品團隊深入了解使用者對產品的使用方式。 這些知識可以進一步用來改善產品體驗。 

此外，有兩個重要更新：

- 修正資料準備中的一個錯誤，避免資料準備封裝中顯示出時間序列偵測器。
- 在命令列工具中，不必身為 Azure 訂用帳戶擁有者也可以佈建 Machine Learning 計算 ACS 叢集。 

## <a name="2017-12-sprint-2"></a>2017-12 (Sprint 2)
**版本號碼**：0.1.1711.15263  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;([尋找您的版本](../desktop-workbench/known-issues-and-troubleshooting-guide.md#find-the-workbench-build-number))

歡迎使用 Azure Machine Learning 的第三次更新。 此更新包括 Workbench 應用程式、命令列介面 (CLI) 和後端服務的改進。 非常感謝您傳送的正反面意見。 以下的更新有許多是直接來自您的意見反應。 

**重要新功能**
- [支援 SQL Server 和 Azure SQL DB 作為資料來源](../desktop-workbench/data-prep-appendix2-supported-data-sources.md#types) 
- [使用 MMLSpark 在具備 GPU 支援的 Spark 上進行深度學習](https://github.com/Azure/mmlspark/blob/master/docs/gpu-setup.md) \(英文\)
- [所有 AML 容器在部署時都與 Azure IoT Edge 裝置相容 (不需要額外步驟)](http://aka.ms/aml-iot-edge-blog) \(英文\)
- Azure 入口網站提供註冊的模型清單和詳細資料檢視
- 除了透過使用者名稱/密碼存取計算目標之外，也可以使用以 SSH 金鑰為基礎的驗證進行存取。 
- 資料準備體驗中全新的模式頻率偵測器。 

**詳細更新內容** 以下是 Azure Machine Learning 在此 Sprint 中各元件區域的詳細更新清單。

### <a name="installer"></a>安裝程式
- 安裝程式可以自行更新，以在使用者不需重新安裝的情況下修正錯誤並支援新功能

### <a name="workbench-authentication"></a>Workbench 驗證
- 對驗證系統進行多項修正。 請讓我們知道您在登入時是否仍會發生問題。
- 可讓您更容易找到 Proxy 管理員設定的 UI 變更。

### <a name="workbench"></a>Workbench
- 唯讀檔案檢視現在具有淡藍色背景
- 將 [編輯] 按鈕移動到右邊，讓您更容易找到。
- "dsource"、"dprep" 和 "ipynb" 檔案格式現在可以以原始文字格式轉譯
- Workbench 現在擁有全新編輯體驗，可引導使用者使用外部 IDE 來編輯指令碼，並只使用 Workbench 來編輯可提供豐富編輯體驗的檔案類型 (例如 Notebook、資料來源、資料準備套件)
- 現在可以更快速地載入使用者擁有存取權之工作區和專案的清單

### <a name="data-preparation"></a>資料準備 
- 可檢視資料行中字串模式的模式頻率偵測器。 您也可以使用這些模式來篩選資料。 這會顯示和值計數偵測器類似的檢視。 差別在於樣式頻率會顯示資料的唯一模式計數，而非唯一資料計數。 您也可以篩選出或篩選掉符合特定模式的所有資料列。

![產品編號上模式頻率偵測器的影像](media/azure-machine-learning-release-notes/pattern-inspector-product-number.png)

- 效能提升，同時建議可在「衍生資料行實例化」轉換中檢閱的邊緣案例

- [支援 SQL Server 和 Azure SQL DB 作為資料來源](../desktop-workbench/data-prep-appendix2-supported-data-sources.md#types) 

![建立新 SQL 伺服器資料來源的影像](media/azure-machine-learning-release-notes/sql-server-data-source.png)

- 已啟用資料列和資料行計數的「概覽」檢視

![概覽之資料列資料行計數的影像](media/azure-machine-learning-release-notes/row-col-count.png)

- 已在所有計算內容中啟用資料準備
- 已在所有計算內容中啟用使用 SQL Server 資料庫的資料來源
- 可以依據資料類型篩選資料準備資料格資料行
- 已修正將多個資料行轉換成日期的問題
- 已修正如果使用者在進階模式中變更輸出資料行名稱，便可以在「衍生資料行實例化」中選取輸出資料行作為來源的問題。

### <a name="job-execution"></a>作業執行
您現在可以依照下列步驟，使用以 SSH 金鑰為基礎的驗證建立及存取 Remotedocker 或叢集類型計算目標：
- 在 CLI 中使用下列命令來附加計算目標

    ```azure-cli
    $ az ml computetarget attach remotedocker --name "remotevm" --address "remotevm_IP_address" --username "sshuser" --use-azureml-ssh-key
    ```
>[!NOTE]
>命令中的 -k (或 --use-azureml-ssh-key) 選項會指定產生及使用 SSH 金鑰。

- Azure ML Workbench 將會產生公開金鑰，並在您的主控台中輸出。 使用相同的使用者名稱登入計算目標，並附加具有此公開金鑰的 ~/.ssh/authorized_keys 檔案。

- 您可以準備此計算目標並將它用於執行，Azure ML Workbench 將會使用此金鑰進行驗證。  

如需建立計算目標的詳細資訊，請參閱[設定 Azure Machine Learning 測試服務](../desktop-workbench/experimentation-service-configuration.md)

### <a name="visual-studio-tools-for-ai"></a>適用於 AI 的 Visual Studio Tools
- 已新增對[適用於 AI 的 Visual Studio Tools](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vstoolsai-vs2017) \(英文\) 的支援。 

### <a name="command-line-interface-cli"></a>命令列介面 (CLI)
- 已新增 `az ml datasource create` 命令來允許從命令列建立資料來源檔案

### <a name="model-management-and-operationalization"></a>模型管理與作業
- [所有 AML 容器在作業時都與 Azure IoT Edge 裝置相容 (不需要額外步驟)](http://aka.ms/aml-iot-edge-blog) \(英文\) 
- o16n CLI 中錯誤訊息的改進
- 模型管理入口網站 UX 中的錯誤修正  
- 詳細資料頁面中模型管理屬性採用一致的字母大小寫
- 即時計分呼叫逾時設定為 60 秒
- Azure 入口網站提供註冊的模型清單和詳細資料檢視

![入口網站中的模型詳細資料](media/azure-machine-learning-release-notes/model-list.jpg)

![入口網站中的模型概觀](media/azure-machine-learning-release-notes/model-overview-portal.jpg)

### <a name="mmlspark"></a>MMLSpark
- 在具備 [GPU 支援](https://github.com/Azure/mmlspark/blob/master/docs/gpu-setup.md) \(英文\) 的 Spark 上進行深度學習
- 支援資源管理員範本，以便於進行資源部署
- 支援 SparklyR 生態系統
- [AZTK 整合](https://github.com/Azure/aztk/wiki/Spark-on-Azure-for-Python-Users#optional-set-up-mmlspark) \(英文\)

### <a name="sample-projects"></a>範例專案
- [鳶尾花](https://github.com/Azure/MachineLearningSamples-Iris) \(英文\) 和 [MMLSpark](https://github.com/Azure/mmlspark) \(英文\) 範例已更新為新的 Azure ML SDK 版本

### <a name="breaking-changes"></a>重大變更
- 將 `az ml computetarget attach` 中的 `--type` 參數升階為子命令。 

    - `az ml computetarget attach --type remotedocker` 現在為 `az ml computetarget attach remotedocker`
    - `az ml computetarget attach --type cluster` 現在為 `az ml computetarget attach cluster`

## <a name="2017-11-sprint-1"></a>2017-11 (Sprint 1) 
**版本號碼**：0.1.1710.31013  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;([尋找您的版本](../desktop-workbench/known-issues-and-troubleshooting-guide.md#find-the-workbench-build-number))

在這個版本中，我們改善了 Workbench 應用程式、CLI 和後端服務層的安全性、穩定性和可維護性。 非常感謝您傳送的肯定 (笑臉) 與不滿意 (苦臉)。 以下的更新有許多是直接來自您的意見反應。 歡迎繼續提供意見！

### <a name="notable-new-features"></a>重要新功能
- 現在有兩個新的 Azure 區域提供 Azure ML：**西歐**和**東南亞**。 它們原先加入的區域是**美國東部 2**、**美國中西部**和**澳洲東部**，現在部署區域總數增加至五個。
- 我們在 Workbench 應用程式中啟用了 Python 程式碼語法醒目提示，讓您更容易閱讀及編輯 Python 原始程式碼。 
- 您現在可以直接從檔案啟動您最愛的 IDE，而不用從整個專案中啟動。  在 Workbench 中開啟檔案，然後按一下 [編輯]，即可啟動目前檔案和專案的 IDE (目前支援 VS Code 和 PyCharm)。  您也可以按一下 [編輯] 按鈕旁邊的箭號，在 Workbench 文字編輯器中編輯檔案。  為防止意外變更，在您按下 [編輯] 之前，檔案都是唯讀的。
- Workbench 應用程式內附熱門的繪圖程式庫 `matplotlib` 2.1.0 版。
- 我們已將資料準備引擎升級至 .NET Core 2.0 版。 這會在 macOS 上安裝應用程式期間移除 brew-install openssl 需求。 這也是在為日後更有趣的資料準備功能鋪路。 
- 我們已啟用特定版本的應用程式首頁，方便您根據目前的應用程式版本取得更接近的版本資訊及更新提示。
- 即使您的本機使用者名稱中有空格，現在也可以成功安裝應用程式。 

### <a name="detailed-updates"></a>詳細的更新
以下是 Azure Machine Learning 此 Sprint 中各元件區域的詳細更新清單。

#### <a name="installer"></a>安裝程式
- 應用程式安裝程式現在會清除舊版應用程式建立的安裝目錄。
- 修正導致安裝程式在 macOS High Sierra 中卡在 100% 的 Bug。
- 如果安裝失敗，使用者現在可以直接連結到安裝目錄，檢閱安裝程式記錄檔。
- 使用者名稱中有空格的使用者現在也可以順利安裝。

#### <a name="workbench-authentication"></a>Workbench 驗證
- 支援 Proxy 管理員的驗證。
- 位於防火牆後方使用者現在可以順利登入。 
- 如果使用者在多個 Azure 區域有試驗帳戶，而某個區域剛好無法使用，應用程式不會再停止回應。
- 當驗證未完成，而且仍然顯示驗證對話方塊時，應用程式不會再嘗試從本機快取載入工作區。

#### <a name="workbench-app"></a>Workbench 應用程式
- 文字編輯器啟用 Python 程式碼語法醒目提示。
- 文字編輯器中的 [編輯] 按鈕可讓您在 IDE (支援 VS Code 和 PyCharm) 或內建文字編輯器中編輯檔案。
- 文字編輯器預設為唯讀模式。 
- [儲存] 按鈕視覺狀態現在在檔案儲存後會變更成停用，因此不會再變更。
- Workbench 會在您起始執行時，儲存_所有_未儲存的檔案。
- Workbench 會記住上次本機電腦使用過的工作區，自動開啟。
- 目前只允許執行單一的 Workbench 執行個體。 以前可以啟動多個執行個體，但會在操作相同專案時發生問題。
- [檔案] 功能表的 [開啟專案] 重新命名為 [Add Existing Folder as Project] (將現有資料夾新增為專案) 
- 索引標籤切換速度變快。
- 說明連結新增到 [Configuring IDE] (設定 IDE) 對話方塊中。
- 意見反應表單現在會記住您上次輸入的電子郵件地址。
- 肯定 (笑臉) 與不滿意 (苦臉) 表單的文字區域現在已加大，所以您可以傳送給我們更多的意見！ 
- 已更正 `az ml workspace create` 的 `--owner` 切換說明文字。
- 新增的 [關於] 對話方塊方便使用者輕鬆檢視及複製應用程式的版本號碼。
- [建議功能] 功能表項目已新增至 [說明] 功能表。
- 應用程式標題列現在會顯示試驗帳戶名稱，位在應用程式名稱 "Azure Machine Learning Workbench" 的前面。
- 現在會根據偵測到的應用程式版本，顯示特定版本的應用程式首頁。

#### <a name="data-preparation"></a>資料準備 
- 不再從地圖偵測器載入外部網站，以避免潛在的安全性問題。
- 長條圖和值計數偵測器現在可以選擇顯示對數刻度圖表。
- 現在在進行計算時，資料品質列會顯示不同的色彩，以表示「計算」狀態。
- 資料行計量現在會顯示類別值資料行的統計資料。
- 不再截斷資料來源名稱中的最後一個字元。
- 切換索引標籤、產生顯著的效能提升時，資料準備套件現在會保持開啟狀態。
- 在資料來源中切換資料檢視和計量檢視時，不再變更資料行的順序。
- 開啟無效的 `.dprep` 或 `.dsource` 檔案，不會再造成 Workbench 損毀。
- 資料準備套件現在可以使用相對路徑處理 [寫入至 CSV] 轉換的輸出。
- [保留資料行] 轉換現在可讓使用者在編輯時新增其他資料行。
- [Replace this] (取代此項) 功能表現在會實際啟動 [Replace Value] (取代值) 對話方塊。
- [Replace Value] (取代值) 轉換功能一如預期，不再擲回錯誤。
- 參考專案資料夾外部的資料檔案時，資料準備套件現在會使用絕對路徑，讓您可以使用資料檔案的絕對路徑在本機環境中執行套件。
- 現在使用 Azure BLOb 作為資料來源時，支援作為取樣策略的 [完整檔案]。
- 產生的 Python 程式碼 (來自資料準備套件) 現在會攜帶 CR 和 LF，方便使用 Windows 處理。
- [選擇度量] 下拉式清單現在切換至 [資料] 檢視時會隱藏屬性。
- Workbench 現在即使使用 Python 執行階段，也可以處理 parquet 檔案。 過去只能使用 Spark 處理 parquet 檔案。 
- 篩選出資料行中有_日期_資料類型的值，不會再造成資料準備引擎損毀。
- [計量] 檢視現在會遵守取樣策略更新。
- 遠端取樣作業現在已正確運作。

#### <a name="job-execution"></a>作業執行
- 引數現在包含在執行歷程記錄中。
- 使用 CLI 開始的作業現在會自動出現在 [Run history Job] (執行歷程記錄作業) 面板中。
- 作業面板現在會顯示新增至 Azure AD 租用戶之來賓使用者所建立的工作。
- 作業面板的取消工和刪除動作都更加穩定。
- 如果設定檔的格式不正確，現在按一下 [執行] 按鈕就會觸發錯誤訊息。
- 結束應用程式不會再干擾以 CLI 開始的作業。
- 使用 CLI 開始的作業現在即使執行一小時後，仍會繼續產出標準輸出。
- 當 Python/PySpark 中的資料準備套件執行失敗時，會顯示更清楚的錯誤訊息。
- `az ml experiment clean` 現在也會清除遠端 VM 中的 Docker 映像。
- `az ml experiment clean` 現在能夠在 macOS 上正確處理本機目標。
- 清除本機或遠端 Docker 執行目標時會出現錯誤訊息，且更容易閱讀。
- 附加為執行目標的 HDInsight 叢集前端節點名稱格式不正確時，會顯示更清楚的錯誤訊息。
- 憑證服務中找不到密碼時，會顯示更清楚的錯誤訊息。 
- 升級 MMLSpark 程式庫以支援 Apache Spark 2.2。
- MMLSpark 現在包含醫療文件的主體編碼轉換 (網狀結構編碼)。
- `matplotlib` 2.1.0 版現在直接隨附於 Workbench。

#### <a name="jupyter-notebook"></a>Jupyter Notebook
- Notebook 名稱搜尋在 [Notebook] 檢視中可正確運作。
- 您現在可以在 [Notebook] 檢視中刪除 Notebook。
- 新增新的魔術 `%upload_artifact` 將 Notebook 執行環境中產生的檔案上傳到執行歷程記錄資料存放區。
- 核心錯誤現在會顯示在 Notebook 作業狀態中以利偵錯。
- 現在當使用者登出應用程式時，Jupyter 伺服器會正確關閉。

#### <a name="azure-portal"></a>Azure 入口網站
- 現在可以在兩個新的 Azure 區域中建立試驗帳戶和模型管理帳戶：西歐和東南亞。
- 現在只有當模型管理帳戶是訂用帳戶中建立的第一個帳戶時，才提供 DevTest 計劃。 
- 更新 Azure 入口網站的 [說明] 連結，指向正確的文件頁面。
- 從 Docker 映像詳細資料頁面中移除 [描述] 欄位，因為它不適用。
- 包括 AppInsights 和自動調整規模設定的詳細資料會新增至 Web 服務詳細資料頁面。
- 現在即使瀏覽器中停用協力廠商 Cookie，也會呈現模型管理頁面。 

#### <a name="operationalization"></a>運算化
- 名稱中有 "score" 的 Web 服務不再失敗。
- 使用者現在只要有 Azure 資源群組或參與者存取權或訂用帳戶，就可以建立部署環境。 不再需要整個訂用帳戶的擁有者存取權。
- 作業 CLI 可在 Linux 上使用索引標籤自動完成。
- 映像建構服務現在支援 Azure IoT 服務/裝置的組建映像。

#### <a name="sample-projects"></a>範例專案
- [_分類光圈_](../desktop-workbench/tutorial-classifying-iris-part-1.md)範例專案：
    - `iris_pyspark.py` 已重新命名為 `iris_spark.py`。
    - `iris_score.py` 已重新命名為 `score_iris.py`。
    - `iris.dprep` 和 `iris.dsource` 已更新，以反映最新的資料準備引擎更新。
    - 已修改 `iris.ipynb` Notebook 使適用於 HDInsight 叢集。
    - `iris.ipynb` Notebook 資料格已開啟執行歷程記錄。
- 已修正[_使用自行車共用資料的進階資料準備_](../desktop-workbench/tutorial-bikeshare-dataprep.md)範例專案「處理錯誤值」步驟。
- [_MMLSpark 成人人口普查資料_](https://github.com/Azure/MachineLearningSamples-mmlspark)範例專案 `docker.runconfig` 格式從 JSON 更新為 YAML。
- [_分散式超參數微調_](../desktop-workbench/scenario-distributed-tuning-of-hyperparameters.md)範例專案 `docker.runconfig` 格式從 JSON 更新為 YAML。
- 新的範例專案[_使用 CNTK 進行影像分類_](../desktop-workbench/scenario-image-classification-using-cntk.md)。


## <a name="2017-10-sprint-0"></a>2017-10 (Sprint 0) 
**版本號碼**：0.1.1710.31013  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;([尋找您的版本](../desktop-workbench/known-issues-and-troubleshooting-guide.md#find-the-workbench-build-number))

歡迎使用我們在 Microsoft Ignite 2017 會議首次公開預覽 Microsoft Azure Machine Learning Workbench 後的第一次更新。 此版本的主要更新為可靠性和穩定化修正。  我們解決的一些重大問題包括：

### <a name="new-features"></a>新功能
- 目前已支援 macOS High Sierra

### <a name="bug-fixes"></a>錯誤修正
#### <a name="workbench-experience"></a>Workbench 體驗
- 將檔案拖放至 Workbench 會導致 Workbench 損毀。
- 在 VS 程式碼中設定為 IDE for Workbench 的終端機視窗無法辨識 _az ml_ 命令。

#### <a name="workbench-authentication"></a>Workbench 驗證
我們做一些更新來改善回報的各種登入及驗證問題。
- 特別是當網際網路連線不穩定時，驗證視窗會持續快顯的問題。
- 改善驗證權杖過期的可靠性。
- 在某些情況下，驗證視窗會出現兩次。
- 當訊息程序已完成並已關閉快顯對話方塊時，Workbench 主視窗仍會顯示「驗證中」訊息。
- 如果沒有網際網路連線，驗證對話方塊會出現空白畫面。

#### <a name="data-preparation"></a>資料準備 
- 篩選特定值後，也會篩選掉錯誤和遺漏值。
- 變更取樣策略可移除後續現有的加入作業。
- 取代遺漏值轉換不會考量 NaN。
- 遇到 null 值時，日期型別推斷會擲回例外狀況。

#### <a name="job-execution"></a>作業執行
- 當作業執行因其超過大小限制而無法上傳專案資料夾時，沒有任何明確的錯誤訊息。
- 如果使用者的 Python 指令碼變更工作目錄，不會追蹤寫入至輸出資料夾的檔案。 
- 如果作用中的 Azure 訂用帳戶不同於目前的專案所屬的訂用帳戶，則提交作業會產生 403 錯誤。
- 當 Docker 不存在時，如果使用者嘗試將 Docker 做為執行目標，不傳回任何明確的錯誤訊息。
- 當使用者按一下 [執行] 按鈕時不會自動儲存 .runconfig 檔案。

#### <a name="jupyter-notebook"></a>Jupyter Notebook
- 如果使用者使用特定的登入類型，則無法啟動 Notebook 伺服器。
- Notebook 伺服器錯誤訊息不會出現在使用者可看見的記錄檔中。

#### <a name="azure-portal"></a>Azure 入口網站
- 選取深色的 Azure 入口網站佈景主題會將 ML 模型管理刀鋒視窗顯示為黑色方塊。

#### <a name="operationalization"></a>運算化
- 重複使用要更新 web 服務的資訊清單，會使用隨機的名稱建立新的 Docker 映像。
- 無法從 Kubernetes 叢集中擷取 web 服務記錄檔。
- 當使用者嘗試建立模型管理帳戶或 ML 計算帳戶而發生權限問題時，可能造成誤導的錯誤訊息。

## <a name="next-steps"></a>後續步驟

閱讀 [Azure Machine Learning](../service/overview-what-is-azure-ml.md) 概觀。
