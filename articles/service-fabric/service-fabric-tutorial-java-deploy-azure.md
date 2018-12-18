---
title: 在 Azure 中將 Java 應用程式部署到 Service Fabric 叢集 | Microsoft Docs
description: 在本教學課程中，了解如何將 Java Service Fabric 應用程式部署到 Azure Service Fabric 叢集。
services: service-fabric
documentationcenter: java
author: suhuruli
manager: msfussell
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: java
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/26/2018
ms.author: suhuruli
ms.custom: mvc
ms.openlocfilehash: 53ad780ff0b199764c354327439fa69c360e7cb5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996803"
---
# <a name="tutorial-deploy-a-java-application-to-a-service-fabric-cluster-in-azure"></a>教學課程：將 Java 應用程式部署到 Azure 中的 Service Fabric 叢集

本教學課程是系列中的第三部分，示範如何將 Service Fabric 應用程式部署到 Azure 中的叢集。

在系列的第三部分中，您將了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立安全的 Linux 叢集
> * 將應用程式部署到叢集

在本教學課程系列中，您將了解如何：

> [!div class="checklist"]
> * [建置 Java Service Fabric Reliable Services 應用程式](service-fabric-tutorial-create-java-app.md)
> * [在本機叢集上部署及偵錯應用程式](service-fabric-tutorial-debug-log-local-cluster.md)
> * 將應用程式部署至 Azure 叢集
> * [設定應用程式的監視和診斷](service-fabric-tutorial-java-elk.md)
> * [設定 CI/CD](service-fabric-tutorial-java-jenkins.md)

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [安裝 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)
* 安裝適用於 [Mac](service-fabric-get-started-mac.md) 或 [Linux](service-fabric-get-started-linux.md) 的 Service Fabric SDK
* [安裝 Python 3](https://wiki.python.org/moin/BeginnersGuide/Download)

## <a name="create-a-service-fabric-cluster-in-azure"></a>在 Azure 中建立 Service Fabric 叢集

下列步驟會建立將應用程式部署至 Service Fabric 叢集所需的資源。 而且會設定使用 ELK (Elasticsearch、Logstash、Kibana) 堆疊監視解決方案健康狀態所需的資源。 具體來說，[事件中樞](https://azure.microsoft.com/services/event-hubs/) 可作為 Service Fabric 記錄的接收端。 它已設定為將 Service Fabric 叢集的記錄傳送至您的 Logstash 執行個體。

1. 開啟終端機並下載下列套件，其中包含在 Azure 中建立資源所需的協助程式指令碼和範本

    ```bash
    git clone https://github.com/Azure-Samples/service-fabric-java-quickstart.git
    ```

2. 登入您的 Azure 帳戶

    ```bash
    az login
    ```

3. 設定您想用來建立資源的 Azure 訂用帳戶

    ```bash
    az account set --subscription [SUBSCRIPTION-ID]
    ```

4. 從 *service-fabric-java-quickstart/AzureCluster* 資料夾，執行下列命令以在金鑰保存庫中建立叢集憑證。 此憑證用來保護您的 Service Fabric 叢集。 提供區域 (必須與您的 Service Fabric 叢集相同)、金鑰保存庫的資源群組名稱、金鑰保存庫名稱、憑證密碼，以及叢集 DNS 名稱。

    ```bash
    ./new-service-fabric-cluster-certificate.sh [REGION] [KEY-VAULT-RESOURCE-GROUP] [KEY-VAULT-NAME] [CERTIFICATE-PASSWORD] [CLUSTER-DNS-NAME-FOR-CERTIFICATE]

    Example: ./new-service-fabric-cluster-certificate.sh 'westus' 'testkeyvaultrg' 'testkeyvault' '<password>' 'testservicefabric.westus.cloudapp.azure.com'
    ```

    上述命令會傳回下列資訊，應記下該資訊以便稍後使用。

    ```
    Source Vault Resource Id: /subscriptions/<subscription_id>/resourceGroups/testkeyvaultrg/providers/Microsoft.KeyVault/vaults/<name>
    Certificate URL: https://<name>.vault.azure.net/secrets/<cluster-dns-name-for-certificate>/<guid>
    Certificate Thumbprint: <THUMBPRINT>
    ```

5. 為儲存記錄的儲存體帳戶建立資源群組

    ```bash
    az group create --location [REGION] --name [RESOURCE-GROUP-NAME]

    Example: az group create --location westus --name teststorageaccountrg
    ```

6. 建立儲存體帳戶，該帳戶將用於儲存即將產生的記錄

    ```bash
    az storage account create -g [RESOURCE-GROUP-NAME] -l [REGION] --name [STORAGE-ACCOUNT-NAME] --kind Storage

    Example: az storage account create -g teststorageaccountrg -l westus --name teststorageaccount --kind Storage
    ```

7. 存取 [Azure 入口網站](https://portal.azure.com)並瀏覽至儲存體帳戶的 [共用存取簽章] 索引標籤。 產生 SAS 權杖，如下所示。

    ![產生儲存體的 SAS](./media/service-fabric-tutorial-java-deploy-azure/storagesas.png)

8. 複製帳戶 SAS URL 並保留起來，以便在建立 Service Fabric 叢集時使用。 它類似下列 URL：

    ```
    ?sv=2017-04-17&ss=bfqt&srt=sco&sp=rwdlacup&se=2018-01-31T03:24:04Z&st=2018-01-30T19:24:04Z&spr=https,http&sig=IrkO1bVQCHcaKaTiJ5gilLSC5Wxtghu%2FJAeeY5HR%2BPU%3D
    ```

9. 建立包含事件中樞資源的資源群組。 事件中樞用來從 Service Fabric 傳送訊息至執行 ELK 資源的伺服器。

    ```bash
    az group create --location [REGION] --name [RESOURCE-GROUP-NAME]

    Example: az group create --location westus --name testeventhubsrg
    ```

10. 使用下列命令建立事件中樞資源。 依照提示輸入 namespaceName、eventHubName、consumerGroupName、sendAuthorizationRule 和 receiveAuthorizationRule 的詳細資料。

    ```bash
    az group deployment create -g [RESOURCE-GROUP-NAME] --template-file eventhubsdeploy.json

    Example:
    az group deployment create -g testeventhubsrg --template-file eventhubsdeploy.json
    Please provide string value for 'namespaceName' (? for help): testeventhubnamespace
    Please provide string value for 'eventHubName' (? for help): testeventhub
    Please provide string value for 'consumerGroupName' (? for help): testeventhubconsumergroup
    Please provide string value for 'sendAuthorizationRuleName' (? for help): sender
    Please provide string value for 'receiveAuthorizationRuleName' (? for help): receiver
    ```

    在前一個命令的 JSON 輸出中複製 **output** 欄位的內容。 建立 Service Fabric 叢集時會使用寄件者資訊。 如果已將 Logstash 服務設定成從事件中樞接收訊息，則應儲存收件者名稱和金鑰，以便使用於下一個教學課程。 以下 Blob 是 JSON 輸出範例：

    ```json
    "outputs": {
        "receiver Key": {
            "type": "String",
            "value": "[KEY]"
        },
        "receiver Name": {
            "type": "String",
            "value": "receiver"
        },
        "sender Key": {
            "type": "String",
            "value": "[KEY]"
        },
        "sender Name": {
            "type": "String",
            "value": "sender"
        }
    }
    ```

11. 執行 eventhubssastoken.py 指令碼，為您建立的 EventHubs 資源產生 SAS url。 Service Fabric 叢集會使用 SAS URL 將記錄傳送至事件中樞。 因此，**寄件者**原則用來產生 URL。 指令碼會針對下一個步驟中使用的事件中樞資源傳回 SAS URL：

    ```python
    python3 eventhubssastoken.py 'testeventhubs' 'testeventhubs' 'sender' '[PRIMARY-KEY]'
    ```

    在傳回的 JSON 中複製 **sr** 欄位的值。 **sr** 欄位值是 EventHubs 的 SAS 權杖。 以下 URL 是 **sr** 欄位的範例：

    ```bash
    https%3A%2F%testeventhub.servicebus.windows.net%testeventhub&sig=7AlFYnbvEm%2Bat8ALi54JqHU4i6imoFxkjKHS0zI8z8I%3D&se=1517354876&skn=sender
    ```

    EventHubs 的 SAS URL 會遵循以下結構： https://<namespacename>.servicebus.windows.net/<eventhubsname>?sr=<sastoken>。 例如， https://testeventhubnamespace.servicebus.windows.net/testeventhub?sr=https%3A%2F%testeventhub.servicebus.windows.net%testeventhub&sig=7AlFYnbvEm%2Bat8ALi54JqHU4i6imoFxkjKHS0zI8z8I%3D&se=1517354876&skn=sender

12. 開啟 sfdeploy.parameters.json 檔案，並取代從先前的步驟中取得的下列內容。 步驟 8 中註明了 [SAS-URL-STORAGE-ACCOUNT]。 步驟 11 中註明了 [SAS-URL-EVENT-HUBS]。

    ```json
    "applicationDiagnosticsStorageAccountName": {
        "value": "teststorageaccount"
    },
    "applicationDiagnosticsStorageAccountSasToken": {
        "value": "[SAS-URL-STORAGE-ACCOUNT]"
    },
    "loggingEventHubSAS": {
        "value": "[SAS-URL-EVENT-HUBS]"
    }
    ```

13. 開啟 **sfdeploy.parameters.json**。 變更下列參數，然後儲存檔案。
    - **clusterName**。 請一律使用小寫字母和數字。
    - **adminUserName** (設為非空白的值)
    - **adminPassword** (設為非空白的值)

14. 執行下列命令以建立 Service Fabric 叢集

    ```bash
    az sf cluster create --location 'westus' --resource-group 'testlinux' --template-file sfdeploy.json --parameter-file sfdeploy.parameters.json --secret-identifier <certificate_url_from_step4>
    ```

## <a name="deploy-your-application-to-the-cluster"></a>將應用程式部署到叢集

1. 在部署您的應用程式之前，您必須將下列程式碼片段新增至 Voting/VotingApplication/ApplicationManifest.xml 檔案。 **X509FindValue** 欄位是從「在 Azure 中建立 Service Fabric 叢集」一節的步驟 4 傳回的指紋。 此程式碼片段會以巢狀形式存在 **Secretscertificate** 欄位 (根欄位) 之下。

    ```xml
    <Certificates>
          <SecretsCertificate X509FindType="FindByThumbprint" X509FindValue="[CERTIFICATE-THUMBPRINT]" />
    </Certificates>
    ```

2. 若要將您的應用程式部署至此叢集，您必須使用 SFCTL 建立叢集的連線。 SFCTL 需要同時具有公用和私密金鑰的 PEM 檔案，才能連線到叢集。 執行下列命令，以產生同時具有公用和私密金鑰的 PEM 檔案。 

    ```bash
    openssl pkcs12 -in <clustername>.<region>.cloudapp.azure.com.pfx -out sfctlconnection.pem -nodes -passin pass:<password>
    ```

3. 執行下列命令以連線至叢集。

    ```bash
    sfctl cluster select --endpoint https://<clustername>.<region>.cloudapp.azure.com:19080 --pem sfctlconnection.pem --no-verify
    ```

4. 若要部署您的應用程式，請瀏覽至 Voting/Scripts 資料夾並執行 **install.sh** 指令碼。

    ```bash
    ./install.sh
    ```

5. 若要存取 Service Fabric 總管，請開啟您慣用的瀏覽器並輸入 https://testlinuxcluster.westus.cloudapp.azure.com:19080。 從憑證存放區選擇您要用來連線到此端點的憑證。 如果您使用 Linux 機器，new-service-fabric-cluster-certificate.sh 指令碼所產生的憑證必須匯入 Chrome 中，才能檢視 Service Fabric 總管。 如果您使用 Mac，則必須將 PFX 檔案安裝到您的金鑰鏈。 您會注意您的應用程式已安裝於叢集上。

    ![SFX Java Azure](./media/service-fabric-tutorial-java-deploy-azure/sfxjavaonazure.png)

6. 若要存取您的應用程式，請輸入 https://testlinuxcluster.westus.cloudapp.azure.com:8080

    ![Voting 應用程式 Java Azure](./media/service-fabric-tutorial-java-deploy-azure/votingappjavaazure.png)

7. 若要從叢集將您的應用程式解除安裝，請執行 **Scripts** 資料夾中的 *uninstall.sh* 指令碼。

    ```bash
    ./uninstall.sh
    ```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立安全的 Linux 叢集
> * 使用 ELK 建立進行監視所需的資源
> * 選擇性：如何使用合作對象叢集來試用 Service Fabric

前進到下一個教學課程：
> [!div class="nextstepaction"]
> [設定監視和診斷](service-fabric-tutorial-java-elk.md)