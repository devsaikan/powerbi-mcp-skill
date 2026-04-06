# powerbi-mcp-skill

A production-ready governance kit for Power BI semantic model sessions using MCP (Model Context Protocol).

Built on the [Agent Skills open standard](https://agentskills.io), which works across 16+ AI tools including Claude Code, GitHub Copilot, Cursor, OpenAI Codex, Gemini CLI, and VS Code.

---

## The Problem

When you connect an AI tool to your Power BI semantic model via MCP, the AI does not automatically know the difference between you thinking out loud and giving it an instruction.

One analyst said this during a live session:

> "I'm going to delete the tables we no longer needed."

They were thinking out loud. The AI executed immediately. Tables gone. Relationships gone. Power Query queries gone.

They recovered because they had saved ten minutes earlier. Barely.

Typing "do not make changes without my approval" at the top of every session is a human habit. Humans forget. This skill file makes that discipline architectural. It is always there before the first message is sent.

---

## What Is a SKILL.md

Agent Skills is an open standard at agentskills.io, originally developed by Anthropic and now adopted by 16+ tools including GitHub Copilot, Cursor, OpenAI Codex, Gemini CLI, and VS Code.

You place the skill folder in your `.claude/skills/` directory and the agent picks it up automatically when relevant. No typing. No remembering. The governance layer activates before the first message is sent.

---

## What Is in This Kit

```
powerbi-mcp/
SKILL.md                   (governance rules and quick reference)
confirmation-templates.md  (exact approval blocks for every action tier)
dax-standards.md           (DAX quality rules, formats, folder taxonomy)
session-checklist.md       (startup checklist, measures catalog, change log)
```

**SKILL.md** is the main file. It loads automatically on every Power BI MCP session and contains the core governance rules, tier classification, ambient statement intercept, and references to the supporting files.

**confirmation-templates.md** contains the exact templates Claude uses when proposing Tier 2 and Tier 3 actions. Loads when needed, not on every session.

**dax-standards.md** contains DAX quality rules, format strings, naming conventions, and display folder taxonomy. Loads when Claude is authoring measures.

**session-checklist.md** contains the startup safety checklist, measures catalog template, session change log, and pre-publish checklist. Loads at session start and session close.

---

## How It Works

Every action against the semantic model is classified into one of three tiers.

**Tier 1 - Read-Only**
Schema reads, null analysis, DAX drafting to chat, dependency mapping, TMDL export. Automatic. No confirmation needed.

**Tier 2 - Structural Write**
Creating measures, relationships, display folders, RLS roles. Claude proposes using a fixed template. You type YES. Then it acts.

**Tier 3 - Destructive**
Deleting anything. Batch operations on two or more objects. Any irreversible structural change. Requires confirmation plus a verified checkpoint (last save, TMDL export, version control) before a single object is touched.

The ambient statement intercept is the most important rule. Claude is explicitly instructed never to execute on patterns like:

- "I'm going to..." - thinking out loud
- "We should probably..." - reasoning
- "We no longer need..." - observation, not a command

Only unambiguous imperatives count as instructions.

---

## Installation

**Claude Desktop or claude.ai (web)**

1. Download this repo as a ZIP (click Code, then Download ZIP)
2. Open Claude Desktop or go to claude.ai
3. Go to Settings, then Capabilities, then Skills
4. Click the + button and select Upload skill
5. Upload the ZIP file
6. Toggle the skill on

The skill activates automatically on any mention of DAX, TMDL, Power BI, tabular model, Analysis Services, or Power Query.

**Claude Code (CLI or desktop Code tab)**

```bash
cp -r powerbi-mcp ~/.claude/skills/
```

The skill folder must be named powerbi-mcp and contain SKILL.md as the entry point.

---

## Compatibility

This skill follows the [Agent Skills open standard](https://agentskills.io) and works with any compatible tool. Tested with Claude Code. Should work with GitHub Copilot, Cursor, OpenAI Codex, Gemini CLI, VS Code, and any other tool that implements the standard.

---

## Background

This kit was built from three sources:

**The original incident** - a real Power BI MCP session where an analyst lost their entire semantic model to an ambient statement. The governance rules map directly to what went wrong.

**The Claude Code source** - the deobfuscated prompt.ts files reveal how Anthropic structures its own internal tool permissions. The tier classification and output discipline patterns are adapted from that permission pipeline.

**The Agent Skills spec** - every frontmatter field, file structure, and description length follows the official specification at agentskills.io.

---

## Microsoft Official Power BI MCP Server

This skill is designed for use with the official Microsoft Power BI Modeling MCP server.

- Documentation: https://learn.microsoft.com/en-us/power-bi/developer/mcp/mcp-servers-overview
- GitHub: https://github.com/microsoft/powerbi-modeling-mcp

---

## License

MIT. Use it, modify it, share it. If you improve it, consider contributing back.

---

## Author

Saif Khan - Solutions Architect and MCP practitioner.
Website: devsaikan.com
