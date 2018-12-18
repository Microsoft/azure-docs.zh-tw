如需有關磁碟的詳細資訊，請參閱[關於虛擬機器的磁碟和 VHD](../articles/virtual-machines/linux/about-disks-and-vhds.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

<a id="attachempty"></a>

## <a name="attach-an-empty-disk"></a>連接空的磁碟
1. 開啟 Azure 傳統 CLI 並 [連接至您的 Azure 訂用帳戶](/cli/azure/authenticate-azure-cli)。 確定處於 Azure 服務管理模式 (`azure config mode asm`)。
2. 輸入 `azure vm disk attach-new` 建立並連接新的磁碟，如下列範例所示。 將 *myVM* 取代為 Linux 虛擬機器的名稱，並指定磁碟大小 (GB) (在此範例中為 100 GB)：

    ```azurecli
    azure vm disk attach-new myVM 100
    ```

3. 資料磁碟在建立並連接之後，就會列在 `azure vm disk list <virtual-machine-name>` 的輸出中，如下列範例所示：
   
    ```azurecli
    azure vm disk list TestVM
    ```

    輸出類似於下列範例：

    ```bash
    info:    Executing command vm disk list
   
   * Fetching disk images
   * Getting virtual machines
   * Getting VM disks
     data:    Lun  Size(GB)  Blob-Name                         OS
     data:    ---  --------  --------------------------------  -----
     data:         30        myVM-2645b8030676c8f8.vhd  Linux
     data:    0    100       myVM-76f7ee1ef0f6dddc.vhd
     info:    vm disk list command OK
    ```

<a id="attachexisting"></a>

## <a name="attach-an-existing-disk"></a>連接現有磁碟
連接現有磁碟要求您在儲存體帳戶中需要有可用的 .vhd。

1. 開啟 Azure 傳統 CLI 並 [連接至您的 Azure 訂用帳戶](/cli/azure/authenticate-azure-cli)。 確定處於 Azure 服務管理模式 (`azure config mode asm`)。
2. 檢查您想要附加的 VHD 是否已上傳至 Azure 訂用帳戶：
   
    ```azurecli
    azure vm disk list
    ```

    輸出類似於下列範例：

    ```azurecli
     info:    Executing command vm disk list
   
   * Fetching disk images
     data:    Name                                          OS
     data:    --------------------------------------------  -----
     data:    myTestVhd                                     Linux
     data:    TestVM-ubuntuVMasm-0-201508060029150744  Linux
     data:    TestVM-ubuntuVMasm-0-201508060040530369
     info:    vm disk list command OK
    ```

3. 如果找不到您想要使用的磁碟，您可以使用 `azure vm disk create` 或 `azure vm disk upload` 將本機 VHD 上傳至您的訂用帳戶。 `disk create` 範例如下：
   
    ```azurecli
    azure vm disk create myVhd .\TempDisk\test.VHD -l "East US" -o Linux
    ```

    輸出類似於下列範例：

    ```azurecli
    info:    Executing command vm disk create
    + Retrieving storage accounts
    info:    VHD size : 10 GB
    info:    Uploading 10485760.5 KB
    Requested:100.0% Completed:100.0% Running:   0 Time:   25s Speed:    82 KB/s
    info:    Finishing computing MD5 hash, 16% is complete.
    info:    https://mystorageaccount.blob.core.windows.net/disks/myVHD.vhd was
    uploaded successfully
    info:    vm disk create command OK
    ```
   
   您也可以使用 `azure vm disk upload` ，將 VHD 上傳至特定的儲存體帳戶。 在[這裡](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)閱讀管理 Azure 虛擬機器資料磁碟之命令的詳細資訊。

4. 現在，您可以將所需的 VHD 連接到您的虛擬機器︰
   
    ```azurecli
    azure vm disk attach myVM myVhd
    ```
   
   請務必將 *myVM* 取代為您的虛擬機器名稱，並將 *myVHD* 取代為您所需的 VHD。

5. 您可以使用 `azure vm disk list <virtual-machine-name>`，確認磁碟是否已連接到虛擬機器：
   
    ```azurecli
    azure vm disk list myVM
    ```

    輸出類似於下列範例：

    ```azurecli
     info:    Executing command vm disk list
   
   * Fetching disk images
   * Getting virtual machines
   * Getting VM disks
     data:    Lun  Size(GB)  Blob-Name                         OS
     data:    ---  --------  --------------------------------  -----
     data:         30        TestVM-2645b8030676c8f8.vhd  Linux
     data:    1    10        test.VHD
     data:    0    100        TestVM-76f7ee1ef0f6dddc.vhd
     info:    vm disk list command OK
    ```

> [!NOTE]
> 新增資料磁碟之後，您必須登入虛擬機器並初始化磁碟，這樣虛擬機器才能使用該磁碟來進行儲存 (如需如何初始化磁碟的詳細資訊，請參閱下列步驟)。
> 
> 

