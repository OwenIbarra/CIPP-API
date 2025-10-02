# CIPP-API Quick Reference Guide for Code Quality

This is a quick reference for developers working on CIPP-API. For detailed explanations, see [CODING_IMPROVEMENTS.md](./CODING_IMPROVEMENTS.md).

## Quick Checklist for New Code

- [ ] Function has proper documentation (SYNOPSIS, DESCRIPTION, PARAMETERS, EXAMPLES)
- [ ] Parameters use `[CmdletBinding()]` and proper validation
- [ ] No `Write-Host` - use `Write-LogMessage` instead
- [ ] Proper error handling with `try/catch` and logging
- [ ] No empty catch blocks
- [ ] No hard-coded URLs or magic strings
- [ ] Follows naming conventions (Verb-CippNoun)
- [ ] Parameters use PascalCase
- [ ] Passes PSScriptAnalyzer

## Common Patterns

### Function Template
```powershell
function Verb-CippNoun {
    <#
    .SYNOPSIS
        Brief description
    .DESCRIPTION
        Detailed description
    .PARAMETER TenantFilter
        Tenant identifier
    .EXAMPLE
        Verb-CippNoun -TenantFilter 'contoso.onmicrosoft.com'
    #>
    [CmdletBinding()]
    param(
        [Parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [string]$TenantFilter,
        
        [Parameter(Mandatory = $false)]
        [string]$APIName = 'Function Name'
    )
    
    try {
        # Your code here
        Write-LogMessage -API $APIName -message 'Success message' -Sev 'Info'
        return $result
    } catch {
        $ErrorMessage = Get-CippException -Exception $_
        Write-LogMessage -API $APIName -message "Error: $($ErrorMessage.NormalizedError)" -Sev 'Error' -LogData $ErrorMessage
        throw
    }
}
```

### Error Handling
```powershell
# ✅ Good
try {
    Do-Something
} catch {
    Write-LogMessage -API 'MyAPI' -message "Error: $($_.Exception.Message)" -Sev 'Error'
    throw
}

# ❌ Bad
try {
    Do-Something
} catch {
    # Silent failure
}
```

### Logging
```powershell
# ✅ Good
Write-LogMessage -API 'MyAPI' -message 'User action completed' -Sev 'Info'
Write-Verbose "Detailed information for debugging"

# ❌ Bad
Write-Host "User action completed"
```

### Parameter Validation
```powershell
# ✅ Good
[Parameter(Mandatory = $true)]
[ValidateSet('Option1', 'Option2', 'Option3')]
[string]$Choice

# ❌ Bad
[string]$Choice  # No validation
```

### Collections
```powershell
# ✅ Good - Fast
$results = [System.Collections.Generic.List[object]]::new()
foreach ($item in $items) {
    $results.Add($item)
}

# ❌ Bad - Slow
$results = @()
foreach ($item in $items) {
    $results += $item
}
```

## Run Quality Checks

```powershell
# Run PSScriptAnalyzer with project settings
Invoke-ScriptAnalyzer -Path ./Modules/CIPPCore -Settings ./PSScriptAnalyzerSettings.psd1 -Recurse

# Run on specific file
Invoke-ScriptAnalyzer -Path ./path/to/file.ps1 -Settings ./PSScriptAnalyzerSettings.psd1

# Exclude directories manually
Invoke-ScriptAnalyzer -Path . -Recurse -ExcludeRule PSAvoidUsingWriteHost -Exclude '.git','node_modules','Cache_*'

# Run tests (when implemented)
Invoke-Pester -Path ./Tests
```

## Common Issues to Avoid

1. **Empty catch blocks** - Always log errors
2. **Write-Host** - Use Write-LogMessage or Write-Verbose
3. **Magic strings** - Use constants
4. **No parameter validation** - Use [ValidateSet], [ValidateNotNullOrEmpty], etc.
5. **Long functions** - Break into smaller, focused functions
6. **No documentation** - Add comment-based help
7. **Inconsistent naming** - Follow Verb-CippNoun pattern

## PSScriptAnalyzer Quick Fixes

### Warning: PSAvoidUsingWriteHost
```powershell
# Replace
Write-Host "Message"

# With
Write-LogMessage -API 'API' -message "Message" -Sev 'Info'
```

### Warning: PSAvoidUsingEmptyCatchBlock
```powershell
# Replace
catch { }

# With
catch {
    Write-LogMessage -API 'API' -message "Error: $($_.Exception.Message)" -Sev 'Warning'
}
```

### Warning: PSUseDeclaredVarsMoreThanAssignments
```powershell
# Remove unused variables or use them
```

## EditorConfig

The `.editorconfig` file enforces:
- UTF-8 encoding
- 4 spaces for PowerShell files
- 2 spaces for JSON
- LF line endings for code files
- CRLF for markdown/text

Your IDE should respect these settings automatically.

## Resources

- [Full Improvements Guide](./CODING_IMPROVEMENTS.md)
- [PSScriptAnalyzer Rules](https://github.com/PowerShell/PSScriptAnalyzer)
- [PowerShell Best Practices](https://docs.microsoft.com/en-us/powershell/scripting/developer/cmdlet/cmdlet-development-guidelines)
