# architecture-visualization

Cursor Agent Skill: interactive architecture / call-chain diagrams as a Canvas SVG DAG, with **click-to-open source** via `useCanvasAction` → `openFile` (ContextWeave-style, no extra VSIX).

## Install (Cursor)

```bash
npx skills add weizewei/architecture-visualization -a cursor -g
```

Or copy into personal skills:

```bash
cp -R architecture-visualization ~/.cursor/skills/
```

## Usage

In Cursor Agent chat, ask for an architecture / flow / 链路图. The agent should:

1. Trace the real code path
2. Write a `.canvas.tsx` under the workspace canvases dir
3. Render a DAG; click a node to jump to the source file/line

## Contents

| File | Role |
|------|------|
| `architecture-visualization/SKILL.md` | Skill instructions (auto-invoke triggers) |
| `architecture-visualization/canvas-dag.md` | Interactive DAG + `openFile` template |

## License

MIT
