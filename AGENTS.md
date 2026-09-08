# AGENTS.md

Watt Toolkit (Steam++) — C#/.NET 11 game toolbox. **This checkout is a Linux-headless fork (branch `xxh`)**: a windowless proxy daemon; the Avalonia UI plus Windows/macOS/Android targets were stripped out. Upstream `README*.md`, `doc/*`, and `.github/workflows/*` describe the full multi-platform product and are stale here — trust `README_XXH.md` and the code.

## Build & verify

- Init git submodules first. Build/restore fails cryptically without them, and `ref/DirectoryPackages/Directory.Packages.props` (the central NuGet version file) lives in a submodule that is currently uninitialized:
  `git submodule update --init --recursive`
- Requires .NET SDK 11 (`global.json` pins `11.0.0`; `# VS 2026`).
  `dotnet build src/BD.WTTS.Client.Avalonia.App -c Release`
  builds the app — the "Avalonia.App" project name survives although this branch is headless.
- Whole-repo build: `dotnet build WattToolkit.slnx` (new XML slnx format, not `.sln`).
- Tests are NUnit in `src/BD.WTTS.UnitTest`: `dotnet test src/BD.WTTS.UnitTest`.

## Architecture

- Entry `src/BD.WTTS.Client.Avalonia.App` → `BD.WTTS.Client` (business services) + `BD.WTTS.Client.Avalonia` (headless/shell), wired in `src/BD.WTTS.Client/Startup/Startup.Commands.cs`.
- Features are plugins under `src/BD.WTTS.Client.Plugins.*`. The proxy feature is `...Plugins.Accelerator` plus a subprocess `...Plugins.Accelerator.ReverseProxy` (YARP engine). The subprocess is auto-spawned by the main process and talks to it over `dotnetCampus.Ipc` — never start it manually.
- Headless run: `dotnet run --project src/BD.WTTS.Client.Avalonia.App -- -clt proxy-headless` (auto-enables proxy + all accelerated sites; default port 26561).
- Cert install/remove on Linux need root: `-clt linux -ceri|-cerd <AppDataDirectory>`; data dir defaults to `$XDG_DATA_HOME/Steam++`.

## Conventions & gotchas

- **TFMs live in shared props**, not csproj: each project imports one of `src/TFM_NETX_*.props` at its bottom. This fork edited those to Linux-only `net11.0` with a `LINUX` define — do not re-add Windows/macOS/Android targets.
- **Global usings are opt-in, not implicit**: predefined sets live in `src/ImplicitUsings.*.cs`; each csproj explicitly `<Compile Include="..\ImplicitUsings.X.cs">` the sets it wants (plus `AssemblyInfo*.cs`). Symbols like `Strings`, `SteamKit2`, or `BD.WTTS.Services` appear globally because of these includes — add the include, don't hunt for a missing `using`.
- **Central Package Management is on** — never put `<PackageReference Version=...>` in a csproj. Versions come from `ref/DirectoryPackages/Directory.Packages.props`; version variables are referenced like `$(Version_SteamKit2)`.
- `.editorconfig` suppresses most StyleCop rules, but these are errors: `CS8618`, `CS8604`, `CA1829`, `IDE1006`, `SA1137`, `SA1312`, StyleCop `LayoutRules` category, and `IDE0055`. Explicit accessibility modifiers are NOT required.
- User-facing text uses the Chinese `Strings` resource (`BD.WTTS.Client.Resources.Strings`) via global using; translations live on Crowdin.
- `BD.WTTS.UnitTest` links specific plugin source files directly (`Certificate/CertGenerator.cs`, `HttpReverseProxyMiddleware.FindScriptInjectInsertPosition.cs`, DNS services) — edits to those plugin files change what the tests exercise.
- Namespaces are `BD.WTTS.*` regardless of folder path (`IDE0130` disabled on purpose).