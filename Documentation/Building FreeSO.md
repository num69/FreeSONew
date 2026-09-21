# Building FreeSO

This document describes how to build the current `archive` branch.

> Note: older FreeSO documentation may still mention .NET Framework 4.5, .NET Core 2.2, MonoGame 3.6 and Protobuild. Those requirements no longer match the current project files in this branch.

## Current Technology

The current projects use:

- **.NET 9**
- **FSO.Windows:** `net9.0-windows`
- **FSO.Server.Core:** `net9.0`
- **MonoGame 3.8.5**
- SDK-style `.csproj` projects
- NuGet package references
- Local .NET tools for the MonoGame content pipeline

The target frameworks in the project files are the authoritative source when this document and older documentation disagree.

## Requirements

### Windows

Install:

- Git
- .NET 9 SDK
- Visual Studio with .NET desktop development support, or Visual Studio Code with the C# tooling
- The Sims Online game files for actually running FreeSO

Check the installed SDK:

```powershell
dotnet --version
```

The result should be a .NET 9 SDK, or a newer SDK capable of building the projects.

## Clone

Clone with submodules:

```powershell
git clone --recursive https://github.com/num69/FreeSONew.git
cd FreeSONew
git checkout archive
git submodule update --init --recursive
```

If the repository was already cloned, only the last two commands are required.

## Restore

The repository contains a local .NET tool manifest under:

```text
TSOClient/.config/dotnet-tools.json
```

It currently provides the MonoGame 3.8.5 content tools.

Restore the solution and tools:

```powershell
cd TSOClient
dotnet tool restore
dotnet restore FreeSO.sln
```

There is no need to run the old `Protobuild.exe --generate` step for the current projects.

## Build

From `TSOClient`:

```powershell
dotnet build FreeSO.sln
```

For an optimized build:

```powershell
dotnet build FreeSO.sln -c Release
```

You can also build individual projects.

### Windows Client

```powershell
dotnet build .\FSO.Windows\FSO.Windows.csproj
```

The Windows client project targets:

```xml
<TargetFramework>net9.0-windows</TargetFramework>
```

and references MonoGame 3.8.5 packages for DesktopGL and WindowsDX.

### Server

```powershell
dotnet build .\FSO.Server.Core\FSO.Server.Core.csproj
```

The server entry project targets:

```xml
<TargetFramework>net9.0</TargetFramework>
```

## Running the Client

Using the .NET CLI:

```powershell
dotnet run --project .\FSO.Windows\FSO.Windows.csproj
```

Or open:

```text
TSOClient/FreeSO.sln
```

and set `FSO.Windows` as the startup project.

The client still requires the original The Sims Online game data. Building the executable successfully does not by itself provide those assets.

## Running the Server

After creating a valid server configuration and preparing the game data/database, run:

```powershell
dotnet run --project .\FSO.Server.Core\FSO.Server.Core.csproj
```

You can also run the compiled DLL:

```powershell
dotnet .\FSO.Server.Core\bin\Debug\net9.0\FSO.Server.Core.dll
```

For production use, build or publish in Release mode.

## Publishing

### Windows Client

Example:

```powershell
dotnet publish .\FSO.Windows\FSO.Windows.csproj -c Release -r win-x64
```

The client project currently has ReadyToRun enabled and AOT disabled.

### Server

Example Windows x64 publish:

```powershell
dotnet publish .\FSO.Server.Core\FSO.Server.Core.csproj -c Release -r win-x64
```

Example Linux x64 publish:

```bash
dotnet publish ./FSO.Server.Core/FSO.Server.Core.csproj -c Release -r linux-x64
```

The server project declares Windows x64 and Linux x64 runtime identifiers.

## MonoGame Content

MonoGame is now referenced through NuGet packages and its content tools are restored using the local .NET tool manifest.

To restore the content tools manually:

```powershell
cd TSOClient
dotnet tool restore
```

The current manifest uses MonoGame 3.8.5 tools such as:

- `mgcb`
- `mgcb-editor`
- `mgcb-editor-windows`

If shaders, fonts or other MonoGame content are changed, use the appropriate MGCB project/tool for that content.

## Visual Studio Code

The current SDK-style .NET 9 projects can be edited and built from Visual Studio Code.

From the repository root:

```powershell
code .
```

Then use the integrated terminal:

```powershell
cd TSOClient
dotnet restore FreeSO.sln
dotnet build FreeSO.sln
```

Visual Studio remains useful for solution-wide debugging, but it is no longer required just to compile the current .NET projects.

## Common Problems

### The required .NET SDK was not found

Run:

```powershell
dotnet --list-sdks
```

Install the .NET 9 SDK if no compatible SDK is present.

### MonoGame tools are missing

Run from `TSOClient`:

```powershell
dotnet tool restore
```

### Game starts but cannot find assets

FreeSO still depends on original The Sims Online content. Verify the configured game path and that the expected game data exists.

### Old instructions ask for .NET Framework 4.5 or .NET Core 2.2

Those instructions refer to an older generation of the project. For this branch, check the current `.csproj` files first.
