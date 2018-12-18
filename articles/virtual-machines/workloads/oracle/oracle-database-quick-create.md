---
title: 在 Azure VM 中建立 Oracle 資料庫 | Microsoft Docs
description: 在您的 Azure 環境中快速啟動並執行 Oracle Database 12c 資料庫。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: romitgirdhar
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/02/2018
ms.author: rogirdh
ms.openlocfilehash: 516552111cc21cacf87fd8179ef49b939e2820d9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46985073"
---
# <a name="create-an-oracle-database-in-an-azure-vm"></a>在 Azure VM 中建立 Oracle 資料庫

本指南詳述如何使用 Azure CLI 從 [Oracle Marketplace 資源庫映像](https://azuremarketplace.microsoft.com/marketplace/apps/Oracle.OracleDatabase12102EnterpriseEdition?tab=Overview)部署 Azure 虛擬機器，以便建立 Oracle 12c 資料庫。 部署伺服器之後，您會透過 SSH 連接，以設定 Oracle 資料庫。 

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az_group_create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```
## <a name="create-virtual-machine"></a>建立虛擬機器

若要建立虛擬機器，請使用 [az vm create](/cli/azure/vm#az_vm_create) 命令。 

下列範例會建立名為 `myVM` 的 VM。 如果預設的金鑰位置還沒有 SSH 金鑰的話，此範例也會建立這些金鑰。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。  

```azurecli-interactive 
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
    --size Standard_DS2_v2 \
    --admin-username azureuser \
    --generate-ssh-keys
```

在您建立 VM 後，Azure CLI 會顯示類似下列範例的資訊。 請記下 `publicIpAddress` 的值。 您必須使用此位址來存取 VM。

```azurecli
{
  "fqdns": "",
  "id": "/subscriptions/{snip}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "westus",
  "macAddress": "00-0D-3A-36-2F-56",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.64.104.241",
  "resourceGroup": "myResourceGroup"
}
```

## <a name="connect-to-the-vm"></a>連接至 VM

若要對 VM 建立 SSH 工作階段，請使用下列命令。 以 VM 的 `publicIpAddress` 值取代 IP 位址。

```bash 
ssh <publicIpAddress>
```

## <a name="create-the-database"></a>建立資料庫

Marketplace 映像上已安裝 Oracle 軟體。 建立範例資料庫，如下所示。 

1.  切換至 oracle 超級使用者，然後將接聽程式初始化以啟用記錄功能：

    ```bash
    $ sudo su - oracle
    $ lsnrctl start
    ```

    輸出大致如下：

    ```bash
    Copyright (c) 1991, 2014, Oracle.  All rights reserved.

    Starting /u01/app/oracle/product/12.1.0/dbhome_1/bin/tnslsnr: please wait...

    TNSLSNR for Linux: Version 12.1.0.2.0 - Production
    Log messages written to /u01/app/oracle/diag/tnslsnr/myVM/listener/alert/log.xml
    Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=myVM.twltkue3xvsujaz1bvlrhfuiwf.dx.internal.cloudapp.net)(PORT=1521)))

    Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
    STATUS of the LISTENER
    ------------------------
    Alias                     LISTENER
    Version                   TNSLSNR for Linux: Version 12.1.0.2.0 - Production
    Start Date                23-MAR-2017 15:32:08
    Uptime                    0 days 0 hr. 0 min. 0 sec
    Trace Level               off
    Security                  ON: Local OS Authentication
    SNMP                      OFF
    Listener Log File         /u01/app/oracle/diag/tnslsnr/myVM/listener/alert/log.xml
    Listening Endpoints Summary...
    (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=myVM.twltkue3xvsujaz1bvlrhfuiwf.dx.internal.cloudapp.net)(PORT=1521)))
    The listener supports no services
    The command completed successfully
    ```

2.  建立資料庫︰

    ```bash
    dbca -silent \
           -createDatabase \
           -templateName General_Purpose.dbc \
           -gdbname cdb1 \
           -sid cdb1 \
           -responseFile NO_VALUE \
           -characterSet AL32UTF8 \
           -sysPassword OraPasswd1 \
           -systemPassword OraPasswd1 \
           -createAsContainerDatabase true \
           -numberOfPDBs 1 \
           -pdbName pdb1 \
           -pdbAdminPassword OraPasswd1 \
           -databaseType MULTIPURPOSE \
           -automaticMemoryManagement false \
           -storageType FS \
           -ignorePreReqs
    ```

    建立資料庫需要幾分鐘的時間。

3. 設定 Oracle 變數

在連線之前，您需要設定兩個環境變數︰ORACLE_HOME 和 ORACLE_SID。

```bash
ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1; export ORACLE_HOME
ORACLE_SID=cdb1; export ORACLE_SID
```
您也可以將 ORACLE_HOME 和 ORACLE_SID 變數新增至 .bashrc 檔案。 這會儲存環境變數以供未來登入。請確認已使用您選擇的編輯器，將下列陳述式新增至 `~/.bashrc` 檔案。

```bash
# Add ORACLE_HOME. 
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1 
# Add ORACLE_SID. 
export ORACLE_SID=cdb1 
```

## <a name="oracle-em-express-connectivity"></a>Oracle EM Express 連線能力

對於您可用來瀏覽資料庫的 GUI 管理工具，請設定 Oracle EM Express。 若要連線到 Oracle EM Express，您必須先在 Oracle 中設定連接埠。 

1. 使用 sqlplus 連線到您的資料庫：

    ```bash
    sqlplus / as sysdba
    ```

2. 連線後，請針對 EM Express 設定連接埠 5502

    ```bash
    exec DBMS_XDB_CONFIG.SETHTTPSPORT(5502);
    ```

3. 如果容器 PDB1 尚未開啟，請加以開啟，但是先檢查狀態：

    ```bash
    select con_id, name, open_mode from v$pdbs;
    ```

    輸出大致如下：

    ```bash
      CON_ID NAME                           OPEN_MODE 
      ----------- ------------------------- ---------- 
      2           PDB$SEED                  READ ONLY 
      3           PDB1                      MOUNT
    ```

4. 如果 `PDB1` 的 OPEN_MODE 不是「讀寫」，則執行下列命令以開啟 PDB1：

   ```bash
    alter session set container=pdb1;
    alter database open;
   ```

您需要鍵入 `quit` 結束 sqlplus 工作階段，並鍵入 `exit` 登出 oracle 使用者。

## <a name="automate-database-startup-and-shutdown"></a>自動進行資料庫啟動和關機

當您重新啟動 VM 時，Oracle 資料庫預設不會自動啟動。 若要將 Oracle 資料庫設定為自動啟動，請先以 root 的身分登入。 接著，建立並更新一些系統檔案。

1. 以 root 的身分登入
    ```bash
    sudo su -
    ```

2.  使用您最愛的編輯器，編輯 `/etc/oratab` 檔案，並將預設的 `N` 變更為 `Y`：

    ```bash
    cdb1:/u01/app/oracle/product/12.1.0/dbhome_1:Y
    ```

3.  建立名為 `/etc/init.d/dbora` 的檔案，並貼上下列內容︰

    ```
    #!/bin/sh
    # chkconfig: 345 99 10
    # Description: Oracle auto start-stop script.
    #
    # Set ORA_HOME to be equivalent to $ORACLE_HOME.
    ORA_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
    ORA_OWNER=oracle

    case "$1" in
    'start')
        # Start the Oracle databases:
        # The following command assumes that the Oracle sign-in
        # will not prompt the user for any values.
        # Remove "&" if you don't want startup as a background process.
        su - $ORA_OWNER -c "$ORA_HOME/bin/dbstart $ORA_HOME" &
        touch /var/lock/subsys/dbora
        ;;

    'stop')
        # Stop the Oracle databases:
        # The following command assumes that the Oracle sign-in
        # will not prompt the user for any values.
        su - $ORA_OWNER -c "$ORA_HOME/bin/dbshut $ORA_HOME" &
        rm -f /var/lock/subsys/dbora
        ;;
    esac
    ```

4.  使用 chmod 變更檔案的權限，如下所示：

    ```bash
    chgrp dba /etc/init.d/dbora
    chmod 750 /etc/init.d/dbora
    ```

5.  建立用於啟動和關閉的符號連結，如下所示：

    ```bash
    ln -s /etc/init.d/dbora /etc/rc.d/rc0.d/K01dbora
    ln -s /etc/init.d/dbora /etc/rc.d/rc3.d/S99dbora
    ln -s /etc/init.d/dbora /etc/rc.d/rc5.d/S99dbora
    ```

6.  若要測試您的變更，請重新啟動 VM：

    ```bash
    reboot
    ```

## <a name="open-ports-for-connectivity"></a>開始連接埠進行連線

最後一項工作是設定一些外部端點。 若要設定可保護 VM 的 Azure 網路安全性群組，請先結束 VM 中您的 SSH 工作階段 (應已在前一個步驟中重新啟動時被移出 SSH)。 

1.  若要開啟您用來從遠端存取 Oracle 資料庫的端點，請使用 [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create) 建立網路安全性群組規則，如下所示： 

    ```azurecli-interactive
    az network nsg rule create \
        --resource-group myResourceGroup\
        --nsg-name myVmNSG \
        --name allow-oracle \
        --protocol tcp \
        --priority 1001 \
        --destination-port-range 1521
    ```

2.  若要開啟您用來從遠端存取 Oracle EM Express 的端點，請使用 [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create) 建立網路安全性群組規則，如下所示：

    ```azurecli-interactive
    az network nsg rule create \
        --resource-group myResourceGroup \
        --nsg-name myVmNSG \
        --name allow-oracle-EM \
        --protocol tcp \
        --priority 1002 \
        --destination-port-range 5502
    ```

3. 如有需要，請使用 [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show) 再次取得 VM 的公用 IP 位址，如下所示：

    ```azurecli-interactive
    az network public-ip show \
        --resource-group myResourceGroup \
        --name myVMPublicIP \
        --query [ipAddress] \
        --output tsv
    ```

4.  從您的瀏覽器連接 EM Express。 確定您的瀏覽器與 EM Express 相容 (需要安裝 Flash)： 

    ```
    https://<VM ip address or hostname>:5502/em
    ```

您可以使用 SYS 帳戶進行登入，然後勾選 as sysdba 核取方塊。 使用您在安裝期間設定的密碼 OraPasswd1。 

![Oracle OEM Express 登入頁面的螢幕擷取畫面](./media/oracle-quick-start/oracle_oem_express_login.png)

## <a name="clean-up-resources"></a>清除資源

完成在 Azure 上探索第一個 Oracle 資料庫而且不再需要 VM 之後，就可以使用 [az group delete](/cli/azure/group#az_group_delete) 命令移除資源群組、VM 和所有相關資源。

```azurecli-interactive 
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

深入了解 [Azure 上的其他 Oracle 解決方案](oracle-considerations.md)。 

嘗試[安裝和設定 Oracle Automated Storage Management](configure-oracle-asm.md) 教學課程。