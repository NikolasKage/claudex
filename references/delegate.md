# claudex › delegate — offload a whole folder to Codex

A heavy mode claudex owns itself (unlike `tango`/`rescue`, which route out).
Hands Codex a task over a **directory**: Codex greps and reads the tree in its
own context and **spawns its own parallel subagents** (`multi_agent_v1.spawn_agent`)
for independent slices. Your chat stays clean.

## When
- Audit / summary / find-everywhere over a folder (N modules, N notes, a corpus).
- The task decomposes into independent slices (Codex reads full files per slice —
  often more accurate than chunk-and-grep).
- You want to offload Claude's context or get a gpt-5.x second opinion.

## Not here
- Fits one read → do it in Claude directly.
- Needs Claude-side skills (`pdf`, `markitdown`, `graphify`) — Codex has its own set.
- Needs a stateful plan loop → `tango`. Needs edits with takeover → `rescue`.

## Run (read-only, in background)
```bash
cat > /tmp/claudex_delegate.txt <<'EOF'
<the task in plain language; tell Codex it may fan out parallel subagents for
 independent parts, and what format the final answer should take>
EOF

# read-only = safe default: reads the whole tree, changes nothing.
# < /dev/null — otherwise codex blocks on stdin in a non-interactive run.
codex exec \
  -C "/path/to/dir" \
  -s read-only \
  --skip-git-repo-check \
  -o /tmp/claudex_delegate_out.md \
  --json \
  "$(cat /tmp/claudex_delegate.txt)" < /dev/null > /tmp/claudex_delegate.log 2>&1
```
Long → always `run_in_background: true`. Watch live: `tail -f /tmp/claudex_delegate.log`.

## After it finishes
```bash
cat /tmp/claudex_delegate_out.md                                    # the answer
grep -ciE "spawn_agent|collab: (Wait|CloseAgent)" /tmp/claudex_delegate.log   # how many subagents
grep -iE "tokens used" /tmp/claudex_delegate.log | tail -1
```
**Spot-check numbers.** Verify any counts/numbers from the answer against reality
(`find`/`grep`) — agents are accurate, but counts are worth checking.

## Flags
| Flag | Meaning |
|---|---|
| `-C, --cd <DIR>` | working root (any folder) |
| `-s, --sandbox` | `read-only` (default) · `workspace-write` (let it write to the folder) · `danger-full-access` (avoid — blocked by auto-mode) |
| `--skip-git-repo-check` | allow running outside a git repo |
| `-o <FILE>` | final answer to a file |
| `--json` | event trace (shows subagent lifecycle) |
| `-m, --model` | agent model |

## Gotchas
- Forgot `--skip-git-repo-check` on a non-git folder → Codex won't start.
- Forgot background → chat hangs for minutes.
- Reaching for `danger-full-access`/bypass → not needed, blocked anyway.
