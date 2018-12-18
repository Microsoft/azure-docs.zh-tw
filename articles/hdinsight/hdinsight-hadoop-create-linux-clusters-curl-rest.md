---
title: 使用 Azure REST API 建立 Hadoop 叢集 - Azure
description: 了解如何將 Azure Resource Manager 範本提交至 Azure REST API 來建立 HDInsight 叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/02/2018
ms.author: jasonh
ms.openlocfilehash: 6d2e0b06f0402ae26212be45a00b4c9a5394a117
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46983696"
---
# <a name="create-hadoop-clusters-using-the-azure-rest-api"></a>使用 Azure REST API 建立 Hadoop 叢集

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

了解如何使用 Azure Resource Manager 範本和 Azure REST API 來建立 HDInsight 叢集。

Azure REST API 可讓您對裝載於 Azure 平台的服務執行管理作業，包括建立新的資源，例如 HDInsight 叢集。

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

> [!NOTE]
> 本文件的步驟是使用 [curl (https://curl.haxx.se/)](https://curl.haxx.se/) 公用程式來與 Azure REST API 進行通訊。

## <a name="create-a-template"></a>建立範本

Azure Resource Manager 範本是描述**資源群組**與其中所有資源 (例如 HDInsight) 的 JSON 文件。此範本型方法可讓您在一個範本中定義 HDInsight 所需的資源。

以下 JSON 文件是 [https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-linux-ssh-password](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-linux-ssh-password) 提供的範本和參數檔合併工具，它會使用密碼來建立以 Linux 為基礎的叢集，以保護 SSH 使用者帳戶。

   ```json
   {
       "properties": {
           "template": {
               "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
               "contentVersion": "1.0.0.0",
               "parameters": {
                   "clusterType": {
                       "type": "string",
                       "allowedValues": ["hadoop",
                       "hbase",
                       "storm",
                       "spark"],
                       "metadata": {
                           "description": "The type of the HDInsight cluster to create."
                       }
                   },
                   "clusterName": {
                       "type": "string",
                       "metadata": {
                           "description": "The name of the HDInsight cluster to create."
                       }
                   },
                   "clusterLoginUserName": {
                       "type": "string",
                       "metadata": {
                           "description": "These credentials can be used to submit jobs to the cluster and to log into cluster dashboards."
                       }
                   },
                   "clusterLoginPassword": {
                       "type": "securestring",
                       "metadata": {
                           "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
                       }
                   },
                   "sshUserName": {
                       "type": "string",
                       "metadata": {
                           "description": "These credentials can be used to remotely access the cluster."
                       }
                   },
                   "sshPassword": {
                       "type": "securestring",
                       "metadata": {
                           "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
                       }
                   },
                   "clusterStorageAccountName": {
                       "type": "string",
                       "metadata": {
                           "description": "The name of the storage account to be created and be used as the cluster's storage."
                       }
                   },
                   "clusterWorkerNodeCount": {
                       "type": "int",
                       "defaultValue": 4,
                       "metadata": {
                           "description": "The number of nodes in the HDInsight cluster."
                       }
                   }
               },
               "variables": {
                   "defaultApiVersion": "2015-05-01-preview",
                   "clusterApiVersion": "2015-03-01-preview"
               },
               "resources": [{
                   "name": "[parameters('clusterStorageAccountName')]",
                   "type": "Microsoft.Storage/storageAccounts",
                   "location": "[resourceGroup().location]",
                   "apiVersion": "[variables('defaultApiVersion')]",
                   "dependsOn": [],
                   "tags": {

                   },
                   "properties": {
                       "accountType": "Standard_LRS"
                   }
               },
               {
                   "name": "[parameters('clusterName')]",
                   "type": "Microsoft.HDInsight/clusters",
                   "location": "[resourceGroup().location]",
                   "apiVersion": "[variables('clusterApiVersion')]",
                   "dependsOn": ["[concat('Microsoft.Storage/storageAccounts/',parameters('clusterStorageAccountName'))]"],
                   "tags": {

                   },
                   "properties": {
                       "clusterVersion": "3.6",
                       "osType": "Linux",
                       "clusterDefinition": {
                           "kind": "[parameters('clusterType')]",
                           "configurations": {
                               "gateway": {
                                   "restAuthCredential.isEnabled": true,
                                   "restAuthCredential.username": "[parameters('clusterLoginUserName')]",
                                   "restAuthCredential.password": "[parameters('clusterLoginPassword')]"
                               }
                           }
                       },
                       "storageProfile": {
                           "storageaccounts": [{
                               "name": "[concat(parameters('clusterStorageAccountName'),'.blob.core.windows.net')]",
                               "isDefault": true,
                               "container": "[parameters('clusterName')]",
                               "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('clusterStorageAccountName')), variables('defaultApiVersion')).key1]"
                           }]
                       },
                       "computeProfile": {
                           "roles": [{
                               "name": "headnode",
                               "targetInstanceCount": "2",
                               "hardwareProfile": {
                                   "vmSize": "Standard_D3"
                               },
                               "osProfile": {
                                   "linuxOperatingSystemProfile": {
                                       "username": "[parameters('sshUserName')]",
                                       "password": "[parameters('sshPassword')]"
                                   }
                               }
                           },
                           {
                               "name": "workernode",
                               "targetInstanceCount": "[parameters('clusterWorkerNodeCount')]",
                               "hardwareProfile": {
                                   "vmSize": "Standard_D3"
                               },
                               "osProfile": {
                                   "linuxOperatingSystemProfile": {
                                       "username": "[parameters('sshUserName')]",
                                       "password": "[parameters('sshPassword')]"
                                   }
                               }
                           }]
                       }
                   }
               }],
               "outputs": {
                   "cluster": {
                       "type": "object",
                       "value": "[reference(resourceId('Microsoft.HDInsight/clusters',parameters('clusterName')))]"
                   }
               }
           },
           "mode": "incremental",
           "Parameters": {
               "clusterName": {
                   "value": "newclustername"
               },
               "clusterType": {
                   "value": "hadoop"
               },
               "clusterStorageAccountName": {
                   "value": "newstoragename"
               },
               "clusterLoginUserName": {
                   "value": "admin"
               },
               "clusterLoginPassword": {
                   "value": "changeme"
               },
               "sshUserName": {
                   "value": "sshuser"
               },
               "sshPassword": {
                   "value": "changeme"
               }
           }
       }
   }
   ```

本文件中的步驟引用此範例。 使用您叢集的值來取代 **Parameters** 區段中的範例值。

> [!IMPORTANT]
> 本範本使用 HDInsight 叢集的背景工作節點預設數目 (4)。 如果您規劃 32 個以上的背景工作節點，則必須選取具有至少 8 個核心和 14 GB RAM 的前端節點大小。
>
> 如需節點大小和相關成本的詳細資訊，請參閱 [HDInsight 定價](https://azure.microsoft.com/pricing/details/hdinsight/)。

## <a name="log-in-to-your-azure-subscription"></a>登入您的 Azure 訂用帳戶

請依照[開始使用 Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2) 所述的步驟操作，並使用 `az login` 命令連接到您的訂用帳戶。

## <a name="create-a-service-principal"></a>建立服務主體

> [!NOTE]
> 以下步驟是[使用 Azure CLI 建立用來存取資源的服務主體](../azure-resource-manager/resource-group-authenticate-service-principal-cli.md)文件中*使用密碼建立服務主體*一節的簡易版。 這些步驟會建立用來驗證 Azure REST API 的服務主體。

1. 從命令列中，使用下列命令列出您的 Azure 訂用帳戶。

   ```bash
   az account list --query '[].{Subscription_ID:id,Tenant_ID:tenantId,Name:name}'  --output table
   ```

    在清單中，選取您要使用的訂用帳戶，並記下 **Subscription_ID** 和 __Tenant_ID__ 資料行。 儲存這些值。

2. 使用以下命令，在 Azure Active Directory 中建立應用程式。

   ```bash
   az ad app create --display-name "exampleapp" --homepage "https://www.contoso.org" --identifier-uris "https://www.contoso.org/example" --password <Your password> --query 'appId'
   ```

    將 `--display-name`、`--homepage`、`--identifier-uris` 的值取代為您自己的值。 為新的 Active Directory 項目提供密碼。

   > [!NOTE]
   > `--home-page` 和 `--identifier-uris` 值不需參考裝載於網際網路上的實際網頁。 它們必須是唯一的 URI。

   此命令傳回的值是新應用程式的 __App ID__。 儲存這個值。

3. 使用下列命令以 **App ID** 建立服務主體。

   ```bash
   az ad sp create --id <App ID> --query 'objectId'
   ```

     此命令傳回的值是 __Object ID__。 儲存這個值。

4. 使用 **Object ID** 值將 **Owner** 角色指派給服務主體。 使用稍早取得的**訂用帳戶 ID** 。

   ```bash
   az role assignment create --assignee <Object ID> --role Owner --scope /subscriptions/<Subscription ID>/
   ```

## <a name="get-an-authentication-token"></a>取得驗證權杖

使用下列命令來擷取驗證權杖︰

```bash
curl -X "POST" "https://login.microsoftonline.com/$TENANTID/oauth2/token" \
-H "Cookie: flight-uxoptin=true; stsservicecookie=ests; x-ms-gateway-slice=productionb; stsservicecookie=ests" \
-H "Content-Type: application/x-www-form-urlencoded" \
--data-urlencode "client_id=$APPID" \
--data-urlencode "grant_type=client_credentials" \
--data-urlencode "client_secret=$PASSWORD" \
--data-urlencode "resource=https://management.azure.com/"
```

將 `$TENANTID`、`$APPID` 和 `$PASSWORD` 設定為先前取得或使用的值。

如果這個要求成功，您會收到 200 系列的回應，且回應本文會包含 JSON 文件。

這項要求所傳回的 JSON 文件包含名為 **access_token** 的元素。 **access_token** 的值是用來驗證對 REST API 的要求。

```json
{
    "token_type":"Bearer",
    "expires_in":"3599",
    "expires_on":"1463409994",
    "not_before":"1463406094",
    "resource":"https://management.azure.com/","access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWoNBVGZNNXBPWWlKSE1iYTlnb0VLWSIsImtpZCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSJ9.eyJhdWQiOiJodHRwczovL21hbmFnZW1lbnQuYXp1cmUuY29tLyIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI2Ny8iLCJpYXQiOjE0NjM0MDYwOTQsIm5iZiI6MTQ2MzQwNjA5NCwiZXhwIjoxNDYzNDA5OTk5LCJhcHBpZCI6IjBlYzcyMzM0LTZkMDMtNDhmYi04OWU1LTU2NTJiODBiZDliYiIsImFwcGlkYWNyIjoiMSIsImlkcCI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0Ny8iLCJvaWQiOiJlNjgxZTZiMi1mZThkLTRkZGUtYjZiMS0xNjAyZDQyNWQzOWYiLCJzdWIiOiJlNjgxZTZiMi1mZThkLTRkZGUtYjZiMS0xNjAyZDQyNWQzOWYiLCJ0aWQiOiI3MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDciLCJ2ZXIiOiIxLjAifQ.nJVERbeDHLGHn7ZsbVGBJyHOu2PYhG5dji6F63gu8XN2Cvol3J1HO1uB4H3nCSt9DTu_jMHqAur_NNyobgNM21GojbEZAvd0I9NY0UDumBEvDZfMKneqp7a_cgAU7IYRcTPneSxbD6wo-8gIgfN9KDql98b0uEzixIVIWra2Q1bUUYETYqyaJNdS4RUmlJKNNpENllAyHQLv7hXnap1IuzP-f5CNIbbj9UgXxLiOtW5JhUAwWLZ3-WMhNRpUO2SIB7W7tQ0AbjXw3aUYr7el066J51z5tC1AK9UC-mD_fO_HUP6ZmPzu5gLA6DxkIIYP3grPnRVoUDltHQvwgONDOw"
}
```

## <a name="create-a-resource-group"></a>建立資源群組

使用下列命令來建立資源群組。

* 將 `$SUBSCRIPTIONID` 設定為建立服務主體時所收到的訂用帳戶識別碼。
* 將 `$ACCESSTOKEN` 設定為上一個步驟中所收到的存取權杖。
* 將 `DATACENTERLOCATION` 取代為您要在其中建立資源群組和資源的資料中心。 例如 'South Central US'。
* 將 `$RESOURCEGROUPNAME` 設定為您想要用於此群組的名稱︰

```bash
curl -X "PUT" "https://management.azure.com/subscriptions/$SUBSCRIPTIONID/resourcegroups/$RESOURCEGROUPNAME?api-version=2015-01-01" \
    -H "Authorization: Bearer $ACCESSTOKEN" \
    -H "Content-Type: application/json" \
    -d $'{
"location": "DATACENTERLOCATION"
}'
```

如果這個要求成功，您會收到 200 系列的回應，且回應主體會包含 JSON 文件，內含群組的相關資訊。 `"provisioningState"` 項目包含的值為 `"Succeeded"`。

## <a name="create-a-deployment"></a>建立部署

使用下列命令，將範本部署到資源群組。

* 將 `$DEPLOYMENTNAME` 設定為您想要用於此部署的名稱。

```bash
curl -X "PUT" "https://management.azure.com/subscriptions/$SUBSCRIPTIONID/resourcegroups/$RESOURCEGROUPNAME/providers/microsoft.resources/deployments/$DEPLOYMENTNAME?api-version=2015-01-01" \
-H "Authorization: Bearer $ACCESSTOKEN" \
-H "Content-Type: application/json" \
-d "{set your body string to the template and parameters}"
```

> [!NOTE]
> 如果您已將範本儲存到檔案，則可使用下列命令，而不是 `-d "{ template and parameters}"`：
>
> `--data-binary "@/path/to/file.json"`

如果這個要求成功，您會收到 200 系列的回應，且回應主體會包含 JSON 文件，內含部署作業的相關資訊。

> [!IMPORTANT]
> 部署已送出，但尚未完成。 部署通常需要大約 15 分鐘才會完成。

## <a name="check-the-status-of-a-deployment"></a>檢查部署的狀態

若要檢查部署的狀態，請使用下列命令：

```bash
curl -X "GET" "https://management.azure.com/subscriptions/$SUBSCRIPTIONID/resourcegroups/$RESOURCEGROUPNAME/providers/microsoft.resources/deployments/$DEPLOYMENTNAME?api-version=2015-01-01" \
-H "Authorization: Bearer $ACCESSTOKEN" \
-H "Content-Type: application/json"
```

這個命令會傳回包含部署作業相關資訊的 JSON 文件。 `"provisioningState"` 項目包含部署的狀態。 如果此元素包含 `"Succeeded"` 的值，則部署已順利完成。

## <a name="troubleshoot"></a>疑難排解

如果您在建立 HDInsight 叢集時遇到問題，請參閱[存取控制需求](hdinsight-administer-use-portal-linux.md#create-clusters)。

## <a name="next-steps"></a>後續步驟

既然您已成功建立 HDInsight 叢集，請使用下列內容來了解如何使用您的叢集。

### <a name="hadoop-clusters"></a>Hadoop 叢集

* [〈搭配 HDInsight 使用 Hivet〉](hadoop/hdinsight-use-hive.md)
* [搭配 HDInsight 使用 Pig](hadoop/hdinsight-use-pig.md)
* [〈搭配 HDInsight 使用 MapReduce〉](hadoop/hdinsight-use-mapreduce.md)

### <a name="hbase-clusters"></a>HBase 叢集

* [開始在 HDInsight 上使用 HBase](hbase/apache-hbase-tutorial-get-started-linux.md)
* [在 HDInsight 上開發適用於 HBase 的 Java 應用程式](hbase/apache-hbase-build-java-maven-linux.md)

### <a name="storm-clusters"></a>Storm 叢集

* [在 HDInsight 上開發適用於 Storm 的 Java 拓撲](storm/apache-storm-develop-java-topology.md)
* [在 HDInsight 上的 Storm 中使用 Python 元件](storm/apache-storm-develop-python-topology.md)
* [在 HDInsight 上使用 Storm 部署和監視拓撲](storm/apache-storm-deploy-monitor-topology-linux.md)
