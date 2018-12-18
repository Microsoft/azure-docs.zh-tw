---
title: 在 Linux VM 上裝載 Ruby on Rails 網站 | Microsoft 文件
description: 在使用 Linux 虛擬機器的 Azure 上設定及裝載 Ruby on Rails 型網站。
services: virtual-machines-linux
documentationcenter: ruby
author: rmcmurray
manager: erikre
editor: ''
tags: azure-service-management
ms.assetid: aad32685-3550-4bff-9c73-beb8d70b3291
ms.service: virtual-machines-linux
ms.workload: web
ms.tgt_pltfrm: vm-linux
ms.devlang: ruby
ms.topic: article
ms.date: 06/27/2017
ms.author: robmcm
ms.openlocfilehash: 6ea1d249b7f9aec3a45923b162a97ce7f83d0d31
ms.sourcegitcommit: d551ddf8d6c0fd3a884c9852bc4443c1a1485899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2018
ms.locfileid: "37901148"
---
# <a name="ruby-on-rails-web-application-on-an-azure-vm"></a>Azure VM 上的 Ruby on Rails Web 應用程式
此教學課程說明如何在 Azure 上使用 Linux 虛擬機器，於 Rails 網站裝載 Ruby。  

此教學課程使用 Ubuntu Server 14.04 LTS 通過驗證。 若使用不同的 Linux 發行版本，您可能需要修改這些步驟以安裝 Rails。

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../../../azure-resource-manager/resource-manager-deployment-model.md)。  本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]
>

## <a name="create-an-azure-vm"></a>建立 Azure VM
開始使用 Linux 映像建立 Azure VM。

若要建立 VM，您可以使用 Azure 入口網站或 Azure 命令列介面 (CLI)。

### <a name="azure-portal"></a>Azure 入口網站
1. 登入 [Azure 入口網站](https://portal.azure.com)
2. 按一下 [建立資源]，然後在搜尋方塊中輸入 "Ubuntu Server 14.04"。 按一下由搜尋傳回的項目。 針對部署模型，選取 [傳統]，然後按一下 [建立]。
3. 在 [基本] 刀鋒視窗中，提供必要欄位的值：名稱 (適用於 VM)、使用者名稱、驗證類型和對應的認證、Azure 訂用帳戶、資源群組及位置。

   ![Create a new Ubuntu Image](./media/virtual-machines-linux-classic-ruby-rails-web-app/createvm.png)

4. 佈建 VM 之後，按一下 VM 名稱，然後按一下 [設定] 分類中的 [端點]。 尋找列在 [獨立] 下方的 SSH 端點。

   ![預設端點](./media/virtual-machines-linux-classic-ruby-rails-web-app/endpointsnewportal.png)

### <a name="azure-cli"></a>Azure CLI
請依照[建立執行 Linux 的虛擬機器][vm-instructions]中的步驟執行。

佈建 VM 之後，您可以透過執行下列命令來取得 SSH 端點：

    azure vm endpoint list <vm-name>  

## <a name="install-ruby-on-rails"></a>安裝 Ruby on Rails
1. 使用 SSH 連線到 VM。
2. 從 SSH 工作階段中，使用下列命令在 VM 上安裝 Ruby：

        sudo apt-get update -y
        sudo apt-get upgrade -y

        sudo apt-add-repository ppa:brightbox/ruby-ng
        sudo apt-get update
        sudo apt-get install ruby2.4

        > [!TIP]
        > The brightbox repository contains the current Ruby distribution.

    安裝可能需要幾分鐘的時間。 完成時，請使用下列命令來確認 Ruby 是否已安裝：

        ruby -v

3. 使用下列命令來安裝 Rails：

        sudo gem install rails --no-rdoc --no-ri -V

    使用 --no-rdoc 與 --no-ri 旗標來略過安裝文件，這樣安裝速度會比較快。
    此命令的執行可能需要很長的時間，所以新增 -V 會顯示安裝進度的相關資訊。

## <a name="create-and-run-an-app"></a>建立及執行應用程式
在仍透過 SSH 登入的情況下，執行下列命令：

    rails new myapp
    cd myapp
    rails server -b 0.0.0.0 -p 3000

[new](http://guides.rubyonrails.org/command_line.html#rails-new) 命令會建立新的 Rails 應用程式。 [server](http://guides.rubyonrails.org/command_line.html#rails-server) 命令會啟動 Rails 隨附的 WEBrick Web 伺服器。 (對於實際執行環境用途，您可能想要使用不同的伺服器，例如 Unicorn 或 Passenger)。

您應該會看到如下所示的輸出。

    => Booting WEBrick
    => Rails 4.2.1 application starting in development on http://0.0.0.0:3000
    => Run `rails server -h` for more startup options
    => Ctrl-C to shutdown server
    [2015-06-09 23:34:23] INFO  WEBrick 1.3.1
    [2015-06-09 23:34:23] INFO  ruby 1.9.3 (2013-11-22) [x86_64-linux]
    [2015-06-09 23:34:23] INFO  WEBrick::HTTPServer#start: pid=27766 port=3000

## <a name="add-an-endpoint"></a>新增端點。
1. 移至 \[Azure 入口網站\][https://portal.azure.com] 並選取您的虛擬機器。

2. 在頁面左邊，選取 [設定] 中的 [端點]。

3. 按一下頁面頂端的 [新增]。

4. 在 [新增端點] 對話方塊頁面中，輸入下列資訊：

   * **名稱**：HTTP
   * **通訊協定**：TCP
   * **公用連接埠**：80
   * **私用連接埠**：3000
   * **浮動 IP 位址**：已停用
   * **存取控制清單 - 順序**：1001，或另一個設定此存取規則之優先順序的值。
   * **存取控制清單 - 名稱**：allowHTTP
   * **存取控制清單 - 動作**：允許
   * **存取控制清單 - 遠端子網路**：1.0.0.0/16

     此端點具有公用連接埠 80，其會將流量路由傳送至 Rails 伺服器所接聽的私用連接埠 3000。 存取控制清單規則允許連接埠 80 上的公用流量。

     ![new-endpoint](./media/virtual-machines-linux-classic-ruby-rails-web-app/createendpoint.png)

5. 按一下 [確定] 以儲存端點。

6. 此時應會出現一則訊息，指出「正在儲存虛擬機器端點」。 這個訊息消失後，端點便在作用中。 您現在可以瀏覽至虛擬機器的 DNS 名稱，測試您的應用程式。 網站應如下所示：

    ![預設 rails 頁面][default-rails-cloud]

## <a name="next-steps"></a>後續步驟
在此教學課程中，您必須手動執行大部分的步驟。 在生產環境中，您會在開發電腦上撰寫應用程式，並將它部署至 Azure VM。 另外，大部分生產環境均代管 Rails 應用程式以及 Apache 或 NginX 之類的其他伺服器程序，處理傳送至多個 Rails 應用程式及執行個體並提供靜態資源的要求。 如需詳細資訊，請參閱 http://guides.rubyonrails.org/routing.html。

若要深入了解 Ruby on Rails，請瀏覽 [Ruby on Rails 指南 (英文)][rails-guides]。

若要從您的 Ruby 應用程式 使用 Azure 服務，請參閱：

* [使用 Blob 儲存非結構化資料][blobs]
* [使用資料表儲存機碼/值組][tables]
* [使用內容傳遞網路提供高頻寬內容][cdn-howto]

<!-- WA.com links -->
[blobs]:../../../storage/blobs/storage-ruby-how-to-use-blob-storage.md
[cdn-howto]:https://azure.microsoft.com/develop/ruby/app-services/
[tables]:../../../cosmos-db/table-storage-how-to-use-ruby.md
[vm-instructions]:createportal-classic.md

<!-- External Links -->
[rails-guides]:http://guides.rubyonrails.org/
[sqlite3]:http://www.sqlite.org/

<!-- Images -->

[default-rails-cloud]:./media/virtual-machines-linux-classic-ruby-rails-web-app/basicrailscloud.png
[vmlist]:./media/virtual-machines-linux-classic-ruby-rails-web-app/vmlist.png
[endpoints]:./media/virtual-machines-linux-classic-ruby-rails-web-app/endpoints.png
[new-endpoint]:./media/virtual-machines-linux-classic-ruby-rails-web-app/newendpoint.png
[new-endpoint1]:./media/virtual-machines-linux-classic-ruby-rails-web-app/newendpoint1.png
