---
title: 在 Azure 中的開發虛擬機器上部署 ElasticSearch
description: 教學課程 - 在 Azure 中的開發 Linux VM 上安裝彈性堆疊
services: virtual-machines-linux
documentationcenter: virtual-machines
author: rloutlaw
manager: justhe
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 10/11/2017
ms.author: routlaw
ms.openlocfilehash: 86461ea5840ec4f2ea70facd642de9e7e5c05fa5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981004"
---
# <a name="install-the-elastic-stack-on-an-azure-vm"></a>在 Azure VM 上安裝彈性堆疊

本文將逐步引導您了解如何在 Azure 中的 Ubuntu VM 上部署 [Elasticsearch](https://www.elastic.co/products/elasticsearch) \(英文\)、[Logstash](https://www.elastic.co/products/logstash) \(英文\) 和 [Kibana](https://www.elastic.co/products/kibana) \(英文\)。 若要查看運作中的彈性堆疊，您可以選擇性地連接到 Kibana，然後處理一些記錄資料的範例。 

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 在 Azure 資源群組中建立 Ubuntu VM
> * 在 VM 上安裝 Elasticsearch、Logstash 和 Kibana
> * 使用 Logstash 將範例資料傳送至 Elasticsearch 
> * 開啟連接埠並處理 Kibana 主控台中的資料


 此部署適用於具有彈性堆疊的基本開發。 如需彈性堆疊的詳細資訊，包括適用於生產環境的建議，請參閱 [Elastic 文件](https://www.elastic.co/guide/index.html) \(英文\) 與 [Azure 架構中心](/azure/architecture/elasticsearch/)。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az_group_create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-virtual-machine"></a>建立虛擬機器

使用 [az vm create](/cli/azure/vm#az_vm_create) 命令來建立 VM。 

下列範例會建立名為 myVM 的 VM，並建立 SSH 金鑰 (如果它們不存在於預設金鑰位置)。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。  

```azurecli-interactive 
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

建立 VM 後，Azure CLI 會顯示類似下列範例的資訊。 記下 `publicIpAddress`。 此位址用來存取 VM。

```azurecli-interactive 
{
  "fqdns": "",
  "id": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```

## <a name="ssh-into-your-vm"></a>透過 SSH 連線到您的 VM

如果您還不知道您 VM 的公用 IP 位址，請執行 [az network public-ip list](/cli/azure/network/public-ip#az_network_public_ip_list) 命令：

```azurecli-interactive
az network public-ip list --resource-group myResourceGroup --query [].ipAddress
```

使用下列命令，建立與虛擬機器的 SSH 工作階段。 替換為您虛擬機器的正確公用 IP 位址。 在此範例中，IP 位址是 *40.68.254.142*。

```bash
ssh azureuser@40.68.254.142
```

## <a name="install-the-elastic-stack"></a>安裝彈性堆疊

匯入 Elasticsearch 簽署金鑰並更新您的 APT 來源清單，以包含 Elastic 套件存放庫：

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
```

在 VM 上安裝 Java 虛擬機器並設定 JAVA_HOME 變數 - 這是執行彈性堆疊元件所需的變數。

```bash
sudo apt update && sudo apt install openjdk-8-jre-headless
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

執行下列命令以更新 Ubuntu 套件來源，並安裝 Elasticsearch、Kibana 和 Logstash。

```bash
sudo apt update && sudo apt install elasticsearch kibana logstash   
```

> [!NOTE]
> 詳細的安裝指示 (包括目錄配置和初始設定) 保留在 [Elastic 文件](https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html) \(英文\) 中

## <a name="start-elasticsearch"></a>啟動 Elasticsearch 

使用下列命令啟動 VM 上的 Elasticsearch：

```bash
sudo systemctl start elasticsearch.service
```

此命令不會產生任何輸出，因此請使用這個 `curl` 命令，確認 Elasticsearch 正在 VM 上執行：

```bash
sudo curl -XGET 'localhost:9200/'
```

如果 Elasticsearch 正在執行，您應該會看到如以下的輸出：

```json
{
  "name" : "w6Z4NwR",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "SDzCajBoSK2EkXmHvJVaDQ",
  "version" : {
    "number" : "5.6.3",
    "build_hash" : "1a2f265",
    "build_date" : "2017-10-06T20:33:39.012Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```

## <a name="start-logstash-and-add-data-to-elasticsearch"></a>啟動 Logstash，並將資料新增至 Elasticsearch

使用下列命令啟動 Logstash：

```bash 
sudo systemctl start logstash.service
```

在互動模式下測試 Logstash ，以確定它正常運作：

```bash
sudo /usr/share/logstash/bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

這是基本的 logstash [管線](https://www.elastic.co/guide/en/logstash/5.6/pipeline.html) \(英文\)，可回應標準輸出的標準輸入。 

```output
The stdin plugin is now waiting for input:
hello azure
2017-10-11T20:01:08.904Z myVM hello azure
```

設定 Logstash 將核心訊息從這個 VM 轉送至 Elasticsearch。 在稱為 `vm-syslog-logstash.conf` 的空白目錄中建立新檔案，然後貼在下列 Logstash 設定中：

```Logstash
input {
    stdin {
        type => "stdin-type"
    }

    file {
        type => "syslog"
        path => [ "/var/log/*.log", "/var/log/*/*.log", "/var/log/messages", "/var/log/syslog" ]
        start_position => "beginning"
    }
}

output {

    stdout {
        codec => rubydebug
    }
    elasticsearch {
        hosts  => "localhost:9200"
    }
}
```

測試這個設定，並將 syslog 資料傳送至 Elasticsearch：

```bash
sudo /usr/share/logstash/bin/logstash -f vm-syslog-logstash.conf
```

當終端機中的 syslog 項目傳送至 Elasticsearch 時，您看到回應。 一旦您傳送一些資料之後，使用 `CTRL+C` 退出 Logstash。

## <a name="start-kibana-and-visualize-the-data-in-elasticsearch"></a>啟動 Kibana，並將 Elasticsearch 中的資料視覺化

編輯 `/etc/kibana/kibana.yml` 並變更 Kibana 接聽的 IP 位址，讓您可以從網頁瀏覽器加以存取。

```bash
server.host:"0.0.0.0"
```

使用下列命令啟動 Kibana：

```bash
sudo systemctl start kibana.service
```

從 Azure CLI 開啟連接埠 5601，以允許從遠端存取 Kibana 主控台：

```azurecli-interactive
az vm open-port --port 5601 --resource-group myResourceGroup --name myVM
```

開啟 Kibana 主控台，然後選取 [建立]，以便根據您稍早傳送至 Elasticsearch 的 syslog 資料，產生預設索引。 

![瀏覽 Kibana 中的 Syslog 事件](media/elasticsearch-install/kibana-index.png)

選取 Kibana 主控台上的 [探索]，以搜尋、瀏覽並篩選 syslog 事件。

![瀏覽 Kibana 中的 Syslog 事件](media/elasticsearch-install/kibana-search-filter.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您將彈性堆疊部署到 Azure 中的開發 VM。 您已了解如何︰

> [!div class="checklist"]
> * 在 Azure 資源群組中建立 Ubuntu VM
> * 在 VM 上安裝 Elasticsearch、Logstash 和 Kibana
> * 將範例資料從 Logstash 傳送至 Elasticsearch 
> * 開啟連接埠並處理 Kibana 主控台中的資料
