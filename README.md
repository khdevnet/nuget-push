# nuget-push
1. Add secret to Repositiry Sercrets "settings/secrets and variables/New repository secret"
```
NUGET_API_KEY
```
2. Use actions yml
```
name: push-nuget

on:
  push:
    tags:
      - 'v*'

jobs:
  push:
    runs-on: windows-latest
    steps:
    - uses: actions/checkout@v2
    - id: get_version
      uses: battila7/get-version-action@v2
    - run: |
        echo "NuGetVersionV2: ${{ steps.get_version.outputs.version-without-v }}"
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 6.0.x
    - name: Restore dependencies
      run: dotnet restore
    - name: Build
      run: dotnet build --no-restore --configuration Release
    - name: Pack with dotnet botsay
      run: dotnet pack ./dotinstall.botsay.csproj --output nuget-packages --configuration Release -p:PackageVersion=${{ steps.get_version.outputs.version-without-v  }}
    - name: Push with dotnet McVerifier
      run: dotnet nuget push nuget-packages/dotinstall.botsay.${{ steps.get_version.outputs.version-without-v  }}.nupkg --api-key ${{ secrets.NUGET_API_KEY }} --source https://api.nuget.org/v3/index.json
```
