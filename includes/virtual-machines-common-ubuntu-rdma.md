1. 安裝 dapl、rdmacm、ibverbs 及 mlx4

  ```bash
  sudo apt-get update

  sudo apt-get install libdapl2 libmlx4-1

  ```

2. 在 /etc/waagent.conf 中，取消註解組態的下列各行來啟用 RDMA。 您必須要有根目錄存取權才能編輯此檔案。
  
  ```
  OS.EnableRDMA=y

  OS.UpdateRdmaDriver=y
  ```

3. 在 /etc/security/limits.conf 檔案中，新增或變更下列記憶體設定 (KB)。 您必須要有根目錄存取權才能編輯此檔案。 基於測試目的，您可以將 memlock 設定為無限制。 例如：`<User or group name>   hard    memlock   unlimited`。

  ```
  <User or group name> hard    memlock <memory required for your application in KB>

  <User or group name> soft    memlock <memory required for your application in KB>
  ```
  
4. 安裝 Intel MPI Library。 從 Intel [購買並下載](https://software.intel.com/intel-mpi-library/) \(英文\) 程式庫 或下載[免費試用版](https://registrationcenter.intel.com/en/forms/?productid=1740) \(英文\)。

  ```bash
  wget http://registrationcenter-download.intel.com/akdlm/irc_nas/tec/9278/l_mpi_p_5.1.3.223.tgz
  ```
 
 只支援 Intel MPI 5.x 執行階段。
 
 如需安裝步驟，請參閱 [Intel MPI Library 安裝指南](http://registrationcenter-download.intel.com/akdlm/irc_nas/1718/INSTALL.html?lang=en&fileExt=.html) \(英文\)。

5. 針對非根、非偵錯工具的處理程序啟用 ptrace (必須為最新版 Intel MPI 執行)。
 
  ```bash
  echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
  ```
