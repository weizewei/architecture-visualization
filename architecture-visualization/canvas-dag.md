# Interactive Canvas DAG + openFile

Pattern: SVG graph + click → IDE source (`useCanvasAction`).

## Jump API

```tsx
const dispatch = useCanvasAction();

dispatch({
  type: "openFile",
  path: "/abs/or/workspace-relative/path/File.php",
  selection: {
    startLineNumber: 34,
    startColumn: 1,
    endLineNumber: 34,
    endColumn: 1,
  },
});
```

- Omit `selection` when `line` unknown.
- Prefer **absolute** `path` in multi-root workspaces.

## Template

Replace `NODES` / `EDGES` / title with real traced data. Keep interaction wiring.

```tsx
import {
  computeDAGLayout,
  Stack,
  Row,
  H1,
  H2,
  Text,
  Button,
  Pill,
  Callout,
  useHostTheme,
  useCanvasAction,
  useCanvasState,
  type DAGLayoutEdge,
  type CanvasAction,
} from "cursor/canvas";

type Kind = "entry" | "service" | "event" | "consumer" | "store" | "job" | "other";

type ArchNode = {
  id: string;
  label: string;
  detail?: string;
  path?: string;
  line?: number;
  kind?: Kind;
};

/** Replace with facts from code — no invented boxes. */
const NODES: ArchNode[] = [
  {
    id: "entry",
    label: "Entry",
    detail: "trigger",
    kind: "entry",
    path: "/ABS/path/Entry.php",
    line: 10,
  },
  {
    id: "store",
    label: "Store",
    detail: "persist",
    kind: "store",
    path: "/ABS/path/Store.php",
    line: 20,
  },
];

const EDGES: Array<{ from: string; to: string; label?: string }> = [
  { from: "entry", to: "store", label: "write" },
];

function openSource(dispatch: (a: CanvasAction) => void, node: ArchNode) {
  if (!node.path) return;
  if (node.line && node.line > 0) {
    dispatch({
      type: "openFile",
      path: node.path,
      selection: {
        startLineNumber: node.line,
        startColumn: 1,
        endLineNumber: node.line,
        endColumn: 1,
      },
    });
  } else {
    dispatch({ type: "openFile", path: node.path });
  }
}

export default function ArchitectureCanvas() {
  const theme = useHostTheme();
  const dispatch = useCanvasAction();
  const [selectedId, setSelectedId] = useCanvasState<string | null>("selected-node", null);
  const [hoverId, setHoverId] = useCanvasState<string | null>("hover-node", null);

  const nodeW = 168;
  const nodeH = 56;
  const layout = computeDAGLayout({
    nodes: NODES.map((n) => ({ id: n.id })),
    edges: EDGES.map((e) => ({ from: e.from, to: e.to })),
    direction: "horizontal",
    nodeWidth: nodeW,
    nodeHeight: nodeH,
    rankGap: 72,
    nodeGap: 28,
    padding: 16,
  });

  const byId = Object.fromEntries(NODES.map((n) => [n.id, n]));
  const selected = selectedId ? byId[selectedId] : undefined;
  const edgeLabel = Object.fromEntries(
    EDGES.filter((e) => e.label).map((e) => [`${e.from}->${e.to}`, e.label!]),
  );
  const jumpable = NODES.filter((n) => n.path);

  return (
    <Stack gap={20}>
      <Stack gap={6}>
        <H1>Flow title</H1>
        <Text tone="secondary" size="small">
          Source: codebase · 点击节点打开源码
        </Text>
      </Stack>

      <svg
        width={layout.width}
        height={layout.height}
        style={{ maxWidth: "100%" }}
        onMouseLeave={() => setHoverId(null)}
      >
        <defs>
          <marker
            id="arch-arrow"
            markerWidth="8"
            markerHeight="8"
            refX="6"
            refY="3"
            orient="auto"
          >
            <path d="M0,0 L6,3 L0,6 Z" fill={theme.stroke.primary} />
          </marker>
        </defs>

        {layout.edges.map((e: DAGLayoutEdge, i: number) => {
          const midX = (e.sourceX + e.targetX) / 2;
          const midY = (e.sourceY + e.targetY) / 2;
          const key = `${e.from}->${e.to}`;
          return (
            <g key={i}>
              <line
                x1={e.sourceX}
                y1={e.sourceY}
                x2={e.targetX}
                y2={e.targetY}
                stroke={theme.stroke.primary}
                strokeWidth={1.5}
                strokeDasharray={e.isBackEdge ? "4 4" : undefined}
                markerEnd="url(#arch-arrow)"
              />
              {edgeLabel[key] ? (
                <text
                  x={midX}
                  y={midY - 6}
                  textAnchor="middle"
                  fill={theme.text.secondary}
                  fontSize={11}
                >
                  {edgeLabel[key]}
                </text>
              ) : null}
            </g>
          );
        })}

        {layout.nodes.map((n) => {
          const meta = byId[n.id];
          const canJump = Boolean(meta?.path);
          const isSelected = selectedId === n.id;
          const isHover = hoverId === n.id;
          const stroke = isSelected || meta?.kind === "entry" || meta?.kind === "store"
            ? theme.accent.primary
            : theme.stroke.primary;
          const fill = isSelected || isHover ? theme.fill.tertiary : theme.bg.elevated;

          return (
            <g
              key={n.id}
              transform={`translate(${n.x}, ${n.y})`}
              style={{ cursor: canJump ? "pointer" : "default" }}
              onMouseEnter={() => setHoverId(n.id)}
              onClick={() => {
                setSelectedId(n.id);
                if (meta) openSource(dispatch, meta);
              }}
            >
              <title>
                {meta?.path
                  ? `${meta.path}${meta.line ? `:${meta.line}` : ""}`
                  : meta?.label}
              </title>
              <rect
                width={nodeW}
                height={nodeH}
                rx={6}
                fill={fill}
                stroke={stroke}
                strokeWidth={isSelected ? 2 : 1.5}
              />
              <text
                x={nodeW / 2}
                y={22}
                textAnchor="middle"
                fill={theme.text.primary}
                fontSize={13}
                fontWeight={600}
              >
                {meta?.label ?? n.id}
              </text>
              {meta?.detail ? (
                <text
                  x={nodeW / 2}
                  y={40}
                  textAnchor="middle"
                  fill={theme.text.secondary}
                  fontSize={11}
                >
                  {meta.detail}
                </text>
              ) : null}
            </g>
          );
        })}
      </svg>

      {selected ? (
        <Callout tone="info" title={selected.label}>
          <Stack gap={8}>
            <Text size="small">
              {selected.path
                ? `${selected.path}${selected.line ? `:${selected.line}` : ""}`
                : "该节点无源码路径"}
            </Text>
            {selected.path ? (
              <Row gap={8}>
                <Button variant="primary" onClick={() => openSource(dispatch, selected)}>
                  在编辑器中打开
                </Button>
              </Row>
            ) : null}
          </Stack>
        </Callout>
      ) : (
        <Text tone="secondary" size="small">
          选中节点可跳转源码
        </Text>
      )}

      {jumpable.length > 0 ? (
        <Stack gap={8}>
          <H2>源码跳转</H2>
          <Row gap={8} wrap>
            {jumpable.map((n) => (
              <Pill
                key={n.id}
                active={selectedId === n.id}
                onClick={() => {
                  setSelectedId(n.id);
                  openSource(dispatch, n);
                }}
              >
                {n.label}
              </Pill>
            ))}
          </Row>
        </Stack>
      ) : null}
    </Stack>
  );
}
```

## Shipping checklist

- [ ] Every jumpable node has a real `path` (absolute if multi-root)
- [ ] `line` set when the symbol line is known
- [ ] Click node → `openFile` (+ line selection when known)
- [ ] Hover + selected styles; dashed back-edges
- [ ] Callout + 在编辑器中打开; Pill jump list
- [ ] Caption includes 点击节点打开源码
- [ ] Theme tokens only; no gradients / emojis / box-shadow
- [ ] If `CanvasAction` / selection types fail check, fall back to `path`-only `openFile`

## Notes

- Marker id `arch-arrow` avoids clashes if the page has multiple SVGs.
- Confirm props in `~/.cursor/skills-cursor/canvas/sdk/*.d.ts` when typecheck fails.
- Do not use `Table` for jumps — it has no row `onClick`; use `Pill` / `Button`.
