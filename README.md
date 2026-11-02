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

`sitrep setup --yes` accepts every default without asking. Run `sitrep setup` again at any time to change an answer.

## Update

Updates run when you ask for them:

```sh
sitrep update            # install the newest release and restart the host
sitrep update --check    # only say whether there is one
```

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

## MCP tools

| Tool | What it does |
| --- | --- |
| `publish_page` | Publish an HTML file, a folder or a single file by `path`, or send `html` with `files`. Returns the URL. |
| `publish_sitrep` | Put a sitrep from a repository on the host. Returns the URL. |
| `list_published` | Every page and sitrep, with URLs. |
| `unpublish` | Take a page or sitrep off the host. |

The MCP server starts the host when it is not running, so publishing works even without the login entry.

## Commands

| Command | |
| --- | --- |
| `sitrep setup` | Onboarding; run again to change answers |
| `sitrep update` | Install the newest release |
| `sitrep host status` | Version, port, login entry, MCP, last update |
| `sitrep host start` / `stop` / `restart` | Control the background host |
| `sitrep host open` | Open the index in the browser |
| `sitrep host autostart on` / `off` | Start at login, or stop doing so |
| `sitrep uninstall` | Remove the login entry and MCP entries, and stop the host (`--purge` also deletes pages) |

## Where things live

| | macOS | Linux | Windows |
| --- | --- | --- | --- |
| Pages, config, logs | `~/Library/Application Support/sitrep` | `~/.local/share/sitrep` | `%LOCALAPPDATA%\sitrep` |
| Login entry | `~/Library/LaunchAgents/dev.sitrep.host.plist` | `~/.config/systemd/user/sitrep.service` | `HKCU\…\CurrentVersion\Run\sitrep` |

`SITREP_HOME` moves the state folder. The host log is `logs/host.log` in that folder.

## Troubleshooting

- **`npm install --global` fails with EACCES (Linux).** npm cannot write its global folder. Point it at your home folder, then install again: `npm config set prefix ~/.local` (make sure `~/.local/bin` is on your PATH). Alternatively, install Node with a version manager such as nvm or fnm.
- **Port 4790 is taken.** Run `sitrep setup --port 4800`. Pages then use 4801.
- **Linux server without a desktop login.** The systemd user service starts when you log in. To start it at boot, run `loginctl enable-linger $USER`.
- **You changed Node versions.** The login entry and the MCP entries point at the Node binary that ran setup. Run `sitrep setup` again.

## Uninstall

```sh
sitrep uninstall
npm uninstall --global sitrep
```
