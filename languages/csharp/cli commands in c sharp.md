# C# .NET CLI Quick Reference

## Project Creation

```bash
# Console app (minimal, top-level statements)
dotnet new console -n MyApp

# Console app (with Program.Main structure)
dotnet new console -n MyApp --use-program-main

# Class library
dotnet new classlib -n MyLib

# Web API
dotnet new webapi -n MyApi

# Blazor WebAssembly
dotnet new blazorwasm -n MyBlazorApp

# List all available templates
dotnet new list
```

## Build & Run

```bash
# Run (builds automatically)
dotnet run

# Build only
dotnet build

# Build for release
dotnet build -c Release

# Clean build artifacts
dotnet clean

# Publish self-contained executable
dotnet publish -c Release -r win-x64 --self-contained

# Publish single file
dotnet publish -c Release -r win-x64 --self-contained -p:PublishSingleFile=true
```

## NuGet Package Management

```bash
# Add a package
dotnet add package Newtonsoft.Json

# Add specific version
dotnet add package Newtonsoft.Json --version 13.0.1

# Remove a package
dotnet remove package Newtonsoft.Json

# List installed packages
dotnet list package

# List outdated packages
dotnet list package --outdated

# Restore packages (usually automatic)
dotnet restore
```

## Solution Management

```bash
# Create solution
dotnet new sln -n MySolution

# Add project to solution
dotnet sln add MyApp/MyApp.csproj

# Add multiple projects
dotnet sln add **/*.csproj

# List projects in solution
dotnet sln list

# Remove project from solution
dotnet sln remove MyApp/MyApp.csproj
```

## Project References

```bash
# Add reference to another project
dotnet add reference ../MyLib/MyLib.csproj

# Remove reference
dotnet remove reference ../MyLib/MyLib.csproj

# List references
dotnet list reference
```

## Testing

```bash
# Create test project
dotnet new xunit -n MyApp.Tests
dotnet new nunit -n MyApp.Tests
dotnet new mstest -n MyApp.Tests

# Run tests
dotnet test

# Run with verbosity
dotnet test -v normal

# Run specific test
dotnet test --filter "MethodName"

# Run with coverage (requires coverlet)
dotnet test --collect:"XPlat Code Coverage"
```

## Tools

```bash
# Install global tool
dotnet tool install -g dotnet-ef

# Update global tool
dotnet tool update -g dotnet-ef

# Uninstall global tool
dotnet tool uninstall -g dotnet-ef

# List global tools
dotnet tool list -g

# Install local tool (project-specific)
dotnet tool install dotnet-ef

# Restore local tools
dotnet tool restore
```

## Entity Framework Core (common commands)

```bash
# Install EF tools
dotnet tool install -g dotnet-ef

# Add migration
dotnet ef migrations add InitialCreate

# Update database
dotnet ef database update

# Remove last migration
dotnet ef migrations remove

# Generate SQL script
dotnet ef migrations script
```

## Useful Flags

|Flag|Description|
|---|---|
|`-o, --output`|Output directory|
|`-n, --name`|Project/solution name|
|`-f, --framework`|Target framework (e.g., `net8.0`)|
|`-c, --configuration`|Build config (`Debug`/`Release`)|
|`-r, --runtime`|Target runtime (e.g., `win-x64`, `linux-x64`)|
|`-v, --verbosity`|`q[uiet]`, `m[inimal]`, `n[ormal]`, `d[etailed]`|

## Quick Experiments

```bash
# Fastest way to test something
mkdir test && cd test
dotnet new console
# edit Program.cs
dotnet run

# Or use C# REPL
dotnet tool install -g dotnet-repl
dotnet repl
```

## Check Environment

```bash
# Show installed SDKs
dotnet --list-sdks

# Show installed runtimes
dotnet --list-runtimes

# Show dotnet info
dotnet --info
```