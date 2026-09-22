---
name: architecture-visualization
description: >-
  Visualizes system architecture, call chains, data/sync flows, module
  dependencies, and event pipelines as an interactive Cursor Canvas SVG DAG
  with click-to-open source (useCanvasAction openFile). Use when the user
  asks to draw architecture, visualize a flow/pipeline, map module
  dependencies, explain how a feature works end-to-end, wants clickable
  nodes that jump to code, or mentions 架构图 / 链路图 / 调用链 /
  architecture diagram / flow diagram / ContextWeave.
---

# Architecture Visualization (ContextWeave-style)

Produce an **interactive Cursor Canvas**: SVG DAG + **click node → open source in IDE**.

Uses built-in `useCanvasAction` → `{ type: "openFile", path, selection? }`. No Feishu, no external MCP, no VSIX jump plugin.

## When to use

- Architecture / module dependency maps
- Business → event → consumer → storage sync chains
- Call / data / state flows across services or packages
- "How does X work end-to-end?" with a visual, clickable answer

Skip when the user only wants a short textual explanation, or explicitly asks for Mermaid / 飞书画板 (then follow those tools instead).

## Workflow

```
Architecture viz:
- [ ] 1. Scope the graph
- [ ] 2. Gather facts + line anchors from code
- [ ] 3. Build nodes + edges (path/line required when known)
- [ ] 4. Write interactive Canvas (DAG + openFile)
- [ ] 5. Link the canvas in the reply
```

### 1. Scope the graph

Ask only if the scope is ambiguous. Default to the **narrowest useful slice** for the latest question, not the whole monorepo.

Direction:
- Pipeline / sync / call chain → `direction: "horizontal"`
- Layered architecture → `direction: "vertical"`

### 2. Gather facts from code

1. Grep / read entry points the user named (or open file / recent discussion).
2. Trace one hop at a time: caller → callee, publish → consume, write → store.
3. Prefer **real symbols** (class/method/event name) over invented boxes.
4. For every code-backed node, record:
   - **path**: prefer **absolute** path (multi-root workspaces); else workspace-relative
   - **line**: 1-based line of the key symbol (function/class/const), when known
5. Cap **6–20 nodes**. Collapse extras; note what was collapsed.

Do **not** invent services, queues, or DBs that are not evidenced in code or by the user.

### 3. Build nodes + edges

```ts
type ArchNode = {
  id: string;
  label: string;
  detail?: string;
  path?: string;       // absolute preferred when jumpable
  line?: number;       // 1-based; used in openFile selection
  kind?: "entry" | "service" | "event" | "consumer" | "store" | "job" | "other";
};

type ArchEdge = {
  from: string;
  to: string;
  label?: string;
};
```

Every node with a real `path` **must** be clickable. Nodes without `path` (pure logical boxes) are not jumpable — show normal cursor, no fake links.

### 4. Write the Canvas

**Mandatory:**
1. Read `~/.cursor/skills-cursor/canvas/SKILL.md` and follow it.
2. Write one file under  
   `/Users/<user>/.cursor/projects/<workspace>/canvases/<name>.canvas.tsx`
3. Import only from `cursor/canvas`. Embed graph data inline.
4. Layout with `computeDAGLayout`; render SVG; wire jumps via `useCanvasAction` (see [canvas-dag.md](canvas-dag.md)).
5. Title + one-line caption (scope + "click a node to open source").
6. Detail strip below/beside graph: selected node label, path, line; `Button` "Open in editor" when `path` exists.
7. Optional `Table` of nodes; path cells / rows also dispatch `openFile`.

**Interaction (required — ContextWeave parity):**
- Click jumpable SVG node → `dispatch({ type: "openFile", path, selection? })` and set selected id (`useCanvasState`).
- Optional `selection`: `{ startLineNumber, startColumn: 1, endLineNumber, endColumn: 1 }` when `line` is known (VS Code-style).
- Hover: pointer cursor + light `fill.tertiary` (tokens only).
- Selected: `stroke` / `accent.primary` highlight.
- Back-edges: dashed stroke.
- Keyboard/secondary: same `openFile` from detail `Button` and table.

**Design:**
- Colors from `useHostTheme()` only (`text.*`, `bg.*`, `fill.*`, `stroke.*`, `accent.*`).
- No gradients, emojis, box-shadows, rainbow colors.
- Never empty canvas; ask if facts are missing.

### 5. Reply

- One or two sentences summarizing the flow.
- Note that nodes with source paths are clickable.
- Markdown link to the `.canvas.tsx` (full absolute path).
- First canvas in workspace: one sentence on what a canvas is.

## Iteration

- Same topic refine → edit the same canvas file.
- New topic → new filename.

## Out of scope

- Feishu / Figma publish
- Third-party jump VSIX / ContextWeave MCP API keys
- Infra diagrams without code evidence
