# architecture-visualization

Cursor Agent Skill：把系统架构 / 调用链 / 数据同步链路画成 **可交互的 Canvas SVG DAG**，节点点击即可在 IDE 中打开对应源码（`useCanvasAction` → `openFile`）。风格接近 ContextWeave，无需额外 VSIX / MCP。

## 安装（Cursor）

```bash
npx skills add weizewei/architecture-visualization -a cursor -g
```

或手动拷贝到个人 skills 目录：

```bash
cp -R architecture-visualization ~/.cursor/skills/
```

已安装过的可更新：

```bash
npx skills update weizewei/architecture-visualization -g
# 或重新 add 覆盖
```

## 用法

在 Cursor Agent 对话里直接说，例如：

- 「画一下 note 同步到 ES 的架构图」
- 「可视化这条调用链」
- 「architecture diagram / flow diagram」

Agent 会：

1. 按代码真实链路梳理节点与边（不编造不存在的服务）
2. 在 workspace 的 `canvases/` 下生成 `.canvas.tsx`
3. 用 `computeDAGLayout` 渲染 SVG；**点击带 path 的节点**跳转到文件（有 line 则定位到行）
4. 支持悬停高亮、选中详情、Pill 快速跳转

## 仓库结构

| 文件 | 说明 |
|------|------|
| `architecture-visualization/SKILL.md` | Skill 主说明与自动触发条件 |
| `architecture-visualization/canvas-dag.md` | 交互 DAG + `openFile` 模板 |

## 能力边界

- ✅ Canvas 交互图、点击跳源码、悬停/选中、Pill 跳转列表
- ❌ 不依赖飞书 / Figma；不安装第三方跳转插件
- ❌ 不臆造代码里没有的组件或存储

## License

MIT

---

## English

Cursor Agent Skill for interactive architecture / call-chain diagrams as a Canvas SVG DAG, with **click-to-open source** via `useCanvasAction` → `openFile` (ContextWeave-style, no extra VSIX).

**Install**

```bash
npx skills add weizewei/architecture-visualization -a cursor -g
```

**Usage:** Ask for an architecture / flow / 链路图 in Agent chat. The agent traces real code, writes a `.canvas.tsx`, and renders a DAG—click a node (or Pill) to open its source file/line.

**Contents:** `SKILL.md` (instructions + triggers), `canvas-dag.md` (interactive DAG template with hover/select/jump).
