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

Now, let's say we have a slightly more complicated script that creates a new VM.
<!-- TODO: Add complicated script, like the one that creates a new VM -->

### Advanced Debugging: 
1. Debugging PowerShell script in Visual Studio Code – Part 1
   * https://devblogs.microsoft.com/scripting/debugging-powershell-script-in-visual-studio-code-part-1/
2. Debugging PowerShell script in Visual Studio Code – Part 2
   * https://devblogs.microsoft.com/scripting/debugging-powershell-script-in-visual-studio-code-part-2/