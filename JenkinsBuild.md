# Jenkins setup for build
First go to the Dashboard/Manage Jenkins and set up the GitHub account with a key

## Build steps
These are the build steps that has been used to build the SVGEditor for HEI4Future

### GitHub checkout
Happens automatically

### Restore Project nuget packages
Type: Windows Batch
Command: 
```
dotnet restore
```

### Build Project
Type: Windows Batch
Command: 
```
dotnet build --no-restore
```

### Test Project (Unit tests)
Type: Windows Batch
Command: 
```
dotnet test --no-build --verbosity normal
```

### Prepare the build, new folders
Type: Windows Batch
Command: 
```
#prepare to deploy, blue/green
#recycle app pool
C:\Windows\System32\inetsrv\appcmd.exe recycle apppool "HEI4FutureSVGEditor"
#stop the site
C:\Windows\System32\inetsrv\appcmd.exe stop site "HEI4FutureSVGEditor"
#create the new green folder, for publish
mkdir c:\websites\SVGEditor_%BUILD_NUMBER%
```

### Publish Project
Type: Windows Batch
Command: 
```
dotnet publish HEI4Future_SVGEditor.csproj -c Release -r win-x64 --self-contained true -o c:\websites\SVGEditor_%BUILD_NUMBER%
```

### Change website directory
Type: Windows Batch
Command: 
```
#Change the directory for the web application
"C:\Windows\System32\inetsrv\appcmd.exe" set site /site.name:"HEI4FutureSVGEditor" /application[path='/'].virtualDirectory[path='/'].physicalPath:"c:\websites\SVGEditor_%BUILD_NUMBER%"
```

### Start website
Type: Windows Batch
Command: 
```
#start the website
C:\Windows\System32\inetsrv\appcmd.exe start site "HEI4FutureSVGEditor"
```

## Delete old folders
Type: Powershell
Command: 
```
# Define the root folder where you want to search and delete folders
$rootFolder = "C:\websites"

# Define the prefix to search for (e.g., "SVGEditor_")
$prefix = "SVGEditor_"

# Get the BUILD_NUMBER from the Jenkins environment (replace with actual variable name)
$buildNumber = $ENV:BUILD_NUMBER

# Define the folder name to exclude based on BUILD_NUMBER
$excludeFolder = "${prefix}${buildNumber}"

# Get a list of folders with the specified prefix
$foldersToDelete = Get-ChildItem -Path $rootFolder -Directory | Where-Object { $_.Name -like "${prefix}*" -and $_.Name -ne $excludeFolder }

# Check if any matching folders were found
if ($foldersToDelete.Count -gt 0) {
    # Iterate through the matching folders and delete them
    foreach ($folder in $foldersToDelete) {
        Write-Host "Deleting folder: $($folder.FullName)"
        Remove-Item -Path $folder.FullName -Recurse -Force
    }
    Write-Host "Deleted $($foldersToDelete.Count) folders with the prefix '$prefix', excluding '$excludeFolder'."
} else {
    Write-Host "No folders with the prefix '$prefix' were found in $rootFolder, excluding '$excludeFolder'."
}
```

