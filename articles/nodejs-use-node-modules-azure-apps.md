---
title: 使用 Node.js 模組
description: 了解如何在使用 Azure App Service 或雲端服務時使用 Node.js 模組。
services: ''
documentationcenter: nodejs
author: TomArcher
manager: routlaw
editor: ''
ms.assetid: c0e6cd3d-932d-433e-b72d-e513e23b4eb6
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 08/17/2016
ms.author: tarcher
ms.openlocfilehash: 76679ea0ff2c1e88d1923488717a245351437165
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23036463"
---
# <a name="using-nodejs-modules-with-azure-applications"></a>使用 Node.js 模組與 Azure 應用程式搭配
本文提供有關使用 Node.js 模組與 Azure 上代管之應用程式搭配的指引。 它提供有關確保應用程式使用模組特定版本，以及搭配原生模組與 Azure 使用的指引。

如果您已熟悉使用 Node.js 模組、**package.json** 和 **npm-shrinkwrap.json** 檔案，則下列資訊可提供本文中討論內容的快速摘要：

* Azure App Service 熟悉 **package.json** 和 **npm-shrinkwrap.json** 檔案，並可根據這些檔案中的項目安裝模組。

* Azure 雲端服務期望在開發環境上安裝所有模組，且 **node\_modules** 目錄會包括為部署封裝的一部分。 提供支援使用雲端服務上的 **package.json** 或 **npm-shrinkwrap.json** 檔案來安裝模組是有可能的，不過，這項設定需要自訂雲端服務專案所使用的預設指令碼。 如需如何設定此環境的範例，請參閱[執行 npm 安裝以避免部署節點模組的 Azure 啟動工作](https://github.com/woloski/nodeonazure-blog/blob/master/articles/startup-task-to-run-npm-in-azure.markdown)

> [!NOTE]
> 本文中將不會討論 Azure 虛擬機器，因為 VM 中的部署經驗會視虛擬機器所代管的作業系統而定。
> 
> 

## <a name="nodejs-modules"></a>Node.js 模組
模組是指可載入的 JavaScript 封裝，可為您的應用程式提供特定功能。 模組的安裝方式通常是使用 **npm** 命令列工具，不過，也有一些模組 (例如 http 模組) 會以核心 Node.js 封裝的一部分提供。

安裝模組時，模組會儲存在應用程式目錄結構之根目錄的 **node\_modules** 目錄中。 **node\_modules** 目錄中的每個模組都會維護它自己的 **node\_modules** 目錄 (其中包含它所依賴的任何模組)，且這個行為會在相依性鏈結一路向下的每個模組中重複進行。 這個環境可讓每個已安裝模組都有它自己所相依的模組版本需求，不過，它會產生相當大的目錄結構。

將 **node\_modules** 目錄作為應用程式一部分進行部署時，相較於使用 **package.json** 或 **npm-shrinkwrap.json** 檔案，它的部署大小會增加；不過，它可以確實保證用於生產的模組版本與開發中所用的模組版本相同。

### <a name="native-modules"></a>原生模組
雖然大多數的模組是簡單的純文字 JavaScript 檔案，有些模組卻是平台特定的二進位影像。 這些模組會在安裝時編譯，通常是採用 Python 和 node-gyp。 由於 Azure 雲端服務仰賴將 **node\_modules** 資料夾當作應用程式的一部分進行部署，任何包括為安裝模組一部分的原生模組應可在雲端服務中運作，只要它是在 Windows 開發系統上安裝與編譯的即可。

Azure App Service 不支援所有的原生模組，而且在編譯具有特定必要元件的模組時，可能會失敗。 雖然某些熱門模組 (如 MongoDB) 具有選擇性原生相依性，而且沒有這些相依性仍照常運作，但兩種因應措施成功證明目前幾乎可使用所有的原生模組：

* 在已安裝所有原生模組之必要元件的 Windows 電腦上執行 **npm install** 。 然後，建立的 **node\_modules** 資料夾部署為 Azure App Service 應用程式的一部分。

  * 開始編譯之前，請確認您的本機 Node.js 安裝有相符的架構，且版本儘可能接近 Azure 中使用的版本 (可在執行階段從 **process.arch** 和 **process.version** 屬性查到目前的值)。

* Azure App Service 可以設定為在部署期間執行自訂 Bash 或 Shell 指令碼，讓您有機會執行自訂命令以及精確地設定 **npm install** 的執行方式。 如需示範如何設定該環境的影片，請參閱[使用 Kudu 自訂網站部署指令碼]。

### <a name="using-a-packagejson-file"></a>使用 package.json 檔案

**package.json** 檔案是一種方法，可用來指定應用程式要求的最上層相依性，以便主控平台可安裝相依性，而不是要求您包含 **node\_modules** 資料夾作為部署的一部分。 在部署應用程式之後，您可使用 **npm install** 命令，來剖析 **package.json** 檔案並安裝所有列出的相依性。

開發期間，當安裝模組將模組項目自動新增至 **package.json** 檔案時，您可以使用 **--save**、**--save-dev** 或 **--save-optional** 參數。 如需詳細資訊，請參閱 [npm-install](https://docs.npmjs.com/cli/install)(英文)。

有關 **package.json** 檔案的一個潛在問題是它只指定最上層相依性的版本。 每個已安裝模組不一定會指定它所相依的模組版本，而且您最終得到的相依性鏈結可能與在開發中所用的不同。

> [!NOTE]
> 部署到 Azure App Service 時，如果您的 <b>package.json</b> 檔案參考原生模組，當使用 Git 發行應用程式時，可能會看到類似以下範例的錯誤：
> 
> npm ERR! module-name@0.6.0 install: 'node-gyp configure build'
> 
> npm ERR! 'cmd "/c" "node-gyp configure build"' failed with 1
> 
> 

### <a name="using-a-npm-shrinkwrapjson-file"></a>使用 npm-shrinkwrap.json 檔案
**npm-shrinkwrap.json** 檔案嘗試解決 **package.json** 檔案的模組版本設定限制。 **package.json** 檔案只包含最上層模組的版本，而 **npm-shrinkwrap.json** 檔案包含完整模組相依性鏈結的版本需求。

當應用程式準備好開始生產時，您可以鎖定版本需求，並使用 **npm shrinkwrap** 命令來建立 **npm-shrinkwrap.json** 檔案。 此命令會使用目前安裝在 **node\_modules** 資料夾中的版本，並將這些版本記錄到 **npm-shrinkwrap.json** 檔案。 在部署應用程式到主控環境之後，使用 **npm install** 命令來剖析 **npm-shrinkwrap.json** 檔案，並安裝所有列出的相依性。 如需詳細資訊，請參閱 [npm-shrinkwrap](https://docs.npmjs.com/cli/shrinkwrap)。

> [!NOTE]
> 部署到 Azure App Service 時，如果您的 <b>npm-shrinkwrap.json</b> 檔案參考原生模組，當使用 Git 發行應用程式時，可能會看到類似以下範例的錯誤：
> 
> npm ERR! module-name@0.6.0 install: 'node-gyp configure build'
> 
> npm ERR! 'cmd "/c" "node-gyp configure build"' failed with 1
> 
> 

## <a name="next-steps"></a>後續步驟
現在，您了解如何搭配使用 Node.js 模組與 Azure，接著了解如何[指定 Node.js 版本]、[建置與部署 Node.js Web 應用程式](app-service/app-service-web-get-started-nodejs.md)，和[如何使用適用於 Mac 和 Linux 的 Azure 命令列介面]。

如需詳細資訊，請參閱 [Node.js 開發人員中心](/nodejs/azure/)。

[指定 Node.js 版本]: nodejs-specify-node-version-azure-apps.md
[如何使用適用於 Mac 和 Linux 的 Azure 命令列介面]:cli-install-nodejs.md
[使用 Kudu 自訂網站部署指令碼]: https://channel9.msdn.com/Shows/Azure-Friday/Custom-Web-Site-Deployment-Scripts-with-Kudu-with-David-Ebbo
