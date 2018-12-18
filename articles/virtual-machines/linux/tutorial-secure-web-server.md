---
title: 教學課程：在 Azure 中使用 SSL 憑證來保護 Linux 網頁伺服器 | Microsoft Docs
description: 在本教學課程中，您將了解如何搭配使用 Azure CLI 與 Azure Key Vault 中儲存的 SSL 憑證，來保護執行 NGINX 網頁伺服器的 Linux 虛擬機器。
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
ms.date: 04/30/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: a0156167142e87ffb7935828de1000a302f12e31
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981681"
---
# <a name="tutorial-secure-a-web-server-on-a-linux-virtual-machine-in-azure-with-ssl-certificates-stored-in-key-vault"></a>教學課程：在 Azure 中使用 Key Vault 內儲存的 SSL 憑證，來保護 Linux 虛擬機器上的網頁伺服器
若要保護網頁伺服器，您可以使用安全通訊端層 (SSL) 憑證將 Web 流量加密。 這些 SSL 憑證可儲存在 Azure Key Vault，並且能夠讓您將憑證安全地部署到 Azure 中的 Linux 虛擬機器 (VM)。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立 Azure Key Vault
> * 產生或上傳憑證至 Key Vault
> * 建立 VM 並安裝 NGINX 網頁伺服器
> * 將憑證插入 VM 並使用 SSL 繫結來設定 NGINX

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。


## <a name="overview"></a>概觀
Azure Key Vault 會保護密碼編譯金鑰和祕密，像是憑證或密碼。 Key Vault 有助於簡化憑證管理程序，並可讓您掌控用來存取這些憑證的金鑰。 您可以在 Key Vault 內建立自我簽署憑證，或上傳您目前已經擁有的受信任憑證。

您不必使用包含了內建憑證的自訂 VM 映像，而是要將憑證插入執行中的 VM。 此程序可確保您在部署期間安裝在網頁伺服器上的憑證會是最新的。 如果您更新或取代憑證，您就不必另外再建立新的自訂 VM 映像。 當您建立其他 VM 時，系統會自動插入最新的憑證。 在整個過程中，憑證絕對不會離開 Azure 平台，或在指令碼、命令列記錄或範本中公開。


## <a name="create-an-azure-key-vault"></a>建立 Azure Key Vault
建立 Key Vault 和憑證之前，請先使用 [az group create](/cli/azure/group#az_group_create) 來建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroupSecureWeb 的資源群組：

```azurecli-interactive 
az group create --name myResourceGroupSecureWeb --location eastus
```

接著，使用 [az keyvault create](/cli/azure/keyvault#az_keyvault_create) 建立 Key Vault，並加以啟用以供您在部署 VM 時使用。 每個 Key Vault 都需要唯一的名稱，且應全部使用小寫。 使用您自己唯一的 Key Vault 名稱來取代下列範例中的 *<mykeyvault>*：

```azurecli-interactive 
keyvault_name=<mykeyvault>
az keyvault create \
    --resource-group myResourceGroupSecureWeb \
    --name $keyvault_name \
    --enabled-for-deployment
```

## <a name="generate-a-certificate-and-store-in-key-vault"></a>產生憑證並儲存於 Key Vault
若要在生產環境中使用，您應該使用 [az keyvault certificate import](/cli/azure/keyvault/certificate#az_keyvault_certificate_import) 來匯入由受信任的提供者所簽署的有效憑證。 在本教學課程中，下列範例示範如何透過使用預設憑證原則的 [az keyvault certificate create](/cli/azure/keyvault/certificate#az_keyvault_certificate_create) 來產生自我簽署憑證：

```azurecli-interactive 
az keyvault certificate create \
    --vault-name $keyvault_name \
    --name mycert \
    --policy "$(az keyvault certificate get-default-policy)"
```

### <a name="prepare-a-certificate-for-use-with-a-vm"></a>準備要與 VM 搭配使用的憑證
若要在 VM 建立程序期間使用憑證，使用 [az keyvault secret list-versions](/cli/azure/keyvault/secret#az_keyvault_secret_list_versions) 來取得憑證的識別碼。 使用 [az vm secret format](/cli/azure/vm/secret#az-vm-secret-format) 轉換憑證。 下列範例會將這些命令的輸出指派給變數，以方便在後續步驟中使用：

```azurecli-interactive 
secret=$(az keyvault secret list-versions \
          --vault-name $keyvault_name \
          --name mycert \
          --query "[?attributes.enabled].id" --output tsv)
vm_secret=$(az vm secret format --secrets "$secret")
```

### <a name="create-a-cloud-init-config-to-secure-nginx"></a>建立 Cloud-init 組態來保護 NGINX
[Cloud-init (英文)](https://cloudinit.readthedocs.io) 是在 Linux VM 初次開機時，廣泛用來自訂它們的方法。 您可以使用 cloud-init 來安裝封裝和寫入檔案，或者設定使用者和安全性。 當 cloud-init 在初次開機程序期間執行時，不需要使用任何額外的步驟或必要的代理程式來套用您的組態。

當您建立 VM 時，憑證和金鑰會儲存在受保護的 /var/lib/waagent/ 目錄中。 若要自動將憑證新增至 VM 並設定網頁伺服器，請使用 cloud-init。 在此範例中，您會安裝和設定 NGINX Web 伺服器。 您可以使用相同的程序來安裝和設定 Apache。 

建立名為 cloud-init-web-server.txt 的檔案，並貼上下列組態：

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 443 ssl;
        ssl_certificate /etc/nginx/ssl/mycert.cert;
        ssl_certificate_key /etc/nginx/ssl/mycert.prv;
      }
runcmd:
  - secretsname=$(find /var/lib/waagent/ -name "*.prv" | cut -c -57)
  - mkdir /etc/nginx/ssl
  - cp $secretsname.crt /etc/nginx/ssl/mycert.cert
  - cp $secretsname.prv /etc/nginx/ssl/mycert.prv
  - service nginx restart
```

### <a name="create-a-secure-vm"></a>建立安全的 VM
現在，使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 使用 `--secrets` 參數，從 Key Vault 插入憑證資料。 使用 `--custom-data` 參數傳入 cloud-init 組態：

```azurecli-interactive 
az vm create \
    --resource-group myResourceGroupSecureWeb \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init-web-server.txt \
    --secrets "$vm_secret"
```

系統需要花幾分鐘的時間來建立 VM、安裝封裝和啟動應用程式。 建立 VM 之後，請注意 Azure CLI 所顯示的 `publicIpAddress`。 您可以使用此位址在網頁瀏覽器中存取您的網站。

若要讓 Web 流量安全到達您的 VM，請使用 [az vm open-port](/cli/azure/vm#az_vm_open_port) 從網際網路開啟通訊埠 443：

```azurecli-interactive 
az vm open-port \
    --resource-group myResourceGroupSecureWeb \
    --name myVM \
    --port 443
```


### <a name="test-the-secure-web-app"></a>測試安全的 Web 應用程式
現在，您可以開啟 Web 瀏覽器，並在網址列輸入 *https://<publicIpAddress>*。 提供您自己從 VM 建立程序中取得的公用 IP 位址。 如果您使用自我簽署憑證，請接受安全性警告：

![接受 Web 瀏覽器安全性警告](./media/tutorial-secure-web-server/browser-warning.png)

接著會顯示受保護的 NGINX 網站，如下列範例所示：

![檢視執行中安全的 NGINX 網站](./media/tutorial-secure-web-server/secured-nginx.png)


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已使用儲存在 Azure Key Vault 中的 SSL 憑證來保護 NGINX 網頁伺服器。 您已了解如何︰

> [!div class="checklist"]
> * 建立 Azure Key Vault
> * 產生或上傳憑證至 Key Vault
> * 建立 VM 並安裝 NGINX 網頁伺服器
> * 將憑證插入 VM 並使用 SSL 繫結來設定 NGINX

用以下連結查看預先建立的虛擬機器指令碼範例。

> [!div class="nextstepaction"]
> [Linux 虛擬機器指令碼範例](./cli-samples.md)
