<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset=".github/assets/xenocast-dark.svg">
  <img src=".github/assets/xenocast-light.svg" alt="XenoCast" width="320">
</picture>

# XenoCast

**Build on a Mac, from the computer you already use**

[![Release](https://img.shields.io/github/v/release/Xeno-CI/xenocast?style=flat-square&color=2563eb)](https://github.com/Xeno-CI/xenocast/releases) [![MIT License](https://img.shields.io/badge/license-MIT-475569?style=flat-square)](LICENSE) [![Docs](https://img.shields.io/badge/docs-xenoci.com-2563eb?style=flat-square)](https://xenoci.com/docs) [![Downloads](https://img.shields.io/github/downloads/Xeno-CI/xenocast/total?style=flat-square&color=475569)](https://github.com/Xeno-CI/xenocast/releases)

[한국어](README.md) | [English](README.en.md)

</div>

## Install

macOS and Linux

```sh
curl -fsSL https://xenoci.com/install.sh | sh
```

Windows PowerShell

```powershell
irm https://xenoci.com/install.ps1 | iex
```

One install command takes you through GitHub login, MCP registration and a connection-check build on your rented Mac, with your approval at the login link

You need an active [Mac rental](https://xenoci.com/app/store) for the connection build, and you handle rental and payment yourself

## Give your AI this line

Paste this into Claude Code, Codex, Cursor or opencode

> Read https://raw.githubusercontent.com/Xeno-CI/xenocast/main/SETUP.md and set up XenoCast on this computer, show me the GitHub approval link, and verify the connection build on my rented Mac

The [setup guide](https://raw.githubusercontent.com/Xeno-CI/xenocast/main/SETUP.md) includes the current release installer and recovery steps, so your AI can follow the complete setup now

## Why XenoCast

**Build iOS from Windows, without a local Mac**
Send your project to a rented Mac mini M4 and run your build there
Xcode runs on the remote Mac while you stay on Windows, Linux or macOS

**Your AI can run the build**
MCP exposes five tools: build, status, cancel, logs and macs
Your agent can start a job, inspect its logs and cancel it without leaving the conversation

**Your rented Mac, not a shared runner queue**
Build on the Mac assigned to your rental rather than a shared CI runner
Check its Xcode version and remaining rental time with `xenocast macs`

**CI that uses build-only tokens**
Use a CI token with GitHub Actions, GitLab, Jenkins or your existing pipeline
It can build, read status and logs, cancel jobs and list Macs, but cannot sign, order or extend rentals

**Signing and TestFlight are in preparation**
Remote builds and artifact downloads are available today
Managed iOS signing and TestFlight upload are not yet verified for this setup, so do not rely on them for release delivery

## Your first build

Run your repository's build script from the project directory

```sh
xenocast build --script ./ci.sh
```

The CLI uploads the current directory, streams logs and exits with the remote build's exit code
Use `--no-wait` to print the build ID without waiting, or press Ctrl+C while waiting to cancel the remote build too

## Everyday commands

| Task | Command |
| --- | --- |
| Sign in and check your account | `xenocast login` / `xenocast whoami` |
| Connect AI and check the setup | `xenocast setup --no-tui` / `xenocast doctor` |
| Check Macs and remaining time | `xenocast macs` |
| Start a build | `xenocast build --script ./ci.sh` |
| Read status and logs | `xenocast status <rb_id>` / `xenocast logs <rb_id>` |
| Diagnose or cancel a build | `xenocast diagnose <rb_id>` / `xenocast cancel <rb_id>` |
| Download artifacts | `xenocast artifacts <rb_id> --out ./dist` |
| Revoke your login | `xenocast logout` |

Use `xenocast <command> --help` for command options, `xenocast --help` for all commands and `--json` for machine-readable output

<details>
<summary>Artifacts and build tracking</summary>

```sh
xenocast build --script ./ci.sh --artifacts 'build/*.xcarchive'
xenocast artifacts <rb_id> --out ./dist
```

Artifacts are retained for 7 days
If you omit `--pr` and `--commit`, the CLI reads tracking information from CI variables and the current Git commit
Build and watch return the remote build's exit code, usage errors return 2, and authentication, server or file errors return 1

</details>

<details>
<summary>GitHub Actions, GitLab and other CI systems</summary>

On a signed-in computer, create a token and pipe it directly into a GitHub secret

```sh
xenocast token --ci --name "owner/name CI" --raw | gh secret set XENOCAST_TOKEN --repo owner/name
```

Pass `token: ${{ secrets.XENOCAST_TOKEN }}` to the [GitHub Action](https://github.com/Xeno-CI/build)
For GitLab or another CI system, add a secret variable named `XENOCAST_TOKEN` and build with the CLI

```sh
xenocast build --script ./ci.sh
```

CI tokens expire after 90 days by default, support `--expires 30d`, and can be revoked with `xenocast token --ci --revoke <ses_id>`
Do not expose tokens to fork PRs or run their code with `pull_request_target`
Follow [SETUP.md](SETUP.md) for installation and CI-specific secret registration

</details>

<details>
<summary>MCP and supported AI agents</summary>

`xenocast setup` registers the `xenoci` MCP server in installed Claude Code, Codex, Cursor, opencode and OMO clients
The server command is `xenocast mcp` and its configuration contains no tokens

| Tool | Purpose |
| --- | --- |
| `build` | Start a build |
| `status` | Read build status |
| `cancel` | Cancel a build |
| `logs` | Read build logs |
| `macs` | List rented Macs |

Start a new AI session after registration
See [manual MCP configuration](mcp/CONNECT.md) and [Claude Code setup](docs/claude-code.md) for examples

</details>

<details>
<summary>Login and security</summary>

Use GitHub login instead of API keys
Credentials use macOS Keychain, Windows Credential Manager or Linux secret-service, with a device-bound AES-256-GCM encrypted file as the fallback
The CLI and MCP refresh access tokens automatically, which last about an hour, and `xenocast logout` revokes the server session too
The installer checks the executable against the release's SHA256SUMS
CI tokens are not written to the credential store and token-shaped values in build logs are redacted

</details>

<details>
<summary>Troubleshooting and environment variables</summary>

| Symptom | Next step |
| --- | --- |
| Command not found | Follow the PATH instruction in the installer output |
| Missing or expired login | Run `xenocast login`, then `xenocast doctor` |
| No active Mac rental | [Rent a Mac](https://xenoci.com/app/store), then run `xenocast macs` |
| Failed build | Run `xenocast diagnose <rb_id>` and `xenocast logs <rb_id>` |
| AI tools are missing | Run `xenocast setup --no-tui`, then start a new AI session |

| Variable | Purpose |
| --- | --- |
| `XENOCAST_TOKEN` | Use a CI token instead of the saved login |
| `XENOCI_API_URL` | API address, defaults to `https://xenoci.com` |
| `XENOCAST_CREDENTIAL_STORE` | `keychain`, `wincred`, `secret-service` or `file` |
| `XENOCI_CONFIG_DIR` | Configuration directory |
| `XENOCI_NO_BROWSER=1` | Do not open the login link automatically |

The legacy command name `xenoci` still works
Follow the `next` guidance in [error responses](https://xenoci.com/docs/errors) to choose your next step

</details>

[Docs](https://xenoci.com/docs) | [AI reference](https://xenoci.com/llms.txt) | [OpenAPI](https://xenoci.com/openapi.json) | [MIT license](LICENSE)
