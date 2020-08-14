# Intro to Using the VS Code Debugger for Azure PowerShell

### Prerequisites:
1. Install our IDE `Visual Studio Code`
   * https://code.visualstudio.com/
2. Install the PowerShell extension
   * https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell

### Additional Documentation: 
1. Using Visual Studio Code for PowerShell Development
   * https://docs.microsoft.com/en-us/powershell/scripting/dev-cross-plat/vscode/using-vscode?view=powershell-7
2. Get started with Azure PowerShell
    * https://docs.microsoft.com/en-us/powershell/azure/get-started-azureps?view=azps-4.3.0
    * https://github.com/Azure/azure-powershell
3. PowerShell in Visual Studio Code
    * https://code.visualstudio.com/docs/languages/powershell

## How to Debug PowerShell Scripts
Let's say you have a variable set to 1.

`$myFirstVariable = 1;`

You make a script that is supposed to check if the variable is 5. If not, increment the variable by 1 until it becomes 5.

    while ($myFirstVariable -gt 5) {
       $myFirstVariable++;
    }
    
But when you print it, the variable is still one?!

`Write-Host $myFirstVariable;`

![Uno](https://github.com/rjmccallumbigl/Using-the-VS-Code-Debugger-for-Azure-PowerShell-intro-/blob/master/pics/variable_equals_one.png)

We know that our script is supposed to increase, but it didn't increase at all. Let's debug it by adding a Breakpoint to our *while* loop. A Breakpoint indicates the script needs to pause here while debugging so we can make sure it's working like it's supposed to.

![Enable Debugging](https://github.com/rjmccallumbigl/Using-the-VS-Code-Debugger-for-Azure-PowerShell-intro-/blob/master/pics/enable_debugging.gif)

1. Go to the "Run" tab (or press CTRL + Shift + D). Make sure your script is saved and open in the IDE.
2. Add a Breakpoint by clicking on the empty space to the left of your Row number. A red circle will appear. This indicates your script will pause before executing this line.
3. Run the script clicking "Run and Debug" (or by pressing F5). Your script will pause on the execution line.

Every variable in your session will display it's value in the box to the left. You can also hover over your variables to get their current value (by the time the script has executed). If you have selected a conditional loop, you can click **Step Into** and your script will walk through each line of code in the loop as it executes.

However, there's something wrong with our loop. Instead of showing us our variable incrementing, the script continues without going inside the loop. This means the conditional in our variable is false or has not been met.

### Our conditional

    while ($myFirstVariable -gt 5) {
       $myFirstVariable++;
    }
    
### Our conditional in Pseudocode

    while ($myFirstVariable is greater than 5) {
       Add 1 to $myFirstVariable;
    }

Well there's our problem. According to the [official documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comparison_operators?view=powershell-7), I used the wrong conditional operator. The loop will never execute properly because the number 1 is not greater than the number 5. 

If we fix this by changing `-gt` (greater than) to `-lt` (less than) and debugging again...

![Debugging helped!](https://github.com/rjmccallumbigl/Using-the-VS-Code-Debugger-for-Azure-PowerShell-intro-/blob/master/pics/debugging_helped.gif)

Our script works as intended. Notice the debugger actually "steps into" the function this time, indicating our loop conditional is true (the numbers 1 through 4 are indeed less than 5).

## How to Debug Azure PowerShell Scripts

Now that we know how to debug PowerShell using VS Code, Azure PowerShell isn't too different. 

### Azure PowerShell Variables

For a basic example, let's say I want to see if my Azure VM "RescueVM" is the first VM returned from running `Get-AzVM` (just because). Consider the following script:

      # Let's grab all of our VMs
      $azureVMs = Get-AzVM;

      # And display the first one
      $azureVMs[0];

      # And display the VM that matches the name "RescueVM"
      $azureVMs | where Name -EQ "RescueVM"

By default and without any parameters, `Get-AzVM` will grab all of the VM objects in your subscription and return their properties in an array ([more info](https://docs.microsoft.com/en-us/powershell/module/az.compute/get-azvm?view=azps-4.4.0)). By assigning this function to our variable `$azureVMs`, we accomplish a few things:

1. If we need to see the VM objects again, we don't need to make the call by running Get-AzVM, we just need to print the content of $azureVMs (unless we have made an update). This is good practice because it saves us time and prevents us from exceeding API calls in heavier scripts.
2. We can grab all of our VMs and narrow it down based on our needs by grabbing a VM object from the array. E.G. `$azureVMs[0]` grabs the first VM object returned from this array, while `$azureVMs | Where-Object Name -eq "RescueVM"` returns the full object and properties of the VM that matches the name "RescueVM" ([more info on array functions](https://docs.microsoft.com/en-us/powershell/scripting/learn/deep-dives/everything-about-arrays?view=powershell-7)).

By running our script, we grab all the VMs, display the VM in the first spot of the array, and then display the properties of "RescueVM". Logging variable properties in your script is an easy and quick way to make sure a variable is what it's supposed to be at a certain point in execution. However, this can be clunky if you are working on a production script because cleanup can be a hassle. If you want to dive into all of the variables returned from an Azure PowerShell command such as `Get-AzVM`, you can use the debugger instead for quicker and cleaner results. Let's add a breakpoint to the next line to pause our execution and view the variables in our 3 line script after calling `Get-AzVM`:

![Inspecting our variable](https://github.com/rjmccallumbigl/Using-the-VS-Code-Debugger-for-Azure-PowerShell-intro-/blob/master/pics/azure_powershell_debugging.gif)

We can see all of the VM objects returned in our array, the internal functions such as "length", the return values' types, etc.

(BTW, if you're curious about catFacts.ps1, I was adapting [this script](https://gist.github.com/johnallers/178576f097a8a7986fcb17a92c88a486) to understand PowerShell API calls, definitely not preparing to prank co-workers, I swear.)

### Azure PowerShell Script

Now, let's say we have a slightly more complicated script that creates a new VM ([repo](https://github.com/rjmccallumbigl/Azure-PowerShell---Create-New-VM/blob/master/createNewVM.ps1)). I looked at the documentation, modified it to automatically grab some things, request other parameters to pass in as variables, and handle it mostly automatically.

```

###########################################################################################################################################################
<#
# .SYNOPSIS
#       Create a new Managed VM.
#
# .DESCRIPTION
#       Create a new Managed VM. If you'd like to modify the defaults, please review change the code prior to running.
#       https://docs.microsoft.com/en-us/powershell/module/az.compute/new-azvm?view=azps-4.4.0
#
# .PARAMETER vmName
#       The name of the VM.
#
# .PARAMETER VMLocalAdminUser
#       The username you will use on your VM.
#
# .PARAMETER VMLocalAdminSecurePassword
#       The password you will use on your VM.
#
# .EXAMPLE
#       createNewVMs -n name -u Username -p Password!234
#>
###########################################################################################################################################################

# Set the Parameters for the script
param (
    [Parameter(Mandatory = $true, HelpMessage = "The name of the VM.")]
    [Alias('n')]
    [string] 
    $vmName,
    [Parameter(Mandatory = $true, HelpMessage = "The username you will use on your VM.")]
    [Alias('u')]
    [string]
    $VMLocalAdminUser,
    [Parameter(Mandatory = $true, HelpMessage = "The password you will use on your VM.")]
    [Alias('p')]
    [SecureString]
    $VMLocalAdminSecurePassword
)

# Declare variables, modify as necessary
$LocationName = "eastus"
$publisherName = "MicrosoftWindowsServer"
$offerName = "windowsserver"
$skuName = "2019-Datacenter"
$ResourceGroupName = $VMName + "RG"
$VMSize = "Standard_D2_v3"
$NetworkName = $VMName + "Net"
$NICName = $VMName + "NIC"
$SubnetName = $VMName + "Subnet"
$SubnetAddressPrefix = "10.0.0.0/24"
$VnetAddressPrefix = "10.0.0.0/16"
$PublicIPAddressName = $VMName + "PIP"

# Create resources
New-AzResourceGroup -Name $ResourceGroupName -Location $LocationName
$SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix
$Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroupName -Location $LocationName -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet
$PIP = New-AzPublicIpAddress -Name $PublicIPAddressName -ResourceGroupName $ResourceGroupName -Location $LocationName -AllocationMethod Dynamic
$NIC = New-AzNetworkInterface -Name $NICName -ResourceGroupName $ResourceGroupName -Location $LocationName -SubnetId $Vnet.Subnets[0] -PublicIpAddressId $PIP
$Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword)

# Create VM configuration
$VirtualMachine = New-AzVMConfig -VMName $VMName -VMSize $VMSize
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $VMName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName $publisherName -Offer $offerName -Skus $skuName -Version latest

# Create VM
New-AzVM -ResourceGroupName $ResourceGroupName -Location $LocationName -VM $VirtualMachine -Verbose
```

The parameters are accepted and the Resource Group is created:

```
PS C:\Users\rymccall\OneDrive - Microsoft\PowerShell> c:\Users\rymccall\Github\Azure-PowerShell---Create-New-VM\createNewVM.ps1
cmdlet createNewVM.ps1 at command pipeline position 1
Supply values for the following parameters:
vmName: examplevm
VMLocalAdminUser: rymccall
VMLocalAdminSecurePassword: *****************


ResourceGroupName : examplevmRG
Location          : eastus
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/81d1b603-b602-4534-952e-a8889d3421a1/resourceGroups/examplevmRG

```

But it looks like we're getting error after error (warnings and verbose logging removed):

```
New-AzNetworkInterface : Property id 'Microsoft.Azure.Commands.Network.Models.PSSubnet' at path 'properties.ipConfigurations[0].properties.subnet.id' is invalid. Expect fully qualified resource Id that 
start with '/subscriptions/{subscriptionId}' or '/providers/{resourceProviderNamespace}/'.
StatusCode: 400
ReasonPhrase: Bad Request
ErrorCode: LinkedInvalidPropertyId
ErrorMessage: Property id 'Microsoft.Azure.Commands.Network.Models.PSSubnet' at path 'properties.ipConfigurations[0].properties.subnet.id' is invalid. Expect fully qualified resource Id that start with  
'/subscriptions/{subscriptionId}' or '/providers/{resourceProviderNamespace}/'.
OperationID : 07a40ecb-c034-4380-8ad2-2cbcf56dec7c
At C:\Users\rymccall\Github\Azure-PowerShell---Create-New-VM\createNewVM.ps1:59 char:8
+ $NIC = New-AzNetworkInterface -Name $NICName -ResourceGroupName $Reso ...
+        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [New-AzNetworkInterface], NetworkCloudException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Network.NewAzureNetworkInterfaceCommand

Add-AzVMNetworkInterface : Cannot validate argument on parameter 'Id'. The argument is null or empty. Provide an argument that is not null or empty, and then try the command again.
At C:\Users\rymccall\Github\Azure-PowerShell---Create-New-VM\createNewVM.ps1:65 char:68
+ ... ualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
+                                                                   ~~~~~~~
    + CategoryInfo          : InvalidData: (:) [Add-AzVMNetworkInterface], ParameterBindingValidationException
    + FullyQualifiedErrorId : ParameterArgumentValidationError,Microsoft.Azure.Commands.Compute.AddAzureVMNetworkInterfaceCommand

VERBOSE: Performing the operation "New" on target "examplevm".
New-AzVM : Required parameter 'networkProfile' is missing (null).
ErrorCode: InvalidParameter
ErrorMessage: Required parameter 'networkProfile' is missing (null).
ErrorTarget: networkProfile
StatusCode: 400
ReasonPhrase: Bad Request
OperationID : bb9bbad1-365a-484c-b46d-9a8017121da7
At C:\Users\rymccall\Github\Azure-PowerShell---Create-New-VM\createNewVM.ps1:69 char:1
+ New-AzVM -ResourceGroupName $ResourceGroupName -Location $LocationNam ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [New-AzVM], ComputeCloudException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Compute.NewAzureVMCommand
```
Needless to say, our script failed and our VM was not created. Okay, so our code has some bugs. No problem, let's break this down by the Error Messages. It looks like there are 3 errors here:

```
ErrorMessage: Property id 'Microsoft.Azure.Commands.Network.Models.PSSubnet' at path 'properties.ipConfigurations[0].properties.subnet.id' is invalid. Expect fully qualified resource Id that start with  
'/subscriptions/{subscriptionId}' or '/providers/{resourceProviderNamespace}/'.
OperationID : 07a40ecb-c034-4380-8ad2-2cbcf56dec7c
At C:\Users\rymccall\Github\Azure-PowerShell---Create-New-VM\createNewVM.ps1:59 char:8
+ $NIC = New-AzNetworkInterface -Name $NICName -ResourceGroupName $Reso ...
```

   * This error message is saying the property we entered on line 59 is incorrect because it's not a resource Id. Thus this command failed, and our variable `$NIC` is broke, null, and void.

```
Add-AzVMNetworkInterface : Cannot validate argument on parameter 'Id'. The argument is null or empty. Provide an argument that is not null or empty, and then try the command again.
At C:\Users\rymccall\Github\Azure-PowerShell---Create-New-VM\createNewVM.ps1:65 char:68
+ ... ualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
```

   * This error message is saying the parameter we entered on line 65 at the 68th character is incorrect because it's a null or empty value. This value is `$NIC.Id` in our script. The previous error failed to set `$NIC` correctly. Thus this command failed because our object `$NIC` is broke, null, and void, and subsequently, there is no Id property.

```
ErrorMessage: Required parameter 'networkProfile' is missing (null).
ErrorTarget: networkProfile
StatusCode: 400
ReasonPhrase: Bad Request
OperationID : bb9bbad1-365a-484c-b46d-9a8017121da7
At C:\Users\rymccall\Github\Azure-PowerShell---Create-New-VM\createNewVM.ps1:69 char:1
+ New-AzVM -ResourceGroupName $ResourceGroupName -Location $LocationNam ...
```

   * This final error message is the response that we failed to create our VM using the supplied script due to a broken networkProfile. This is because of the same initial error with our `$NIC` object. So basically, if the Networking portion is broken, we will not be able to successfully provision a VM using our script.

So we understand our error messages and we have narrowed it down to the declared `$NIC` object on line 59 grandfathering all of our other problems. But we don't know the proper format for this parameter. What now?

Let's use the Debugger!



### Advanced Debugging: 
1. Debugging PowerShell script in Visual Studio Code – Part 1
   * https://devblogs.microsoft.com/scripting/debugging-powershell-script-in-visual-studio-code-part-1/
2. Debugging PowerShell script in Visual Studio Code – Part 2
   * https://devblogs.microsoft.com/scripting/debugging-powershell-script-in-visual-studio-code-part-2/