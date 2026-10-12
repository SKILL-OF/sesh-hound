---
name: sesh-hound
description: Use sesh-hound effectively to find, tell apart, and resume local Claude Code / Codex / Copilot sessions from an office folder — session ids, forks, "which agent is that", and what discovery is NOT (liveness, contact, control).
---

# sesh-hound — how to use it effectively

`sesh-hound` lists every coding-agent session whose recorded working
directory is **inside a folder you name**. It is one tool in
[TOOLS-OF/local-agent-discovery](https://github.com/TOOLS-OF/local-agent-discovery)
(`AS/sesh-hound`); that repo holds several tools and this skill is only about
this one. For the generic *discover, don't contact or control* procedure see
[SKILL-OF/local-agent-discovery](https://github.com/SKILL-OF/local-agent-discovery).

## Reach for it first when

- You need a session id, cwd, last-active time, or transcript path.
- Two sessions look like the same agent (a fork, a relaunch, a resumed copy).
- Someone asks "can that agent be resumed?" or "what did that agent say?".
- You are about to `ls`/`grep` `~/.claude/projects` by hand. Don't — that is
  this tool's job, and a hand search silently misses Codex and Copilot.

## Invocation

```
sesh-hound <OFFICE folder> [--json]
```

- **Point it at an office folder, not a product repo.** Agents run *from*
  office folders and edit products as a side effect, so a product folder
  returns nothing even while agents are busy on it.
- It matches sessions whose cwd equals the folder **or is below it**, so a
  parent office folder also surfaces subfolder sessions (e.g. named subagents
  launched from `.meridian/subagents/<name>`).
- Output per session: tool, session id, cwd, file path, last-active time,
  newest first. `--json` for machine use.

## Reading the result well

1. **Same cwd, both recent = probably a fork or a parallel instance.** Tell
   them apart from their own files, not from the title: each JSONL carries its
   own `sessionId` on every record, and the session title lives in the same
   file. Several sessions can share a title ("Meridian"); the id is the key.
2. **A fork is just another session file.** A forked session is seeded with
   its parent's history and then continues under its own id — it appears
   beside the parent, not inside it. If it looks like any other session on
   disk, treat it as resumable.
3. **Resume by id, from that session's cwd:** `claude --resume <session-id>`.
   Never by a shared title. (sesh-hound discovers; it does not resume.)
4. **Last-active is a record, not a heartbeat.** An entry proves a session
   existed and when it last wrote — not that it is alive now. For liveness
   use the live surface: wmux `surface_list`/`pane_list` or the harness's own
   agent list, which is where "is this pane/agent responding" is answered.
5. **Subagent transcripts are separate files** under
   `<session-id>/subagents/`. A *named* agent launched in its own folder shows
   up as its own session with its own cwd.

## What it is not

- Not contact: to message an agent use that agent's native transport
  (`SendMessage`, wmux channel, etc.), with a return address.
- Not control: it never attaches to, resumes, stops, or edits a session.
- Not a content reader: to turn a transcript into readable dialogue use
  `sesh-bee` (below).

## Neighbouring tools (same repo, different concerns)

| Need | Tool | One line |
| --- | --- | --- |
| Find sessions | **sesh-hound** | this skill |
| Read what was said | `sesh-bee` | extract real dialogue turns from Claude Code JSONL into scannable markdown |
| Start one correctly | `sesh-falcon` | launch with every required parameter enforced explicitly, across Claude Code and Codex |
| Understand compaction | `sesh-nautilus` | reconstruct compaction/context-boundary structure that cross-harness converters drop |
| Move one between harnesses | `sesh-stork` | deliver a finished session file into a target harness's storage and verify it boots |

Typical chain: *hound* finds the id → *bee* lets you read it → *falcon* /
`claude --resume` continues it.

## Known limits and how to report them

- A live-background session may not appear in office results. Report that as
  a discovery gap with the smallest reproducing office query and the expected
  id; do not fall back to treating storage paths as the interface.
- The `sesh-hound` on `PATH` is a link to a working checkout, so its code can
  be stale or on a dev branch. When a result drives a decision, confirm that
  checkout is a clean `main`.
