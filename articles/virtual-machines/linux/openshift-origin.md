---
title: 在 Azure 中部署 OKD | Microsoft Docs
description: 在 Azure 中部署 OKD。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: haroldw
manager: najoshi
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: ''
ms.author: haroldw
ms.openlocfilehash: 75a02e61adf3e5477b9945afc778e867d5d9c88c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958130"
---
# <a name="deploy-okd-in-azure"></a>在 Azure 中部署 OKD

您可使用這兩種方式之一在 Azure 中部署 OKD (之前稱為 OpenShift Origin)：

- 您可以手動部署所需的所有 Azure 基礎結構元件，然後依照 OKD [文件](https://docs.okd.io/3.10/welcome/index.html) \(英文\) 進行。
- 您也可以使用能夠簡化 OKD 叢集部署的現有 [Resource Manager 範本](https://github.com/Microsoft/openshift-origin)。

## <a name="deploy-by-using-the-okd-template"></a>使用 OKD 範本部署

使用您稍早針對 `aadClientId` 參數所建立之服務主體中的 `appId` 值。

下列範例會建立名為 azuredeploy.parameters.json，且具有所有必要輸入的參數檔案。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "masterVmSize": {
            "value": "Standard_E2s_v3"
        },
        "infraVmSize": {
            "value": "Standard_E2s_v3"
        },
        "nodeVmSize": {
            "value": "Standard_E2s_v3"
        },
        "openshiftClusterPrefix": {
            "value": "mycluster"
        },
        "masterInstanceCount": {
            "value": 3
        },
        "infraInstanceCount": {
            "value": 2
        },
        "nodeInstanceCount": {
            "value": 2
        },
        "dataDiskSize": {
            "value": 128
        },
        "adminUsername": {
            "value": "clusteradmin"
        },
        "openshiftPassword": {
            "value": "{Strong Password}"
        },
        "sshPublicKey": {
            "value": "{SSH Public Key}"
        },
        "keyVaultResourceGroup": {
            "value": "keyvaultrg"
        },
        "keyVaultName": {
            "value": "keyvault"
        },
        "keyVaultSecret": {
            "value": "keysecret"
        },
        "aadClientId": {
            "value": "11111111-abcd-1234-efgh-111111111111"
        },
        "aadClientSecret": {
            "value": "{Strong Password}"
        },
        "defaultSubDomainType": {
            "value": "nipio"
        }
    }
}
```

### <a name="deploy-by-using-azure-cli"></a>使用 Azure CLI 進行部署


> [!NOTE] 
> 下列命令需要 Azure CLI.8 或更新版本。 您可以使用 `az --version` 命令來確認 CLI 版本。 若要更新 CLI 版本，請參閱[安裝 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。

下列範例會使用 myOpenShiftCluster 的部署名稱，將 OKD 叢集和所有相關的資源部署到名稱為 myResourceGroup 的資源群組。 範本使用名為 azuredeploy.parameters.json 的本機參數檔案，直接參考自 Github 存放庫。

```azurecli 
az group deployment create -g myResourceGroup --name myOpenShiftCluster \
      --template-uri https://raw.githubusercontent.com/Microsoft/openshift-origin/master/azuredeploy.json \
      --parameters @./azuredeploy.parameters.json
```

根據部署的節點總數，部署需要至少 25 分鐘才能完成。 當部署完成時，OKD 主控台的 URL 和 OpenShift 主機的 DNS 名稱會列印到終端機。

```json
{
  "OpenShift Console Uri": "http://openshiftlb.cloudapp.azure.com:8443/console",
  "OpenShift Master SSH": "ssh clusteradmin@myopenshiftmaster.cloudapp.azure.com -p 2200"
}
```

## <a name="connect-to-the-okd-cluster"></a>連線到 OKD 叢集

部署完成時，使用 `OpenShift Console Uri` 可透過瀏覽器連線至 OKD 主控台。 或者，您可以使用下列命令，連線至 OKD 主機：

```bash
$ ssh -p 2200 clusteradmin@myopenshiftmaster.cloudapp.azure.com
```

## <a name="clean-up-resources"></a>清除資源

當不再需要資源時，您可以使用 [az group delete](/cli/azure/group#az_group_delete) 命令，移除資源群組、OpenShift 叢集和所有相關資源。

```azurecli 
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

- [部署後工作](./openshift-post-deployment.md)
- [針對 OpenShift 部署進行疑難排解](./openshift-troubleshooting.md)
- [開始使用 OKD](https://docs.okd.io/latest/getting_started/index.html)
