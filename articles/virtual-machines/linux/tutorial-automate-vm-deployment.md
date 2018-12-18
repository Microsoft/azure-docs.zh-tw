---
title: 教學課程 - 在 Azure 中使用 cloud-init 自訂 Linux VM | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 cloud-init 和 Key Vault，以便在 Linux VM 初次於 Azure 中開機時加以自訂
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/30/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: a023d9f600f63d1ead4cf7c7e2a3cceec794d28b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46972585"
---
# <a name="tutorial---how-to-use-cloud-init-to-customize-a-linux-virtual-machine-in-azure-on-first-boot"></a>教學課程 - 如何使用 cloud-init 在首次開機時於 Azure 中自訂 Linux 虛擬機器

在先前的教學課程中，您學到了如何以 SSH 連線到虛擬機器 (VM) 並手動安裝 NGINX。 若要以快速且一致的方式建立 VM，您通常需要借助某種形式的自動化。 在初次開機時自訂 VM 的常見方法是使用 [cloud-init (英文)](https://cloudinit.readthedocs.io)。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立 Cloud-init 組態檔
> * 建立 VM 來使用 Cloud-init 檔案
> * 在建立 VM 之後，檢視執行中的 Node.js 應用程式
> * 使用 Key Vault 安全地儲存憑證
> * 使用 Cloud-init 來安全地自動部署 NGINX

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="cloud-init-overview"></a>Cloud-Init 概觀
[Cloud-init (英文)](https://cloudinit.readthedocs.io) 是在 Linux VM 初次開機時，廣泛用來自訂它們的方法。 您可以使用 cloud-init 來安裝封裝和寫入檔案，或者設定使用者和安全性。 當 cloud-init 在初次開機程序期間執行時，不需要使用任何額外的步驟或必要的代理程式來套用您的組態。

Cloud-init 也適用於散發套件。 例如，您不使用 **apt-get install** 或 **yum install** 來安裝套件。 您可以改為定義要安裝的套件清單。 Cloud-init 會針對您選取的散發套件自動使用原生的套件管理工具。

我們正在與合作夥伴合作，以期在他們提供給 Azure 的映像中包含和使用 cloud-init。 下表概述 cloud-init 目前在 Azure 平台映像上的可用性：

| Alias | 發行者 | 供應項目 | SKU | 版本 |
|:--- |:--- |:--- |:--- |:--- |:--- |
| UbuntuLTS |Canonical |UbuntuServer |16.04-LTS |最新 |
| UbuntuLTS |Canonical |UbuntuServer |14.04.5-LTS |最新 |
| CoreOS |CoreOS |CoreOS |Stable |最新 |
| | OpenLogic | CentOS | 7-CI | 最新 |
| | RedHat | RHEL | 7-RAW-CI | 最新 |


## <a name="create-cloud-init-config-file"></a>建立 Cloud-init 組態檔
若要查看作用中的 cloud-init，請建立 VM 來安裝 NGINX 並執行簡單 'Hello World' Node.js 應用程式。 下列 cloud-init 組態會安裝必要的封裝、建立 Node.js 應用程式，然後初始化並啟動應用程式。

您目前的殼層中，建立名為 cloud-init.txt 的檔案，並貼上下列組態。 例如，在 Cloud Shell 中建立不在本機電腦上的檔案。 您可以使用任何您想要的編輯器。 輸入 `sensible-editor cloud-init.txt` 可建立檔案，並查看可用的編輯器清單。 請確定已正確複製整個 cloud-init 檔案，特別是第一行：

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

如需 Cloud-init 組態選項的詳細資訊，請參閱 [Cloud-init 組態範例](https://cloudinit.readthedocs.io/en/latest/topics/examples.html) \(英文\)。

## <a name="create-virtual-machine"></a>建立虛擬機器
建立 VM 之前，請先使用 [az group create](/cli/azure/group#az-group-create) 來建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroupAutomate 的資源群組：

```azurecli-interactive
az group create --name myResourceGroupAutomate --location eastus
```

現在，使用 [az vm create](/cli/azure/vm#az-vm-create) 建立 VM。 使用 `--custom-data` 參數以傳入 cloud-init 組態檔。 如果您將檔案儲存於目前工作目錄之外的位置，請提供 cloud-init.txt 組態的完整路徑。 下列範例會建立名為 myAutomatedVM 的 VM：

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupAutomate \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
```

系統需要花幾分鐘的時間來建立 VM、安裝封裝和啟動應用程式。 在 Azure CLI 將您返回提示字元之後，背景工作會繼續執行。 可能需要再等候幾分鐘，才能存取應用程式。 建立 VM 之後，請注意 Azure CLI 所顯示的 `publicIpAddress`。 此位址是用來透過 Web 瀏覽器存取 Node.js 應用程式。

若要讓 Web 流量到達您的 VM，請使用 [az vm open-port](/cli/azure/vm#az-vm-open-port) 從網際網路開啟通訊埠 80：

```azurecli-interactive
az vm open-port --port 80 --resource-group myResourceGroupAutomate --name myVM
```

## <a name="test-web-app"></a>測試 Web 應用程式
現在，您可以開啟 Web 瀏覽器，並在網址列輸入 *http://<publicIpAddress>*。 提供您自己從 VM 建立程序中取得的公用 IP 位址。 您的 Node.js 應用程式即會顯示，如下列範例所示：

![檢視執行中的 NGINX 網站](./media/tutorial-automate-vm-deployment/nginx.png)


## <a name="inject-certificates-from-key-vault"></a>從 Key Vault 插入憑證
您可以選擇性閱讀這一節，其中示範如何安全地將憑證儲存在 Azure Key Vault 中，並在 VM 部署期間插入它們。 不需使用包含憑證的自訂映像內建中，此程序可確保會在初次開機時將最新憑證插入至 VM。 過程中，憑證絕對不會離開 Azure 平台，或在指令碼、命令列記錄或範本中公開。

Azure Key Vault 會保護密碼編譯金鑰和祕密，像是憑證或密碼。 Key Vault 有助於簡化金鑰管理程序，並可讓您控管存取和加密資料的金鑰。 此案例會引進一些 Key Vault 概念來建立和使用憑證，但並不是如何使用 Key Vault 的詳盡概觀。

下列步驟將示範如何：

- 建立 Azure Key Vault
- 產生或上傳憑證至 Key Vault
- 從憑證建立祕密以插入至 VM
- 建立 VM 並插入憑證

### <a name="create-an-azure-key-vault"></a>建立 Azure Key Vault
首先，使用 [az keyvault create](/cli/azure/keyvault#az-keyvault-create) 建立 Key Vault，並加以啟用以供您在部署 VM 時使用。 每個 Key Vault 需要唯一的名稱，而且應該全部小寫。 使用您自己唯一的 Key Vault 名稱來取代下列範例中的 mykeyvault：

```azurecli-interactive
keyvault_name=mykeyvault
az keyvault create \
    --resource-group myResourceGroupAutomate \
    --name $keyvault_name \
    --enabled-for-deployment
```

### <a name="generate-certificate-and-store-in-key-vault"></a>產生憑證並儲存於 Key Vault
若要在生產環境中使用，您應該使用 [az keyvault certificate import](/cli/azure/keyvault/certificate#az-keyvault-certificate-import) 來匯入由受信任的提供者所簽署的有效憑證。 在本教學課程中，下列範例示範如何透過使用預設憑證原則的 [az keyvault certificate create](/cli/azure/keyvault/certificate#az-keyvault-certificate-create) 來產生自我簽署憑證：

```azurecli-interactive
az keyvault certificate create \
    --vault-name $keyvault_name \
    --name mycert \
    --policy "$(az keyvault certificate get-default-policy)"
```


### <a name="prepare-certificate-for-use-with-vm"></a>準備要與 VM 搭配使用的憑證
若要在 VM 建立程序期間使用憑證，使用 [az keyvault secret list-versions](/cli/azure/keyvault/secret#az-keyvault-secret-list-versions) 來取得憑證的識別碼。 VM 需要特定格式的憑證，才能在開機時將其插入，因此請將憑證轉換為 [az vm secret format](/cli/azure/vm#az-vm-secret-format)。 下列範例會將這些命令的輸出指派給變數，以方便在後續步驟中使用：

```azurecli-interactive
secret=$(az keyvault secret list-versions \
          --vault-name $keyvault_name \
          --name mycert \
          --query "[?attributes.enabled].id" --output tsv)
vm_secret=$(az vm secret format --secret "$secret")
```


### <a name="create-cloud-init-config-to-secure-nginx"></a>建立 Cloud-init 組態來保護 NGINX
當您建立 VM 時，憑證和金鑰會儲存在受保護的 /var/lib/waagent/ 目錄中。 若要將憑證自動新增至 VM 並設定 NGINX，您可以從上一個範例中使用更新的 cloud-init 組態。

建立名為 cloud-init-secured.txt 的檔案，並貼上下列組態。 同樣地，如果您使用 Cloud Shell，請在該處而非在本機電腦上建立 cloud-init 組態檔。 使用 `sensible-editor cloud-init-secured.txt` 可建立檔案，並查看可用的編輯器清單。 請確定已正確複製整個 cloud-init 檔案，特別是第一行：

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        listen 443 ssl;
        ssl_certificate /etc/nginx/ssl/mycert.cert;
        ssl_certificate_key /etc/nginx/ssl/mycert.prv;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - secretsname=$(find /var/lib/waagent/ -name "*.prv" | cut -c -57)
  - mkdir /etc/nginx/ssl
  - cp $secretsname.crt /etc/nginx/ssl/mycert.cert
  - cp $secretsname.prv /etc/nginx/ssl/mycert.prv
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

### <a name="create-secure-vm"></a>建立安全的 VM
現在，使用 [az vm create](/cli/azure/vm#az-vm-create) 建立 VM。 使用 `--secrets` 參數，從 Key Vault 插入憑證資料。 如同先前範例，您也要使用 `--custom-data` 參數來傳入 cloud-init 組態：

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupAutomate \
    --name myVMSecured \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init-secured.txt \
    --secrets "$vm_secret"
```

系統需要花幾分鐘的時間來建立 VM、安裝封裝和啟動應用程式。 在 Azure CLI 將您返回提示字元之後，背景工作會繼續執行。 可能需要再等候幾分鐘，才能存取應用程式。 建立 VM 之後，請注意 Azure CLI 所顯示的 `publicIpAddress`。 此位址是用來透過 Web 瀏覽器存取 Node.js 應用程式。

若要讓 Web 流量安全到達您的 VM，請使用 [az vm open-port](/cli/azure/vm#az-vm-open-port) 從網際網路開啟通訊埠 443：

```azurecli-interactive
az vm open-port \
    --resource-group myResourceGroupAutomate \
    --name myVMSecured \
    --port 443
```

### <a name="test-secure-web-app"></a>測試安全的 Web 應用程式
現在，您可以開啟 Web 瀏覽器，並在網址列輸入 *https://<publicIpAddress>*。 提供您自己的公用 IP 位址，其已顯示在先前 VM 建立程序中的輸出中。 如果您使用自我簽署憑證，請接受安全性警告：

![接受 Web 瀏覽器安全性警告](./media/tutorial-automate-vm-deployment/browser-warning.png)

接著會顯示受保護的 NGINX 網站和 Node.js 應用程式，如下列範例所示：

![檢視執行中安全的 NGINX 網站](./media/tutorial-automate-vm-deployment/secured-nginx.png)


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已在初次開機時使用 Cloud-init 來設定 VM。 您已了解如何︰

> [!div class="checklist"]
> * 建立 Cloud-init 組態檔
> * 建立 VM 來使用 Cloud-init 檔案
> * 在建立 VM 之後，檢視執行中的 Node.js 應用程式
> * 使用 Key Vault 安全地儲存憑證
> * 使用 Cloud-init 來安全地自動部署 NGINX

前進至下一個教學課程，以了解如何建立自訂的 VM 映像。

> [!div class="nextstepaction"]
> [建立自訂的 VM 映像](./tutorial-custom-images.md)
