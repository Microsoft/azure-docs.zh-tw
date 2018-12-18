---
title: 在 Azure Batch 中的使用者帳戶執行工作 | Microsoft Docs
description: 設定在 Azure Batch 中執行工作所需的使用者帳戶
services: batch
author: dlepow
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 05/22/2017
ms.author: danlep
ms.openlocfilehash: d5ec76a62b56769ee3065cac3542f5a94df4a1c6
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37133257"
---
# <a name="run-tasks-under-user-accounts-in-batch"></a>在 Batch 中的使用者帳戶執行工作

Azure Batch 中的工作一律會在使用者帳戶下執行。 根據預設，會在標準使用者帳戶中執行工作，而不需要系統管理員權限。 這些預設使用者帳戶設定通常已足夠。 不過，在特定案例中，最好能夠在您想要執行工作的使用者帳戶進行設定。 本文討論使用者帳戶的類型，以及如何針對您的案例來設定它們。

## <a name="types-of-user-accounts"></a>使用者帳戶的類型

Azure Batch 提供執行工作所需的兩種使用者帳戶類型︰

- **自動使用者帳戶。** 自動使用者帳戶是由 Batch 服務自動建立的內建使用者帳戶。 根據預設，工作會在自動使用者帳戶下執行。 您可以針對工作設定自動使用者規格，來表示工作應該在哪一個自動使用者帳戶下執行。 自動使用者規格可讓您指定執行工作之自動使用者帳戶的範圍及提高權限層級。 

- **具名的使用者帳戶。** 當您建立集區時，可以指定一或多個集區的具名使用者帳戶。 每個使用者帳戶都會建立在集區的每個節點上。 除了帳戶名稱之外，您會指定使用者帳戶密碼、提高權限層級，以及針對 Linux 集區指定 SSH 私密金鑰。 當您新增一項工作時，可以指定應執行該工作的具名使用者帳戶。

> [!IMPORTANT] 
> Batch 服務版本 2017-01-01.4.0 導入了重大變更，要求您更新程式碼以呼叫該版本。 如果您是從較舊版本的 Batch 移轉程式碼，請注意，REST API 或 Batch 用戶端程式庫不再支援 **runElevated** 屬性。 使用工作的新 **userIdentity** 屬性來指定提高權限層級。 如果您使用的是其中一個用戶端程式庫，請參閱標題為[將您的程式碼更新至最新的 Batch 用戶端程式庫](#update-your-code-to-the-latest-batch-client-library)的章節，以取得更新 Batch 程式碼的快速指導方針。
>
>

> [!NOTE] 
> 基於安全性考量，本文所討論的使用者帳戶不支援遠端桌面通訊協定 (RDP) 或安全殼層 (SSH)。 
>
> 若要連線到透過 SSH 執行 Linux 虛擬機器設定的節點，請參閱[在 Azure 中使用 Linux VM 的遠端桌面](../virtual-machines/virtual-machines-linux-use-remote-desktop.md)。 若要連線到透過 RDP 執行 Windows 的節點，請參閱[連線到 Windows Server VM](../virtual-machines/windows/connect-logon.md)。<br /><br />
> 若要連線到透過 RDP 執行雲端服務設定的節點，請參閱[在 Azure 雲端服務中啟用角色的遠端桌面連線](../cloud-services/cloud-services-role-enable-remote-desktop-new-portal.md)。
>
>

## <a name="user-account-access-to-files-and-directories"></a>使用者帳戶存取檔案和目錄

自動使用者帳戶和具名的使用者帳戶具有讀取/寫入權限，可存取工作的工作目錄、共用的目錄和多重執行個體的工作目錄。 這兩種類型的帳戶都具有啟動和作業準備工作目錄的讀取權限。

如果相同帳戶下執行的工作用來執行啟動工作，則工作具有啟動工作目錄的讀寫權限。 同樣地，如果相同帳戶下執行的工作用來執行作業準備工作，則工作具有作業準備工作目錄的讀寫權限。 如果是在與啟動工作或作業準備工作不同的帳戶下執行工作，則工作只有個別目錄的讀取權限。

如需從工作存取檔案和目錄的相關詳細資訊，請參閱[使用 Batch 開發大規模的平行計算解決方案](batch-api-basics.md#files-and-directories)。

## <a name="elevated-access-for-tasks"></a>提高工作的權限 

使用者帳戶的提高權限層級會表示工作是否使用提高權限的存取權來執行。 自動使用者帳戶和具名的使用者帳戶都可以使用提高權限的存取權來執行。 提高權限層級的兩個選項如下︰

- **NonAdmin：** 工作是以標準使用者身分執行，沒有提高權限的存取權。 Batch 使用者帳戶的預設提高權限層級一律為 **NonAdmin**。
- **系統管理員︰** 工作是以具有提高權限存取權的使用者身分執行，並以完整系統管理員權限運作。 

## <a name="auto-user-accounts"></a>自動使用者帳戶

根據預設，Batch 中的工作會在自動使用者帳戶下執行，以標準使用者身分執行，沒有提高權限的存取權，並具有工作範圍。 針對工作範圍設定自動使用者規格之後，Batch 服務只會建立該工作的自動使用者帳戶。

工作範圍的替代方案是集區範圍。 當針對集區範圍設定工作的自動使用者規格時，會在集區中任何工作可用的自動使用者帳戶下執行工作。 如需集區範圍的詳細資訊，請參閱標題為[以自動使用者身分在集區範圍中執行工作](#run-a-task-as-the-autouser-with-pool-scope)的章節。   

預設範圍在 Windows 和 Linux 節點上有所差異︰

- 在 Windows 節點上，工作根據預設會在工作範圍下執行。
- Linux 節點一律會在集區範圍下執行。

自動使用者規格有四個可能的設定，其中每一個都對應至唯一的自動使用者帳戶︰

- 具有工作範圍 (預設的自動使用者規格) 的非系統管理員存取權
- 具有工作範圍的系統管理員 (提升權限) 存取權
- 具有集區範圍的非系統管理員存取權
- 具有集區範圍的系統管理員存取權

> [!IMPORTANT] 
> 在工作範圍下執行的工作沒有節點上其他工作的既定存取權。 不過，具有帳戶存取權的惡意使用者只要提交以系統管理員權限執行的工作，並存取其他的工作目錄，便可以解決這項限制。 惡意使用者也可以使用 RDP 或 SSH 連線到節點。 請務必保護您 Batch 帳戶金鑰的存取權，以避免發生此情況。 如果您懷疑您的帳戶已遭入侵，請務必重新產生金鑰。
>
>

### <a name="run-a-task-as-an-auto-user-with-elevated-access"></a>以具有提高權限之存取權的自動使用者身分執行工作

當您需要以提高權限的存取權執行工作時，可以針對系統管理員權限設定自動使用者規格。 例如，啟動工作可能需要提高權限的存取權才可在節點上安裝軟體。

> [!NOTE] 
> 一般而言，最好只在必要時才使用提高權限的存取。 最佳做法建議授與達成所需結果所需的最低權限。 例如，如果啟動工作為目前的使用者而不是所有使用者安裝軟體，您可能會避免將提高權限的存取權授與給工作。 您可以針對集區範圍設定自動使用者規格，以及針對需要在相同帳戶中執行的所有工作 (包括啟動工作) 設定非系統管理員存取權。 
>
>

下列程式碼片段示範如何設定自動使用者規格。 範例會將提高權限層級設定為 `Admin`，以及將範圍設定為 `Task`。 工作範圍是預設設定，但是也包含在此作為範例。

#### <a name="batch-net"></a>Batch .NET

```csharp
task.UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.Admin, scope: AutoUserScope.Task));
```
#### <a name="batch-java"></a>Batch Java

```java
taskToAdd.withId(taskId)
        .withUserIdentity(new UserIdentity()
            .withAutoUser(new AutoUserSpecification()
                .withElevationLevel(ElevationLevel.ADMIN))
                .withScope(AutoUserScope.TASK));
        .withCommandLine("cmd /c echo hello");                        
```

#### <a name="batch-python"></a>Batch Python

```python
user = batchmodels.UserIdentity(
    auto_user=batchmodels.AutoUserSpecification(
        elevation_level=batchmodels.ElevationLevel.admin,
        scope=batchmodels.AutoUserScope.task))
task = batchmodels.TaskAddParameter(
    id='task_1',
    command_line='cmd /c "echo hello world"',
    user_identity=user)
batch_client.task.add(job_id=jobid, task=task)
```

### <a name="run-a-task-as-an-auto-user-with-pool-scope"></a>以具有集區範圍的自動使用者身分執行工作

當佈建節點時，會在集區的每個節點上建立兩個全集區的自動使用者帳戶，一個具有提高權限的存取權，另一個則沒有提高權限的存取權。 將自動使用者的範圍設為特定工作的集區範圍，會在兩個全集區的自動使用者帳戶其中之一執行工作。 

當您指定自動使用者的集區範圍時，使用系統管理員存取權執行的所有工作，會在相同的全集區自動使用者帳戶下執行。 同樣地，不具系統管理員權限執行的工作，也會在單一的全集區自動使用者帳戶下執行。 

> [!NOTE] 
> 這兩個全集區的自動使用者帳戶是不同的帳戶。 在全集區的系統管理帳戶下執行的工作，不能與標準帳戶下執行的工作共用資料，反之亦然。 
>
>

在相同自動使用者帳戶下執行的優點是，工作都能與相同節點上執行的其他工作共用資料。

在工作之間共用祕密這個案例中，在兩個全集區的自動使用者帳戶其中之一執行工作會很實用。 例如，假設啟動工作需要將祕密佈建到其他工作可以使用的節點上。 您可以使用 Windows 資料保護 API (DPAPI)，但它需要系統管理員權限。 相反地，您可以保護使用者層級的祕密。 在相同使用者帳戶下執行的工作，無需提高權限的存取即可存取祕密。

您可能要在集區範圍的自動使用者帳戶執行工作的另一種情況，是訊息傳遞介面 (MPI) 檔案共用。 當 MPI 工作中的節點必須使用相同的檔案資料時，MPI 檔案共用便很有用。 如果子節點是在相同的自動使用者帳戶下執行，前端節點會建立子節點可以存取的檔案共用。 

下列程式碼片段會將自動使用者的範圍設為 Batch .NET 中的工作集區範圍。 提高權限層級會予以省略，因此工作會在標準全集區的自動使用者帳戶下執行。

```csharp
task.UserIdentity = new UserIdentity(new AutoUserSpecification(scope: AutoUserScope.Pool));
```

## <a name="named-user-accounts"></a>具名的使用者帳戶

當您建立集區時，可以定義具名的使用者帳戶。 具名的使用者帳戶具有您提供的名稱和密碼。 您可以指定具名使用者帳戶的提高權限層級。 您也可以提供適用於 Linux 節點的 SSH 私密金鑰。

具名的使用者帳戶存在於集區中的所有節點上，且可在這些節點上執行的所有工作上使用。 您可以針對集區定義任意數目的使用者名稱。 當您新增工作或工作集合時，可以指定要在集區上定義之其中一個具名使用者帳戶下執行的工作。

當您想要在相同使用者帳戶下執行作業中的所有工作，而同時要將這些工作與其他作業中執行的工作隔離時，具名的使用者帳戶便很有用。 例如，您可以為每個作業建立具名的使用者，並在該具名的使用者帳戶下執行每個作業的工作。 接著，每個作業可與它自己的工作共用祕密，但不可與其他作業中執行的工作共用祕密。

您也可以使用具名的使用者帳戶來執行工作，其會設定外部資源 (例如檔案共用) 上的設定權限。 使用具名的使用者帳戶，您可以控制使用者身分識別，且可以使用該使用者身分識別來設定權限。  

具名的使用者帳戶會啟用 Linux 節點之間的無密碼 SSH。 您可以搭配使用具名的使用者帳戶與需要執行多個執行個體工作的 Linux 節點。 集區中的每個節點都可以在整個集區上定義的使用者帳戶內執行工作。 如需多個執行個體工作的詳細資訊，請參閱[使用多重\-執行個體工作來執行 MPI 應用程式](batch-mpi.md)。

### <a name="create-named-user-accounts"></a>建立具名的使用者帳戶

若要在 Batch 中建立具名使用者帳戶，請將使用者帳戶的集合新增至集區。 下列程式碼片段示範如何在 .NET、Java 和 Python 建立具名的使用者帳戶。 這些程式碼片段示範如何在集區上建立系統管理員和非系統管理員的具名帳戶。 範例會使用雲端服務設定來建立集區，但當您使用虛擬機器設定來建立 Windows 或 Linux 集區時，才使用相同的方法。

#### <a name="batch-net-example-windows"></a>Batch .NET 範例 (Windows)

```csharp
CloudPool pool = null;
Console.WriteLine("Creating pool [{0}]...", poolId);

// Create a pool using the cloud service configuration.
pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 3,
    virtualMachineSize: "standard_d1_v2",
    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));   

// Add named user accounts.
pool.UserAccounts = new List<UserAccount>
{
    new UserAccount("adminUser", "xyz123", ElevationLevel.Admin),
    new UserAccount("nonAdminUser", "123xyz", ElevationLevel.NonAdmin),
};

// Commit the pool.
await pool.CommitAsync();
```

#### <a name="batch-net-example-linux"></a>Batch .NET 範例 (Linux)

```csharp
CloudPool pool = null;

// Obtain a collection of all available node agent SKUs.
List<NodeAgentSku> nodeAgentSkus =
    batchClient.PoolOperations.ListNodeAgentSkus().ToList();

// Define a delegate specifying properties of the VM image to use.
Func<ImageReference, bool> isUbuntu1404 = imageRef =>
    imageRef.Publisher == "Canonical" &&
    imageRef.Offer == "UbuntuServer" &&
    imageRef.Sku.Contains("14.04");

// Obtain the first node agent SKU in the collection that matches
// Ubuntu Server 14.04. 
NodeAgentSku ubuntuAgentSku = nodeAgentSkus.First(sku =>
    sku.VerifiedImageReferences.Any(isUbuntu1404));

// Select an ImageReference from those available for node agent.
ImageReference imageReference =
    ubuntuAgentSku.VerifiedImageReferences.First(isUbuntu1404);

// Create the virtual machine configuration to use to create the pool.
VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration(imageReference, ubuntuAgentSku.Id);

Console.WriteLine("Creating pool [{0}]...", poolId);

// Create the unbound pool.
pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 3,                                             
    virtualMachineSize: "Standard_A1",                                      
    virtualMachineConfiguration: virtualMachineConfiguration);                  

// Add named user accounts.
pool.UserAccounts = new List<UserAccount>
{
    new UserAccount(
        name: "adminUser",
        password: "xyz123",
        elevationLevel: ElevationLevel.Admin,
        linuxUserConfiguration: new LinuxUserConfiguration(
            uid: 12345,
            gid: 98765,
            sshPrivateKey: new Guid().ToString()
            )),
    new UserAccount(
        name: "nonAdminUser",
        password: "123xyz",
        elevationLevel: ElevationLevel.NonAdmin,
        linuxUserConfiguration: new LinuxUserConfiguration(
            uid: 45678,
            gid: 98765,
            sshPrivateKey: new Guid().ToString()
            )),
};

// Commit the pool.
await pool.CommitAsync();
```


#### <a name="batch-java-example"></a>Batch Java 範例

```java
List<UserAccount> userList = new ArrayList<>();
userList.add(new UserAccount().withName(adminUserAccountName).withPassword(adminPassword).withElevationLevel(ElevationLevel.ADMIN));
userList.add(new UserAccount().withName(nonAdminUserAccountName).withPassword(nonAdminPassword).withElevationLevel(ElevationLevel.NONADMIN));
PoolAddParameter addParameter = new PoolAddParameter()
        .withId(poolId)
        .withTargetDedicatedNodes(POOL_VM_COUNT)
        .withVmSize(POOL_VM_SIZE)
        .withCloudServiceConfiguration(configuration)
        .withUserAccounts(userList);
batchClient.poolOperations().createPool(addParameter);
```

#### <a name="batch-python-example"></a>Batch Python 範例

```python
users = [
    batchmodels.UserAccount(
        name='pool-admin',
        password='******',
        elevation_level=batchmodels.ElevationLevel.admin)
    batchmodels.UserAccount(
        name='pool-nonadmin',
        password='******',
        elevation_level=batchmodels.ElevationLevel.nonadmin)
]
pool = batchmodels.PoolAddParameter(
    id=pool_id,
    user_accounts=users,
    virtual_machine_configuration=batchmodels.VirtualMachineConfiguration(
        image_reference=image_ref_to_use,
        node_agent_sku_id=sku_to_use),
    vm_size=vm_size,
    target_dedicated=vm_count)
batch_client.pool.add(pool)
```

### <a name="run-a-task-under-a-named-user-account-with-elevated-access"></a>在具有提高權限之存取權的具名使用者帳戶下執行工作

若要以提高權限的使用者身分執行工作，將工作的 **UserIdentity** 屬性設定為建立時將 **ElevationLevel** 屬性設定為 `Admin` 的具名使用者帳戶。

此程式碼片段會指定須在具名的使用者帳戶下執行工作。 集區建立時，會在集區上定義此具名的使用者帳戶。 在此情況下，會使用系統管理員權限來建立具名的使用者帳戶︰

```csharp
CloudTask task = new CloudTask("1", "cmd.exe /c echo 1");
task.UserIdentity = new UserIdentity(AdminUserAccountName);
```

## <a name="update-your-code-to-the-latest-batch-client-library"></a>將程式碼更新至最新的 Batch 用戶端程式庫

Batch 服務版本 2017-01-01.4.0 導入重大變更，將舊版中的 **runElevated** 屬性取代為 **userIdentity** 屬性。 下表提供的簡單對應可供您用來將舊版的用戶端程式庫的程式碼進行更新。

### <a name="batch-net"></a>Batch .NET

| 如果您的程式碼使用...                  | 將它更新為...                                                                                                 |
|---------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `CloudTask.RunElevated = true;`       | `CloudTask.UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.Admin));`    |
| `CloudTask.RunElevated = false;`      | `CloudTask.UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.NonAdmin));` |
| `CloudTask.RunElevated` 未指定 | 不需要更新                                                                                               |

### <a name="batch-java"></a>Batch Java

| 如果您的程式碼使用...                      | 將它更新為...                                                                                                                       |
|-------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| `CloudTask.withRunElevated(true);`        | `CloudTask.withUserIdentity(new UserIdentity().withAutoUser(new AutoUserSpecification().withElevationLevel(ElevationLevel.ADMIN));`    |
| `CloudTask.withRunElevated(false);`       | `CloudTask.withUserIdentity(new UserIdentity().withAutoUser(new AutoUserSpecification().withElevationLevel(ElevationLevel.NONADMIN));` |
| `CloudTask.withRunElevated` 未指定 | 不需要更新                                                                                                                     |

### <a name="batch-python"></a>Batch Python

| 如果您的程式碼使用...                      | 將它更新為...                                                                                                                       |
|-------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| `run_elevated=True`                       | `user_identity=user`，其中 <br />`user = batchmodels.UserIdentity(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`auto_user=batchmodels.AutoUserSpecification(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`elevation_level=batchmodels.ElevationLevel.admin)) `                |
| `run_elevated=False`                      | `user_identity=user`，其中 <br />`user = batchmodels.UserIdentity(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`auto_user=batchmodels.AutoUserSpecification(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`elevation_level=batchmodels.ElevationLevel.nonadmin)) `             |
| `run_elevated` 未指定 | 不需要更新                                                                                                                                  |


## <a name="next-steps"></a>後續步驟

* 如需 Batch 的深入概觀，請參閱[使用 Batch 開發大規模的平行計算解決方案](batch-api-basics.md)。