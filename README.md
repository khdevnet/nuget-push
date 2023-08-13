# nuget push dotnet install tool
1. Update proj file with properties
   ```
   	<PropertyGroup>
		<OutputType>Exe</OutputType>
		<TargetFrameworks>netcoreapp3.1;net6.0</TargetFrameworks>
		<ImplicitUsings>enable</ImplicitUsings>
		<LangVersion>10</LangVersion>
		<Nullable>enable</Nullable>
		<PackAsTool>true</PackAsTool>
		<ToolCommandName>botsay</ToolCommandName>
		<PackageOutputPath>./nupkg</PackageOutputPath>
	</PropertyGroup>
   ```
2. Create Readme.md file
3. Update the project file with tool details
```
<PropertyGroup>
		<GeneratePackageOnBuild>true</GeneratePackageOnBuild>
		<!-- Do not include the generator as a lib dependency -->
		<RepositoryType>git</RepositoryType>
		<RepositoryUrl>https://github.com/khdevnet/mc-verifier</RepositoryUrl>
		<PackageProjectUrl>https://github.com/khdevnet/mc-verifier</PackageProjectUrl>
		<Description>Message contract testing in event-driven microservice architecture. Message contract tests. Event, command contracts tests.</Description>
		<PackageReadmeFile>README.md</PackageReadmeFile>
		<Company>khdevnet</Company>
		<Authors>khdevnet</Authors>
	</PropertyGroup>

	<ItemGroup>
	  <Content Include="Readme.md">
	    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
	  </Content>
	</ItemGroup>
	<ItemGroup>
		<None Include="Readme.md">
			<Pack>True</Pack>
			<PackagePath>\</PackagePath>
		</None>
	</ItemGroup>
```
4. Add the secret to Repository Secrets "settings/secrets and variables/New repository secret"
```
NUGET_API_KEY
```
5. Use actions yml
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

6. To trigger nuget package push create new release with tag "v1.0.0", use 3 symbols to specify version

### NOTES
#### error: Source parameter was not specified.
```
dotnet nuget push nuget-packages/dotinstall.botsay.1.0.0.nupkg --api-key "" --source https://api.nuget.org/v3/index.json --skip-duplicate
error: Source parameter was not specified.
```
Parameter "-k" our other has empty value 
