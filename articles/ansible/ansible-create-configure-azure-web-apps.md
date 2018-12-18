---
title: 使用 Ansible 建立 Azure Web Apps (預覽)
description: 深入了解如何使用 Ansible 來建立 Web App，該應用程式在 Linux 上的 App Service 中具有 Java 8 和 Tomcat 容器執行階段
ms.service: ansible
keywords: ansible、azure、devops、bash、劇本、Azure App Service、Web App、Java
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 09/20/2018
ms.openlocfilehash: 1899b1fc1e0a38d859fb3a7ce2153585579650f3
ms.sourcegitcommit: 5843352f71f756458ba84c31f4b66b6a082e53df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47586647"
---
# <a name="create-azure-app-service-web-apps-using-ansible-preview"></a>使用 Ansible 建立 Azure App Service Web Apps (預覽)
[Azure App Service Web Apps](https://docs.microsoft.com/azure/app-service/app-service-web-overview) (或簡稱 Web Apps) 是用來裝載 Web 應用程式、REST API 和行動後端的服務。 您可以使用您慣用的語言進行開發，不管是 .NET、.NET Core、Java、Ruby、Node.js、PHP 還是 Python 都可以。

Ansible 可讓您將環境中的資源部署和設定自動化。 這篇文章會示範如何使用 Ansible 來建立具有 Java 執行階段的 Web App。 

## <a name="prerequisites"></a>必要條件
- **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。
- [!INCLUDE [ansible-prereqs-for-cloudshell-use-or-vm-creation1.md](../../includes/ansible-prereqs-for-cloudshell-use-or-vm-creation1.md)] [!INCLUDE [ansible-prereqs-for-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-for-cloudshell-use-or-vm-creation2.md)]

> [!Note]
> 必須使用 Ansible 2.7，才能執行本教學課程中的下列範例劇本。 您無法藉由執行 `sudo pip install ansible[azure]==2.7.0rc2` 來安裝 Ansible 2.7 RC 版本。 Ansible 2.7 將於 2018 年 10 月發行。 發行之後，您就不需要指定版本，因為預設版本是 2.7。 

## <a name="create-a-simple-app-service"></a>建立簡單的 App Service
本節提供會定義下列資源的範例 Ansible 腳本：
- 資源群組，這是您 App Service 方案和 Web 應用程式的部署目標
- Web 應用程式，該應用程式在 Linux 上的 App Service 中具有 Java 8 和 Tomcat 容器執行階段

```
- hosts: localhost
  connection: local
  vars:
    resource_group: myfirstResourceGroup
    webapp_name: myfirstWebApp
    location: eastus
  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"

    - name: Create App Service on Linux with Java Runtime
      azure_rm_webapp:
        resource_group: "{{ resource_group }}"
        name: "{{ webapp_name }}"
        plan:
          resource_group: "{{ resource_group }}"
          name: myappplan
          is_linux: true
          sku: S1
          number_of_workers: 1
        frameworks:
          - name: "java"
            version: "8"
            settings:
              java_container: tomcat
              java_container_version: 8.5
```
將上述劇本儲存為 firstwebapp.yml。

若要執行劇本，請使用 **ansible-playbook** 命令，如下所示：
```bash
ansible-playbook firstwebapp.yml
```

執行 Ansible 劇本後的輸出會顯示已成功建立 Web 應用程式：

```
TASK [Create a resource group] *************************************************
changed: [localhost]

TASK [Create App Service on Linux with Java Runtime] ******************************
 [WARNING]: Azure API profile latest does not define an entry for
WebSiteManagementClient
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0   
```

## <a name="create-app-service-with-traffic-manager"></a>建立具有流量管理員的 App Service
您可以使用 [Azure 流量管理員](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager)，來控制如何將來自 Web 用戶端的要求分散至 Azure App Service 中的應用程式。 將 App Service 端點新增至 Azure 流量管理員設定檔時，Azure 流量管理員就會持續追蹤您 App Service 應用程式的狀態 (執行中、已停止或已刪除)，以判定其中哪些端點應接收流量。

在 App Service 中，應用程式會在 [App Service 方案](https://docs.microsoft.com/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
)中執行。 App Service 方案會針對要執行的 Web 應用程式定義一組計算資源。 您可以在不同的群組中管理 App Service 方案和 Web App。

本節提供會定義下列資源的範例 Ansible 腳本：
- 資源群組，這是您 App Service 方案的部署目標
- App Service 方案
- 次要資源群組，這是您 Web 應用程式的部署目標
- Web 應用程式，該應用程式在 Linux 上的 App Service 中具有 Java 8 和 Tomcat 容器執行階段
- 流量管理員設定檔
- 流量管理員端點，使用已建立的網站

```
- hosts: localhost
  connection: local
  vars:
    resource_group: myResourceGroup
    plan_resource_group: planResourceGroup
    app_name: myLinuxWebApp
    location: eastus
    linux_plan_name: myAppServicePlan
    traffic_manager_profile_name: myTrafficManagerProfile
    traffic_manager_endpoint_name: myTrafficManagerEndpoint

  tasks:
  - name: Create resource group
    azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"

  - name: Create secondary resource group
    azure_rm_resourcegroup:
        name: "{{ plan_resource_group }}"
        location: "{{ location }}"

  - name: Create App Service Plan
    azure_rm_appserviceplan:
      resource_group: "{{ plan_resource_group }}"
      name: "{{ linux_plan_name }}"
      location: "{{ location }}"
      is_linux: true
      sku: S1
      number_of_workers: 1

  - name: Create App Service on Linux with Java Runtime
    azure_rm_webapp:
        resource_group: "{{ resource_group }}"
        name: "{{ app_name }}"
        plan:
          resource_group: "{{ plan_resource_group }}"
          name: "{{ linux_plan_name }}"
          is_linux: true
          sku: S1
          number_of_workers: 1
        app_settings:
          testkey: "testvalue"
        frameworks:
          - name: java
            version: 8
            settings:
              java_container: "Tomcat"
              java_container_version: "8.5"

  - name: Get web app facts
    azure_rm_webapp_facts:
      resource_group: "{{ resource_group }}"
      name: "{{ app_name }}"
    register: webapp
    
  - name: Create Traffic Manager Profile
    azure_rm_trafficmanagerprofile:
      resource_group: "{{ resource_group }}"
      name: "{{ traffic_manager_profile_name }}"
      location: global
      routing_method: performance
      dns_config:
        relative_name: "{{ traffic_manager_profile_name }}"
        ttl:  60
      monitor_config:
        protocol: HTTPS
        port: 80
        path: '/'

  - name: Add endpoint to traffic manager profile, using created web site
    azure_rm_trafficmanagerendpoint:
      resource_group: "{{ resource_group }}"
      profile_name: "{{ traffic_manager_profile_name }}"
      name: "{{ traffic_manager_endpoint_name }}"
      type: azure_endpoints
      location: "{{ location }}"
      target_resource_id: "{{ webapp.webapps[0].id }}"

```
將上述劇本儲存為 webapp.yml，或者[下載劇本](https://github.com/Azure-Samples/ansible-playbooks/blob/master/webapp.yml)。

若要執行劇本，請使用 **ansible-playbook** 命令，如下所示：
```bash
ansible-playbook webapp.yml
```

執行 Ansible 劇本後的輸出會顯示已成功建立 App Service 方案、Web 應用程式、流量管理員設定檔及端點：
```
TASK [Create resource group] ****************************************************************************
changed: [localhost]

TASK [Create resource group for app service plan] ****************************************************************************
changed: [localhost]

TASK [Create App Service Plan] ****************************************************************************
 [WARNING]: Azure API profile latest does not define an entry for WebSiteManagementClient

changed: [localhost]

TASK [Create App Service on Linux with Java Runtime] ****************************************************************************
changed: [localhost]

TASK [Get web app facts] *****************************************************************************
ok: [localhost]

TASK [Create Traffic Manager Profile] *****************************************************************************
 [WARNING]: Azure API profile latest does not define an entry for TrafficManagerManagementClient

changed: [localhost]

TASK [Add endpoint to traffic manager profile, using the web site created above] *****************************************************************************
changed: [localhost]

TASK [Get Traffic Manager Profile facts] ******************************************************************************
ok: [localhost]

PLAY RECAP ******************************************************************************
localhost                  : ok=9    changed=6    unreachable=0    failed=0
```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"] 
> [Ansible on Azure](https://docs.microsoft.com/azure/ansible/)