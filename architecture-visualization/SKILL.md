---
name: architecture-visualization
description: >-
  Visualizes system architecture, call chains, data/sync pipelines, module
  dependencies, and event flows as an interactive Cursor Canvas SVG DAG with
  click-to-open source (useCanvasAction openFile). Use when the user asks to
  draw or visualize architecture, 架构图, 链路图, 调用链, 同步链路, 流程图,
  模块依赖, end-to-end flow, clickable diagram, architecture diagram,
  flow diagram, or ContextWeave-style diagrams.
---

# Architecture Visualization (ContextWeave-style)

Interactive **Cursor Canvas**: SVG DAG + **click node → open source** in the IDE.

Jump via built-in `useCanvasAction` → `{ type: "openFile", path, selection? }`.  
No Feishu, no external MCP, no VSIX.

## When to use / skip

**Use:** architecture maps, sync/event pipelines, call/data flows, “X 怎么端到端跑通” with a clickable diagram.

**Skip:** short text-only answers; user explicitly wants Mermaid / 飞书画板 (use those tools instead).

## Workflow

```
Architecture viz:
- [ ] 1. Scope
- [ ] 2. Trace code + path:line
- [ ] 3. Nodes / edges
- [ ] 4. Interactive canvas
- [ ] 5. Reply with canvas link
```

### 1. Scope

- Default: **narrowest slice** for the latest question (one feature / one pipeline).
- Ask only if scope is ambiguous.
- Direction: pipeline/sync/call chain → `horizontal`; layered UI→DB → `vertical`.

### 2. Trace code

1. Start from named entry, focused file, or recent discussion.
2. Walk one hop at a time (call / publish / consume / write).
3. Real symbols only — never invent services, queues, or DBs.
4. Per code-backed node:
   - `path`: **absolute** if multi-root workspace; else workspace-relative
   - `line`: 1-based line of the key symbol when known
5. **6–20 nodes**. Collapse the rest; note what was collapsed in the caption.

**Multi-root:** resolve files under the correct root (e.g. `WebDev/...` vs `realtime_censor/...`); prefer absolute paths so `openFile` does not miss.

### 3. Graph model

```ts
type ArchNode = {
  id: string;
  label: string;       // short
  detail?: string;     // one line
  path?: string;       // jumpable if set
  line?: number;       // 1-based
  kind?: "entry" | "service" | "event" | "consumer" | "store" | "job" | "other";
};

type ArchEdge = {
  from: string;
  to: string;
  label?: string;      // e.g. publish / upsert by _id
};
```

- `path` set ⇒ node **must** be clickable.
- No `path` ⇒ logical box only (default cursor, no fake jump).

### 4. Canvas (required)

1. Read `~/.cursor/skills-cursor/canvas/SKILL.md` first.
2. Write **one** file:  
   `/Users/<user>/.cursor/projects/<workspace>/canvases/<kebab-name>.canvas.tsx`  
   (do not mkdir; managed dir).
3. Import **only** `cursor/canvas`; embed data inline; no `fetch`.
4. Layout with `computeDAGLayout`; render SVG; wire jumps — follow [canvas-dag.md](canvas-dag.md).
5. UI must include:
   - Title + caption (`Source: … · 点击节点打开源码`)
   - SVG DAG (back-edges dashed)
   - Hover highlight (`fill.tertiary`) + selected stroke (`accent.primary`)
   - Click jumpable node → select + `openFile` (with `selection` when `line` known)
   - Detail `Callout` + **Open in editor** `Button`
   - Jump list: `Pill`/`Button` per jumpable node (Table has no row onClick)

**Theme:** only `useHostTheme()` tokens (`text.*` `bg.*` `fill.*` `stroke.*` `accent.*`).  
No gradients, emojis, box-shadows, rainbow kinds (accent sparingly on `entry`/`store` or selected).

**Empty:** never ship placeholders — ask for missing facts instead.

### 5. Reply

- 1–2 sentences summarizing the flow.
- Say nodes with paths are clickable.
- Markdown link to the `.canvas.tsx` (**full absolute path**).
- First canvas in that workspace: one sentence on what a canvas is.

## Iteration

- Same topic → edit the **same** canvas file.
- New topic → new kebab filename.

## Out of scope

- Feishu / Figma publish  
- Third-party jump plugins / ContextWeave API keys  
- Infra diagrams without code evidence
