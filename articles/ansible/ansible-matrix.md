---
title: Azure 的 Ansible 模組和版本對照表
description: Azure 的 Ansible 模組和版本對照表
ms.service: ansible
keywords: ansible, 角色, 對照表, 版本, azure, devops
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.date: 09/22/2018
ms.topic: article
ms.openlocfilehash: f574203f31a21f945115296693f04cff649614c6
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47165631"
---
# <a name="ansible-module-and-version-matrix"></a>Ansible 模組和版本對照表

## <a name="ansible-modules-for-azure"></a>Azure 的 Ansible 模組
Ansible 隨附於許多模組，這些模組可以直接在遠端主機上或透過劇本來執行。
此文章列出 Azure 的 Ansible 模組，這些模組可以佈建 Azure 雲端資源，例如虛擬機器、網路及容器服務。 您可以從 Ansible 的正式發行版本或從 Microsoft 發佈的下列劇本角色，取得這些模組。

| Azure 的 Ansible 模組                   |  Ansible 2.4 |  Ansible 2.5 |  Ansible 2.6 | Ansible 2.7 | [Ansible 角色](#introduction-to-azurepreviewmodule) | 
|---------------------------------------------|--------------|--------------|-----------------------------|-------------------------------------|-------------------------------------| 
| **計算**                    |           |                          |                          |                            |                                | 
| azure_rm_availabilityset                    | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_availabilityset_facts              | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_deployment                         | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_resource                           | -            | -                           | 是          | yes          | 是                                 | 
| azure_rm_resource_facts                     | -            | -                           | 是          | yes          | 是                                 | 
| azure_rm_virtualmachine_scaleset_facts      | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_virtualmachineimage_facts          | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_resourcegroup                      | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_resourcegroup_facts                | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_virtualmachine                     | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_virtualmachine_facts               | -            | -                           | -            | 是          | 是                                 | 
| azure_rm_virtualmachine_extension           | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_virtualmachine_scaleset            | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_image                              |              | 是                         | yes          | yes          | 是                                 | 
| **網路功能**                    |           |                          |                          |                             |                               | 
| azure_rm_virtualnetwork                     | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_virtualnetwork_facts               | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_subnet                             | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_networkinterface                   | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_networkinterface_facts             | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_publicipaddress                    | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_publicipaddress_facts              | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_dnsrecordset                       | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_dnsrecordset_facts                 | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_dnszone                            | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_dnszone_facts                      | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_loadbalancer                       | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_loadbalancer_facts                 | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_appgateway                         | -            | -                           | -            | 是          | 是                                 | 
| azure_rm_appgwroute                         | -            | -                           | -            | -            | 是                                 | 
| azure_rm_appgwroute                         | -            | -                           | -            | -            | 是                                 |
| azure_rm_appgwroute_facts                   | -            | -                           | -            | -            | 是                                 |
| azure_rm_appgwroutetable                    | -            | -                           | -            | -            | 是                                 |
| azure_rm_appgwroutetable_facts              | -            | -                           | -            | -            | 是                                 | 
| azure_rm_securitygroup                      | 是          | yes                         | yes          | yes          | 是                                 |
| azure_rm_route                              | -            | -                           | -            | 是          | 是                                 | 
| azure_rm_routetable                         | -            | -                           | -            | 是          | 是                                 | 
| azure_rm_routetable_facts                   | -            | -                           | -            | 是          | 是                                 | 
| **儲存體**                    |           |                          |                          |                             |                               | 
| azure_rm_storageaccount                     | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_storageaccount_facts               | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_storageblob                        | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_managed_disk                       | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_managed_disk_facts                 | 是          | yes                         | yes          | yes          | 是                                 | 
| **容器**                    |           |                          |                          |                            |                                | 
| azure_rm_aks                                | -            | -                           | 是          | yes          | 是                                 | 
| azure_rm_aks_facts                          | -            | -                           | 是          | yes          | 是                                 | 
| azure_rm_acs                                | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_containerinstance                  | -            | 是                         | yes          | yes          | 是                                 | 
| azure_rm_containerinstance_facts            | -            | -                           | -              | -            | 是                                 | 
| azure_rm_containerregistry                  | -            | 是                         | yes          | yes          | 是                                 | 
| azure_rm_containerregistry_facts            | -            | -                           | -            | 是          | 是                                 | 
| azure_rm_containerregistryreplication       | -            | -                           | -            | -            | 是                                 | 
| azure_rm_containerregistryreplication_facts | -            | -                           | -            | -            | 是                                 | 
| azure_rm_containerregistrywebhook           | -            | -                           | -            | -            | 是                                 | 
| azure_rm_containerregistrywebhook_facts     | -            | -                           | -            | -            | 是                                 | 
| **Azure Functions**                    |           |                          |                          |                            |                                | 
| azure_rm_functionapp                        | 是          | yes                         | yes          | yes          | 是                                 | 
| azure_rm_functionapp_facts                  | 是          | yes                         | yes          | yes          | 是                                 | 
| **資料庫**                    |           |                          |                          |                             |                               | 
| azure_rm_sqlserver                          | -            | 是                         | yes          | yes          | 是                                 | 
| azure_rm_sqlserver_facts                    | -            | 是                         | yes          | yes          | 是                                 | 
| azure_rm_sqldatabase                        | -            | 是                         | yes          | yes          | 是                                 | 
| azure_rm_sqldatabase_facts                  | -            | -                           | -            | -            | 是                                 | 
| azure_rm_sqlelasticpool                     | -            | -                           | -            | -            | 是                                 | 
| azure_rm_sqlelasticpool_facts               | -            | -                           | -            | -            | 是                                 | 
| azure_rm_sqlfirewallrule                    | -            | -                           | -            | 是          | 是                                 | 
| azure_rm_sqlfirewallrule_facts              | -            | -                           | -            | -            | 是                                 | 
| azure_rm_mysqlserver                        | -            | 是                         | yes          | yes          | 是                                 | 
| azure_rm_mysqlserver_facts                  | -            | -                           | -            | 是          | 是                                 | 
| azure_rm_mysqldatabase                      | -            | 是                         | yes          | yes          | 是                                 | 
| azure_rm_mysqldatabase_facts                | -            | -                           | -            | 是          | 是                                 | 
| azure_rm_mysqlfirewallrule                  | -            | -                           | -            | -            | 是                                 | 
| azure_rm_mysqlfirewallrule_facts            | -            | -                           | -            | -            | 是                                 | 
| azure_rm_mysqlconfiguration                 | -            | -                           | -            | -            | 是                                 | 
| azure_rm_mysqlconfiguration_facts           | -            | -                           | -            | -            | 是                                 | 
| azure_rm_postgresqlserver                   | -            | 是                         | yes          | yes          | 是                                 | 
| azure_rm_postgresqlserver_facts             | -            | -                           | -            | 是          | 是                                 | 
| azure_rm_postgresqldatabase                 | -            | 是                         | yes          | yes          | 是                                 | 
| azure_rm_postgresqldatabase_facts           | -            | -                           | -            | 是          | 是                                 | 
| azure_rm_postgresqlfirewallrule             | -            | -                           | -            | -            | 是                                 | 
| azure_rm_postgresqlfirewallrule_facts       | -            | -                           | -            | -            | 是                                 | 
| azure_rm_postgresqlconfiguration            | -            | -                           | -            | -            | 是                                 | 
| azure_rm_postgresqlconfiguration_facts      | -            | -                           | -            | -            | 是                                 | 
| **金鑰保存庫**                    |           |                          |                          |                             |                               | 
| azure_rm_keyvault                           | -            | 是                         | yes          | yes          | 是                                 |
| azure_rm_keyvault_facts                     | -            | -                           | -              | -              | 是                               |
| azure_rm_keyvaultkey                        | -            | 是                         | yes          | yes          | 是                                 |
| azure_rm_keyvaultsecret                     | -            | 是                         | yes          | yes          | 是                                 |
| **Web Apps**                    |           |                          |                          |                             |                               | 
| azure_rm_appserviceplan                          | -            | -                         | -          | 是          | 是                                 | 
| azure_rm_appserviceplan_facts                    | -            | -                         | -          | 是          | 是                                 | 
| azure_rm_webapp                                  | -            | -                         | -          | 是          | 是                                 | 
| azure_rm_webapp_facts                            | -            | -                         | -          | 是          | 是                                 | 
| **流量管理員**                    |           |                          |                          |                             |                               | 
| azure_rm_trafficmanagerendpoint                  | -            | -                         | -          | 是          | 是                                 | 
| azure_rm_trafficmanagerendpoint_facts            | -            | -                         | -          | 是          | 是                                 | 
| azure_rm_trafficmanagerprofile                   | -            | -                         | -          | 是          | 是                                 | 
| azure_rm_trafficmanagerprofile_facts             | -            | -                         | -          | 是          | 是                                 | 
| **自動調整**                    |           |                          |                          |                             |                               | 
| azure_rm_autoscale                  | -            | -                         | -          | 是          | 是                                 | 
| azure_rm_autoscale_facts            | -            | -                         | -          | 是          | 是                                 | 

## <a name="introduction-to-playbook-role-for-azure"></a>Azure 劇本角色簡介
[azure_preview_module 劇本角色](https://galaxy.ansible.com/Azure/azure_preview_modules/)是最完整的角色，並內含所有最新的 Azure 模組。 相較於官方 Ansible 版本，此角色會更加及時地推出更新和錯誤修正。 如果您針對 Azure 資源佈建用途來使用 Ansible，建議您安裝 azure_preview_module 角色。

azure_preview_module 劇本角色每三週發行一次。

## <a name="next-steps"></a>後續步驟
如需劇本角色的相關詳細資訊，請參閱[建立可重複使用的劇本](http://docs.ansible.com/ansible/latest/playbooks_reuse.html)。 
