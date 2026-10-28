# cctop

A single-file Python script: an fzf dashboard of running Claude Code sessions for a pinned Warp tab. See the
README for what it does.

## Constraints

- Keep it one file, Python standard library only. Python 3, macOS only.
- Never switch Warp tabs by cycling through them (sending next-tab keystrokes): it's jarring and was tried and
  rejected. Jumping uses the `WARP_FOCUS_URL` (`warp://session/<uuid>`) each tab exports; if that ever breaks,
  look for another direct route.
- Demo mode (`CCTOP_DEMO=1`) must never read real sessions or write to Claude Code or Warp files.
- Don't make fzf the slow path: the background ticker collects data and fzf only reloads a prebuilt file
  (`reload-sync(cat …)`), so keystrokes never wait on a refresh.

## Testing

- `CCTOP_UI= ./cctop list` and `CCTOP_UI= ./cctop preview <key>` print what the view renders, without fzf.
- For the interactive view, run it in a pseudo-terminal (Python's `pty`) and query fzf's state over its
  `--listen` socket (`GET /`), rather than eyeballing escape codes.
- Never exercise `^x` (close), reopen, reopen-all or `^n` against real sessions: they stop processes and open
  tabs. Test their pieces directly (e.g. with a stand-in process or scratch copies of the cache files).
- When importing the script as a module in tests, set `sys.dont_write_bytecode = True` so no `__pycache__`
  lands next to it.

## Fragile dependencies

Only `claude agents --json` is a documented interface. Everything else can change with a Claude Code or Warp
update:

- `~/.claude/sessions/<pid>.json`: live `status`, `statusUpdatedAt`, `nameSource`, `sessionId`.
- Transcript records in `~/.claude/projects/*/<sessionId>.jsonl`: `ai-title`, `custom-title`, `pr-link`,
  `last-prompt`, `entrypoint` and `turnPosition` on user records. `parse_tail` reads these.
- Warp: `WARP_FOCUS_URL` in each tab's environment, tab configs in `~/.warp/tab_configs/` opened with
  `warp://tab_config/<name>`, and Warp's state database (`warp_tab_order`) for tab order.

`cctop`'s own state lives in `~/.cache/cctop/`.
