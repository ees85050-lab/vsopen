# vsopen

Open files in your code-server window from any shell on the server, such as an ssh session, a cron job, or an AI agent's shell tool.

```sh
vsopen notes/report.md src/main.py
```

## Why

AI coding agents write file paths in their replies as markdown links. In a code-server (VS Code in the browser) setup, those links can be dead: in ours, the chat panel couldn't open workspace-relative, percent-encoded or absolute links. So the agent runs `vsopen <file>` instead, and the file opens in the window you're looking at.

code-server ships a remote CLI that opens files in a window, given that window's socket in `VSCODE_IPC_HOOK_CLI`. The window's own terminals are set up for it. Other shells either lack the variable (ssh, cron, systemd) or don't have the remote CLI on their PATH, so the `code-server` they find is the server itself. vsopen finds both the CLI and a live socket.

## Install

```sh
curl -fsSL -o ~/.local/bin/vsopen https://raw.githubusercontent.com/ees85050-lab/vsopen/main/vsopen
chmod +x ~/.local/bin/vsopen
```

Then tell your agent, for example in CLAUDE.md: "To show me a file, run `vsopen <path>` instead of writing a link."

## How it works

1. It lists the `vscode-ipc-*.sock` sockets in `$XDG_RUNTIME_DIR` (or `/run/user/<uid>`) and in `$TMPDIR` (or `/tmp`), newest first.
2. It looks for code-server's remote CLI in this order: `$VSOPEN_CLI`; the newest `~/.local/lib/code-server-*/lib/vscode/bin/remote-cli/code-server` (where code-server's install script puts it); `/usr/lib/code-server/lib/vscode/bin/remote-cli/code-server` (the .deb and .rpm packages); and next to the `code-server` on your PATH.
3. It runs `code-server -r <files>` against the 8 newest sockets in turn, with a 10-second timeout each, and stops at the first one that accepts.

A closed window leaves its socket behind, and after a few weeks there can be hundreds of them. A stale socket refuses the connection at once, so vsopen moves on to the next one. If you have several windows open, the most recently opened one gets the files.

Exit status:

- 0: a window accepted the files. vsopen can't tell whether it's the window in front of you.
- 1: no sockets, no remote CLI, or none of the sockets accepted. The message says which.
- 2: no files given.

## Requirements

Linux, bash 4 or newer, coreutils, and code-server 4.x. Tested with code-server 4.133.

## License

MIT

## 中文简介

vsopen：在服务器上的任何 shell 里（ssh、定时任务、AI 代理的 shell 工具），把文件推到你正在用的 code-server 窗口里打开。AI 代理在聊天面板里写的文件链接有时点不开，让它改跑 `vsopen <文件>` 就行。窗口关掉会留下失效的 socket，vsopen 从新到旧逐个试，试通一个就停。
