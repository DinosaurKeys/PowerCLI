PowerCLI Comprehensive Notes for VMware Administration
This document consolidates common and essential PowerCLI commands, organized for quick reference. It's intended as a living document that you can expand and customize based on your specific VMware environment and administrative needs.

PowerCLI is a command-line interface for managing and automating VMware vSphere, vSAN, NSX-T, VMware Cloud on AWS, VMware Cloud Director, and other VMware products. It's built on PowerShell and provides cmdlets (command-let) that interact with VMware APIs.

Table of Contents
Getting Started: Installation & Connection

Virtual Machine (VM) Management

Host (ESXi) Management

Storage (Datastore) Management

Networking Management

Reporting & Inventory

Automation & Scripting Best Practices

1. Getting Started: Installation & Connection
1.1. Installation & Module Management
Check if PowerCLI module is installed:

Get-Module -ListAvailable -Name VMware.PowerCLI

Install PowerCLI module (Run as Administrator in PowerShell):

Install-Module -Name VMware.PowerCLI -Scope CurrentUser
# Or for all users:
# Install-Module -Name VMware.PowerCLI -Scope AllUsers

Note: You might need to set your PowerShell execution policy first: Set-ExecutionPolicy RemoteSigned

Update PowerCLI module:

Update-Module -Name VMware.PowerCLI

Uninstall PowerCLI module:

Uninstall-Module -Name VMware.PowerCLI

1.2. Connecting to vCenter Server or ESXi Host
Connect securely (prompts for credentials):

Connect-VIServer -Server <vCenter_IP_or_Hostname>

Connect with specified credentials (not recommended for scripts, use SecureString for production):

Connect-VIServer -Server <vCenter_IP_or_Hostname> -User <username> -Password <password>

To prompt for username and password securely:

$cred = Get-Credential
Connect-VIServer -Server <vCenter_IP_or_Hostname> -Credential $cred

Allow invalid certificates (use with caution in test environments only):

Set-PowerCLIConfiguration -InvalidCertificateAction Ignore -Confirm:$false
Connect-VIServer -Server <vCenter_IP_or_Hostname>

1.3. Disconnecting from vCenter Server
Disconnect from all connected servers:

Disconnect-VIServer -Confirm:$false

Disconnect from a specific server:

Disconnect-VIServer -Server <vCenter_IP_or_Hostname> -Confirm:$false

1.4. Basic Help & Information
Get help for a specific cmdlet:

Get-Help Connect-VIServer -Full
Get-Help Get-VM -Examples

List all PowerCLI cmdlets:

Get-Command -Module VMware.PowerCLI

2. Virtual Machine (VM) Management
2.1. Getting VMs
Get all VMs:

Get-VM

Get a specific VM by name:

Get-VM -Name "MyVM01"

Get VMs matching a pattern:

Get-VM -Name "ProdVM*"

Get VMs on a specific host:

Get-VM -Location (Get-VMHost "esxi-host-01")

Get VMs with a specific power state:

Get-VM | Where-Object {$_.PowerState -eq "PoweredOn"}

Get VM properties (select specific properties):

Get-VM "MyVM01" | Select Name, NumCpu, MemoryGB, PowerState, Guest

2.2. VM Power Operations
Start a VM:

Start-VM -VM "MyVM01" -Confirm:$false

Stop a VM (graceful shutdown via VMware Tools):

Stop-VM -VM "MyVM01" -Confirm:$false

Stop a VM (hard power off, equivalent to pulling the plug):

Stop-VM -VM "MyVM01" -Confirm:$false -Kill

Restart a VM (graceful restart via VMware Tools):

Restart-VM -VM "MyVM01" -Confirm:$false

Suspend a VM:

Suspend-VM -VM "MyVM01" -Confirm:$false

2.3. VM Snapshot Management
Create a snapshot:

New-Snapshot -VM "MyVM01" -Name "BeforePatch" -Description "Snapshot before applying OS patches"

Get snapshots for a VM:

Get-Snapshot -VM "MyVM01"

Revert a VM to a specific snapshot:

Get-Snapshot -VM "MyVM01" -Name "BeforePatch" | Set-Snapshot -Revert -Confirm:$false

Remove a specific snapshot:

Get-Snapshot -VM "MyVM01" -Name "BeforePatch" | Remove-Snapshot -Confirm:$false

Remove all snapshots for a VM:

Get-VM "MyVM01" | Get-Snapshot | Remove-Snapshot -RemoveChildren -Confirm:$false

2.4. VM Creation, Cloning & Templates
Create a new VM (basic example):

New-VM -Name "NewWebSrv01" -VMHost (Get-VMHost "esxi-host-01") -Datastore (Get-Datastore "Datastore01") -NumCpu 2 -MemoryGB 4 -DiskGB 50 -NetworkName "VM Network" -GuestId "windows2019srv_64Guest"

Note: GuestId can be found using Get-VMGuestConfiguration | Select Id, Description

Clone a VM:

Get-VM "SourceVM" | New-VM -Name "ClonedVM01" -VMHost (Get-VMHost "esxi-host-02") -Datastore (Get-Datastore "Datastore02")

Clone a VM to a template:

Get-VM "SourceVM" | New-Template -Name "Ubuntu18Template" -Location (Get-Folder "Templates")

Convert VM to Template:

Set-VM -VM "MyVM01" -ToTemplate

Convert Template to VM:

Set-Template -Template "Ubuntu18Template" -ToVM

Deploy VM from Template:

New-VM -Name "NewAppSrv" -Template "Ubuntu18Template" -VMHost (Get-VMHost "esxi-host-03") -Datastore (Get-Datastore "Datastore03") -OSCustomizationSpec (Get-OSCustomizationSpec "LinuxCustomization")

2.5. VM Configuration
Change VM CPU count and memory:

Set-VM -VM "MyVM01" -NumCpu 4 -MemoryGB 8 -Confirm:$false

Add a new hard disk to a VM:

New-HardDisk -VM "MyVM01" -CapacityGB 100 -StorageFormat Thin -Datastore (Get-Datastore "Datastore01")

Remove a hard disk (be very careful!):

Get-VM "MyVM01" | Get-HardDisk -Name "Hard Disk 2" | Remove-HardDisk -Confirm:$false

Add a new network adapter:

New-NetworkAdapter -VM "MyVM01" -NetworkName "DMZ Network" -Type Vmxnet3 -Confirm:$false

Change network adapter settings:

Get-VM "MyVM01" | Get-NetworkAdapter -Name "Network adapter 1" | Set-NetworkAdapter -Type E1000 -Confirm:$false

2.6. Guest OS Interaction (Requires VMware Tools)
Execute a command inside a guest OS:

Invoke-VMScript -VM "MyVM01" -GuestCredential (Get-Credential) -ScriptText "ipconfig /all" -ScriptType Bat

Copy a file to guest OS:

Copy-VMGuestFile -VM "MyVM01" -Source "C:\local_file.txt" -Destination "C:\temp\remote_file.txt" -GuestCredential (Get-Credential) -LocalToGuest

Copy a file from guest OS:

Copy-VMGuestFile -VM "MyVM01" -Source "C:\windows\system32\drivers\etc\hosts" -Destination "C:\temp\hosts_file.txt" -GuestCredential (Get-Credential) -GuestToLocal

3. Host (ESXi) Management
3.1. Getting Hosts
Get all ESXi hosts:

Get-VMHost

Get a specific ESXi host by name:

Get-VMHost -Name "esxi-host-01"

Get hosts in a specific cluster:

Get-Cluster "ProductionCluster" | Get-VMHost

3.2. Host Configuration
Set host NTP server:

Get-VMHost "esxi-host-01" | Set-VMHostNtpServer -NtpServer "pool.ntp.org"

Start NTP service on host:

Get-VMHost "esxi-host-01" | Start-VMHostService -Service (Get-VMHostService -VMHost "esxi-host-01" | Where-Object {$_.Key -eq "ntpd"})

Set host DNS configuration:

Get-VMHost "esxi-host-01" | Set-VMHostNetwork -DnsAddress "8.8.8.8", "8.8.4.4" -DnsSearchDomains "mylocal.domain"

3.3. Maintenance Mode
Enter maintenance mode (VMs must be migrated or shut down):

Set-VMHost -VMHost "esxi-host-01" -State Maintenance -Confirm:$false

Exit maintenance mode:

Set-VMHost -VMHost "esxi-host-01" -State Connected -Confirm:$false

4. Storage (Datastore) Management
4.1. Getting Datastores
Get all datastores:

Get-Datastore

Get a specific datastore by name:

Get-Datastore -Name "Datastore01"

Get datastores accessible by a specific host:

Get-VMHost "esxi-host-01" | Get-Datastore

Get datastores with free space greater than a threshold:

Get-Datastore | Where-Object {$_.FreeSpaceGB -gt 500}

4.2. Datastore Operations
Browse a datastore:

Get-Datastore "Datastore01" | Get-DatastoreItem

Create a new folder on a datastore:

New-DatastoreFolder -Datastore "Datastore01" -Name "ISO_Images"

Copy a file to a datastore:

Copy-DatastoreItem -Item "C:\MyVMDK.vmdk" -Destination "vmstore:\Datastore01\MyVMs\NewVM\"

Remove a datastore (be extremely careful!):

Remove-Datastore -Datastore "OldDatastore" -Confirm:$true

5. Networking Management
5.1. Getting Switches & Port Groups
Get all standard vSwitches:

Get-VirtualSwitch

Get standard vSwitches on a specific host:

Get-VMHost "esxi-host-01" | Get-VirtualSwitch

Get all distributed vSwitches:

Get-VDSwitch

Get all port groups (standard & distributed):

Get-VirtualPortGroup
Get-VDPortgroup

5.2. vSwitch & Port Group Configuration
Create a new standard vSwitch:

New-VirtualSwitch -Name "vSwitch1" -VMHost (Get-VMHost "esxi-host-01") -Nic "vmnic1"

Create a new port group on a standard vSwitch:

New-VirtualPortGroup -Name "Prod_Network" -VirtualSwitch (Get-VirtualSwitch "vSwitch0" -VMHost "esxi-host-01") -VLanId 100

Add a physical NIC to a vSwitch:

Add-VirtualSwitchPhysicalNetworkAdapter -VirtualSwitch (Get-VirtualSwitch "vSwitch0" -VMHost "esxi-host-01") -PhysicalNic (Get-VMHostNetworkAdapter -VMHost "esxi-host-01" -Physical | Where-Object {$_.Name -eq "vmnic2"})

Set VMkernel adapter IP address:

Get-VMHostNetworkAdapter -VMHost "esxi-host-01" -VMkernel | Where-Object {$_.PortGroupName -eq "Management Network"} | Set-VMHostNetworkAdapter -IP "192.168.1.10" -SubnetMask "255.255.255.0"

6. Reporting & Inventory
6.1. Common Reporting Examples
List all VMs with their power state, CPU, and memory:

Get-VM | Select Name, PowerState, NumCpu, MemoryGB, @{N='OS';E={$_.Guest.OSFullName}} | Format-Table -AutoSize

List all hosts with their model, build, and uptime:

Get-VMHost | Select Name, Manufacturer, Model, Build, Version, @{N='UptimeDays';E={($_.State.UptimeSeconds / 86400).ToString("N0")}} | Format-Table -AutoSize

List all datastores with their capacity and free space:

Get-Datastore | Select Name, Type, CapacityGB, FreeSpaceGB, @{N='FreeSpace%';E={($_.FreeSpaceGB / $_.CapacityGB * 100).ToString("N2")}} | Format-Table -AutoSize

Find VMs with old snapshots (e.g., older than 7 days):

Get-VM | Get-Snapshot | Where-Object {$_.Created -lt (Get-Date).AddDays(-7)} | Select VM, Name, Created, SizeGB | Format-Table -AutoSize

6.2. Exporting Data
Export VM list to CSV:

Get-VM | Select Name, PowerState, NumCpu, MemoryGB | Export-Csv -Path "C:\Temp\VM_Inventory.csv" -NoTypeInformation -UseCulture

Export host network configuration to CSV:

Get-VMHostNetworkAdapter | Select VMHost, Name, Mac, IP, SubnetMask, PortGroupName | Export-Csv -Path "C:\Temp\Host_Network_Config.csv" -NoTypeInformation -UseCulture

Export to JSON:

Get-VM | Select Name, PowerState, NumCpu, MemoryGB | ConvertTo-Json | Out-File "C:\Temp\VM_Inventory.json"

7. Automation & Scripting Best Practices
7.1. Error Handling
Using try-catch-finally:

try {
    Get-VM -Name "NonExistentVM" -ErrorAction Stop
    Write-Host "This line will not be reached if an error occurs."
}
catch {
    Write-Host "An error occurred: $($_.Exception.Message)"
    # You can add custom error handling logic here, e.g., logging
}
finally {
    Write-Host "This block always executes."
}

Setting ErrorAction Preference:

$ErrorActionPreference = "Stop" # Or "Continue", "SilentlyContinue", "Inquire"
# Cmdlet specific: Get-VM -Name "NonExistentVM" -ErrorAction Stop

7.2. Pipelines
PowerCLI heavily leverages the PowerShell pipeline, allowing you to pass objects from one cmdlet to the next.

Example: Get a VM and then restart it:

Get-VM -Name "MyVM01" | Restart-VM -Confirm:$false

Example: Get all VMs in a cluster and power them off:

Get-Cluster "TestCluster" | Get-VM | Stop-VM -Confirm:$false -Kill

7.3. Variables & Objects
Storing cmdlet output in variables:

$myVM = Get-VM "MyVM01"
$myVM.MemoryGB = 16
Set-VM -VM $myVM -Confirm:$false

Exploring object properties:

Get-VM "MyVM01" | Get-Member
Get-VM "MyVM01" | Format-List *

7.4. Functions
Encapsulate reusable code into functions.

function Get-VMInfo {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
        [string]$Name
    )
    process {
        try {
            $vm = Get-VM -Name $Name -ErrorAction Stop
            Write-Output "VM Name: $($vm.Name)"
            Write-Output "Power State: $($vm.PowerState)"
            Write-Output "CPU Count: $($vm.NumCpu)"
            Write-Output "Memory (GB): $($vm.MemoryGB)"
        }
        catch {
            Write-Error "Could not retrieve info for VM '$Name': $($_.Exception.Message)"
        }
    }
}

# Example usage:
Get-VMInfo -Name "MyVM01"
"MyVM02", "MyVM03" | Get-VMInfo

This is a solid foundation for your PowerCLI notes. Feel free to copy and paste this into a Markdown file (e.g., PowerCLI_Notes.md) and upload it to your GitHub or GitLab repository.