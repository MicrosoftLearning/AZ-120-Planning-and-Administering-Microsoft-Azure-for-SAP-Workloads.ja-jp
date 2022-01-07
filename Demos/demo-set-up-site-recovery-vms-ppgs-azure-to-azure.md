# デモ: 近接通信配置グループの仮想マシンの Site Recovery を設定する - Azure から Azure へ

> **注:** この機能は現在 PowerShell を介して使用でき、Managed Disks を使用するすべての Azure VM をサポートします。

## 前提条件

1. Azure PowerShell Az モジュールがあることを確認します。Azure PowerShell をインストールまたはアップグレードする必要がある場合は、この[ガイドに従って Azure PowerShell をインストールして構成します](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-5.0.0)。
2. 最小バージョンの Azure PowerShell Az は 4.1.0 である必要があります。現在のバージョンを確認するには、次のコマンドを使用します。

    ```azurepowershell
	Get-InstalledModule -Name Az
    ```

>**注:** ターゲットの近接通信配置グループの一意の ID が手元に用意されていることを確認します。新しい近接通信配置グループを作成する場合は、[ここ](https://docs.microsoft.com/azure/virtual-machines/windows/proximity-placement-groups#create-a-proximity-placement-group)でコマンドを確認し、既存の近接通信配置グループを使用している場合は、[ここ](https://docs.microsoft.com/azure/virtual-machines/windows/proximity-placement-groups#list-proximity-placement-groups)でコマンドを使用します。

## Microsoft Azure サブスクリプションにサインインします。

1. `Connect-AzAccount`コマンドレットを使用して Azure サブスクリプションにログインします。

    ```azurepowershell
    Connect-AzAccount
    ```

1. Azure サブスクリプションを選択します。`Get-AzSubscription`コマンドレットを使用して、アクセスできる Azure サブスクリプションの一覧を取得します。`Set-AzContext`コマンドレットを使用して、操作する Azure サブスクリプションを選択します。

    ```azurepowershell
    Set-AzContext -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    ```

## レプリケートする仮想マシンの詳細を取得する

1. このデモでは、米国東部リージョンの仮想マシンが米国西部 2 リージョンにレプリケートされ、復旧されます。レプリケートされる仮想マシンには、OS ディスクと 1 つのデータ ディスクがあります。この例で使用されている仮想マシンの名前は `AzureDemoVM`です。

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

1. 仮想マシンのディスクの詳細を取得します。ディスクの詳細は、後で仮想マシンのレプリケーションを開始するときに使用されます。

    ```azurepowershell
    $OSDiskVhdURI = $VM.StorageProfile.OsDisk.Vhd
    $DataDisk1VhdURI = $VM.StorageProfile.DataDisks[0].Vhd
    ```

## Recovery Services コンテナーを作成する

1. Recovery Services コンテナーを作成するリソース グループを作成します。

    > **重要:**
    > * Recovery Services コンテナーと保護される仮想マシンは、Azure の別の場所に存在する必要があります。
    > * Recovery Services コンテナーのリソース グループと保護される仮想マシンは、Azure の別の場所に存在する必要があります。
    > * Recovery Services コンテナーと、それが属するリソース グループは、同じ Azure の場所に配置できます。

1. このデモでは、保護される仮想マシンは米国東部リージョンにあります。ディザスター リカバリーのために選択された復旧リージョンは、米国西部 2 リージョンです。Recovery Services コンテナー、およびコンテナーのリソース グループは、両方とも復旧リージョン (米国西部 2) にあります。

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

1. Recovery Services コンテナーを作成します。この例では、`a2aDemoRecoveryVault`という名前のRecovery Services コンテナーが米国西部 2 リージョンに作成されます。

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

## コンテナー コンテキストの設定

1. PowerShell セッションで使用するコンテナー コンテキストを設定します。コンテナー コンテキストを設定すると、PowerShell セッションでの Azure Site Recovery 操作は、選択したコンテナーのコンテキストで実行されます。

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

1. Azure から Azure への移行の場合、新しく作成されたコンテナーにコンテナー コンテキストを設定できます。

    ```azurepowershell
    #Set the vault context for the PowerShell session.
    Set-AzRecoveryServicesAsrVaultContext -Vault $vault
    ```

## Azure Virtual Machines のレプリケートを開始するためのコンテナーの準備

### プライマリ (ソース) リージョンを表す Site Recovery ファブリック オブジェクトを作成する

コンテナー内のファブリック オブジェクトは、Azure リージョンを表します。プライマリ ファブリック オブジェクトは、コンテナーに対して保護されている仮想マシンが属する Azure リージョンを表すために作成されます。このデモでは、保護される仮想マシンは米国東部リージョンにあります。

- リージョンごとに作成できるファブリック オブジェクトは 1 つだけです。
- Azure portal で VM の Site Recovery レプリケーションを以前に有効にしている場合、Site Recovery によってファブリック オブジェクトが自動的に作成されます。あるリージョンにファブリック オブジェクトが存在する場合、新しいオブジェクトを作成することはできません。

開始する前に、Site Recovery 操作が非同期的に実行されることを理解してください。操作を開始すると、Azure Site Recovery ジョブが送信され、ジョブ追跡オブジェクトが返されます。

1. ジョブ追跡オブジェクトを使用して、ジョブの最新の状態 (`Get-AzRecoveryServicesAsrJob`)を取得し、操作の状態を監視します。

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

1. 複数の Azure リージョンの仮想マシンが同じコンテナーに保護されている場合は、ソース Azure リージョンごとに 1 つのファブリック オブジェクトを作成します。

### 復旧リージョンを表す Site Recovery ファブリック オブジェクトを作成する

復旧ファブリック オブジェクトは、復旧 Azure の場所を表します。フェールオーバーが発生した場合、仮想マシンは復旧ファブリックで表される復旧リージョンにレプリケートされ、復旧されます。この例で使用されている復旧 Azure リージョンは、米国西部 2 です。

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

## Site Recovery 保護コンテナーを作成する

### プライマリ ファブリックに Site Recovery 保護コンテナーを作成する

保護コンテナーは、ファブリック内のレプリケートされたアイテムをグループ化するために使用されるコンテナーです。

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

### 復旧ファブリック内に Site Recovery 保護コンテナーを作成する

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

## レプリケーション ポリシーを作成する

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

## 保護コンテナー マッピングを作成する

### プライマリと復旧の保護コンテナー間の保護コンテナー マッピングを作成する

保護コンテナー マッピングは、プライマリ保護コンテナーを復旧保護コンテナーおよびレプリケーション ポリシーにマッピングします。保護コンテナー ペア間で仮想マシンをレプリケートするために使用するレプリケーション ポリシーごとに 1 つのマッピングを作成します。

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

### フェールバック用の保護コンテナー マッピングを作成する (フェールオーバー後のリバース レプリケーション)

フェールオーバー後、フェールオーバーした仮想マシンを元の Azure リージョンに戻す準備ができたら、フェールバックを実行します。フェール バックするには、フェールオーバーした仮想マシンが、フェールオーバーしたリージョンから元のリージョンに逆レプリケートされます。リバース レプリケーションの場合、元のリージョンと復旧リージョンの役割が切り替わります。元のリージョンが新しい復旧リージョンになり、元の復旧リージョンがプライマリ リージョンになります。リバース レプリケーションの保護コンテナー マッピングは、元のリージョンと復旧リージョンの切り替えロールを表します。

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

## キャッシュ ストレージ アカウントとターゲット ストレージ アカウントを作成する

キャッシュ ストレージ アカウントは、レプリケートされる仮想マシンと同じ Azure リージョン内の標準ストレージ アカウントです。キャッシュ ストレージ アカウントは、変更が復旧 Azure リージョンに移動される前に、レプリケーションの変更を一時的に保持するために使用されます。仮想マシンのディスクごとに異なるキャッシュ ストレージ アカウントを指定する必要はありませんが、選択できます。

```azurepowershell
#Create Cache storage account for replication logs in the primary region
$EastUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestorage" -ResourceGroupName "A2AdemoRG" -Location 'East US' -SkuName Standard_LRS -Kind Storage
```

**Managed Disks を使用しない**仮想マシンの場合、ターゲット ストレージ アカウントは、仮想マシンのディスクがレプリケートされる復旧リージョンのストレージ アカウントです。ターゲット ストレージ アカウントは、標準ストレージ アカウントまたはプレミアム ストレージ アカウントのいずれかです。ディスクのデータ変更率 (IO 書き込み速度) と、ストレージの種類に対する Azure Site Recovery でサポートされるチャーン制限に基づいて、必要なストレージ アカウントの種類を選択します。

```azurepowershell
#Create Target storage account in the recovery region. In this case a Standard Storage account
$WestUSTargetStorageAccount = New-AzStorageAccount -Name "a2atargetstorage" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -SkuName Standard_LRS -Kind Storage
```

## ネットワーク マッピングを作成する

ネットワーク マッピングは、プライマリ リージョンの仮想ネットワークを復旧リージョンの仮想ネットワークにマッピングします。ネットワーク マッピングでは、プライマリ仮想ネットワーク内の仮想マシンがフェールオーバーする復旧リージョンの Azure 仮想ネットワークを指定します。1 つの Azure 仮想ネットワークは、復旧リージョン内の 1 つの Azure 仮想ネットワークにのみマップできます。

1. フェールオーバー先の復旧リージョンに Azure 仮想ネットワークを作成する。

   ```azurepowershell
    #Create a Recovery Network in the recovery region
    $WestUSRecoveryVnet = New-AzVirtualNetwork -Name "a2arecoveryvnet" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -AddressPrefix "10.0.0.0/16"

    Add-AzVirtualNetworkSubnetConfig -Name "default" -VirtualNetwork $WestUSRecoveryVnet -AddressPrefix "10.0.0.0/20" | Set-AzVirtualNetwork

    $WestUSRecoveryNetwork = $WestUSRecoveryVnet.Id
   ```

1. プライマリ仮想ネットワークを取得します。仮想マシンが接続されている VNet:

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

1. プライマリ仮想ネットワークと復旧仮想ネットワークの間にネットワーク マッピングを作成する。

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

1. 逆方向 (フェールバック) のネットワーク マッピングを作成する。

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

## マネージド ディスクを使用している Azure 仮想マシンをレプリケートします。

次の Power Shell コマンドレットを使用して、マネージド ディスクを使用して Azure 仮想マシンをレプリケートします。

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

複数のデータ ディスクのレプリケーションを有効にする場合は、次の Power Shell コマンドレットを使用する。

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

近接配置グループを使用してゾーン間レプリケーションを有効にすると、レプリケーションを開始するコマンドが Power Shell コマンドレットと交換されます。

    ```azurepowershell
    $TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id -RecoveryAvailabilityZone "2"
    ```

レプリケーションを開始する操作が成功すると、仮想マシンのデータがリカバリ領域にレプリケートされます。

レプリケーション プロセスは、最初に、復旧リージョン内で仮想マシンのレプリケートする側のディスクのコピーをシード処理することによって始まります。このフェーズは、*初期レプリケーション* フェーズと呼ばれます。

初期レプリケーションが完了すると、レプリケーションは*差分同期*フェーズに移行します。この時点で、仮想マシンは保護されており、仮想マシンでテスト フェールオーバー操作を実行できます。初期レプリケーションの完了後、仮想マシンを表すレプリケートされた項目のレプリケーション状態は、保護された状態に移行します。

それに対応するレプリケーションの保護された項目の詳細を取得することによって、仮想マシンのレプリケーション状態とレプリケーション正常性を監視します。

    ```azurepowershell
    Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $PrimaryProtContainer | Select FriendlyName, ProtectionState, ReplicationHealth
    ```

## テスト フェール オーバーの実行、検証、クリーンアップ

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

テスト フェールオーバーを行った仮想マシンでテストが完了したら、テスト フェールオーバー操作のクリーンアップを開始して、テスト コピーをクリーンアップします。この操作により、テスト フェールオーバーによって作成された仮想マシンのテスト コピーが削除されます。

```azurepowershell
$Job_TFOCleanup = Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $ReplicationProtectedItem

Get-AzRecoveryServicesAsrJob -Job $Job_TFOCleanup | Select State
```

```Output
State
-----
Succeeded
```

## 仮想枚ｓンをフェール オーバーする

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

## 再保護とソース リージョンへのフェールバック

次の Power Shell コマンドレットを使用して、再保護し、ソース領域にフェール バックします。

```azurepowershell
#Create a cache storage account for replication logs in the primary region.
$WestUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestoragewestus" -ResourceGroupName "A2AdemoRG" -Location 'West US' -SkuName Standard_LRS -Kind Storage

#Use the recovery protection container, the new cache storage account in West US, and the source region VM resource group. 
Update-AzRecoveryServicesAsrProtectionDirection -ReplicationProtectedItem $ReplicationProtectedItem -AzureToAzure -ProtectionContainerMapping $WusToEusPCMapping -LogStorageAccountId $WestUSCacheStorageAccount.Id -RecoveryResourceGroupID $sourceVMResourcegroup.ResourceId -RecoveryProximityPlacementGroupId $vm.ProximityPlacementGroup.Id
```

## レプリケーションを無効にする

`Remove-AzRecoveryServicesAsrReplicationProtectedItem` コマンドレットを使用してレプリケーションを無効にできます。

```azurepowershell
Remove-AzRecoveryServicesAsrReplicationProtectedItem -ReplicationProtectedItem $ReplicationProtectedItem
```