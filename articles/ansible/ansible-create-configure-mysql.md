---
title: 使用 Ansible 來建立及設定適用於 MySQL 的 Azure 資料庫伺服器 (預覽)
description: 深入了解如何使用 Ansible 來建立及設定適用於 MySQL 的 Azure 資料庫伺服器
ms.service: ansible
keywords: ansible, azure, devops, bash, 劇本, mysql, 資料庫
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 09/23/2018
ms.openlocfilehash: 508274d11a9693d28a9b3a01bd6ebbd7198e8711
ms.sourcegitcommit: 5843352f71f756458ba84c31f4b66b6a082e53df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47586644"
---
# <a name="create-and-configure-an-azure-database-for-mysql-server-using-ansible-preview"></a>使用 Ansible 來建立及設定適用於 MySQL 的 Azure 資料庫伺服器 (預覽)
[適用於 MySQL 的 Azure 資料庫](https://docs.microsoft.com/azure/mysql/)是一個受控服務，您可用來在雲端執行、管理及調整高可用性 MySQL 資料庫。 本快速入門說明如何使用 Azure 入口網站，在大約五分鐘內建立適用於 MySQL 的 Azure 資料庫伺服器。 

Ansible 可讓您將環境中的資源部署和設定自動化。 這篇文章示範如何使用 Ansible 來建立適用於 MySQL 的 Azure 資料庫伺服器，並且在 5 分鐘內設定其防火牆規則。 

## <a name="prerequisites"></a>必要條件
- **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。
- [!INCLUDE [ansible-prereqs-for-cloudshell-use-or-vm-creation1.md](../../includes/ansible-prereqs-for-cloudshell-use-or-vm-creation1.md)] [!INCLUDE [ansible-prereqs-for-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-for-cloudshell-use-or-vm-creation2.md)]

> [!Note]
> 必須使用 Ansible 2.7，才能執行本教學課程中的下列範例劇本。 您無法藉由執行 `sudo pip install ansible[azure]==2.7.0rc2` 來安裝 Ansible 2.7 RC 版本。 Ansible 2.7 將於 2018 年 10 月發行。 發行之後，您就不需要指定版本，因為預設版本是 2.7。

## <a name="create-a-resource-group"></a>建立資源群組
資源群組是在其中部署與管理 Azure 資源的邏輯容器。  

下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    location: eastus 
  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
```

將上述劇本儲存為 rg.yml。 若要執行劇本，請使用 **ansible-playbook** 命令，如下所示：
```bash
ansible-playbook rg.yml
```

## <a name="create-mysql-server-and-database"></a>建立 MySQL 伺服器和資料庫
下列範例會建立名為 **mysqlserveransible** 的 MySQL 伺服器，以及名為 **mysqldbansible**的「適用於 MySQL 的 Azure 資料庫」。 這是第 5 代基本用途伺服器，其具有 1 個 vCore。 請注意，**mysqlserver_name** 的值必須是唯一的，並請參閱[定價層文件](https://docs.microsoft.com/azure/mysql/concepts-pricing-tiers)，以了解每個區域和每一層的有效值。 

輸入您自己的 `<server_admin_password>`：

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    location: eastus
    mysqlserver_name: mysqlserveransible
    mysqldb_name: mysqldbansible
    admin_username: mysqladmin
    admin_password: <server_admin_password> 
  tasks:
    - name: Create MySQL Server
      azure_rm_mysqlserver:
        resource_group: "{{ resource_group }}"
        name: "{{ mysqlserver_name }}"
        sku:
          name: B_Gen5_1
          tier: Basic
        location: "{{ location }}"
        version: 5.6
        enforce_ssl: True
        admin_username: "{{ admin_username }}"
        admin_password: "{{ admin_password }}"
        storage_mb: 51200
    - name: Create instance of MySQL Database
      azure_rm_mysqldatabase:
        resource_group: "{{ resource_group }}"
        server_name: "{{ mysqlserver_name }}"
        name: "{{ mysqldb_name }}"
```

將上述劇本儲存為 mysql_create.yml。 若要執行劇本，請使用 **ansible-playbook** 命令，如下所示：
```bash
ansible-playbook mysql_create.yml
```

## <a name="configure-firewall-rule"></a>設定防火牆規則
伺服器層級防火牆規則可允許外部應用程式 (例如 **mysql** 命令列工具或 MySQL Workbench) 穿過 Azure MySQL 服務防火牆連線到您的伺服器。 下列範例會建立名為 **extenalaccess** 的防火牆規則，以允許來自任何外部 IP 位址的連線。 

請根據與您連線來源相對應的 IP 位址範圍，輸入您自己的 **startIpAddress** 和 **endIpAddress**： 

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    mysqlserver_name: mysqlserveransible
  tasks:
  - name: Open firewall to access MySQL Server from outside
    azure_rm_resource:
      api_version: '2017-12-01'
      resource_group: "{{ resource_group }}"
      provider: dbformysql
      resource_type: servers
      resource_name: "{{ mysqlserver_name }}"
      subresource:
        - type: firewallrules
          name: externalaccess
      body:
        properties:
          startIpAddress: "0.0.0.0"
          endIpAddress: "255.255.255.255"
```

> [!NOTE]
> Azure Database for MySQL 的連線透過連接埠 3306 進行通訊。 如果您嘗試從公司網路內進行連線，可能不允許透過連接埠 3306 的輸出流量。 若是如此，除非 IT 部門開啟連接埠 3306，否則您無法連線到您的伺服器。
> 

這裡的 **azure_rm_resource** 模組是用來執行這個工作，包括允許直接使用 REST API。

將上述劇本儲存為 mysql_firewall.yml。 若要執行劇本，請使用 **ansible-playbook** 命令，如下所示：
```bash
ansible-playbook mysql_firewall.yml
```

## <a name="connect-to-the-server-using-command-line-tool"></a>使用命令列工具連線到伺服器
您可以從[這裡](https://dev.mysql.com/downloads/)下載 MySQL，再將它安裝於電腦上。 相反地，您也可以按一下程式碼範例上的 [試用] 按鈕，或是 Azure 入口網站右上角工具列的 `>_` 按鈕，並啟動 **Azure Cloud Shell**。

輸入下一個命令： 

1. 使用 **mysql** 命令列工具連線到伺服器︰
```azurecli-interactive
 mysql -h mysqlserveransible.mysql.database.azure.com -u mysqladmin@mysqlserveransible -p
```

2. 檢視伺服器狀態：
```sql
 mysql> status
```

如果一切順利，命令列工具應輸出下列文字︰

```
demo@Azure:~$ mysql -h mysqlserveransible.mysql.database.azure.com -u mysqladmin@mysqlserveransible -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 65233
Server version: 5.6.39.0 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> status
--------------
mysql  Ver 14.14 Distrib 5.7.23, for Linux (x86_64) using  EditLine wrapper

Connection id:          65233
Current database:
Current user:           mysqladmin@13.76.42.93
SSL:                    Cipher in use is AES256-SHA
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         5.6.39.0 MySQL Community Server (GPL)
Protocol version:       10
Connection:             mysqlserveransible.mysql.database.azure.com via TCP/IP
Server characterset:    latin1
Db     characterset:    latin1
Client characterset:    utf8
Conn.  characterset:    utf8
TCP port:               3306
Uptime:                 36 min 21 sec

Threads: 5  Questions: 559  Slow queries: 0  Opens: 96  Flush tables: 3  Open tables: 10  Queries per second avg: 0.256
--------------
```

## <a name="using-facts-to-query-mysql-servers"></a>使用事實來查詢 MySQL 伺服器
下列範例會在 **myResourceGroup** 中查詢 MySQL 伺服器，接著繼續查詢伺服器上的所有資料庫：

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    mysqlserver_name: mysqlserveransible
  tasks:
    - name: Query MySQL Servers in current resource group
      azure_rm_mysqlserver_facts:
        resource_group: "{{ resource_group }}"
      register: mysqlserverfacts

    - name: Dump MySQL Server facts
      debug:
        var: mysqlserverfacts

    - name: Query MySQL Databases
      azure_rm_mysqldatabase_facts:
        resource_group: "{{ resource_group }}"
        server_name: "{{ mysqlserver_name }}"
      register: mysqldatabasefacts

    - name: Dump MySQL Database Facts
      debug:
        var: mysqldatabasefacts
```

將上述劇本儲存為 mysql_query.yml。 若要執行劇本，請使用 **ansible-playbook** 命令，如下所示：

```bash
ansible-playbook mysql_query.yml
```

然後您會看到 MySQL 伺服器的下列輸出： 
```json
"servers": [
    {
        "admin_username": "mysqladmin",
        "enforce_ssl": false,
        "fully_qualified_domain_name": "mysqlserveransible.mysql.database.azure.com",
        "id": "/subscriptions/685ba005-af8d-4b04-8f16-a7bf38b2eb5a/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/mysqlserveransible",
        "location": "eastus",
        "name": "mysqlserveransible",
        "resource_group": "myResourceGroup",
        "sku": {
            "capacity": 1,
            "family": "Gen5",
            "name": "B_Gen5_1",
            "tier": "Basic"
        },
        "storage_mb": 5120,
        "user_visible_state": "Ready",
        "version": "5.6"
    }
]
```

您也會看到 MySQL 資料庫的下列輸出：
```json
"databases": [
    {
        "charset": "utf8",
        "collation": "utf8_general_ci",
        "name": "information_schema",
        "resource_group": "myResourceGroup",
        "server_name": "mysqlserveransible"
    },
    {
        "charset": "latin1",
        "collation": "latin1_swedish_ci",
        "name": "mysql",
        "resource_group": "myResourceGroup",
        "server_name": "mysqlserveransibler"
    },
    {
        "charset": "latin1",
        "collation": "latin1_swedish_ci",
        "name": "mysqldbansible",
        "resource_group": "myResourceGroup",
        "server_name": "mysqlserveransible"
    },
    {
        "charset": "utf8",
        "collation": "utf8_general_ci",
        "name": "performance_schema",
        "resource_group": "myResourceGroup",
        "server_name": "mysqlserveransible"
    }
]
```

## <a name="clean-up-resources"></a>清除資源

如果您不需要這些資源，可以執行下列範例將它們刪除。 這樣會刪除名為 **myResourceGroup** 的資源群組。 

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
  tasks:
    - name: Delete a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        state: absent
```

將上述劇本儲存為 rg_delete.yml。 若要執行劇本，請使用 **ansible-playbook** 命令，如下所示：
```bash
ansible-playbook rg_delete.yml
```

若您只想要刪除新建立的 MySQL 伺服器，則可執行下列範例加以刪除：

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    mysqlserver_name: mysqlserveransible
  tasks:
    - name: Delete MySQL Server
      azure_rm_mysqlserver:
        resource_group: "{{ resource_group }}"
        name: "{{ mysqlserver_name }}"
        state: absent
```

將上述劇本儲存為 mysql_delete.yml。 若要執行劇本，請使用 **ansible-playbook** 命令，如下所示：
```bash
ansible-playbook mysql_delete.yml
```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"] 
> [Ansible on Azure](https://docs.microsoft.com/azure/ansible/)