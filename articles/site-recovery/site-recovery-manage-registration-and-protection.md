---
title: 移除伺服器與停用保護 | Microsoft Docs
description: 本文說明如何取消註冊 Site Recovery 保存庫中的伺服器，並停用虛擬機器和實體伺服器的保護。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: raynew
ms.openlocfilehash: 998cebdf8b63013128d91571f2610df7d6752e81
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37918776"
---
# <a name="remove-servers-and-disable-protection"></a>移除伺服器並停用保護

本文說明如何將「復原服務」保存庫中的伺服器取消註冊，以及如何針對由 Site Recovery 保護的機器停用保護。


## <a name="unregister-a--configuration-server"></a>將組態伺服器取消註冊

如果您將 VMware VM 或 Windows/Linux 實體伺服器複寫到 Azure，您可以將未連線的組態伺服器從保存庫取消註冊，方法如下：

1. [停用虛擬機器的保護](#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure)。
2. [取消關聯或刪除](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy)複寫原則。
3. [刪除組態伺服器](vmware-azure-manage-configuration-server.md#delete-or-unregister-a-configuration-server)

## <a name="unregister-a-vmm-server"></a>取消註冊 VMM 伺服器

1. 在您想要移除的 VMM 伺服器上，停止在雲端複寫虛擬機器。
2. 在您想要刪除的 VMM 伺服器上，刪除任何由雲端使用的網路對應。 在 [Site Recovery 基礎結構] > [適用於 System Center VMM] > [網路對應] 中，以滑鼠右鍵按一下網路對應 > [刪除]。
3. 記下 VMM 伺服器的識別碼。
4. 將您想要移除的 VMM 伺服器上取消雲端和複寫原則的關聯。  在 [Site Recovery 基礎結構] > [適用於 System Center VMM] >  [複寫原則] 中，按兩下相關聯的原則。 以滑鼠右鍵按一下雲端 > [取消關聯]。
5. 刪除 VMM 伺服器或作用中的節點。 在 [Site Recovery 基礎結構] > [適用於 System Center VMM] > [VMM 伺服器] 中，以滑鼠右鍵按一下伺服器 > [刪除]。
6. 如果您的 VMM 伺服器處於中斷連線的狀態，則在 VMM 伺服器上下載並執行[清除指令碼](http://aka.ms/asr-cleanup-script-vmm)。 使用 [以系統管理員身分執行] 選項開啟 PowerShell，以變更預設 (LocalMachine) 範圍的執行原則。 在指令碼中，指定您想要移除的 VMM 伺服器的識別碼。 該指令碼會從伺服器移除註冊及雲端配對資訊。
5. 在任何次要 VMM 伺服器上執行清除指令碼。
6. 在已安裝提供者的任何其他被動 VMM 叢集節點上執行清除指令碼。
7. 在 VMM 伺服器上手動將提供者解除安裝。 如果您有叢集，請將它從所有節點中移除。
8. 如果您的虛擬機器是複寫到 Azure，必須將 Microsoft 復原服務代理程式從已刪除之雲端中的 Hyper-V 主機上解除安裝。

## <a name="unregister-a-hyper-v-host-in-a-hyper-v-site"></a>取消註冊 Hyper-V 站台中的 Hyper-V 主機

未受 VMM 管理的 Hyper-V 主機會集合成 Hyper-V 站台。 移除 Hyper-V 站台中的主機，方法如下：

1. 針對主機上的 Hyper-V VM 停用複寫。
2. 取消關聯 Hyper-V 站台的原則。 在 [Site Recovery 基礎結構] > [適用於 Hyper-V 站台] >  [複寫原則] 中，按兩下相關聯的原則。 以滑鼠右鍵按一下站台 > [取消關聯]。
3. 刪除 Hyper-V 主機。 在 [Site Recovery 基礎結構] > [適用於 Hyper-V 網站] > [Hyper-V 主機] 中，以滑鼠右鍵按一下伺服器 > [刪除]。
4. 從 Hyper-V 站台移除所有主機之後，將 Hyper-V 站台刪除。 在 [Site Recovery 基礎結構] > [適用於 Hyper-V 網站] > [Hyper-V 網站] 中，以滑鼠右鍵按一下網站 > [刪除]。
5. 如果您的 HYPER-V 主機處於**中斷連線的**狀態，則在您移除的每一部 HYPER-V 主機上執行下列指令碼。 該指令碼會清除伺服器上的設定，並從保存庫取消註冊該伺服器。



        pushd .
        try
        {
             $windowsIdentity=[System.Security.Principal.WindowsIdentity]::GetCurrent()
             $principal=new-object System.Security.Principal.WindowsPrincipal($windowsIdentity)
             $administrators=[System.Security.Principal.WindowsBuiltInRole]::Administrator
             $isAdmin=$principal.IsInRole($administrators)
             if (!$isAdmin)
             {
                "Please run the script as an administrator in elevated mode."
                $choice = Read-Host
                return;       
             }

            $error.Clear()    
            "This script will remove the old Azure Site Recovery Provider related properties. Do you want to continue (Y/N) ?"
            $choice =  Read-Host

            if (!($choice -eq 'Y' -or $choice -eq 'y'))
            {
            "Stopping cleanup."
            return;
            }

            $serviceName = "dra"
            $service = Get-Service -Name $serviceName
            if ($service.Status -eq "Running")
            {
                "Stopping the Azure Site Recovery service..."
                net stop $serviceName
            }

            $asrHivePath = "HKLM:\SOFTWARE\Microsoft\Azure Site Recovery"
            $registrationPath = $asrHivePath + '\Registration'
            $proxySettingsPath = $asrHivePath + '\ProxySettings'
            $draIdvalue = 'DraID'

            if (Test-Path $asrHivePath)
            {
                if (Test-Path $registrationPath)
                {
                    "Removing registration related registry keys."    
                    Remove-Item -Recurse -Path $registrationPath
                }

                if (Test-Path $proxySettingsPath)
            {
                    "Removing proxy settings"
                    Remove-Item -Recurse -Path $proxySettingsPath
                }

                $regNode = Get-ItemProperty -Path $asrHivePath
                if($regNode.DraID -ne $null)
                {            
                    "Removing DraId"
                    Remove-ItemProperty -Path $asrHivePath -Name $draIdValue
                }
                "Registry keys removed."
            }

            # First retrive all the certificates to be deleted
            $ASRcerts = Get-ChildItem -Path cert:\localmachine\my | where-object {$_.friendlyname.startswith('ASR_SRSAUTH_CERT_KEY_CONTAINER') -or $_.friendlyname.startswith('ASR_HYPER_V_HOST_CERT_KEY_CONTAINER')}
            # Open a cert store object
            $store = New-Object System.Security.Cryptography.X509Certificates.X509Store("My","LocalMachine")
            $store.Open('ReadWrite')
            # Delete the certs
            "Removing all related certificates"
            foreach ($cert in $ASRcerts)
            {
                $store.Remove($cert)
            }
        }catch
        {    
            [system.exception]
            Write-Host "Error occured" -ForegroundColor "Red"
            $error[0]
            Write-Host "FAILED" -ForegroundColor "Red"
        }
        popd



## <a name="disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure"></a>停用 VMware VM 或實體伺服器的保護 (VMware 至 Azure)

1. 在 [受保護的項目] > [已複寫的項目] 中，以滑鼠右鍵按一下機器 > [停用複寫]。
2. 在 [停用複寫] 頁面中，選取下列其中一個選項：
    - **停用複寫並移除 (建議)** - 此選項會將複寫的項目從 Azure Site Recovery 移除，且機器的複寫會遭到停止。 系統會清除組態伺服器上的複寫設定，並停止此受保護伺服器的 Site Recovery 計費。
    - **移除** - 只有在來源環境遭到刪除或無法存取 (未連線) 時，才使用此選項。 這會將複寫的項目從 Azure Site Recovery 移除 (停止計費)。 組態伺服器上的複寫設定**將不會**遭到清除。 

> [!NOTE]
> 在這兩個選項中，行動服務將無法從受保護的伺服器解除安裝，您必須手動解除安裝。 如果您打算再次使用相同的組態伺服器保護伺服器，可以略過解除安裝行動服務。

## <a name="disable-protection-for-a-hyper-v-virtual-machine-hyper-v-to-azure"></a>停用 Hyper-V 虛擬機器的保護 (Hyper-V 至 Azure)

> [!NOTE]
> 如果您是在沒有 VMM 伺服器的情況下將 Hyper-V VM 複寫到 Azure，請使用此程序。 如果您要使用 **System Center VMM 至 Azure** 案例複寫虛擬機器，則依照[使用 System Center VMM 至 Azure 案例，停用 Hyper-V 虛擬機器複寫的保護](#disable-protection-for-a-hyper-v-virtual-machine-replicating-using-the-system-centet-vmm-to-azure-scenario)中的指示進行

1. 在 [受保護的項目] > [已複寫的項目] 中，以滑鼠右鍵按一下機器 > [停用複寫]。
2. 在 [停用複寫] 中，您可以選取下列選項：
     - **停用複寫並移除 (建議)** - 此選項會將複寫的項目從 Azure Site Recovery 移除，且機器的複寫會遭到停止。 系統將會清除內部部署虛擬機器上的複寫設定，並停止此受保護伺服器的 Site Recovery 計費。
    - **移除** - 只有在來源環境遭到刪除或無法存取 (未連線) 時，才使用此選項。 這會將複寫的項目從 Azure Site Recovery 移除 (停止計費)。 內部部署虛擬機器上的複寫設定**將不會**遭到清除。 

    > [!NOTE]
    > 如果您選擇 [移除] 選項，則執行下列一組指令碼來清除內部部署 Hyper-V 伺服器上的複寫設定。
1. 在來源 Hyper-V 主機伺服器上，移除虛擬機器的複寫。 將 SQLVM1 取代為您虛擬機器的名稱，並從系統管理 PowerShell 執行指令碼


    
    $vmName = "SQLVM1"  $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'"  $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"  $replicationService.RemoveReplicationRelationship($vm.__PATH)
    

## <a name="disable-protection-for-a-hyper-v-virtual-machine-replicating-to-azure-using-the-system-center-vmm-to-azure-scenario"></a>使用 System Center VMM 至 Azure 案例停用複寫至 Azure 之 HYPER-V 虛擬機器的保護

1. 在 [受保護的項目] > [已複寫的項目] 中，以滑鼠右鍵按一下機器 > [停用複寫]。
2. 在 [停用複寫] 中，選取下列其中一個選項：

    - **停用複寫並移除 (建議)** - 此選項會將複寫的項目從 Azure Site Recovery 移除，且機器的複寫會遭到停止。 系統會清除內部部署虛擬機器上的複寫設定，並停止此受保護伺服器的 Site Recovery 計費。
    - **移除** - 只有在來源環境遭到刪除或無法存取 (未連線) 時，才使用此選項。 這會將複寫的項目從 Azure Site Recovery 移除 (停止計費)。 內部部署虛擬機器上的複寫設定**將不會**遭到清除。 

    > [!NOTE]
    > 如果您選擇 [移除] 選項，則執行下列指令碼來清除內部部署 VMM 伺服器上的複寫設定。
3. 在來源 VMM 伺服器上，使用 VMM 主控台中的 PowerShell (需要系統管理員權限) 執行這個指令碼。 將預留位置 **SQLVM1** 取代為您虛擬機器的名稱。

        $vm = get-scvirtualmachine -Name "SQLVM1"
        Set-SCVirtualMachine -VM $vm -ClearDRProtection
4. 上述步驟會清除 VMM 伺服器上的複寫設定。 若要停止複寫在 Hyper-V 主機伺服器上執行的虛擬機器，請執行此指令碼。 將 SQLVM1 取代成您的虛擬機器的名稱，並將 host01.contoso.com 取代成 Hyper-V 主機伺服器的名稱。

    
    $vmName = "SQLVM1"  $hostName  = "host01.contoso.com"  $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'" -computername $hostName  $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"  -computername $hostName  $replicationService.RemoveReplicationRelationship($vm.__PATH)
    
       
 
## <a name="disable-protection-for-a-hyper-v-virtual-machine-replicating-to-secondary-vmm-server-using-the-system-center-vmm-to-vmm-scenario"></a>使用 System Center VMM 至 VMM 案例停用複寫至次要 VMM 伺服器之 Hyper-V 虛擬機器的保護

1. 在 [受保護的項目] > [已複寫的項目] 中，以滑鼠右鍵按一下機器 > [停用複寫]。
2. 在 [停用複寫] 中，選取下列其中一個選項：

    - **停用複寫並移除 (建議)** - 此選項會將複寫的項目從 Azure Site Recovery 移除，且機器的複寫會遭到停止。 系統會清除內部部署虛擬機器上的複寫設定，並停止此受保護伺服器的 Site Recovery 計費。
    - **移除** - 只有在來源環境遭到刪除或無法存取 (未連線) 時，才使用此選項。 這會將複寫的項目從 Azure Site Recovery 移除 (停止計費)。 內部部署虛擬機器上的複寫設定**將不會**遭到清除。 執行下列一組指令碼，以清除內部部署虛擬機器上的複寫設定。
> [!NOTE]
> 如果您選擇 [移除] 選項，則執行下列指令碼來清除內部部署 VMM 伺服器上的複寫設定。

3. 在來源 VMM 伺服器上，使用 VMM 主控台中的 PowerShell (需要系統管理員權限) 執行這個指令碼。 將預留位置 **SQLVM1** 取代為您虛擬機器的名稱。

         $vm = get-scvirtualmachine -Name "SQLVM1"
         Set-SCVirtualMachine -VM $vm -ClearDRProtection
4. 在次要 VMM 伺服器上，執行此指令碼來清除次要虛擬機器的設定：

        $vm = get-scvirtualmachine -Name "SQLVM1"
        Remove-SCVirtualMachine -VM $vm -Force
5. 在次要 VMM 伺服器上，重新整理 Hyper-V 主機伺服器上的虛擬機器，以便再次於 VMM 主控台中偵測次要 VM。
6. 上述步驟會清除 VMM 伺服器上的複寫設定。 如果您想要停止複寫虛擬機器，請在主要和次要 VM 上執行以下指令碼。 將 SQLVM1 取代成您的虛擬機器的名稱。

        Remove-VMReplication –VMName “SQLVM1”




