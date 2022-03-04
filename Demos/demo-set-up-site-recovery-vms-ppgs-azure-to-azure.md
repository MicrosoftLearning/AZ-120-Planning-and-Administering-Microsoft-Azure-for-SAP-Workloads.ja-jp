---
ms.openlocfilehash: 3c8f55e525f1ec25f6cd9518c4dd3d99734e34c9
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857615"
---
# <a name="demonstration-set-up-site-recovery-for-virtual-machines-in-proximity-placement-groups---azure-to-azure"></a>デモ:近接通信配置グループの仮想マシンの Site Recovery を設定する - Azure から Azure へ

> **注:**  この機能は現在 PowerShell を介して使用でき、Managed Disks を使用するすべての Azure VM をサポートします。

## <a name="prerequisites"></a>前提条件

1. Azure PowerShell Az モジュールがあることを確認します。 Azure PowerShell をインストールまたはアップグレードする必要がある場合は、[Azure PowerShell モジュールのインストールと構成のガイド](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-5.0.0)に関するページをご覧ください。
2. Azure PowerShell Az のバージョンは 4.1.0 以上である必要があります。 現在のバージョンを確認するには、次のコマンドを使用します。

    ```azurepowershell
    Get-InstalledModule -Name Az
    ```

>**注:**  ターゲットの近接通信配置グループの一意の ID が手元に用意されていることを確認します。 新しい近接配置グループを作成している場合は、[こちら](https://docs.microsoft.com/azure/virtual-machines/windows/proximity-placement-groups#create-a-proximity-placement-group)のコマンドを確認し、既存の近接配置グループを使用している場合は、[こちら](https://docs.microsoft.com/azure/virtual-machines/windows/proximity-placement-groups#list-proximity-placement-groups)のコマンドを使用します。

## <a name="sign-in-to-your-microsoft-azure-subscription"></a>Microsoft Azure サブスクリプションにサインインする

1. `Connect-AzAccount` コマンドレットを使用してお使いの Azure サブスクリプションにサインインします。

    ```azurepowershell
    Connect-AzAccount
    ```

1. Azure サブスクリプションを選択します。 `Get-AzSubscription` コマンドレットを使って、アクセスできる Azure サブスクリプションの一覧を取得します。 `Set-AzContext` コマンドレットを使って、利用する Azure サブスクリプションを選びます。

    ```azurepowershell
    Set-AzContext -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    ```

## <a name="get-details-of-the-virtual-machine-to-be-replicated"></a>レプリケートする仮想マシンの詳細を取得する

1. このデモでは、米国東部リージョンの仮想マシンが米国西部 2 リージョンにレプリケートされ、復旧されます。 レプリケートされる仮想マシンには、OS ディスクと 1 つのデータ ディスクが与えられます。 この例で使用されている仮想マシンの名前は `AzureDemoVM` です。

    ```azurepowershell
    # Get details of the virtual machine
    $VM = Get-AzVM -ResourceGroupName "A2AdemoRG" -Name "AzureDemoVM"

    Write-Output $VM
    ```

    ```Output
    ResourceGroupName  : A2AdemoRG
    Id                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/A2AdemoRG/providers/Microsoft.Compute/virtualMachines/AzureDemoVM
    VmId               : 1b864902-c7ea-499a-ad0f-65da2930b81b
    Name               : AzureDemoVM
    Type               : Microsoft.Compute/virtualMachines
    Location           : eastus
    Tags               : {}
    DiagnosticsProfile : {BootDiagnostics}
    HardwareProfile    : {VmSize}
    NetworkProfile     : {NetworkInterfaces}
    OSProfile          : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
    ProvisioningState  : Succeeded
    StorageProfile     : {ImageReference, OsDisk, DataDisks}
    ```

1. 仮想マシンのディスクについて、ディスク詳細を取得します。 ディスクの詳細は、後ほど、仮想マシンのレプリケーションを開始するときに使用されます。

    ```azurepowershell
    $OSDiskVhdURI = $VM.StorageProfile.OsDisk.Vhd
    $DataDisk1VhdURI = $VM.StorageProfile.DataDisks[0].Vhd
    ```

## <a name="create-a-recovery-services-vault"></a>Recovery Services コンテナーを作成する

1. Recovery Services コンテナーを作成するリソース グループを作成します。

    > **重要:**
    > * Recovery Services コンテナーと保護される仮想マシンは、Azure の別の場所に存在する必要があります。
    > * Recovery Services コンテナーのリソース グループと保護される仮想マシンは、Azure の別の場所に存在する必要があります。
    > * Recovery Services コンテナーと、それが属するリソース グループは、同じ Azure の場所にあってもかまいません。

1. このデモでは、保護される仮想マシンは米国東部リージョンにあります。 ディザスター リカバリー用に選択された復旧リージョンは、米国西部 2 リージョンです。 Recovery Services コンテナーと、コンテナーのリソース グループは、いずれも復旧リージョン (米国西部 2) 内にあります。

    ```azurepowershell
    #Create a resource group for the recovery services vault in the recovery Azure region
    New-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"
    ```

    ```Output
    ResourceGroupName : a2ademorecoveryrg
    Location          : westus2
    ProvisioningState : Succeeded
    Tags              :
    ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg
    ```

1. Recovery Services コンテナーを作成します。 この例では、`a2aDemoRecoveryVault` という名前の Recovery Services コンテナーが米国西部 2 リージョン内に作成されます。

    ```azurepowershell
    #Create a new Recovery services vault in the recovery region
    $vault = New-AzRecoveryServicesVault -Name "a2aDemoRecoveryVault" -ResourceGroupName "a2ademorecoveryrg" -Location "West US 2"

    Write-Output $vault
    ```

    ```Output
    Name              : a2aDemoRecoveryVault
    ID                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoRecoveryVault
    Type              : Microsoft.RecoveryServices/vaults
    Location          : westus2
    ResourceGroupName : a2ademorecoveryrg
    SubscriptionId    : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
    ```

## <a name="set-the-vault-context"></a>コンテナーのコンテキストを設定する

1. PowerShell セッションで使用するための、コンテナーのコンテキストを設定します。 コンテナーのコンテキストが設定されると、PowerShell セッションでの Azure Site Recovery 操作は、選択されているコンテナーのコンテキストで実行されます。

    ```azurepowershell
    #Setting the vault context.
    Set-AzRecoveryServicesAsrVaultContext -Vault $vault
    ```

    ```Output
    ResourceName         ResourceGroupName ResourceNamespace          ResourceType
    ------------         ----------------- -----------------          -----------
    a2aDemoRecoveryVault a2ademorecoveryrg Microsoft.RecoveryServices Vaults
    ```

    ```azurepowershell
    #Delete the downloaded vault settings file
    Remove-Item -Path $Vaultsettingsfile.FilePath
    ```

1. Azure から Azure への移行では、コンテナーのコンテキストを新しく作成されたコンテナーに設定できます。

    ```azurepowershell
    #Set the vault context for the PowerShell session.
    Set-AzRecoveryServicesAsrVaultContext -Vault $vault
    ```

## <a name="prepare-the-vault-to-start-replicating-azure-virtual-machines"></a>Azure 仮想マシンのレプリケートを開始するようにコンテナーを準備する

### <a name="create-a-site-recovery-fabric-object-to-represent-the-primary-source-region"></a>プライマリ (ソース) リージョンを表す Site Recovery ファブリック オブジェクトを作成する

コンテナー内のファブリック オブジェクトは、Azure リージョンを表します。 プライマリ ファブリック オブジェクトは、コンテナーの保護対象となっている仮想マシンが属する Azure リージョンを表すために作成します。 このデモでは、保護される仮想マシンは米国東部リージョンにあります。

- リージョンごとに作成できるファブリック オブジェクトは 1 つだけです。
- Azure portal で VM の Site Recovery レプリケーションを以前に有効にしている場合、Site Recovery によってファブリック オブジェクトが自動的に作成されます。 リージョンにファブリック オブジェクトが存在する場合は、新しいファブリック オブジェクトを作成することはできません。

開始する前に、Site Recovery 操作が非同期的に実行されることを理解してください。 操作を開始すると、Azure Site Recovery ジョブが送信されて、ジョブ追跡オブジェクトが返されます。

1. このジョブ追跡オブジェクトを使用して、ジョブ (`Get-AzRecoveryServicesAsrJob`) の最新の状態を取得し、操作の状態を監視します。

    ```azurepowershell
    #Create Primary ASR fabric
    $TempASRJob = New-AzRecoveryServicesAsrFabric -Azure -Location 'East US'  -Name "A2Ademo-EastUS"

    # Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
            #If the job hasn't completed, sleep for 10 seconds before checking the job status again
            sleep 10;
            $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
    }

    #Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
    Write-Output $TempASRJob.State

    $PrimaryFabric = Get-AzRecoveryServicesAsrFabric -Name "A2Ademo-EastUS"
    ```

1. 同じコンテナーに対して、複数の Azure リージョンの仮想マシンが保護されている場合は、ソースの Azure リージョンごとにファブリック オブジェクトを 1 つ作成します。

### <a name="create-a-site-recovery-fabric-object-to-represent-the-recovery-region"></a>復旧リージョンを表す Site Recovery ファブリック オブジェクトを作成する

復旧ファブリック オブジェクトは、復旧用の Azure の場所を表します。 フェールオーバーが行われると、仮想マシンがレプリケートされ、復旧ファブリックで表わされる復旧リージョンで復旧されます。 この例で使用される復旧用 Azure リージョンは、米国西部 2 です。

```azurepowershell
#Create Recovery ASR fabric
$TempASRJob = New-AzRecoveryServicesAsrFabric -Azure -Location 'West US 2'  -Name "A2Ademo-WestUS"

# Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$RecoveryFabric = Get-AzRecoveryServicesAsrFabric -Name "A2Ademo-WestUS"
```

## <a name="create-site-recovery-protection-containers"></a>Site Recovery 保護コンテナーを作成する

### <a name="create-a-site-recovery-protection-container-in-the-primary-fabric"></a>プライマリ ファブリックに Site Recovery 保護コンテナーを作成する

保護コンテナーは、ファブリック内でレプリケートされる項目をグループ化するのに使用されるコンテナーです。

```azurepowershell
#Create a Protection container in the primary Azure region (within the Primary fabric)
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainer -InputObject $PrimaryFabric -Name "A2AEastUSProtectionContainer"

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

Write-Output $TempASRJob.State

$PrimaryProtContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $PrimaryFabric -Name "A2AEastUSProtectionContainer"
```

### <a name="create-a-site-recovery-protection-container-in-the-recovery-fabric"></a>復旧ファブリック内に Site Recovery 保護コンテナーを作成する

```azurepowershell
#Create a Protection container in the recovery Azure region (within the Recovery fabric)
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainer -InputObject $RecoveryFabric -Name "A2AWestUSProtectionContainer"

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"

Write-Output $TempASRJob.State

$RecoveryProtContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $RecoveryFabric -Name "A2AWestUSProtectionContainer"
```

## <a name="create-a-replication-policy"></a>レプリケーション ポリシーを作成する

```azurepowershell
#Create replication policy
$TempASRJob = New-AzRecoveryServicesAsrPolicy -AzureToAzure -Name "A2APolicy" -RecoveryPointRetentionInHours 24 -ApplicationConsistentSnapshotFrequencyInHours 4

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$ReplicationPolicy = Get-AzRecoveryServicesAsrPolicy -Name "A2APolicy"
```

## <a name="create-protection-container-mappings"></a>保護コンテナー マッピングを作成する

### <a name="create-a-protection-container-mapping-between-the-primary-and-recovery-protection-container"></a>プライマリ コンテナーと復旧保護コンテナー間に保護コンテナー マッピングを作成する

保護コンテナー マッピングでは、プライマリ保護コンテナーを、回復保護コンテナーおよびレプリケーション ポリシーとマップします。 保護コンテナー ペア間で仮想マシンをレプリケートするのに使用するレプリケーション ポリシーごとに、1 つのマッピングを作成します。

```azurepowershell
#Create Protection container mapping between the Primary and Recovery Protection Containers with the Replication policy
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "A2APrimaryToRecovery" -Policy $ReplicationPolicy -PrimaryProtectionContainer $PrimaryProtContainer -RecoveryProtectionContainer $RecoveryProtContainer

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$EusToWusPCMapping = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $PrimaryProtContainer -Name "A2APrimaryToRecovery"
```

### <a name="create-a-protection-container-mapping-for-failback-reverse-replication-after-a-failover"></a>フェールバック (フェールオーバー後のレプリケーションの反転) のための保護コンテナー マッピングを作成する

フェールオーバー後、フェールオーバーした仮想マシンを元の Azure リージョンに戻す準備ができたら、フェールバックを行います。 フェールバックするために、フェールオーバーした仮想マシンは、フェールオーバーしたリージョンから元のリージョンへと逆方向にレプリケートされます。 レプリケーションの反転の際には、元のリージョンと復旧リージョンの役割が入れ替わります。 元のリージョンはこれで新しい復旧リージョンになり、元は復旧リージョンであったものは今後プライマリ リージョンになります。 レプリケーションの反転のための保護コンテナー マッピングは、元のリージョンと復旧リージョンで入れ替えた役割を表します。

```azurepowershell
#Create Protection container mapping (for fail back) between the Recovery and Primary Protection Containers with the Replication policy
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "A2ARecoveryToPrimary" -Policy $ReplicationPolicy -PrimaryProtectionContainer $RecoveryProtContainer -RecoveryProtectionContainer $PrimaryProtContainer

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$WusToEusPCMapping = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $RecoveryProtContainer -Name "A2ARecoveryToPrimary"
```

## <a name="create-cache-storage-account-and-target-storage-account"></a>キャッシュ ストレージ アカウントとターゲット ストレージ アカウントを作成する

キャッシュ ストレージ アカウントは、レプリケートされる仮想マシンと同じ Azure リージョン内の標準的ストレージ アカウントです。 キャッシュ ストレージ アカウントは、変更が復旧 Azure リージョンに移動される前に、レプリケーションの変更を一時的に保持するために使用されます。 必須ではありませんが、仮想マシンのディスクごとに異なるキャッシュ ストレージ アカウントを指定できます。

```azurepowershell
#Create Cache storage account for replication logs in the primary region
$EastUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestorage" -ResourceGroupName "A2AdemoRG" -Location 'East US' -SkuName Standard_LRS -Kind Storage
```

**マネージド ディスクを使用していない** 仮想マシンの場合、ターゲット ストレージ アカウントは、仮想マシンのディスクのレプリケート先となる復旧リージョン内のストレージ アカウントです。 ターゲット ストレージ アカウントは、Standard Storage アカウントと Premium Storage アカウントのどちらでもかまいません。 ディスクのデータ変化率 (IO の書き込み率) と、Azure Site Recovery が各ストレージの種類でサポートしているデータ変更頻度の上限に基づいて、必要なストレージ アカウントの種類を選択します。

```azurepowershell
#Create Target storage account in the recovery region. In this case a Standard Storage account
$WestUSTargetStorageAccount = New-AzStorageAccount -Name "a2atargetstorage" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -SkuName Standard_LRS -Kind Storage
```

## <a name="create-network-mappings"></a>ネットワーク マッピングを作成する

ネットワーク マッピングは、プライマリ リージョン内の仮想ネットワークを、復旧リージョン内の仮想ネットワークにマップします。 ネットワーク マッピングでは、復旧リージョン内の Azure 仮想ネットワークを指定します。これが、プライマリ仮想ネットワーク内の仮想マシンのフェールオーバー先になる必要があります。 1 つの Azure 仮想ネットワークをマップできるのは、復旧リージョン内の 1 つの Azure 仮想ネットワークのみです。

1. フェールオーバー先の復旧リージョン内に Azure 仮想ネットワークを作成します。

   ```azurepowershell
    #Create a Recovery Network in the recovery region
    $WestUSRecoveryVnet = New-AzVirtualNetwork -Name "a2arecoveryvnet" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -AddressPrefix "10.0.0.0/16"

    Add-AzVirtualNetworkSubnetConfig -Name "default" -VirtualNetwork $WestUSRecoveryVnet -AddressPrefix "10.0.0.0/20" | Set-AzVirtualNetwork

    $WestUSRecoveryNetwork = $WestUSRecoveryVnet.Id
   ```

1. プライマリ仮想ネットワークを取得します。 仮想マシンの接続先である VNet:

   ```azurepowershell
    #Retrieve the virtual network that the virtual machine is connected to

    #Get first network interface card(nic) of the virtual machine
    $SplitNicArmId = $VM.NetworkProfile.NetworkInterfaces[0].Id.split("/")

    #Extract resource group name from the ResourceId of the nic
    $NICRG = $SplitNicArmId[4]

    #Extract resource name from the ResourceId of the nic
    $NICname = $SplitNicArmId[-1]

    #Get network interface details using the extracted resource group name and resource name
    $NIC = Get-AzNetworkInterface -ResourceGroupName $NICRG -Name $NICname

    #Get the subnet ID of the subnet that the nic is connected to
    $PrimarySubnet = $NIC.IpConfigurations[0].Subnet

    # Extract the resource ID of the Azure virtual network the nic is connected to from the subnet ID
    $EastUSPrimaryNetwork = (Split-Path(Split-Path($PrimarySubnet.Id))).Replace("\","/")
   ```

1. プライマリ仮想ネットワークと、復旧用の仮想ネットワーク間のネットワーク マッピングを作成します。

   ```azurepowershell
    #Create an ASR network mapping between the primary Azure virtual network and the recovery Azure virtual network
    $TempASRJob = New-AzRecoveryServicesAsrNetworkMapping -AzureToAzure -Name "A2AEusToWusNWMapping" -PrimaryFabric $PrimaryFabric -PrimaryAzureNetworkId $EastUSPrimaryNetwork -RecoveryFabric $RecoveryFabric -RecoveryAzureNetworkId $WestUSRecoveryNetwork

    #Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
            sleep 10;
            $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
    }

    #Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
    Write-Output $TempASRJob.State
   ```

1. 逆方向 (フェールバック) のネットワーク マッピングを作成します。

    ```azurepowershell
    #Create an ASR network mapping for fail back between the recovery Azure virtual network and the primary Azure virtual network
    $TempASRJob = New-AzRecoveryServicesAsrNetworkMapping -AzureToAzure -Name "A2AWusToEusNWMapping" -PrimaryFabric $RecoveryFabric -PrimaryAzureNetworkId $WestUSRecoveryNetwork -RecoveryFabric $PrimaryFabric -RecoveryAzureNetworkId $EastUSPrimaryNetwork

    #Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
            sleep 10;
            $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
    }

    #Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
    Write-Output $TempASRJob.State
    ```

## <a name="replicate-an-azure-virtual-machine-with-managed-disks"></a>マネージド ディスクを使用している Azure 仮想マシンをレプリケートします。

次の PowerShell コマンドレットを使用して、マネージド ディスクと Azure 仮想マシンをレプリケートします。

    ```azurepowershell
    #Get the resource group that the virtual machine must be created in when it's failed over.
    $RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

    #Specify replication properties for each disk of the VM that will be replicated (create disk replication configuration).
    #Make sure to replace the variable $OSdiskName with the OS disk name.

    #OS Disk
    $OSdisk = Get-AzDisk -DiskName $OSdiskName -ResourceGroupName "A2AdemoRG"
    $OSdiskId = $OSdisk.Id
    $RecoveryOSDiskAccountType = $OSdisk.Sku.Name
    $RecoveryReplicaDiskAccountType = $OSdisk.Sku.Name

    $OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $OSdiskId -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

    #Make sure to replace the variable $datadiskName with the data disk name.

    #Data disk
    $datadisk = Get-AzDisk -DiskName $datadiskName -ResourceGroupName "A2AdemoRG"
    $datadiskId1 = $datadisk[0].Id
    $RecoveryReplicaDiskAccountType = $datadisk[0].Sku.Name
    $RecoveryTargetDiskAccountType = $datadisk[0].Sku.Name

    $DataDisk1ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $datadiskId1 -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType

    #Create a list of disk replication configuration objects for the disks of the virtual machine that will be replicated.

    $diskconfigs = @()
    $diskconfigs += $OSDiskReplicationConfig, $DataDisk1ReplicationConfig

    #Start replication by creating a replication protected item. Use a GUID for the name of the replication protected item to ensure uniqueness of the name.

    $TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id
    ```

    When you're enabling replication for multiple data disks, use the following PowerShell cmdlet:

    ```azurepowershell
    #Get the resource group that the virtual machine must be created in when it's failed over.
    $RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

    #Specify replication properties for each disk of the VM that will be replicated (create disk replication configuration).
    #Make sure to replace the variable $OSdiskName with the OS disk name.

    #OS Disk
    $OSdisk = Get-AzDisk -DiskName $OSdiskName -ResourceGroupName "A2AdemoRG"
    $OSdiskId = $OSdisk.Id
    $RecoveryOSDiskAccountType = $OSdisk.Sku.Name
    $RecoveryReplicaDiskAccountType = $OSdisk.Sku.Name

    $OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $OSdiskId -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

    $diskconfigs = @()
    $diskconfigs += $OSDiskReplicationConfig

    #Data disk

    # Add data disks
    Foreach( $disk in $VM.StorageProfile.DataDisks)
    {
        $datadisk = Get-AzDisk -DiskName $datadiskName -ResourceGroupName "A2AdemoRG"
        $dataDiskId1 = $datadisk[0].Id
        $RecoveryReplicaDiskAccountType = $datadisk[0].Sku.Name
        $RecoveryTargetDiskAccountType = $datadisk[0].Sku.Name
        $DataDisk1ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id `
             -DiskId $dataDiskId1 -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType `
             -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType
        $diskconfigs += $DataDisk1ReplicationConfig
    }

    #Start replication by creating a replication protected item. Use a GUID for the name of the replication protected item to ensure uniqueness of the name.

    $TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id
    ```

    When you're enabling zone-to-zone replication with a proximity placement group, the command to start replication will be exchanged with the PowerShell cmdlet:

    ```azurepowershell
    $TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id -RecoveryAvailabilityZone "2"
    ```

    After the operation to start replication succeeds, virtual machine data is replicated to the recovery region.

    The replication process starts by initially seeding a copy of the replicating disks of the virtual machine in the recovery region. This phase is called the *initial replication* phase.

    After initial replication finishes, replication moves to the *differential synchronization* phase. At this point, the virtual machine is protected, and you can perform a test failover operation on it. The replication state of the replicated item that represents the virtual machine goes to the protected state after initial replication finishes.

    Monitor the replication state and replication health for the virtual machine by getting details of the replication protected item that corresponds to it:

    ```azurepowershell
    Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $PrimaryProtContainer | Select FriendlyName, ProtectionState, ReplicationHealth
    ```

## <a name="perform-validate-and-clean-up-a-test-failover"></a>テスト フェール オーバーの実行、検証、クリーンアップ

仮想マシンのレプリケーションが保護された状態になったら、仮想マシンに (仮想マシンのレプリケーション保護項目に) テスト フェールオーバー操作を実行できます。

```azurepowershell
#Create a separate network for test failover (not connected to my DR network)
$TFOVnet = New-AzVirtualNetwork -Name "a2aTFOvnet" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -AddressPrefix "10.3.0.0/16"

Add-AzVirtualNetworkSubnetConfig -Name "default" -VirtualNetwork $TFOVnet -AddressPrefix "10.3.0.0/20" | Set-AzVirtualNetwork

$TFONetwork= $TFOVnet.Id
```

テスト フェールオーバーを実行します。

```azurepowershell
$ReplicationProtectedItem = Get-AzRecoveryServicesAsrReplicationProtectedItem -FriendlyName "AzureDemoVM" -ProtectionContainer $PrimaryProtContainer

$TFOJob = Start-AzRecoveryServicesAsrTestFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem -AzureVMNetworkId $TFONetwork -Direction PrimaryToRecovery
```

テスト フェールオーバーの操作が完了するのを待ちます。

```azurepowershell
Get-AzRecoveryServicesAsrJob -Job $TFOJob
```

```Output
Name             : 3dcb043e-3c6d-4e0e-a42e-8d4245668547
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoR
                   ecoveryVault/replicationJobs/3dcb043e-3c6d-4e0e-a42e-8d4245668547
Type             : Microsoft.RecoveryServices/vaults/replicationJobs
JobType          : TestFailover
DisplayName      : Test failover
ClientRequestId  : 1ef8515b-b130-4452-a44d-91aaf071931c ActivityId: 907bb2bc-ebe6-4732-8b66-77d0546eaba8
State            : Succeeded
StateDescription : Completed
StartTime        : 4/25/2018 4:29:43 AM
EndTime          : 4/25/2018 4:33:06 AM
TargetObjectId   : ce86206c-bd78-53b4-b004-39b722c1ac3a
TargetObjectType : ProtectionEntity
TargetObjectName : azuredemovm
AllowedActions   :
Tasks            : {Prerequisites check for test failover, Create test virtual machine, Preparing the virtual machine, Start the virtual machine}
Errors           : {}
```

テスト フェールオーバーのジョブが正常に完了したら、テスト フェールオーバーが行われた仮想マシンに接続し、テスト フェールオーバーを検証できます。

テスト フェールオーバーを行った仮想マシンでテストが完了したら、テスト フェールオーバー操作のクリーンアップを開始して、テスト コピーをクリーンアップします。 この操作により、テスト フェールオーバーによって作成された仮想マシンのテスト コピーが削除されます。

```azurepowershell
$Job_TFOCleanup = Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $ReplicationProtectedItem

Get-AzRecoveryServicesAsrJob -Job $Job_TFOCleanup | Select State
```

```Output
State
-----
Succeeded
```

## <a name="fail-over-the-virtual-machine"></a>仮想枚ｓンをフェール オーバーする

仮想マシンを特定の復旧ポイントにフェールオーバーします。

```azurepowershell
$RecoveryPoints = Get-AzRecoveryServicesAsrRecoveryPoint -ReplicationProtectedItem $ReplicationProtectedItem

#The list of recovery points returned may not be sorted chronologically and will need to be sorted first, in order to be able to find the oldest or the latest recovery points for the virtual machine.
"{0} {1}" -f $RecoveryPoints[0].RecoveryPointType, $RecoveryPoints[-1].RecoveryPointTime
```

```Output
CrashConsistent 4/24/2018 11:10:25 PM
```

```azurepowershell
#Start the fail over job
$Job_Failover = Start-AzRecoveryServicesAsrUnplannedFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem -Direction PrimaryToRecovery -RecoveryPoint $RecoveryPoints[-1]

do {
        $Job_Failover = Get-AzRecoveryServicesAsrJob -Job $Job_Failover;
        sleep 30;
} while (($Job_Failover.State -eq "InProgress") -or ($JobFailover.State -eq "NotStarted"))

$Job_Failover.State
```

```Output
Succeeded
```

フェールオーバー ジョブが正常に完了したら、フェールオーバー操作をコミットできます。

```azurepowershell
$CommitFailoverJOb = Start-AzRecoveryServicesAsrCommitFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem

Get-AzRecoveryServicesAsrJob -Job $CommitFailoverJOb
```

```Output
Name             : 58afc2b7-5cfe-4da9-83b2-6df358c6e4ff
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoR
                   ecoveryVault/replicationJobs/58afc2b7-5cfe-4da9-83b2-6df358c6e4ff
Type             : Microsoft.RecoveryServices/vaults/replicationJobs
JobType          : CommitFailover
DisplayName      : Commit
ClientRequestId  : 10a95d6c-359e-4603-b7d9-b7ee3317ce94 ActivityId: 8751ada4-fc42-4238-8de6-a82618408fcf
State            : Succeeded
StateDescription : Completed
StartTime        : 4/25/2018 4:50:58 AM
EndTime          : 4/25/2018 4:51:01 AM
TargetObjectId   : ce86206c-bd78-53b4-b004-39b722c1ac3a
TargetObjectType : ProtectionEntity
TargetObjectName : azuredemovm
AllowedActions   :
Tasks            : {Prerequisite check, Commit}
Errors           : {}
```

## <a name="reprotect-and-fail-back-to-the-source-region"></a>再保護とソース リージョンへのフェールバック

次の Power Shell コマンドレットを使用して、再保護し、ソース領域にフェール バックします。

```azurepowershell
#Create a cache storage account for replication logs in the primary region.
$WestUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestoragewestus" -ResourceGroupName "A2AdemoRG" -Location 'West US' -SkuName Standard_LRS -Kind Storage

#Use the recovery protection container, the new cache storage account in West US, and the source region VM resource group. 
Update-AzRecoveryServicesAsrProtectionDirection -ReplicationProtectedItem $ReplicationProtectedItem -AzureToAzure -ProtectionContainerMapping $WusToEusPCMapping -LogStorageAccountId $WestUSCacheStorageAccount.Id -RecoveryResourceGroupID $sourceVMResourcegroup.ResourceId -RecoveryProximityPlacementGroupId $vm.ProximityPlacementGroup.Id
```

## <a name="disable-replication"></a>レプリケーションを無効にする

`Remove-AzRecoveryServicesAsrReplicationProtectedItem` コマンドレットでレプリケーションを無効にできます。

```azurepowershell
Remove-AzRecoveryServicesAsrReplicationProtectedItem -ReplicationProtectedItem $ReplicationProtectedItem
```