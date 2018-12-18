---
title: 使用 Chef 的 Azure 虛擬機器部署 | Microsoft Docs
description: 了解如何使用 Chef 執行自動化的虛擬機器部署和設定 Microsoft Azure
services: virtual-machines-windows
documentationcenter: ''
author: diegoviso
manager: jeconnoc
tags: azure-service-management,azure-resource-manager
editor: ''
ms.assetid: 0b82ca70-89ed-496d-bb49-c04ae59b4523
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-multiple
ms.devlang: na
ms.topic: article
ms.date: 05/30/2017
ms.author: diviso
ms.openlocfilehash: 3a6fbc8410dbc5aec4522f0972a29c67527edb23
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037978"
---
# <a name="automating-azure-virtual-machine-deployment-with-chef"></a>使用 Chef 自動化 Azure 虛擬機器部署
[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

Chef 是個很棒的工具，可提供自動化和所需狀態組態。

在我們最新的雲端應用程式 API 版本中，Chef 提供與 Azure 的緊密整合，您可以透過單一命令佈建和部署組態狀態。

在本文中，您將設定可佈建 Azure 虛擬機器的 Chef 環境，並會逐步指導您建立原則或 “CookBook”，然後將此操作手冊部署到 Azure 虛擬機器中。

讓我們開始吧！

## <a name="chef-basics"></a>Chef 基本概念
開始之前，請[檢閱 Chef 的基本概念](http://www.chef.io/chef) \(英文\)。 

下圖說明高層級的 Chef 架構。

![][2]

Chef 有三個主要的架構元件：Chef 伺服器、Chef 用戶端 (節點) 和 Chef 工作站。

Chef 伺服器是管理的重點，Chef 伺服器包含兩個選項：代管解決方案或內部部署解決方案。 我們將使用代管解決方案。

Chef 用戶端 (節點)是位於您所管理之伺服器上的代理程式。

Chef 工作站是系統管理工作站，我們可以在這裡建立原則並執行管理命令。 我們可以從 Chef 工作站執行管理基礎結構的 **knife** 命令。

此外還有 “Cookbooks” 和 “Recipes” 的概念。 這些實際上是我們定義並套用至伺服器的原則。

## <a name="preparing-the-workstation"></a>準備工作站
首先準備工作站。 使用標準的 Windows 工作站。 我們需要建立可儲存設定檔和 CookBook 的目錄。

首先，建立名為 C:\chef 的目錄。

然後建立名為 c:\chef\cookbooks 的第二個目錄。

我們現在必須下載 Azure 設定檔，Chef 才能與 Azure 訂用帳戶進行通訊。

使用 PowerShell Azure [Get-AzurePublishSettingsFile](https://docs.microsoft.com/powershell/module/servicemanagement/azure/get-azurepublishsettingsfile?view=azuresmps-4.0.0) 命令來下載您的發佈設定。 

將發行設定檔儲存在 C:\chef 中。

## <a name="creating-a-managed-chef-account"></a>建立受控 Chef 帳戶
在 [這裡](https://manage.chef.io/signup)註冊代管的 Chef 帳戶。

在註冊過程中，我們將要求您建立新的組織。

![][3]

建立組織後，請下載「入門套件」。

![][4]

> [!NOTE]
> 如果您收到提示，警告您將重新設定金鑰，您可以繼續作業，因為我們尚未設定任何基礎結構。
> 
> 

此入門套件 zip 檔案包含您的組織組態檔和金鑰。

## <a name="configuring-the-chef-workstation"></a>設定 Chef 工作站
將 chef-starter.zip 的內容解壓縮到 C:\chef。

將 chef-starter\chef-repo\.chef 下的所有檔案複製到您的 c:\chef 目錄。

您的目錄現在看起來應該類似以下範例。

![][5]

您現在應該會有 4 個檔案，包括 c:\chef 根目錄中的 Azure 發行檔案。

PEM 檔案包含可進行通訊的組織和管理員私密金鑰，而 knife.rb檔案則包含 knife 組態。 我們將需要編輯 knife.rb 檔案。

在您選擇的編輯器中開啟此檔案，並修改 “cookbook_path” (移除其路徑中的 /../)，因此它會顯示如下所示。

    cookbook_path  ["#{current_dir}/cookbooks"]

並新增下列反映 Azure 發行設定檔名稱的程式碼行。

    knife[:azure_publish_settings_file] = "yourfilename.publishsettings"

現在 knife.rb 檔案看起來應該會類似下列範例。

![][6]

這幾行程式碼可確保 Knife 會參考 c:\chef\cookbooks 底下的 cookbooks 目錄，並在 Azure 作業期間使用 Azure 發行設定檔。

## <a name="installing-the-chef-development-kit"></a>安裝 Chef 開發套件
接下來 [下載並安裝](http://downloads.getchef.com/chef-dk/windows) ChefDK (Chef 開發套件) 以設定 Chef 工作站。

![][7]

安裝在 c:\opscode 預設位置。 此安裝大約需要 10 分鐘的時間。

確認您的 PATH 變數包含 C:\opscode\chefdk\bin、C:\opscode\chefdk\embedded\bin、c:\users\yourusername\.chefdk\gem\ruby\2.0.0\bin 等項目

如果沒有，請確定您已加入這些路徑 ！

*請注意，路徑的順序很重要！* 如果您的 opscode 路徑順序不正確，則會出現問題。

在繼續之前，請重新啟動您的工作站。

接下來，我們將安裝 Knife Azure 延伸模組。 這會以「Azure 外掛程式」的形式提供 Knife。

執行下列命令。

    chef gem install knife-azure ––pre

> [!NOTE]
> -pre 引數可確保您會收到最新的 RC 版本 Knife Azure 外掛程式，該版本可讓您存取最新的 API 組合。
> 
> 

同時也可能安裝多個相依性。

![][8]

若要確保一切都已正確設定，請執行下列命令。

    knife azure image list

如果一切都已正確設定，您會在捲動時看到可用的 Azure 映像清單。

恭喜！ 工作站已設定！

## <a name="creating-a-cookbook"></a>建立 Cookbook
Chef 會使用 Cookbook 來定義一組您想在受控用戶端上執行的命令。 建立 Cookbook 非常簡單，我們可以使用 **chef generate cookbook** 命令來產生 Cookbook 範本。 我將呼叫我的 Cookbook Web 伺服器，因為我需要可自動部署 IIS 的原則。

在 C:\Chef 目錄下，執行下列命令。

    chef generate cookbook webserver

這會在 C:\Chef\cookbooks\webserver 目錄下產生一組檔案。 我們現在需要定義一組需要 Chef 用戶端在受控虛擬機器上執行的命令。

這些命令會儲存在 default.rb. 檔案中在這個檔案中，請定義一組用來安裝 IIS、啟動 IIS 並將範本檔案複製到 wwwroot 資料夾的命令。

修改 C:\chef\cookbooks\webserver\recipes\default.rb 檔並加入下列幾行程式碼。

    powershell_script 'Install IIS' do
         action :run
         code 'add-windowsfeature Web-Server'
    end

    service 'w3svc' do
         action [ :enable, :start ]
    end

    template 'c:\inetpub\wwwroot\Default.htm' do
         source 'Default.htm.erb'
         rights :read, 'Everyone'
    end

完成後，請儲存檔案。

## <a name="creating-a-template"></a>建立範本
如先前所述，我們需要產生可作為 default.html 頁面的範本檔案。

執行下列命令以產生範本。

    chef generate template webserver Default.htm

現在瀏覽至 C:\chef\cookbooks\webserver\templates\default\Default.htm.erb 檔案。 加入一些簡單的 "Hello World" HTML 程式碼來編輯檔案，然後儲存檔案。

## <a name="upload-the-cookbook-to-the-chef-server"></a>將 Cookbook 上傳到 Chef 伺服器
在此步驟中，我們會將在本機電腦上建立的 Cookbook 複本，上傳到 Chef 代管伺服器。 上傳後，Cookbook 便會出現在 [原則]  索引標籤底下。

    knife cookbook upload webserver

![][9]

## <a name="deploy-a-virtual-machine-with-knife-azure"></a>使用 Knife Azure 部署虛擬機器
我們現在要部署 Azure 虛擬機器，並套用 “Webserver” Cookbook，如此便會安裝 IIS Web 服務和預設網頁。

若要這樣做，請使用 **knife azure server create** 命令。

接下來會顯示此命令的範例。

    knife azure server create --azure-dns-name 'diegotest01' --azure-vm-name 'testserver01' --azure-vm-size 'Small' --azure-storage-account 'portalvhdsxxxx' --bootstrap-protocol 'cloud-api' --azure-source-image 'a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-Datacenter-201411.01-en.us-127GB.vhd' --azure-service-location 'Southeast Asia' --winrm-user azureuser --winrm-password 'myPassword123' --tcp-endpoints 80,3389 --r 'recipe[webserver]'

這些參數一看就懂。 替換特定變數並執行。

> [!NOTE]
> 透過命令列，我還打算使用 -tcp-endpoints 參數將端點網路篩選規則自動化。 我已經開放連接埠 80 和 3389 以供網頁和 RDP 工作階段存取。
> 
> 

執行命令後，前往 Azure 入口網站，您會看到已經開始佈建您的機器。

![][13]

命令提示字元會顯示下一步。

![][10]

部署完成之後，我們應該能夠透過連接埠 80 連接到 Web 服務，因為我們使用 Knife Azure 命令佈建虛擬機器時已將此連接埠開啟。 由於此虛擬機器是我的雲端服務中唯一的虛擬機器，我要使用雲端服務 URL 來進行連接。

![][11]

如您所見，我的 HTML 程式碼開始有點意思。

別忘了我們也可以透過連接埠 3389，從 Azure 入口網站的 RDP 工作階段進行連線。

我希望這對您有所幫助！ 現在就開始使用 Azure 來體驗基礎結構即程式碼！

<!--Image references-->
[2]: media/chef-automation/2.png
[3]: media/chef-automation/3.png
[4]: media/chef-automation/4.png
[5]: media/chef-automation/5.png
[6]: media/chef-automation/6.png
[7]: media/chef-automation/7.png
[8]: media/chef-automation/8.png
[9]: media/chef-automation/9.png
[10]: media/chef-automation/10.png
[11]: media/chef-automation/11.png
[13]: media/chef-automation/13.png


<!--Link references-->
