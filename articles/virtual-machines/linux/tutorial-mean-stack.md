---
title: 教學課程 - 在 Azure 的 Linux 虛擬機器上建立 MEAN 堆疊 | Microsoft Docs
description: 在本教學課程中，您會了解如何在 Azure 中的 Linux VM 上建立 MongoDB、Express、AngularJS 和 Node.js (MEAN) 堆疊。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/08/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: a23dd71b9453a11298192fc909c5a3c9708f11b0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46962295"
---
# <a name="tutorial-create-a-mongodb-express-angularjs-and-nodejs-mean-stack-on-a-linux-virtual-machine-in-azure"></a>教學課程：在 Azure 中的 Linux 虛擬機器上建立 MongoDB、Express、AngularJS 和 Node.js (MEAN) 堆疊

本教學課程示範如何在 Azure 中的 Linux 虛擬機器 (VM) 上實作 MongoDB、Express、AngularJS 和 Node.js (MEAN) 堆疊。 您所建立的 MEAN 堆疊可在資料庫中新增、刪除和列出書籍。 您會了解如何：

> [!div class="checklist"]
> * 建立 Linux VM
> * 安裝 Node.js
> * 安裝 MongoDB 並設定伺服器
> * 安裝 Express 並設定通往伺服器的路由
> * 使用 AngularJS 存取路由
> * 執行應用程式

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。


## <a name="create-a-linux-vm"></a>建立 Linux VM

使用 [az group create](https://docs.microsoft.com/cli/azure/group#az_group_create) 命令建立資源群組，並使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#az_vm_create) 命令建立 Linux VM。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

下列範例使用 Azure CLI 在 eastus 位置建立名為 myResourceGroupMEAN 的資源群組。 如果預設的金鑰位置還沒有 SSH 金鑰的話，此範例也會建立具有這些金鑰的 VM (名為 myVM)。 若要使用一組特定金鑰，請使用 --ssh-key-value 選項。

```azurecli-interactive
az group create --name myResourceGroupMEAN --location eastus
az vm create \
    --resource-group myResourceGroupMEAN \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --admin-password 'Azure12345678!' \
    --generate-ssh-keys
az vm open-port --port 3300 --resource-group myResourceGroupMEAN --name myVM
```

建立 VM 後，Azure CLI 會顯示類似下列範例的資訊： 

```azurecli-interactive
{
  "fqdns": "",
  "id": "/subscriptions/{subscription-id}/resourceGroups/myResourceGroupMEAN/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.72.77.9",
  "resourceGroup": "myResourceGroupMEAN"
}
```
記下 `publicIpAddress`。 此位址用來存取 VM。

使用下列命令來對 VM 建立 SSH 工作階段。 請務必使用正確的公用 IP 位址。 在上面的範例中，我們的 IP 位址是 13.72.77.9。

```bash
ssh azureuser@13.72.77.9
```

## <a name="install-nodejs"></a>安裝 Node.js

[Node.js](https://nodejs.org/en/) 是建置在 Chrome V8 JavaScript 引擎上的 JavaScript 執行階段。 本教學課程使用 Node.js 來設定 Express 路由和 AngularJS 控制器。

在 VM 上使用您透過 SSH 所開啟的 bash 殼層來安裝 Node.js。

```bash
sudo apt-get install -y nodejs
```

## <a name="install-mongodb-and-set-up-the-server"></a>安裝 MongoDB 並設定伺服器
[MongoDB](http://www.mongodb.com) 會將資料儲存在類似 JSON 的彈性文件中。 資料庫中的欄位會隨著文件而不同，而且資料結構也會隨著時間而改變。 在我們的應用程式範例中，我們要新增書籍記錄到 MongoDB，這些記錄中包含了書籍名稱、isbn 編號、作者和頁數。 

1. 在 VM 上使用您透過 SSH 所開啟的 bash 殼層來設定 MongoDB 金鑰。

    ```bash
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
    echo "deb [ arch=amd64 ] http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
    ```

2. 使用該金鑰更新套件管理員。
  
    ```bash
    sudo apt-get update
    ```

3. 安裝 MongoDB。

    ```bash
    sudo apt-get install -y mongodb
    ```

4. 啟動伺服器。

    ```bash
    sudo service mongodb start
    ```

5. 我們也必須安裝[本文剖析器](https://www.npmjs.com/package/body-parser-json)套件，以協助我們處理透過要求傳遞至伺服器的 JSON。

    安裝 npm 套件管理員。

    ```bash
    sudo apt-get install npm
    ```

    安裝本文剖析器套件。
    
    ```bash
    sudo npm install body-parser
    ```

6. 建立名為 Books 的資料夾，並於其中新增名為 server.js 的檔案，此檔案中包含 Web 伺服器的組態。

    ```node.js
    var express = require('express');
    var bodyParser = require('body-parser');
    var app = express();
    app.use(express.static(__dirname + '/public'));
    app.use(bodyParser.json());
    require('./apps/routes')(app);
    app.set('port', 3300);
    app.listen(app.get('port'), function() {
        console.log('Server up: http://localhost:' + app.get('port'));
    });
    ```

## <a name="install-express-and-set-up-routes-to-the-server"></a>安裝 Express 並設定通往伺服器的路由

[Express](https://expressjs.com) 是最小且具有彈性的 Node.js Web 應用程式架構，可為 Web 和行動應用程式提供功能。 本教學課程使用 Express 以在 MongoDB 資料庫中傳入和傳出書籍資訊。 [Mongoose](http://mongoosejs.com) 提供簡單的結構描述型解決方案，來為您的應用程式資料建立模型。 本教學課程使用 Mongoose 來為資料庫提供書籍結構描述。

1. 安裝 Express 和 Mongoose。

    ```bash
    sudo npm install express mongoose
    ```

2. 在 Books 資料夾中，建立名為 apps 的資料夾，並新增定義了 Express 路由且名為 routes.js 的檔案。

    ```node.js
    var Book = require('./models/book');
    module.exports = function(app) {
      app.get('/book', function(req, res) {
        Book.find({}, function(err, result) {
          if ( err ) throw err;
          res.json(result);
        });
      }); 
      app.post('/book', function(req, res) {
        var book = new Book( {
          name:req.body.name,
          isbn:req.body.isbn,
          author:req.body.author,
          pages:req.body.pages
        });
        book.save(function(err, result) {
          if ( err ) throw err;
          res.json( {
            message:"Successfully added book",
            book:result
          });
        });
      });
      app.delete("/book/:isbn", function(req, res) {
        Book.findOneAndRemove(req.query, function(err, result) {
          if ( err ) throw err;
          res.json( {
            message: "Successfully deleted the book",
            book: result
          });
        });
      });
      var path = require('path');
      app.get('*', function(req, res) {
        res.sendfile(path.join(__dirname + '/public', 'index.html'));
      });
    };
    ```

3. 在 apps 資料夾中，建立名為 models 的資料夾，並新增定義了書籍模型組態且名為 book.js 的檔案。  

    ```node.js
    var mongoose = require('mongoose');
    var dbHost = 'mongodb://localhost:27017/test';
    mongoose.connect(dbHost);
    mongoose.connection;
    mongoose.set('debug', true);
    var bookSchema = mongoose.Schema( {
      name: String,
      isbn: {type: String, index: true},
      author: String,
      pages: Number
    });
    var Book = mongoose.model('Book', bookSchema);
    module.exports = mongoose.model('Book', bookSchema); 
    ```

## <a name="access-the-routes-with-angularjs"></a>使用 AngularJS 存取路由

[AngularJS](https://angularjs.org) 提供一種 Web 架構讓您在 Web 應用程式中建立動態檢視。 在本教學課程中，我們會使用 AngularJS 來以 Express 連線我們的網頁，並對我們的書籍資料庫執行動作。

1. 將目錄變更回到 Books (`cd ../..`)，然後建立名為 public 的資料夾，並新增定義了控制器組態且名為 script.js 的檔案。

    ```node.js
    var app = angular.module('myApp', []);
    app.controller('myCtrl', function($scope, $http) {
      $http( {
        method: 'GET',
        url: '/book'
      }).then(function successCallback(response) {
        $scope.books = response.data;
      }, function errorCallback(response) {
        console.log('Error: ' + response);
      });
      $scope.del_book = function(book) {
        $http( {
          method: 'DELETE',
          url: '/book/:isbn',
          params: {'isbn': book.isbn}
        }).then(function successCallback(response) {
          console.log(response);
        }, function errorCallback(response) {
          console.log('Error: ' + response);
        });
      };
      $scope.add_book = function() {
        var body = '{ "name": "' + $scope.Name + 
        '", "isbn": "' + $scope.Isbn +
        '", "author": "' + $scope.Author + 
        '", "pages": "' + $scope.Pages + '" }';
        $http({
          method: 'POST',
          url: '/book',
          data: body
        }).then(function successCallback(response) {
          console.log(response);
        }, function errorCallback(response) {
          console.log('Error: ' + response);
        });
      };
    });
    ```
    
2. 在 public 資料夾中，建立定義了網頁且名為 index.html 的檔案。

    ```html
    <!doctype html>
    <html ng-app="myApp" ng-controller="myCtrl">
      <head>
        <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
        <script src="script.js"></script>
      </head>
      <body>
        <div>
          <table>
            <tr>
              <td>Name:</td> 
              <td><input type="text" ng-model="Name"></td>
            </tr>
            <tr>
              <td>Isbn:</td>
              <td><input type="text" ng-model="Isbn"></td>
            </tr>
            <tr>
              <td>Author:</td> 
              <td><input type="text" ng-model="Author"></td>
            </tr>
            <tr>
              <td>Pages:</td>
              <td><input type="number" ng-model="Pages"></td>
            </tr>
          </table>
          <button ng-click="add_book()">Add</button>
        </div>
        <hr>
        <div>
          <table>
            <tr>
              <th>Name</th>
              <th>Isbn</th>
              <th>Author</th>
              <th>Pages</th>
            </tr>
            <tr ng-repeat="book in books">
              <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
              <td>{{book.name}}</td>
              <td>{{book.isbn}}</td>
              <td>{{book.author}}</td>
              <td>{{book.pages}}</td>
            </tr>
          </table>
        </div>
      </body>
    </html>
    ```

##  <a name="run-the-application"></a>執行應用程式

1. 將目錄變更回到 Books (`cd ..`)，並執行這個命令來啟動伺服器：

    ```bash
    nodejs server.js
    ```

2. 將網頁瀏覽器開啟到您為 VM 記錄的位址。 例如：*http://13.72.77.9:3300*。 您應該會看到如下列網頁的內容：

    ![書籍記錄](media/tutorial-mean/meanstack-init.png)

3. 在文字方塊中輸入資料，然後按一下 [新增]。 例如︰

    ![新增書籍記錄](media/tutorial-mean/meanstack-add.png)

4. 在重新整理網頁之後，您應該會看到如下列網頁的內容：

    ![列出書籍記錄](media/tutorial-mean/meanstack-list.png)

5. 您可以按一下 [刪除]，從資料庫中移除書籍記錄。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立 Web 應用程式以在 Linux VM 上使用 MEAN 堆疊來留下書籍記錄。 您已了解如何︰

> [!div class="checklist"]
> * 建立 Linux VM
> * 安裝 Node.js
> * 安裝 MongoDB 並設定伺服器
> * 安裝 Express 並設定通往伺服器的路由
> * 使用 AngularJS 存取路由
> * 執行應用程式

前進到下一個教學課程，以了解如何使用 SSL 憑證保護 Web 伺服器。

> [!div class="nextstepaction"]
> [使用 SSL 保護網路伺服器](tutorial-secure-web-server.md)
