---
title: 在 Azure Linux VM 上實作 Oracle Golden Gate | Microsoft Docs
description: 快速在您的 Azure 環境中啟動並執行 Oracle Golden Gate。
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
ms.openlocfilehash: f0ae48cadf2e90dc685a24aff54d89f86a11c287
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39494288"
---
# <a name="implement-oracle-golden-gate-on-an-azure-linux-vm"></a>在 Azure Linux VM 上實作 Oracle Golden Gate 

Azure CLI 可用來從命令列或在指令碼中建立和管理 Azure 資源。 本指南詳述如何使用 Azure CLI 從 Azure Marketplace 資源庫映像部署 Oracle 12c 資料庫。 

這份文件逐步示範如何在 Azure VM 上建立、安裝及設定 Oracle Golden Gate。

開始之前，請確定已安裝 Azure CLI。 如需詳細資訊，請參閱 [Azure CLI 安裝指南](https://docs.microsoft.com/cli/azure/install-azure-cli)。

## <a name="prepare-the-environment"></a>準備環境

若要執行 Oracle Golden Gate 的安裝，您需要在相同的可用性設定組建立兩個 Azure VM。 您用來建立 VM 的 Marketplace 映像是 **Oracle:Oracle-Database-Ee:12.1.0.2:latest**。

您也需要熟悉 Unix 編輯器 vi，並且對 x11 (X Windows) 有基本了解。

環境設定的摘要如下：
> 
> |  | **主要網站** | **複寫網站** |
> | --- | --- | --- |
> | **Oracle 版本** |Oracle 12c 版本 2 – (12.1.0.2) |Oracle 12c 版本 2 – (12.1.0.2)|
> | **機器名稱** |myVM1 |myVM2 |
> | **作業系統** |Oracle Linux 6.x |Oracle Linux 6.x |
> | **Oracle SID** |CDB1 |CDB1 |
> | **複寫結構描述** |TEST|TEST |
> | **Golden Gate 擁有者/複寫** |C##GGADMIN |REPUSER |
> | **Golden Gate 流程** |EXTORA |REPORA|


### <a name="sign-in-to-azure"></a>登入 Azure 

使用 [az login](/cli/azure/reference-index#az_login) 命令登入您的 Azure 訂用帳戶。 然後，遵循螢幕上的指示來進行。

```azurecli
az login
```

### <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az_group_create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

下列範例會在 `westus` 位置建立名為 `myResourceGroup` 的資源群組。

```azurecli
az group create --name myResourceGroup --location westus
```

### <a name="create-an-availability-set"></a>建立可用性設定組

下列步驟為選用步驟，但建議執行。 如需詳細資訊，請參閱 [Azure 可用性設定組指南](https://docs.microsoft.com/azure/virtual-machines/windows/infrastructure-availability-sets-guidelines)。

```azurecli
az vm availability-set create \
    --resource-group myResourceGroup \
    --name myAvailabilitySet \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 2
```

### <a name="create-a-virtual-machine"></a>建立虛擬機器

使用 [az vm create](/cli/azure/vm#az_vm_create) 命令來建立 VM。 

下列範例會建立兩個 VM，名為 `myVM1` 和 `myVM2`。 如果預設的金鑰位置還沒有 SSH 金鑰，請建立這些金鑰。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。

#### <a name="create-myvm1-primary"></a>建立 myVM1 (主要)：
```azurecli
az vm create \
     --resource-group myResourceGroup \
     --name myVM1 \
     --availability-set myAvailabilitySet \
     --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
     --size Standard_DS1_v2  \
     --generate-ssh-keys \
```

建立 VM 後，Azure CLI 會顯示類似下列範例的資訊。 (記下 `publicIpAddress`。 此位址用來存取 VM。)

```azurecli
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "westus",
  "macAddress": "00-0D-3A-36-2F-56",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.64.104.241",
  "resourceGroup": "myResourceGroup"
}
```

#### <a name="create-myvm2-replicate"></a>建立 myVM2 (複寫)：
```azurecli
az vm create \
     --resource-group myResourceGroup \
     --name myVM2 \
     --availability-set myAvailabilitySet \
     --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
     --size Standard_DS1_v2  \
     --generate-ssh-keys \
```

在建立之後一樣記下 `publicIpAddress`。

### <a name="open-the-tcp-port-for-connectivity"></a>開啟用於連線的 TCP 通訊埠

下一個步驟是設定外部端點，可讓您從遠端存取 Oracle 資料庫。 若要設定外部端點，請執行下列命令。

#### <a name="open-the-port-for-myvm1"></a>開啟 myVM1 的連接埠：

```azurecli
az network nsg rule create --resource-group myResourceGroup\
    --nsg-name myVm1NSG --name allow-oracle\
    --protocol tcp --direction inbound --priority 999 \
    --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 1521 --access allow
```

結果看起來應該會像下面的回應這樣：

```bash
{
  "access": "Allow",
  "description": null,
  "destinationAddressPrefix": "*",
  "destinationPortRange": "1521",
  "direction": "Inbound",
  "etag": "W/\"bd77dcae-e5fd-4bd6-a632-26045b646414\"",
  "id": "/subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myVmNSG/securityRules/allow-oracle",
  "name": "allow-oracle",
  "priority": 999,
  "protocol": "Tcp",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sourceAddressPrefix": "*",
  "sourcePortRange": "*"
}
```

#### <a name="open-the-port-for-myvm2"></a>開啟 myVM2 的連接埠：

```azurecli
az network nsg rule create --resource-group myResourceGroup\
    --nsg-name myVm2NSG --name allow-oracle\
    --protocol tcp --direction inbound --priority 999 \
    --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 1521 --access allow
```

### <a name="connect-to-the-virtual-machine"></a>連接至虛擬機器

使用下列命令，建立與虛擬機器的 SSH 工作階段。 以虛擬機器的 `publicIpAddress` 取代 IP 位址。

```bash 
ssh <publicIpAddress>
```

### <a name="create-the-database-on-myvm1-primary"></a>在 myVM1 (主要) 上建立資料庫

Oracle 軟體已安裝於 Marketplace 映像上，因此下一個步驟是安裝資料庫。 

以 'oracle' 超級使用者的身分執行軟體：

```bash
sudo su - oracle
```

建立資料庫︰

```bash
$ dbca -silent \
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
輸出結果看起來應該會像下面的回應這樣：

```bash
Copying database files
1% complete
2% complete
8% complete
13% complete
19% complete
27% complete
Creating and starting Oracle instance
29% complete
32% complete
33% complete
34% complete
38% complete
42% complete
43% complete
45% complete
Completing Database Creation
48% complete
51% complete
53% complete
62% complete
70% complete
72% complete
Creating Pluggable Databases
78% complete
100% complete
Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/cdb1/cdb1.log" for more details.
```

設定 ORACLE_SID 和 ORACLE_HOME 變數。

```bash
$ ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1; export ORACLE_HOME
$ ORACLE_SID=cdb1; export ORACLE_SID
$ LD_LIBRARY_PATH=ORACLE_HOME/lib; export LD_LIBRARY_PATH
```

您可以視需要將 ORACLE_HOME 和 ORACLE_SID 新增到 .bashrc 檔案中，如此可將這些設定儲存，以供日後登入使用：

```bash
# add oracle home
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
# add oracle sid
export ORACLE_SID=cdb1
# add Oracle library path
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
```

### <a name="start-oracle-listener"></a>啟動 Oracle 接聽程式
```bash
$ sudo su - oracle
$ lsnrctl start
```

### <a name="create-the-database-on-myvm2-replicate"></a>在 myVM2 (複寫) 上建立資料庫

```bash
sudo su - oracle
```
建立資料庫︰

```bash
$ dbca -silent \
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
設定 ORACLE_SID 和 ORACLE_HOME 變數。

```bash
$ ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1; export ORACLE_HOME
$ ORACLE_SID=cdb1; export ORACLE_SID
$ LD_LIBRARY_PATH=ORACLE_HOME/lib; export LD_LIBRARY_PATH
```

您可以視需要將 ORACLE_HOME 和 ORACLE_SID 新增到 .bashrc 檔案中，如此可將這些設定儲存，以供日後登入使用。

```bash
# add oracle home
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
# add oracle sid
export ORACLE_SID=cdb1
# add Oracle library path
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
```

### <a name="start-oracle-listener"></a>啟動 Oracle 接聽程式
```bash
$ sudo su - oracle
$ lsnrctl start
```

## <a name="configure-golden-gate"></a>設定 Golden Gate 
若要設定 Golden Gate，請採取本節中的步驟。

### <a name="enable-archive-log-mode-on-myvm1-primary"></a>在 myVM1 (主要) 上啟用封存記錄模式

```bash
$ sqlplus / as sysdba
SQL> SELECT log_mode FROM v$database;

LOG_MODE
------------
NOARCHIVELOG

SQL> SHUTDOWN IMMEDIATE;
SQL> STARTUP MOUNT;
SQL> ALTER DATABASE ARCHIVELOG;
SQL> ALTER DATABASE OPEN;
```
啟用強制記錄功能，並確定至少有一個記錄檔存在。

```bash
SQL> ALTER DATABASE FORCE LOGGING;
SQL> ALTER SYSTEM SWITCH LOGFILE;
SQL> ALTER SYSTEM set enable_goldengate_replication=true;
SQL> ALTER PLUGGABLE DATABASE PDB1 OPEN;
SQL> ALTER SESSION SET CONTAINER=PDB1;
SQL> ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
SQL> EXIT;
```

### <a name="download-golden-gate-software"></a>下載 Golden Gate 軟體
若要下載並準備 Oracle Golden Gate 軟體，請完成下列步驟︰

1. 從 [Oracle Golden Gate 下載分頁](http://www.oracle.com/technetwork/middleware/goldengate/downloads/index.html)下載 **fbo_ggs_Linux_x64_shiphome.zip** 檔案。 在下載標題 **Oracle GoldenGate 12.x.x.x for Oracle Linux x86-64** 底下，應該有一組 .zip 檔可以下載。

2. 將 .zip 檔下載到用戶端電腦之後，請使用「安全複製通訊協定 (SCP)」將這些檔案複製到 VM：

  ```bash
  $ scp fbo_ggs_Linux_x64_shiphome.zip <publicIpAddress>:<folder>
  ```

3. 將 .zip 檔移到 **/opt** 資料夾。 接著，變更檔案的擁有者，如下所示︰

  ```bash
  $ sudo su -
  # mv <folder>/*.zip /opt
  ```

4. 將檔案解壓縮 (如果您尚未安裝 Linux 解壓縮公用程式，請加以安裝)︰

  ```bash
  # yum install unzip
  # cd /opt
  # unzip fbo_ggs_Linux_x64_shiphome.zip
  ```

5. 變更權限：

  ```bash
  # chown -R oracle:oinstall /opt/fbo_ggs_Linux_x64_shiphome
  ```

### <a name="prepare-the-client-and-vm-to-run-x11-for-windows-clients-only"></a>讓用戶端和 VM 準備好執行 x11 (僅適用於 Windows 用戶端)
這是選擇性步驟。 如果您使用 Linux 用戶端或是已設定 x11，可以跳過此步驟。

1. 將 PuTTY 和 Xming 下載到 Windows 電腦︰

  * [下載 PuTTY](http://www.putty.org/)
  * [下載 Xming](https://xming.en.softonic.com/)

2.  在 PuTTY 安裝好之後，請在 PuTTY 資料夾 (例如 C:\Program Files\PuTTY) 執行 puttygen.exe (PuTTY 金鑰產生器)。

3.  在 PuTTY 金鑰產生器中︰

  - 若要產生金鑰，請選取 [產生] 按鈕。
  - 複製金鑰的內容 (**Ctrl+C**)。
  - 選取 [儲存私密金鑰] 按鈕。
  - 略過隨之出現的警告，然後選取 [確定]。

    ![PuTTY 金鑰產生器頁面的螢幕擷取畫面](./media/oracle-golden-gate/puttykeygen.png)

4.  在您的 VM 中，執行下列命令︰

  ```bash
  # sudo su - oracle
  $ mkdir .ssh (if not already created)
  $ cd .ssh
  ```

5. 建立名為 **authorized_keys** 的檔案。 在此檔案中貼上金鑰的內容，然後儲存檔案。

  > [!NOTE]
  > 金鑰中必須包含字串 `ssh-rsa`。 此外，金鑰的內容必須是單行文字。
  >  

6. 啟動 PuTTY。 在 [類別] 窗格中，選取 [連線] > [SSH] > [驗證]。在 [用於驗證的私密金鑰檔] 方塊中，瀏覽至您稍早產生的金鑰。

  ![[設定私密金鑰] 頁面上的螢幕擷取畫面](./media/oracle-golden-gate/setprivatekey.png)

7. 在 [類別] 窗格中，選取 [連線] > [SSH] > [X11]。 然後選取 [啟用 X11 轉送] 方塊。

  ![[啟用 X11] 頁面的螢幕擷取畫面](./media/oracle-golden-gate/enablex11.png)

8. 在 [類別] 窗格中，移至 [工作階段]。 輸入主機資訊，然後選取 [開啟]。

  ![工作階段分頁的螢幕擷取畫面](./media/oracle-golden-gate/puttysession.png)

### <a name="install-golden-gate-software"></a>安裝 Golden Gate 軟體

若要安裝 Oracle Golden Gate，請完成下列步驟：

1. 以 oracle 的身分登入。 (您應該能夠順利登入，而不會收到需要輸入密碼的提示)。請確定 Xming 已在執行，然後才開始安裝。
 
  ```bash
  $ cd /opt/fbo_ggs_Linux_x64_shiphome/Disk1
  $ ./runInstaller
  ```
2. 選取 'Oracle GoldenGate for Oracle Database 12c'。 然後選取 [下一步] 以繼續操作。

  ![安裝程式之 [選取安裝] 分頁的螢幕擷取畫面](./media/oracle-golden-gate/golden_gate_install_01.png)

3. 變更軟體位置。 然後選取 [啟動管理員] 方塊並輸入資料庫位置。 選取 [下一步] 以繼續操作。

  ![[選取安裝] 分頁的螢幕擷取畫面](./media/oracle-golden-gate/golden_gate_install_02.png)

4. 變更清查目錄，然後選取 [下一步] 以繼續操作。

  ![[選取安裝] 分頁的螢幕擷取畫面](./media/oracle-golden-gate/golden_gate_install_03.png)

5. 在 [摘要] 畫面上，選取 [安裝] 以繼續操作。

  ![安裝程式之 [選取安裝] 分頁的螢幕擷取畫面](./media/oracle-golden-gate/golden_gate_install_04.png)

6. 系統可能會提示您以 'root' 的身分執行指令碼。 如果是這樣，請開啟不同的工作階段，ssh 為 VM、sudo 為 root，然後執行指令碼。 選取 [確定] 以繼續操作。

  ![[選取安裝] 分頁的螢幕擷取畫面](./media/oracle-golden-gate/golden_gate_install_05.png)

7. 當安裝完成時，選取 [關閉] 以完成流程。

  ![[選取安裝] 分頁的螢幕擷取畫面](./media/oracle-golden-gate/golden_gate_install_06.png)

### <a name="set-up-service-on-myvm1-primary"></a>設定 myVM1 (主要) 上的服務

1. 建立或更新 tnsnames.ora 檔案：

  ```bash
  $ cd $ORACLE_HOME/network/admin
  $ vi tnsnames.ora

  cdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=cdb1)
      )
    )

  pdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=pdb1)
      )
    )
  ```

2. 建立 Golden Gate 擁有者和使用者帳戶。

  > [!NOTE]
  > 擁有者帳戶必須有 C## 前置詞。
  >

    ```bash
    $ sqlplus / as sysdba
    SQL> CREATE USER C##GGADMIN identified by ggadmin;
    SQL> EXEC dbms_goldengate_auth.grant_admin_privilege('C##GGADMIN',container=>'ALL');
    SQL> GRANT DBA to C##GGADMIN container=all;
    SQL> connect C##GGADMIN/ggadmin
    SQL> ALTER SESSION SET CONTAINER=PDB1;
    SQL> EXIT;
    ```

3. 建立 Golden Gate 測試使用者帳戶：

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ sqlplus system/OraPasswd1@pdb1
  SQL> CREATE USER test identified by test DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
  SQL> GRANT connect, resource, dba TO test;
  SQL> ALTER USER test QUOTA 100M on USERS;
  SQL> connect test/test@pdb1
  SQL> @demo_ora_create
  SQL> @demo_ora_insert
  SQL> EXIT;
  ```

4. 設定擷取參數檔案。

 啟動 Golden Gate 命令列介面 (ggsci)：

  ```bash
  $ sudo su - oracle
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ ./ggsci
  GGSCI> DBLOGIN USERID test@pdb1, PASSWORD test
  Successfully logged into database  pdb1
  GGSCI>  ADD SCHEMATRANDATA pdb1.test
  2017-05-23 15:44:25  INFO    OGG-01788  SCHEMATRANDATA has been added on schema test.
  2017-05-23 15:44:25  INFO    OGG-01976  SCHEMATRANDATA for scheduling columns has been added on schema test.

  GGSCI> EDIT PARAMS EXTORA
  ```
5. 將下列項目新增至 EXTRACT 參數檔案 (使用 vi 命令)。 按下 Esc 鍵，':wq!' 以儲存檔案。 

  ```bash
  EXTRACT EXTORA
  USERID C##GGADMIN, PASSWORD ggadmin
  RMTHOST 10.0.0.5, MGRPORT 7809
  RMTTRAIL ./dirdat/rt  
  DDL INCLUDE MAPPED
  DDLOPTIONS REPORT 
  LOGALLSUPCOLS
  UPDATERECORDFORMAT COMPACT
  TABLE pdb1.test.TCUSTMER;
  TABLE pdb1.test.TCUSTORD;
  ```
6. 註冊 extract--integrated 擷取：

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ ./ggsci

  GGSCI> dblogin userid C##GGADMIN, password ggadmin
  Successfully logged into database CDB$ROOT.

  GGSCI> REGISTER EXTRACT EXTORA DATABASE CONTAINER(pdb1)

  2017-05-23 15:58:34  INFO    OGG-02003  Extract EXTORA successfully registered with database at SCN 1821260.

  GGSCI> exit
  ```
7. 設定擷取檢查點，並啟動即時擷取：

  ```bash
  $ ./ggsci
  GGSCI>  ADD EXTRACT EXTORA, INTEGRATED TRANLOG, BEGIN NOW
  EXTRACT (Integrated) added.

  GGSCI>  ADD RMTTRAIL ./dirdat/rt, EXTRACT EXTORA, MEGABYTES 10
  RMTTRAIL added.

  GGSCI>  START EXTRACT EXTORA

  Sending START request to MANAGER ...
  EXTRACT EXTORA starting

  GGSCI > info all

  Program     Status      Group       Lag at Chkpt  Time Since Chkpt

  MANAGER     RUNNING
  EXTRACT     RUNNING     EXTORA      00:00:11      00:00:04
  ```
在此步驟中，您會找到開始 SCN，將會在稍後於不同區段中使用：

  ```bash
  $ sqlplus / as sysdba
  SQL> alter session set container = pdb1;
  SQL> SELECT current_scn from v$database;
  CURRENT_SCN
  -----------
      1857887
  SQL> EXIT;
  ```

  ```bash
  $ ./ggsci
  GGSCI> EDIT PARAMS INITEXT
  ```

  ```bash
  EXTRACT INITEXT
  USERID C##GGADMIN, PASSWORD ggadmin
  RMTHOST 10.0.0.5, MGRPORT 7809
  RMTTASK REPLICAT, GROUP INITREP
  TABLE pdb1.test.*, SQLPREDICATE 'AS OF SCN 1857887'; 
  ```

  ```bash
  GGSCI> ADD EXTRACT INITEXT, SOURCEISTABLE
  ```

### <a name="set-up-service-on-myvm2-replicate"></a>設定 myVM2 (複寫) 上的服務


1. 建立或更新 tnsnames.ora 檔案：

  ```bash
  $ cd $ORACLE_HOME/network/admin
  $ vi tnsnames.ora

  cdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=cdb1)
      )
    )

  pdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=pdb1)
      )
    )
  ```

2. 建立複寫帳戶：

  ```bash
  $ sqlplus / as sysdba
  SQL> alter session set container = pdb1;
  SQL> create user repuser identified by rep_pass container=current;
  SQL> grant dba to repuser;
  SQL> exec dbms_goldengate_auth.grant_admin_privilege('REPUSER',container=>'PDB1');
  SQL> connect repuser/rep_pass@pdb1 
  SQL> EXIT;
  ```

3. 建立 Golden Gate 測試使用者帳戶：

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ sqlplus system/OraPasswd1@pdb1
  SQL> CREATE USER test identified by test DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
  SQL> GRANT connect, resource, dba TO test;
  SQL> ALTER USER test QUOTA 100M on USERS;
  SQL> connect test/test@pdb1
  SQL> @demo_ora_create
  SQL> EXIT;
  ```

4. REPLICAT 參數檔案以複寫變更： 

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ ./ggsci
  GGSCI> EDIT PARAMS REPORA  
  ```
  REPORA 參數檔案的內容：

  ```bash
  REPLICAT REPORA
  ASSUMETARGETDEFS
  DISCARDFILE ./dirrpt/repora.dsc, PURGE, MEGABYTES 100
  DDL INCLUDE MAPPED
  DDLOPTIONS REPORT
  DBOPTIONS INTEGRATEDPARAMS(parallelism 6)
  USERID repuser@pdb1, PASSWORD rep_pass
  MAP pdb1.test.*, TARGET pdb1.test.*;
  ```

5. 設定複寫檢查點：

  ```bash
  GGSCI> ADD REPLICAT REPORA, INTEGRATED, EXTTRAIL ./dirdat/rt
  GGSCI> EDIT PARAMS INITREP

  ```

  ```bash
  REPLICAT INITREP
  ASSUMETARGETDEFS
  DISCARDFILE ./dirrpt/tcustmer.dsc, APPEND
  USERID repuser@pdb1, PASSWORD rep_pass
  MAP pdb1.test.*, TARGET pdb1.test.*;   
  ```

  ```bash
  GGSCI> ADD REPLICAT INITREP, SPECIALRUN
  ```

### <a name="set-up-the-replication-myvm1-and-myvm2"></a>設定複寫 (myVM1 和 myVM2)

#### <a name="1-set-up-the-replication-on-myvm2-replicate"></a>1.設定 myVM2 (複寫) 上的複寫

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ ./ggsci
  GGSCI> EDIT PARAMS MGR
  ```
使用下列內容更新檔案：

  ```bash
  PORT 7809
  ACCESSRULE, PROG *, IPADDR *, ALLOW
  ```
然後重新啟動管理員服務：

  ```bash
  GGSCI> STOP MGR
  GGSCI> START MGR
  GGSCI> EXIT
  ```

#### <a name="2-set-up-the-replication-on-myvm1-primary"></a>2.設定 myVM1 (主要) 上的複寫

啟動初始載入並且檢查錯誤：

```bash
$ cd /u01/app/oracle/product/12.1.0/oggcore_1
$ ./ggsci
GGSCI> START EXTRACT INITEXT
GGSCI> VIEW REPORT INITEXT
```
#### <a name="3-set-up-the-replication-on-myvm2-replicate"></a>3.設定 myVM2 (複寫) 上的複寫

使用您之前取得的數字變更 SCN 編號：

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ ./ggsci
  START REPLICAT REPORA, AFTERCSN 1857887
  ```
複寫已開始進行，您可以將新記錄插入測試資料表，以進行測試。


### <a name="view-job-status-and-troubleshooting"></a>檢視作業狀態和疑難排解

#### <a name="view-reports"></a>檢視報告
若要檢視 myVM1 上的報告，請執行下列命令：

  ```bash
  GGSCI> VIEW REPORT EXTORA 
  ```
 
若要檢視 myVM2 上的報告，請執行下列命令：

  ```bash
  GGSCI> VIEW REPORT REPORA
  ```

#### <a name="view-status-and-history"></a>檢視狀態和記錄
若要檢視 myVM1 上的狀態和記錄，請執行下列命令：

  ```bash
  GGSCI> dblogin userid c##ggadmin, password ggadmin 
  GGSCI> INFO EXTRACT EXTORA, DETAIL
  ```

若要檢視 myVM2 上的狀態和記錄，請執行下列命令：

  ```bash
  GGSCI> dblogin userid repuser@pdb1 password rep_pass 
  GGSCI> INFO REP REPORA, DETAIL
  ```
這樣便已完成 Oracle Linux 上的 Golden Gate 安裝和設定。


## <a name="delete-the-virtual-machine"></a>刪除虛擬機器

若不再需要，您可以使用下列命令來移除資源群組、VM 和所有相關資源。

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

[建立高可用性的虛擬機器教學課程](../../linux/create-cli-complete.md)

[瀏覽 VM 部署 CLI 範例](../../linux/cli-samples.md)
