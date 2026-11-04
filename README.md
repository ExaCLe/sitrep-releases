# sitrep

sitrep is a review tool for AI-written code. The agent fills a structured, visual explanation of a change down to a tier the human picks. The human reads, comments and decides. The agent answers every comment with evidence.

It also runs a small host on your machine that keeps every page your agents make reachable: sitreps, and any HTML page with its images, styles and scripts.

## Install

You need [Node.js](https://nodejs.org) 20 or newer. On macOS, Linux and Windows:

```sh
npm install --global https://github.com/ExaCLe/sitrep-releases/releases/latest/download/sitrep.tgz
sitrep setup
```

`sitrep setup` asks three questions and then does the work:

1. **Start the host when you log in?** It adds a LaunchAgent on macOS, a systemd user service on Linux, or a Run entry on Windows. None of them needs administrator rights.
2. **Add sitrep to Claude Code?** Asked when `claude` is installed. Yes adds the MCP server and the sitrep skill, which teaches the agent to write a sitrep.
3. **Add sitrep to Codex?** Asked when `codex` is installed. Same as for Claude Code.

When [T3 Code](#t3-code-optional) is installed, setup also offers to connect it. That one is never taken by `--yes`; use `--t3` or `sitrep connect t3`.

`sitrep setup --yes` accepts every default without asking. Run `sitrep setup` again at any time to change an answer.

## Update

Updates run when you ask for them:

```sh
sitrep update            # install the newest release and restart the host
sitrep update --check    # only say whether there is one
```

The updater installs into the folder of the command you ran, even when npm's configured global folder differs. It checks the archive, installed CLI and restarted host before reporting success. A verification failure leaves the update marked failed in `sitrep host status`.

Older updaters may report success while another installation remains active. On Windows, this PowerShell command installs the latest release into the folder of the active npm shim and refreshes that installation:

```powershell
$sitrepPrefix = Split-Path (Get-Command sitrep.cmd -ErrorAction Stop).Source
npm install --global --prefix $sitrepPrefix https://github.com/ExaCLe/sitrep-releases/releases/latest/download/sitrep.tgz
if ($LASTEXITCODE -eq 0) { & "$sitrepPrefix\sitrep.cmd" setup --refresh; & "$sitrepPrefix\sitrep.cmd" --version }
```

This preserves the existing host configuration and review data. The corrected updater takes effect after installing a release containing it.

## Publish pages

Agents publish through the MCP server's `publish_page` tool and give you the URL. From a terminal:

```sh
sitrep publish ./report.html     # the file plus the local images, CSS and scripts it references
sitrep publish ./site            # a whole folder; opens at index.html (or --entry page.html)
sitrep publish ./chart.pdf       # any single file
sitrep publish ./report.html --id weekly   # publishing the same id again replaces the page
sitrep published                 # everything hosted, with URLs
sitrep unpublish weekly
```

A sitrep goes on the host with `sitrep publish --sitrep <id>`, run in its repository. The page is live, so comments and approval work there.

Everything hosted is listed at <http://127.0.0.1:4790/>. Pages are served from <http://127.0.0.1:4791/>, a separate origin, so a script in a page cannot reach the sitrep API. Both addresses are reachable from this machine only.

## T3 Code (optional)

When your agents run in [T3 Code](https://github.com/pingdotgg/t3code) on this machine, a sitrep can remember the chat that wrote it. The page then shows **Open chat** beside Send back, and **Send to agent** saves your note and delivers it to that same chat. The agent gets it as its next turn when it is busy, collects the feedback and updates the sitrep in place. Without T3 nothing changes: Send back saves the note, and you tell the agent "I gave feedback".

```sh
sitrep connect t3                    # finds the T3 CLI (T3CODE_CLI_PATH, ~/.t3/bin/t3, or t3 on PATH) and checks it runs
sitrep connect t3 --web-url https://<machine>.<tailnet>.ts.net/t3   # where Open chat goes, e.g. from your phone
sitrep connect                       # the state
sitrep disconnect t3
```

`--cli <path>` and `--base-dir <dir>` (T3's data folder, default `T3CODE_HOME`) point at a T3 that is not found by itself. The connection is stored in `config.json`; each sitrep's chat is stored with the sitrep.

Once connected, `sitrep open` run inside a T3 chat binds that chat (`T3CODE_THREAD_ID`), unless it is a sub-agent's. `sitrep link-chat <id> --thread <T3 thread id>` binds another one, `open --chat <thread id>` does it while opening, and `link-chat <id> --unlink` removes it. A chat is bound only when T3 says it works in the sitrep's repository: the same worktree, or the project T3 resolves for it.

Delivery runs in the host, through the T3 CLI, never in the browser. Before sending, the host checks again that the chat still exists in the same project and worktree. Reports in the primary checkout can bind a chat in a linked worktree of that repository. The note is saved before anything is sent, so it survives any failure. If the CLI times out or exits with an error, delivery is uncertain because the message may already have arrived. The page offers Open chat instead of sending again. Try again is available only after proven non-delivery, such as a program that never started. Delivery status survives reloads; pending attempts refresh their status without sending again.

## MCP tools

| Tool | What it does |
| --- | --- |
| `publish_page` | Publish an HTML file, a folder or a single file by `path`, or send `html` with `files`. Returns the URL. |
| `publish_sitrep` | Put a sitrep from a repository on the host. Returns the URL. |
| `collect_feedback` | Read comments and receive the saved marked PNGs directly as image content, with their anchors and original versions. |
| `list_published` | Every page and sitrep, with URLs. |
| `unpublish` | Take a page or sitrep off the host. |

The MCP server starts the host when it is not running, so publishing works even without the login entry.

`collect_feedback` takes `repo` and `id`. It reads open comments by default; `open: false` includes answered comments.
Each marked image follows a label with its comment id, mark ids, authored version and view.
The tool returns the displayed context, a close-up around the marks, and a natural-resolution snapshot when saved.
It flags missing pictures and outdated content. Existing drawings retain whichever snapshots were originally saved.
Identical images within one comment share one attachment. Responses include at most eight images and 8 MiB of base64 image data.
If the response says `MORE IMAGES`, call again with its `imageOffset` and the same filters. `commentId` narrows image attachments to one comment.
Collection reads the sitrep's data folder directly and works without a running host. Update sitrep and reconnect the agent's MCP server to expose a newly added tool.

## Commands

| Command | |
| --- | --- |
| `sitrep setup` | Onboarding; run again to change answers |
| `sitrep update` | Install the newest release |
| `sitrep host status` | Version, port, login entry, MCP, last update |
| `sitrep host start` / `stop` / `restart` | Control the background host |
| `sitrep host open` | Open the index in the browser |
| `sitrep host autostart on` / `off` | Start at login, or stop doing so |
| `sitrep connect t3` / `sitrep disconnect t3` | Let Send back deliver to a sitrep's T3 Code chat, or stop |
| `sitrep link-chat <id>` | Bind a sitrep to a T3 Code chat (`--thread <id>`, `--unlink`) |
| `sitrep storage` | Where sitreps are kept and how much space they take |
| `sitrep uninstall` | Remove the login entry and MCP entries, and stop the host (`--purge` also deletes pages) |

## Where things live

| | macOS | Linux | Windows |
| --- | --- | --- | --- |
| Sitreps, pages, config, logs | `~/Library/Application Support/sitrep` | `~/.local/share/sitrep` | `%LOCALAPPDATA%\sitrep` |
| Login entry | `~/Library/LaunchAgents/dev.sitrep.host.plist` | `~/.config/systemd/user/sitrep.service` | `HKCU\…\CurrentVersion\Run\sitrep` |

`SITREP_HOME` (or `--home`) moves the state folder. The host log is `logs/host.log` in that folder.

Sitreps live in `data/` of that folder, one subfolder per checkout, never inside a repository, so no repository needs an ignore rule. They stay when the checkout or worktree is removed, and the host keeps serving them. Every version, comment and approval is kept; documents are gzipped, and each distinct screenshot or mark picture is stored once. `sitrep storage` shows usage by documents, images, renders and published pages. There is no shared disk budget or budget configuration; history is never deleted automatically.

Each new version retains an immutable compressed diff and its image references. A single atomic document replacement selects those files, so interrupted writes cannot pair an old document with a new diff. The version API at `api/versions/<number>` returns historical data and immutable image addresses. The reader shows the latest version.

Sitrep versions before this one kept sitreps in `.sitrep/` inside the checkout. The first command in such a checkout copies them into `data/` under the legacy writer's lock and leaves the content intact; delete the old folder yourself after checking that import succeeded and no older sitrep runs there. An older sitrep keeps writing to `.sitrep/` and never sees sitreps made since. When the two copies diverge, commands keep reading the imported one and say so; `sitrep storage --import-legacy <id> --as <new id>` keeps the other beside it. Screenshots of versions before the latest were never kept by older sitreps and stay missing. Historical code without a stored snapshot is reconstructed from Git when possible, otherwise the API explicitly reports it unavailable.

## Troubleshooting

- **`npm install --global` fails with EACCES (Linux).** npm cannot write its global folder. Point it at your home folder, then install again: `npm config set prefix ~/.local` (make sure `~/.local/bin` is on your PATH). Alternatively, install Node with a version manager such as nvm or fnm.
- **Port 4790 is taken.** Run `sitrep setup --port 4800`. Pages then use 4801.
- **Reviewing from another device on your own network.** The host answers only `localhost`. To put it behind a proxy you control, such as `tailscale serve --bg --set-path /sitrep http://127.0.0.1:4790`, add `"publicHosts": ["<your machine's name>"]` and `"basePath": "/sitrep"` to `config.json` in the state folder (or set `SITREP_PUBLIC_HOSTS` and `SITREP_BASE_PATH`), then `sitrep host restart`. The index and sitreps then answer that name too, and links keep the prefix. Published pages stay on localhost.
- **Linux server without a desktop login.** The systemd user service starts when you log in. To start it at boot, run `loginctl enable-linger $USER`.
- **You changed Node versions.** The login entry and the MCP entries point at the Node binary that ran setup. Run `sitrep setup` again.

## Uninstall

```sh
sitrep uninstall
npm uninstall --global sitrep
```
