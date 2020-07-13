# Intro to Using the VS Code Debugger for Azure PowerShell

### Prerequisites:
1. Install our IDE VS Code: https://code.visualstudio.com/
2. Install the PowerShell extension: https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell

### Additional Documentation: 
1. Using Visual Studio Code for PowerShell Development: https://docs.microsoft.com/en-us/powershell/scripting/dev-cross-plat/vscode/using-vscode?view=powershell-7
2. Get started with Azure PowerShell
    * https://docs.microsoft.com/en-us/powershell/azure/get-started-azureps?view=azps-4.3.0
    * https://github.com/Azure/azure-powershell

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
