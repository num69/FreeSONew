# Initial Setup

This document describes initial setup for the current `archive` branch.

> Older setup guides may mention .NET Framework 4.5 and .NET Core 2.2. The current client and server projects in this branch target .NET 9.

## Requirements

You need:

- .NET 9 SDK
- A compiled FreeSO client/server, or the source repository
- The Sims Online game files
- A database supported by the server configuration
- An NFS/data directory for lot and object saves
- A configured `config.json`

For development, Visual Studio or Visual Studio Code can be used.

## Build First

From the repository:

```powershell
cd TSOClient
dotnet tool restore
dotnet restore FreeSO.sln
dotnet build FreeSO.sln
```

The primary projects are:

- `FSO.Windows` — Windows client, targeting `net9.0-windows`
- `FSO.Server.Core` — server executable, targeting `net9.0`

## The Sims Online Game Data

FreeSO still relies on original The Sims Online game data for objects, avatars, UI resources, tuning and other content.

Configure `gameLocation` so that it points to the directory containing the expected TSO client data.

For example, if this file exists:

```text
./game/tuning.dat
```

then a suitable value is:

```json
"gameLocation": "./game/"
```

## Server Data Directory

Configure `simNFS` to a directory where the server can store persistent lot/object data, thumbnails and related server files.

Example:

```json
"simNFS": "./nfs"
```

Create the directory if it does not exist.

## config.json

Prepare a `config.json` based on the sample configuration included with the server.

At minimum, review the following values:

1. `gameLocation`
2. `simNFS`
3. `secret`
4. database connection settings
5. service bindings
6. public host addresses
7. shard/city configuration

### Secret

Use a unique secret for communication between server components.

Do not deploy the sample/default secret on a public server.

### Database

Set the database connection string to match your server.

Example structure:

```json
"database": {
  "connectionString": "server=127.0.0.1;uid=fsoserver;pwd=password;database=fso;"
}
```

Refer to `Database Setup.md` for database creation and schema details.

### Public Hosts

For services that clients must reach, configure the appropriate `public_host` values for the machine or public IP/domain hosting the server.

For a local development server, loopback/local addresses may be sufficient.

## Run the Server

From `TSOClient`:

```powershell
dotnet run --project .\FSO.Server.Core\FSO.Server.Core.csproj
```

Or run a compiled build directly:

```powershell
dotnet .\FSO.Server.Core\bin\Debug\net9.0\FSO.Server.Core.dll
```

For a Release build:

```powershell
dotnet build .\FSO.Server.Core\FSO.Server.Core.csproj -c Release
dotnet .\FSO.Server.Core\bin\Release\net9.0\FSO.Server.Core.dll
```

## Run the Client

From `TSOClient`:

```powershell
dotnet run --project .\FSO.Windows\FSO.Windows.csproj
```

Or open `FreeSO.sln` and run `FSO.Windows`.

## Connecting the Client to a Custom Server

On the FreeSO login screen, the debug/server URL interface can be used to point the client at your custom API endpoint.

For a client distributed specifically for one server, configure the default endpoint as part of your client build/configuration rather than requiring users to change it manually.

## Client and Server Version Mismatch

If the client and server use incompatible version/update information, the client may request an update.

For development, keep client and server built from compatible source revisions. For production, configure the normal FreeSO update infrastructure rather than relying on bypass behavior.

See `Updates.md` for details.

## Development Notes

The current branch is built with modern SDK-style projects. Do not follow legacy steps that require:

- .NET Framework 4.5 targeting packs
- .NET Core 2.2 SDK/runtime
- MonoGame 3.6
- `Protobuild.exe --generate`

For the current branch, use the target frameworks and package versions declared in the project files as the source of truth.
