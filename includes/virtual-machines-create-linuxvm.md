
1. 使用[從 Azure 傳統 CLI 連線到 Azure](/cli/azure/authenticate-azure-cli) 中列出的步驟登入 Azure 訂用帳戶。

2. 請確定您處於傳統部署模式，如下所示：

    ```azurecli
    azure config mode asm
    ```

3. 找出您想要從可用映像載入的 Linux 映像，如下所示：

   ```azurecli   
    azure vm image list | grep "Linux"
    ```
   
    在 Windows 命令提示字元視窗中，請使用 **find** ，而不要使用 grep。
   
4. 使用 `azure vm create` 並搭配先前清單中的 Linux 映像建立 VM。 這個步驟會建立雲端服務及儲存體帳戶。 您也可以利用 `-c` 選項將此 VM 連接到現有的雲端服務。 建立 SSH 端點以利用 `-e` 選項登入 Linux 虛擬機器。 下列範例會建立名為 `myVM` 的 VM，方法是使用 `West US` 位置中的 `Ubuntu-14_04_4-LTS` 映像，並且新增使用者名稱 `ops`：
   
    ```azurecli
    azure vm create myVM \
        b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_4-LTS-amd64-server-20160516-en-us-30GB \
        -g ops -p P@ssw0rd! -z "Small" -e -l "West US"
    ```

    輸出類似於下列範例：

    ```azurecli
    info:    Executing command vm create
    + Looking up image b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_4-LTS-amd64-server-20160516-en-us-30GB
    + Looking up cloud service
    info:    cloud service myVM not found.
    + Creating cloud service
    + Retrieving storage accounts
    + Creating VM
    info:    vm create command OK
    ```
   
   > [!NOTE]
   > 針對 Linux 虛擬機器，您必須在 `vm create` 中提供 `-e`。 建立虛擬機器後，就無法啟用 SSH。 如需 SSH 的詳細資料，請參閱[如何在 Azure 上搭配使用 SSH 和 Linux](../articles/virtual-machines/linux/mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

5. 您可以使用 `azure vm show` 命令來確認 VM 的屬性。 下列範例會列出名為 `myVM` 的 VM 的資訊：

    ```azurecli   
    azure vm show myVM
    ```

6. 使用 `azure vm start` 命令啟動您的 VM，如下所示：

    ```azurecli
    azure vm start myVM
    ```

## <a name="next-steps"></a>後續步驟
如需所有 Azure 傳統 CLI 虛擬機器命令的詳細資料，請參閱[搭配傳統部署 API 使用 Azure 傳統 CLI](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)。

