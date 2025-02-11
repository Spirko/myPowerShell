# myPowerShell
PowerShell techniques and scripts

## Cleaning old User Profiles

Our lab computers become cluttered and useless after a few years.  Freeing up disc space from students who attended years ago is one way to restore them.

Here are some PowerShell code snippets that go into the main script(s):

```
# List all of the UserProfile CIM objects
Get-CimInstance Win32_UserProfile
```

```
# List the home folders
(Get-CimInstance Win32_UserProfile).LocalPath
```

Methods of filtering, using WQL.  % is a wildcard:
```
# Filter the list for a certain username
Get-CimInstance Win32_UserProfile -Filter "LocalPath LIKE '%username'"
Get-CimInstance Win32_UserProfile -Filter "NOT LocalPath LIKE '%username'"
```

Lookup the Domain account name:
```
... | ForEach-Object { $cim = $_ 
   $ntAccount = (New-Object
       System.Security.Principal.SecurityIdentifier($cim.SID)
      ).Translate([System.Security.Principal.NTAccount])
}
```

Print LocalPaths, ntAccounts (if available) and be ready to remove:
```
Get-CimInstance Win32_UserProfile | ForEach-Object {
   $cim = $_
   try {
     $ntAccount = (New-Object System.Security.Principal.SecurityIdentifier($_.SID)
      ).Translate([System.Security.Principal.NTAccount]).Value
     Write-Host $cim.LocalPath $ntAccount
   } catch {
     Write-Host $cim.LocalPath
     # $cim | Remove-CimInstance -Confirm:$false
   }
 }
```

# Useful PowerShell Functions

## Delete UserProfile by Path
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
