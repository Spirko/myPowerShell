# myPowerShell
PowerShell techniques and scripts

## Cleaning old User Profiles

Our lab computers become cluttered and useless after a few years.  Freeing up disc space from students who attended years ago is one way to restore them.

### Snippets

Here are some PowerShell code snippets that go into the main script(s):

```
# List all of the UserProfile CIM objects
Get-CimInstance Win32_UserProfile
```

```
# List the home folders
( Get-CimInstance Win32_UserProfile ).LocalPath
```

Methods of filtering, using WQL.  % is a wildcard:
```
# Filter the list for a certain username, using built-in `-Filter` or `Where-Object`
Get-CimInstance Win32_UserProfile -Filter "LocalPath LIKE '%username'"
Get-CimInstance Win32_UserProfile -Filter "NOT LocalPath LIKE '%username'"
Get-CimInstance Win32_UserProfile | Where-Object { $_.LocalPath -like '*username' }
Get-CimInstance Win32_UserProfile | Where-Object { $_.LocalPath -notlike '*username' }
```

Lookup the Domain account name (can throw error):
```
... | ForEach-Object { $cim = $_ 
   $ntAccount = (New-Object
       System.Security.Principal.SecurityIdentifier($cim.SID)
      ).Translate([System.Security.Principal.NTAccount])
}
```

### Full Commands

List Stale accounts.  Uncomment the Remove line to delete them.
```
Get-CimInstance Win32_UserProfile | ForEach-Object {
   $cim = $_
   try {
     $ntAccount = (New-Object System.Security.Principal.SecurityIdentifier($_.SID)
      ).Translate([System.Security.Principal.NTAccount]).Value
     # These are the valit domain accounts.
     # Write-Host $cim.LocalPath $ntAccount
   } catch {
     # These are the stale domain accounts.
     Write-Host $cim.LocalPath
     # $cim | Remove-CimInstance -Confirm:$false
   }
 }
```

### PowerShell Functions

Delete UserProfile by Path:
```
function Remove-UserProfile-by-Path {
  param(
    [Parameter(Mandatory = $true)]
    [string]$LocalPath
  )

  try {
    $profile = Get-CimInstance -ClassName Win32_UserProfile -Filter "LocalPath = '$LocalPath'"

    if (-not $profile) {
      Write-Warning "User profile not found at '$LocalPath'."
      return $false # Indicate failure
    }

    $result = $profile.Delete()

    if ($result.ReturnValue -eq 0) {
      Write-Host "User profile deleted successfully."
    } else {
      Write-Error "Failed to delete user profile. Error code: $($result.ReturnValue)"
      return $false # Indicate failure
    }

    if (Test-Path $LocalPath) {
      try {
        Remove-Item -Path $LocalPath -Force -Recurse -ErrorAction Stop
        Write-Host "Profile directory removed."
      }
      catch {
        Write-Error "Failed to remove profile directory: $_"
        return $false # Indicate failure
      }
    }

    return $true # Indicate success

  }
  catch {
    Write-Error "An unexpected error occurred: $_"
    return $false # Indicate failure
  }
}
```
