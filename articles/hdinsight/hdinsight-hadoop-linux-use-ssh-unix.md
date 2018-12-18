---
title: 使用 SSH 搭配 Hadoop - Azure HDInsight
description: 您可以使用安全殼層 (SSH) 存取 HDInsight。 本文提供從 Windows、Linux、Unix 或 macOS 用戶端使用 ssh 和 scp 命令連線到 HDInsight 的資訊。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
keywords: linux 中的 hadoop 命令,hadoop linux 命令,hadoop macos,ssh hadoop,ssh hadoop 叢集
ms.service: hdinsight
ms.topic: conceptual
ms.date: 04/26/2018
ms.author: jasonh
ms.custom: H1Hack27Feb2017,hdinsightactive,hdiseo17may2017
ms.openlocfilehash: 7054bf34fea5b499fdec167667242d918cc01c8e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981694"
---
# <a name="connect-to-hdinsight-hadoop-using-ssh"></a>使用 SSH 連線到 HDInsight (Hadoop)

了解如何使用[安全殼層 (SSH)](https://en.wikipedia.org/wiki/Secure_Shell) 安全地連線到 Hadoop on Azure HDInsight。 

HDInsight 可以使用 Linux (Ubuntu) 作為 Hadoop 叢集節點的作業系統。 下表包含使用 SSH 用戶端連線到以 Linux 為基礎的 HDInsight 時所需的位址和連接埠資訊︰

| 位址 | 連接埠 | 連線到... |
| ----- | ----- | ----- |
| `<clustername>-ed-ssh.azurehdinsight.net` | 22 | 邊緣節點 (HDInsight 上的 ML 服務) |
| `<edgenodename>.<clustername>-ssh.azurehdinsight.net` | 22 | 邊緣節點 (任何其他叢集類型，如果邊緣節點存在的話) |
| `<clustername>-ssh.azurehdinsight.net` | 22 | 主要前端節點 |
| `<clustername>-ssh.azurehdinsight.net` | 23 | 次要前端節點 |

> [!NOTE]
> 將 `<edgenodename>` 替換為邊緣節點的名稱。
>
> 使用您叢集的名稱取代 `<clustername>`。
>
> 如果您的叢集包含邊緣節點，我們建議您__一律使用 SSH 連線到邊緣節點__。 前端節點會裝載對於 Hadoop 健康狀態至關重要的服務。 邊緣節點則只會執行您放在上面的服務。
>
> 如需使用邊緣節點的詳細資訊，請參閱[在 HDInsight 中使用邊緣節點](hdinsight-apps-use-edge-node.md#access-an-edge-node)。

> [!TIP]
> 當您初次連線至 HDInsight，您的 SSH 用戶端可能會顯示警告，指出無法建立主機的真確性。 在系統提示時，選取 [是] 將主機新增至您的 SSH 用戶端信任的伺服器清單。
>
> 如果您之前曾連線至相同名稱的伺服器，您可能會收到警告，指出預存的主機金鑰與伺服器的主機金鑰不符。 如需了解如何移除現有的伺服器名稱項目，請參閱您的 SSH 用戶端文件。

## <a name="ssh-clients"></a>SSH 用戶端

Linux、Unix 和 macOS 系統提供 `ssh` 和 `scp` 命令。 `ssh` 用戶端通常用來建立以 Linux 或 Unix 為基礎之系統的遠端命令列工作階段。 `scp` 用戶端用來安全地複製用戶端與遠端系統之間的檔案。

Microsoft Windows 預設不會安裝任何 SSH 用戶端。 `ssh` 和 `scp` 用戶端均可透過下列套件使用於 Windows︰

* OpenSSH 用戶端搶鮮版 (Beta)：在 Fall Creators Update 中，移至 [設定] > [應用程式和功能] > [管理選擇性功能] > [新增功能] 以及選取 [OpenSSH 用戶端]。 

    > [!NOTE]
    > 如果啟用這項功能之後，無法在 PowerShell 中使用 `ssh` 和 `scp` 命令，請登出而後重新登入。

* [位於 Windows 10 之 Ubuntu 上的 Bash](https://msdn.microsoft.com/commandline/wsl/about)：`ssh` 和`scp` 命令可透過 Windows 命令列上的 Bash 來取得。

* [OpenSSH 用戶端 (搶鮮版 (Beta))](https://blogs.msdn.microsoft.com/powershell/2017/12/15/using-the-openssh-beta-in-windows-10-fall-creators-update-and-windows-server-1709/)：這是在 Windows 10 Fall Creators Update 中導入的選用功能。

* [Azure Cloud Shell](../cloud-shell/quickstart.md)：Cloud Shell 在瀏覽器中提供 Bash 環境，並提供`ssh``scp` 和其他一般 Linux 命令。

* [Git (https://git-scm.com/)](https://git-scm.com/)：`ssh` 和 `scp` 命令可透過 GitBash 命令列來取得。

另有數個圖形化的 SSH 用戶端，如 [PuTTY (http://www.chiark.greenend.org.uk/~sgtatham/putty/)](http://www.chiark.greenend.org.uk/~sgtatham/putty/) 和 [MobaXterm (http://mobaxterm.mobatek.net/)](http://mobaxterm.mobatek.net/)。 雖然這些用戶端可用來連線到 HDInsight，但連線的程序與使用 `ssh` 公用程式時不同。 如需詳細資訊，請參閱您使用之圖形化用戶端的文件。

## <a id="sshkey"></a>驗證︰SSH 金鑰

SSH 金鑰會使用[公開金鑰加密](https://en.wikipedia.org/wiki/Public-key_cryptography)來驗證 SSH 工作階段。 SSH 金鑰比密碼更安全，並提供簡單的方式來保護 Hadoop 叢集的存取。

如果您使用金鑰來保護 SSH 帳戶，當您連線時，用戶端必須提供對應的私密金鑰︰

* 大部分用戶端均可設定為使用__預設金鑰__。 例如，`ssh` 用戶端會在 Linux 和 Unix 環境的 `~/.ssh/id_rsa` 中尋找私密金鑰。

* 您可以指定__私密金鑰的路徑__。 使用 `ssh` 用戶端時，您可以使用 `-i` 參數來指定私密金鑰的路徑。 例如： `ssh -i ~/.ssh/id_rsa sshuser@myedge.mycluster-ssh.azurehdinsight.net`。

* 如果您有__多個私密金鑰__可搭配不同的伺服器使用，請考慮使用 [ssh-agent (https://en.wikipedia.org/wiki/Ssh-agent)](https://en.wikipedia.org/wiki/Ssh-agent) 之類的公用程式。 `ssh-agent` 公用程式可用來自動選取在建立 SSH 工作階段時所要使用的金鑰。

> [!IMPORTANT]
>
> 如果您使用複雜密碼來保護私密金鑰，您必須在使用金鑰時輸入複雜密碼。 `ssh-agent` 之類的公用程式可以快取密碼以方便您使用。

### <a name="create-an-ssh-key-pair"></a>建立 SSH 金鑰組

使用 `ssh-keygen` 命令來建立公開和私密金鑰檔案。 下列命令會產生 2048 位元的 RSA 金鑰組，以供搭配 HDInsight 使用︰

    ssh-keygen -t rsa -b 2048

在金鑰建立程序期間，系統會提示您提供資訊。 例如，金鑰的儲存位置或是否要使用複雜密碼。 程序完成之後，系統會建立兩個檔案：公開金鑰和私密金鑰。

* __公開金鑰__可用來建立 HDInsight 叢集。 公開金鑰的副檔名為 `.pub`。

* __私密金鑰__可用來向 HDInsight 叢集驗證用戶端。

> [!IMPORTANT]
> 您可以使用複雜密碼來保護金鑰。 複雜密碼實際上就是私密金鑰的密碼。 即使有人取得您的私密金鑰，他們必須有複雜密碼才能使用該金鑰。

### <a name="create-hdinsight-using-the-public-key"></a>使用公開金鑰建立 HDInsight

| 建立方法 | 如何使用公開金鑰 |
| ------- | ------- |
| **Azure 入口網站** | 取消核取 [使用與叢集登入相同的密碼]，然後選取 [公開金鑰] 作為 SSH 驗證類型。 最後，選取公開金鑰檔案，或將檔案的文字內容貼到 [SSH 公開金鑰] 欄位。</br>![建立 HDInsight 叢集時的 [SSH 公開金鑰] 對話方塊](./media/hdinsight-hadoop-linux-use-ssh-unix/create-hdinsight-ssh-public-key.png) |
| **Azure PowerShell** | 使用 `New-AzureRmHdinsightCluster` Cmdlet 的 `-SshPublicKey` 參數，並以字串形式傳遞公開金鑰的內容。|
| **Azure 傳統 CLI** | 使用 `azure hdinsight cluster create` 命令的 `--sshPublicKey` 參數，並以字串形式傳遞公開金鑰的內容。 |
| **Resource Manager 範本** | 如需對範本使用 SSH 金鑰的範例，請參閱[使用 SSH 金鑰在 Linux 上部署 HDInsight](https://azure.microsoft.com/resources/templates/101-hdinsight-linux-ssh-publickey/)。 [azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/101-hdinsight-linux-ssh-publickey/azuredeploy.json) 檔案中的 `publicKeys` 元素可用來在建立叢集時將金鑰傳遞至 Azure。 |

## <a id="sshpassword"></a>驗證︰密碼

您可以使用密碼來保護 SSH 帳戶。 當您使用 SSH 連線到 HDInsight 時，系統會提示您輸入密碼。

> [!WARNING]
> Microsoft 不建議對 SSH 使用密碼驗證。 密碼可以猜到，因此很容易遭受暴力密碼破解攻擊。 相反地，我們會建議您使用 [SSH 金鑰來進行驗證](#sshkey)。

> [!IMPORTANT]
> SSH 帳戶密碼會在 HDInsight 叢集建立後 70 天到期。 如果密碼到期，您可以使用[管理 HDInsight](hdinsight-administer-use-portal-linux.md#change-passwords) 文件中的資訊更換。

### <a name="create-hdinsight-using-a-password"></a>使用密碼建立 HDInsight

| 建立方法 | 如何指定密碼 |
| --------------- | ---------------- |
| **Azure 入口網站** | 根據預設，SSH 使用者帳戶會具有和叢集登入帳戶相同的密碼。 若要使用不同的密碼，請取消核取 [使用與叢集登入相同的密碼]，然後在 [SSH 密碼] 欄位中輸入密碼。</br>![建立 HDInsight 叢集時的 [SSH 密碼] 對話方塊](./media/hdinsight-hadoop-linux-use-ssh-unix/create-hdinsight-ssh-password.png)|
| **Azure PowerShell** | 使用 `New-AzureRmHdinsightCluster` Cmdlet 的 `--SshCredential` 參數，並傳遞包含 SSH 使用者帳戶名稱和密碼的 `PSCredential` 物件。 |
| **Azure 傳統 CLI** | 使用 `azure hdinsight cluster create` 命令的 `--sshPassword` 參數，並提供密碼值。 |
| **Resource Manager 範本** | 如需對範本使用密碼的範例，請參閱[使用 SSH 密碼在 Linux 上部署 HDInsight](https://azure.microsoft.com/resources/templates/101-hdinsight-linux-ssh-password/)。 [azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/101-hdinsight-linux-ssh-password/azuredeploy.json) 檔案中的 `linuxOperatingSystemProfile` 元素可用來在建立叢集時將 SSH 帳戶名稱和密碼傳遞至 Azure。|

### <a name="change-the-ssh-password"></a>變更 SSH 密碼

如需有關變更 SSH 使用者帳戶密碼的資訊，請參閱[管理 HDInsight](hdinsight-administer-use-portal-linux.md#change-passwords) 文件的__變更密碼__一節。

## <a id="domainjoined"></a>驗證︰已加入網域的 HDInsight

如果您使用__已加入網域的 HDInsight 叢集__，您必須在使用 SSH 本機使用者連線之後使用 `kinit` 命令。 此命令會提示您輸入網域使用者和密碼，並向與叢集相關聯的 Azure Active Directory 網域驗證您的工作階段。

您也可以在每個加入網域的節點 (例如前端節點、邊緣節點) 上啟用 Kerberos 驗證，以便使用網域帳戶進行 ssh 連線。 若要進行此作業，請編輯 sshd 組態檔：
```bash
sudo vi /etc/ssh/sshd_config
```
取消註解，並將 `KerberosAuthentication` 變更為 `yes`

```bash
sudo service sshd restart
```

在任何時間，若要確認 Kerberos 驗證是否已成功，請使用 `klist` 命令。

如需詳細資訊，請參閱[設定已加入網域的 HDInsight](./domain-joined/apache-domain-joined-configure.md)。

## <a name="connect-to-nodes"></a>連線到節點

可在連接埠 22 和 23 上透過網際網路存取前端節點和邊緣節點 (如果有的話)。

* 連線到__前端節點__時，請使用連接埠 __22__ 連線到主要前端節點，以及使用連接埠 __23__ 連線到次要前端節點。 要使用的完整網域名稱為 `clustername-ssh.azurehdinsight.net`，其中 `clustername` 是您的叢集名稱。

    ```bash
    # Connect to primary head node
    # port not specified since 22 is the default
    ssh sshuser@clustername-ssh.azurehdinsight.net

    # Connect to secondary head node
    ssh -p 23 sshuser@clustername-ssh.azurehdinsight.net
    ```
    
* 連線到__邊緣節點__時，請使用連接埠 22。 完整網域名稱為 `edgenodename.clustername-ssh.azurehdinsight.net`，其中 `edgenodename` 是您建立邊緣節點時提供的名稱。 `clustername` 是叢集的名稱。

    ```bash
    # Connect to edge node
    ssh sshuser@edgnodename.clustername-ssh.azurehdinsight.net
    ```

> [!IMPORTANT]
> 先前的範例假設您使用密碼驗證，或該憑證驗證自動發生。 如果您使用 SSH 金鑰組進行驗證，但未自動使用憑證，請使用 `-i` 參數來指定私密金鑰。 例如： `ssh -i ~/.ssh/mykey sshuser@clustername-ssh.azurehdinsight.net`。

連線之後，字元會變更，以指示 SSH 使用者名稱和您所連線的節點。 例如，以 `sshuser` 身分連線到主要前端節點時，提示為 `sshuser@hn0-clustername:~$`。

### <a name="connect-to-worker-and-zookeeper-nodes"></a>連線至背景工作角色和 Zookeeper 節點

無法從網際網路直接存取背景工作角色節點和 Zookeeper 節點。 從叢集前端節點或邊緣節點即可加以存取。 以下是連接至其他節點的一般步驟：

1. 使用 SSH 連接前端或邊緣節點：

        ssh sshuser@myedge.mycluster-ssh.azurehdinsight.net

2. 從 SSH 到前端或邊緣節點的連線，使用 `ssh` 命令來連接叢集中的背景工作角色節點︰

        ssh sshuser@wn0-myhdi

    若要擷取節點名稱清單，請參閱[使用 Ambari REST API 管理 HDInsight](hdinsight-hadoop-manage-ambari-rest-api.md#example-get-the-fqdn-of-cluster-nodes) 文件。

如果使用__密碼__來保護 SSH 帳戶，請在連線時輸入密碼。

如果使用 __SSH 金鑰__來保護 SSH 帳戶，請確定用戶端上的 SSH 轉送已啟用。

> [!NOTE]
> 若要直接存取叢集中的所有節點，另一種方式是將 HDInsight 安裝到 Azure 虛擬網路。 然後，您可以將遠端機器加入相同的虛擬網路，並直接存取叢集中的所有節點。
>
> 如需詳細資訊，請參閱[搭配 HDInsight 使用虛擬網路](hdinsight-extend-hadoop-virtual-network.md)。

### <a name="configure-ssh-agent-forwarding"></a>設定 SSH 代理程式轉送

> [!IMPORTANT]
> 以下步驟假設您使用以 Linux 或 UNIX 為基礎的系統，並使用 Bash on Windows 10 進行操作。 如果這些步驟不適用於您的系統，您可能需要參閱 SSH 用戶端的文件。

1. 使用文字編輯器開啟 `~/.ssh/config`。 如果該檔案不存在，您可以藉由在命令列中輸入 `touch ~/.ssh/config` 加以建立。

2. 將下列文字新增至 `config` 檔案。

        Host <edgenodename>.<clustername>-ssh.azurehdinsight.net
          ForwardAgent yes

    將 __Host__ 資訊替換為您使用 SSH 連線到的節點位址。 先前的範例使用邊緣節點。 這個項目會為指定節點設定 SSH 代理程式轉送。

3. 從終端機使用下列命令，測試 SSH 代理程式轉送：

        echo "$SSH_AUTH_SOCK"

    此命令會傳回類似以下文字的資訊：

        /tmp/ssh-rfSUL1ldCldQ/agent.1792

    如果未傳回任何資訊，表示 `ssh-agent` 未執行。 如需詳細資訊，請參閱[透過 ssh 使用 ssh-agent (http://mah.everybody.org/docs/ssh)](http://mah.everybody.org/docs/ssh)) 的代理程式啟動指令碼資訊，或參閱 SSH 用戶端文件。

4. 一旦確認 **ssh-agent** 正在執行，請使用下列項目將您的 SSH 私密金鑰新增至代理程式：

        ssh-add ~/.ssh/id_rsa

    如果您的私密金鑰儲存在不同的檔案，請將 `~/.ssh/id_rsa` 取代為檔案的路徑。

5. 使用 SSH 連線到叢集的邊緣節點或前端節點。 然後，使用 SSH 命令連線到背景工作角色或 Zookeeper 節點。 該連線會使用轉送的金鑰來建立。

## <a name="copy-files"></a>複製檔案

`scp` 公用程式可用於雙向複製叢集中個別節點的檔案。 例如，下列命令可將 `test.txt` 目錄從本機系統複製到主要前端節點：

```bash
scp test.txt sshuser@clustername-ssh.azurehdinsight.net:
```

因為未在 `:` 之後指定路徑，所以此檔案會置於 `sshuser` 主目錄中。

下列範例可將 `test.txt` 檔案從主要前端節點上的 `sshuser` 主目錄複製到本機系統：

```bash
scp sshuser@clustername-ssh.azurehdinsight.net:test.txt .
```

> [!IMPORTANT]
> `scp` 只能存取叢集內個別節點的檔案系統。 它不能用來存取叢集的 HDFS 相容儲存體中的資料。
>
> 當您需要從 SSH 工作階段上傳資源以供使用時，請使用 `scp`。 例如，上傳 Python 指令碼，然後從 SSH 工作階段執行指令碼。
>
> 如需直接將資料載入 HDFS 相容儲存體的資訊，請參閱下列文件：
>
> * [使用 Azure 儲存體的 HDInsight](hdinsight-hadoop-use-blob-storage.md)。
>
> * [使用 Azure Data Lake Store 的 HDInsight](hdinsight-hadoop-use-data-lake-store.md)。

## <a name="next-steps"></a>後續步驟

* [對 HDInsight 使用 SSH 通道](hdinsight-linux-ambari-ssh-tunnel.md)
* [對 HDInsight 使用虛擬網路](hdinsight-extend-hadoop-virtual-network.md)
* [在 HDInsight 中使用邊緣節點](hdinsight-apps-use-edge-node.md#access-an-edge-node)
