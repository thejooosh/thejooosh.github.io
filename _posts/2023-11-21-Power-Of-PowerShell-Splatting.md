---
title: The Power of Powershell Splatting 
---

## Basics of Splatting

Splatting, while well-known by most Powershell scripters, still tends to be underutilized by many. Splatting allows you to create an object with command parameters and then pass that object into a function. The function will evaluate the object members as the parameters themselves.

Let's say we want to call the `Get-ChildItem` function, we want to define the path to `C:\` with a recursion depth of `2`.

A normal one-liner of such a command would look like this:

```powershell
Get-ChildItem -Path 'C:\' -Depth 2 
```

With the power of splatting, we can create an object with each param and their respective value, which would look like this:

```powershell
$GetChildItemSplat = @{
    Path = 'C:\'
    Depth = 2 
}

Get-ChildItem @GetChildItemSplat
```

You can also splat using an array, here's an example from [Microsoft's Documentation](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting?view=powershell-7.4):
```Powershell
$ArrayArguments = "test.txt", "test2.txt"
Copy-Item @ArrayArguments
```
## Benefits of Splatting

So, what's the benefit of using splatting?

1. Improves the readability of your code
2. Improves your ability to catch errors/issues
3. Allows more advanced logic when defining function parameters without making it more difficult to read

## Splatting with Function Parameters

Now that we know the basics of splatting, we can talk about a few scenarios where splatting can be useful for writing a syntactic sugar function as an example.

In this first example, we still want to use the `Get-ChildItem` function, but in this scenario, we want to check the path and ensure we're only checking the `C:\` path when running `Get-ChildItem`. I cannot think of any real-world scenario where this would be useful, but it provides a simple platform to show the power of Splatting.

Let's start by defining a simple function that returns the Get-ChildItem value

```powershell
Function Invoke-MyGetChildItem {
    Get-ChildItem -Path 'C:\' -Depth 2
}
```

Running this function will invoke the Get-ChildItem with a path of `C:\` and a depth of `2`.
Now let's add our own Path and Depth parameter to our function.

Some of you may be inclined to write the function similar to this:

```powershell
Function Invoke-MyGetChildItem {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]$Path,
        [Parameter(Mandatory)]
        [int]$Depth
    )
    Get-ChildItem -Path $Path -Depth $Depth
}
```

However, this is where the power of Splatting really begins to shine. We can pass the `$PSBoundParameters` variable to the `Get-ChildItem` function as a splat.

```powershell
Function Invoke-MyGetChildItem {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]$Path,
        [Parameter(Mandatory)]
        [int]$Depth
    )
    Get-ChildItem @PSBoundParameters
}
```

There is an alternative, which is arguably wrong, and a waste of space.

```powershell
Function Invoke-MyGetChildItem {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]$Path,
        [Parameter(Mandatory)]
        [int]$Depth
    )
    $ChildItemSplat = @{
        Path = $Path
        Depth = $Depth
    }
    Get-ChildItem @ChildItemSplat
}
```

Definitely don't do the last example. As you can see it's repetitive. Since the variable names align with the function, it makes it a great option.
The benefit of calling $PSBoundParamters is that only parameters that are called via a function will be passed.
If we have a list of 5 params, some of which are optional, when passing the $PSBoundParameters to a function as a splat, the splat will only contain the parameters specified.

Let's add an additional optional parameter called `ItemType` to our function. ItemType is also a parameter of the `Get-ChildItem` function.

```powershell
Function Invoke-MyGetChildItem {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]$Path,
        [Parameter(Mandatory)]
        [int]$Depth,
        [string]$ItemType
    )
    Write-Output $PSBoundParameters
    #Get-ChildItem @PSBoundParameters
}
```

We can now call that function without the `ItemType` param.

```powershell
Invoke-MyGetChildItem -Path 'c:\' -Depth 2

Key   Value
---   -----
Path  c:\
Depth 2
```

You can see this function only returns the two parameters we specified. Meaning, these are the only two parameters provided in `$PSBoundParameters` Splat we'll be providing to `Get-ChildItem`.

## Splatting with ScriptBlocks
In another example, we can show that you can add a ScriptBlock to a splat and then pass that to a function. See the example below.
```powershell
$StartJobSplat = @{
    Name = 'Test Job'
    ScriptBlock = {
        Get-ChildItem -Path 'C:\'
    }
}

Start-Job @StartJobSplat
```
