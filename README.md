# Fable Advisor

**Keep your session cheap. Consult the smartest model only when it counts.**

This is an advisor-only Claude Code plugin: your session runs on a cheaper model (Sonnet) doing all the reading, writing, and executing, and at commitment boundaries — architecture decisions, migrations, API design, a debugging effort that's failed twice — it consults a read-only `fable-advisor` subagent pinned to **Fable 5**, Anthropic's most capable model. The advisor reads your actual code and returns a verdict in under 300 words. It never implements.

## Install

```
claude plugin marketplace add justanotherkevin/fable-advisor
claude plugin install fable-advisor@fable-advisor
```

Updating an existing installation:

```
claude plugin marketplace update fable-advisor
claude plugin update fable-advisor@fable-advisor
```

**Even simpler — one file, no plugin install.** Copy [`agents/fable-advisor.md`](agents/fable-advisor.md) into `~/.claude/agents/` directly.

## Use it

Keep your session on Sonnet:

```
/model sonnet
```

Then just ask for work. Bring the `fable-advisor` agent in yourself at a commitment boundary, or make it automatic by adding this to your project's `CLAUDE.md`:

```
Before committing to any architecture decision, migration, or refactor
touching 3+ files, consult the fable-advisor agent and act on its verdict.
```

## Requirements

- Claude Code with a subscription that includes Fable 5 access.
- **No Fable access?** Edit `agents/fable-advisor.md` and change `model: fable` to `model: opus` — same pattern, one tier down.

## License

MIT
