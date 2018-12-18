# [雲端服務文件](index.md)

# 概觀
## [什麼是雲端服務？](cloud-services-choose-me.md)
## [雲端服務組態檔與封裝](cloud-services-model-and-package.md)

# 開始使用
## [.NET 雲端服務範例](cloud-services-dotnet-get-started.md)
## [Python for Visual Studio 雲端服務範例](cloud-services-python-ptvs.md)
## [使用 Microsoft HPC Pack 設定混合式 HPC 叢集](cloud-services-setup-hybrid-hpcpack-cluster.md)

# 作法

## 規劃
### [虛擬機器大小](cloud-services-sizes-specs.md)
### [更新](cloud-services-update-azure-service.md)

## 開發
### [建立 PHP Web 和背景工作角色](../cloud-services-php-create-web-role.md)
### [建置和部署 Node.js 應用程式](cloud-services-nodejs-develop-deploy-app.md)
### [使用 Express 建置 Node.js Web 應用程式](cloud-services-nodejs-develop-deploy-express-app.md)
### 儲存體和 Visual Studio
#### [Blob 儲存體和已連接的服務](../visual-studio/vs-storage-cloud-services-getting-started-blobs.md)
#### [佇列儲存體和已連接的服務](../visual-studio/vs-storage-cloud-services-getting-started-queues.md)
#### [表格儲存體和已連接的服務](../visual-studio/vs-storage-cloud-services-getting-started-tables.md)
### [設定角色的流量規則](cloud-services-enable-communication-role-instances.md)
### [處理雲端服務生命週期事件](cloud-services-role-lifecycle-dotnet.md)
### [Socket.io (Node.js)](cloud-services-nodejs-chat-app-socketio.md)
### [使用 Twilio 撥打電話 (.NET)](../partner-twilio-cloud-services-dotnet-phone-call-web-role.md)

### 設定啟動工作
#### [建立啟動工作](cloud-services-startup-tasks.md)
#### [常見啟動工作](cloud-services-startup-tasks-common.md)
#### [使用工作在雲端服務角色上安裝 .NET](cloud-services-dotnet-install-dotnet.md)

### 設定遠端桌面
#### [入口網站](cloud-services-role-enable-remote-desktop-new-portal.md)
#### [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md)
#### [Visual Studio](cloud-services-role-enable-remote-desktop-visual-studio.md)

## 部署
### [在入口網站中建立和部署雲端服務](cloud-services-how-to-create-deploy-portal.md)
### [使用 PowerShell 建立空的雲端服務容器](cloud-services-powershell-create-cloud-container.md)
### [設定自訂網域名稱](cloud-services-custom-domain-name-portal.md)
### [連接到自訂網域控制站](cloud-services-connect-to-custom-domain.md)

## 管理服務
### [一般管理工作](cloud-services-how-to-manage-portal.md)
### [設定雲端服務](cloud-services-how-to-configure-portal.md)
### [使用 Azure 自動化管理雲端服務](automation-manage-cloud-services.md)
### [設定自動調整](cloud-services-how-to-scale-portal.md)
### [使用 Python 管理 Azure 資源](cloud-services-python-how-to-use-service-management.md)
### [減輕風險的理論式執行](mitigate-se.md)

### [客體 OS 修補程式](cloud-services-guestos-msrc-releases.md)
### 客體 OS 淘汰
#### [淘汰原則](cloud-services-guestos-retirement-policy.md)
#### [系列 1 淘汰通知](cloud-services-guestos-family1-retirement.md)
### [客體 OS 發佈新聞](cloud-services-guestos-update-matrix.md)
### [雲端服務角色設定 XPath 速查表](cloud-services-role-config-xpath.md)

## 管理憑證
### [雲端服務與管理憑證](cloud-services-certs-create.md)
### [設定 SSL](cloud-services-configure-ssl-certificate-portal.md)

## 監視
### [監視雲端服務](cloud-services-how-to-monitor.md)
### [使用效能計數器](diagnostics-performance-counters.md)
### [測試效能](../vs-azure-tools-performance-profiling-cloud-services.md)
#### [使用 Visual Studio 分析工具進行測試](cloud-services-performance-testing-visual-studio-profiler.md)
### 啟用診斷
#### [Azure PowerShell](cloud-services-diagnostics-powershell.md)
#### [.NET](cloud-services-dotnet-diagnostics.md)
#### [Visual Studio](../vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines.md)
### [在 Azure 儲存體中儲存和檢視診斷資料](../monitoring-and-diagnostics/azure-diagnostics-storage.md?toc=%2fazure%2fcloud-services%2ftoc.json )
### [使用診斷追蹤雲端服務](cloud-services-dotnet-diagnostics-trace-flow.md)

## 疑難排解
### 偵錯 
#### [雲端服務的選項](../vs-azure-tools-debugging-cloud-services-overview.md)
#### [使用 Visual Studio 的本機雲端服務](../vs-azure-tools-debug-cloud-services-virtual-machines.md)
#### [使用 Visual Studio 的已發佈雲端服務](../vs-azure-tools-intellitrace-debug-published-cloud-services.md)
### [雲端服務配置失敗](cloud-services-allocation-failures.md)
### [回收雲端服務角色的常見原因](cloud-services-troubleshoot-common-issues-which-cause-roles-recycle.md)
### [預設 TEMP 資料夾大小對角色而言太小](cloud-services-troubleshoot-default-temp-folder-size-too-small-web-worker-role.md)
### [常見部署問題](cloud-services-troubleshoot-deployment-problems.md)
### [角色無法啟動](cloud-services-troubleshoot-roles-that-fail-start.md)
### [復原指引](cloud-services-disaster-recovery-guidance.md)
### 雲端服務常見問題集
#### [應用程式和服務可用性常見問題集](cloud-services-application-and-service-availability-faq.md)
#### [組能和管理常見問題](cloud-services-configuration-and-management-faq.md)
#### [連線和網路服務常見問題](cloud-services-connectivity-and-networking-faq.md)
#### [部署常見問題](cloud-services-deployment-faq.md)

# 參考
## [程式碼範例](https://azure.microsoft.com/resources/samples/?service=cloud-services)
## [.csdef XML 結構描述](schema-csdef-file.md)
### [LoadBalancerProbe 結構描述](schema-csdef-loadbalancerprobe.md)
### [WebRole 結構描述](schema-csdef-webrole.md)
### [WorkerRole 結構描述](schema-csdef-workerrole.md)
### [NetworkTrafficRules 結構描述](schema-csdef-networktrafficrules.md)
## [.cscfg XML 結構描述](schema-cscfg-file.md)
### [角色結構描述](schema-cscfg-role.md)
### [NetworkConfiguration 結構描述](schema-cscfg-networkconfiguration.md)
## [REST](/rest/api/compute/cloudservices/)

# 資源
## [Azure 藍圖](https://azure.microsoft.com/roadmap/?category=compute)
## [學習路徑](https://azure.microsoft.com/documentation/learning-paths/cloud-services/)
## [MSDN 論壇](https://social.msdn.microsoft.com/Forums/en-us/home?forum=windowsazuredevelopment)
## [定價](https://azure.microsoft.com/pricing/details/cloud-services/)
## [定價計算機](https://azure.microsoft.com/pricing/calculator/)
## [服務更新](https://azure.microsoft.com/updates/?product=cloud-services&updatetype=&platform=)
## [影片](https://azure.microsoft.com/documentation/videos/index/?services=cloud-services)
