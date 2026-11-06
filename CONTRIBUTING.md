# Contributing to WandEnhancer

Thank you for helping improve WandEnhancer. This fork keeps the patch pipeline narrow, reversible and compatible with the upstream `k1tbyte/Wand-Enhancer` project.

## Table of Contents

- [Development Environment Setup](#development-environment-setup)
- [Project Structure](#project-structure)
- [Bug Reports](#bug-reports)
- [Feature Suggestions](#feature-suggestions)
- [Creating a Pull Request](#creating-a-pull-request)
- [Release Process](#release-process)
- [Code Style](#code-style)
- [Testing](#testing)
- [Security and Sensitive Data](#security-and-sensitive-data)
- [License](#license)

## Code of Conduct

By participating in this project, you commit to maintaining respectful interactions with all community members. Any form of insults, harassment, or other unacceptable behavior will not be tolerated.

## Project Structure

The repository contains:

- `WandEnhancer/` - WPF application, patch engine, launcher and desktop services.
- `AsarSharp/` - ASAR archive reader, writer and extraction helpers.
- `web-panel/` - LAN Remote Web Panel, Preact UI and Electron bridge.
- `WandEnhancer/Patches/` - Embedded JavaScript patch payloads.
- `scripts/` - Build, compatibility, locator and release validation scripts.
- `.github/workflows/` - CI, metadata validation and automated upstream releases.

The Remote Web Panel uses HTTP/WebSocket traffic on local port `3223`. It can show trainer/game state, control the active trainer, display installed games and load reviewed custom renderer scripts. It has no pairing or authentication layer, so changes must preserve the trusted-LAN warning in the README.

## Development Environment Setup

1. Clone the repository:
   ```
   git clone https://github.com/Andercore/Wand-WeMod-PRO.git
   cd Wand-WeMod-PRO
   ```

2. Use Windows with Visual Studio 2022 or Visual Studio Build Tools, MSBuild and the .NET Framework 4.8 Developer/Targeting Pack.

3. Install Node.js 22 and pnpm 10.

4. Run the complete build from PowerShell:
   ```
   .\build.ps1 -Configuration Release
   ```

The build installs the web-panel dependencies, runs linting, type checks, production validation and tests, builds the bridge and panel, compiles the WPF application, and runs desktop compatibility checks. The optional update-notification code is compile-time only:

```
.\build.ps1 -Configuration Release -EnableUpdateNotifications
```

Normal builds intentionally exclude that code.

## Bug Reports

If you've found a bug, please create an Issue with a detailed description:

- WandEnhancer version
- Wand version and release channel
- Enabled patches and patch strategy
- Detailed steps to reproduce the bug
- Expected and actual behavior
- Relevant `launcher.log` output with private paths removed
- Screenshots or recordings, if they do not contain personal data or proprietary bundles

## Feature Suggestions

Suggestions for new features or improvements are welcome! Create an Issue describing your idea, explaining:

- What problem the proposed improvement solves
- How you envision implementing this feature
- Potential alternatives you've considered

## Creating a Pull Request

1. Fork the repository.
2. Create a branch with a descriptive name:
   ```
   git checkout -b feature/feature-name
   ```
   or
   ```
   git checkout -b fix/fix-name
   ```

3. Keep changes focused. Preserve patch locator invariants, fallback behavior and reversible backups.

4. Make the necessary changes and commit with clear, descriptive messages.

5. Ensure the relevant checks pass and do not commit extracted `.source` or `.sources` directories, generated build output, account data or proprietary Wand files.

6. Push the branch to your fork:
   ```
   git push origin your-branch-name
   ```

7. Create a Pull Request to `master`.

8. In the Pull Request description, explain the problem, the behavioral change, validation performed and any compatibility limitations.

## Release Process

This fork's normal release path is `.github/workflows/upstream-release.yml`:

- It checks `k1tbyte/Wand-Enhancer` every 15 minutes and can also be started manually.
- It builds the executable from the upstream source and publishes it as the latest `upstream-*` release.
- It keeps only the newest automated release and tag, and removes older workflow runs.
- The fork's source changes must be applied deliberately in the workflow; do not assume a checkout of the upstream repository contains fork-only files.

The upstream repository also contains a tag-driven `release.yml` for its own official release process. It is guarded so it does not publish releases from this fork.

For an upstream-compatible release or a source version change:

1. Update `WandEnhancer/Properties/AssemblyInfo.cs` when the source version itself changes.
2. Add a new top section with the same version to `CHANGELOG.md`.
3. Configure local hooks once:
   ```
   git config core.hooksPath .githooks
   ```
4. Run the relevant validation and commit the version/changelog changes.
5. Create and push a tag only when the release workflow is intended to run, for example:
   ```
   git tag 1.0.8.0
   git push origin 1.0.8.0
   ```

Do not delete or rewrite upstream history to update the fork. Release cleanup must only target automated `upstream-*` releases and tags.

## Code Style

- Use C# naming conventions:
  - PascalCase for class, method, and property names
  - camelCase for local variables and parameters
  - _camelCase for private fields

- Add comments only for non-obvious logic, especially patching, process lifecycle or compatibility decisions.

- Follow SOLID and DRY principles
- Keep production web-panel output free of mock data, debug routes, sourcemaps, local fonts and unnecessary dependencies.
- Route web-panel `localStorage` access through `web-panel/src/shared/storage.ts`.
- Keep shared protocol values in `web-panel/protocol/web-contract.json` and bridge-only constants in `web-panel/bridge/src/constants.ts`.
- Use structural patch anchors and scoped locators; do not hardcode minified identifiers or write regexes spanning entire bundles.

## Testing

The complete build is:

```powershell
.\build.ps1 -Configuration Release
```

It runs web lint, type checks, production validation, Vitest, desktop state/interop checks, structural patch fixtures and auxiliary integrity checks. It does not launch or patch a user's live Wand installation.

Run a focused web test from `web-panel` with `pnpm exec vitest run <test-file>`.

The web-panel build can also be run directly:

```powershell
cd web-panel
pnpm run build
node --check dist/bridge.cjs
node --check dist/renderer-scripts/remote-popup-cleanup.js
```

After building, validate patch locators against an original extracted Wand bundle directory:

```powershell
.\scripts\test-patch-locators.ps1 -AssemblyPath .\WandEnhancer\bin\Release\WandEnhancer.exe -BundleDirectory .\.source\11.6.0-clean
```

The locator harness applies patches only to temporary copies and runs `node --check`. Its fixtures cover prettified and reminified identifiers separately. Do not commit proprietary extracted bundles.

Before releasing an RC, record the exact commit, Wand version/channel and enabled patches, then manually check:

- Fresh patch, upgrade from the previous Enhancer, failed patch followed by retry, and Restore.
- Repeated Wand startup and an overlay opened after a game starts later in the session.
- Wand update followed by automatic re-patching, including a failed re-patch.
- Remote desktop-to-panel and panel-to-desktop values, trainer switching, reconnect, mobile scrolling and exact slider input.
- Remote Play/Stop, installed-game artwork fallback and game/trainer lifecycle status.
- Drawer search with a keyboard and mobile keyboard, Escape and focus restoration.

Use `launcher.log` and `launcher.prev.log` for startup diagnostics. Remove private paths and never share tokens or account storage.

## Security and Sensitive Data

- Do not upload Wand account tokens, storage dumps, bearer credentials, private installation paths or proprietary ASAR/bundle files.
- Do not expose Remote Web Panel port `3223` directly to the internet; test it only on a trusted LAN or private VPN.
- Test ASAR and auxiliary-service mutations against copied files, never a live installation.
- Custom renderer scripts run with Wand renderer privileges and must be reviewed before inclusion.
- Report security issues privately rather than opening a public issue with exploitable details.

## License

By contributing, you agree that your contributions will be licensed under the [Apache License 2.0](LICENSE.md).

---

Thank you for contributing to the WandEnhancer project!
