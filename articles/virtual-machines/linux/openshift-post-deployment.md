---
title: Azure 上的 OpenShift 部署後工作 | Microsoft Docs
description: 在部署 OpenShift 叢集之後的其他工作。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: haroldwongms
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
ms.openlocfilehash: d400512c2e96e0e24bbf965b2e201adf92ccbb0f
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434886"
---
# <a name="post-deployment-tasks"></a>部署後工作

部署 OpenShift 叢集之後，您可以設定其他項目。 本文涵蓋下列內容：

- 如何使用 Azure Active Directory (Azure AD) 設定單一登入
- 如何設定 Log Analytics 來監視 OpenShift
- 如何設定計量與記錄

## <a name="configure-single-sign-on-by-using-azure-active-directory"></a>使用 Azure Active Directory 設定單一登入

若要用 Azure Active Directory 進行驗證，請先建立 Azure AD 的應用程式註冊。 此程序牽涉兩個步驟：建立應用程式註冊，以及設定權限。

### <a name="create-an-app-registration"></a>建立應用程式註冊

這些步驟使用 Azure CLI 建立應用程式註冊，以及使用 GUI (入口網站) 設定權限。 若要建立應用程式註冊，您需要下列五項資訊：

- 顯示名稱：應用程式註冊名稱 (例如 OCPAzureAD)
- 首頁：OpenShift 主控台 URL (例如， https://masterdns343khhde.westus.cloudapp.azure.com:8443/console)
- 識別碼 URI：OpenShift 主控台 URL (例如， https://masterdns343khhde.westus.cloudapp.azure.com:8443/console)
- 回覆 URL：主要公用 URL 及應用程式註冊名稱 (例如， https://masterdns343khhde.westus.cloudapp.azure.com/oauth2callback/OCPAzureAD)
- 密碼：安全密碼 (使用強式密碼)

下列範例將會使用上述資訊建立應用程式註冊：

```azurecli
az ad app create --display-name OCPAzureAD --homepage https://masterdns343khhde.westus.cloudapp.azure.com:8443/console --reply-urls https://masterdns343khhde.westus.cloudapp.azure.com/oauth2callback/OCPAzureAD --identifier-uris https://masterdns343khhde.westus.cloudapp.azure.com:8443/console --password {Strong Password}
```

如果命令成功，您會獲得類似以下的 JSON 輸出：

```json
{
  "appId": "12345678-ca3c-427b-9a04-ab12345cd678",
  "appPermissions": null,
  "availableToOtherTenants": false,
  "displayName": "OCPAzureAD",
  "homepage": "https://masterdns343khhde.westus.cloudapp.azure.com:8443/console",
  "identifierUris": [
    "https://masterdns343khhde.westus.cloudapp.azure.com:8443/console"
  ],
  "objectId": "62cd74c9-42bb-4b9f-b2b5-b6ee88991c80",
  "objectType": "Application",
  "replyUrls": [
    "https://masterdns343khhde.westus.cloudapp.azure.com/oauth2callback/OCPAzureAD"
  ]
}
```

記下命令在之後的步驟傳回的 appId 屬性。

在 Azure 入口網站中：

1.  選取 [Azure Active Directory]  >  [應用程式註冊]。
2.  搜尋您的應用程式註冊 (例如 OCPAzureAD)。
3.  在結果中，按一下 [應用程式註冊]。
4.  在 [設定] 下，選取 [必要權限]。
5.  在 [必要權限] 下，選取 [新增]。

  ![App 註冊](media/openshift-post-deployment/app-registration.png)

6.  按一下「步驟 1：選取 API」，然後按一下 [Windows Azure Active Directory (Microsoft.Azure.ActiveDirectory)]。 按一下底部的 [選取]。

  ![應用程式註冊選取 API](media/openshift-post-deployment/app-registration-select-api.png)

7.  在「步驟 2：選取權限」，選取 [委派的權限] 底下的 [登入及讀取使用者個人檔案]，然後按一下 [選取]。

  ![應用程式註冊存取](media/openshift-post-deployment/app-registration-access.png)

8.  選取 [完成] 。

### <a name="configure-openshift-for-azure-ad-authentication"></a>設定 OpenShift 進行 Azure AD 驗證

若要將 OpenShift 設定為使用 Azure AD 作為驗證提供者，必須在所有主要節點上編輯 /etc/origin/master/master-config.yaml 檔案。

使用下列 CLI 命令找到租用戶識別碼：

```azurecli
az account show
```

在 yaml 檔案中，尋找下列幾行：

```yaml
oauthConfig:
  assetPublicURL: https://masterdns343khhde.westus.cloudapp.azure.com:8443/console/
  grantConfig:
    method: auto
  identityProviders:
  - challenge: true
    login: true
    mappingMethod: claim
    name: htpasswd_auth
    provider:
      apiVersion: v1
      file: /etc/origin/master/htpasswd
      kind: HTPasswdPasswordIdentityProvider
```

緊接在上述幾行之後，插入下列這幾行：

```yaml
  - name: <App Registration Name>
    challenge: false
    login: true
    mappingMethod: claim
    provider:
      apiVersion: v1
      kind: OpenIDIdentityProvider
      clientID: <appId>
      clientSecret: <Strong Password>
      claims:
        id:
        - sub
        preferredUsername:
        - unique_name
        name:
        - name
        email:
        - email
      urls:
        authorize: https://login.microsoftonline.com/<tenant Id>/oauth2/authorize
        token: https://login.microsoftonline.com/<tenant Id>/oauth2/token
```

使用下列 CLI 命令找到租用戶識別碼：```az account show```

重新啟動所有主要節點上的 OpenShift 主要服務：

**OpenShift Origin**

```bash
sudo systemctl restart origin-master-api
sudo systemctl restart origin-master-controllers
```

**具有多個主機的 OpenShift 容器平台 (OCP)**

```bash
sudo systemctl restart atomic-openshift-master-api
sudo systemctl restart atomic-openshift-master-controllers
```

**具有單一主機的 OpenShift 容器平台**

```bash
sudo systemctl restart atomic-openshift-master
```

在 OpenShift 主控台中，您現在會看到兩個用於驗證的選項：htpasswd_auth 和 [應用程式註冊]。

## <a name="monitor-openshift-with-log-analytics"></a>使用 Log Analytics 監視 OpenShift

若要用 Log Analytics 監視 OpenShift，可以使用這兩個選項之一：VM 主機上的 OMS Agent 安裝，或 OMS 容器。 本文提供部署 OMS 容器的相關指示。

## <a name="create-an-openshift-project-for-log-analytics-and-set-user-access"></a>建立適用於 Log Analytics 的 OpenShift 專案，然後設定使用者存取權

```bash
oadm new-project omslogging --node-selector='zone=default'
oc project omslogging
oc create serviceaccount omsagent
oadm policy add-cluster-role-to-user cluster-reader system:serviceaccount:omslogging:omsagent
oadm policy add-scc-to-user privileged system:serviceaccount:omslogging:omsagent
```

## <a name="create-a-daemon-set-yaml-file"></a>建立精靈集 yaml 檔案

建立名為 ocp-omsagent.yml 的檔案：

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: oms
spec:
  selector:
    matchLabels:
      name: omsagent
  template:
    metadata:
      labels:
        name: omsagent
        agentVersion: 1.4.0-45
        dockerProviderVersion: 10.0.0-25
    spec:
      nodeSelector:
        zone: default
      serviceAccount: omsagent
      containers:
      - image: "microsoft/oms"
        imagePullPolicy: Always
        name: omsagent
        securityContext:
          privileged: true
        ports:
        - containerPort: 25225
          protocol: TCP
        - containerPort: 25224
          protocol: UDP
        volumeMounts:
        - mountPath: /var/run/docker.sock
          name: docker-sock
        - mountPath: /etc/omsagent-secret
          name: omsagent-secret
          readOnly: true
        livenessProbe:
          exec:
            command:
              - /bin/bash
              - -c
              - ps -ef | grep omsagent | grep -v "grep"
          initialDelaySeconds: 60
          periodSeconds: 60
      volumes:
      - name: docker-sock
        hostPath:
          path: /var/run/docker.sock
      - name: omsagent-secret
        secret:
         secretName: omsagent-secret
````

## <a name="create-a-secret-yaml-file"></a>建立祕密 yaml 檔案

若要建立祕密 yaml 檔案，將需要兩項資訊：Log Analytics 工作區識別碼、Log Analytics 工作區共用金鑰。 

ocp-secret.yml 檔案範例： 

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: omsagent-secret
data:
  WSID: wsid_data
  KEY: key_data
```

將 wsid_data 取代為 Base64 編碼的 Log Analytics 工作區識別碼。 將 key_data 取代為 Base64 編碼的 Log Analytics 工作區識別碼共用金鑰。

```bash
wsid_data='11111111-abcd-1111-abcd-111111111111'
key_data='My Strong Password'
echo $wsid_data | base64 | tr -d '\n'
echo $key_data | base64 | tr -d '\n'
```

## <a name="create-the-secret-and-daemon-set"></a>建立祕密與精靈集

部署祕密檔案：

```bash
oc create -f ocp-secret.yml
```

部署 OMS Agent 精靈集：

```bash
oc create -f ocp-omsagent.yml
```

## <a name="configure-metrics-and-logging"></a>設定計量與記錄

適用於 OpenShift 容器平台的 Azure Resource Manager 範本可提供輸入參數，用於啟用計量與記錄。 OpenShift 容器平台服務 Marketplace 供應項目有此功能，OpenShift Origin Resource Manager 範本則沒有。

如果您使用 OCP Resource Manager 範本，安裝時不會啟用計量與記錄；或者，如果使用 OCP Marketplace 供應項目，則可以在事後輕鬆啟用這些功能。 如果使用 OpenShift Origin Resource Manager 範本，則需要一些事前工作。

### <a name="openshift-origin-template-pre-work"></a>OpenShift Origin 範本事前工作

1. 使用連接埠 2200 透過 SSH 連線到第一個主要節點。

   範例：

   ```bash
   ssh -p 2200 clusteradmin@masterdnsixpdkehd3h.eastus.cloudapp.azure.com 
   ```

2. 編輯 /etc/ansible/hosts 檔案，然後在「身分識別提供者」區段 (# Enable HTPasswdPasswordIdentityProvider) 後面加入下列幾行：

   ```yaml
   # Setup metrics
   openshift_hosted_metrics_deploy=false
   openshift_metrics_cassandra_storage_type=dynamic
   openshift_metrics_start_cluster=true
   openshift_metrics_hawkular_nodeselector={"type":"infra"}
   openshift_metrics_cassandra_nodeselector={"type":"infra"}
   openshift_metrics_heapster_nodeselector={"type":"infra"}
   openshift_hosted_metrics_public_url=https://metrics.$ROUTING/hawkular/metrics

   # Setup logging
   openshift_hosted_logging_deploy=false
   openshift_hosted_logging_storage_kind=dynamic
   openshift_logging_fluentd_nodeselector={"logging":"true"}
   openshift_logging_es_nodeselector={"type":"infra"}
   openshift_logging_kibana_nodeselector={"type":"infra"}
   openshift_logging_curator_nodeselector={"type":"infra"}
   openshift_master_logging_public_url=https://kibana.$ROUTING
   ```

3. 在相同的 /etc/ansible/hosts 檔案中，將 $ROUTING 取代為用於 openshift_master_default_subdomain 選項的字串。

### <a name="azure-cloud-provider-in-use"></a>Azure 雲端提供者使用中

在第一個主要節點 (Origin) 或防禦節點 (OCP) 上，使用部署期間提供的認證進行 SSH。 發出以下命令：

```bash
ansible-playbook $HOME/openshift-ansible/playbooks/byo/openshift-cluster/openshift-metrics.yml \
-e openshift_metrics_install_metrics=True \
-e openshift_metrics_cassandra_storage_type=dynamic

ansible-playbook $HOME/openshift-ansible/playbooks/byo/openshift-cluster/openshift-logging.yml \
-e openshift_logging_install_logging=True \
-e openshift_hosted_logging_storage_kind=dynamic
```

### <a name="azure-cloud-provider-not-in-use"></a>Azure 雲端提供者未使用中

在第一個主要節點 (Origin) 或防禦節點 (OCP) 上，使用部署期間提供的認證進行 SSH。 發出以下命令：

```bash
ansible-playbook $HOME/openshift-ansible/playbooks/byo/openshift-cluster/openshift-metrics.yml \
-e openshift_metrics_install_metrics=True 

ansible-playbook $HOME/openshift-ansible/playbooks/byo/openshift-cluster/openshift-logging.yml \
-e openshift_logging_install_logging=True 
```

## <a name="next-steps"></a>後續步驟

- [開始使用 OpenShift 容器平台](https://docs.openshift.com/container-platform/3.6/getting_started/index.html) \(英文\)
- [開始使用 OpenShift Origin](https://docs.openshift.org/latest/getting_started/index.html)
