---
title: 在 Azure 中使用 Python 建立並管理 Windows VM | Microsoft Docs
description: 了解如何在 Azure 中使用 Python 建立並管理 Windows VM。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 06/22/2017
ms.author: cynthn
ms.openlocfilehash: dbe8f1603433f381c3c28cb47d2dbda543b462e0
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31528336"
---
# <a name="create-and-manage-windows-vms-in-azure-using-python"></a>在 Azure 中使用 Python 建立並管理 Windows VM

[Azure 虛擬機器](overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (VM) 需要數個支援的 Azure 資源。 本文涵蓋使用 Python 建立、管理和刪除 VM 資源。 您會了解如何：

> [!div class="checklist"]
> * 建立 Visual Studio 專案
> * 安裝套件
> * 建立認證
> * 建立資源
> * 執行管理工作
> * 刪除資源
> * 執行應用程式

執行這些步驟大約需要 20 分鐘的時間。

## <a name="create-a-visual-studio-project"></a>建立 Visual Studio 專案

1. 如果您尚未安裝 [Visual Studio](https://docs.microsoft.com/visualstudio/install/install-visual-studio)，請進行安裝。 在 [工作負載] 頁面上選取 [Python 開發]，然後按一下 [安裝]。 在摘要中，您可以看到會自動為您選擇 **Python 3 64 位元 (3.6.0)**。 如果您已安裝 Visual Studio，您可以使用 Visual Studio Launcher 新增 Python 工作負載。
2. 安裝並啟動 Visual Studio 後，按一下 [檔案] > [新增] > [專案]。
3. 按一下 [範本] > [Python] > [Python 應用程式]，輸入 myPythonProject 作為專案的名稱，選取專案的位置，然後按一下 [確定]。

## <a name="install-packages"></a>安裝套件

1. 在 [方案總管] 的 myPythonProject 下，以滑鼠右鍵按一下 [Python 環境]，然後選取 [新增虛擬環境]。
2. 在 [新增虛擬環境] 畫面上，接受 env 的預設名稱、確定選取 Python 3.6 (64 位元) 作為基底解譯器，然後按一下 [建立]。
3. 以滑鼠右鍵按一下您所建立的 env 環境、按一下 [安裝 Python 套件]、在搜尋方塊中輸入 azure，然後按 Enter 鍵。

您應該會在輸出視窗中看到已成功安裝的 azure 套件。 

## <a name="create-credentials"></a>建立認證

在開始此步驟之前，請確定您具有 [Active Directory 服務主體](../../azure-resource-manager/resource-group-create-service-principal-portal.md)。 您也應該記錄應用程式識別碼、驗證金鑰以及租用戶識別碼，您在稍後的步驟會需要這些項目。

1. 開啟所建立的 myPythonProject.py 檔案，然後新增此程式碼，讓您的應用程式可執行：

    ```python
    if __name__ == "__main__":
    ```

2. 若要匯入所需的程式碼，請將這些陳述式新增至 .py 檔案頂端：

    ```python
    from azure.common.credentials import ServicePrincipalCredentials
    from azure.mgmt.resource import ResourceManagementClient
    from azure.mgmt.compute import ComputeManagementClient
    from azure.mgmt.network import NetworkManagementClient
    from azure.mgmt.compute.models import DiskCreateOption
    ```

3. 接著在 .py 檔案中，在匯入陳述式之後新增變數，來指定用於程式碼中的一般值：
   
    ```
    SUBSCRIPTION_ID = 'subscription-id'
    GROUP_NAME = 'myResourceGroup'
    LOCATION = 'westus'
    VM_NAME = 'myVM'
    ```

    以您的訂用帳戶識別碼取代 **subscription-id**。

4. 若要建立您提出要求所需的 Active Directory 認證，請在 .py 檔案的變數之後新增此函式：

    ```python
    def get_credentials():
        credentials = ServicePrincipalCredentials(
            client_id = 'application-id',
            secret = 'authentication-key',
            tenant = 'tenant-id'
        )

        return credentials
    ```

    將 **application-id**、**authentication-key** 和 **tenant-id** 取代為您先前建立 Azure Active Directory 服務主體時收集的值。

5. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

    ```python
    credentials = get_credentials()
    ```

## <a name="create-resources"></a>建立資源
 
### <a name="initialize-management-clients"></a>初始化管理用戶端

需要管理用戶端才能在 Azure 中使用 Python SDK 建立及管理資源。 若要建立管理用戶端，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

```python
resource_group_client = ResourceManagementClient(
    credentials, 
    SUBSCRIPTION_ID
)
network_client = NetworkManagementClient(
    credentials, 
    SUBSCRIPTION_ID
)
compute_client = ComputeManagementClient(
    credentials, 
    SUBSCRIPTION_ID
)
```

### <a name="create-the-vm-and-supporting-resources"></a>建立 VM 和支援的資源

所有資源都必須包含在[資源群組](../../azure-resource-manager/resource-group-overview.md)中。

1. 若要建立資源群組，請在 .py 檔案的變數之後新增此函式：

    ```python
    def create_resource_group(resource_group_client):
        resource_group_params = { 'location':LOCATION }
        resource_group_result = resource_group_client.resource_groups.create_or_update(
            GROUP_NAME, 
            resource_group_params
        )
    ```

2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

    ```python
    create_resource_group(resource_group_client)
    input('Resource group created. Press enter to continue...')
    ```

維護您應用程式所使用的虛擬機器時，使用[可用性設定組](tutorial-availability-sets.md)可讓您更加輕鬆。

1. 若要建立可用性設定組，請在 .py 檔案的變數之後新增此函式：
   
    ```python
    def create_availability_set(compute_client):
        avset_params = {
            'location': LOCATION,
            'sku': { 'name': 'Aligned' },
            'platform_fault_domain_count': 3
        }
        availability_set_result = compute_client.availability_sets.create_or_update(
            GROUP_NAME,
            'myAVSet',
            avset_params
        )
    ```

2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

    ```python
    create_availability_set(compute_client)
    print("------------------------------------------------------")
    input('Availability set created. Press enter to continue...')
    ```

必須要有[公用 IP 位址](../../virtual-network/virtual-network-ip-addresses-overview-arm.md)才能與虛擬機器進行通訊。

1. 若要建立虛擬機器的公用 IP 位址，請在 .py 檔案的變數之後新增此函式：

    ```python
    def create_public_ip_address(network_client):
        public_ip_addess_params = {
            'location': LOCATION,
            'public_ip_allocation_method': 'Dynamic'
        }
        creation_result = network_client.public_ip_addresses.create_or_update(
            GROUP_NAME,
            'myIPAddress',
            public_ip_addess_params
        )

        return creation_result.result()
    ```

2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

    ```python
    creation_result = create_public_ip_address(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

虛擬機器必須在[虛擬網路](../../virtual-network/virtual-networks-overview.md)的子網路中。

1. 若要建立虛擬網路，請在 .py 檔案的變數之後新增此函式：

    ```python
    def create_vnet(network_client):
        vnet_params = {
            'location': LOCATION,
            'address_space': {
                'address_prefixes': ['10.0.0.0/16']
            }
        }
        creation_result = network_client.virtual_networks.create_or_update(
            GROUP_NAME,
            'myVNet',
            vnet_params
        )
        return creation_result.result()
    ```

2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：
   
    ```python
    creation_result = create_vnet(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

3. 若要將子網路新增至虛擬網路，請在 .py 檔案的變數之後新增此函式：
    
    ```python
    def create_subnet(network_client):
        subnet_params = {
            'address_prefix': '10.0.0.0/24'
        }
        creation_result = network_client.subnets.create_or_update(
            GROUP_NAME,
            'myVNet',
            'mySubnet',
            subnet_params
        )

        return creation_result.result()
    ```
        
4. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：
   
    ```python
    creation_result = create_subnet(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

虛擬機器需要網路介面以在虛擬網路上通訊。

1. 若要建立網路介面，請在 .py 檔案的變數之後新增此函式：

    ```python
    def create_nic(network_client):
        subnet_info = network_client.subnets.get(
            GROUP_NAME, 
            'myVNet', 
            'mySubnet'
        )
        publicIPAddress = network_client.public_ip_addresses.get(
            GROUP_NAME,
            'myIPAddress'
        )
        nic_params = {
            'location': LOCATION,
            'ip_configurations': [{
                'name': 'myIPConfig',
                'public_ip_address': publicIPAddress,
                'subnet': {
                    'id': subnet_info.id
                }
            }]
        }
        creation_result = network_client.network_interfaces.create_or_update(
            GROUP_NAME,
            'myNic',
            nic_params
        )

        return creation_result.result()
    ```

2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

    ```python
    creation_result = create_nic(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

既然您已經建立所有支援的資源，您可以建立虛擬機器。

1. 若要建立虛擬機器，請在 .py 檔案的變數之後新增此函式：
   
    ```python
    def create_vm(network_client, compute_client):  
        nic = network_client.network_interfaces.get(
            GROUP_NAME, 
            'myNic'
        )
        avset = compute_client.availability_sets.get(
            GROUP_NAME,
            'myAVSet'
        )
        vm_parameters = {
            'location': LOCATION,
            'os_profile': {
                'computer_name': VM_NAME,
                'admin_username': 'azureuser',
                'admin_password': 'Azure12345678'
            },
            'hardware_profile': {
                'vm_size': 'Standard_DS1'
            },
            'storage_profile': {
                'image_reference': {
                    'publisher': 'MicrosoftWindowsServer',
                    'offer': 'WindowsServer',
                    'sku': '2012-R2-Datacenter',
                    'version': 'latest'
                }
            },
            'network_profile': {
                'network_interfaces': [{
                    'id': nic.id
                }]
            },
            'availability_set': {
                'id': avset.id
            }
        }
        creation_result = compute_client.virtual_machines.create_or_update(
            GROUP_NAME, 
            VM_NAME, 
            vm_parameters
        )
    
        return creation_result.result()
    ```

    > [!NOTE]
    > 本教學課程中會建立執行 Windows Server 作業系統版本的虛擬機器。 若要深入了解如何選取其他映像，請參閱 [使用 Windows PowerShell 和 Azure CLI 來瀏覽和選取 Azure 虛擬機器映像](../linux/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
    > 
    > 

2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

    ```python
    creation_result = create_vm(network_client, compute_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

## <a name="perform-management-tasks"></a>執行管理工作

在虛擬機器的生命週期內，您可以執行一些管理工作，例如啟動、停止或刪除虛擬機器。 此外，您可以建立程式碼來自動執行重複或複雜的工作。

### <a name="get-information-about-the-vm"></a>取得 VM 的相關資訊

1. 若要取得虛擬機器的相關資訊，請在 .py 檔案的變數之後新增此函式：

    ```python
    def get_vm(compute_client):
        vm = compute_client.virtual_machines.get(GROUP_NAME, VM_NAME, expand='instanceView')
        print("hardwareProfile")
        print("   vmSize: ", vm.hardware_profile.vm_size)
        print("\nstorageProfile")
        print("  imageReference")
        print("    publisher: ", vm.storage_profile.image_reference.publisher)
        print("    offer: ", vm.storage_profile.image_reference.offer)
        print("    sku: ", vm.storage_profile.image_reference.sku)
        print("    version: ", vm.storage_profile.image_reference.version)
        print("  osDisk")
        print("    osType: ", vm.storage_profile.os_disk.os_type.value)
        print("    name: ", vm.storage_profile.os_disk.name)
        print("    createOption: ", vm.storage_profile.os_disk.create_option.value)
        print("    caching: ", vm.storage_profile.os_disk.caching.value)
        print("\nosProfile")
        print("  computerName: ", vm.os_profile.computer_name)
        print("  adminUsername: ", vm.os_profile.admin_username)
        print("  provisionVMAgent: {0}".format(vm.os_profile.windows_configuration.provision_vm_agent))
        print("  enableAutomaticUpdates: {0}".format(vm.os_profile.windows_configuration.enable_automatic_updates))
        print("\nnetworkProfile")
        for nic in vm.network_profile.network_interfaces:
            print("  networkInterface id: ", nic.id)
        print("\nvmAgent")
        print("  vmAgentVersion", vm.instance_view.vm_agent.vm_agent_version)
        print("    statuses")
        for stat in vm_result.instance_view.vm_agent.statuses:
            print("    code: ", stat.code)
            print("    displayStatus: ", stat.display_status)
            print("    message: ", stat.message)
            print("    time: ", stat.time)
        print("\ndisks");
        for disk in vm.instance_view.disks:
            print("  name: ", disk.name)
            print("  statuses")
            for stat in disk.statuses:
                print("    code: ", stat.code)
                print("    displayStatus: ", stat.display_status)
                print("    time: ", stat.time)
        print("\nVM general status")
        print("  provisioningStatus: ", vm.provisioning_state)
        print("  id: ", vm.id)
        print("  name: ", vm.name)
        print("  type: ", vm.type)
        print("  location: ", vm.location)
        print("\nVM instance status")
        for stat in vm.instance_view.statuses:
            print("  code: ", stat.code)
            print("  displayStatus: ", stat.display_status)
    ```
2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

    ```python
    get_vm(compute_client)
    print("------------------------------------------------------")
    input('Press enter to continue...')
    ```

### <a name="stop-the-vm"></a>停止 VM

您可以停止虛擬機器並保留其所有的設定，但仍繼續計費，或您可以停止虛擬機器並將其解除配置。 當解除配置虛擬機器時，與其相關聯的所有資源也都會解除配置且其計費會結束。

1. 若要停止虛擬機器而不解除配置，請在 .py 檔案的變數之後新增此函式：

    ```python
    def stop_vm(compute_client):
        compute_client.virtual_machines.power_off(GROUP_NAME, VM_NAME)
    ```

    如果您想要解除配置虛擬機器，請將 PowerOff 呼叫變更為下列程式碼︰

    ```python
    compute_client.virtual_machines.deallocate(GROUP_NAME, VM_NAME)
    ```

2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

    ```python
    stop_vm(compute_client)
    input('Press enter to continue...')
    ```

### <a name="start-the-vm"></a>啟動 VM

1. 若要啟動虛擬機器，請在 .py 檔案的變數之後新增此函式：

    ```python
    def start_vm(compute_client):
        compute_client.virtual_machines.start(GROUP_NAME, VM_NAME)
    ```

2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

    ```python
    start_vm(compute_client)
    input('Press enter to continue...')
    ```

### <a name="resize-the-vm"></a>調整 VM 的大小

決定虛擬機器的大小時，應該考慮部署的許多層面。 如需詳細資訊，請參閱 [VM 大小](sizes.md)。

1. 若要變更虛擬機器的大小，請在 .py 檔案的變數之後新增此函式：

    ```python
    def update_vm(compute_client):
        vm = compute_client.virtual_machines.get(GROUP_NAME, VM_NAME)
        vm.hardware_profile.vm_size = 'Standard_DS3'
        update_result = compute_client.virtual_machines.create_or_update(
            GROUP_NAME, 
            VM_NAME, 
            vm
        )

    return update_result.result()
    ```

2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

    ```python
    update_result = update_vm(compute_client)
    print("------------------------------------------------------")
    print(update_result)
    input('Press enter to continue...')
    ```

### <a name="add-a-data-disk-to-the-vm"></a>將資料磁碟新增至 VM

虛擬機器可以有一或多個[資料磁碟](about-disks-and-vhds.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)，而這些磁碟會儲存成 VHD。

1. 若要將資料磁碟新增至虛擬機器，請在 .py 檔案的變數之後新增此函式： 

    ```python
    def add_datadisk(compute_client):
        disk_creation = compute_client.disks.create_or_update(
            GROUP_NAME,
            'myDataDisk1',
            {
                'location': LOCATION,
                'disk_size_gb': 1,
                'creation_data': {
                    'create_option': DiskCreateOption.empty
                }
            }
        )
        data_disk = disk_creation.result()
        vm = compute_client.virtual_machines.get(GROUP_NAME, VM_NAME)
        add_result = vm.storage_profile.data_disks.append({
            'lun': 1,
            'name': 'myDataDisk1',
            'create_option': DiskCreateOption.attach,
            'managed_disk': {
                'id': data_disk.id
            }
        })
        add_result = compute_client.virtual_machines.create_or_update(
            GROUP_NAME,
            VM_NAME,
            vm)

        return add_result.result()
    ```

2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：

    ```python
    add_result = add_datadisk(compute_client)
    print("------------------------------------------------------")
    print(add_result)
    input('Press enter to continue...')
    ```

## <a name="delete-resources"></a>刪除資源

您將為 Azure 中所使用的資源支付費用，因此，將不再需要的資源刪除永遠是最好的做法。 如果您想要刪除虛擬機器及所有支援的資源，您只需要刪除資源群組。

1. 若要刪除資源群組及所有資源，請在 .py 檔案的變數之後新增此函式：
   
    ```python
    def delete_resources(resource_group_client):
        resource_group_client.resource_groups.delete(GROUP_NAME)
    ```

2. 若要呼叫您先前新增的函式，請在 .py 檔案結尾的 **if** 陳述式下新增此程式碼：
   
    ```python
    delete_resources(resource_group_client)
    ```

3. 儲存 myPythonProject.py。

## <a name="run-the-application"></a>執行應用程式

1. 若要執行主控台應用程式，請在 Visual Studio 中按一下 [啟動]。

2. 傳回每個資源的狀態之後，按下 **Enter** 鍵。 在狀態資訊中，您應該會看到「成功」佈建狀態。 建立虛擬機器之後，您可以將所建立的所有資源刪除。 在您按下 **Enter** 鍵以開始刪除資源之前，可以先花幾分鐘的時間來確認 Azure 入口網站中的建立情況。 如果您讓 Azure 入口網站維持開啟，可能必須重新整理刀鋒視窗以便查看新的資源。  

    此主控台應用程式從開始到完成的完整執行應該需要五分鐘左右。 應用程式已完成之後、所有資源和資源群組刪除之前，可能需要幾分鐘的時間。


## <a name="next-steps"></a>後續步驟

- 如果部署有問題，下一個步驟就是查看 [使用 Azure 入口網站針對資源群組部署進行疑難排解](../../resource-manager-troubleshoot-deployments-portal.md)
- 深入了解 [Azure Python 程式庫](https://docs.microsoft.com/python/api/overview/azure/?view=azure-python)

