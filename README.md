<div align="center">

![WandEnhancer logo](./assets/icon.svg)

# WandEnhancer

Open-source client-side enhancements for Wand.

[![Build executable](https://github.com/Andercore/Wand-WeMod-PRO/actions/workflows/upstream-release.yml/badge.svg)](https://github.com/Andercore/Wand-WeMod-PRO/actions/workflows/upstream-release.yml)
[![Latest release](https://img.shields.io/github/v/release/Andercore/Wand-WeMod-PRO?display_name=tag)](https://github.com/Andercore/Wand-WeMod-PRO/releases/latest)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE.md)

</div>

WandEnhancer is a Windows desktop patcher that applies local, reversible enhancements to an installed Wand client. It also includes an optional web panel for controlling a running trainer from another device on the same trusted network.

> **This is an unofficial third-party project.** Wand and WeMod are trademarks of their respective owners. WandEnhancer does not distribute Wand, does not include proprietary Wand files, and does not provide an account or server-side bypass.

## Download and installation

No compilation is required for normal use. The latest executable is published automatically in this repository:

**[Download the latest WandEnhancer release](https://github.com/Andercore/Wand-WeMod-PRO/releases/latest)**

The release is built on a Windows GitHub Actions runner from the latest commit in the upstream repository [`k1tbyte/Wand-Enhancer`](https://github.com/k1tbyte/Wand-Enhancer). The automation checks upstream every 15 minutes, publishes a new release when the commit changes, and removes older automated releases.

### Install in three steps

1. Open the [latest release](https://github.com/Andercore/Wand-WeMod-PRO/releases/latest).
2. Under **Assets**, download `WandEnhancer.exe`.
3. Run the downloaded file and follow the patcher's instructions.

Keep the executable in a folder where you can write files. The patcher will detect the installed Wand client; select the patches you want and choose **Patch**. Close Wand first if the patcher asks you to.

The executable is unsigned. Windows SmartScreen or antivirus software may show a warning for an unsigned, uncommon application. Only use a binary downloaded from a source you trust, and verify the source code and workflow before running it.

## What it can do

- Apply structural compatibility patches to a local Wand installation.
- Enable or disable optional client-side features.
- Choose between supported static and supervised patch strategies.
- Re-apply a saved patch configuration after Wand updates.
- Inject trusted custom renderer scripts.
- Provide a LAN web panel for trainer status and controls.
- Restore the original installation files from backups.
- Open developer tools with `F12` when that patch is enabled.

Available patch types and behavior can change as Wand evolves. The patcher reports unsupported or failed patch locations instead of silently claiming success.

## See it in action

The following screenshots and video show the patcher interface and the Remote Web Panel:

<div align="center">

![WandEnhancer main window](./assets/screenshots/app1.png)

![WandEnhancer patch controls](./assets/screenshots/app2.png)

</div>

<video src="./assets/usage-guide.mp4" controls width="720"></video>

## Remote web panel

The Remote Web Panel lets you view trainer state and control the active trainer from a phone or another computer.

1. Enable **Remote Web Panel** in the patch dialog.
2. Apply the patch and start Wand.
3. Open the **Connect** control in WandEnhancer.
4. Scan the QR code with a device on the same trusted LAN.

The panel uses HTTP and WebSocket traffic on local port `3223`.

**Security notes**

- There is no pairing code or authentication layer.
- Anyone who can reach the port can view the panel and send supported controls.
- Use it only on a trusted LAN or VPN.
- Do not expose port `3223` directly to the internet.
- Windows Firewall may need an inbound rule for TCP port `3223` on your private network.
- The panel does not expose the Wand bearer token or installation path through its protocol.

For remote access across different networks, use a private VPN such as [Tailscale](https://tailscale.com/) rather than port forwarding.

## Custom renderer scripts

Custom JavaScript files can be injected into Wand when the Remote Web Panel patch is enabled.

- Add existing `.js` files in the patch dialog, or
- Place `.js` files in a `renderer-scripts/` directory next to `WandEnhancer.exe`.

Scripts run inside Wand's renderer with the same privileges as the client. Only use scripts you have reviewed and trust. A script may run more than once during startup, so one-time initialization should be guarded:

```js
if (!globalThis.__myScriptInstalled) {
  globalThis.__myScriptInstalled = true;
  WandEnhancer.log("Custom script loaded");
}
```

The injected environment provides:

```js
WandEnhancer.log(message);
WandEnhancer.remoteUrl;
WandEnhancer.apiVersion;
```

## Data and network behavior

The default patcher works on the selected local installation and does not send telemetry.

Network activity depends on the selected features:

- The optional Remote Web Panel listens on the local network and may use Wand's existing API/CDN paths for trainer and artwork data.
- Builds compiled with update notifications may query the public GitHub Releases API when WandEnhancer starts.
- The updater only shows release information. It does not download or install updates.

The automatic release workflow builds the executable in GitHub Actions. It does not run on the user's computer.

## Build from source

Building from source is only needed for contributors, testing or custom builds. Normal users should use the executable from [Releases](https://github.com/Andercore/Wand-WeMod-PRO/releases/latest).

Builds require Windows and the following tools:

- Visual Studio 2022 or Visual Studio Build Tools with MSBuild.
- .NET Framework 4.8 Developer/Targeting Pack.
- Node.js 22.
- pnpm 10.

From PowerShell:

```powershell
git clone https://github.com/Andercore/Wand-WeMod-PRO.git
cd Wand-WeMod-PRO
.\build.ps1 -Configuration Release
```

The build script installs the web-panel dependencies, runs linting and tests, builds the production panel and bridge, compiles the WPF application, and runs desktop patch and compatibility checks. The executable is written to:

```text
WandEnhancer/bin/Release/WandEnhancer.exe
```

To include the optional compile-time update notification code:

```powershell
.\build.ps1 -Configuration Release -EnableUpdateNotifications
```

The normal build intentionally excludes that code.

## Project layout

| Path | Purpose |
| --- | --- |
| `WandEnhancer/` | Desktop application, patch engine and user interface |
| `AsarSharp/` | ASAR archive reader and writer |
| `web-panel/` | Remote web panel and Electron bridge |
| `WandEnhancer/Patches/` | Embedded JavaScript patch payloads |
| `scripts/` | Build, test and release validation scripts |
| `.github/workflows/` | CI, validation and automated upstream release workflows |

## Contributing

Bug reports and pull requests are welcome. When reporting a problem, include:

- WandEnhancer version and source release.
- Wand version and release channel.
- Enabled patch types and strategy.
- Steps to reproduce the problem.
- Relevant `launcher.log` output with private paths removed.

Do not upload account tokens, storage dumps, proprietary Wand bundles, or personal installation data.

Before submitting a change, run the relevant checks. The complete desktop build is:

```powershell
.\build.ps1 -Configuration Release
```

Do not commit extracted Wand `.source` or `.sources` directories.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the development and review process.

## Disclaimer

WandEnhancer is provided for local interoperability, customization and research purposes. Use it at your own risk. Always keep backups and use the built-in restore option before removing or replacing a patched installation.

## License

This project is licensed under the [Apache License 2.0](LICENSE.md).
