# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
go build                # Build the binary
go run .                # Run without building
go vet ./...            # Vet for common issues
go install github.com/felixschmelzer/ding@latest  # Install binary
```

No test files exist in this project.

## Architecture

`ding` is a single-package Go application (4 files) that wraps any shell command, runs it in a PTY, and sends a Telegram notification when it finishes.

**Entry point & flow (`main.go`):**
- Parses CLI args; `--config` flag launches the TUI setup instead of running a command
- Starts an optional goroutine for periodic "still running" Telegram pings (configurable interval)
- Calls `runCommand()` from `runner.go`, then sends the final Telegram notification
- Optionally prints a terminal summary if `ShowSummary` is set in config

**Key files:**
- `runner.go` — Runs the command in a PTY (`github.com/creack/pty`), mirrors I/O and terminal resize signals; falls back to normal exec if PTY unavailable
- `config.go` — Bubble Tea TUI for interactive setup; loads/saves JSON config at `~/.config/ding/config.json`
- `telegram.go` — Sends HTTP POST to Telegram Bot API; formats success/failure/running messages using HTML parse mode

**Config struct** (`config.go`):
```go
type Config struct {
    BotToken       string `json:"bot_token"`
    ChatID         string `json:"chat_id"`
    NotifyInterval int    `json:"notify_interval"` // minutes; 0 = disabled
    ShowSummary    bool   `json:"show_summary"`
}
```
Config is stored at `~/.config/ding/config.json`.

## Notes

- Telegram messages use HTML parse mode (`<code>`, `<b>` tags)
- The README mentions `.toml` config extension but the actual implementation uses `.json`

## Claude Behavior

- Keep this CLAUDE.md up to date after making changes to the codebase (new commands, architectural changes, config changes, etc.)
