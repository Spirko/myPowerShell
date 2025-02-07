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
