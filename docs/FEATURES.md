# pier.nvim — feature outline

A Neovim plugin that bridges nvim and pi running independently in tmux.
pi's TUI stays the harness (sessions, steering, queueing, model switching);
the plugin only does what a terminal can't: move editor state in, and pi's
results back out.

## Pairing

nvim and pi are paired automatically — no configuration:

- nvim must be inside tmux (`$TMUX` set).
- Target pi pane = same tmux session **and** same project root
  (`pane_current_path` vs. nvim's `vim.fs.root`).
- Targets are tmux pane IDs (`%5`), which survive renumbering.

Edge cases:

- No pi pane found → offer `tmux new-window -c <root> -n pi pi`.
- Multiple matches → use the first, remember it; `:PierAttach` to switch.
- Pane died → re-run discovery on next send.

## v1

### 1. Send editor context to pi

- `:PierSend` (normal + visual) — prompt built from the current file /
  selection as `@path:from-to` mentions + user text, pasted into the pi TUI.
- `:PierDiag` — send current buffer's LSP diagnostics as context ("fix these").
- Multi-line prompts go via `tmux load-buffer -` + `tmux paste-buffer -p`
  (bracketed paste), then Enter — never line-by-line `send-keys`.

### 2. pi's edits → nvim buffers

- Watch the project dir (`vim.uv` fs watchers); auto-`:checktime`/reload
  on external changes.
- Never clobber dirty buffers: offer diff or keep.

### 3. Review pi's changes

- `:PierReview` — list files pi touched as diffs against the VCS base
  (jj/git); accept/discard per file.

### 4. Jump to locations

- Paths/line refs from pi's responses → quickfix list.

## Later

- Session fork / new session shortcuts.
- Model / thinking-level cycling.
- Prompt history.
- Prompt queueing while a run is active.

## Explicitly out of scope

- RPC/JSON modes, chat rendering, streaming UI — pi's TUI already does this.
- A pi extension counterpart — no natural channel to the nvim session;
  file watching + stdin are enough.
- Multi-agent orchestration, virtual text decorations.
